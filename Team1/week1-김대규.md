# 타입스크립트 스터디

대분류: 스터디
시작 날짜: 2024/10/07

2024.10.07 스터디 1주차 정리

# 1장 타입스크립트 알아보기

예제 코드 레포

[https://github.com/danvk/effective-typescript](https://github.com/danvk/effective-typescript)

추가로 읽을만한 책

이펙티브 자바스크립트

타입스크립트의 실행은 자바스크립트로 이루어짐

컴파일면에서는 타입스크립트로 작성한 파일은 자바스크립트로 컴파일됨

### 1.타입스크립트와 자바스크립트의 관계

> ***타입스크립트 프로그램은 자바스크립트와 타입체커를 통과한 타입스크립트의 교집합이다***
> 

자바스크립트 문법 오류도 타입 체커가 지적해준다.
그러나 문법의 유효성과 동작의 이슈는 독립적인 문제이다.

자바스크립트 확장자 : js (리액트 등은 jsx)

타입스크립트 확장자 : ts, tsx

타입스크립트는 js의 슈퍼셋(상위집합)이므로 main.js를 main.ts로 바꿔도 상관없음

→ js 파일을 ts로 마이그레이션하느데 좋음, 기존 코드를 유지하면서 일부분에만 ts 적용이 가능

일반 js 파일도 ts 컴파일러로 실행하여 타입체커가 문제점을 찾아 줄 수 있음

(~~~string.toUppercase는 존재하지 않는 함수 이름 오류,, string.toUpperCase로 바꾸세요)

문자열임을 알려주지 않아도 초깃값(initial state)로 타입을 추론함

→ 3장에서 자세히 다룸

타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것

```jsx
const states = [
	{name: 'Alabama1, capital: 'Montgomery1},
	{name: 'Alaska', capital: 'Juneau'},
	{name: 'Arizona'； capital: ‘Phoenix’},
// ...
]；
for (const state of states) {
	console.log(state.capitol); //capital로 고쳐야함
}
//js에서는 어떠한 오류도 없이 undefined 값을 콘솔에 띄우지만
//ts에서는 의도하지 않은 코드를 찾아내 오류로 던짐
//-> 즉 타입스크립트가 코드의 의도를 파악해줌, 
//타입을 추가하면 의도를 더 잘 알려주기 때문에 더 많은 오류를 찾아줌
```

```jsx
interface State {
	name: string;
	capital: string;
}
const states: State[] = [
	{name: 'Alabama', capitol:'Montgomery'},
	{name: 'Alaska', capitol: 'Juneau'},
	{name: 'Arizona', capitol: 'Phoenix'},
	//개체 리터럴은 알려진 속성만 지정할 수 있지만
	//'State' 형식에 'capitol1 이（가） 없습니다
	//'capital'을（를） 쓰려고 했습니까?
	// ...
]；
for (const state of states) {
	console.log(state.capital);
}

```

states를 명시적으로 선언해주면 states와 for문 중 어디에서 오류가 난건지 판단해줌

### 2.타입스크립트 설정

tsconfig.json에서 컴파일 옵션 설정

tsc -init 실행시 설정파일(tsconfig) 생성

- noImplicitAny : 암시적 any를 허용하지 않음
- strictNullChecks : true시 null과 undefined가 타입에 허용될 수 없음
    
    ```jsx
    const el = document.getElementByld(1 status');
    el.textcontent = 'Ready'; //~개체가 'null'인 것 같습니다
    if (el) {
    	el.textContent = 'Ready'; // 정상
    }
    el!.textcontent = 'Ready'; // 정상
    //el이 null이 아님을 단언합니다
    ```
    
    true 시 이런식으로 코드를 작성해야함 (으; 귀찮)
    
- noImplicitThis
- strictFunctionTypes

→ strict로 설정하면 모두 체크함

### 3.코드 생성과 타입은 관계없음

크게봤을 때 타입스크립트 컴파일러는 두 가지 역할을 수행함

- 최신 타입스크립트/자바스크립트를 브라우저에서 동작 할 수 있도록 구버전의 자바스크립트로 트랜스파일
- 코드의 타입 오류를 체크한다

이 두가지는 서로 독립적으로 작동함

타입스크립트가 할 수 있는 일과 할 수 없는 일들

- **타입 오류가 있는 코드도 js로 컴파일이 자동으로 됨**
    
    →noEmitOnError 설정시 오류가 있을 때 컴파일하지 않음
    
- **런타임에는 타입 체크가 불가능함**
    
    →속성 체크로 값이 존재하는지 체크 or ‘태그’ 기법 사용
    
    if(’height’ in shape);
    
    값이 존재하는지 체크
    
    type Shape = Square | Rectangle;
    
    Shape 타입은 ‘태그된 유니온’임 이 기법은 런타임에 타입 정보를 유지 할 수 있음
    
    타입은 런타임 접근 불가함, 값은 런타임 접근 가능
    
    인터페이스는 타입으로만 사용 가능하지만 클래스는 타입과 값으로 모두 사용 가능함
    
- **타입 연산은 런타임에 영향을 주지 않음**
    
    return val as number 같이 타입 연산(타입 단언문)은 실제 런타임에서 아무런 영향이 없음
    
    return Number(val)이런식으로 해야함
    
- **런타입 타입은 선언된 타입과 다를 수 있음**
    
    선언된 타입이 달라질 수 있는 경우가 있음
    
    js에서 선언된 타입과 다른 타입으로 함수를 호출하는 경우
    
    ts에서 api로 받아온 값이 선언된 타입과 다른 타입으로 받아올 경우
    
    런타임의 타입과 선언된 타입이 맞지 않을 수 있음
    
- **타입스크립트 타입으로는 함수를 오버로드 할 수 없음**
    
    ```jsx
    function add(a: number, b: number) { return a + b; }
    	// ~ 중복된 함수 구현입니다
    function add(a: string, b: string) { return a + b; }
    	// ~ 중복된 함수 구현입니다
    ```
    
- **타입스크립트 타입은 런타임 성능에 영향을 주지 않음**
    
    타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에 런타임의 성능에 아무런 영향을 주지 않음
    

<aside>
<img src="https://www.notion.so/icons/light-bulb_yellow.svg" alt="https://www.notion.so/icons/light-bulb_yellow.svg" width="40px" />

트랜스파일 : 소스코드를 다른 형태의 소스코드로 변환하는 행위, 결과물이 여전히 컴파일되어야 하는 소스코드이기 때문에 컴파일과는 구분해서 부름

</aside>

### 4.구조적 타이핑

자바스크립트는 덕 타이핑 기반

객체의 구조가 다른 객체의 구조와 호환 된다면 다른 타입이라도 함수 호출에 사용할 수 있음

→ 구조적 타이핑

함수 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질거라 생각하기 쉬움

but 타입스크립트 타입 시스템에서는 이렇게 표현할 수  없음, 타입이 열려 있음

타입이 열려있다는 것은 타입에 선언된 속성 외에 임의의 속성을 추가하더라도 오류가 발생하지 않는 것

```jsx
interface Vector3D {
	x: number;
	y: number;
	z: number;
}
function calculateLengthL1(v: Vector3D){
	let length = 0;
	for (const axis of Object.keys(v)) {
		const coord = v[axis];
		// ----------- ’string'은 ’Vector3D'의 인덱스로 사용할 수 없기에
		//엘리먼트는 암시적으로 'any' 타입입니다
		length += Math.abs(coord);
	}
return length;
}

const vec3D = {x: 3, y: 4, z: 1, address: ’123 Broadway'};
calculateLengthLl(vec3D); // 정상, NaN을 반환합니다
```

Vector3D의 타입은 열려있음,즉 v는 어떤 속성이든 가질 수 있기 때문에  axis의 타입이 string이 될 수 잇음

### 5.any타입은 any됨

**any 타입에는 타입 안정성이 없음**

as any를 사용하면

타입체커는 선언에 따라 타입을 판단하지만 정작 다른 값을 할당 가능

ex) age += 1 //런타임에는 정상, age는 “121”

**any는 함수 시그니쳐를 무시함**

함수를 적성할 때, 호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입의 출력을 반환함.

그러나 any타입을 사용하면 이런 약속을 어길 수 있음

```jsx
function calculateAge(birthDate: Date): number {
	// ...
}
let birthDate: any = '1990—01-19';
calculateAge(birthDate); // 정상
```

birthData 매개변수는 string이 아닌 Date 타입이어야 함
그러나 any 타입을 사용하면 calculateAge의 시그니처를 무시하게 됩니다

**any 타입에는 언어 서비스가 적용되지 않음**

어떤 심벌에 타입이 있다면  타입스크립트 언어서비스는 자동완성 기능과 도움말, 이름 변경 기능을 제공함

그러나 any타입을 사용하면 자동완성으로 속성이 나타나지 않음

타입의 심볼을 변경해도 이름을 변경해주지 않음

**any 타입은 코드 리팩터링 때 버그를 감춤**

**any는 타입 설계를 감춰버림**

any 타입을 사용하면 설계가 잘 되었는지. 설계가 어떻게 되어 있는지 전혀 알 수 없음

설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋음

**any는 타입시스템의 신뢰도를 떨어트림**

---

### 궁금한점

interface와 type은 무슨 차이일까?

https://yceffort.kr/2021/03/typescript-interface-vs-type