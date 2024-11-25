# 타입스크립트 스터디 5주차

# 한꺼번에 객체 생성하기

**변수의 값은 변경될 수 있지만, 타입은 일반적으로 변경되지 않는다.**

- 자바스크립트 패턴을 타입스크립트로 모델링 하는게 쉬워진다.
- 객체를 생성할 때는 여러 속성을 한꺼번에 생성하는 것이 타입 추론에 유리하다.

## **객체를 생성하는 방법**

```jsx
const pt = {};
pt.x = 3; // ~ '{}' 형식에 'x'속성이 없습니다.
pt.y = 4; // ~ '{}' 형식에 'y'속성이 없습니다.
```

- **javascript**
    - 문제 없는 구문
- **typescript**
    - 선언된 `pt` 타입: {}을 기준으로 추론
    → 존재하지 않는 속성이므로 추가할 수 없다.
        - `Point` 인터페이스를 추가 시 오류의 위치가 변경된다.
            
            ```jsx
            interface Point { x: number; y: number;}
            const pt: Point = {}; // ~ '{}' 형식에 'Point' 형식의 x, y 속성이 없습니다.
            pt.x = 3;
            pt.y = 4;
            ```
            

### **해결 방법**

```jsx
const pt = {
	x: 3,
	y: 4,
}; // 정상
```

**객체를 한 번에 정의하여 해결할 수 있다.**

```jsx
const pt = {} as Point;
pt.x = 3;
pt.y = 4; // 정상
```

객체를 반드시 나눠서 추가해야 하는 경우, 타입 단언문(as)을 사용해서 타입 체커를 통과할 수 있다.

```jsx
const pt: Point = {
	x: 3,
	y: 4,
};
```

그러나 선언할 때 객체를 한꺼번에 만드는 것이 가장 낫다.

## 작은 객체를 조합해 큰 객체를 만드는 방법

### **Bad Case: 복잡한 단계를 거치는 방법**

```jsx
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
	// ~~~ '{}' 형식에 'name' 속성이 없습니다.
```

### **Good Case: 객체 전개 연산자(…)를 사용하는 방법**

```jsx
const namedPoint = {...pt, ...id};
namedPoint.name; // 정상, 타입이 string
```

**객체 전개 연산자의 장점**

- 큰 객체를 한꺼번에 생성
- 타입 걱정 없이 필드 단위로 객체 생성

```jsx
const pt0 = {};
const pt1 = {...pt0, x: 3};
const pt: Point = {...pt1, y: 4}; // 정상
```

→ 객체에 속성을 추가하고, 타입스크립트가 새로운 타입을 추론할 수 있게 한다.

## 타입에 안전한 방식으로 조건부 속성 추가하기

### 속성을 추가하지 않는 null 또는 {}로 객체 전개를 사용

```jsx
declare let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};
```

<img width="195" alt="image" src="https://github.com/user-attachments/assets/093f3c5e-1950-4b0e-ade3-23866fc85c96">


→ middle이 선택적 속성으로 추가됨

### 한꺼번에 여러 속성을 추가

```jsx
declare let hasDates: boolean;
const nameTitle = {name: 'Khufu', title: 'Pharaoh'};
const pharaoh = {
	...nameTitle,
	...(hasDates ? {start: -2589, end: -2566} : {})
};
```

```jsx
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

→ start와 end에 의해 객체의 구조가 변경되어 유니온 타입으로 지정됨

<img width="187" alt="image 1" src="https://github.com/user-attachments/assets/2929488d-5772-4589-ba15-6edd5f561020">

```jsx
pharaoh.start
// ~~~~ '{ name: string; title: string; }' 형식에
// 'start' 속성이 없습니다
```

**유니온을 사용하는 게 가능한 값의 집합을 더 정확히 표현할 있지만, 선택적 필드가 다루기에는 더 쉬울 수 있다.**

⇒ 선택적 필드 방식으로 표현하려면 헬퍼 함수를 사용할 수 있다.

```jsx
function addOptional<T extends object, U extends object>(
	a: T, b: U | null
): T & Partial<U> {
	return {...a, ...b};
}
const pharaoh = addOptionaK
	nameTitle,
	hasDates ? {start: -2589, end: -2566} : null
)；
pharaoh.start // 정상,타입이 number | undefined
```

**객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶은 경우**

- 루프 대신 내장된 함수형 기법 또는 Lodash 같은 유틸 라이브러리를 사용하자
    - 한꺼번에 객체 생성하기 관점에서 더욱 옳은 방식이다.

[https://velog.io/@kysung95/짤막글-lodash-알고-쓰자](https://velog.io/@kysung95/%EC%A7%A4%EB%A7%89%EA%B8%80-lodash-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90)

# 일관성 있는 별칭 사용하기

## 어떤 값에 새 이름을 할당하는 예제

```jsx
const borough = {name: 'Brooklyn', location: [40.688, -79.233]};
const loc = borough.location;
```

### 별칭의 값 변경 시 원래 속성값도 변경됨

```jsx
loc[0] = 0;
borough.location // [0, -79.233]
```

### 별칭을 남발해서 사용할 경우 제어 흐름을 분석하기 어렵다.

**다각형을 표현하는 자료구조**

```jsx
interface Coordinate {
	x: number;
	y: number;
}

interface BoundingBox {
	x: [number, number];
	y: [number, number];
}

interface Polygon {
	exterior: Coordinate[];
	holes: Coordinate[][] 
	bbox?: BoundingBox;
}
```

![image 2](https://github.com/user-attachments/assets/4dfd70bb-3a3b-4360-ac5e-7590fa67bb4e)

→ `bbox` : 최적화 속성(옵션)

```jsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
	if (polygon.bbox) {
	if (pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] ||
		pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1]) {
	return false;
	}
}

// ...
}
```

⇒ 타입 체커를 통과하고 잘 동작하지만, 반복되는 부분이 존재한다.

- ploygon.bbox
    - 중복을 줄이기 위해 임시 변수를 뽑아내자.

```jsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
	const box = polygon.bbox;	
	if (polygon.bbox) {
		if (pt.x < box.x[0] || pt.x > box.x[1] ||
		//         ~~~~~~~~           ~~~~~~~ 객체가 'undefined'일 수 있습니다.
			pt.y < box.y[0] || pt.y > box.y[1]) {
		//       ~~~~~~~~           ~~~~~~~ 객체가 'undefined'일 수 있습니다.
		return false;
	}
}

// ...
}
// strictNullChecks를 활성화했다고 가정
```

⇒ `polygon.bbox`를 별도의 `box` 라는 별칭으로 분리하여, 제어 흐름 분석을 방해하였다.

- if (polygon.bbox) ⇒ if(box) (속성 체크를 별칭으로)
    - **원칙: 별칭을 일관성 있게 사용한다.**

### 객체 비구조화

보다 간결한 문법으로 일관된 이름을 사용하는 방법

```jsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const {bbox} = polygon;
  if (bbox) {
    const { x, y } = bbox;
	  if (pt.x < x[0] || pt.x < x[1] ||
				pt.y < y[0] || pt.y < y[1]) {
					return false;
    }
  }
	// ...
}
```

- x, y가 각각 선택적 속성일 경우 추가적인 속성 체크가 필요하다.
- `bbox`는 선택적 속성(?)이 적합하나 `holes`는 부적합하다.
    - `holse` 를 선택적 속성으로 나타내는 경우 구멍이 없음을 나타내는 상태가 빈배열([])과 undefined 두 가지가 된다.
        - 차이가 없으나 이름을 구별한 것이 된다.

### 별칭은 타입체커와 런타임에 혼동을 야기할 수 있다.

```jsx
const { bbox } = polygon;
if (!bbox) {
  calculatePolygonBbox(polygon); // polygon.bbox가 채워지는 함수 
  // 이제 polygon.bbox와 bbox는 다른 값을 참조합니다.
}
```

- polygon이 변경될 경우 polygon.bbox와 bbox는 다른 값을 참조하게 된다.

```jsx
function fn (p: Polygon) { /*...*/ }; //polygon.bbox를 제거할 가능성이 있다.

polygon.bbox // BoundingBox | undefined
if(polygon.bbox) {
  polygon.bbox // 타입이 BoundingBox
  fn(polygon); // polygon.bbox 제거할 가능성 있음
  polygon.bbox // 타입이 BoundingBox
}
```

- 함수를 호출할 때마다 속성 체크를 반복해야한다면 성능에 좋지 않다
    - ⇒ 타입스크립트는 내부 함수가 타입 정제를 무효화하지 않는다고 가정
    - `polygon.bbox`로 사용하는 대신 `bbox` 지역변수로 뽑아내 사용하면 `bbox` 타입은 정확히 유지되나, `polygon.bbox`의 값과 같지 않게 유지될 수 있음

# 비동기 코드에서는 콜백 대신 async 함수 사용하기

- 과거 JS : 콜백 지옥을 필연적으로 마주할 수 밖에 없는 상황
    
    ```jsx
    fetchURL(url1, function(response1) {
        fetchURL(url2, function(response2) {
            fetchURL(url3, function(response3) {
                //...
                console.log(1);
            })
            console.log(2);
        })
        console.log(3);
    })
    console.log(4);
    ```
    
- → ES2015 : 콜백 지옥 개선을 위해 프로미스(`promise`) 개념 도입
    
    ```jsx
    const page1Promise = fetch(url1);
    page1Promise.then(response1 => {
        return fetch(url2);
    }).then(response2 => {
        return fetch(url3);
    }).then(response3 => {
        // ...
    }).catch(error => {
        // ...
    })
    ```
    
- → ES2017 : `async` & `await` 키워드 도입`await` : 각각의 프로미스가 처리(resolve)될 때까지 `fetchPages` 함수 실행 멈춤
    
    ```jsx
    async function fetchPages() {
        const response1 = await fetch(url1);
        const response2 = await fetch(url2);
        const response3 = await fetch(url3);
    }
    ```
    
    - `async` 함수 내에서 `await` 중인 프로미스가 거절(reject)되면 예외 던짐→ 일반적인 `try`/`catch` 구문 사용 가능
        
        ```jsx
        async function fetchPages() {
            try {
                const response1 = await fetch(url1);
                const response2 = await fetch(url2);
                const response3 = await fetch(url3);
            } catch (e) {
                // ...
            }
        }
        ```
        

ES5- 버전 대상으로 할 때의 TS 컴파일러 : `async` & `await`가 동작하도록 정교한 변환 수행

→ **TS는 런타임에 관계 없이 `async` & `await` 사용 가능**

콜백보다 `프로미스`나 `async`/`await`를 사용해야 하는 이유

- 콜백보다 프로미스가 **코드 작성**이 쉬움
- 콜백보다 프로미스가 **타입 추론**이 쉬움

ex) 병렬로 페이지 로드하고 싶다면 `Promise.all` 사용해 프로미스 조합

```jsx
async function fetchPages() {
    const [response1, response2, response3] = await Promise.all([
        fetch(url1), fetch(url2), fetch(url3)
    ]);
    // ...
}
```

→ best case. `await`와 구조 분해 할당

- TS : 세가지 `response` 변수 각각의 타입을 Response로 추론

```jsx
function fetchPagesCB() {
    let numDone = 0;
    const response: string[] = [];
    const done = () => {
        const [response1, response2, response3] = response;
        // ...
    };
    const urls = [url1, url2, url3];
    urls.forEach((url, i) => {
        fetchURL(url, r => {
            response[i] = url;
            numDone++;
            if (numDone === urls.length) done();
        })
    })
}
```

: 위 코드에 오류 처리 포함 & `Promise.all`과 같은 일반적 코드로의 확장은 어려움

- 입력된 프로미스 중 첫 번째가 처리될 때 완료되는 `Promise.race`도 타입 추론과 잘 맞음→ `Promise.race` 사용해 프로미스에 타임아웃을 추가하는 방법도 많이 사용됨

```jsx
function timeout(millis: number): Promise<never> {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject('timeout'), millis)
  })
}

async function fetchWithTimeout(url: string, ms: number) {
  return Promise.race([fetch(url), timeout(ms)])
}
```

- 프로미스 생성보다 `async`/`await`를 사용해야 하는 이유
    - 일반적으로 더 간결하고 직관적인 코드가 됨
    - async 함수는 항상 프로미스를 반환하도록 강제됨

```jsx
// function getNumber(): Promise<number>
async function getNumber() {
	return 42;
}

// async 화살표 함수
const getNumber = async () => 42; // type : () => Promise<number>

// 프로미스르 직접 생성하는 방법
const getNumber = () => Promise.resolve(42); // type : () => Promise<number>
```

: 즉시 사용 가능한 값에도 프로미스를 반환하는 것이 이상하게 보일 수 있으나,

**비동기 함수로 통일하도록 강제**하는 데 도움

→ 함수는 **항상 동기** 또는 **항상 비동기**로 실행되어야 하며, 혼용해서는 안됨

# 타입 추론에 문맥이 어떻게 사용되는지 이해하기

TS가 타입을 추론하는 방식 : 값 고려 → 값이 존재하는 곳의 문맥 고려

문맥을 고려해 타입 추론 시 종종 이상한 결과 발생

→ 타입 추론에 문맥이 어떻게 사용되는지 이해함으로써 대처

- 동일한 두 문장

```jsx
// 인라인 형태
setLanguage('Javascript');

// 참조 형태
let language = 'Javascript';
setLanguage(language);
```

```jsx
// 타입스크립트에서의 리팩터링
function setLanguage(language: string) { /* ... */}
setLanguage('Javascript'); // 정상

let language = 'Javascript';
setLanguage(language); // 정상
```

인라인 형태에서 TS는 함수 선언을 통해 매개변수가 Language 타입임을 알고 있음

이 값을 변수로 분리해내면, TS는 **할당 시점**에 타입 추론

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';

function setLanguage(language: Language) {
    console.log(language);
}

setLanguage('JavaScript') // inline

let language = 'JavaScript';
setLanguage(language); // variable
//          ~~~~~~~~ 'string' 형식의 인수는 'Language' 형식의 매개변수에 할당될 수 없습니다.
```

: `string`으로 추론 → `Language` 타입으로 할당이 불가능하므로 오류 발생

- 해결 방법
    1. 타입 선언에서 `language`의 가능한 값을 제한
        
        ```jsx
        let language: Language = 'Javascript';
        setLanguage(language); // 정상
        ```
        
    2. `language`를 상수로 만드는 방법
        
        ```jsx
        const language = 'Javascript';
        setLanguage(language); // 정상
        ```
        
        문맥으로부터 값을 분리함 → 근본적 문제 발생의 원인이 됨
        

### 튜플 사용 시 주의점

> 해당 문제의 발생 원인 : const는 얕게(shallow) 동작하기 때문
> 

```jsx
function panTo(where: [number, number]){
    console.log(where);
}
panTo([10, 20]); // inline

const loc = [10, 20]; // 문맥에서 값 분리
panTo(loc);
```

`loc`는 문맥에서 값을 분리하여 할당 시점에 `number[]`(길이를 알 수 없는 숫자의 배열)로 추론됨

- TS가 정확히 파악할 수 있도록 타입 선언을 제공하는 방법

```jsx
// 1. any를 사용하지 않고 오류를 고칠 수 있는 방법
const loc: [number, number] = [10, 20];
panTo(loc); // 정상

// 2. '상수 문맥'을 제공하는 방법
const loc = [10, 20] as const;
panTo(loc);
//    ~~~ 'readonly [10,20] 형식은 'readonly'이며
//         변경 가능한 형식 '[number, number]'에 할당할 수 없습니다.
```

1번 방식 : `const`는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow) 상수

2번 방식 : `as const`는 그 값이 내부까지(deeply) 상수

라는 사실을 TS에 알려줌

`any`를 사용하지 않고 오류를 고칠 수 있는 최선의 해결책 : `panTo` 함수에 `readonly` 구문 추가

```jsx
function panTo(where: readonly [number, number]){
        console.log(where)
    }
    const loc = [10, 20] as const;
    panTo(loc);
```

### 객체 사용 시 주의점

문맥에서 값을 분리하는 문제 : 큰 객체에서 상수를 뽑아낼 때도 발생

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage {
    language: Language;
    organization: string;
}

function complain(language: GovernedLanguage) {
    console.log(language.language);
}

complain({language : 'JavaScript', organization: 'Microsoft'});

const ts = {
    language : 'JavaScript',
    organization: 'Microsoft'
}
complain(ts);
```

`ts`객체에서의 `language` 타입은 `string`으로 추론

→ 타입 선언 추가(`const ts: GoveredLanguage = …`) or 상수 단언(`as const`)으로 해결

### 콜백 사용 시 주의점

콜백을 다른 함수로 전달 시, TS는 콜백의 매개변수 타입 추론을 위해 문맥 사용

```jsx
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
    fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
    console.log(a + b); // a, b type : number
})

const func = (a…, b) => {
    console.log(a + b); // a, b type : any
}

callWithRandomNumbers(func);
```

해당 경우는 매개변수에 타입 구문을 추가해 해결 가능

```jsx
const func = (a: number, b: number) => {
     console.log(a + b);
}

type callback = (a: number, b: number) => void;
const func2: callback = (a, b) => {
     console.log(a + b);
}
```

# 함수형 기법과 라이브러리로 타입 흐름 유지하기

**라이브러리 + TS**를 조합해 사용할 시 효과적

→ **타입 정보가 그대로 유지되며 타입 흐름이 지속적으로 전달되므로**

cf) 루프를 구현하면 타입 체크에 대한 관리도 직접 해야 함

- **csv 데이터 파싱 예제(JS)**
    - vanilla JS : 절차형 프로그래밍으로 구현 가능
        
        ```jsx
        const csvData = '...';
        const rawRows = csvData.split('\n');
        const headers = rawRows[0].split(',');
        
        const rows = rawRows.slice(1).map((rowStr) => {
          const row = {};
          rowStr.split(',').forEach((val, j) => {
            row[headers[j]] = val;
          });
          return row;
        });
        ```
        
    - 함수형 프로그래밍 선호 개발자 : `reduce` 사용 → 행 객체 생성
        
        ```jsx
        const rows = rawRows
          .slice(1)
          .map((rowStr) =>
            rowStr
              .split(',')
              .reduce((row, val, i) => ((row[headers[i]] = val), row), {})
          );
        ```
        
    - 키와 값의 배열로 취합해 객체로 만드는 `zipObject` 함수 이용
        
        ```jsx
        import _ from 'lodash';
        const rows = rawRows
          .slice(1)
          .map((rowStr) => _.zipObject(headers, rowStr.split(',')));
        ```
        

→ **JS** : 서드 파티 라이브러리 종속성을 추가할 때 신중해야 함

→ 코드를 짧게 줄이는 데 시간이 많이 들 경우, 사용 지양할 것

→ **TS** : **서드파티 라이브러리 사용이 무조건 유리**

→ 타입 정보를 참고해 작업할 수 있으므로, 서드파티 라이브러리 기반으로 변형 시 시간 단축

```jsx
// CSV 파서 절차형 & 함수형 버전 모두 같은 오류 발생
const rowsA = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(',').forEach((val, j) => {
    row[headers[j]] = val;
    // ~ '{}' 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없습니다.
  });
  return row;
});

const rowsB = rawRows.slice(1).map(
  (rowStr) =>
    rowStr
      .split(',')
      .reduce((row, val, i) => row[((headers[i] = val), row)], {})
  // ~ '{}' 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없습니다.
);
```

→ 두 버전 모두 `{}` 타입으로 `{[column: string]: string}` or `Record<string, string>` 제공 시 오류 해결

- 로대시 버전 : 별도 수정 없이 타입 체커 통과

```jsx
const rows = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(',')));
// 타입 : _.Dictionary<string>[]
```

→ 타입 구문 없이도 `rows`의 타입이 정확함

데이터의 가공이 정교해질수록 장점은 더욱 분명해짐

### NBA 팀 선수 명단으로 살펴 보는 데이터 가공 예제

```jsx
interface BasketballPlayer {
name: string;
team: string;
salary: number;
}
declare const rosters: {[team: string]: BasketbaIIPlayer[]};
```

**단순(flat) 목록 만들기**

- **루프**

```jsx
let allPlayers = [];
	// 'allPlayers' 변수는 형식을 확인할 수 없는 경우
	// 일부 위치에서 암시적으로 'any[]' 형식입니다.
for (const players of Object.values(rosters)) {
	allPlayers = allPlaye rs.concat(playe rs);
	// 'allPlayers'변수에는 암시적으로
	// 'any[]' 형식이 포함됩니다.
}
```

⇒ 오류 해결: 타입 구문 추가

```jsx
let allPlayers: BasketballPlayer[] = [];
for (const players of Object.values(rosters)) {
	allPlayers = allPlayers. concat(players); // 정상
}
```

- **Array.prototype.flat 사용하기**

```jsx
const allPlayers = Object.values(rosters).flat();
// 정상, 타입이 BasketballPlayer[]
```

**연봉  순으로 정렬해서 최고 연봉 선수의 명단 만들기**

- **로대시 없이 만들기**

```jsx
const teamToPlayers: {[team: string]: BasketbaIIPlayer[]} = {};
for (const player of allPlayers) {
	const {team} = player;
	teamToPlayers[team] = teamToPlayers[team] || [];
	teamToPlayers[team].push(player);
}

for (const players of Object.values(teamToPlayers)) {
	players.sort((a, b) => b.salary - a.salary);
}

const bestPaid = Object.values(teamToPlayers).map(players => players[0]);
bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary);
console.log(bestPaid);
```

- **로대시 활용하기**

```jsx
const bestPaid = _(allPlayers)
.groupBy(player => player.team)
.mapValues(players => _.maxBy(players, p => p.salary)!)
.values()
.sortBy(p => -p.salary)
.value() // 타입이 BasketballPlayer[]
```

- 길이가 절반으로 줄었다.
- 보기에 깔끔하다
- 로대시와 언더스코어의 개념인 ‘체인’을 사용했기 때문에, 더 자연스러운 순서로 일련의 연산을 작성할 수 있었다.

**체인X**

```jsx
_.c(_.b(_.a(v)))
```

**체인O**

```jsx
_(v).a().b().c().value()
```

⇒ 연산자의 등장 순서와 실행 순서가 동일하게 된다.

- _(v) 값을 래핑(wrap)
- .value() 값을 언래핑(unwrap)

내장된 함수형 기법들과 로대시 같은 라이브러리에 타입 정보가 잘 유지되는 것은 우연이 아니다.

→ 함수 호출 시 전달된 매개변수 값을 건드리지 않고 **매번 새로운 값을 반환해 새로운 타입으로 안전하게 반환 가능**

타입스크립트의 많은 부분이 **자바스크립트 라이브러리의 동작을 정확히 모델링하기 위해서 개발되었다.**

⇒ 라이브러리를 사용할 때 타입 정보가 잘 유지되는 점을 제대로 활용해야 타입스크립트의 목적을 달성할 수 있다.

### 대규

로대시가 구식 라이브러리 아닌가?

- 로대시는 아직도 많이 쓰나요?
    
    네, Lodash는 여전히 많은 프로젝트에서 사용됩니다. 특히 JavaScript와 TypeScript 개발 환경에서 유틸리티 함수들이 필요한 경우 Lodash는 매우 유용합니다. 다만 최근에는 다음과 같은 이유로 Lodash를 사용하지 않는 경우도 많아졌습니다:
    
    1. **ES6+ 기능 확장**
        
        JavaScript 자체가 발전하면서 Lodash가 제공하던 많은 기능을 대체할 수 있게 되었습니다.
        
        예를 들어, `Array.prototype.map`, `filter`, `reduce` 같은 메서드나, `Object.entries`, `Object.fromEntries` 등의 기능 덕분에 Lodash의 필요성이 줄어들었습니다.
        
    2. **Tree-shaking 및 번들 크기 관리**
        
        Lodash는 전체 라이브러리를 불러오면 번들 크기가 커질 수 있습니다. 하지만 `lodash-es` 같은 ES 모듈 버전이나 개별 함수만 가져오는 방법(예: `import cloneDeep from 'lodash/cloneDeep'`)으로 이를 해결할 수 있습니다.
        
    3. **경량 라이브러리 사용**
        
        Lodash 대신 더 작은 유틸리티 라이브러리를 사용하는 경우도 있습니다. 예:
        
        - **Ramda**: 함수형 프로그래밍 스타일을 지원.
        - **date-fns**: 날짜 처리에 특화.
        - **Native**: Lodash 없이 순수 JS로 해결.
    
    ### Lodash가 여전히 유용한 경우
    
    1. **호환성**
        
        Lodash는 오래된 브라우저나 Node.js 환경에서도 안정적으로 작동하기 때문에, 호환성이 중요한 프로젝트에서 선호됩니다.
        
    2. **복잡한 유틸리티 함수**
        
        Lodash의 `cloneDeep`, `debounce`, `throttle` 등은 여전히 강력하고 간단하게 사용할 수 있어 유용합니다.
        
    3. **코드 가독성과 생산성**
        
        Lodash 함수는 간결하고 직관적이어서 코드 가독성과 유지보수성이 높아집니다.
        
    
    ### 결론
    
    - 새로운 프로젝트에서는 Lodash를 적극적으로 사용하지 않을 수도 있지만, 여전히 특정 상황에서 생산성을 높이는 데 유용합니다.
    - Lodash의 대체 가능 여부는 프로젝트의 요구 사항과 팀의 선호도에 따라 달라질 것입니다.
    
    Lodash를 사용할 때는 **필요한 함수만 가져오는 방식**을 추천합니다. 😊
    

### 지원

문맥이라는 말이 어색하다 ⇒ 동의합니다.

### 스터디 활동 사진

![IMG_8193](https://github.com/user-attachments/assets/d39bc80f-0fa4-40c7-9b70-3e97c5d2c6f0)
