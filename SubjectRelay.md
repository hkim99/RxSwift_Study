# RxSwift_Study
RxSwift 정리 메모

## Subject
- Observable + Observer = Subject
- 실시간으로 Observable에 새로운 값을 추가하고 subscriber에 방출하는 것
- Observable 이자 Observer 인 Subject 라는..
- PublishSubject, BehaviorSubject, ReplaySubject

### PublishSubject
- PublishSubject 는 .completed, .error 이벤트가 발생할때까지(종료될때까지) subscribe 한 이후부터 이벤트를 방출
- 구독된 순간 새로운 이벤트 수신을 알리고 싶을때 주로 사용

```swift
let subject = PublishSubject<Int>()

subject.onNext("Hi~") // subscribe 하기전이므로 밑에 print 출력되지 않음

let subjectOne = subject
    .subscribe(onNext: { (num) in
        print("subjectOne :",num)
    })

subject.onNext(1)
subject.onNext(2)

let subjectTwo = subject
    .subscribe(onNext: { (num) in
        print("subjectTwo :", num)
    })

subject.onNext(3)
subject.onNext(4)
subject.onNext(5)

/*
 subjectOne : 1
 subjectOne : 2
 subjectOne : 3
 subjectTwo : 3
 subjectOne : 4
 subjectTwo : 4
 subjectOne : 5
 subjectTwo : 5
 */
```

밑에 예는 subject가 종료되면 새로운 subscriber가 생겨도 다시 작동하지 않음(후에 구독한 것도 같이 종료됨)
즉, subject가 종료되었을때 후에 구독한 subscriber에게 종료이벤트를 알려줌

```swift
let subject = PublishSubject<Int>()

let subjectOne = subject
    .subscribe(onNext: { (num) in
        print("subjectOne :",num)
    }, onError: { (error) in
        print("subjectOne Erorr: ",error)
    }, onCompleted: {
        print("subjectOne onCompleted")
    })

subject.onNext(1)
subject.onNext(2)

subject.onCompleted() // 추가

let subjectTwo = subject
    .subscribe(onNext: { (num) in
        print("subjectTwo :",num)
    }, onError: { (error) in
        print("subjectTwo Erorr: ",error)
    }, onCompleted: {
        print("subjectTwo onCompleted")
    })

subject.onNext(3)
subject.onNext(4)
subject.onNext(5)

/*
 subjectOne : 1
 subjectOne : 2
 subjectOne onCompleted
 subjectTwo onCompleted
 */
```

### BehaviorSubject
PublishSubject 와 유사하지만 초기값을 갖는다 (초기값이 필요없다면 PublishSubject를 사용)
진전의 값부터 구독함

```swift
let subject = PublishSubject<Int>()

subject.onNext("Hi~") // 진적의 값

let subjectOne = subject
    .subscribe(onNext: { (num) in
        print("subjectOne :",num)
    })

subject.onNext(1)
subject.onNext(2)

/*
 subjectOne : Hi~
 subjectOne : 1
 subjectOne : 2
 */
```

### RelaySubject
생성시 특정 크기만큼 일시적으로 버퍼를 저장한 다음, 그 크기만큼의(최신) 버퍼를 새 구독자에게 방출

```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)
let disposeBag = DisposeBag()
subject.onNext("1")
subject.onNext("2")
subject.onNext("3")
subject
    .subscribe {
        print(label: "1)", event: $0)
    }
    .disposed(by: disposeBag)

/*
 1) : 2
 2) : 3
 */
```

### Relay
Relay Class는 PublishRelay와 BehaviorRelay클래스가 존재함
RxSwift인 Subject와는 다르게 Relay는 RxCocoa의 클래스

#### PublishRelay
PublishRelay는 PublishSubject의 Wrapper 클래스
PublishSubject 처럼 구독 이후의 발생하는 이벤트만 알수있음

```swift
public final class PublishRelay<Element>: ObservableType {
  private let _subject: PublishSubject<Element>
  public init() {
      _subject = PublishSubject()
  }
}
```

#### BehaviorRelay
BehaviorRelay는 BehaviorSubject의 Wrapper 클래스
.value를 통해서 현재의 값을 가져올 수 있음
Variable이 Deprecate되면서 대신에 BehaviorRelay를 사용

```swift
public final class BehaviorRelay<Element>: ObservableType {
  private let _subject: BehaviorSubject<Element>

  public var value: Element {
      return try! _subject.value()
  }

  public init(value: Element) {
      _subject = BehaviorSubject(value: value)
  }
}
```

### Subject 와 Relay의 차이점
~Subject는 .completed .error 의 이벤트가 발생하면 subscribe가 종료되는 반면
~Relay는 .completed .error 가 발생하지 않고 Dispose되기전까지 계속 작동함(UI Event 에서 사용하기 적절함)
