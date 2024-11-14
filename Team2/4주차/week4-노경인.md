## Ch3.타입 추론

### 아이템19 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트가 타입을 위한 언어이기 때문에, 변수를 선언할 때마다 타입을 명시할 필요는 없음

→ 모든 변수에 타입을 선언하는 것은 비생산적

**타입 추론이 가능한 경우에는 타입 구문이 불필요**

- 타입스크립트는 복잡한 객체도 추론이 가능

```jsx
const person: {
	name: string;
	born: {
	where: string;
	when: string;
}； 
died: {
	where: string;
	when: string;
}
} = {
name: 'Sojourner Truth',
born: {
	where: 'Swartekill, NY’,
	when: 'c.1797',
},
died: {
	where: 'Battle Creek, MI',
	when: 'Nov. 26, 1883'
	}
}；
```

```jsx
const person = {
	name: 'Sojourner Truth',
	born: {
	where: 'Swartekill, NY',
	when: ’c.1797’,
	},
	died: {
		where: 'Battle Creek, MI’,
		when: 'Nov. 26, 1883'
	}
};
```

person의 타입은 동일하기 때문에 값에 추가로 타입을 작성하는 것은 불필요함.

+) 배열의 경우도 객체와 마찬가지로 타입스크립트가 입력을 받아 연산하는 함수가 어떤 타입을 반환하는 지 알고 있기 때문에 불필요한 타입 선언은 지양하는 게 좋음

- 비구조화 할당문:  배열이나 객체의 값을 쉽게 분해하여 변수에 할당할 수 있게 해주는 JavaScript의 문법
    - 배열의 비구조화 할당 → 배열의 순서를 기반으로 변수 할당
    
    ```jsx
    const fruits = ["apple", "banana", "cherry"];
    const [first, second, third] = fruits;
    ```
    
    - 객체의 비구조화 할당 → key를 기반으로 변수를 할당
    
    ```jsx
    const person = { name: "Alice", age: 25, city: "Seoul" };
    const { name, age, city } = person;
    ```
    

→ 타입스크립트는 최종 사용처까지 고려하지 않기 때문에, 최종 사용처에서 타입 에러가 발생하지 않게 하기 위해 타입을 선언 해주는 것

타입스크립트 코드는 함수/메서드 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역변수에는 타입 구문을 넣지 않음 → *“타입스크립트가 지역변수에 대해서는 타입 추론이 가능하기 때문에”*

타입 정보가 있는 라이브러리를 사용할 경우에는 콜백 함수의 매개변수 타입은 자동으로 추론되기 때문에, 타입 선언이 필요하지 않음

```jsx
app.get('/health1, (request: express.Request, response: express.Response) => {
response.send('OK1);
})；// 잘못된 타입 선언

app.get('/health', (request, response) => {
response.send('OK');
})；
```

- **객체 리터럴 정의 시, 타입 선언**

객체 리터럴 정의에서 타입을 명시하면, 잉여 속성 체크가 동작

하지만, 타입을 선언하지 않을 경우, 잉여 속성 체크가 동작하지 않아 타입 오류가 발생함

```jsx
const elmo: Product = {
	name: 'Tickle Me Elmo',
	id: ‘048188 627152',
	price: 28.99,
}
const furby = {
	name: 'Furby',
	id: 630509430963, // id는 string 타입
	price: 35,
}

logProduct(furby)
```

*logProduct* 메서드는 매개변수가 Product타입이어야 함. 하지만 *'furby*'의 경우 *id*에 할당된 값이 *number*이기 때문에  *furby*는 *Product* 타입이 아니게 되어 타입 오류 발생.

→ 이런 오류를 잡기 위해 타입을 ‘명시적’으로 선언하는 게 좋음

함수의 반환에도 타입을 명시하여 오류를 방지할 수 있음

```jsx
const cache: {[ticker: string]: number} = {};
function getQuote(ticker: string) {
	if (ticker in cache) {
		return cache[ticker];
	}
	return fetch('https://quotes.example.com/?q=${ticker}')
	.then(response => response.json())
	.then(quote => {
		cache[ticker] = quote;
		return quote;
	})；
}
```

해당 코드에서는 *getQuote* 함수가 *Promise<number*>를 반환 해야 한다. 하지만,  if 조건문에서 반환하는 값의 타입이 *cache[ticker]*의 타입인 *numbe*r를 반환하기 때문에 오류가 발생. 

- *getQuote* 메서드에 타입을 선언하지 않은 경우

→ 실행 시, 오류가 발생하지만 정확히 어느 부분에서 발생한 오류인 지, 알 수 없음

- *getQuote* 메서드에 타입을 선언한 경우

→ 실행 시, *getQuote* 내부가 아닌 *getQuote*를 호출한 코드에서 오류가 발생했음을 정확히 알 수 있음

**반환 타입을 명시해야 하는 이유 2가지**

1. 반환 타입을 명시하면 함수에 대해 명확히 알 수 있음
2. 명명된 타입을 사용하기 위함

### 아이템 20 다른 타입에는 다른 변수 사용하기

타입스크립트에서 변수에 할당되는 값은 바뀔 수 있지만, 변수의 타입은 보통 바뀌지 않음

```jsx
let id = "12-34-56";
fetchProduct(id);
id = 123456; // '123456* 형식은 'string' 형식에 할당x
fetchProductBySerialNumber(id); // string 형식의 인수는 'number' 형식의 매개변수에 할당x
```

타입을 바꿀 수 있는 방법이 있지만, 타입을 확장하는 게 아니라, 타입을 더 작게 제한하는 것

앞선 예제에서의 오류를 해결하기 위해서는 id를 ‘**유니온 타입으로 선언’**하는 게 하나의 방법

→ id를 *string*과 *number*를 둘 다 가질 수 있다고 선언하면 코드가 정상적으로 동작하지만, id를 사용할 때마다 값이 어떤 타입인지 확인 해야하기 때문에 유니온 타입은 *string*이나 *number* 같은 타입보다 다루기 어려움.

예제에서 다뤄지는 id는 서로 관련이 없고 단지 변수를 재사용했을 뿐임. → 변수의 재사용이 많아지면 좋지 않음 → 별도의 변수를 사용하는 게 좋음

- 관련이 없는 값 분리 가능
- 변수명 구체화 가능
- 타입 추론에 용이
- 타입이 간결해짐
- const로 변수 선언이 가능 → const로 변수를 선언하며느 타입 체커가 타입을 추론하기 좋음

### 아이템 21 타입 넓히기

타입 넓히기: 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추하는 것

```jsx
interface Vector3 { x: number; y: number; z: number; }

function getComponent(vector: Vectors, axis: 'x' | 'y' | 'z') {
	return vector[axis];
}

let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
```

→ 런타임에서는 오류가 없지만, 편집기 상에서는 오류가 표시됨.

: 변수 x는 값이 할당되는 시점에 ‘넓히기’가 동작하여 타입이 ‘string’으로 추론됨. 따라서 x는 ‘string’ 타입의 값만을 받을 수 있지만, 변수 vec안에서 x는 number 타입의 값을 할당하고 있기 때문에 오류가 발생함.

타입 넓히기 시점에서 주어진 값으로 추론 가능한 타입이 여러 가지이기 때문에 모호한 점이 있음.

```jsx
const mixed = ['x',1];
```

다음과 같은 경우에서 추론될 수 있는 타입은 여러가지가 존재함. 타입스크립트는 작성자의 의도를 추측하여 타입을 결정하지만 추측한 타입이 항상 작성자의 의도와 같지 않기 때문에 넓히기의 과정을 제어할 방법이 필요함.

1. const 사용

: 더 좁은 타입이 됨. → 예제의 경우 변수 x의 타입이 string에서 ‘x’로 좁아짐

```jsx
const x = 'x'; // 타입이 "x"
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x); 
```

하지만, 여전히 문제가 존재 → mixed 예제의 경우를 const를 이용해서 해결하긴 힘듦 + 객체에서 타입 선언의 경우

1. 추론의 강도를 제어
- 타입스크립트의 기본 동작을 재정의 해야함
    - 명시적 타입 구문 제공
    - 타입 체커에 추가적인 문맥을 제공
    - const 단언문 사용 → 타입 공간의 기법, 최대한 좁은 타입으로 추론 (배열을 튜플 타입으로 추론할 때에도 사용가능)

### 아이템 22 타입 좁히기

: 넓은 타입으로부터 좁은 타입으로 진행하는 과정(↔ 타입 넓히기) (ex, null 체크)

```jsx
const el = document. getElementByld (' foo'); // 타입이 HTMLElement | null
if (el) {
	el // Eb입이 HTMLElement
	el.innerHTWL = 'Party Time' -blinkO;
} else {
	el // 타입이 null
	alert('No element #foo');
}
```

el이 null 값일 경우 첫 번째 블록이 실행되지 않음 → null 타입 제거, 더 좁은 타입이 됨

타입 체커는 조건문에서 타입 좁히기를 잘하지만, 타입 별칭이 존재한다면 그렇지 못할 수 있음

- 분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수 있음
- instanceof를 사용하여 타입을 좁힐 수 있음
- 속성 체크로도 타입 좁히기 가능
- Array.isArray 같은 내장 함수로도 타입 좁히기 가능
- 명시적 ‘태그’를 붙여 타입 좁히기 가능 → 태그된 유니온 or 구별된 유니온 이라고 불림

타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위한 커스텀 함수를 도입 → ‘사용자 정의 타입 가드’

```jsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if (islnputElement(el)) {
		el; // 타입이 HTMLInputElement
		return el.value;
	}
	el; // 타입이 HlMLElement
	return el.textContent;
}
```

배열 탐색의 경우에서 undefined를 걸러내기 위해, filter 메소드를 사용하지만 잘 동작하지 않을 수 있음. → ‘타입 가드’를 사용하여 타입을 좁힐 수 있음.

```jsx
function isDefined<T>(x: T | undefined): x is T {
	return x !== undefined;
}

const members = ['Janet', 'Michael'].map(
	who => jackson5.find(n => n === who)
).filter(isDefined); // 타입이 string!]
```
