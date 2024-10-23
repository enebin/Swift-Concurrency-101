[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/)

## Concurrency

Swift는 비동기 및 병렬 코드를 구조적으로 작성할 수 있는 내장 지원 기능을 제공합니다. 비동기 코드는 한 번에 하나의 프로그램만 실행되지만 일시 중단했다가 나중에 다시 시작할 수 있습니다. 프로그램에서 코드를 일시 중단하고 다시 실행하면 UI 업데이트와 같은 단기 작업을 계속 진행하면서 네트워크를 통한 데이터 가져오기나 파일 분석과 같은 장기 작업도 동시에 수행할 수 있습니다.

병렬 코드는 여러 코드 조각이 동시에 실행되는 것을 의미합니다. 예를 들어, 4코어 프로세서를 가진 컴퓨터는 4개의 코드 조각을 동시에 실행할 수 있으며 각 코어는 하나의 작업을 수행합니다. 병렬 및 비동기 코드를 사용하는 프로그램은 여러 작업을 동시에 수행하고 외부 시스템을 기다리는 작업은 일시 중단합니다.

병렬 또는 비동기 코드에서 제공하는 추가적인 스케줄링 유연성은 복잡성이 증가하는 비용도 수반합니다. Swift는 컴파일 타임에 일부 검사를 가능하게 하는 방식으로 의도를 표현할 수 있게 해줍니다. 예를 들어, 액터(`actor`)를 사용하여 안전하게 변경 가능한 상태에 접근할 수 있습니다.

그러나 느리거나 버그가 있는 코드에 동시성을 추가한다고 해서 반드시 빠르거나 올바르게 동작하는 것은 아닙니다. 실제로 동시성을 추가하면 코드를 디버깅하기가 더 어려워질 수도 있습니다. 그러나 동시성이 필요한 코드에 Swift의 언어 수준 지원을 사용하면 Swift는 컴파일 타임에 문제를 잡을 수 있도록 도와줍니다.

이 장의 나머지 부분에서는 비동기 및 병렬 코드를 조합한 일반적인 개념을 설명하기 위해 '동시성(concurrency)'이라는 용어를 사용합니다.

<aside> 💡

**Note**

이전에 동시성 코드를 작성해본 경험이 있다면, 스레드를 사용해 작업하는 것에 익숙할 수 있습니다. Swift의 동시성 모델은 스레드 위에 구축되어 있지만 스레드를 직접 다루지는 않습니다.

Swift에서 비동기 함수는 자신이 실행되고 있던 스레드를 포기할 수 있으며 이는 첫 번째 함수가 차단된 동안 다른 비동기 함수가 해당 스레드에서 실행될 수 있게 해줍니다. 비동기 함수가 재개될 때 Swift는 해당 함수가 어떤 스레드에서 실행될지에 대해 보장하지 않습니다.

</aside>

### 비동기 코드를 사용하면 좋은 부분

Swift 언어 차원의 지원을 사용하지 않고도 동시성 코드를 작성하는 것이 가능하지만 그러한 코드는 읽기 어려운 경향이 있습니다. 예를 들어, 다음 코드는 사진 이름 목록을 다운로드하고 해당 목록에서 첫 번째 사진을 다운로드한 후 그 사진을 사용자에게 보여줍니다:

```swift
listPhotos(inGallery: "Summer Vacation") { photoNames in
    let sortedNames = photoNames.sorted()
    let name = sortedNames[0]
    downloadPhoto(named: name) { photo in
        show(photo)
    }
}
```

이 단순한 경우에서도 코드를 일련의 컴플리션 핸들러로 작성해야 하기 때문에 중첩된 클로저를 작성하게 됩니다. 이런 스타일에서는 복잡한 코드가 깊이 중첩되면 금방 다루기 어려워질 수 있습니다.

## Defining and Calling Asynchronous Functions

_비동기 함수_ 또는 _비동기 메서드_는 실행 중간에 일시 중단될 수 있는 특별한 종류의 함수 또는 메서드입니다. 이는 일반적인 동기 함수 및 메서드와 대조적입니다.

동기 함수 및 메서드는 완료될 때까지 실행되거나, 오류를 발생시키거나, 또는 반환하지 않습니다. 비동기 함수나 메서드도 결국은 이러한 세 가지 중 하나를 수행하지만, 무엇인가를 기다릴 때 중간에 일시 중단될 수 있습니다. 비동기 함수나 메서드의 본문에서는 코드 실행이 일시 중단될 수 있는 위치를 표시해야 합니다.

### 비동기 메서드 작성하기

함수 또는 메서드가 비동기임을 나타내기 위해서는 매개변수 뒤에 `async` 키워드를 선언합니다. 이는 오류를 던지는 함수를 표시할 때 `throws`를 사용하는 방식과 유사합니다. 함수나 메서드가 값을 반환하는 경우, 반환 화살표(`->`) 앞에 `async`를 작성합니다. 예를 들어, 아래는 갤러리에서 사진 이름을 가져오는 방법입니다:

```swift
func listPhotos(inGallery name: String) async -> [String] {
    let result = // ... 일부 비동기 네트워킹 코드 ...
    return result
}
```

함수가 비동기이면서 오류를 던질 수 있는 경우, `throws` 앞에 `async`를 작성합니다.

### `await` 키워드의 사용

비동기 메서드를 호출할 때는 해당 메서드가 반환될 때까지 실행이 일시 중단됩니다. 일시 중단 가능한 지점을 표시하기 위해 호출 앞에 `await`를 작성합니다. 이는 오류가 발생할 때 프로그램 흐름의 변화 가능성을 표시하기 위해 던지는 함수를 호출할 때 `try`를 작성하는 것과 유사합니다.

비동기 메서드 내부에서는 다른 비동기 메서드를 호출할 때만 실행 흐름이 중단되며, 일시 중단은 암시적으로 또는 [선점적](https://ko.wikipedia.org/wiki/%EC%84%A0%EC%A0%90_%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81)으로 발생하지 않습니다. 즉, 모든 가능한 일시 중단 지점은 `await`로 표시됩니다. 코드에서 모든 가능한 일시 중단 지점을 명시적으로 표시하면 동시성 코드가 더 읽기 쉽고 이해하기 쉬워집니다.

예를 들어, 아래 코드는 갤러리의 모든 사진 이름을 가져온 후 첫 번째 사진을 사용자에게 보여줍니다:

```swift
let photoNames = await listPhotos(inGallery: "Summer Vacation")
let sortedNames = photoNames.sorted()
let name = sortedNames[0]
let photo = await downloadPhoto(named: name)

show(photo)
```

`listPhotos(inGallery:)`와 `downloadPhoto(named:)` 함수는 모두 네트워크 요청을 해야 하기 때문에 완료하는 데 시간이 걸릴 수 있습니다. 반환(return) 화살표 앞에 `async`를 작성해 두 함수를 비동기로 만들면, 이 코드를 실행하는 동안 앱의 나머지 코드가 계속 실행될 수 있습니다.

위 코드의 동시적인 성격을 이해하기 위해 실행 순서의 한 가지 예를 들어 보겠습니다:

1. 코드가 첫 번째 줄부터 실행되고 첫 번째 `await`에 도달할 때까지 실행됩니다. 그런 다음 `listPhotos(inGallery:)` 함수를 호출하고, 해당 함수가 반환될 때까지 실행이 일시 중단됩니다.
2. 이 코드의 실행이 일시 중단된 동안, 동일한 프로그램의 다른 동시 코드가 실행됩니다. 예를 들어, 백그라운드에서 실행 중인 장기 작업이 새로운 사진 갤러리 목록을 계속 업데이트할 수 있습니다. 이 코드는 다음 일시 중단 지점(즉, `await`로 표시된 곳) 또는 완료될 때까지 실행됩니다.
3. `listPhotos(inGallery:)`가 반환되면, 이 코드는 그 시점에서 다시 실행되어 반환된 값을 `photoNames`에 할당합니다.
4. `sortedNames`와 `name`을 정의하는 줄은 일반적인 동기 코드입니다. 이 줄들에는 `await`가 없으므로 일시 중단 지점이 없습니다.
5. 다음 `await`는 `downloadPhoto(named:)` 함수를 호출하는 지점입니다. 이 코드는 해당 함수가 반환될 때까지 다시 실행을 일시 중단하여, 다른 동시 코드가 실행될 수 있게 합니다.
6. `downloadPhoto(named:)`가 반환된 후, 반환된 값이 `photo`에 할당되고, `show(_)` 함수 호출 시 인수로 전달됩니다.

코드에서 `await`로 표시된 일시 중단 지점은 현재 코드가 비동기 함수 또는 메서드가 반환될 때까지 실행을 일시 중단할 수 있음을 나타냅니다. 이는 "스레드를 양보한다(yield the thread)"라고도 불리는데, 내부적으로 Swift가 현재 스레드에서 코드 실행을 중단하고 해당 스레드에서 다른 코드를 실행하는 것을 의미합니다.

`await`이 있는 코드는 실행을 일시 중단할 수 있어야 하므로 프로그램에서 비동기 함수 또는 메서드를 호출할 수 있는 위치는 제한되어 있습니다:

- 비동기(asynchronous) 함수, 메서드 또는 프로퍼티의 body 안
- `@main`으로 표시(mark)된 구조체, 클래스, 열거형의 `static main()` 메서드 안
- 아래의 [Unstructured Concurrency](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency#Unstructured-Concurrency)와 같은 비구조적 자식 태스크(Unstructured child task) 코드

### 명시적으로 Task 일시 중단: `yield`

명시적으로 일시 중단 지점을 추가하려면 `Task.yield()` 메서드를 호출합니다.

```swift
func generateSlideshow(forGallery gallery: String) async {
    let photos = await listPhotos(inGallery: gallery)
    for photo in photos {
        // ... 이 사진에 대한 몇 초간의 비디오 렌더링 ...
        await Task.yield()
    }
}
```

비디오를 렌더링하는 코드가 동기 코드라고 가정하면, 이 코드는 일시 중단 지점을 포함하지 않습니다. 비디오 렌더링 작업은 오래 걸릴 수 있지만, 주기적으로 `Task.yield()`를 호출하여 명시적으로 일시 중단 지점을 추가할 수 있습니다.

이렇게 장기 실행 코드를 구성하면 Swift가 이 작업을 진행하면서 다른 작업도 병행할 수 있게 균형을 맞출 수 있습니다.

→ 참고자료: [https://engineering.linecorp.com/ko/blog/about-swift-concurrency-performance](https://engineering.linecorp.com/ko/blog/about-swift-concurrency-performance)

### 정해진 시간동안 Task 일시 중단: `sleep`

`Task.sleep(for:tolerance:clock:)` 메서드는 동시성이 어떻게 작동하는지 배우기 위한 간단한 코드 작성에 유용합니다.

이 메서드는 지정된 시간만큼 현재 태스크를 일시 중단합니다. 다음은 네트워크 작업을 기다리는 것을 시뮬레이션하기 위해 `sleep(for:tolerance:clock:)`을 사용하는 `listPhotos(inGallery:)` 함수의 버전입니다:

```swift
func listPhotos(inGallery name: String) async throws -> [String] {
    try await Task.sleep(for: .seconds(2))
    return ["IMG001", "IMG99", "IMG0404"]
}
```

위 코드의 `listPhotos(inGallery:)` 버전은 비동기이면서 오류를 던질 수 있습니다. `Task.sleep(for:tolerance:clock:)` 호출에서 오류가 발생할 수 있기 때문입니다.

이 버전의 `listPhotos(inGallery:)`를 호출할 때는 `try`와 `await`를 모두 작성합니다:

```swift
let photos = try await listPhotos(inGallery: "A Rainy Weekend")
```

### 비동기 vs 동기: 서로의 맥락에서 사용할 때

비동기 함수는 오류를 던지는 함수와 몇 가지 유사점이 있습니다. 비동기 함수 또는 오류를 던지는 함수를 정의할 때 `async` 또는 `throws`로 표시하고, 그 함수를 호출할 때는 `await` 또는 `try`를 작성합니다. 비동기 함수는 다른 비동기 함수를 호출할 수 있는 것처럼, 오류를 던지는 함수는 다른 오류를 던지는 함수를 호출할 수 있습니다.

그러나 중요한 차이점이 하나 있습니다. `throw` 코드는 `do-catch` 블록으로 감싸서 오류를 처리하거나 `Result`를 사용하여 다른 코드에서 오류를 처리할 수 있습니다. 이러한 접근 방식은 오류를 던지는 함수를 동기 코드에서 호출할 수 있게 해줍니다. 예를 들어:

```swift
func availableRainyWeekendPhotos() -> Result<[String], Error> {
    return Result {
        try listDownloadedPhotos(inGallery: "A Rainy Weekend")
    }
}
```

반면, 비동기 코드를 감싸 동기 코드에서 안전하게 호출하고 결과를 기다릴 수 있는 방법은 없습니다. Swift 표준 라이브러리는 이러한 불안전(unsafe)한 기능을 의도적으로 제외하였습니다. 이를 직접 구현하면 미묘한 경쟁 상태(subtle races), 스레드 문제, 교착 상태와 같은 문제가 발생할 수 있습니다.

기존 프로젝트에 동시성 코드를 추가할 때는 최상위 계층부터 변환하는 방식으로 작업해야 합니다. 즉, 동시성을 사용할 수 있도록 최상위 계층 코드를 변환한 후, 해당 코드에서 호출하는 함수 및 메서드를 하나씩 변환하며 프로젝트의 아키텍처를 계층별로 처리해 나가야 합니다.

하위 계층부터 변환하는 방식은 불가능합니다. 동기 코드는 비동기 코드를 호출할 수 없기 때문입니다.

## Asynchronous Sequences

이전 섹션의 `listPhotos(inGallery:)` 함수는 배열의 모든 요소가 준비된 후에 전체 배열을 한 번에 비동기적으로 반환합니다. 다른 접근 방식으로는 비동기 시퀀스를 사용하여 컬렉션의 요소를 하나씩 기다리는 방법이 있습니다. 다음은 비동기 시퀀스를 순회하는 방법입니다:

```swift
import Foundation

let handle = FileHandle.standardInput
for try await line in handle.bytes.lines {
		print(line)
}
```

일반적인 `for`-`in` 루프를 사용하는 대신, 위 예제에서는 `for` 뒤에 `await`를 사용합니다. 비동기 함수나 메서드를 호출할 때처럼, `await`을 작성하면 잠재적인 중단 지점을 나타냅니다. `for`-`await`-`in` 루프는 각 반복의 시작 부분에서 다음 요소가 준비될 때까지 실행을 잠재적으로 중단시킬 수 있습니다.

`Sequence` 프로토콜을 준수하도록 추가하여 `for`-`in` 루프에서 사용자 정의 타입을 사용할 수 있는 것처럼, `AsyncSequence` 프로토콜을 준수하도록 추가하여 `for`-`await`-`in` 루프에서도 사용자 정의 타입을 사용할 수 있습니다.

## Calling Asynchronous Functions in Parallel

`await`를 사용하여 비동기 함수를 호출하면 한 번에 하나의 코드만 실행됩니다. 비동기 코드가 실행되는 동안 호출자는 해당 코드가 완료될 때까지 기다린 후 다음 코드를 실행합니다. 예를 들어, 갤러리에서 첫 세 장의 사진을 가져오기 위해서는 다음과 같이 `downloadPhoto(named:)` 함수를 세 번 호출하고 각 호출을 기다릴 수 있습니다:

```swift
let firstPhoto = await downloadPhoto(named: photoNames[0])
let secondPhoto = await downloadPhoto(named: photoNames[1])
let thirdPhoto = await downloadPhoto(named: photoNames[2])

let photos = [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

이 접근 방식에는 중요한 단점이 있습니다. 다운로드는 비동기적으로 진행되며, 진행 중 다른 작업이 가능하지만, `downloadPhoto(named:)` 호출은 한 번에 하나씩만 실행됩니다. 즉, 다음 사진을 다운로드하기 전에 이전 사진 다운로드가 완료되어야 합니다. 하지만 이러한 작업들이 기다릴 필요는 없으며, 각 사진은 독립적으로 또는 동시에 다운로드될 수 있습니다.

### 병렬실행 준비: `async let`

비동기 함수를 호출하고 주변 코드와 병렬로 실행되도록 하려면 상수를 정의할 때 `let` 앞에 `async`를 작성하고, 상수를 사용할 때마다 `await`를 작성합니다.

```swift
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])

let photos = await [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

이 예에서는 세 번의 `downloadPhoto(named:)` 호출이 이전 호출이 완료될 때까지 기다리지 않고 시작됩니다. 시스템 자원이 충분하다면, 이 호출들은 동시에 실행될 수 있습니다.

이 함수 호출들에는 `await`가 없는데, 이는 함수의 결과를 기다리기 위해 실행이 중단되지 않음을 의미합니다. 대신, 프로그램은 `photos`가 정의되는 줄까지 계속 실행되며, 그 시점에 비로소 이 비동기 호출들의 결과가 필요하므로 `await`를 사용하여 세 사진이 모두 다운로드될 때까지 실행을 일시 중지합니다.

이 두 가지 접근 방식의 차이를 다음과 같이 생각할 수 있습니다:

- 이후 코드가 해당 함수의 결과에 의존할 때는 `await`를 사용하여 비동기 함수를 호출합니다. 이는 순차적으로 수행되는 작업을 만듭니다.
- 코드에서 나중에 결과가 필요할 때는 `async let`을 사용하여 비동기 함수를 호출합니다. 이는 병렬로 실행될 수 있는 작업을 만듭니다.
- `await`와 `async let` 모두 중단되는 동안 다른 코드가 실행될 수 있게 합니다.
- 두 경우 모두 비동기 함수가 반환될 때까지 실행이 중단될 수 있음을 나타내기 위해 `await`로 중단 지점을 표시합니다.

또한, 두 가지 접근 방식을 동일한 코드에서 혼합하여 사용할 수도 있습니다.

## Tasks and Task Groups

작업(task)은 프로그램의 일부로 비동기적으로 실행될 수 있는 작업 단위입니다. 모든 비동기 코드는 어떤 작업의 일부로 실행됩니다. 작업 자체는 한 번에 하나의 일만 수행하지만, 여러 작업을 생성하면 Swift는 이를 동시에 실행할 수 있도록 스케줄링할 수 있습니다.

### 부모-자식 태스크와 `TaskGrup`

이전 섹션에서 설명한 `async let` 구문은 암시적으로 자식 작업(child task)을 생성합니다. 이 구문은 프로그램에서 실행할 작업이 이미 정해져 있을 때 유용합니다. 또한, `TaskGroup` 인스턴스를 생성하여 명시적으로 자식 작업을 추가할 수도 있습니다. 이 방식은 우선순위와 취소에 대한 더 많은 제어를 제공하며, 동적으로 작업을 생성할 수 있게 합니다.

작업은 계층적으로 배치됩니다. 주어진 작업 그룹의 각 작업은 동일한 부모 작업을 가지며, 각 작업은 자식 작업을 가질 수 있습니다. 작업과 작업 그룹 간의 명시적 관계 덕분에 이러한 방식을 구조화된 동시성(structured concurrency)이라고 부릅니다. 작업 간의 명시적인 부모-자식 관계는 몇 가지 장점을 제공합니다:

- 부모 작업에서 자식 작업이 완료될 때까지 기다리지 않는 실수를 방지할 수 있습니다.
- 자식 작업에 더 높은 우선순위를 설정할 때 부모 작업의 우선순위도 자동으로 상승됩니다.
- 부모 작업이 취소되면 자식 작업도 자동으로 취소됩니다.
- 작업 로컬 값이 자식 작업에 효율적이고 자동적인 방식으로 전파됩니다.

다음은 여러 장의 사진을 다운로드할 수 있는 코드의 또 다른 버전입니다:

```swift
await withTaskGroup(of: Data.self) { group in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        group.addTask {
            return await downloadPhoto(named: name)
        }
    }

    for await photo in group {
        show(photo)
    }
}
```

위 코드는 새 작업 그룹을 생성한 다음, 갤러리의 각 사진을 다운로드하기 위해 자식 작업을 생성합니다. Swift는 가능한 조건에서 이 작업들을 동시에 실행합니다. 자식 작업 중 하나가 사진 다운로드를 완료하면 해당 사진이 표시됩니다. 자식 작업이 완료되는 순서에 대한 보장은 없으므로, 갤러리의 사진들은 어떤 순서로든 표시될 수 있습니다.

<aside> 💡

**Note**

사진을 다운로드하는 코드에서 오류가 발생할 수 있는 경우, `withThrowingTaskGroup(of:returning:body:)`를 호출해야 합니다.

</aside>

위 코드에서는 각 사진이 다운로드된 후 표시되므로 작업 그룹이 별도의 결과를 반환하지 않습니다. 작업 그룹이 결과를 반환하려면, `withTaskGroup(of:returning:body:)`에 전달하는 클로저 내부에서 결과를 누적하는 코드를 추가합니다.

```swift
let photos = await withTaskGroup(of: Data.self) { group in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        group.addTask {
            return await downloadPhoto(named: name)
        }
    }

    var results: [Data] = []
    for await photo in group {
        results.append(photo)
    }

    return results
}
```

이 예제도 이전 예제처럼 각 사진을 다운로드하기 위해 자식 작업을 생성합니다. 하지만 이전 예제와 달리 `for`-`await`-`in` 루프는 다음 자식 작업이 완료되기를 기다린 후 그 결과를 배열에 추가하고, 모든 자식 작업이 완료될 때까지 이를 반복합니다. 마지막으로 작업 그룹은 다운로드된 사진 배열을 결과로 반환합니다.

## Task Cancellation

[https://developer.apple.com/videos/play/wwdc2021/10134/?time=595](https://developer.apple.com/videos/play/wwdc2021/10134/?time=595)

Swift의 동시성은 협력적인(cooperative) 취소 모델을 사용합니다. 각 작업은 실행 중 적절한 시점에서 취소 여부를 확인하고, 취소된 경우 적절하게 대응합니다. 작업의 성격에 따라 취소에 대한 대응은 다음 중 하나일 수 있습니다:

- `CancellationError`와 같은 오류를 발생시킴
- `nil` 또는 빈 컬렉션 반환
- 부분적으로 완료된 작업 반환

사진의 다운로드 작업은 사진이 크거나 네트워크가 느릴 경우 시간이 오래 걸릴 수 있습니다. 사용자가 모든 작업이 완료되기 전에 작업을 중단할 수 있도록, 각 작업은 취소 여부를 확인하고 취소된 경우 작업을 중단해야 합니다.

작업이 이를 수행하는 방법은 두 가지가 있습니다. `Task.checkCancellation()` 타입 메서드를 호출하거나, `Task.isCancelled` 타입 프로퍼티를 읽는 것입니다.

`checkCancellation()`을 호출하면 작업이 취소된 경우 오류를 발생시키며 오류를 발생시키는 작업은 작업 외부로 오류를 전달하여 작업을 중단할 수 있습니다. 이 방법은 구현과 이해가 간단하다는 장점이 있습니다.

더 유연한 방법을 원한다면, `isCancelled` 프로퍼티를 사용하여 네트워크 연결을 종료하거나 임시 파일을 삭제하는 등의 정리 작업을 할 수 있습니다.

```swift
let photos = await withTaskGroup(of: Optional<Data>.self) { group in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        let added = group.addTaskUnlessCancelled {
            guard !Task.isCancelled else { return nil }
            return await downloadPhoto(named: name)
        }
        guard added else { break }
    }

    var results: [Data] = []
    for await photo in group {
        if let photo { results.append(photo) }
    }
    return results
}
```

위 코드에서는 이전 버전과 몇 가지 다른 점이 있습니다:

- 각 작업은 `TaskGroup.addTaskUnlessCancelled(priority:operation:)` 메서드를 사용해 추가되어 취소 후에 새 작업을 시작하지 않도록 합니다.
- 각 `addTaskUnlessCancelled(priority:operation:)` 호출 후, 새 자식 작업이 추가되었는지 확인합니다. 그룹이 취소된 경우 `added`의 값이 `false`이므로 이 경우 추가적인 사진 다운로드 시도를 중단합니다.
- 각 작업은 사진을 다운로드하기 전에 취소 여부를 확인합니다. 취소된 경우 작업은 `nil`을 반환합니다.
- 마지막에 작업 그룹은 결과를 수집할 때 `nil` 값을 건너뜁니다. 취소를 `nil`을 반환하는 방식으로 처리하면, 작업 그룹은 취소 시 이미 다운로드된 사진들을 반환할 수 있으며, 완료된 작업을 폐기하지 않고 부분 결과를 제공할 수 있습니다.

<aside> 💡

**Note**

작업 외부에서 해당 작업이 취소되었는지 확인하려면, 타입 프로퍼티가 아닌 인스턴스 프로퍼티인 `Task.isCancelled`을 사용하세요.

</aside>

즉각적인 취소 알림이 필요한 작업의 경우, `Task.withTaskCancellationHandler(operation:onCancel:isolation:)` 메서드를 사용합니다.

예를 들어:

```swift
let task = await Task.withTaskCancellationHandler {
    // ...
} onCancel: {
    print("Canceled!")
}

// ... some time later...
task.cancel()  // Prints "Canceled!"
```

취소 핸들러를 사용할 때도 작업 취소는 협력적입니다. 작업은 완료될 때까지 실행되거나, 취소 여부를 확인하고 조기에 중단됩니다. 작업이 취소 핸들러가 시작될 때 여전히 실행 중일 수 있으므로 작업과 취소 핸들러 간에 상태를 공유하는 것을 피해야 합니다. 이는 경쟁 조건을 발생시킬 수 있기 때문입니다.

## Unstructured Concurrency

앞에서 설명한 구조적 동시성 접근 방식 외에도, Swift는 비구조적 동시성(Unstructured Concurrency)도 지원합니다.

태스크 그룹의 일부인 태스크와 달리, 비구조적 태스크는 상위 태스크가 없습니다. 비구조적 태스크는 프로그램이 필요로 하는 방식으로 완전히 자유롭게 관리할 수 있지만, 그만큼 그들의 정확성에 대한 책임도 전적으로 사용자에게 있습니다.

현재 액터에서 실행되는 비구조적 태스크를 생성하려면 `Task.init(priority:operation:)` 이니셜라이저를 호출하세요. 현재 액터의 일부가 아닌, 좀 더 구체적으로 분리된 태스크로 알려진 비구조적 태스크를 생성하려면, `Task.detached(priority:operation:)` 클래스 메서드를 호출하면 됩니다. 이 두 작업 모두 상호작용할 수 있는 태스크를 반환합니다. 예를 들어, 결과를 기다리거나 태스크를 취소할 수 있습니다.

```swift
let newPhoto = // ... some photo data ...

let handle = Task {
		return await add(newPhoto, toGalleryNamed: "Spring Adventures")
}

let result = await handle.value
```

분리된 태스크(Detached Task) 관리에 대한 자세한 내용은 [Task](https://developer.apple.com/documentation/swift/task)를 참조하세요.

## Actors

태스크를 사용하여 프로그램을 격리된(isolated) 동시 실행 가능한 조각으로 나눌 수 있습니다. 태스크는 서로 격리되어 있어 동시에 실행하는 것이 안전하지만, 때로는 태스크 간에 정보를 공유해야 할 때가 있습니다. 액터는 **동시 실행되는 코드 간에 안전하게 정보를 공유할 수 있게 해줍니다.**

클래스처럼 액터도 참조 타입이므로, 값 타입과 참조 타입에 대한 비교가 액터에도 적용됩니다. 하지만 클래스와 달리, 액터는 한 번에 하나의 태스크만 해당 액터의 가변(mutable) 상태에 접근할 수 있도록 하여 여러 태스크에서 동일한 액터 인스턴스와 상호작용할 때도 안전하게 만들어 줍니다. 예를 들어, 온도를 기록하는 액터는 다음과 같습니다:

```swift
actor TemperatureLogger {
    let label: String
    var measurements: [Int]
    private(set) var max: Int

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }
}
```

액터는 `actor` 키워드로 정의하며, 중괄호 안에 해당 정의를 작성합니다. `TemperatureLogger` 액터는 다른 코드가 접근할 수 있는 속성을 가지고 있으며, `max` 프로퍼티는 액터 내부 코드만이 수정할 수 있도록 제한됩니다.

액터 인스턴스를 생성할 때는 구조체나 클래스처럼 이니셜라이저 구문을 사용합니다. 액터의 속성이나 메서드에 접근할 때는, 잠재적인 중지점(suspension point)를 나타내기 위해 `await`를 사용해야 합니다. 예를 들어:

```swift
let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
print(await logger.max)
// "25" 출력
```

이 예에서 `logger.max`에 접근하는 것은 잠재적인 중지점입니다. 액터는 한 번에 하나의 태스크만 가변 상태에 접근할 수 있으므로, 다른 태스크가 이미 `logger`와 상호작용 중이면 이 코드는 속성에 접근하기 위해 대기합니다.

반면, 액터 내부에 속한 코드에서는 속성에 접근할 때 `await`를 사용할 필요가 없습니다. 예를 들어, 새로운 온도로 `TemperatureLogger`를 업데이트하는 메서드는 다음과 같습니다:

```swift
extension TemperatureLogger {
    func update(with measurement: Int) {
        measurements.append(measurement)
        if measurement > max {
            max = measurement
        }
    }
}
```

`update(with:)` 메서드는 이미 액터에서 실행 중이므로, `max` 같은 속성에 접근할 때 `await`를 사용하지 않습니다. 이 메서드는 액터가 한 번에 하나의 태스크만 상태에 접근하도록 제한하는 이유 중 하나를 보여줍니다. 액터의 상태를 업데이트할 때 일시적으로 불변 조건이 깨질 수 있기 때문입니다.

`TemperatureLogger` 액터는 온도 목록과 최대 온도를 기록하고, 새로운 측정값을 기록할 때 최대 온도를 업데이트합니다. 새로운 측정값을 추가하고 `max`를 업데이트하기 전까지는 일시적으로 일관성이 없는 상태에 놓일 수 있습니다.

여러 태스크가 동시에 같은 인스턴스와 상호작용하는 것을 막음으로써 다음과 같은 문제가 발생하는 것을 방지합니다:

1. 코드가 `update(with:)` 메서드를 호출하여 먼저 `measurements` 배열을 업데이트합니다.
2. 코드가 `max`를 업데이트하기 전에 다른 곳에서 최대값과 온도 배열을 읽습니다.
3. 이후 코드가 `max` 값을 업데이트하여 변경을 완료합니다.

이 경우, 코드가 다른 곳에서 실행되면 `update(with:)` 호출 중 데이터가 일시적으로 유효하지 않은 상태에서 액터에 접근하여 잘못된 정보를 읽게 됩니다. Swift의 액터를 사용할 때 이런 문제를 방지할 수 있는 이유는, **액터는 한 번에 하나의 작업만 상태에 대해 작동하도록 허용**하며, ****그 작업은 **오직 `await`로 표시된 중단 지점에서만 인터럽트될 수 있기 때문**입니다.

`update(with:)` 메서드에는 중단 지점이 없으므로, 업데이트 도중 다른 코드가 데이터를 중간에 접근할 수 없습니다. 액터 외부의 코드가 구조체나 클래스의 속성에 직접 접근하려 하면 컴파일 오류가 발생합니다. 예를 들어:

```swift
print(logger.max)  *// 오류*
```

`await` 없이 `logger.max`에 접근하려고 하면 실패하는데, 이는 액터의 속성이 그 액터의 격리된 로컬 상태의 일부이기 때문입니다. 이 속성에 접근하는 코드는 액터의 일부로 실행되어야 하며 이는 비동기 작업이므로 `await`가 필요합니다. Swift는 액터 내에서 실행되는 코드만 해당 액터의 로컬 상태에 접근할 수 있다는 것을 보장합니다. 이러한 보장은 ’액터 격리(actor isolation)’라고 합니다.

Swift 동시성 모델의 다음 요소들은 가변 상태를 공유하는 코드를 더 쉽게 이해할 수 있도록 돕습니다:

- 중단 가능 지점 사이에 있는 코드는 순차적으로 실행되며, 다른 동시 실행 코드에 의해 중단되지 않습니다.
- 액터의 로컬 상태와 상호작용하는 코드는 오직 해당 액터에서만 실행됩니다.
- 액터는 한 번에 하나의 코드만 실행합니다.

이러한 보장 덕분에, `await`를 포함하지 않는 액터 내부의 코드는 다른 곳에서 일시적으로 유효하지 않은 상태를 관찰할 위험 없이 상태를 업데이트할 수 있습니다. 예를 들어, 아래 코드는 측정된 온도를 화씨에서 섭씨로 변환하는 작업을 수행합니다:

```swift
extension TemperatureLogger {
    func convertFahrenheitToCelsius() {
        measurements = measurements.map { measurement in
            (measurement - 32) * 5 / 9
        }
    }
}
```

위의 코드는 `measurements` 배열을 한 번에 하나씩 변환합니다. `map` 작업이 진행되는 동안 일부 온도는 화씨, 다른 온도는 섭씨로 표시됩니다. 그러나 코드에는 `await`가 포함되어 있지 않으므로 이 메서드에는 잠재적인 일시 중단 지점이 없습니다. 이 메서드가 수정하는 상태는 액터에 속하므로 해당 코드가 액터에서 실행될 때를 제외하고는 코드를 읽거나 수정하지 못하도록 보호합니다. 즉, 단위 변환이 진행되는 동안 다른 코드에서 부분적으로 변환된 온도 목록을 읽을 수 있는 방법이 없습니다.

잠재적인 중단 지점을 생략하여 일시적인 유효하지 않은 상태를 보호하는 액터에 코드를 작성하는 것 외에도 해당 코드를 동기식 메서드로 이동할 수 있습니다. 위의 `convertFahrenheitToCelsius()` 메서드는 동기 메서드이므로 잠재적인 정지 지점이 포함되지 않음이 보장됩니다. 이 기능은 데이터 모델을 일시적으로 불일치하게 만드는 코드를 캡슐화하고 작업을 완료하여 데이터 일관성이 복원되기 전에는 다른 코드를 실행할 수 없다는 것을 코드를 읽는 사람이 더 쉽게 인식할 수 있도록 합니다.

앞으로 이 함수에 동시(concurrent) 코드를 추가하여 일시 중단 지점을 도입하려고 하면 버그가 발생하는 대신 컴파일 타임 오류가 발생하게 됩니다.

## Sendable Types

태스크와 액터를 사용하면 프로그램을 안전하게 동시 실행 가능한 여러 조각으로 나눌 수 있습니다. 태스크나 액터의 인스턴스 내부에서 가변 상태(변수나 속성)를 포함하는 프로그램의 부분을 동시성 도메인(concurrency domain)이라고 합니다. 일부 데이터는 동시성 도메인 간에 공유될 수 없는데, 이는 해당 데이터가 가변 상태를 포함하면서도 중첩 접근을 방지하지 않기 때문입니다.

동시성 도메인 간에 공유될 수 있는 타입을 **`sendable` **타입**이라고 합니다. 예를 들어, 이는 액터 메서드를 호출할 때 인자로 전달되거나 태스크의 결과로 반환될 수 있습니다.

앞서 설명된 예제에서는 전달되는 데이터가 동시성 도메인 간에 안전하게 공유될 수 있는 간단한 값 타입을 사용했기 때문에 `sendability`에 대해 언급하지 않았습니다. 반면에, 일부 타입은 동시성 도메인 간에 안전하게 전달될 수 없습니다. 예를 들어, 가변 속성을 포함하고 해당 속성에 대한 접근을 직렬화하지 않은 클래스는 서로 다른 태스크 간에 인스턴스를 전달하면 예측할 수 없거나 잘못된 결과를 초래할 수 있습니다.

타입을 `sendable`로 표시하려면 해당 타입이 `Sendable` 프로토콜을 준수하도록 선언해야 합니다. 이 프로토콜은 코드 요구 사항이 없지만, Swift가 강제하는 의미적(semantic) 요구 사항이 있습니다. 일반적으로 타입이 `sendable`이 되는 방법은 세 가지입니다:

1. 타입이 값(value) 타입이고, 해당 타입의 가변 상태가 다른 `sendable` 데이터를 구성하는 경우. 예를 들어, `sendable` 속성을 가진 저장 속성을 포함하는 구조체나 `sendable` 연관 값을 가진 `enum`.
2. 타입이 가변 상태를 가지지 않고, 해당 불변 상태가 다른 `sendable` 데이터로 이루어진 경우. 예를 들어, 읽기 전용 속성만 가진 구조체나 클래스.
3. 타입이 가변 상태의 안전성을 보장하는 코드를 포함하는 경우. 예를 들어, `@MainActor`로 표시된 클래스나 특정 스레드 또는 큐에서 프로퍼티에 대한 접근을 직렬화(serialize, 순차실행)하는 클래스.

자세한 의미적 요구 사항은 `Sendable` 프로토콜 [링크](https://developer.apple.com/documentation/swift/sendable)에서 확인할 수 있습니다.

일부 타입은 항상 `sendable`입니다. 예를 들어, `sendable` 속성만 가진 구조체나 `sendable` 연관 값(associated value)만 가진 열거형이 있습니다. 다음은 그 예입니다:

```swift
struct TemperatureReading: Sendable {
    var measurement: Int
}

extension TemperatureLogger {
    func addReading(from reading: TemperatureReading) {
        measurements.append(reading.measurement)
    }
}

let logger = TemperatureLogger(label: "Tea kettle", measurement: 85)
let reading = TemperatureReading(measurement: 45)
await logger.addReading(from: reading)
```

`TemperatureReading`은 `sendable` 속성만 가진 구조체이며, `public`이나 `@usableFromInline`으로 표시되지 않았기 때문에 암시적으로 `sendable`로 간주됩니다. 다음은 `Sendable` 프로토콜의 준수를 암시하는 구조체의 버전입니다:

```swift
struct TemperatureReading {
    var measurement: Int
}
```

타입을 명시적으로 `sendable`이 아니라고 표시하고 `Sendable` 프로토콜의 암시적 준수를 무효화하려면 `extension`을 사용하십시오:

```swift
struct FileDescriptor {
    let rawValue: CInt
}

**@available(*, unavailable)**
extension FileDescriptor: Sendable { }
```

위 코드는 POSIX 파일 디스크립터를 감싸는 일부 코드를 보여줍니다. 파일 디스크립터의 인터페이스는 열린 파일과 상호작용하기 위해 정수 값을 사용하지만, 파일 디스크립터는 동시성 도메인 간에 안전하게 전달될 수 없습니다.

위 코드에서 `FileDescriptor`는 암시적으로 `sendable`이 될 수 있는 구조체입니다. 그러나 확장을 통해 `Sendable` 준수를 사용할 수 없게 만들어, 해당 타입이 `sendable`로 사용되지 않도록 방지합니다.

## Additional

- AsyncStream