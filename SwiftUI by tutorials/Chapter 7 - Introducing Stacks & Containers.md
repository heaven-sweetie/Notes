# Introducing Stacks & Containers
## Layout and priorities
* UIKit, AppKit에서 View에 제약을 걸기위해 AutoLayout을 사용
* 정적으로 크기를 설정하지 않는 한 부모가 자식의 크기를 결정
* AutoLayout: 보수적인(conservative)모델
* SwiftUI: 부모가 제안한 크기에 반응하여 **스스로의 크기를 선택**

### Layout for views with a single child
* 모든 View는 기본적으로 부모의 중앙에 배치됨
* 가능한 부모의 제안을 채택하려고 하지만 view가 어떤 타입이냐에 따라 다름
* `Text`
	* 
* `Image`
	* 
* `.padding`
	* 

## Stack views
### Layout for container views
* *가장 제한적인 제약 조건*을 가진 자식 view를 선택하거나 동등한 제약 조건의 경우 가장 작은 크기를 가진 자식 View를 선택
### Layout priority
* Container View은 제한 정도에 따라 정렬(제한적인 제약조건이 가장 많은 것에서 가장 적은 것으로…), 동등한 경우 가장 작은 것을 우선
#### Modifier
* `Image`
	* 가장 적응력이 낮은 구성요소
	* `resizable` modifier를 사용하면 부모로부터 제한된 어떤 크기도 맹목적으로 받아들임
* `Text`
	* 적응력이 매우 높음
	* `lineLimit`modifier를 최대 줄 수를 강제하면 적응력이 떨어짐
#### Priority
* `.layoutPriority`modifier
* 양수 또는 음수의 `Double` 값을 사용 (기본값: `0`)
* `Stack`은 절대 최대값부터 절대 최저 순으로 처리
* 수동으로 설정하면 정렬 순서 뿐만 아니라 제안된 크기도 변함
### The HStack and the VStack
* alignment: 하위view가 어떻게 정렬될지 결정(기본값: `.center`)
* spacing: 하위view 사이의 간격(기본값: `nil` - platform마다 기본값이 사용됨)
	* `0`을 원한다면 명시적으로 설정
* content:
	* `@ViewBuilder` 속성: closure가 여러 하위 view를 반환할 수 있게 함
#### A note on alignment
* `VStack`:  `.center`, `.leading`, `.trailing`
* `HStack`: `.center`, `.top`, `.bottom`
	* `firstTextBaseline`: 가장 위 텍스트 기준 view를 기준으로 정렬
	* `lastTextBaseline`: 가장 아래 텍스트 기준 view를 기준으로 정렬
	* -> 크기나 폰트가 다른 text를 정렬할 때 유용함
### The ZStack
### Other container views

## Back to Kuchi
### The Congratulations View
### Completing the challenge view
### Reworking the App Launch

## The Lazy Stacks
### Practice History

## Key points


## Where to go from here?



#06_study/SwiftUI_by_Tutorial/Introducint_Stacks_&_Containers