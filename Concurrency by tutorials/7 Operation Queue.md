# 7. Operation Queues

* Operation Queue: Operation의 일정과 동시에 실행할 수 있는 최대 작업 수를 관리
* Operation Queue에 Operation을 추가하는 3가지 방법
	* Operation 전달
	* closure 전달
	* Operation의 배열 전달
* 하나의 Operation은 동기 작업 -> 메인스레드 외의 GCD queue에 추가하여 비동기 작업이 가능하지만 OperationQueue 로도 모든 비동기적 이점을 얻을 수 있음

## OperationQueue management
* OperationQueue는 ready 상태의 operation을 QoS에 따라 실행
* 동일한 Operation을 OperationQueue에 추가하면 다른 OperationQueue에 추가할 수 없음
	* 여러번 실행이 필요한 경우 subclass를 만들 수 있음
### Waiting for completion
* `waitUntilAllOperationsAreFinished`: 
	* wait == block
	* 현재 스레드를 block
	* main thread 에서 호출하면 안됨
	* 필요하다면 private serial DispatchQueue를 설정(blocking 되지 않도록)
* `addOperations(_:waitUntilFinished:)`: 모든 Operation의 완료를 기다릴 필요없이 operation을 설정
### Quality of service
* DispatchGroup에서의 QoS와 비슷함
* 기본값: `.background`
* OperationQueue에서 QoS를 설정할 수 있지만 개별 Operation의 QoS로 재정의(override)될 수 있음
### Pausing the queue
* 일시정지: `isSuspended` property = true
* 진행 중인 Operation은 계속 진행, 새로 추가된 작업은 예약되지 않음
### Maximum number of operations
* `maxConcurrentOperationCount`:
	* 동시에 진행할 최대 Operation수 를 제한
	* 기본적으로는 디바이스가 할 수 있는 만큼만 진행
	* `1`을 설정한 경우 serial queue를 효과적으로 생성
### Underlying DispatchQueue
* OperationQueue에 operation을 추가하기 전에 기존 DispatchQueue를 underlyingQueue로 지정할 수 있음
* Qos: DispatchQueue -> OperationQueue
* main queue를 underlyingQueue로 지정하면 안됨
## Fix the previous project
### UIActivityIndicator
### Updating the table
* OperationQueue를 property로 생성
* cell을 구성할 때 Operation을 queue에 추가
* cell을 구성하는 추가적인 구현은 Operation의 completionBlock에 추가: DispatchQueue.main closure로 감쌈
* 직접 `start` method를 호출하지 않아도 queue에 의해서 알아서 실행됨