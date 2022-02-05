# 9. Operation Dependencies

* Operation 간의 종속성의 두 가지 이점
	1. 선행되는 Operation이 완료된 후 종석된 Operation이 시작하는 것을 보장
	2. 첫 번째 Operation에서 두 번째 Operation으로 데이터를 자동으로 전달하는 깨끗한 방법을 제공
* Operation간의 종속성을 활성화하는 것이 `Operation`을 사용하기로 선택한 주된 이유 중 하나

## Modular design 

* 두 개의 작업(네트워크에서 다운로드, Tilt Shift)을 하나의 Operation으로 만들 수 있지만 좋은 아키텍쳐 설계는 아님(재사용성, 하나의 작업만 수행 등) -> 장기적으로 유지보수를 어렵게 만든다는 설명(다운로드 방식을 변경한다던가…)

## Specifying dependencies

* 추가: `addDependency(op:)`
```
let networkOp = NetworkImageOperation()
let decryptOp = DecryptOperation()
let tiltShiftOp = TiltShiftOperation()

decryptOp.addDependency(op: networkOp)
tiltShiftOp.addDependency(op: decryptOp)
```
* 제거: `removeDependency(op:)`
```
tiltShiftOp.removeDependency(op: decryptOp)
```
* 종속된 Operation의 배열: `dependencies`(read-only)

### Avoiding the pyramid of doom

* GCD를 사용하면 파멸의 피라미드
* 읽기도 어렵고 오류 체크나 retain cycle을 고려한다면 더욱 어려울 듯
```
let network = NetworkClass()
network.onDownloaded { raw in
  guard let raw = raw else { return }

  let decrypt = DecryptClass(raw)
  decrypt.onDecrypted { decrypted in
    guard let decrypted = decrypted else { return }
    
    let tilt = TiltShiftClass(decrypted)
    tilt.onTiltShifted { tilted in
      guard let tilted = tilted else { return }
    }
  }
}
```

## Watch out for deadlock

* Operation이 다른 Operation에 의존할 때 어떤 형태로든 loop를 만들면 교착 상태에 빠질 수 있음
* no silver-bullet -> re-architect

## Passing data between operations

* Protocol
* Operation의 이점 중 일부는 그들이 제공하는 캡슐화와 재사용성

### Using protocols

* "이 Operation이 끝나면, 모든 것이 잘 진행된다면, `UIImage` 유형의 이미지를 제공할 것입니다."
```
// ImageDataProvider.swift
import UIKit

protocol ImageDataProvider {
  var image: UIImage? { get }
}
```

* `UIImage`의 출력이 있는 모든 Operation은 해당 프로토콜을 구현해야 	함
	* Property 이름이 같다면 삶이 쉬워질 것
	* `TiltShiftOperation`의 경우 `CIFilter` 명명 규칙에 따라 `outputImage`라고 정의한 경우 의 방법을 `Adding extensions`에서 확인

### Adding extensions

* Property 이름이 같은 경우
```
// NetworkImageOperation.swift
extension NetworkImageOperation: ImageDataProvider {}
```
* 클래스 정의에 `ImageDataProvider`를 추가할 수 있었지만, 스위프트 스타일 가이드는 대신 `Extension` 접근 방식을 권장

* `TiltShiftOperation`의 경우. 이미 출력 이미지가 있지만, Property 이름은 protocol에 의해 정의된 `이미지`가 아님
```
// TiltShiftOperation.swift
extension TiltShiftOperation: ImageDataProvider {
  var image: UIImage? { return outputImage }
}
```

* 외부 프레임워크를 사용하는 경우 ThirdPartyOperation+Extension.swift와 같은 프로젝트 내의 파일에 Extension을 직접 추가할 수 있음

### Searching for the protocol

* `TiltShiftOperation`은 입력으로 `UIImage`가 dependencies의 출력을로 제공하는지 확인
```
// TiltShiftOperation.swift
// main()
let dependencyImage = dependencies
  .compactMap { ($0 as? ImageDataProvider)?.image }
  .first

guard let inputImage = inputImage ?? dependencyImage else {
  return
}
```

* 이미지의 종속성 체인을 확인하기 때문에, 입력 이미지를 제공하지 않고 `TiltShiftOperation`을 초기화하는 방법
```
// TiltShiftOperation.swift
init(image: UIImage? = nil) {
  inputImage = image
  super.init()
}
```


## Updating the table view controller

```
// TiltShiftTableViewController.swift
// tableView(_:cellForRowAt:)

let downloadOp = NetworkImageOperation(url: urls[indexPath.row])
let tiltShiftOp = TiltShiftOperation()
tiltShiftOp.addDependency(downloadOp)
```

```
tiltShiftOp.completionBlock = {
  DispatchQueue.main.async {
    guard let cell = tableView.cellForRow(at: indexPath) 
      as? PhotoCell else { return }

    cell.isLoading = false
    cell.display(image: tiltShiftOp.image)
  }
}
```

```
queue.addOperation(downloadOp)
queue.addOperation(tiltShiftOp)
```
* Queue는 종속성을 추적하고 다운로드가 완료된 후에만 Tilt Shift Operation을 시작함

### Custom completion handler

*  `Operation` 클래스에서 제공하는 기본 `completionBlock`을 사용
* 결과로 나온 image를 main queue로 다시 보내기 위한 추가 작업 필요 -> custom completion block
```
// TiltShiftOperation.swift
/// Callback which will be run *on the main thread* 
/// when the operation completes.
var onImageProcessed: ((UIImage?) -> Void)?
```

```
// TiltShiftOperation.swift
// main()
if let onImageProcessed = onImageProcessed {
  DispatchQueue.main.async { [weak self] in
    onImageProcessed(self?.outputImage)
  }
}
```

```
// TiltShiftTableViewController.swift
// `tableView(_:cellForRowAt:)`
tiltShiftOp.onImageProcessed = { image in
  guard let cell = tableView.cellForRow(at: indexPath) 
    as? PhotoCell else {
    return
  }

  cell.isLoading = false
  cell.display(image: image)
}
```
* 기능적으로나 성능상 차이가 없지만 가능한 retain cycle에 대한 혼동을 제거하고 기본 UI 스레드에서 자동으로 올바르게 작동하도록 보장함
* completion handler가 operation queue가 제공한 스레드 대신 main UI 스레드에서 실행되고 있다는 사실을 `문서화`하는 것이 매우 중요
* 최종 사용자는 응용 프로그램에 영향을 미칠 수 있는 일을 하지 않도록 스레드를 전환하고 있다는 것을 알아야 합니다.
* `/` 세개로 주석을 달면 Xcode가 Quick Help Inspector에 표시해줌
