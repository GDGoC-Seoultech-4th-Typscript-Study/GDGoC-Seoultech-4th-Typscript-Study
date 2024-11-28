# typescript 6주차

---

### item 28 유효한 상태만 표현하는 타입을 지향하기

효과적으로 타입을 설계하기 위해서, 유효한 상태만 표현할 수 있는 타입을 만드는 것이 가장 중요하다.

ex)

페이지를 그리는 renderPage 함수를 작성할 때 상태 객체의 필드 전부 고려해 상태 표시를 분기 해야 함.

```jsx
function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```

분기 조건이 명확히 분리되어있지 않다.

isLoading이 true이고 동시에 error 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 구분할 수 없다.

페이지를 전환하는 changePage 함수

```jsx
async function changePage(state: State, newPage: string) {
  state.isLoading = true; //
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false; 
    state.pageText = text;
  } catch (e) {
    state.error = '' + e; 
  }
}
```

changePage의 문제점

- 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져 있다.
- state.error를 초기화하지 않아 페이지 전환 중에 과거의 오류 메세지를 보여주게 된다.
- 페이지 로딩 중에 사용자가 페이지를 바꾸면  어떤 일이 벌어질지 예상할 수 없다.

> 상태 값의 두가지 속성이 동시에 정보가 부족하거나, 두가지 속성이 충돌함
> 

→ 무효한 상태가 존재하면 renderPage와 changePage 둘 다 구현할 수 없다.

ex) 유효한 상태

‘성공 상태’, ‘로딩 상태’, ‘오류 상태’로 각각 인터페이스를 분리한다

```jsx
interface RequestPending {
  state: 'pending';
}
interface RequestError {
  state: 'error';
  error: string;
}
interface RequestSuccess {
  state: 'ok';
  pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: {[page: string]: RequestState};
}
```

상태를 나타내는 타입 코드의 길이가 서너 배 길어졌지만, 무효한 상태를 허용하지 않도록 개선되었다.ㄱ

→ renderPage와 changePage 함수는 쉽게 구현 가능

---

### item 29 사용할 때는 너그럽게, 생성할 때는 엄격하게

존 포스텔의 견고성 원칙

> TCP 구현체는 견고성의 일반적인 원칙을 따라야한다. 당신의 작업은 엄격하게 하고, 다른사람의 작업은 너그럽게 받아들여야 한다.
> 

이는 함수 시그니처의 타입에도 적용되는 말이다. 함수의 매개변수는 타입의 범위가 넓어도 괜찮지만, 결과를 반환할 때는 일반적으로 타입의 범위가 ****구체적이어야 한다.

하지만 너무 범위를 넓게 잡게 된다면  undefined로 추론될 수 있다.

매개변수 타입의 범위가 넓으면 사용하기 편리하지만 반환 타입의 범위가 넓으면 불편하다.

선택적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 일반적이다.타입 선언 시에는 명시적으로 엄격하게 선언하고, 타입 조건을 완화하여 느슨한 타입으로 만들어 매개변수에 사용하는 것이 좋다. 반대로 타입을 반환할 때에는 반드시 엄격한 타입을 반환해야 사용성이 좋아진다.

---

### item 30 문서에 타입 정보를 쓰지 않기

ex) 

```jsx
/**
* 전경색(foreground) 문자열을 반환한다.
* 0 개 또는 1개의 매개변수를 받는다.
* 매개변수가 없을 때는 표준 전경색을 반환한다.
* 매개변수가 있을 때는 특정 페이지의 전경색을 반환한다.
*/

function getForegroundColor(page? : string) {
  return page === 'login' ? {r: 127, g: 127, b: 127} : {r: 0, g: 0, b: 0};
}
```

코드와 주석의 정보가 맞지 않아 잘못된 상태임은 분명하다.

의도된 동작이 코드에 제대로 반영되고 있다고 가정하면, 주석의 문제점

1. 함수가 string 형태의 색깔을 반환한다고 적혀 있지만 실제로는 {r, g, b} 객체를 반환한다.
2. 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하고 있지만, 타입 시그니처만 봐도 명확하게 알 수 있는 정보이다.
3. 불필요하게 장황하다. 함수 선언과 구현체보다 주석이 더 길다.

→ 함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 나은 방법이다.

누군가 강제하지 않는 이상 주석은 코드와 동기화되지 않는다. 하지만 주석 대신 타입 정보는 코드가 변경된다 하더라도 정보가 정확히 동기화된다.

ex) 주석을 개선한 코드

```jsx
/** 애플리케이션 또는 특정 페이지의 전경색을 가져옵니다 */
function getForegroundColor(page?: string): Color {
  // ...
}
```

만약 주석에서 특정 매개변수를 설명하고 싶다면 JSDoc의 @param 구문을 사용하면 된다.

값을 변경하지 않는다는 주석도 좋지 않다.

```jsx
/** nums를 변경하지 않습니다.  */
function sort(nums: number[]) {/* ... */ }
```

이것은 타입스크립트의 readonly를 사용하여 타입스크립트가 규칙을 강제할 수 있게 하면 되기 때문이다.

```jsx
function sort(nums: readonly number[]) {/* ... */}
```

주석에 적용한 규칙은 변수명에도 그대로 적용할 수 있다. 변수명에 타입 정보를 넣지 않도록 해야한다.

단위가 있는 숫자들은 예외로, 단위가 무엇인지 확실치 않다면 변수명 또는 속성 이름에 단위를 포함할 수 있다.

---

### item  31 타입 주변에 null 값 배치하기

stirctNullChecks 설정을 처음 키면, null이나 undefined 값 관련된 오류들이 갑자기 나타난다. 한 범위 안의 변수가 null인 경우와 그렇지 않은 경우보다, 모두 null이거나 전부 null이 아닌 경우로 분명히 구분하는 것이 쉽다.

ex) 숫자들의 최솟값과 최대값을 계산하는 extent 함수

```jsx
const extent = (nums: number[]) => {
  let min, max;

  for (const num of nums) {
    if (!min) {
      min = num;
      max = nu;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num); // 에러 발생
    }
  }
  return [min, max]
}
```

strictNUllChecks 없이 생기는 문제점

- 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버린다.
- nums 배열이 비어 있다면 함수는 [undefined, undefined]를 반환한다.

strickNullChecks 설정을 켰을 때 문제점

- extent의 반환 타입이 (number | undefined)[]로 추론되어 extent를 호출하는 곳마다 타입 오류의 형태로 나타난다. 이 오류는 undefined를 min에서는 제외되지만 max에서는 제외하지 않아 발생되었다.

ex)  해법: min과 max를 한 객체 안에 넣고 null이거나 아니게 한다.

```jsx
  const extent = (nums: number[]) => {
    let result: [number, number] | null = null;
    for (const num of nums) {
      if (!result) {
        result = [num, num];
      } else {
        result = [Math.min(num, result[0]), Math.max(num, result[1])];
      }
    }
    return result;
  };
```

반환 타입이 [number, number] | null이 되어서 사용하기가 더 수월해 졌다.

- 설계 시 한 값의 null여부가 다른 값의 null 여부에 암시적으로 관련되도록 하면 X
- API 작성 시 반환 타입을 큰 객체로 만들고, 타입 전체가 null이거나 null이 아니게 만들어야한다.
- 클래스를 만들 때 필요한 모든 값이 준비되었을때 생성하여 null이 존재하지 않도록 해야함

---

### item 32 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유티온 타입의 속성을 가지는 인터페이스보다, 인터페이스의 유니온 타입을 사용하는 것이 더 알맞다. 

ex)

```jsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

layout이 LineLayout형태이면서 paint 속성이 FillPaint인 것은 말이 안된다.

더 나은 방법: 각각 타입 계층을 분리된 인터페이스로 둬야 한다.

```jsx
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer{
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

이런 형태로 인터페이스의 유니온을 사용하게 되면 잘못된 조합으로 타입이 섞이는 것을 방지할 수 있다.

태그된 유니온 

```jsx
interface Layer {
  type: 'fill' | 'line' | 'point';
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

type: ‘fill’과 함께 LineLayout과 PointPaint 타입이 쓰이는 것은 말이 되지 않다.

Layer를 인터페이스의 유니온으로 변환

```jsx
interface FillLayer {
  type: 'fill';
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer{
  type:'paint';
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

type 속성은 태그이며, 어떤 타입의 Layer가 사용되었는지 판단하여 범위를 좁힐 때 사용될수도 있다.

타입스크립트의 코드가 정확성을 체크하는 데 도움은 되지만 반복되는 코드가 많아 보여서 복잡해 보인다.

어떤 타입을 태그된 유니온으로 표현할 수 있다면, 그렇게 하는 것이 좋다. 여러개의 선택적 필드가 동시에 값이 존재하거나, undefined인 경우에 태그된 유니온 패턴을 이용하면 문제점을 잘 해결할 수 있다.

ex) 

```jsx
interface Person {
  name: string;
  birth?: {
    place: string;
    date : Date;
  }
}
```

place 만 있고, date가 없는 경우에는 오류가 발생한다.

만약 타입 구조를 직접 손 댈 수 없는 상황이면, 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링 할 수 있다. 

```jsx
interface Name {
  name: string;
}
interface PersonWithBirth extends Name{
  placeOfBirth: string;
  dateOfBirth: Date;
}
type Person = Name | PersonWithBirth;
```

결론: 유니온 타입의 속성을 여러 개 가지는 인터페이스는 속성 관의 관계가 분명하지 않아 유효하지 않은 상황이 발생할 수 있고 실수도 발생할 수 있기 때문에 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하는 게 더욱 정확할 수 있다. 또한, 제어 흐름을 분석하기 위하여 타입 태그를 추가하는 것도 좋은 고려사항 중의 하나이다.