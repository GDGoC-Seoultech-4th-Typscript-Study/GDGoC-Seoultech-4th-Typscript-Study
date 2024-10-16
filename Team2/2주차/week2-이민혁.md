# 타입스크립트의 타입 시스템

## 타입스크립트 개요
타입스크립트는 자바스크립트의 상위집합으로, 자바스크립트 코드에 정적 타입을 추가하여 더 안전한 코드를 작성할 수 있도록 함.
자바스크립트는 동적 타입 언어이기 때문에 코드 실행 중에 발생하는 오류를 사전에 발견하기 어려운 반면, 
타입스크립트는 정적 타입을 통해 코드를 작성할 때 미리 오류를 찾아내고 수정할 수 있음.

타입스크립트는 자바스크립트 코드를 컴파일하여 자바스크립트로 변환해주는 컴파일러 역할도 함께 수행함.
따라서 타입스크립트로 작성된 코드는 최종적으로 자바스크립트로 변환되며, 이는 대부분의 브라우저와 실행 환경에서 그대로 사용할 수 있음.
또한 타입스크립트의 타입 시스템은 코드의 복잡성을 줄이고, 개발자의 의도를 명확히 하여 유지보수성과 확장성을 크게 향상시킴

## 편집기를 사용한 타입 시스템 탐색
타입스크립트를 설치하면, 개발자들은 다음 두 가지 도구를 사용할 수 있음

- 타입스크립트 컴파일러(tsc): 코드를 자바스크립트로 컴파일해주는 도구
- 타입스크립트 서버(tsserver): 타입스크립트 코드를 편집하는 동안 실시간으로 타입을 확인하고 코드 내 오류를 표시하는 서버
- 타입스크립트의 타입 시스템을 효과적으로 탐색하려면, 다음과 같은 편집기 기능을 활용할 수 있음

심벌 위에 마우스 커서를 올리기: 변수나 함수, 클래스 등의 심벌 위에 마우스를 올려놓으면 타입스크립트가 해당 심벌의 타입을 어떻게 추론하는지 확인할 수 있음

객체 내 타입 추론 방식 확인: 객체의 속성에 대해 타입스크립트가 어떤 타입을 추론했는지 확인할 수 있습니다. 이를 통해 객체의 구조와 속성 간의 상호작용을 쉽게 파악할 수 있음

제네릭 타입 확인: 함수 호출에서 타입스크립트가 추론한 제네릭 타입을 확인할 수 있습니다. 제네릭 타입은 함수나 클래스에서 여러 타입을 지원할 수 있는 유연한 방식

정의로 이동(Go to Definition): 이 기능을 사용하면, 함수나 변수의 정의로 빠르게 이동하여 해당 코드가 어떻게 작성되었는지 확인할 수 있음.
예를 들어, DOM 관련 라이브러리가 어떻게 타입 정의되어 있는지 확인하려면 lib.dom.d.ts 파일을 열어볼 수 있다.

## 타입을 값들의 집합으로 이해하기
타입스크립트는 집합 연산을 사용하여 타입 간의 관계를 정의할 수 있음

- 교집합(&): 두 타입의 공통된 속성만을 가진 값을 나타냄. 두 타입 모두의 속성을 만족해야 하므로, 주로 여러 인터페이스를 결합하여 더 구체적인 타입을 만들 때 사용

- 합집합(|): 두 타입 중 하나만 만족해도 포함되는 타입을 나타냄. 여러 타입을 하나의 유니온 타입으로 묶어서, 해당 타입 중 어느 하나라도 일치하면 허용

<예시코드>
```tsx
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
PersonSpan 타입은 Person과 Lifespan 두 인터페이스 모두의 속성을 가져야 하는 타입. 반면에, PersonOrLifespan 타입은 두 인터페이스 중 하나의 속성만 가지고 있어도 타입이 유효함.

## 서브타입 관계
타입스크립트에서는 서브타입이라는 개념을 통해 타입 간의 상속 관계를 설명할 수 있다. 서브타입은 상위 타입의 모든 속성을 상속받아 사용하되, 추가적인 속성을 가질 수 있는 타입.

<예시코드>
```tsx
interface Vector1D { x: number; }
interface Vector2D extends Vector1D { y: number; }
interface Vector3D extends Vector2D { z: number; }
```
위 예시에서 Vector3D는 Vector2D의 서브타입이며, Vector2D는 Vector1D의 서브타입임. 
이는 타입 간의 부분집합 관계로도 설명할 수 있으며, 서브타입 관계는 타입 재사용성을 높이고 코드를 더 간결하게 만들어줌.

## 타입 공간과 값 공간의 심벌 구분하기
타입스크립트에서 같은 이름의 심벌이라도 그것이 타입 공간에 속하는지, 값 공간에 속하는지에 따라 다르게 해석됨

<예시코드>
```tsx
interface Cylinder {
  radius: number;
  height: number;
}
const Cylinder = (radius: number, height: number) => ({radius, height});
```

위 코드에서 interface Cylinder는 타입으로 사용되고, const Cylinder는 값으로 사용됨. 같은 이름이지만 사용하는 공간에 따라 다르게 작동

- 타입으로 사용되는 경우: type이나 interface 선언에 사용되거나, 타입 단언(as) 구문에서 사용됨.
- 값으로 사용되는 경우: const나 let 선언에 사용되거나, 함수 호출 또는 값 할당에서 사용됨.

### class와 enum의 경우
클래스(class)와 열거형(enum)은 타입과 값 모두로 사용될 수 있는 특별한 예입니다. 클래스는 타입으로 사용될 때는 그 클래스의 형태(속성과 메서드)를 나타내고, 값으로 사용될 때는 생성자 함수로서 사용됩니다.

#### typeof 연산자
typeof 연산자는 타입과 값에서 각각 다른 역할을 함.

- 타입 관점: 값의 타입을 가져와 타입스크립트 타입으로 반환.
- 값 관점: 자바스크립트 런타임에서 값의 타입을 확인.

<예시코드>
```tsx
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
위 예시에서 typeof user는 객체의 타입 정보를 반환하여 anotherUser의 타입을 자동으로 유추하게 함

## 타입 단언보다는 타입 선언을 사용하기
타입스크립트에서 변수에 타입을 지정하는 방법은 두 가지가 있음

1. 타입 선언:
```tsx
const alice: Person = { name: 'Alice' };
```
타입 선언은 할당된 값이 해당 타입을 만족하는지 검사한다

2. 타입 단언:
```tsx
const bob = { name: 'Bob' } as Person;
```
타입 단언은 강제로 타입을 지정하여 타입스크립트의 타입 검사를 우회함

타입 선언은 타입스크립트가 값이 올바른 타입인지 검사하지만,
타입 단언은 타입 검사 없이 강제적으로 타입을 적용하기 때문에 런타임 오류를 유발할 수 있음. 
따라서 타입 단언보다는 타입 선언을 사용하는 것이 더 안전하다






