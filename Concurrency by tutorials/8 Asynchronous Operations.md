# 8 Asynchronous Operations

* 동기적, State machine
* `isReady` 상태가 되면 시스템은 가능한 스레드 검색을 시작할 수 있다는 것을 앎
* 스케쥴러가 실행할 스레드를 찾으면 `isExecuting` 상태로 전환
* 실행되고 완료되면 `isFinished`상태가 됨
* 비동기 작업?
	* 비동기 작업이 시작한 후 바로 main은 종료됨
	* 비동기 메소드가 아직 완료되지 않았기 때문에 `isFinished`로 전환할 수 없음

## Asynchronous operations
* Operation의 실행이 언제 완료되는지 자동으로 결정할 수 없어 상태 변경을 수동으로 관리
* 상태와 관련된 프로퍼티는 모두 읽기 전용
* 모든 비동기 Operation의 base class를 생성
### AsyncOperation
#### State tracking
* 직접 변경 사항을 추적할 수 있는 방법(읽기 전용 -> 읽기/쓰기)
```
extension AsyncOperation {
	enum State: String {
		case ready, executing, finished
		
		fileprivate var keyPath: String {
			return "is\(rawValue.capitalized)"
		}
	}
}
```
* KVO notification 사용
* keyPath: 정의한 State에 `is`키워드를 붙여 Operation의 property와 일치하는 키를 반환
* `fileprivate`: 외부가 아닌 이 파일 전체에서만 사용가능
	* `private`이었다면 enum 밖에서는 사용할 수 없음
* 현재 상태를 유지하는 변수 생성
```
var state = State.ready {
	willSet {
		willChangeValue(forKey: "isExecuting")
		willChangeValue(forKey: "isReady")
	}
	didSet {
		didChangeValue(forKey: "isReady")
		didChangeValue(forKey: "isExecuting")
	} 
}
```
* default state: `ready`
* `state`의 값을 변경하면 4개의 KVO notification을 받음
	1. Will change for isReady.
	2. Will change for isExecuting.
	3. Did change for isReady.
	4. Did change for isExecuting.
* Operation base class의 isExecuting과 isReady 프로퍼티가 모두 변경됨
#### Base properties
* 앞서 구현한 상태 변경을 추적하고 동작했는지 알릴 수 있는 방법으로Operation base class의 해당 인스턴스 메소드를 override에 사용
```
override var isReady: Bool {
	return super.isReady && state == .ready
}
override var isExecuting: Bool {
	return state == .executing
}
override var isFinished: Bool {
	return state == .finished
}
```
* 스케쥴러가 사용할 스레드를 찾을 준비가 되었는지 여부를 결정하는 동안 진행되는 모든것을 인식하지 못하기 때문에 base class의 `isReady` 메소드에 대한 확인을 포함


* 비동기로 Operation이 동작할 것을 지정
```
override var isAsynchronous: Bool {
	return true
}
```
#### Starting the operation
* `start`메소드 구현
	* (수동으로 실행하든 operation queue에 의해서든) `main` 메소드를 호출
	* 상태를 `.executing` 으로 변경
```
override func start() {
	main()
	state = .executing
}
```
* `super.start()` 메소드를 호출하지 않음(공식문서 참고)
	* 실행 상태를 업데이트
	* operation이 실제로 실행될 수 있는지 확인
		* 취소되었거나 완료된 경우(10장 Canceling Operations)
		* 현재 실행중이거나 실행할 준비가 되지 않은 경우
	* `Your custom implementation must not call super at any time`
	* 실행 환경을 구성
	* 작업의 상태를 추적하고 적절한 상태 전환을 제공
		* 작업이 실행되고 작업을 완료할 때, 각각 isExecuting과 isFinished 키 경로에 대한 KVO 알림을 생성
* 추상 클래스 -> 직접 이 class를 사용하지 않도록 함(AsyncOperation을 서브클래스해서 사용)
### Math is fun!
* 중요한 것은 비동기 작업이 완료되면 작업의 상태를 수동으로 .finished로 설정해야 한다는 것
* 상태를 변경하는 것을 잊으면 완료로 표시되지 않고, 무한 루프가 됨

## Networked TiltShift
### NetworkImageOperation
* 작업에 대한 요구 사항
	1. URL이나 실제 URL을 나타내는 문자열을 사용해야 합니다.
	2. 지정된 URL에서 데이터를 다운로드해야 합니다.
	3. URLSession-type 완료 핸들러가 제공되는 경우, 디코딩 대신 사용하세요.
	4. 성공하면 완료 핸들러가 없으며 이미지입니다. 선택적 UIImage 값을 설정해야 합니다.
* AsyncOperation을 서브클래싱하고 클래스에 필요한 변수를 선언
```
import UIKit
typealias ImageOperationCompletion = ((Data?, URLResponse?,
Error?) -> Void)?
final class NetworkImageOperation: AsyncOperation {
	var image: UIImage?
	private let url: URL
	private let completion: ImageOperationCompletion
}
```
* `completion` signiture는 URLSession 메소드의 것과 동일, optional

```
init(
	url: URL,
	completion: ImageOperationCompletion = nil) {
	self.url = url
	self.completion = completion
	super.init()
}
convenience init?(
	string: String,
	completion: ImageOperationCompletion = nil) {
	guard let url = URL(string: string) else { return nil }
	self.init(url: url, completion: completion)
}
```
* URL로 변환할 수 없는 문자열을 전달하면 생성자는 nil을 반환

```
override func main() {
	URLSession.shared.dataTask(with: url) {
		[weak self] data, response, error in
	}.resume() 
}
```
* `weak` capture group 사용: 비동기 작업, 객체가 제거될 수 있음

* 완료된 작업을 처리
```
guard let self = self else { return }
defer { self.state = .finished }
if let completion = self.completion {
	completion(data, response, error)
	return
}
guard error == nil, let data = data else { return }
self.image = UIImage(data: data)
```
* Defer를 사용하면 작업을 완료로 표시, 앞으로 어떤 변화를 일으킬지 절대 알 수 없음

* 만약 실패한다면, 이미지 속성은 nil이 될 것이고, 잘못되었다는 것을 알게 될 것
### Using NetworkImageFilter
```
private var urls: [URL] = []
override func viewDidLoad() {
	super.viewDidLoad()
	guard let plist = Bundle.main.url(forResource: "Photos",
                                    withExtension: "plist"),
		let contents = try? Data(contentsOf: plist),
		let serial = try?
PropertyListSerialization.propertyList(
                          from: contents,
                          format: nil),
		let serialUrls = serial as? [String] else {
			print("Something went horribly wrong!")
			return 
		}
	urls = serialUrls.compactMap(URL.init)
}
```
* .plist 파일의 내용을 읽고 문자열을 실제 URL 객체로 변환
* compactMap: 모든 nil 항목을 제외하고 unwrapped non-optional 값만 반환 -> 유효한 URL 객체를 포함한 배열

```
let image = UIImage(named: "\(indexPath.row).png")!
let op = TiltShiftOperation(image: image)
``
->
```
let op = NetworkImageOperation(url: urls[indexPath.row])
```

```
cell.display(image: op.outputImage)
```
->
```
cell.display(image: op.image)
```
