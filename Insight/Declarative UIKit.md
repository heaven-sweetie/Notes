# Declarative UIKit

---

## SwiftUI의 선언적(declarative) 구문

* 화면을 구성하는 컴포넌트들의 레이아웃 모양과 이에 대한 디테일 내역을 직접 설계하는 대신에, 단순하면서도 직관적인 구문을 이용하여 화면을 기술
* 레이아웃의 위치와 제약조건, 그리고 렌더링 방법에 대한 복잡한 사항들은 모두 SwiftUI 내부적으로 처리
* 시뮬레이터 혹은 실기기에 빌드하고 실행하지 않아도 프리뷰 캔버스에서 확인할 수 있음
* 상태가 변경되면 UI는 자동으로 업데이트 됨

등등 여러 이점이 있음

---
  
하지만 바로 적용할 수 있을까?

---

🙅

---

그러면 일부를 채용해보는 것은 어떨까?

---

## 1. Preview




---

## 2. 메소드 채이닝

```
let alertController = UIAlertController(
  title: "", 
  message: "", 
  preferredStyle: .alert
)
alertController.addAction(
  UIAlertAction(
	title: NSLocaliedString("확인", comment: "확인"), 
	style: .default,
	handler: { _ in
      Log.l("Confirmed")
	}
  )
)
alertController.addAction(
  UIAlertAction(
	title: NSLocaliedString("취소", comment: "취소"), 
	style: .cancel,
	handler: { _ in
      Log.l("Cancelled")
	}
  )
)
self.present(self, animated: true, completion: completion)
```

---

* 속성을 적용하는 메소드가 객체 스스로를 반환

```
typealias AlertAction = (UIAlertAction) -> Void

extension UIAlertController {

  func confirm(title: String? = nil, action: AlertAction? = nil) -> Self {
    addAction(
      UIAlertAction(
        title: title ?? NSLocaliedString("확인", comment: "확인"), 
        style: .default,
        handler: action
      )
    )
  }
  
  func cancel(title: String? = nil, action: AlertAction? = nil) -> Self {
    addAction(
      UIAlertAction(
        title: title ?? NSLocaliedString("취소", comment: "취소"), 
        style: .cancel,
        handler: action
      )
    )
  }

  func present(in presenting: UIViewController, completion: (() -> Void)? = nil) {
    presenting.present(self, animated: true, completion: completion)
  }
}

```

---

```
UIAlertController(title: "", message: "", preferredStyle: .alert)
  .confirm { _ in
    Log.l("Confirmed")
  } 
  .cancel { _ in
    Log.l("Cancelled")
  }
  .present(in: self)
```

---
## 참고

 * [UIKit으로 SwiftUI(처럼)만들기](https://www.youtube.com/watch?v=DRRAV3pyQJ8&list=PL03rJBlpwTaA-RiPm1m8R8xajSaG0SNq-)
	 * 메소드 체이닝을 포함한 UIKit + SnapKit + RxSwift를 이용해서 SwiftUI처럼 구현하는 방법을 설명하는 영상
 * [당장 적용하는 SwiftUI의 아이디어](https://youtu.be/7fanl8FrAbY)
	 * RxSwift와 UIKit 조합으로 SwiftUI에서와 같은 MVVM 아키텍쳐 구현 설명하는 영상
 * [MihaelIsaev/UIKitPlus](https://github.com/MihaelIsaev/UIKitPlus)
	 * UIKit을 사용하면서 반응형 UI처럼 구현할 수 있게 해주는 오픈소스 프레임워크