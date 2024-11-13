# typescript 3주차

---

### item 13 타입과 인터페이스의 차이점 알기

- 타입과 인터페이스 정의 방법

```
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}
```

대부분 타입, 인터페이스 모두 사용 가능하다

### 타입, 인터페이스 공통점

- 인덱스 시그니처

```
type TDict = { [key: string]: string };

interface IDict {
  [key: string]: string;}
```

- 함수 타입

```
type TFn = (x: number) => string;

  interface IFn {
    (x: number): string;
  }

  const toStrT: TFn = x => '' + x;  // 정상
  const toStrI: IFn = x => '' + x;  // 정상
```

- 제너릭
- 확장 가능

인터페이스는 타입을 확장할 수 있으며, 타입은 인터페이스를 확장할 수 있다

- 클래스 구현

### 타입, 인터페이스 차이점

- union은 타입만 가능하다
`type AorB = 'a' | 'b';`

type  키워드는 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용된다. 튜플도 type 키워드로 구현하는 것이 낫다.

- 보강은 인터페이스만 가능하다

```
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
};  // 정상
```

선언 병합: 같은 이름의 인터페이스로 다른 속성 정의 시 속성이 확장되어 합쳐짐.

> **프로젝트에서 어떤 문법을 사용할지 결정할 때 한 가지 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려해야 한다**
> 

---

### item 14 타입 연산과 제너릭 사용으로 반복 줄이기

💡 DRY: 같은 코드를 반복하지 말라는 원칙

타입 중복은 코드의 중복만큼 많은 문제를 발생시킨다

> 반복을 줄이는 방법
> 

1. 타입에 이름 붙이기
2. 타입을 확장해서 반복 제거하기

```tsx
interface Person {
	firstName : string;
	lastName : string;
	}
interface PersonWithBirthDate extends Person {
	birth: Date;
	}
```

1. pick

`type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'> ;`  

pick은 제너럴 타입이며, T,K의 두가지 타입을 받아서 결과를 반환한다. 

매핑된 타입은 배열의 필드를 루프 도는 것과 같은 방식이다. 

특정 타입에서 몇 개의 속성을 선택해 타입을 정의하고 싶을 때 pick 사용

1. partial

```tsx
interface Options {
	width: number;
	height: number;
	color: number;
}
interface OptionsUpdate {
	width: number;
	height: number;
	color: number;
}
```

update 메서드의 매개변수 타입은 생성자와 동일한 매개변수이면서, 타입 대부분이 선택적 필드가 됩니다.

이 때 매핑된 타입과 keyof를 사용하면 불필요한 중복을 피할 수 있다.

`type OptionsUpdate = {[k in keyof Options]? : Options[k]}`

 매핑된 타입은 순회하며 Optinos 내 k 값에 해당하는 속성이 있는지 찾고 이 때 ? 는 각 속성을 선택적으로 만듭니다.

1. typeof

값의 형태에 해당하는 타입을 정하고 싶을 때 사용한다.

선언 순서에 주의해야 한다. 타입 정의를 먼저하고, 그 값이 타입에 할당 가능하다고 선언해야한다.

> 제너릭 타입은 타입을 위한 함수, 타입을 반복하는 대신 제너릭 타입을 사용해 타입 간에 매핑을 하는 것이 좋다. 제너릭 타입을 제한하려면 extends를 사용해야 한다.
> 

---

### item 15 동적 데이터에 인덱스 시그니처 사용하기

- **인덱스 시그니처란 ?**

 `[property: string] : string`이 인덱스 시그니처이다.

의미 : 

1.**키의 이름**: 키의 위치만 표시하는 용도라고 한다. 즉 타입 체커에서는 사용하지 않는다.

2.**키의 타입**: string이나 number, symbol 그리고 Template Literal 타입이어야한다. 하지만 보통 string이라고 한다.

3.**값의 타입**: 어떤 것이든 가능하다.

- 문제점
1. 잘못된 키를 포함해 모든 키를 허용한다.
2. 특정 키가 필요하지 않다
3. 키마다 다른 타입을 가질 수 없다.
4. 자동 완성을 지원해주지 못한다. 

📌 인덱스 시그니처는 부정확해 더 나은 방법을 찾아야 한다.

- Record 사용하기

키 타입에 유연성을 제공해주는 제너릭 타입으로 string의 부분 집합을 사용할 수 있다.

- 매핑된 타입을 사용하기

매핑된 타입은 키마다 별도의 타입을 사용하게 해준다.

```tsx
type Vec3D = {[k in 'x' | 'y' | 'z'] : number}
// type Vec3D = {
// 	x : number;
// 	y : number;
// 	z : number;
// }

type ABG = {[k in 'a' | 'b' | 'c'] : k extends 'b' ? string : number}
// type ABC = {
// 	x : number;
// 	y : string;
// 	z : number;
// }
```

---

### item 16  인덱스 시그니처보다 Array, 튜플, ArrayLike를 사용하기

자바스크립트에서 객체란 키/쌍 모음이다. 

키는 문자열이고, 값은 어떤 것이든 될 수 있다.

자바스크립트 엔진에서 object의 키는 string, symbol 타입만 가능합니다.

하지만, 자바스크립트 엔진에서 자동으로 형 변환이 되기에 number 타입의 키로도 접근이 가능합니다

타입스크립트는 일관성을 위해 number 타입의 키를 허용한다.

📌 인덱스 시그니처에 number를 사용하기 보다 Array나 튜플, ArrayLike 타입을 사용하는 것이 좋습니다. 

---

### item 17 변경 관련된 오류 방지를 위해 readonly 사용하기

**readonly의 특성** 

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다
- legth를 읽을 수 있지만, 바꿀 수 없다 (배열을 변경)
- pop을 비롯한 다른 매서드를 호출할 수 없다

number[]는 readonly number[]보다 기능이 많기 때문에, readonly number[]의 서브타입이 됩니다. 따라서 변경 가능한 배열을 readonly 배열에 할당할 수 있지만, 그 반대는 불가능하다

```tsx
const a: number[] = [1,2,3];
const b: readonly number[] = a;
const c: number[] = b; // 할당 불가능
```

매개변수를 readonly로 선언하면

- 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크합니다
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 됩니다
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있습니다