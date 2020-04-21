#  RxSwift_Study
RxSwift 정리 메모

## 디버깅
RxSwift는 디버깅 할려면 참 뭐같다

### debug 연산자
- 먼저 debug연산자를 사용하여 이벤트를 출력하는 방법

```swift
let observable = Observable<Any>.never()
    let disposeBag = DisposeBag()
    observable
    	.debug("debug!")
    	.subscribe()
    	.disposed(by: disposeBag)
```
