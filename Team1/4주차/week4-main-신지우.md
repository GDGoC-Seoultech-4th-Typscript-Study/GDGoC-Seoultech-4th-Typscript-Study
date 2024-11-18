## 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

<aside>
### 💡 19.1 타입스크립트가 타입을 추론할 수 있다면 타입 구문을 작성하지 않는 게 좋다
</aside>

- 타입스크립트의 많은 타입 구문은 사실 불필요하다.
- 타입 추론이 된다면 명시적 타입 구문은 필요하지 않다.
- 타입을 확신하지 못한다면 편집기를 통해 체크하면 된다.

```jsx
let x: number = 12; // 불필요

let x = 12; // 가능
```

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

를 다음과 같이 작성해도 좋다.

```jsx
const person = {
	~~name: string;
	born: {
		where: string;
		when: string;
	}；
	died: {
		where: string;
		when: string;
	}
} = {~~
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
const axis1: string = ‘x1; // 범용 타입 string
const axis2 = 'y'; // 리터럴 타입 "y"
```

axis2변수는 타입스크립트가 추론한 리터럴 타입 “y”가 더 정확한 타입이다.

리터럴 타입은 더 정확한 타입이므로 코드 안정성과 타입 체크의 엄격함을 높여준다. 따라서 타입이 추론되면 리팩토링이 용이해진다. 

비구조화 할당문 

다른 타입이 추가로 들어있을 가능성이 있는 경우, 비구조화 할당문을 사용해 구현한다.(Ex. id의 타입을 number로 지정했을 때, id에 문자도 들어 있을 수 있음을 나중에 알게 된 경우에는 id 변수 선언에 있는 타입과 맞지 않기 때문에 오류 발생, 명시적 타입 구문이 없었다면 코드는 아무런 수정 없이 타입 체커를 통과했을 것이다)

비구조화 할당문은 모든 지역 변수의 타입이 추론되게 한다. 추가로 명시적 타입 구문을 넣는다면 불필요한 타입 선언으로 코드가 번잡해 진다.                  

명시적 타입 구문   

정보가 부족해서 타입스크립트가 스스로 타입을 판단하기 어려운 상황에 사용한다.    

Ex. logProduct 함수에서 매개변수 타입을 product로 명시한 경우

```jsx
function logProduct(product: Product) {
	const {id, name, price}: {id: string; name: string; price: number} = product;
	console.log(id, name, price);
}
```

<aside>
💡

### **19.2 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없다.**

</aside>

함수 매개변수에 타입 구문을 생략하는 경우도 간혹 있다.

```jsx
function parseNumber(str: string, base=10) {
	// ...
}
```

base의 타입은 number로 추론

타입 정보가 있는 라이브러리에서, 콜백 함수의 매개변수 타입은 자동으로 추론된다.

Ex. express HTTP 서버 라이브러리를 사용하는 request와 response의 타입 선언은 필요하지 않는다.

```jsx
// 이렇게 하지 맙시다
app.get('/health', (request: express.Request, response: express.Response) => {
	response.send('OK');
})；

// 이렇게 합시다.
app.get('/health', (request, response) => {
	response.send('OK');
})；
```

<aside>

### 💡 19.3 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야 한다. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지해 준다.

</aside>

타입이 추론될 수 있음에도 여전히 타입을 명시하고 싶은 상황

1. 객체 리터럴 정의할 때 

```jsx
const elmo: Product = {
	name: 'Tickle Me Elmo',
	id: ‘048188 627152',
	price: 28.99,
}；
```

이런 정의에 타입을 명시하면 잉여 속성 체크가 동작한다. 

> 잉여 속성 체크 : 객체 리터럴이 특정 타입에 할당될 때 그 타입에 정의되지 않은 속성이 포함되었는지 확인하는 메커니즘
> 
- 코드
    
    ```jsx
    interface User {
      name: string;
      age: number;
    }
    
    const validUser: User = {
      name: "Alice",
      age: 25, // 올바른 속성
    };
    
    const invalidUser: User = {
      name: "Bob",
      age: 30,
      hobby: "reading", // 오류: 잉여 속성
    };
    
    ```
    
    `invalidUser`는 `User` 타입에 정의되지 않은 `hobby` 속성을 포함하고 있어 타입스크립트가 잉여 속성으로 간주하고 오류를 발생시킵니다.
    

잉여 속성 체크는 특히 선택적 속성이 있는 타입의 오류(ex. 오타)를 잡는 데 효과적이다. 또한 변수가 사용되는 순간이 아닌 **할당하는 시점(실수가 발생한 부분)에 오류가 표시**되도록 해준다. 

- 타입 구문이 없는 경우, 잉여 속성 체크가 동작하지 않고, 객체를 선언한 곳이 아니라 함수 호출 시점에서야 타입 오류가 발생하므로 디버깅이 어려워질 수 있다.

```jsx
const furby = {
	name: 'Furby',
	id: 630509430963,
	price: 35,
}；

logProduct(furby);
	// ------ ... 형식의 인수는 'Product' 형식의 매개변수에 할당될 수 없습니다.
	// 'id' 속성의 형식이 호환되지 않습니다.
	// 'number' 형식은 'string' 형식에 할당할 수 없습니다.
```

- 타입 구문이 있는 경우, 잉여 속성 체크를 활용하면 오류를 더 빠르게 발견하고 정확한 지점에서 문제를 해결할 수 있다.

```jsx
const furby: Product = {
	name: 1Furby',
	id: 630509430963,
// 'number' 형식은 'string' 형식에 할당할 수 없습니다.
	price: 35,
}；
logProduct(furby);
```

1. 함수 반환할 때  

타입 추론이 가능할 지라도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는 게 좋다.

Ex. 주식 시세를 조회하는 함수

```jsx
function getQuote(ticker: string) {
	return fetch('https://quotes.example.com/?q=${ticker}')
		.then(response => response.jsonO);
}
//캐시추가
const cache: {[ticker: string]: number} = {};
function getQuote(ticker: string) {
	if (tRker in cache) {
		return cache [ticker];
	}
	return fetch('https://quotes.example.com/?q=${ticker}')
		.then(response => response.json())
		.then(quote => {
			cachetticker] = quote;
			return quote;
		})；
}
```

getQuote는 항상 Promise를 반환하므로  if 구문에는 cache [ticker] 아니라 Promise. resolve(cache [ticker])가 반환되도록 해야 한다.

- 타입을 명시하지 않았으므로, 오류는 getQuote 내부가 아닌 getQuote를 호출한 코드에서 발생

```jsx
getQuote('MSFT').then(considerBuying);
	// ~~ 'number | Promise<any>' 형식에 'then' 속성이 없습니다.
	// 'number' 형식에 'then' 속성이 없습니다.
```

- 의도된 반환 타입(Promise<number>)을 명시한다면, 정확한 위치에 오류가 표시

```jsx
const cache: {[ticker: string]: n나mber} = {};
function getQuote(ticker: string): Promise<number> {
	if (ticker in cache) {
		return cachetticker];
			// ~~~~~ 'number' 형식 'Promise<number>' 형식에
			// 할당할 수 없습니다.
	}
	// ...
}
```

반환 타입을 명시해야 하는 이유

1. 반환 타입을 명시하면 함수에 대해 더욱 명확하게 알 수 있음.
    1. 추후에 코드가 조금 변경되어도 그 함수의 시그니처는 쉽게 바뀌지 않음.
    2. 함수 구현 전에 테스트 먼저 작성하는 테스트 주도 개발(TDD)와 비슷.                  
2. 명명된 타입을 사용하기 위해서.
    1. 반환 타입을 명시하면 직관적인 표현이 된다.

린터(linter)를 사용하고 있다면 eslint 규칙 중 no-inferrable-types를 사용해서 작성된 모든 타입 구문이 정말로 필요한지 확인할 수 있다.

- no-inferrable-types
    
    타입스크립트의 “타입 추론”을 활용하여, 명시적으로 선언하지 않아도 되는 **타입 구문을 제거**함으로써 코드를 간결하게 유지한다. 
    
    명확히 추론될 수 있는 타입에 대해 타입 구문이 명시된 경우, 경고 발생.
    
    - ESLint 설정 예시(.eslintrc.js)
    
    ```jsx
    module.exports = {
      rules: {
        '@typescript-eslint/no-inferrable-types': 'warn', // 불필요한 타입 구문 감지
      },
    };
    ```
    
    - 경고 출력 예시
    
    ```jsx
    const age: number = 30; // ESLint: 'no-inferrable-types' 경고 발생
    ```
    
    명시적 타입이 필요한 경우
    
    1. 함수 시그니처
    
    ```jsx
    function getLength(input: string): number {
      return input.length;
    }
    ```
    
    1. 암묵적 any 방지
    
    ```jsx
    let userName: string; // 초기값이 없으므로 타입 명시 필요
    userName = "Alice";
    ```
    
    1. 리터럴 타입 강제
    
    ```jsx
    const direction: "left" | "right" = "left";
    ```
    

## 아이템 20 다른 타입에는 다른 변수 사용하기

<aside>


### 💡 20.1 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않는다.

</aside>

자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 된다. 

```jsx
let id = "12-34-56";
fetchProduct(id); // string으로 사용

id = 123456;
fetchProductBySerialNumber(id); // number로 사용
```

반면, 타입스크립트에서는 두 가지 오류가 발생함.

```jsx
let id = "12-34-56";
fetchProduct(id);

id = 123456;
// '123456 형식은 'string' 형식에 할당할 수 없습니다.
fetchProductBySerialNumber(id);
	// ~~ 'string' 형식의 인수는
	// 'number' 형식의 매개변수에 할당될 수 없습니다.
```

타입스크립트는 "12-34-56"이라는 값을 보고, id의 타입을 string으로 추론했으나, string 타입에는 number 타입을 할당할 수 없기 때문에 오류가 발생한다.

“변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다”

타입을 바꿀 수 있는 한 가지 방법은 범위를 좁히는 것인데(아이템 22) 타입을 더 작게 제한하는 것을 말한다.

오류 해결 : string과 number를 모두 포함할 수 있도록 타입을 확장 string|number로 표현하며, 유니온(union) 타입이라고 한다.

```jsx
let id: string|number = ”12-34-56";
fetchProduct(id);

id = 123456; // 정상
fetchProductBySerialNumber(id); // 정상
```

할당문에서 유니온 타입으로 범위가 좁혀졌음을 알 수 있다.

그러나, id를 사용할 때마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 string이나 number 같은 간단한 타입에 비해 다루기 더 어렵다. 따라서 별도의 변수를 도입하는 것이 낫다.

```jsx
const id = "12-34-56";
fetchProduct(id);

const serial = 123456; // 정상
fetchProductBySerialMimber(serial); // 정상
```

별도의 변수 도입시

- **** 서로 관련이 없는 두 개의 값을 분리한다.(id와 serial).
- **** 변수명을 더 구체적으로 지을 수 있다.
- **** 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
- **** 타입이 좀 더 간결해진다.(string |number 대신 string과 number를 사용).
- let 대신 const로 변수를 선언하게 된다. const로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입을 추론하기에도 좋다.

<aside>

### 💡 20.2 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.

</aside>

타입이 바뀌는 변수는 되도록 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용해야 한다.

‘가려지는(shadowed)’ 변수를 혼동해서는 안 된다.  

> 가려지는 변수(shadwoed variable) : 같은 이름을 가진 변수가 중첩된 스코프에서 선언되며, 바깥쪽 스코프에 있는 변수에 접근하지 못하게 되는 상황. 즉, 안쪽 스코프에서 선언된 변수가 바깥쪽 스코프의 변수를 ‘가리는’ 현상
> 

```jsx
const id = "12-34-56"; // 바깥쪽
fetchProduct(id);
{
	const id = 123456; // 정상
	fetchProductBySerialMimber(id); // 정상 // 바깥쪽 id는 가려짐  
}
```

두 id는 이름이 같지만 실제로는 서로 아무런 관계가 없다. 각기 다른 타입으로 사용되어도 문제없다. 그러나 혼란을 줄 수 있으므로, 목적이 다른 곳에는 별도의 변수명을 사용하는 것이 좋다.  

## 아이템 21 타입 넓히기

<aside>

### 💡 21.1 타입스크립트가 넓히기를 통해 상수의 타입을 추론하는 법을 이해해야 한다.

</aside>

런타임에 모든 변수는 유일한 값을 가진다. 정적 분석 시점에, 변수는 ‘가능한’ 값들의 집합인 타입을 가진다. 상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야 한다. 

- 타입체커는 어떻게 변수의 타입을 결정할까?
    
    런타임
    
    - 코드가 실제 실행되는 시점
    - 런타임에 변수는 항상 하나의 유일한 값을 가지고 있음.
    - ex. 숫자 42, 문자열 “hello” 같은 실제 값.
    
    정적 분석 시점
    
    - 코드 실행 전 타입스크립트가 코드를 확인하는 시점
    - 변수는 하나의 특정 값을 가지지 않으며, 가능한 값들의 집합으로 표현, 이를 타입이라고 한다.
    
    타입스크립트는 변수를 초기화할 때, **초기 값**을 기반으로 변수의 타입을 **자동으로 추론한다.**      
    
    ```jsx
    const x = 42; // 타입 체커는 x의 타입을 "42"(리터럴 타입)로 추론
    let y = "hello"; // 타입 체커는 y의 타입을 string으로 추론
    ```
    
    - `const x = 42`:
        - 상수 `x`는 **변경되지 않으므로** 타입 체커는 `x`의 타입을 리터럴 타입 `"42"`로 추론합니다.
    - `let y = "hello"`:
        - 변수 `y`는 값을 변경할 수 있으므로 더 일반적인 타입인 `string`으로 추론됩니다.

넓히기(widening) : 타입스크립트가 특정 값을 기반으로 **더 넓은 범위의 타입**을 추론하는 과정이다, 값이 상수(`const`)로 선언되지 않았거나, 명시적으로 좁은 타입(리터럴 타입)이 지정되지 않으면, 타입스크립트는 값을 일반적인 타입으로 넓혀 추론한다.

```jsx
let x = "x";

interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vectors, axis: 'x' | 'y' | 'z') {
	return vector[axis];
}

getComponent(vec, x); 
// 오류: 'string' 형식의 인수는 '"x" | "y" | "z"' 형식에 할당될 수 없습니다.
```

1. **타입 넓히기 동작** : getComponent 함수는 두 번째 매개변수에 "x" | "y" | "z" 타입을 기대했지만, x의 타입은 할당 시점에 넓히기가 동작해서 string으로 추론된다. 
    - `x`는 `"x"`라는 리터럴 값을 갖고 있지만, 타입스크립트는 **넓히기**를 통해 `x`의 타입을 `string`으로 추론한다.
    - 따라서, `x`는 `"x"`로 좁혀진 타입이 아닌, 모든 문자열(`string`)을 포함할 수 있는 타입으로 간주된다.
2. **타입 불일치** : string 타입은 "x" | "y" | "z" 타입에 할당이 불가능하다
    - `getComponent` 함수는 두 번째 매개변수로 `"x" | "y" | "z"` 타입만을 허용한다.
    - 하지만, `x`의 타입은 `string`으로 추론되었기 때문에, 더 좁은 범위의 타입(`"x" | "y" | "z"`)과 호환되지 않아 오류가 발생한다.
- 해결하려면?
    
    방법 1:  `const`를 사용하여 타입 좁히기
    
    - `const` 선언은 타입 넓히기를 방지하여, `x`의 타입을 `"x"`로 유지
    
    ```tsx
    const x = "x"; // x의 타입은 "x"
    getComponent(vec, x); // 정상 작동
    ```
    
    방법 2: 명시적 타입 지정
    
    - 변수 `x`의 타입을 명시적으로 `"x" | "y" | "z"`로 선언
    
    ```tsx
    let x: 'x' | 'y' | 'z' = "x"; // 명시적 타입 선언
    getComponent(vec, x); // 정상 작동
    ```
    
    방법 3: Type Assertion (타입 단언)
    
    - `as` 키워드를 사용해 타입을 단언
    - 그러나, 이는 타입 안전성을 약화시킬 수 있으므로 신중히 사용
    
    ```tsx
    const x = "x";
    getComponent(vec, x as 'x'); // 정상 작동
    ```
    

타입 넓히기 예시

```jsx
const mixed = ['x', 1];
```

- ('x' | 1)[]
- ['X', 1]
- [string, number]
- readonly [string, number]
- (string|number)[]
- readonly (string|number)[]
- [any, any]
- any[]

타입스크립트가 작성자의 의도를 추측((string|number) []으로 추측)

타입스크립트는 x의 타입을 string으로 추론할 때, 명확성(const)과 유연성(let) 사이의 균형을 유지하려고 한다. 

초기값으로 혼합된 타입(string | RegExp, string | string[]) 보다는 단일 타입(string)을 사용하는 게 낫다. 복잡하기 때문. 

넓히기

1. const 사용 : let 대신 const로 변수를 선언하면 더 좁은 타입이 된다.

```jsx
const x = 'x'; // 타입이 "x"
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x); // 정상
```

x는 재할당될 수 없으므로 타입스크립트는 의심의 여지 없이 더 좁은 타입("x")으로 추론할 수 있습니다.
그리고 문자 리터럴 타입 "X"는 "x"|"y"|"z"에 할당 가능하므로 코드가 타입 체커를 통과한다.

const 선언은 일반적으로 값의 불변성을 보장하지만, 객체와 배열에서는 내부 요소의 불변성을 보장하지 않기 때문에 타입 추론에 문제가 발생할 수 있다. 

아이템 초반에 있는 mixed 예제(const mixed = ['x', 1];)는 배열에 대한 문제를 보여준다. 

튜플 타입을 추론해야 할지, 요소들은 어떤 타입으로 추론해야 할지 알 수 없다.

```tsx
const mixed = ['x', 1];
// mixed의 타입: (string | number)[]
```

- **리터럴 타입으로 추론되지 않음**
    - 타입스크립트는 `const mixed`를 선언했지만, `['x', 1]`을 "튜플 타입" (`[string, number]`)로 추론하지 않음.
    - 대신, 배열 전체를 **유연하게 다루기 위해** `(string | number)[]` 타입으로 추론.
- **배열의 불변성 문제**
    - `const`로 선언했더라도, 배열의 참조는 불변이지만 내부 요소는 여전히 변경할 수 있음.
    - 따라서, 타입스크립트는 타입 추론에서 더 제한적인 "리터럴 타입" 대신 더 유연한 배열 타입을 사용.
    

비슷한 문제가 객체에서도 발생한다.

자바스크립트에서 정상입니다

```jsx
const v = {
	x: 1,
}；
v.x = 3;
v.x = '3';
v.y = 4;
v.name = 'Pythagoras';
```

v의 타입은 구체적인 정도에 따라 다양한 모습으로 추론된다.

- 구체적, {readonly x: 1}
- 추상적, {x: number}
- 가장 추상적, {[key: string]: number} or object
- 객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 let으로 할당된 것처럼 다룬다.
- 따라서, v의 타입은 {x: number}
- v.x를 다른  숫자로 재할당할 수 있게 되지만 string으로는 안 된다. 그리고 다른 속성을 추가하지도 못한다. 따라서 객체는 한번에 만들어야 한다.

다음 코드는 마지막 세 문장에서 오류가 발생한다. 

```jsx
const v = {
	x: 1,
}；
v.x = 3; // 정상
v.x = '3';
// ~ '"3"'형식은 'number' 형식에 할당할 수 없습니다.
v.y = 4;
// ~ '{ x: number; }' 형식에 'y‘속성이 없습니다.
v.name = 'Pythagoras';
// ’{ x: number; }' 형식에 'name' 속성이 없습니다.
```

<aside>

### 💡 21.2 동작에 영향을 줄 수 있는 방법인 const, 타입 구문, 문맥, as const에 익숙해져야 한다.

</aside>

**타입 추론의 강도를 직접 제어**하기 위한 타입스크립트의 기본 동작을 재정의하는 세 가지 방법

1. 명시적 타입 구문을 제공(함수의 매개변수로 값 전달)   

```jsx
const v: { x: 1|3|5 } = {
	x: 1,
}； 
// v의 타입 { x: 1|3|5; }
// x의 타입 1 | 3 | 5 
```

- 여기서 타입 넓히기 알고리즘을 통해 x의 타입이 number가 되어야 하는 것이 아닌가?
    1. 명시적 타입 선언  
    
    ```tsx
    const v: { x: 1 | 3 | 5 } = { x: 1 };
    ```
    
    객체 v의 타입을 명시적으로 `{ x: 1 | 3 | 5 }`로 선언했기 때문에, 타입 추론 대신 **명시된 타입을 우선적으로 사용**
    
    따라서 속성 x의 타입은 넓히기 없이 `1 | 3 | 5` 로 고정
    
    1. 넓히기 적용되는 경우  
    
    초기화 값만으로 타입을 추론할 때  
    
    ```tsx
    const v = { x: 1 };
    // 타입스크립트는 v의 타입을 { x: number }로 추론
    ```
    
1. 타입 체커에 추가적인 문맥을 제공

```tsx
function processValue(v: { x: 1 | 3 | 5 }) {
  console.log(v.x); // v.x는 1, 3, 5 중 하나
}

processValue({ x: 1 }); // 타입 체커가 { x: 1 | 3 | 5 }로 추론
```

1. const 단언문을 사용

```jsx
const vl = {
	x: 1,
	y： 2,
}; // 타입은 { x: number; y: number; }
const v2 = {
	x: 1 as const,
	y： 2,
}; // 타입은 { x: 1; y: number; }
const v3 = {
	x: 1,
	y： 2,
} as const; // 타입은 { readonly x: 1; readonly y: 2; }
```

- v3
    - 값 뒤에 as const를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론
    - 넓히기 동작 X
    - v3이 진짜 상수라면, 주석에 보이는 추론된 타입이 실제로 원하는 형태
    - 배열을 튜플 타입으로 추론할 때에도 as const를 사용
    

```jsx
const al = [1, 2, 3]; // 타입이 number[]
const a2 = [1, 2, 3] as const; // 타입이 readonly [1, 2, 3]
```

넓히기로 인해 오류가 발생한다고 생각되면, 명시적 타입 구문 또는 const 단언문을 추가하는 것을 고려해야 한다.
단언문으로 인해 추론이 어떻게 변화하는지 편집기에서 주기적으로 타입을 살펴보자.

## 아이템 22 타입 좁히기

타입 넓히기의 반대는 타입 좁히기이다. 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정이다.

<aside>

### 💡 22.1 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야 한다.

</aside>

Ex1. null 체크

```jsx
const el = document. getElementByld ('foo'); // el 타입 : HTMLElement | null
if (el) {
	el // 타입이 HTMLElement
	el.innerHTML = 'Party Time'.blinkO; 
} else {
	el // 타입이 null이면 innerHTML에 접근할 수 없다.    
	alert('No element #foo');
}
```

if 문에서 el이 null이라면, 분기문의 첫 번째 블록이 실행되지 않는다. 첫 번째 블록에서 HTMLElement | null 타입의 null을 제외하므로, 더 좁은 타입이 되어 작업이 훨씬 쉬워진다. 

Ex2. 분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수도 있다.

```jsx
const el = document.getElementByld('foo'); // 타입 : HTMLElement | null
if (!el) throw new Error( 'Unable to find #foo');
el; // 이제 타입은 HTMLElement
el.innerHTML = 'Party Time'.blinkO;
```

`throw new Error( 'Unable to find #foo');` 는 실행 흐름을 완전히 종료시키는 문장이므로, `throw`이후 코드가 실행 안되므로, if 조건문 통과한 경우 반드시 el이 `HTMLElement` 타입이다.               

Ex3. instanceof를 사용

```jsx
function contains(text: string, search: string|RegExp) {
	if (search instanceof RegExp) { // search가 RegExp의 인스턴스인지 확인   
		search // 타입이 RegExp
		return !!search.exec(text);
	}
	search // 타입이 string
	return text.includes(search);
}
```

Ex4. 속성 체크

```jsx
interface A { a: number }
interface B { b: number }
function pickAB(ab: A | B) {
	if ('a' in ab) {
		ab // 타입이 A
	} else {
		ab // 타입이 B
	}
	ab // 타입이 A | B
}
```

Ex5. Array.isArray 같은 일부 내장 함수

```jsx
function contains(text: string, terms: string|string[]) {
	const termList = Array.isArray(terms) ? terms : [terms];
	termList // 타입이 string []
	// …
}
```

`Array.isArray(terms)`는 런타임에서 `terms`가 배열인지 확인한다.

- `true` (배열인 경우) `terms`가 `string[]` 타입임을 보장
- `false` (배열이 아닌 경우) `terms`가 `string` 타입임을 보장

<aside>

### 💡 22.2 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있다.

</aside>

Ex6. 명시적 ‘태그’를 붙이기

```jsx
interface UploadEvent { type: 'upload'; filename: string; contents: string }

interface DownloadEvent { type: 'download'; filename: string; }

type AppEvent = UploadEvent | DownloadEvent; // -> 태그된 유니온 타입  

function handleEvent(e: AppEvent) {
	switch (e.type) { // e.type은 태그 필드로, 런타임에서 e의 타입을 결정하는 기준      
		case 'download':
			e // 타입이 DownloadEvent
			break;
		case 'upload':
			e; // 타입이 UploadEvent
			break;
	}
}
```

이 패턴은 ‘태그된 유니온(tagged union)’ 또는 ‘구별된 유니온(discriminated union)’이라고 불린다.

- 태그된 유니온 패턴
    
     타입스크립트에서 유니온 타입을 안전하게 처리한다.
    
    - 공통된 `type` 필드(태그)를 사용하여 각 타입을 구분한다.
    - 타입스크립트는 조건문이나 `switch` 문을 통해 각 타입을 자동으로 좁힐 수 있다.
    - 이 패턴은 **코드의 안전성**, **가독성**, **유지보수성**을 크게 향상시킨다.

잘못된 경우

Ex1. 유니온 타입에서 null 제외 잘못됨

```jsx
const el = document. getElementByld ('foo'); // 타입이 HTMLElement | null
if (typeof el === 'object') {
	el; // 타입이 HTMLElement | null
}
```

자바스크립트에서 typeof 연산자를 이용해 null을 제외하려고 했으나 typeof null의 결과가 "object"이기 때문에, if 구문에서 null이 제외되지 않음

Ex2. 기본형 값 잘못됨

```jsx
function foo(x?: number|string|null) {
	if (!x) {
		x; // 타입이 string | number | null | undefined
	}
}
```

빈 문자열 ‘ ‘과 0 모두 false가 되기 때문에, 타입은 전혀 좁혀지지 않았고 x는 여전히 블록 내에서 string 또는 number가 된다.

- 자바스크립트 false
    - `false`
    - `0`
    - `''` (빈 문자열)
    - `null`
    - `undefined`
    - `NaN`

사용자 정의 타입 가드    

- 타입스크립트가 타입을 식별하지 못한다면 식별을 돕기 위해 커스텀 함수를 도입할 수 있다.
- 이 기법을 ‘사용자 정의 타입 가드’라고 한다.

Ex1.          

```jsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if (islnputElement(el)) {
		el; // 타입이 HTMLInputElement
		return el.value;
	}
	el; // 타입이 HTMLElement
	return el.textContent;
}
```

- 타입 가드 반환 타입 `el is HTMLInputElement`
    - `'value' in el`: `HTMLInputElement`에는 `value` 속성이 있으므로, 이를 사용하여 타입을 확인한다.
    - `true` 반환 시, 타입스크립트는 `el`의 타입을 `HTMLInputElement`로 좁힌다.
    - `false` 반환 시, ****타입스크립트는 `el`의 타입을 그대로 유지한다(`HTMLElement`).
- `getElementContent` 로 타입 가드 사용
    - `isInputElement(el)`의 결과가 `true`라면, `el`은 `HTMLInputElement` 타입으로 좁혀진다.

Ex2. 타입 가드를 사용하여 **배열과 객체의 타입 좁히기**를 하는 경우

배열에서 어떤 탐색을 수행할 때 undefined가 될 수 있는 타입 사용 가능

```jsx
const jackson5 = ['Jackie', 'Tito', 'Jermaine', ‘Marlon1, ’Michael'];
const members = ['Janet', 'Michael1].map(
	who => jackson5.find(n => n === who)
); // 타입이 (string | undefined)[]
```

`Array.prototype.find`는 조건에 맞는 요소를 반환하거나, 없으면 `undefined`를 반환한다.

따라서, `jackson5.find`의 반환 타입은 `string | undefined` 이고  `map`의 반환 타입은 `(string | undefined)[]` 이다.

```jsx
const members = ['Janet', 'Michael'].map(
	who => jackson5.find(n => n === who)
).filter(who => who !== undefined); // 타입이 (string | undefined) []
```

filter 함수를 사용해 undefined를 제거하려고 했지만, 배열 요소의 타입을 좁히지 못하고 여전히 `(string | undefined)[]`로 유지한다. 이 경우, **사용자 정의 타입 가드 사용.**    

```jsx
function isDefined<T>(x: T | undefined): x is T {
	return x !== undefined;
}

const members = ['Janet', 'Michael'].map(
	who => jackson5.find(n => n === who)
).filter(isDefined); // 타입이 string!]
```

- `x is T`는 타입스크립트에 **타입 가드의 역할을 명시적으로 알리는 반환 타입**이다.
    - 함수의 반환 값이 `true`일 경우, `x`의 타입이 `T` 이다.
    - 함수의 반환 값이 `false`일 경우, `x`가 `undefined`이다.
- 사용자 정의 타입 가드 `isDefined`는 입력 타입 `T | undefined`에서 `undefined`를 제외한 타입 `T`만 남기도록 타입스크립트가 타입을 좁히게 만듭니다.

편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 자연스레 익힐 수 있다.

타입스크립트에서 타입이 어떻게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있다.
