# 타입스크립트의 타입 시스템

---

- 타입스크립트 역할
    - 타입스크립트 → 자바스크립트
    - **타입시스템**

## 편집기를 사용하여 타입 시스템 탐색

---

타입스크립트를 설치하면,

- 타입스크립트 컴파일러(tsc)
- 단독 실행이 가능한 타입스크립트 서버(tsserver)

를 실행할 수 있음.

1. 심벌 위에 마우스 커서를 대면 타입스크립트가 그 타입을 어떻게 판단하는 지 확인 가능.
2. 타입스크립트가 객체 내 타입 추론 방식을 확인 가능
3. 메서드 호출에서 추론된 제너릭 타입 확인 가능
4. ‘Go to Definition’ 기능을 통해,  라이브러리가 어떻게 모델링 됐는지 알 수 있음 (*lib.dom.d.ts* file)

## 타입이 값들의 집합이라고 생각하기

---

타입스크립트가 코드 실행 전, 오류를 체크하는 순간에는 ‘타입’을 가짐. → ‘*할당 가능한 값들의 집합*’

타입 범위

- never(공집합)타입 : 아무 값도 포함하지 않는 공집합, 아무런 값도 할당할 수 없음
- literal(리터럴)타입 : unit(유닛)타입이라고 불리며, 한 가지 값만 포함하는 타입
- union(유니온)타입 : 두 개 or 세 개로 묶을 경우 ( ‘ | ’ 를 이용해 여러 타입을 가짐), 값 집합들의 합집합을 말함

### 타입의 집합

& (intersection, 교집합) vs | (union, 합집합)

```jsx
interface Person {
name: string;
}
interface Lifespan {
birth: Date;
death?: Date;
}
type PersonSpan = Person & Lifespan;
type PersonOrLifespan = Person | Lifespan;
```

*PersonSpan*의 경우, 공통으로 가지는 속성이 없기 때문에 공집합으로 생각하기 쉽지만, Person과 Lifespan을 둘 다 가지는 값은 인터섹션 타입에 속하기 때문에  Person과 Lifespan의 속성 모두를 가져야 함.

**PersonOrLifespan**의 경우, Person 또는 Lifespan에서 하나의 속성만 가져도 됨.

*PersonSpan*의 경우, extends의 의미로 생각하면 됨

```jsx
interface Person {
name: string;
}
interface PersonSpan extends Person {
birth: Date;
death?: Date;
}
```

*PersonSpan* 타입의 모든 값은 문자열 ‘name’ 속성을 가져야 하고, 다른 ‘birth’ 속성을 가져야 제대로 된 부분 집합이 된다.

- 이전 챕터1에서 언급된 vector의 경우

```jsx
interface VectorlD { x: number; }
interface Vector2D extends VectorlD { y: number; }
interface Vector3D extends Vector2D { z: number; }
```

→ 이런 식으로 vector 인터페이스를 작성할 수 있는데, ‘**서브타입**’이라는 용어로 부분집합의 관계를 나타낼 수 있다.

*“Vector3D는 Vector2D의 서브타입이고 Vector2D는 Vector1D의 서브타입이다.”*

## 타입 공간과 값 공간의 심벌 구분하기

---

타입스크립트에서 심벌은 이름이 같더라도 속하는 공간에 따라 다르게 사용될 수 있음

```jsx
interface Cylinder {
radius: number;
height: number;
}
const Cylinder = (radius: number, height: number) => ({「adius, height});
```

해당 코드에서 interface로 사용되는 Cylinder는 타입으로 사용, const Cylinder는 값으로 사용됨.

→ 다음과 같은 부분이 오류를 야기함

- **타입**
    - type이나 interface 다음에 나오는 경우
    - 타입 선언(:) 또는 단언문(as) 다음에 나오는 경우
- **값**
    - const나 let 선언에 쓰이는 경우
    - = 다음에 나오는 모든 경우

### class와 enum의 경우 상황에 따라 타입과 값 두 가지 모두 가능한 예약어

클래스(class)가 *타입*으로 사용되는 경우, 형태(속성과 메서드)가 사용됨.

*값*으로 쓰일 때는 생성자가 사용됨.

**typeof** 연산자의 경우, 타입에서 쓰일 경우와 값에서 쓰일 경우에 다른 기능을 하게 됨.

- 타입의 관점 : 값을 읽어서 타입스크립트 타입을 반환 (값의 타입을 가져오는 역할)

```jsx
const user = {
  name: "Alice",
  age: 25
};

type UserType = typeof user;

const anotherUser: UserType = {
  name: "Bob",
  age: 30
};
```

- 값의 관점 : 자바스크립트 런타임의 typeof 연산자가 됨 (타입을 확인하는 용)
    
    → (string, number,boolean, undefined, object,function)의 런타임 타입만이 존재
    
    ```jsx
    let myVar = "Hello, World!";
    console.log(typeof myVar); // "string"
    
    myVar = 42;
    console.log(typeof myVar); // "number"
    ```
    
    속성 접근자 [] 의 경우, obj[’field’]와 obj.field의 값이 동일하더라도 타입이 다를 수 있음
    
    ```jsx
    const first: Person ['first'] = p['first']; // 또는 p.first
    ```
    
    Person[’first’]의 경우, 타입 맥락에서 쓰였기 때문에 **타입**이지만, p[’first’]의 경우 ‘=’ 뒤에 쓰였기 때문에 **값**으로 사용됨.
    
    *타입의 속성을 얻을 경우에는 반드시 obj[’field’]의 방법을 사용해야 됨.*
    
    **구조 분해 (destructuring) 할당** : 객체 내의 각 속성을 로컬 변수로 만들어 줌.
    
    ```jsx
    function email({
    person: Person, subject: string,body: string}){} 
    ```
    
    구조 분해 할당을 사용하면서 각 속성에 개별 타입을 지정할 때는 **객체 전체에 대한 타입을 지정**하거나, **개별 속성에 타입**을 붙이는 올바른 구문을 사용해야함.
    
    ```jsx
    function email({
      person,
      subject,
      body,
    }: { person: string; subject: string; body: string }) {
      // 함수 내용
    }
    ```
    
    or
    
    ```jsx
    interface EmailParams {
      person: Person;
      subject: string;
      body: string;
    }
    
    function email({ person, subject, body }: EmailParams) {
      // 함수 내용
    }
    ```
    
    다음과 같은 두 방식으로 타입을 지정해야 됨.
    

## 타입 단언보다는 타입 선언을 사용하기

---

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법

1. 타입 선언 
    
    ex) const alice **: Person**= { name: 'Alice' } 
    
2. 타입 단언 
    
    ex) const bob = { name: 'Bob' } **as** Person; 
    

타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사, 하지만 타입 단언은 강제로 타입을 지정하기 때문에 타입 체커에게 오류를 무시하라고 함

→ 따라서 타입 단언보다는 타입 선언을 사용하는 게 나음 (안전성 체크 가능)

### 화살표 함수의 타입 선언

```jsx
const people = ['alice'； 'bob', 'jan'].map(name => ({name}));
```

map 메서드 안에 name => ({name} as Person) 다음 타입 단언을 사용하는 코드를 넣으면 해결되는 것 처럼 보이지만 런타임에 문제가 발생, 

```jsx
const people = ['alice’, ' bob', ' jan'] .map(name => ({} as Person)); // 오류 없음
```

{name}이 아닌 비어있는 { }를 사용하였지만 오류를 잡아내지 못함

```jsx
const people = ['alice', ’bob', 'jan’].map(name => {
	const person: Person = {name};
	return person
});

or

const people = ['alice', 'bob', 'jan’].map(
	(name): Person => ({name})
);
```

다음 두 방식을 이용해 화살표 함수의 반환 타입을 선언 가능

<aside>
💡

(name): Person과 (name: Person)의 차이

전자의 경우, name의 반환 타입이 X, 반환 타입이 Person이라고 명시함

후자의 경우, name의 타입이 Person임을 명시하고 반환 타입이 없기 때문에 오류가 발생

</aside>

- *타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문을 사용하면 됨.*

## 객체 래퍼 타입 피하기

> 기본형 값들에 대한 일곱 가지 타입 (string, number, boolean, null, undefined, symbol, bigint)
> 
> - 불변(immutable)의 속성을 가짐
> - 메서드를 가지지 않음.

다음과 같은 특징들로 인해 객체와 구분

*자바스크립트는 기본형과 객체 타입을 서로 자유롭게 변환 가능함*

```jsx
> 'primitive'.charAt(3) // "m"
```

기본형인 string을 String 객체로 래핑(wrap) → 메서드를 호출 → 래핑한 객체를 버림 

*(기본형인 string의 메서드 X, 객체타입 String의 메서드)*

- 타입스크립트에서 기본형과 객체 래퍼 타입의 모델링
    - string과 String
    - number과 Number
    - boolean과 Boolean
    - symbol과 Symbol
    - bigint와 BigInt

- 런타임의 값은 객체가 아니고 기본형임.
- 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트에서 기본형 타입을 사용하는 것이 좋음.

### → **타입스크립트 객체 래퍼 타입은 지양하고, 기본형 타입을 사용하는 게 좋음**

## 잉여 속성 체크의 한계 인지하기

---

> **잉여 속성 체크** : 불필요한 속성이 포함되어 있는지 확인하는 과정

**할당 가능 검사** : 특정 변수에 값이나 다른 변수를 할당할 때, 해당 값이 변수에 할당될 수 있는지 검토하는 과정
> 

→ 할당 가능 검사는 타입 호환성을 중심으로, 잉여 속성 검사는 불필요한 속성 제거하여 객체 리터럴을 엄격하게 관리하는 차이가 있음 (둘은 서로 별도의 과정)

잉여 속성 체크가 되지 않는 경우

1. 객체 리터럴이 아닌 경우 (객체는 체크 가능)
2. 타입 단언문을 사용하는 경우 : 단언문보다 선언문을 사용해야 하는 이유

잉여 속성 체크를 원치 않는 경우에는, 인덱스 시그니처를 이용해 타입스크립트가 추가적인 속성을 예상하도록 함.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타 같은 실수를 잡는 데 효과적인 방법

## 함수 표현식에 타입 적용하기

---

자바스크립트(타입스크립트)에서는 함수 ‘**문장**’과 ‘**표현식**’을 다르게 인식함.

```jsx
function rollDiceKsides: number): number {/*-..*/} // 문장
const rollDice2 = function (sides: number): number {/*•••*/}; // 표현식
```

함수 표현식으로 사용하는 것이 좋음. → 함수 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에서 재사용 가능 → 코드의 반복을 줄임

- 다른 함수의 시그니처를 참조하려면 *typeof fn*을 사용

```jsx
async function checkedFetch(input: Requestinfo, init?: Requestlnit) {
	const response = await fetch(input, init);
	if (!response.ok) {
		// 비동기 함수 내에서는 거절된 프로미스로 변환합니다.
		throw new Error(‘Request failed: ' + response.status);
	}
	return response;
}
```

*checkFetch* 함수를 다음과 같이 작성해도 잘 동작하지만,

```jsx
const checkedFetch: typeof fetch = async (input, init) => {
	const response = await fetch(input, init);
	if (!response.ok) {
		throw new Error(‘Request failed: ’ + response.status);
	}
	return response;
}
```

다음과 같이 함수 표현식으로 바꿔줄 경우, 함수 전체에 타입(typeof fetch)를 적용하여 타입스크립트가 input과 init의 타입을 추론할 수 있도록 해주는 장점이 있음.

→ 매개변수에 타입을 선언하는 것보다 함수 표현식 전체에 타입을 정의하는 게 전체적인 코드도 더욱 간결해지고 안전해짐.
