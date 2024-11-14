# 타입 추론

프로그래밍 언어에서 타입 추론은 타입을 명시적으로 지정하지 않아도 컴파일러가 변수나 표현식의 타입을 추론하는 과정입니다. 이는 코드를 더욱 간결하게 만들어주는 장점이 있으며, 특히 TypeScript에서 잘 활용되고 있습니다. 이 장에서는 타입 추론의 다양한 기법과 주의사항을 소개하고, 코드의 가독성과 유지보수성을 높이기 위한 최적의 방법을 안내합니다.

## 추론 가능한 타입을 사용해 장황한 코드 방지하기
타입스크립트에서는 명확한 타입 주석을 남기는 것이 중요하지만, 추론이 가능한 경우에는 불필요한 타입 선언을 줄이는 것이 좋습니다. 다음 예시는 불필요한 타입 선언을 줄이는 방식입니다:

```tsx
let x: number = 12; // 불필요한 타입 주석
let x = 12; // 타입스크립트가 x를 `number`로 추론
```

복잡한 객체도 타입을 명시하지 않고 선언할 수 있습니다:

```tsx
const person = {
  name: "Sojourner Truth",
  born: {
    where: "Swartekill, NY",
    when: "c.1797",
  },
  died: {
    where: "Battle Creek, MI",
    when: "Nov. 26, 1883",
  },
}; // person의 구조를 타입스크립트가 자동 추론
```


## 다른 타입에는 다른 변수 사용하기

각 변수는 특정 용도를 갖기 때문에 타입이 바뀌는 변수는 피하고, 목적이 다른 변수는 별도의 변수명으로 선언하는 것이 바람직합니다. 동일한 변수명으로 서로 다른 타입을 사용하는 경우 코드는 실행될 수 있지만 혼란을 초래할 수 있습니다.
```tsx
const id = "12-34-56";
fetchProduct(id);

const id = 123456; // 서로 다른 용도의 id 변수
fetchProductBySerialNumber(id); 
```


## 타입 넓히기
타입 넓히기에서는 타입스크립트가 값을 string으로 예상할 수 있는 경우에도 가능한 한 구체적인 타입으로 추론하는 과정을 다룹니다. 예를 들어, const로 선언한 변수는 고정된 문자열 리터럴로 타입이 추론됩니다.

```tsx
const axis1: string = "x"; // 타입은 string
const axis2 = "y"; // 타입은 "y" (구체적인 리터럴 타입)

```
이렇게 구체적인 리터럴 타입으로 추론하면, 특정 값에 대한 추가적인 제약을 걸 수 있습니다.

## 타입 좁히기
타입스크립트는 조건문과 같은 제어 흐름에서 타입을 좁히는 기능을 제공합니다. 이를 통해 오류를 줄이고 타입 안전성을 높일 수 있습니다. 예를 들어, 배열에서 undefined가 될 수 있는 값을 필터링할 때 타입 가드를 활용하면 타입을 명확히 좁힐 수 있습니다.

<예시코드>
```tsx
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}

const members = ["Janet", "Michael"]
  .map((who) => jackson5.find((n) => n === who))
  .filter(isDefined); // 타입이 string[]
```

## 한꺼번에 객체 생성하기
타입스크립트에서는 객체를 한꺼번에 생성하는 것이 좋습니다. 속성을 나누어 추가할 경우 타입 추론에 어려움이 생기고, 오류가 발생할 수 있습니다.

```tsx
const pt = {}; // 초기에는 빈 객체
pt.x = 3; // 오류 발생: '{}' 형식에 'x' 속성이 없습니다.
pt.y = 4; // 오류 발생: '{}' 형식에 'y' 속성이 없습니다.

// 올바른 방법
const pt = { x: 3, y: 4 }; // 객체를 한꺼번에 생성
```
이와 같은 패턴은 타입스크립트에서 타입 오류를 방지하는 데 유용합니다.


## 일관성 있는 별칭 사용하기
코드에서 별칭을 사용할 때는 일관성을 유지하는 것이 중요합니다. 별칭은 하나의 값을 가리키는 또 다른 이름으로, 값이 수정될 때 원본도 같이 수정되는 특성이 있습니다. 별칭의 사용은 코드 가독성을 높일 수 있지만, 무분별하게 사용할 경우 코드 흐름을 파악하기 어려워지고, 예기치 못한 오류가 발생할 수 있습니다.

```tsx
const borough = { name: 'Brooklyn', location: [40.688, -73.979] };
const loc = borough.location;

loc[0] = 0;
console.log(borough.location); // [0, -73.979]
```
이처럼 loc이 borough.location을 가리키고 있어 loc의 변경이 원본에 영향을 미칩니다. 따라서 별칭을 사용할 때는 가급적 조심스럽게 사용하여 코드의 명확성을 유지하는 것이 좋습니다.

## 비동기 코드에는 콜백 대신 async 함수 사용하기
타입스크립트에서 비동기 코드는 전통적인 콜백 방식보다 async/await 구문을 사용하는 것이 좋습니다. 콜백 방식은 중첩된 코드가 증가하면서 일명 “콜백 지옥”을 유발할 수 있습니다.


```tsx
async function fetchData() {
  try {
    const response1 = await fetch(url1);
    const response2 = await fetch(url2);
    const response3 = await fetch(url3);
    console.log(response1, response2, response3);
  } catch (error) {
    console.error(error);
  }
}

```

## 타입 추론에 문맥이 어떻게 사용되는지 이해하기
타입스크립트는 문맥적 단서를 통해 변수의 타입을 추론합니다. 함수 호출 시 매개변수나 리턴 타입이 문맥에 따라 달라지며, 이러한 문맥을 통해 더욱 정확한 타입 추론이 가능합니다.

```tsx
function panTo(where: [number, number]) { /* ... */ }
const loc = [10, 20] as [number, number];
panTo(loc); // 정상적으로 호출
```

여기서 as [number, number]와 같은 타입 단언을 통해 loc의 타입이 정확히 추론되도록 유도할 수 있습니다. 문맥에 맞는 타입 단언은 코드의 오류를 줄이는 데 유용합니다.

## 함수형 기법과 라이브러리로 타입 흐름 유지하기
타입스크립트는 함수형 프로그래밍 기법을 지원하며, 로대시(Lodash)와 같은 함수형 라이브러리를 통해 데이터 처리를 간결하게 구현할 수 있습니다. 이러한 기법을 활용하면 데이터 흐름이 깔끔하게 유지되며, 코드의 유지보수성이 높아집니다.

```tsx
import _ from 'lodash';

const bestPaid = _(allPlayers)
  .groupBy(player => player.team)
  .mapValues(players => _.maxBy(players, 'salary')!)
  .values()
  .sortBy('salary')
  .reverse()
  .value();

console.log(bestPaid);
```
이와 같이 체이닝을 통해 코드의 가독성을 높일 수 있으며, 타입스크립트는 체인의 각 단계에서 타입을 정확하게 추론합니다. 함수형 기법과 라이브러리를 사용하면 긴 코드가 짧고 명료하게 정리됩니다.

# 3장에서 중요하게 생각하는 부분
3장의 핵심은 타입스크립트의 타입 추론을 최대한 활용해 불필요한 타입 선언을 줄이고 코드의 가독성과 안전성을 높이는 것입니다. 이를 통해 더 간결하고 유지보수하기 쉬운 코드를 작성할 수 있습니다.







