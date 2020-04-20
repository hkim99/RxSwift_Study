# RxSwift_Study
RxSwift 정리 메모

## Observable
포인트만. 이벤트를 비동기적으로 생성

- 생명주기. next, completed, error 로 구성
- ObservableType Protocol(Observable<T>)
    - next：최신/다음 데이터 전달
    - completed：성공, 추가 이벤트 생성하지 않음
    - error：에러발생, 추가 이벤트 생성하지 않음

### 생성

#### just
오직 하나의 element를 포함하는 Observable Sequenc를 생성

```swift
let observable: Observable<Int> = Observable<Int>.just(1)
let observable: Observable<String> = Observable<String>.just("1")
```

#### of
가변적인 element를 포함하는 Observable Sequenc를 생성

```swift
let observable: Observable<Int> = Observable<Int>.of(1,2,3,4,5)

// Array를 인자로 넣으면 단일요소를 갖게됨
let observable: Observable<Int> = Observable<Int>.of([1,2,3,4,5])
// [1,2,3,4,5]
```

#### from
Array의 요소들로 Observable Sequence 생성.
타입은 Observable<Int>고, 오직 Array만 취함

```swift
let observable: Observable<Int> = Observable<Int>.from([1,2,3,4,5])
//1
//2
//3
//4
//5
```

#### empty
요소를 가지지 않는 Observable .complete 이벤트만 방출함

```swift
let observable = Observable<Void>.empty()
```

#### never
empty와 반대. 이벤트를 방출조차 하지 않음 (흠;)

```swift
let observable = Observable<Any>.never()
```

#### range
start부터 count크기만큼

```swift
let observable = Observable<Int>.range(start: 3, count: 6)
//3
//4
.
.
//8
```

#### repeatElement
지정된 Element를 계속 방출

```swift
let observable = Observable<Int>.repeatElement(3)
//3
//3
//3
```

#### interval
지정된 시간에 한번씩 이벤트를 방출

```swift
Observable<Int>.interval(3, scheduler: MainScheduler.instance)
//0
//1
//2
.
.
//3초마다 0부터 숫자가 증가
```

#### create
Observer에 직접 이벤트를 방출

```swift
Observable<Int>.create({ (observer) -> Disposable in
        
  observer.onNext(3)
  observer.onNext(4)
  observer.onNext(5)
        
  observer.onCompleted()
        
  return Disposables.create()
})

//print
3
4
5
Completed
```

### 구독
Observable은 그냥 sequence 정의일뿐. 구독되기 전에는 아무런 이벤트도 보내지 않음

#### subscribe

```swift
let observable = Observable.of(1, 2, 3)
observable.subscribe(onNext: { (element) in
  print(element)
 })

// print
1
2
3
completed
```

### Disposing와 종료 

#### dispose()
dispose는 구독을 취소하여 Observable을 종료시킴

```swift
let observable = Observable.of(1, 2, 3)
let subscription = observable.subscribe({ num in
  print(num)
})
subscription.dispose()
```

#### disposeBag()
각각의 구독에 대하여 dispose()하는것은 효율적이지 못하기 때문에 disposeBag()을 사용
여러 observable을 disposeBag()에 모아서 관리한다고 보면 될듯

```swift
let disposeBag = DisposeBag()
let observable = Observable.of(1, 2, 3)
let subscription = observable.subscribe({ num in
  print(num)
})
.disposed(by: disposeBag)
```

###
