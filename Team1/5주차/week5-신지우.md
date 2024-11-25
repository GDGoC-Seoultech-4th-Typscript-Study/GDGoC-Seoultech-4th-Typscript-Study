# 아이템 23 한꺼번에 객체 생성하기

**변수의 값은 변경될 수 있지만, 타입은 일반적으로 변경되지 않는다.**

- 자바스크립트 패턴을 타입스크립트로 모델링 하는게 쉬워진다.
- 객체를 생성할 때는 여러 속성을 한꺼번에 생성하는 것이 타입 추론에 유리하다.

### **속성 추가 시 오류 발생 (TypeScript)**

```tsx
typescript
코드 복사
const pt = {};
pt.x = 3; // ~ '{}' 형식에 'x'속성이 없습니다.
pt.y = 4; // ~ '{}' 형식에 'y'속성이 없습니다.
```

**해결 방법**

1. 객체를 한 번에 정의

```tsx
const pt = { x: 3, y: 4 }; // 정상
```

1. 타입 단언문 사용:

```tsx
const pt = {} as { x: number; y: number };
pt.x = 3;
pt.y = 4; // 정상
```

1. 선언 시 타입 지정:

```tsx
interface Point { x: number; y: number; }
const pt: Point = { x: 3, y: 4 }; // 정상
```

## 작은 객체를 조합해서 큰 객체 만들기

**객체 전개 연산자 사용:**

```tsx
const pt = { x: 3, y: 4 };
const id = { name: 'Pythagoras' };
const namedPoint = { ...pt, ...id }; // 정상
namedPoint.name; // 타입이 string
```

## **조건부 속성 추가**

```tsx
declare let hasMiddle: boolean;
const firstLast = { first: 'Harry', last: 'Truman' };

const president = {
  ...firstLast,
  ...(hasMiddle ? { middle: 'S' } : {})
};
// middle은 선택적 속성으로 추가됨

```

# 아이템 24 일관성 있는 별칭 사용하기

### **별칭 문제**

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (polygon.bbox) {
    if (pt.x < box.x[0] || pt.x > box.x[1]) { /* 오류 발생 */ }
  }
}
```

- **해결책: 객체 비구조화**

```tsx
typescript
코드 복사
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon;
  if (bbox) {
    const { x, y } = bbox;
    if (pt.x < x[0] || pt.x > x[1]) { /* 정상 */ }
  }
}
```

# 아이템 25 비동기 코드에서는 콜백 대신 async 함수 사용하기

## 비동기 코드에서는 콜백 대신 `async` 함수 사용하기

### **👎 콜백 지옥**

```tsx
fetchURL(url1, (response1) => {
  fetchURL(url2, (response2) => {
    fetchURL(url3, (response3) => { /* ... */ });
  });
});
```

### **👍** `async`/`await`

```tsx
async function fetchPages() {
  try {
    const response1 = await fetch(url1);
    const response2 = await fetch(url2);
    const response3 = await fetch(url3);
  } catch (e) { /* ... */ }
}
```

**병렬 처리:**

```tsx
async function fetchPages() {
  const [response1, response2, response3] = await Promise.all([
    fetch(url1),
    fetch(url2),
    fetch(url3),
  ]);
}
```

# 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

### **문맥 추론 문제**

```tsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';

function setLanguage(language: Language) { /* ... */ }
let language = 'JavaScript';
setLanguage(language); // 오류: 'string' 형식은 'Language'에 할당 불가
```

**해결책:**

1. 타입 선언:

```tsx
let language: Language = 'JavaScript'; // 정상
```

1. 상수 사용:

```tsx
const language = 'JavaScript'; // 정상
```

# 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

### **로대시 활용 예제**

**단순 목록 생성:**

```tsx
const allPlayers = Object.values(rosters).flat(); // 타입: BasketballPlayer[]
```

**최고 연봉 선수 정렬:**

```tsx
const bestPaid = _(allPlayers)
  .groupBy((player) => player.team)
  .mapValues((players) => _.maxBy(players, (p) => p.salary)!)
  .values()
  .sortBy((p) => -p.salary)
  .value(); // 타입: BasketballPlayer[]
```
