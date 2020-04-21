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
