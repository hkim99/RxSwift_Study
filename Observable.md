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

### Traits

Traits는 일반적인 Observable보다 좁은 범위의 Observable을 선택적으로 사용할수 있음

코드 가독성을 높일 수 있음

#### 종류

Single, Maybe, Completable

##### Single
- .success(value) or .error 이벤트를 방출
- .success(value) = .next + .completed
- 성공 또는 실패같은 1회성 프로세스에서 사용

##### Completable
- 오직 .completed or .error 만을 방출
- 연산이 제대로 완료되었는지 확인하고 싶을때 등에 사용

##### Maybe
- Single 와 Completable을 섞어 놓은것
- .success(value), .completable, .error 모두 방출 가능
- 프로세스가 성공, 실패 여부와 함께 출력값도 방출할수 있을때

##### 샘플
```swift
     let disposeBag = DisposeBag()
     enum FileReadError: Error {
         case fileNotFound, unreadable, encodingFailed
     }
     func loadText(from name: String) -> Single<String> {
         return Single.create{ single in
             let disposable = Disposables.create()
             guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
                 single(.error(FileReadError.fileNotFound))
                 return disposable
             }
             guard let data = FileManager.default.contents(atPath: path) else {
                 single(.error(FileReadError.unreadable))
                 return disposable
             }
             guard let contents = String(data: data, encoding: .utf8) else {
                 single(.error(FileReadError.encodingFailed))
                 return disposable
             }
             single(.success(contents))
             return disposable
         }
     }
     
     loadText(from: "Hello")
         .subscribe{
             switch $0 {
             case .success(let string):
                 print(string)
             case .error(let error):
                 print(error)
             }
         }
         .disposed(by: disposeBag)
```

### Challenges

#### 부수작용 구현 do 연산자
- 이벤트 변화 없이 어떤 작업을 추가 가능 
- do는 subscribe는 가지고 있지 않는 onSubscribe 핸들러를 갖고 있음
- do 연산자를 이용할 수 있는 method는 do(onNext:onError:onCompleted:onSubscribe:onDispose)

```swift
     let observable = Observable<Any>.never()
     let disposeBag = DisposeBag()
     observable.do(
         onSubscribe: { print("Subscribed")}
         ).subscribe(
             onNext: { (element) in
                 print(element)
         },
             onCompleted: {
                 print("Completed")
         }
     )
```
- do의 onSubscribe 에서 구독했음을 표시하는 문구를 프린트
