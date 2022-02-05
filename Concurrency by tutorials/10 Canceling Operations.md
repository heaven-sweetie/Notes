# 10. Canceling Operations

* Operation은 사용자가 화면을 떠나거나 테이블 뷰의 셀에서 멀리 스크롤하는 등 결과를 보지 못할 경우 취소할 수 있음

## The magic of cancel

* Operation을 Operation Queue에 예약하면, 더 이상 제어할 수 없음
*  Queue에 추가되면 만들 수 있는 유일한 변경 사항은 Operation의 cancel 메소드를 호출하는 것
* `cancel()` 호출하면 
	* isCancelled computed property는 true를 반환
	* 다른 일은 자동으로 일어나지 않음
* Operation을 취소하는 것은 OS에 어떤 의미인가요?
	* Operation은 단순히 예외를 던져야 하는가?
	* 청소가 필요한가요?
	* 실행 중인 network call를 취소할 수 있나요?
	* 작업이 중단되었음을 알리기 위해 서버 측을 보내는 메시지가 있나요?
	* 작업이 멈추면, 데이터가 손상될까요?
* 취소가 요청되었음을 식별하는 플래그를 설정하는 것이 자동으로 가능한 전부인 이유를 알 수 있음
* Operation의 기본 `start` 구현은 먼저 isCancelled 플래그가 참인지 확인하고, 있는 경우 즉시 종료

## Cancel and cancelAllOperations

* `OperationQueue`에 정의된 `cancelAllOperations` 메소드를 이용하면 모든 Operation을 취소할 수 있음

## Updating AsyncOperation

```
// AsyncOperation.swift

override func start() {
	main()
	state = .executing
}
```
* 적절한 위치에서 `isCancelled`변수의 값을 확인
```
// AsyncOperation.swift

override func start() {
	if isCancelled {
		state = .finished
		return 
	}
	main()
	state = .executing
}
```
* 위 수정으로 operation이 시작하기 전에 취소할 수 있음

## Canceling a running operation

* 실행중인 operation을 취소하려면 `isCancelled`를 전반적으로 체크해야함

```
// NetworkImageOperation.swift
// main(): after `defer`

guard !self.isCancelled else { return }
```
* 이미지 다운로드가 완료된 시점에서 취소하고 반환하지 않을 것인지, 아니면 생성할지 프로젝트의 요구 사항에 따라 다르게 구현

* 네트워크 요청이 진행되는 동안 취소하는 방법을 추가
```
// NetworkImageOperation.swift

private var task: URLSessionDataTask?
```
* 네트워크 작업이 실행되는 동안 유지

```
// NetworkImageOperation.swift
// main()

task = URLSession.shared.dataTask(with: url) {
	[weak self]
```

```
// NetworkImageOperation.swift
// main()

task?.resume()
```

```
// NetworkImageOperation.swift

override func cancel() {
  super.cancel()
  task?.cancel()
}
```
---
```
// TiltShiftOperation.swift
// main(): 
// 		* before setting the `fromRect`variable
// 		* before setting `outpuImage`

guard !isCancelled else { return }
```
---

* 스크롤할 때 셀에 대한 작업이 취소되도록 이것을 테이블 뷰에 연결
```
// TiltShiftTableViewController.swift

private var operations: [IndexPath: [Operation]] = [:]
```
* 특정 cell의 operations(Download 및 tilt shifting)을 유지

```
// TiltShiftTableViewController.swift
// tableView(_:cellForRowAt:): before `return cell`

if let existingOperations = operations[indexPath] {
	for operation in existingOperations {
		operation.cancel()
	}
}
operations[indexPath] = [tiltShiftOp, downloadOp]
```
* 이미 해당 index path에 operation이 있는 경우 취소하고 새로운 operation을 저장

```
// TiltShiftTableViewController.swift

override func tableView(
	_ tableView: UITableView,
	didEndDisplaying cell: UITableViewCell,
	forRowAt indexPath: IndexPath) {
	if let operations = operations[indexPath] {
		for operation in operations {
			operation.cancel()
		}
	}
 }
```
* 셀이 화면을 벗어날 때 호출되는 table view의 delegate 메소드
	*  -> 디바이스의 리소스를 보이는 셀에서만 사용할 것을 보장 
	*  -> 네트워크 트래픽과 배터리 수명을 절약하고 앱을 더 빠르게 실행
* 이제 테이블 뷰를 빠르게 스크롤하면 앱이 화면을 빠르게 지나간 각 셀에 대한 이미지를 로드하고 필터링하지 않음

## Where to go from here?

* prefetching delegate(UITableView, UICollectionView / iOS 10 +)
	* controller가 prefetch할 때 operation을 생성
	* controller가 prefeching을 취소하면 operation도 취소