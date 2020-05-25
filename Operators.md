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
