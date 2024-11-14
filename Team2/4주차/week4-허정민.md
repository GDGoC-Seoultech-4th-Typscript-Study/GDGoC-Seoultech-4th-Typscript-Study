# typescript 4주차

---

### item 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입 추론은 명시해야 하는 타입 구문 수를 줄여 코드의 전체적인 안정성을 향상
- 복잡한 객체도 타입 추론이 가능하고 오히려 타입스크립트가 더 정확할 수도 있다

```jsx
const axis1: string = 'x';  //타입은 string
const axis2 = 'y';          //타입은 "y"
```

- 비구조화 할당문으로 모든 지역 변수의 타입이 추론된다

```jsx
function logProduct(product: Product){
  const {id, name, price} = product;
  console.log(id, name, price);
}
```

정보가 부족해 타입스크립트가 스스로 타입을 판단하기 어려운 상황에 명시적 타입 구문이 필요하다. 타입스크립트는 최종 사용처까지 고려하지 않는다. 

함수 매개변수에 타입 구문을 생략하는 경우

1. 기본 값이 있는 경우
2. 타입 정보가 있는 라이브러리에서, 콜백 함수의 매개변수 타입은 자동으로 추론된다

타입 추론이 가능하지만 명시하고 싶은 경우

1. 객체 리터럴을 정의할 때 

잉여 속성 체크가 동작한다. 이는 선택적 속성이 있는 타입의 오타 같은 오류를 잘 잡는다. 

변수가 할당하는 시점에 오류가 표시되도록 해준다. 

타입 문구를 제대로 명시하면, 실수가 발생한 부분에 오류를 표시해준다

1. 함수의 반환에 타입을 명시한다

구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입을 명시한다

함수에 대해 명확히 알 수 있다 

명명된 타입을 사용하기 위해 타입을 명시해야 한다.

---

### item 20 다른 타입에는 다른 변수 사용하기

- 자바스크립트에서는 한 변수를 다른 목적을 가지는 타입으로 재사용 가능

but, 타입스크립트는 오류가 발생한다. 

> 변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다
> 

타입을 바꾸려면 타입을 더 작게 제한해야 한다.

```jsx
let id: string | number = "12-34-56";
fetchProduct(id);
id = 123456
fetchProductBySerialNumber(id);
```

할당문에서 유니온타입으로 범위가 좁혀져 동작은 하지만 더 큰 문제 발생 가능

다른 타입에 별도의 변수를 사용하는게 좋은 이유

- 서로 관련이 없는 두 개의 값을 분리한다.
- 변수명을 더욱 구체적으로 적는다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
- 타입이 조금 더 간결해진다.
- let 대신 const로 변수를 선언할 수 있다.

---

### item 21 타입 넓히기

- 넓히기 과정

정적 분석 시점에, 변수는 가능한 값들의 집합인 타입을 가진다. 변수가 초기화 될 때 타입을 명시하지 않으면 타입체커는 직접 타입을 결정하고, 값을 가지고 할당 가능한 값들의 집합을 유추하는 과정

```jsx
let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
                 // ~ 'string' 형식의 인수는 '"x"|"y"|"z"'
                 // 형식의 매개변수에 할당될 수 없습니다.
```

실행은 되지만 편집기에 오류 발생

→ x의 타입은 넓히기가 동작해 string으로 추론 됐고, 이는 "x"|"y"|"z"타입에 할당 불가

- 타입스크립트의 넓히기 과정 제어

const 변수로 선언하기

```jsx
const x = 'x';  //타입이 x
let vec = { x: 10, y:20, z: 30 };
getComponent(vec, x) //정상
```

const는 만능이 아니다. 객체와 배열에선 여전히 문제가 있다.

- 타입스크립트의 기본 동작을 정의하는 방법
1. 명시적 타입 구문 제공
2. 타입 체커에 추가적인 문맥을 제공
3. const 단언문 사용

```jsx
const v1 = {
  x: 1,
  y: 2,
}; //타입은 { x: number; y: number; }
const v2 = {
  x: 1 as const,
  y: 2m
}; //타입은 { x:1 , y: number; }
const v3 = {
  x: 1,
  y: 2,
} as const; //타입은 { readonly x: 1; readonly y: 2; }
```

값 뒤에 as const를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론한다.

→ 넓히기가 작용하지 않는다.

---

### item 22 타입 좁히기

타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정

- null 체크

```jsx
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if (el) {                                  // 타입 좁히기
  el                                       // 타입이 HTMLElement
  el.innerHTML = 'Party Time'.blink();
} else {
  el                                       // 타입이 null
  alert('No element #foo');
}
```

조건문에서 타입  좁히기를 잘 해내지만, 타입 별칭이 존재하면 그러지 못할 수 있다

- instanceof 사용

```jsx
function contains(text:string, search:string|RegExp){
  if (search instanceof RegExp) {
    search;   // 타입이 RegExp
    return !!search.exec(text);
  }
  search     // 타입이 string
  return text.includes(search);
}
```

- 속성 체크

- Array.isArray같은 내장 함수

```jsx
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList // 타입이 string[]
}
```

- 명시적 태그를 붙이기 (태그된 유니온, 구별된 유니온)

```jsx
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: ’download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
   switch (e.type) {
     case 'download':
       e // 타입이 DownloadEvent
       break;
     case 'upload':
       e; // 타입이 UploadEvent
       break;
  }
}
```

사용자 정의 타입 가드 

반환 타입의 el is HTML InputElement는 함수의 반환이 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려줍니다.