## 2장 타입스크립트의 타입 시스템

타입 시스템의 기초

### 편집기를 사용하여 타입 시스템 탐색하기

타입스크립트 설치시 두가지 실행가능

- 타입스크립트 컴파일러(tsc)
- 단독으로 실행 가능한 타입스크립트 서버(tsserver)

요약

- 편집기를 이용해서 언어 서비스(코드 자동 완성, 명세, 검사, 검색, 리팩터링)을 사용한다.
- 이 타입이 기대한 것과 다르다면 타입 선언을 직접 명시해야 한다.
- 심벌 위에 마우스 커서를 올리면 타입스크립트가 그 타입을 어떻게 판단하고 있는지 확인 가능.
- 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 그리고 타입스크립트가 어떻게 타입을 추론하는지 알 수 있다
- 타입스크립트가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아보는 방법을 터득해야 합니다.

### 타입이 값들의 집합이라고 생각하기

집타입 스크립트가 오류를 체크 할 때는 ‘타입’을 체크함

타입이란, ‘할당 가능한 값들의 집합’ a.k.a 타입의 범위

아무 값도 포함하지 않는 공집합 : never 타입, 선언된 변수의 범위가 공집합이기 때문에 아무런 값도 할당 할 수 없음

한 가지 값만 포함하는 타입 **:** 유닛(unit) 타입, 리터럴(literal) 타입

유니온(union) 타입 : 집합들의 합집합, 타입1 | 타입2,

인터섹션(교집합) : 인터페이스의 속성이 아니라, 값의 집합(타입의 범위)에 적용됨, 추가적인 속성을 가지는 값도 여전히 그 타입에 속함
각 타입내의 속성을 모두 포함하는것이 일반적

extends : 제너릭에서 한정자로도 쓰임, ‘~의 부분 집합’

‘할당될 수 없습니다’ → 상속의 관점에서 ‘상속할 수 없습니다’로 이해 할 수 있음

```jsx
interface Point {
	x: number;
	y: number;
}
type PointKeys = keyof Point; // 타입은 "x" | "y"
function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
	// ...
}
const pts: Point[] = [{x: 1, y: 1}, {x: 2, y: 0}];
sortBy(pts, 'x'); //정상, 'x' 는 'x'|’y' 를 상속(즉, keyof T)
sortBy(pts, 'y'); //정상, 'y'는 'x'|'y'를 상속
sortBy(pts, Math.random() < 0.5 ? 'x' : 'y1'); // 정상, 'x' | ’y’는 'x' | 'y1 를 상속
sortBy(pts, 'z');
// ~ '"z"' 형식의 인수는 '"x" | "y"' 형식의 매개변수에 할당될 수 없습니다
.
```

```jsx
const list = [1, 2]; //타입은 number []
const tuple: [number, number] = list;
// 'number[]' 타입은 '[number, number]'타입의 0, 1 속성에 없습니다
.
```

리스트는 튜플의 부분 집합이 아니므로 할당될 수 없음, 그러나 반대로는 가능

- 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합(벤 다이어그램)으로 표현됨, 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있음
- A&B는 집합의 범위에 적용,A의 범위와 B의 범위의 인터섹션, 객체 타입에서는 두 객체의 속성을 모두 가짐
- ‘A는 B를 상속’, ‘A는 B에 할당 가능’, ‘A는 B의 서브타입’은 ‘A는 B의 부분 집합’과 같은 의미

### 타입 공간과 값 공간의 심벌 구분하기

```java
interface Cylinder {
	radius: number;
	height: number;
}

const Cylinder = { radius: number, height: number) => ({radius, height});

```

interface Cylinder에서 Cylinder는 타입으로 쓰인다. const Cylinder에서 Cylinder와 이름은 같지만 값으로 쓰인다.

Cylinder는 타입으로 쓰일 수 있고, 값으로 쓰일 수도 있다.

https://www.typescriptlang.org/play

**타입**

- 일반적으로 type이나 interface 뒤에 나오는 심벌은 타입이다.
- 타입 선언(:), 단언문(as) 다음에 나오는 심벌은 타입이다.

**값**

- const나 let 선언에 쓰이는 것은 값이다.
- = 다음에 나오는 모든 것은 값이다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/1d0b7a97-ea6f-46d5-ac9a-a1cd951a497c/a26462b4-d519-43d4-a06f-7ceee8d541ee/image.png)

class와 enum은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어이다.

타입으로 쓰일때는 타입 공간의 typeof는 값을 읽어서 타입을 반환, 값으로 쓰일 때는 런타입의 typeof 연산자가 됨, 대상 심벌의 런타임 타입(string, number, boolean, undefined, object, function을 가져올 수 있음.

```jsx
type Tl = typeof p; //타입은 Person
type T2 = typeof email; // 타입은 (p: Person, subject: string, body: string) => Response
const vl = typeof p; // 값은 "object"
const v2 = typeof email; // 값은 "function"
```

속성 접근자

```jsx
type PersonEl = Person["first" | "last"]; // 타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number]; // 타입은 string | number | Date
```

### 타입 단언보다는 타입 선언을 사용하기

```java
interface Person { name: string; };

const alice: Person = { name: 'Alice' }
const bob = { name: 'Bob' } as Person;

```

첫 번째 alice: Person은 변수에 ‘타입 선언’을 붙여서 그 값이 선언된 타입임을 명시한다.

두 번째 as Person은 ‘타입 단언’을 수행한다. 그러면 타입스크립트가 추론한 타입이 있더라도 Person 타입으로 간주한다.

타입 단언 < 타입 선언

타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다.

그러나 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 한다.

- 접미사 !은 그 값이 null이 아니라는 단언문으로 해석된다.
- 모든 타입은 unknown의 서브타입이기 때문에 unknown이 포함된 단언문은 항상 동작한다.
- 타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문을 사용한다.

### 객체 래퍼 타입 피하기

자바스크립트 객체 이외의 기본형 값들에 대한 일곱 가지 타입(string, number, boolean, null, undefined, symbol, bigint)이 있다.

기본형들은 불변이며 메서드를 가지지 않는다는 점에서 객체와 구분된다.

string 동작 과정

<aside>

‘primitive’.charAt(3)

</aside>

charAt은 string의 메서드가 아니다.

그러나, String ‘객체’타입은 메서드를 가진다.

기본형과 객체 타입은 서로 자유롭게 변환 가능하다.

string 기본형에 charAt 같은 메서드를 사용할 때,
자바스크립트는 기본형을 String 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.

기본형과 객체 래퍼 타입

- string, String
- number, Number
- boolean, Boolean
- symbol, Symbol
- bigint, BigInt

string은 String에 할당할 수 있지만, String은 string에 할당할 수 없다.

객체 래퍼 타입을 지양하고, 기본형 타입을 사용하자.

### 잉여 속성 체크의 한계 인지하기

> 잉여 속성 체크(Excess Property Checks) : 객체 리터럴을 특정 인터페이스나 타입에 할당할 때, 해당 타입에 정의되지 않은 추가적인 속성들이 있는지를 검사하는 기능입니다. 이 체크는 개발자가 오타나 불필요한 속성을 포함하여 실수로 잘못된 데이터를 입력하는 것을 방지합니다.

```jsx
interface User {
  name: string;
  age: number;
}

const user: User = {
  name: "John",
  age: 25,
  email: "john@example.com", // 오류 발생: 'email'은 User 타입에 존재하지 않는 속성입니다.
};
```

위 코드에서 `email` 속성은 `User` 인터페이스에 정의되어 있지 않으므로 TypeScript는 컴파일 오류를 발생시킵니다. 이것이 바로 잉여 속성 체크입니다

- 잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타같은 실수를 잡는 데 효과적이지만, 적용범위가 제한적이고, 오직 객체 리터럴에만 적용된다.
- 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다는 한계가 있다.

### 함수 표현식에 타입 적용하기

자바스크립트, 타입스크립트는 함수 ‘문장(statement)’과 함수 ‘표현식(expression)’을 다르게 인식한다.

```jsx
function rollDiceKsides: number): number {/*-..*/} // 문장
const rollDice2 = function (sides: number): number {/*•••*/}; // 표현식
const rollDice3 = (sides: number): number =>{/*•..*/}; // 표현식

```

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다

함수의 매개변수, 반환값 등을 함수 타입으로 선언하여 함수 표현식에 재사용 가능하기 때문.

```jsx
function add(a: number,
function sub(a: number,
function mul(a: number；
function div(a: number,
b: number) { return a + b; }
b: number) { return a - b; }
b: number) { return a * b; }
b: number) { return a / b; }

```

이 사칙연산 코드를

```jsx
type BinaryFn = (a: number； b: n나mber) => number;
const add: BinaryFn = (a； b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

하나의 함수 타입으로 통합하여 작성할 수 있다.

궁금한점

인터섹션과 유니온의 의미가 헷갈린다
