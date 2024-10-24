[https://developer.apple.com/documentation/swift/asyncstream](https://developer.apple.com/documentation/swift/asyncstream)

## 개요

`AsyncStream`은 `AsyncSequence`를 준수하여 비동기 반복자를 수동으로 구현하지 않고도 비동기 시퀀스를 쉽게 생성할 수 있는 방법을 제공합니다. 특히, 비동기 스트림은 콜백 또는 위임 기반 API를 `async`-`await`과 함께 사용할 수 있도록 적응시키는 데 적합합니다.

`AsyncStream`을 초기화할 때, `AsyncStream.Continuation`을 받는 클로저를 사용합니다. 이 클로저 안에서 요소를 생성한 후, `continuation`의 `yield(_:)` 메서드를 호출하여 스트림에 요소를 제공합니다. 더 이상 생성할 요소가 없으면 `continuation`의 `finish()` 메서드를 호출하여 스트림이 `nil`을 반환하고, 이는 시퀀스를 종료시킵니다. `Continuation`은 `Sendable`을 준수하므로, `AsyncStream`의 반복과는 별개의 동시성 컨텍스트에서 호출할 수 있습니다.

요소의 임의의 소스는 호출자가 요소를 반복하는 것보다 더 빠르게 요소를 생성할 수 있습니다. 이러한 이유로 `AsyncStream`은 버퍼링 동작을 정의하여 스트림이 가장 오래된 또는 가장 최신의 특정 수의 요소를 버퍼링할 수 있도록 합니다. 기본적으로 버퍼 한도는 `Int.max`로 설정되어 있으며, 이는 값에 제한이 없다는 것을 의미합니다.

## 기존 코드를 스트림으로 바꾸기

기존 콜백 코드를 `async`-`await`으로 적응하려면, 콜백을 사용하여 `continuation`의 `yield(_:)` 메서드를 통해 스트림에 값을 제공하면 됩니다.

가상의 `QuakeMonitor` 타입을 고려해 보겠습니다. 이 타입은 지진이 감지될 때마다 호출자에게 `Quake` 인스턴스를 제공합니다. 호출자가 콜백을 받기 위해서는 `quakeHandler` 속성에 커스텀 클로저를 설정해야 하며, 이 클로저는 모니터가 필요할 때 호출됩니다.

```swift
class QuakeMonitor {
		var quakeHandler: ((Quake) -> Void)?
		
		func startMonitoring() {…}
		func stopMonitoring() {…}		
}
```

이를 `async`-`await`으로 적응시키기 위해 `QuakeMonitor`를 확장하여 `AsyncStream<Quake>` 타입의 `quakes` 속성을 추가할 수 있습니다. 이 속성의 `getter`에서 `AsyncStream`을 반환하며, 이 스트림을 생성하는 클로저에서는 다음 단계를 수행합니다:

1. `QuakeMonitor` 인스턴스를 생성합니다.
2. 모니터의 `quakeHandler` 속성에 각 `Quake` 인스턴스를 받아서 이를 `continuation`의 `yield(_:)` 메서드를 호출하여 스트림에 전달하는 클로저를 설정합니다.
3. `continuation`의 `onTermination` 속성에 모니터의 `stopMonitoring()`을 호출하는 클로저를 설정합니다.
4. `startMonitoring()`을 호출하여 모니터링을 시작합니다.

```swift
extension QuakeMonitor {
		static var quakes: AsyncStream<Quake> {
		    AsyncStream { continuation in
		        let monitor = QuakeMonitor()
		        monitor.quakeHandler = { quake in
		            continuation.yield(quake)
		        }
		        continuation.onTermination = { @Sendable _ in
		             monitor.stopMonitoring()
		        }
		        monitor.startMonitoring()
		    }
		}
}
```

이 스트림은 `AsyncSequence`이기 때문에 호출 지점에서 `for`-`await`-`in` 구문을 사용하여 스트림이 생성하는 각 `Quake` 인스턴스를 처리할 수 있습니다:

```swift
for await quake in QuakeMonitor.quakes {
		print("Quake: \\(quake.date)")
}
print("Stream finished.")
```