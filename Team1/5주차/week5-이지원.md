# Typescript study

> Week 5

## 3장 타입 추론 (cont'd)

### 한꺼번에 객체 생성하기

값은 변경될 수 있지만, 타입은 일반적으로 변경되지 않는다.

```ts
const pt = {};
pt.x = 3; // typescript에서는 에러가 발생함
```

- 위 예시에서 pt의 타입은 {} 값으로 추론된다.
- 따라서 'x' 속성이 없다는 에러가 발생하게 됨

### 객체 전개 연산자

작은 객체를 조합해서 큰 객체를 만들어야 할 때 여러 단계를 거쳐 합치기보다 `...` 연산자를 통해 한 번에 작은 객체들의 속성을 추가하기

- 필드 단위로 객체를 생성하는 것이 가능해짐

```ts
const pt0 = {};
const pt1 = { ...pt0, x: 3 };
const pt: Point = { ...pt1, y: 4 };
```

- 안전하게 조건부로 속성을 추가하려면 null 또는 {}를 이용한 객체 전개를

```ts
declare let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S' : {}})};

// president 타입은 아래와 같이 추론됨
const president: {
    middle?: string; // 있을 수도 있고 없을 수도 있고
    first: string;
    last: string;
}
```

+유니온을 사용하면 가능한 값의 집합을 더 정확히 표현할 수 있음

### 일관성 있는 별칭 사용

어떤 객체에 별칭을 사용하고 별칭의 값을 바꾸면 원래 속성값도 변경된다.
그러나 지나친 별칭 사용은 제어 흐름 분석을 방해할 수 있다.

```ts
interface Polygon {
    exterior: Coord[];
    holes: Coord[][];
    bbox? : Boundingbox; // Boundingbox | undefined
}

function isPointInPolygon(polygon: Polygon, pt: Coord) {
    const box = polygon.bbox;
    if(polygon.bbox) {
        if(pt.x < box.x[0] || pt.x > box.x[1] || ...)
        // error
    }
}
```

- 별칭인 `box`는 BoundingBox | undefined 타입으로 추정됨
- 속성 체크를 통해 `polygon.bbox`는 undefined가 되지 않도록 validation 과정을 거쳤으나, if문 안에서 별칭 `box`를 사용하여 오류가 발생함
  -> 결과적으로 별칭 사용은 타입을 좁히는 데에 방해가 됨

### 객체 비구조화

```ts
function isPointlnPolygon(polygon: Polygon, pt: Coordinate) {
    const {bbox} = polygon;
    if (bbox) {
        const {x, y} = bbox;
        if (pt.x < x[0] || pt.x > x[l] | |
            pt.y < y[0] || pt.y > y[l]) {
        return false;
        }
    }
}
// ...
```

`const {bbox}`와 같이 작성하는 것이 **객체 비구조화**
(cf. 구조 분해 할당됨)

### 비동기 코드에 콜백 대신 async 사용하기

```ts
fetchURL(urll, function(responsel) {
    fetchURL(url2, function(response?) {
        fetchURL(url3, function(responses) {
// ...
            console.log(l);
        })；
        console.log(2);
        })；
    console.log(3);
    })；
console.log(4);
```

위 코드의 실행 결과 로그 출력은 4, 3, 2, 1 순서로 나타남

-> 프로미스를 사용하여 수정한 코드

```ts
const pagelPromise = fetch(urll);
pagelPromise.then(responsel => {
    return fetch(url2);
}).then(response? => {
    return fetch(url3);
}).then(responses => {
    // ...
}).catch(error => {
    // ...
})；
```

- 병렬적으로 페이지를 로드하는 경우

```ts
async function fetchPages() {
    const [responsel, response?, responses] = await Promise.all([
        fetch(urll), fetch(ur!2), fetch(ur!3)
    ])；
    // ...
}
```

-> 내부는 병렬적으로 처리되는 코드

**Promise.race**
입력된 프로미스들 중 첫 번째가 처리되면 완료됨

-
- async, await를 사용하는 것이 좋음
- 어떤 함수가 Promise를 반환한다면 async로 선언하는 것이 좋음

### 타입 추론에 문맥이 어떻게 사용되는지 이해하기

```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python*;
function setLang나age(language: Language) {/*...*/}

setLanguage('JavaScript');

let language = 'JavaScript';
setLanguage(language); // error
```

문자열 리터럴 'Javescript'는 가능하지만, let으로 분리해내면 'Javascript'는 string 타입으로 추론되어 할당 시 오류가 발생함

- 객체에서 상수를 뽑아낼 때

```ts
type Language = 'JavaScript' | 'TypeScript' |
’Python';
interface GovernedLanguage {
    language: Language;
    organization: string;
}

function complain(language: GovernedLanguage) {/*■■■*/}

complain({ language: 'TypeScript', organization: 'Microsoft' }); // 정상

const ts = {
    language: 'TypeScript',
    organization: 'Microsoft',
}；
complain(ts); // error
```

이 예시에서도 역시 string 형식을 'Language' 형식에 할당할 수 없다는 에러가 발생함.
=> 타입 선언을 추가하거나 단언을 추가하여 해결 가능함

- 콜백의 매개변수 추론

```ts
function callWithRandNum(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

const fn = (a, b) => {
  // a, b는 any일 가능성이 생겨버림
  console.log(a + b);
};

callWithRandNum(fn);
```

이 예시는 `const fn`에 a와 b의 타입 구문을 추가하면 해결된다.
콜백을 상수로 뽑아내는 과정에서 문맥이 소실되어 발생한 오류이다.
