# Typescript study

> Week 2

## 2장 타입스크립트의 타입 시스템

- 타입 시스템이 무엇인지
- 어떻게 타입 시스템을 사용해야 하는지
- 무엇을 결정해야 하는지
- 무엇을 가급적 사용하지 말아야 하는지

---

`tsc` 타입스크립트 컴파일러
`tsserver` 단독으로 실행할 수 있는 타입스크립트 서버

### `tsserver`에서 제공하는 언어 서비스

- 코드 자동 완성
- specification
- 검색
- 리팩터링
  -> 서버에서 언어 서비스를 제공하도록 설정하면 코드 작성이 간편해진다.

심벌 위에 마우스 커서 올려서 타입 보기
-> 타입스크립트가 어떻게 타입을 추론하고 있는지 확인 가능

```ts
function logMessage(message: string | null) {
  // message type: string | null
  if (message) {
    message; // message type: string
  }
}
```

**타입 추론 정보가 왜 중요한데?**

함수 호출이 길게 이어질 때, 디버깅 하는데에 필요함

> JS에서 null의 type은 "object"이다.
> 따라서 아래 코드에서는 오류가 발생함

```ts
function getElement(elOrID: string | HTMLelement | null): HTMLElement {
  if (typeof elOrID === "object") {
    // elOrID might be null
    return elOrID;
  }
}
```

- 'Go To Definition'으로 선언된 타입을 알아볼 수 있음

### 타입은 할당 가능한 값들의 집합

> **JS 자료형 8개**

    - 숫자형 / number (Infinity, -Infinity, NaN 포함)
    - BigInt
    - 문자형 / string(글자형(char) 없음)
    - 불린형 / boolean
    - null (typeof null == object, 하지만 진짜 객체는 아니고 언어 자체의 오류)
    - undefined (typeof undefined == undefined)
    - 객체 / object
    - 심볼 / symbol

\*런타임 타입은 BigInt와 symbol을 제외한 6개라고 적혀 있음

> typeof x는 연산자, typeof(x)는 함수형 표현

- 변수에는 다양한 값을 할당할 수 있다.
- 그러나 타입스크립트가 오류를 체크하는 순간에는 어떤 변수에는 할당할 수 있는 값이 있고, 없는 값이 있다.
- 그러므로 **할당 가능한 값의 집합**을 타입의 **범위**라고 하기도 함

`const x: never` x라는 변수가 할당받을 수 있는 값은 없다. (공집합)  
`type A = 'A'` 한 가지 값만 포함하는 타입, unit 타입, literal 타입  
`type AB = 'A' | 'B'` AB는 A 타입일 수도 있고, B 타입일 수도 있음 (합집합)
`const a: AB = 'A'` // 가능  
`const c: AB = 'C'` // 불가능. 'C'라는 유닛 타입은 AB 타입의 부분집합이 아님

⇒ 집합 관점으로 타입 체커는 하나의 집합이 다른 집합의 부분집합인지 검사하는 것이라고 해석 가능

- `interface Identified { id: string, }`이고, 어떤 객체가 `string`으로 할당 가능한 `id` 속성을 가지고 있으면 그 객체는 `Identified`이다. (cf. 구조적 타이핑)

### 집합 관점에서 보는 타입 연산자

1. `&`는 교집합 연산

```ts
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;

// PersonSpan은 never? No.
```

- 타입 연산자는 interface의 속성이 아닌, **값의 집합에 적용됨**
- 그래서 만약 어떤 값이 속성으로 `name`, `birth`, `death`를 모두 가지고 있다면 `PersonSpan` 타입에 속하게 된다.
- 만약 그보다 많은 값을 가져도 `PersonSpan` 타입에 속한다.

2. `|`는 합집합 연산

```ts
type K = keyof (Person | Lifespan);
```

> `keyof`는 각 타입 내의 속성(키)들을 반환하는 연산자이다.

- 여기서 K의 type은 never

### 타입 공간과 값 공간의 심벌 구분

> recap: 컴파일 과정에서 타입 정보는 제거된다.

- www.typescriptlang.org/play/ 에서 TS -> JS 변환된 결과물에서 사라지는 심벌 == 타입
- 클래스가 값으로 쓰이면 생성자가 사용되고, 타입으로 쓰일 때는 속성과 메서드가 사용된다.

- `this`가 값으로 쓰일 떄는 JS의 `this` 키워드이고, 타입으로 쓰일 때는 다형성 `this`라고 불리는 this의 타입스크립트 타입이다.

### 타입 단언과 타입 선언

**웬만하면 타입 단언보다는 타입 선언이 낫다.**

- `as Type`는 타입 단언

  - 강제로 타입을 지정했으니 타입 체커에게 오류를 무시해라
  - 잉여 속성 체크가 동작하지 않음
  - 그러나 DOM 엘리먼트를 다루는 경우, 타입 단언은 거의 필수적이다. 타입스크립트가 DOM에 접근할 수 없기 때문
  - `!`로 `null`이 아니라는 단언
  - 마찬가지로 컴파일 과정에서 제거됨

- `const alice: Person = { ... }` 는 타입 선언

  - [잉여 속성 체크](#잉여-속성-체크) 동작

- 화살표 함수 등을 사용할 때 타입과 함께 변수를 선언하는 것이 직관적

### 객체 래퍼 타입 피하기

**`string`과 `String`은 다르다!**

- 기본형에는 메소드가 없고, 객체 타입에는 메소드가 있다
- JS는 기본형을 객체로 래핑하고, 메소드를 호출하는 형태
- 타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링함
- 타입스크립트에서 코드를 작성할 때 객체 래퍼 타입으로 작성해도 런타임의 값은 기본형

### 잉여 속성 체크

잉여 속성 검사라는 것은 곧 **엄격한 객체 리터럴 체크**라고도 할 수 있다.

```ts
interface Room {
  numDoors: number;
  height: number;
}

const r: Room = {
  numDoors: 1,
  height: 10,
  elephant: "present", // error
};

const obj = {
  numDoors: 2,
  height: 1,
  elephant: "present", // 구조적 타이핑에 의해 허용
};

const o: Room = obj; // no error
```

- `r: Room = {}`은 객체 리터럴로 선언된 객체
- 그렇기 때문에 **잉여 속성 체크**가 이뤄지고, 이는 구조적 타이핑으로 인해 할당 가능한지 체크하는 것과는 별개
  → 구조적 타이핑을 무시하는 예외적인 경우

- `o: Room = obj;`는 `Room` 타입의 필수 속성을 모두 가지고 있음 == `Room` 타입의 부분 집합을 포함함
- 그렇기 때문에 **구조적 타이핑**에 의해 할당이 가능해짐

> _참고: [velog - 잉여 속성 검사(엄격한 객체 리터럴 검사)](https://velog.io/@cks3066/%EC%9E%89%EC%97%AC-%EC%86%8D%EC%84%B1-%EA%B2%80%EC%82%AC%EC%97%84%EA%B2%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%EA%B2%80%EC%82%AC)_

### 함수 표현식에 타입 적용하기

**타입스크립트에서는 함수 표현식을 사용하는 것이 좋다**

- 함수 표현식?
  함수를 생성하고 변수에 값을 할당하는 것처럼 함수를 작성하는 방식  
  _참고: [모던 Javascript 튜토리얼 - 자바스크립트 기본 - 함수 표현식] (https://ko.javascript.info/function-expressions)_

**왜?**

- 함수의 매개변수부터 반환값까지 전체를 함수 *타입*으로 선언하여 재사용할 수 있음
- 반복되는 함수 시그니처를 하나의 함수 타입으로 통합 가능
  > **함수 시그니처?**  
  >  함수의 원형에 명시되는 매개변수 리스트
- 그래서 매개변수나 반환 값에 타입을 명시하는 것보다 함수 표현식 전체에 타입 구문을 적용하는 것이 좋음

# Summary

- 타입을 할당 가능한 값의 집합 관계로 생각하는 것이 좋음
- 타입 공간과 값 공간에서 유사하게 작성된 심볼은 다르게 동작함
- 잉여 속성 체크는 객체 리터럴을 검사하는데, 이것은 구조적 타이핑이 적용되지 않는 예외적인 경우
- 타입 체커는 오류 뿐만 아니라 의도와 다르게 작성된 코드도 검사함
- 타입과 인터페이스 모두 타입을 정의하는 방법으로 유사한 점이 많지만, 보강이 가능한 인터페이스, 유니온이 가능한
