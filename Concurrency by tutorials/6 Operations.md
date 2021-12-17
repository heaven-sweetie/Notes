# 6. Operations

## Reusability
* Swift object
* 작업 단위를 묶을 수 있고 미래에 실행하거나 쉽게 한번 이상 실행할 수 있음

## Operation states
* State machine: Read-only Boolean property
	* `isReady`: 초기화되고 실행될 준비가 된 상태
	* `isExecuting`: `start`메소드를 호출한 후의 상태
	* `isCancelled: `isFinished` 상태가 되기전에 App이 `cancel` 메소드를 호출한 상태
	* `isFinished`:  취소되지 않고 완료된 상태

## BlockOperation
* default global queue에서 하나 이상의 클로저의 _동시_ 실행을 관리
* 별도의 `DispatchQueue`를 생성하지 않고 `OperationQueue`를 사용하는 객체 지향적인 wrapper를 제공
* KVO 노티피케이션의 이점을 활용할 수 있음
* dependencies and everything else that an Operation provides
* 모든 클로져가 완료되었을 때 스스로를 끝내는 것으로 표시한다는 점에서 dispatch group과 유사하게 작용
* Task가 동시적으로 실행되는데, 연속적으로 실행하려면 별도의 DispatchQueue에 submit하거나 종속성(dependency)를 설정
### Multiple block operations
* `addExecutionBlock` method: `BlockOperation`에 closure를 추가
* `completionBlock`closure:  operation에 추가된 모든 closure가 완료되면 실행

## Subclassing operation
* BlockOperation 클래스는 간단한 작업에 적합
* 더 복잡한 작업을 수행하거나 재사용 가능한 구성 요소를 수행하는 경우 직접 Operation을 하위 클래스화해야 함
### Tilt shift the wrong way
* '실패란 가장 위대한 스승이다'
### Tilt shift almost correctly"
* Operation으로 작업 이동
* 변경되지 않을 것은 `initializer`로 전달 받아 `private`으로 지정
* `CIContext`: Thread-safe
* `main` method를 override해서 동작할 작업을 구현
	* operation이 start될 때 호출
* `start` method를 바로 호출하면 **현재 쓰레드(`main thread`)**에서 **동기적**으로 동작
	* 작업이 시작될 준비가 되지 않은 경우 exception이 발생할 수 있음
	* `start` method를 수동으로 호출하면 안됨