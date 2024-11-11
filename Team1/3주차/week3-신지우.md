## 아이템 13 타입과 인터페이스의 차이점 알기

명명된 타입을 정의하는 방법 두 가지

1. 타입

```jsx
type TState = {
  name: string,
  capital: string,
};
```

1. 인터페이스

```jsx
interface IState {
  name: string;
  capital: string;
}
```

- 공통점
  - 추가 속성을 할당 시 오류가 발생
  - 인덱스 시그니쳐 사용
  - 함수 타입 정의 가능
  - 제너릭이 가능
  - 서로를 확장 가능 ( 인터페이스는 유니온 타입 같은 복잡한 타입을 확장할 수 없음) -> 복잡한 타입 확장은 타입과 &를 사용
  - 클래스 구현시, 둘 다 사용 가능
- 차이점
  - 유니온 타입은 존재, 유니온 인터페이스는 없음
  - 타입은 유니온이 될 수도 있고, 매핑된 타입 or 조건부 타입 같은 기능으로 활용됨
  - 타입은 튜플, 배열 타입도 더 간결하게 표현 가능, 인터페이스는 튜플과 같이 구현할 경우 concat과 같은 메서드 사용 불가능, 튜플은 타입으로 구현하는 것이 좋음
  - 인터페이스는 보강(augment)이 가능
- 선언병합(declaration merging) : 인터페이스는 여러 선언이 있을 경우 자동으로 병합해 하나의 선언으로 취급하는 기능이 있음, 타입에서는 불가능

```jsx
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const Wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000,
};
```

## **아이템 14 타입 연산과 제너릭 사용으로 반복 줄이기**

타입 중복은 코드 중복 만큼 많은 문제를 발생시킴.

DRY(Don’t Repeat Yourself)

타입 중복을 피하는 방법 :

- 타입에 이름을 붙여서 사용 : 시그니쳐를 명명된 타입으로 분리해 반복을 줄일 수 있음
- 인터페이스에서의 중복은 한 인터페이스가 다른 인터페이스를 확장해서 반복을 제거 가능
- 제너릭을 사용해서 타입 재사용

```jsx
type TPair<T> = {
  first: T,
  second: T,
};

interface IPair<T> {
  first: T;
  second: T;
}
```

- 동적으로 타입을 매핑

## 아이템 15 **동적 데이터에 인덱스 시그니처 사용하기**

인덱스 시그니처는 동적 키와 값을 다룰 수 있도록 한다.

자바스크립트에서는 문자열 키를 객체에 매핑할 때, 키의 타입이나 값의 타입을 강제하지 않지만 타입스크립트에서는 이를 유연하게 표현하기 위해 **인덱스 시그니처**를 사용할 수 있다.

```jsx
type Rocket = { [property: string]: string };
const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};
```

- **인덱스 시그니처의 단점**
  - 모든 문자열 키를 허용하기 때문에 잘못된 키도 통과될 수 있다. 예를 들어, `name` 대신 `Name`을 사용해도 오류가 발생하지 않는다.
  - 특정 키가 반드시 있어야 한다는 보장이 없다.
  - 키마다 서로 다른 값의 타입을 가질 수 없다. 예를 들어 `thrust`는 `number`여야 할 수도 있지만, 모든 값이 문자열로 제한된다.
  - 타입스크립트의 자동 완성 기능이 제대로 동작하지 않을 수 있다.
- 대안 : 인터페이스 사용

## **아이템 16 number 인덱스 시그니처보다는 Array, Tuple, ArrayLike를 사용하기**

- **타입스크립트의 배열 타입 선언**

```jsx
interface Array<T> {
  [n: number]: T;
}
```

배열의 키는 런타임에서는 여전히 문자열로 처리되지만, 타입 시스템에서는 이를 `number`로 인식하여 타입 체크 시점에 오류를 잡고 타입 안정성을 보장

- **배열 순회 시 주의할 점**
  - `for-in` 루프는 배열을 순회하는 데 적합하지 않다. 이 루프는 키가 문자열로 처리되어 혼란을 야기할 수 있다. 대신 `for-of` 또는 `Array.prototype.forEach`를 사용하는 것이 좋다.

## **아이템 17. 변경 관련된 오류 방지를 위해 readonly 사용하기**

**함수가 배열을 변경하지 않는다고 가정하는 문제를 해결하기 위해**

- **readonly 접근 제어자 도입**
- readonly의 특성
  - 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
  - length를 읽을 수 있지만, 바꿀 수 없다.
  - 배열을 변경하는 pop을 비롯한 다른 메서드를 호출할 수 없다.
- 매개변수를 readonly로 선언하면 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크하고 함수가 매개변수를 변경하지 않는다고 가정하지만, 명시적인 방법을 사용하는 것이 좋다. 코드에서 배열을 변경하는 부분을 아예 제거하는 식으로 고칠 수 있다.

```jsx
function arraySum(arr: readonly number[]) {
	let sum = 0;
	for (const num of arr) {
		sum += num;
	}
	return sum;
}
```

## **아이템 18. 매핑된 타입을 사용하여 값을 동기화하기**

**최적화 접근법 1: 보수적 접근**

```jsx
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
): boolean {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true;  // onClick은 체크하지 않음
    }
  }
  return false;
}
```

- ‘실패에 닫힌 접근’
- 만약 새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때마다 차트를 다시 그림.
- 이 방식은 차트를 너무 자주 다시 그릴 수 있지만, 정확한 렌더링을 보장

**최적화 접근법 2: 실패에 열린 접근**

```jsx
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps): boolean {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

- 이 방식은 재렌더링을 최소화하지만, 누락된 속성이 있을 경우 차트가 갱신되지 않는 위험이 존재
- 일반적인 경우에 쓰이는 방법은 X

**매핑된 타입을 활용한 최적화**

매핑된 타입을 사용해 새로운 속성이 추가될 때 직접 shouldUpdate를 고치도록 하는 게 낫다.

- 매핑된 타입

```jsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
): boolean {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

- 매핑된 타입의 안정성

```jsx
interface ScatterProps {
  onDoubleClick: () => void;
}

const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  // 오류 발생: onDoubleClick 속성이 정의되지 않았음
};
```

매핑된 타입을 사용하면 속성의 추가 또는 변경 시 타입 체커가 경고를 제공할 수 있다. 예를 들어, 새로운 속성을 추가할 경우 타입 체커가 자동으로 오류를 잡아주기 때문에, 속성 추가나 삭제 시 실수를 방지할 수 있다.
