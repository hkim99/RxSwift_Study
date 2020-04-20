# RxSwift_Study
RxSwift 정리 메모

## 참고 자료
https://jinshine.github.io/2019/01/01/RxSwift/1.RxSwift란/
https://github.com/fimuxd/RxSwift/blob/master/Lectures/01_HelloRxSwift/Ch.1%20Hello%20RxSwift.md

이분들 자료가 알기쉽고 정리도 잘되어 있어서 도움이 많이됬다..

이직한 회사 프로젝트에서 RxSwift를 사용하고 있어 코드 만지기 시작한지 좀 되었는데 무작정 코드만 보면서 적응할려니 시간이 참 오래 걸리는듯하다.
뭐든 기초부터 차근차근 공부하는게 중요한듯...

## RxSwift 란?
간단히 말하자면 Swift로 FRP(Functional Reactive Programming)가 가능하게 해주는 라이브러리

[설치방법](https://github.com/ReactiveX/RxSwift)

## RxSwift 구성요소
Observables、Operators、Schedulers

### 1.Observables
- Observable<T> 클래스는 Rx의 기초
- `T`타입의 데이터를 전달 (비동기 생성)
- ObservableType Protocol(Observable<T>)
    - next：최신/다음 데이터 전달
    - completed：성공, 추가 이벤트 생성하지 않음
    - error：에러발생, 추가 이벤트 생성하지 않음

#### Finite Observable Sequences (한정된 이벤트)
파일을 다운로드하는 코드

```swift
API.download(file: "https://www...")
 	.subscribe(onNext: { data in
 		... append data to temporary file
 	},
 	onError: { error in 
 		... display error to user
 	},
 	onCompleted: {
 		... use downloaded file
 	})
```
- 다운로드가 시작되고 data를 감시
- 성공하면 onCompleted 와 함께 이벤트 종료, 실패하면 onError 와 함께 이벤트 종료

#### Infinite Observable Sequences(무한한 이벤트)
위의 다운로드와는 달리 무한한 이벤트르 갖는 경우도 있음. UI이벤트같은 경우. 

```swift
UIDevice.rx.orientation
 	.subscribe(onNext: { current in
 		switch current {
 			case .landscape:
 				... re-arrange UI for landscape
 			case .portrait:
 				... re-arrange UI for portrait
 		}
 	})
```
- 유저가 계속해서 조작을 하는한 이벤트 발생
- onError onCompleted 이벤트는 절대 발생할 일이 없기 때문에 생략 가능

### 2. Operators
- 입력된 이벤트값을 독자적으로 처리하여 방출. 여러개를 함께 쓸수있음

```swift
UIDevice.rx.orientation
  .filter { value in
    return value != .landscape
  }
  .map { _ in
    return "Only Portrait"
  }
  .subscribe(onNext: { string in
    showAlert(string)
  })
```
- 위와같이 filter map 등등 입력값을 처리하여 방출(next)

### 3.Schedulers
- dispatch queue 와 동일. 훨씬 강력학 쓰기 쉽다고함..
- RxSwift에서는 여러가지의 스케쥴러가 이미 정의되어 있고, 개발자가 따로 스케쥴러르 생성하는 일은 거의 없다고함

### RxCocoa
UIKit등의 정보를 갖고 RxSwift와 함깨 사용됨. 같이 사용하는것은 거의 필수라고 보면됨
