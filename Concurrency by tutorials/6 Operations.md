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
### Multiple block operations


## Subclassing operation
### Tilt shift the wrong way
### Tilt shift almost correctly"