## 아이템 23 한꺼번에 객체 생성하기

- 타입은 일반적으로 변경되지 않는다는 특성 때문에, 일부 자바스크립트 패턴을 타입스크립트로 모델링하는 게 쉬워짐 → 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입추론에 유리.

```kotlin
interface Point { x: number; y: number; }
const pt: Point = {}; // '{}' 형식에 'Point' 형식의 x, y 속성이 없습니다

pt.x = 3;
pt.y = 4;
```

```kotlin
const pt = {
	x: 3,
	y： 4,
};
```

다음 예제와 같이, 객체 선언 후에, 각 속성을 추가하려고 하면 타입스크립트에서는 에러가 발생하기 때문에, 객체선언 시, 속성을 모두 정의해야 함.

→ 객체를 각각 나눠서 만들어야 한다면, 타입 단언문(as)를 이용해 타입 체커를 통과 가능.

- 작은 객체를 조합해 큰 객체를 만드는 경우에도 여러 단계를 거치는 것보다 큰 객체를 한꺼번에 만드는 게 좋음.

```kotlin
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id); // 'Object.assign'은 객체를 복사하거나 병합하는 데 사용되는 메서드
namedPoint.name;
```

- 객체 전개 연산자(…)를 사용하면 큰 객체를 한꺼번에 만들어 낼 수 있음.

```kotlin
const namedPoint = {...pt； ...id};
namedPoint.name;
```

객체 전개 연산자를 사용하면 타입 걱정 없이 필드 단위로 객체 생성 가능. 

모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는 게 중요.

타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 null 또는 {} 객체 전개를 사용

```kotlin
declare let hasDates: boolean;
const nameTitle = {name: ‘Khirf', title: ’Pharaoh'};
const pharaoh = {
	...nameTitle,
	...(hasDates ? {start: -2589, end: -2566} : {})
}
```

```kotlin
const pharaoh: {
	start: number;
	end: number;
	name: string;
	title: string;
} | {
	name: string;
	title: string;
}
```

→ 해당 타입에서는 start와 end는 같이 정의되기 때문에 start를 읽을 수 없음.


## 아이템 24 일관성 있는 별칭 사용하기

- box와 polygon.bbox를 동시에 사용한 경우 (별칭의 무분별한 사용)

```kotlin
function isPointlnPolygon(polygon: Polygon, pt: Coordinate) {
	polygon.bbox // 타입이 BoundingBox | undefined
	const box = polygon.bbox;
	box // 타입이 BoundingBox | undefined
	if (polygon.bbox) {
		polygon.bbox // 타입이 BoundingBox
		box // 타입이 BoundingBox | undefined
	}
}
```

동작은 하지만 편집기에서 오류로 표시.

→ 속성 체크는 polygon.bbox의 타입을 정제했지만, box는 그렇지 않았기 때문에 오류 발생

따라서, *‘별칭은 일관성 있는 사용이 필요*’ 함.

무분별한 별칭을 남발할 경우, 제어 흐름을 분석하는데 어려움이 생김.

- 속성 체크에 box를 사용한 경우

```kotlin
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
	const box = polygon.bbox;
	if (box) {
		if (pt.x < box.x || pt.x > box.x[1] ||
			pt.y < box.y [0] || pt.y > box.y [1]) { // 정상
		return false;
		}
	}
	// ...
}
```

객체 비구조화를 이용하면 보다 간결한 문법으로 일관된 이름을 사용할 수 있음.

- 객체의 비구조화

```kotlin
interface Polygon {
	exterior: Coordinate [];
	holes: Coordinate[] [];
	bbox?: BoundingBox;
}

const {bbox} = polygon; // polygon의 bbox라는 객체의 속성을 bbox라는 변수에 값을 할당.
```

*객체의 비구조화는 객체의 속성을 분해하여 개별 변수로 추출하는 문법*

객체 비구조화 이용 시, 주의할 점 2가지

1. 위의 예시의 경우, bbox 속성이 아니라, x와 y가 선택적 속성일 경우에 속성 체크가 더 필요. 타입의 경계에 null 값을 추가하는 것이 좋음
2. bbox에는 선택적 속성이 적합했지만 holes는 그렇지 않음. holes가 선택적이라면, 값이 없거나 빈 배열이었을 것.

> 타입스크리트의 제어 흐름 분석은 지역 변수에는 잘 작동하지만, 객체 속성에서는 주의가 필요함.
함수 호출이 객체 속성의 타입 정제를 무효화 할 수 있기 때문에, 속성보다는 지역 변수를 사용하면 타입 정제를 믿을 수 있음.
> 

## 아이템 25 비동기 코드에는 콜백 대신 async함수 사용하기

과거 자바스크립트에서는 비동기 동작을 모델링하기 위해 콜백을 사용 → 실행의 순서는 코드의 순서와 반대라 이해하기 어려움이 있음 → 이를 해결하기 위해, Promise 개념을 도입 → 이를 더 개선하기 위해, async, await 키워드를 도입해 콜백을 해결.

await : 각각의 프로미스가 처리될 때까지 fetchPages 함수의 실행을 멈춤

async 함수 내에서 await 중인 프로미스가 거절되면 예외를 던짐. → try / catch 구문 사용

async / await를 사용하는 이유

- 콜백보다는 프로미스가 코드 작성하기 쉬움
- 콜백보다는 프로미스가 타입 추론이 쉬움

Promise.race : Promise에 타임아웃을 추가하는 방법

```kotlin
async function fetchWithTimeout(url: string, ms: number) {
	return Promise.race([fetch(url), timeout(ms)]);
} // fetch를 하는 시간을 제한시키는 함수
```

프로미스를 직접 생성해야 하는 경우, 특히 setTimeout과 같은 콜백 API를 래핑하는 경우가 있지만, 일반적으로 프로미스를 생성하기 보다는 async/await를 사용해야 함. 

- 일반적으로 더 간결하고 직관적임.
- async 함수는 항상 프로미스를 반환하도록 강제됨.

즉시 사용 가능한 값에도 프로미스를 반환하는 것이 이상할 순 있지만(fetch를 기다리지 않는 메소드), 비동기 함수로 통일하도록 강제하는 데 도움됨. → 함수는 항상 동기 또는 비동기로 실행되어야 하기 때문

- async 함수를 사용하면 항상 비동기 코드를 작성할 수 있음. ( 콜백 or 프로미스를 사용하면 실수로 반동기 코드를 작성할 수 있음)
- async 함수에서 프로미스를 반환하면 또 다른 프로미스로 래핑되지 않음.
    
    → 반환 타입은 Promise<Promise<T>>가 아니라 Promise<T>가 됨.
    

## 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 타입 추론 시, 값만 고려하는 게 아니라 문맥도 고려함. → 타입 추론에 있어 문맥이 어떻게 사용되는지 이해하는 게 중요함.

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python*;
function setLang나age(language: Language) {/*...*/}

setl_anguage('JavaScript1); // 정상

let language = 'JavaScript';
setLanguage(language); // 에러
```

인라인 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 Language 타입이어야 한다는 것을 알고 있음.

해당 타입에 문자열 리터럴 ‘JavaScript’는 할당 가능함.

하지만, 이 값을 변수로 분리할 경우, 타입스크립트는 string으로 추론하기 때문에 Language 타입으로 할당이 불가능하여 오류 발생.

해결책

1. 타입 선언에서 language의 가능한 값을 제한

```jsx
let language: Language = 'JavaScript';
```

만약 language에 할당되는 값에 오타가 있을 경우, 이를 표시해줌.

1. language를 상수로 만들어줌. (const 사용)

```jsx
const language = 'JavaScript';
```

const를 사용하여 타입 체커에게 language는 변경이 불가능하다고 알려줌.

- 튜플 사용 시 주의점

```jsx
function panTo(where: [numberf number]) {/*••.*/}

panTo([10, 20]); // 정상

const loc = [10, 20];
panTo(loc); // number[] 형식의 인수는 [number,number] 형식의 매개변수에 할당 불가능
```

→ ‘상수 문맥’을 제공하여 오류를 해결 가능. (as const 사용)

const는 단지 값이 가리키는 참조가 변하지 않는 상수인 반면, as const는 해당 값의 내부까지 상수라는 것을 알려줌

하지만, 이 경우에 loc 의 타입이 readonly로 제한됨. panTo의 타입 시그니처는 where의 내용이 불변이라고 보장하지 않기 때문에 동작하지 않는다. 따라서 최선의 해결책은 panTo 함수에 readonly 구문을 추가하는 것임

‘as const’가 문맥 손실과 관련한 문제를 해결할 수 있지만 타입 정의에 실수가 있다면, 오류가 타입 정의가 아닌 호출되는 곳에서 발생함.

- 객체 사용 시 주의점

문맥에서 값을 분리하는 문제는 객체에서 상수를 뽑아낼 때도 발생함.

```jsx
type Language = 'JavaScript' | 'TypeScript' |
’Python';
interface GovernedLanguage {
	language: Language;
	organization: string;
}

function complain(language: GovernedLanguage) {/*...*/}
complain({ language: 'TypeScript', organization: 'Microsoft' }); // 정상

const ts = {
	language: 'TypeScript',
	organization: 'Microsoft',
}；

complain(ts); // language 속성의 형식이 호환되지 않음
```

해당 경우 역시 타입 선언을 추가하거나 (const ts : GovernedLanguage = …) 상수 단언 (as const)를 사용해 해결.

- 콜백 사용 시 주의점

콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용하는데, 콜백을 상수로 뽑아내면 문맥이 소실되고 오류 발생.

→ 매개변수에 타입 구문을 추가해서 해결 가능. or 전체 함수 표현식에 타입 선언을 적용하여 해결

## 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

라이브러리들의 일부 기능(map, filter 등)은 loop를 대체할 수 있기 때문에 자바스크립트에서 유용하게 사용되며, 타입스크립트와 조합하여 사용하면 더욱 좋음. → 타입 정보를 유지하면서 타입 흐름이 계속 전달되도록 함.

(루프를 직접 구현하면 타입 체크에 대한 관리도 직접 해야 함)

로대시 라이브러리 : 자바스크립트에서 데이터 조작과 유틸리티 기능을 제공하는 **라이브러리 →** 배열, 객체, 문자열, 숫자 등 다양한 데이터 타입을 다루는 데 유용한 함수를 제공함 

ex) 

```jsx
_.chunk(['a', 'b', 'c', 'd'], 2);
// [['a', 'b'], ['c', 'd']] , 배열을 지정된 크기로 나눈 배열의 배열을 반환

_.compact([0, 1, false, 2, '', 3]);
// [1, 2, 3] , 배열에서 false, null 값은 falsy 값을 제거해주는 함수
```

로대시 라이브러리를 사용하면 타입 정보가 잘 유지됨. → 매개변수 값을 건드리지않고 매번 새로운 값을 반환하여 새로운 타입으로 안전하게 반환이 가능함.


***

## 경인 : 궁금한 점

Q. 아이템23 파트에서, 타입으로 start와 end가 들어있는데 start or end를 각각 따로 읽을 수 없는 이유?
A. 유니온 타입에서 특정 필드(start)를 접근하려면 TypeScript가 해당 필드가 모든 구성 요소에 존재한다고 확신해야 합니다. 하지만 위의 정의에서는 start와 end가 첫 번째 타입에만 존재하고, 두 번째 타입에는 존재하지 않음

따라서, TypeScript는 안전성을 보장하기 위해 pharaoh가 첫 번째 타입이라는 것을 확정하지 않는 한 start 필드에 접근을 허용하지 않음

---

## 하늘 : 더 알아볼 점 - 비동기 코드와 async/await의 조합

### Promise.all과 async/await의 조합

`async/await`는 비동기 코드를 더 직관적이고 동기적인 코드처럼 작성할 수 있게 해주는 구문이고, `Promise.all`은 여러 개의 비동기 작업을 병렬로 처리하는 데 유용한 메서드이다. 이 두 가지를 결합하면, 여러 비동기 작업을 병렬로 실행하면서도 코드를 깔끔하게 유지할 수 있다.

- 예시: `Promise.all`과 `async/await`의 조합
    
    ```tsx
    async function fetchData() {
        const url1 = "https://api.example.com/data1";
        const url2 = "https://api.example.com/data2";
    
        try {
            // 여러 비동기 작업을 병렬로 실행
            const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
            
            // 데이터 처리
            const jsonData1 = await data1.json();
            const jsonData2 = await data2.json();
    
            console.log(jsonData1, jsonData2);
        } catch (error) {
            console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
        }
    }
    
    fetchData();
    ```
    

### 병렬 실행과 직렬 실행의 차이

- **병렬 실행**
    - `Promise.all`을 사용하면 여러 비동기 작업을 동시에 시작하여, 모든 작업이 완료될 때까지 기다린다. 각 작업은 서로 독립적으로 실행되므로, 작업이 완료되는 순서에 관계없이 기다리지 않고 동시에 처리된다.
        
        ```tsx
        async function fetchData() {
            const url1 = "https://api.example.com/data1";
            const url2 = "https://api.example.com/data2";
        
            try {
                // 여러 비동기 작업을 병렬로 실행
                const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
                
                // 데이터 처리
                const jsonData1 = await data1.json();
                const jsonData2 = await data2.json();
        
                console.log(jsonData1, jsonData2);
            } catch (error) {
                console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
            }
        }
        
        fetchData();
        ```
        
- **직렬 실행**
    - `await`를 사용하면 각 비동기 작업이 순차적으로 실행된다. 이전 작업이 완료되어야만 다음 작업이 시작되므로, 병렬 실행보다 시간이 더 걸릴 수 있다.
        
        ```tsx
        async function fetchData() {
            const url1 = "https://api.example.com/data1";
            const url2 = "https://api.example.com/data2";
        
            try {
                // 여러 비동기 작업을 병렬로 실행
                const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
                
                // 데이터 처리
                const jsonData1 = await data1.json();
                const jsonData2 = await data2.json();
        
                console.log(jsonData1, jsonData2);
            } catch (error) {
                console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
            }
        }
        
        fetchData();
        ```
---
## 민혁 : 중요한 점

타입스크립트에서 객체는 한 번에 생성해야 타입 추론이 정확하게 이루어지고 불필요한 오류를 방지할 수 있다는 점입니다. 속성을 나누어 추가하면 타입스크립트가 객체의 구조를 제대로 이해하지 못해 오류가 발생할 수 있으므로, 모든 속성을 한 번에 정의하는 것이 안전하고 가독성이 높은 코드를 작성하는 방법이라고 생각합니다.

---

## 활동 사진
![image](https://github.com/user-attachments/assets/515dd048-7435-4b0b-8b8b-5a438f59b98e)

