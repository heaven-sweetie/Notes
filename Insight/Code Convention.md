* 명확성은 항상 간결함을 능가한다
	* `setAlpComp` -> `setAlphaComponents`
* 이름의 각 단어는 속성, 함수 또는 객체에 대한 필수 정보를 전달한다.
	* `priceArray.sortArray()` -> `priceArray.sort()`
* 이름은 절대 자기 참조가 되어서는 안 된다.
	* `firstNameObject` -> `firstName`
* 모든 이름은 camelCase로 작성되었습니다.
	* `maxselectedItems` -> `MaxSelectedItems` -> `maxSelectedItems`
* 프로토콜은 행동을 구성하는 방법에 따라 명명되어야 한다. 프로토콜의 단수 표현식인 클래스는 준수하는 프로토콜과 같은 이름을 가져야 한다. 그러나 우리는 여러 클래스에 대한 정의를 시행하는 프로토콜을 가질 수 있다. 이 경우, 프로토콜이 클래스와 혼동되지 않도록 하기 위해 Gerund 양식(...ing)을 사용하는 것이 일반적인 규칙이다.
	* `Decode`: class name -> `Decoding`: protocol name