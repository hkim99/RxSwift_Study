# RxSwift_Study
RxSwift 정리 메모

## Operators
입력된 이벤트값을 독자적으로 처리하여 방출. 여러개를 함께 쓸수있음
map, flatMap, filter 등등

### Filter
원하는 값을 덛거나 거를때 사용. Bool을 반환( true면 값을 포함, false면 배제 )

```swift
let disposeBag = DisposeBag()
Observable.of(1,2,3,4,5,6)
     .filter({ (int) -> Bool in
        int % 2 == 0
      })
      .subscribe(onNext: {
         print($0)
      })
      .disposed(by: disposeBag)
// [2, 4]
```
### .ingnoreElement()
.next 이벤트를 무시 .completed .error 같은 종료 이벤트는 허용함

```swift
let disposeBag = DisposeBag()
    let ignore = PublishSubject<String>()
    ignore
        .ignoreElements()
        .subscribe({ _ in
            print("Completed")
        })
        .disposed(by: disposeBag)
    
    ignore.onNext("1")
    ignore.onNext("2")
    ignore.onNext("3")
    
    ignore.onCompleted()
```
- ignore.onNext("") 에서는 아무일도 일어나지 않음
- ignore.onCompleted 이벤트에서 subscribe 됨

### .elementAt()
지정한 index의 이벤트만 발생함

```swift
let elementAt = PublishSubject<String>()
    elementAt
        .elementAt(2)
        .subscribe({ (str) in
            print(str)
        })
        .disposed(by: disposeBag)
    
    elementAt.onNext("1")
    elementAt.onNext("2")
    elementAt.onNext("3")

// next(3)
// completed
```

### .skip
지정한 n개의 요소를 skip

```swift
Observable.of("A", "B", "C", "D")
    .skip(2)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// C D 가 출력됨
```

### .skipWhile
fileter 와는 반대로 조건이 flase 일때만 이벤트를 방출하며 조건을 충족하여 한번 skip 하면 더이상 상관없이 이벤트 방출

```swift
Observable.of(2, 2, 3, 4, 4)
    .skipWhile({ (int) -> Bool in
        int % 2 == 0
    })
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// 3 4 4 가 출력됨
```

### .skipUntil
다른 observable 의 이벤트가 발생할때까지 자신의 이벤트를 skip

```swift
let subject = PublishSubject<String>()
let trigger = PublishSubject<String>()
    
    subject
        .skipUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    subject.onNext("A")
    subject.onNext("B")
    
    trigger.onNext("X")
    
    subject.onNext("C")

// C 가 출력됨
```
이유는 trigger 이벤트가 발생하기전까지의 자신의 이벤트는 전부 skip 했기때문

### .take
skip 의 반대 개념

```swift
Observable.of(1,2,3,4,5)
    .take(2)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// 1 2 가 출력됨
```

### .takeWhile
skipWhile 의 반대 개념. true 만 출력됨

```swift
Observable.of(2, 2, 3, 4, 4)
    .skipWhile({ (int) -> Bool in
        int % 2 == 0
    })
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// 2 2 가 출력됨
```

### .takeUntil
skipUntil 와는 반대로 다른 observable 의 이벤트가 발생하기 전까지의 이벤트를 받는다

```swift
let subject = PublishSubject<String>()
let trigger = PublishSubject<String>()
    
    subject
        .takeUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    subject.onNext("A")
    subject.onNext("B")
    
    trigger.onNext("X")
    
    subject.onNext("C")

// A B 가 출력됨
```
이유는 trigger 이벤트가 발생하기전까지의 자신의 이벤트는 전부 skip 했기때문

### .distinctUntilChanged
중복해서 이어지는 값을 무시

```swift
Observable.of(2, 2, 3, 4, 4)
    .distinctUntilChanged()
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// 2 3 4 가 출력됨
```

### toArray
Observable의 요소들을 배열로 변환

```swift
Observable.of(2, 3, 4)
    .toArray()
    .subscribe({
        print($0)
    })
    .disposed(by: disposeBag)

// [2, 3, 4] 가 출력됨
```

### map
Observable 에서 동작 하는 그냥 Swift 일반적으로 쓰이는 map랑 같음

```swift
Observable.of(2, 3, 4)
    .map{ $0 * 2 }
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// [2, 3, 4] 가 출력됨
```

### flatMap
1차원 배열에서 nil 제거. Swift 4.1이상에서는 compactMap 쓰라고 나옴

```swift
let array = [1, 2, 3, nil, 5]
    let flatMap = array.flatMap{ $0 }
    print(flatMap)

// [1, 2, 3, 5] 가 출력됨
```

2차원 배열의 경우. 1차원 배열로 만듬. nil 은 제거하지 않음
```swift
let array = [[1, 2, nil],[4, nil],[6, 7, nil, 9]]
    let flatMap = array.flatMap{ $0 }
    print(flatMap)

// [Optional(1), Optional(2), nil, Optional(4), nil, Optional(6), Optional(7), nil, Optional(9)] 가 출력됨
```

### compactMap
1차원 배열에서 nil 제거

```swift
let array = [1, 2, 3, nil, 5]
    let compactMap = array.compactMap{ $0 }
    print(compactMap)

// [1, 2, 3, 5] 가 출력됨
```

2차원 배열의 경우. nil 은 제거하지 않음
```swift
let array = [[1, 2, nil],[4, nil],[6, 7, nil, 9]]
    let compactMap = array.compactMap{ $0 }
    print(compactMap)

// [[Optional(1), Optional(2), nil], [Optional(4), nil], [Optional(6), Optional(7), nil, Optional(9)]] 가 출력됨
```

### flatMap
가장 최근의 observable 에서 나오는 값만 받음

```swift
struct Student {
    var score: BehaviorSubject<Int>
}

let ryan = Student(score: BehaviorSubject(value: 80))
    let charlotte = Student(score: BehaviorSubject(value: 90))
    let student = PublishSubject<Student>()
    
    student
        .flatMapLatest { $0.score }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    student.onNext(ryan)
    ryan.score.onNext(85)
    
    student.onNext(charlotte)
    
    ryan.score.onNext(95)
    charlotte.score.onNext(100)

// 80 85 90 100 가 출력됨
```

### compactMap
1차원 배열에서 nil 제거

```swift
let array = [1, 2, 3, nil, 5]
    let compactMap = array.compactMap{ $0 }
    print(compactMap)

// [1, 2, 3, 5] 가 출력됨
```

2차원 배열의 경우. nil 은 제거하지 않음

```swift
let array = [[1, 2, nil],[4, nil],[6, 7, nil, 9]]
    let compactMap = array.compactMap{ $0 }
    print(compactMap)

// [[Optional(1), Optional(2), nil], [Optional(4), nil], [Optional(6), Optional(7), nil, Optional(9)]] 가 출력됨
```

### flatMapLatest
가장 최근의 observable 에서 나오는 값만 받음

```swift
struct Student {
    var score: BehaviorSubject<Int>
}

let ryan = Student(score: BehaviorSubject(value: 80))
    let charlotte = Student(score: BehaviorSubject(value: 90))
    let student = PublishSubject<Student>()
    
    student
        .flatMapLatest { $0.score }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    student.onNext(ryan)
    ryan.score.onNext(85)
    
    student.onNext(charlotte)
    
    ryan.score.onNext(95)
    charlotte.score.onNext(100)

// 80 85 90 100 가 출력됨
```

### startWith(_:)

초기값을 설정할수 있음

```swift
Observable.of(2, 3, 4)
    .startWith(1)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

// 1, 2, 3, 4 가 출력됨
```

### Observable.concat(_:)

sequence 를 연결함

```swift
let first = Observable.of(1, 2, 3)
let second = Observable.of(4, 5, 6)
let observable = Observable.concat([first, second])

observable
	.subscribe(onNext: {
    print($0)
	})
	.disposed(by: disposeBag)

// 1, 2, 3, 4, 5, 6 가 출력됨
```

### concat(_:)

Observable.concat(_:) 과 같은 방법

```swift
let first = Observable.of(1, 2, 3)
let second = Observable.of(4, 5, 6)
let observable = first.concat(second)
observable.subscribe(onNext: { print($0) })

// 1, 2, 3, 4, 5, 6 가 출력됨
```

### concatMap(_:)

flatMap 과 concat  을 합쳐놓은거갓은것

flatMap 과 concatMap의 차이점은 concatMap 은 순서가 보장된다는 것

```swift
    let sequences = [
        "123": Observable.of("1", "2", "3"),
        "ABC": Observable.of("A", "B", "C")
    ]
    let observable = Observable.of("123", "ABC")
    
    observable.flatMap {
        data in sequences[data] ?? .empty()
        
    }.subscribe({ print($0.element ?? "") })
// 1, 2, A, 3, B, C 가 출력됨
    
    observable.concatMap({ (data) in
        return sequences[data] ?? .empty()
    }).subscribe({ print($0.element ?? "") })
// 1, 2, 3, A, B, C 가 출력됨 <- 순서 보장
```

### merge()

sequence 합치기 가장 쉬운 방법

각각이 요소들이 도착하는대로 받아서 방출. 순서가 보장되지는 않는듯

```swift
    let first = Observable.of(1, 2, 3)
    let second = Observable.of(4, 5, 6)
    
    Observable.of(first, second)
        .merge()
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
// 1, 2, 4, 3, 5, 6 가 출력됨
```

### merge(maxConcurrent:)

합칠수 있는 sequence의 수를 제한할때 사용

limit에 도달한 이후에 들어오는 obsevable을 대기열에 넣고, 현재 sequence중 하나가 완료되자마자 구독을 시작

리소스를 제한하거나 할때 사용됨

```swift
struct Player {
    init(score: Int) {
        self.score = BehaviorSubject(value: score)
    }
    let score: BehaviorSubject<Int>
}

		let hk = Player(score: 80)
    let cj = Player(score: 90)
    let pk = Player(score: 50)

    Observable.of(hk.score, cj.score, pk.score)
    .merge(maxConcurrent: 2)
    .subscribe(onNext: {
        print("merge  : \($0)")
    }, onError: { error in
        print("merge error")
    })
    .disposed(by: disposeBag)

    hk.score.onNext(85)
    pk.score.onNext(55)
    cj.score.onNext(100)
    hk.score.onCompleted()
    pk.score.onNext(60)
    cj.score.onNext(86)

/* 
merge  : 80
merge  : 90
merge  : 85
merge  : 100
merge  : 55
merge  : 60
merge  : 86
*/
```

## 이하 아직 코드 수정/실행 안해봄

### combineLatest(::resultSelector:)

결합된 sequence를 방출할때마다, 제공한 클로저르 호출하며 각각의 내부 sequence들의 최종값을 받음

```swift
     let left = PublishSubject<String>()
     let right = PublishSubject<String>()

     let observable = Observable.combineLatest(left, right, resultSelector: { lastLeft, lastRight in
         "\(lastLeft) \(lastRight)"
     })

     let disposable = observable.subscribe(onNext: {
         print($0)
     })

     print("> Sending a value to Left")
     left.onNext("Hello,")
     print("> Sending a value to Right")
     right.onNext("world")
     print("> Sending another value to Right")
     right.onNext("RxSwift")
     print("> Sending another value to Left")
     left.onNext("Have a good day,")

     disposable.dispose()

     /* Prints:
      > Sending a value to Left
      > Sending a value to Right
      Hello, world
      > Sending another value to Right
      Hello, RxSwift
      > Sending another value to Left
      Have a good day, RxSwift
     */
```

### zip

또다른 결합 연산자. 밑의 예에서 Vienna가 출력됮 않은 이유는.
둘중 하나의 observable이랃 완료되면 zip도 같이 완료되기 때문

```swift
     enum Weatehr {
         case cloudy
         case sunny
     }

     let left:Observable<Weatehr> = Observable.of(.sunny, .cloudy, .cloudy, .sunny)
     let right = Observable.of("Lisbon", "Copenhagen", "London", "Madrid", "Vienna")

     let observable = Observable.zip(left, right, resultSelector: { (weather, city) in
         return "It's \(weather) in \(city)"
     })

     observable.subscribe(onNext: {
         print($0)
     })

     /* Prints:
      It's sunny in Lisbon
      It's cloudy in Copenhagen
      It's cloudy in London
      It's sunny in Madrid
      */
```

### withLatestFrom(_:)

여러개의 이벤트를 한번에 받는경우. 최신값만 출력.

```swift
     let button = PublishSubject<Void>()
     let textField = PublishSubject<String>()

     let observable = button.withLatestFrom(textField)
     _ = observable.subscribe(onNext: { print($0) })

     textField.onNext("Par")
     textField.onNext("Pari")
     textField.onNext("Paris")
     button.onNext(())
     button.onNext(())
// Paris 가 출력됨
```

### sample(_:)

`withLatestFrom(_:)` 와 거의 똑같이 작동하지만 한번마 방출.
// onNext 여러번 해도 한번만 방출하고 종료된다는 뜻인가??

```swift
     let button = PublishSubject<Void>()
     let textField = PublishSubject<String>()

     let observable = textField.sample(button)

     textField.onNext("Par")
     textField.onNext("Pari")
     textField.onNext("Paris")
     button.onNext(())
     button.onNext(())
// Paris 가 출력됨

// withLatestFrom(_:)을 가지고 sample(_:)처럼 작동하게 하려면 distinctUntilChanged()와 함께 사용하면 된다. 
// 아래의 코드처럼 작성하면 Paris가 한번 출력된다.
let observable = button.withLatestFrom(textField)
     _ = observable
         .distinctUntilChanged()
         .subscribe(onNext: { print($0) })

// withLatestFrom(_:)은 데이터 observable을 파라미터로 받고, sample(_:)은 trigger observable을 파라미터로 받는다. 
// 실수하기 쉬운 부분이니 주의할 것
```

### amb(_:)

두개의 sequence 이벤트 중 어떤것을 구독할지 선택할수 있게 함
둘중 하나가 먼저 이벤트르 방출하며 나머지 하나는 구독을 중단하고 처음 작동한 이벤트만 방출

```swift
        let left = PublishSubject<String>()
 	let right = PublishSubject<String>()

 	let observable = left.amb(right)
 	let disposable = observable.subscribe(onNext: { value in
 		print(value)
 	})

 	left.onNext("Lisbon")
 	right.onNext("Copenhagen")
 	left.onNext("London")
 	left.onNext("Madrid")
 	right.onNext("Vienna")

 	disposable.dispose()
//  가 출력됨
```

### switchLatest(_:)

설정한 observable 의 마지막 sequence 아이템만 구독. flatMapLatest랑 비슷

```swift
     let one = PublishSubject<String>()
     let two = PublishSubject<String>()
     let three = PublishSubject<String>()

     let source = PublishSubject<Observable<String>>()

     let observable = source.switchLatest()
     let disposable = observable.subscribe(onNext: { print($0) })

     source.onNext(one)
     one.onNext("Some text from sequence one")
     two.onNext("Some text from sequence two")

     source.onNext(two)
     two.onNext("More text from sequence two")
     one.onNext("and also from sequence one")

     source.onNext(three)
     two.onNext("Why don't you see me?")
     one.onNext("I'm alone, help me")
     three.onNext("Hey it's three. I win")

     source.onNext(one)
     one.onNext("Nope. It's me, one!")

     disposable.dispose()

     /* Prints:
      Some text from sequence one
      More text from sequence two
      Hey it's three. I win
      Nope. It's me, one!
      */
```

### reduece(::)

Swift 표준 reduce(:_:_) 랑 비슷

```swift
     let source = Observable.of(1, 3, 5, 7, 9)

     let observable = source.reduce(0, accumulator: +)
     observable.subscribe(onNext: { print($0) } )

     // 위와 같은 의미
     let observable2 = source.reduce(0, accumulator: { summary, newValue in
         return summary + newValue
     })
     observable2.subscribe(onNext: { print($0) })
//  25 가 출력됨

// 그냥 Swift로 하면
let array = [1, 3, 5, 7, 9]
let result = array.reduce(0) { $0 + $1 }
print(result) // 25
```

### scan(_:accumulator)

reduce(:_:_) 처럼 작동하지마 리턴값이 Observable

```swift
     let source = Observable.of(1, 3, 5, 7, 9)

     let observable = source.scan(0, accumulator: +)
     observable.subscribe(onNext: { print($0) })

     /* Prints:
      1
      4
      9
      16
      25
     */
```

### replay

지정한 버퍼의 크기만큼 이벤트를 저장하고 전달

```swift
```

### replayAll

모든 이벤트를 저장하고 전달

```swift
```

### buffer(timeSpan:cout:scheduler:)

```swift

```

### delaySubscription(_:scheduler:)

```swift

```

### Observable.interval(_:scheduler:)

```swift

```

### Observable.timer(_:period:scheduler:)

```swift

```
