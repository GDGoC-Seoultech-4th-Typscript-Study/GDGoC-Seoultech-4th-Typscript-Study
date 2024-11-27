# Typescript study

> Week 6

## 4장 타입 설계

### 유효한 상태만 표현하는 타입을 지향하기

```ts
// not good for representing state of the app
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

function renderPage(state: State) {
  if (state.error) {
    return "error: ${state.error}";
  } else if (state.isLoading) {
    return "Loading";
  }
  return "${currentPage}";
}

async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error("${newPage} loading unable: ${response.statusText}");
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = "" + e;
  }
}
```

### 문제점

- `renderPage`에 전달된 state가 isLoading도 true고 error도 true라면 -> 둘 중 뭐가 발생한 것인지?
- `changePage`에서 error가 발생한 상황에 isLoading은 여전히 true이다
- `state.error`의 초기화 과정이 빠짐 -> 페이지 전환 중에 과거의 오류 메시지를 띄울 가능성이 있음
- 페이지를 로딩하던 중에 사용자가 페이지를 바꿔버리면 어떤 일이 일어날 지 알 수 없음
  => 무효한 상태를 모두 허용하고 있는 상황

**문제점 수정**

```ts
// better way to write
interface RequestPending {
  state: "pending";
}

interface RequestError {
  state: "error";
  error: string;
}

interface RequestSuccess {
  state: "ok";
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}

function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];

  switch (requestState.state) {
    case "pending":
      return "Loading ${currentPage}";
    case "error":
      return "error: ${currentPage} loading unable: ${requestState.error}";
    case "ok":
      return "${currentPage}";
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: "other" };
  state.currentPage = newPage;

  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error("${newPage} loading unable: ${response.statusText}");
    }
    const pageText = await response.text();
    state.requests[newPage] = { state: "ok", pageText };
  } catch (e) {
    state.requests[newPage] = { state: "error", error: "" + e };
  }
}
```

수정된 코드:

- 무효한 상태 발생을 방지함
- 하나의 요청은 단 하나의 상태와만 대응됨

### 사용할 때는 너그럽게, 생성할 때는 엄격하게

**Robustness Principle**
TCP와 관련해서 "당신의 작업은 엄격하게 하고, 다른 사람의 작업은 너그럽게 받아들여라"는 말을 인용.

함수 시그니처 구성할 때 이런 규칙을 적용해라:

- 매개변수의 타입은 넓어도 되지만 - 너그럽게 받고
- 결과를 반환할 때는 구체적으로 해라 - 엄격하게 주기

```ts
declare function setCamera(cam: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
type LngLat =
  | { lng: number; lat: number }
  | { lon: number; lat: number }
  | [number, number];
```

ex: 3D 지도 보여주는 API(3D 매핑 API)

- `LngLat`: 위도와 경도를 표현하는 방식
- 다양한 객체를 지원해서 유연성을 높임
- `CameraOptions`의 필드도 선택적으로 받음

-> 이렇게 정의된 타입들을 가지고 함수를 작성했을 때

```ts
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {
    center: { lat, lng }, // error
    zoom,
  } = camera;

  zoom;
  window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

- `center` 자체가 undefined일 가능성 존재 -> `lat`, `lng`에 접근 불가
- `zoom`이 undefined일 가능성?
  -> 왜 발생함? `viewportForBounds`의 반환 타입이 너무 자유롭다.

해결: array와 arrayLike의 관계처럼, `LngLatLike`를 정의해서 사용

### 문서에 타입 정보를 쓰지 않기

: 주석 작성할 때 tip과 같은 이야기

- 주석을 장황하게 적기보다 타입 구문을 포함하여 코드를 깔끔하게 작성하는게 낫다.
-
- 코드와 주석이 일관적이지 못한 타입을 설명하는 경우

```ts
/* 전경색 문자열을 반환합니다. */
/* 0 또는 1개의 매개변수를 받습니다. */
/* 매개변수가 없으면 표준 전경색을 반환합니다. */
function getForegroundColor(page?: string) {
  return page === "login" ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
}
```

**문제점**

- 주석은 string을 return한다고 적어놨지만, 코드는 {r, g, b} 객체를 반환하고 있음
- 타입 시그니처를 통해 이미 매개변수를 어떻게 받는지가 명시적으로 설명됨
- 주석 길이 >> 함수 선언 및 구현

=> 어차피 타입 구문은 컴파일러가 체크해 주니까, 굳이 주석에 장황하게 적지 마라.
대신 리턴값의 타입 구문을 작성해서 코드가 추후 수정되더라도 동기화될 수 있도록 하자.

개선:

```ts
/* 앱 또는 페이지의 전경색을 가져옵니다. */
function getForegroundColor(page?: string): Color {
  // ...
}
```

- 불변값 설명
  : 주석으로 적지 말고 `readonly` 키워드를 사용해라

> +변수명에도 타입 정보를 담지 말 것
> `ageNum` -> `age: number`

### 타입 주변에 null 값 배치하기

값이 전부 null이거나, 전부 null이 아닌 경우 -> 값이 섞여있을 때보다 다루기 쉬움
=> 그러므로 타입에 null을 추가해서 모델링하자

```ts
// passes type checker w/o strictNullchecks
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      // if min == 0 ; !false => true
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max]; // number | undefined
  // when strictNullchecks is enabled
}
```

- `min`이나 `max`가 0이라면? => `min`과 `max`는 0으로 초기화됨 -> `!min`은 여전히 true이므로 num == 1일때도 `(!min)` 조건을 만족하여 1이라는 값이 덧씌워지게 됨
- `nums`가 빈 배열이라면? => return [undefined, undefined]

**`strictNullchecks`가 활성화됐을 때의 문제**

- `!min`이라는 조건을 통해 `min`이 `undefined`일 가능성은 배제, `max`는 그렇지 않으므로 `max`의 타입은 `number | undefined`로 추정됨

개선:

```ts
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}
```

return의 타입을 `[number, number] | null`로 만들어놓기

- null 아님 단언(!)을 사용하기 -> 전부 null이 아닌 경우
- if문으로 null 잡아내기

=> extend의 결과값으로 단일 객체를 return하는 방식으로 설계를 개선한 것.

### 클래스에서 null 정리하기

```ts
class UserPosts {
  user: UserInfo | null;
  posts: Posts[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }
  async init(userId: string) {
    return Promise.all([
      async () => (this.user = await fetchUser(userId)),
      async () => (this.posts = await fetchPostsForUser(userId)),
    ]);
  }

  getUserName() {
    // null인가요? 아닌가요? * 4 cases
  }
}
```

**발생할 수 있는 경우**

- user와 posts가 모두 null일 경우
- user만 null일 경우
- posts만 null일 경우
- user와 posts가 모두 null이 아닐 경우
  => 각 경우를 모두 체크함, 좋지 못한 설계

개선:

```ts
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promis<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId),
    ]);
    return new UserPosts(user, posts);
  }

  getUserName() {
    return this.user.name;
  }
}
```

"모든 값이 준비되었을 때 생성하도록 함"

- UserPosts 클래스는 절대 null이 아님

### 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 벡터를 그리는 프로그램을 작성한다면

```ts
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- layout은 LineLayout인데 paint는 FillPaint인, 무효한 상태의 가능성이 있게 됨

개선:

```ts
// union of interfaces
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}

interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}

interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

-> layout과 paint가 잘못된 조합으로 섞이는 것을 방지
동시에 "유효한 상태만을 표현하는 타입 정의 방식"

- 태그된 유니온

```ts
interface Layer {
  type: "fill" | "line" | "point";
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

이 역시 무효한 상태를 가질 수 있는 유니온의 인터페이스

인터페이스의 유니온으로 개선(+ Tagged Union):

```ts
interface FillLayer {
  type: "fill";
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: "line";
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: "paint";
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- type의 속성은 태그
- 이를 이용해서 어떤 타입의 Layer가 사용되는지 타입 범위 좁히기가 가능해짐

---

> 혼동 방지
> **인터페이스는 유니온으로 쓸 수 없다고 하지 않았나요**

- `interface Layer{ }`로 선언한 인터페이스는 **속성**을 유니온으로 가지는 것이고, 인터페이스 자체는 단일 구조로 생성됨
- 즉 불가능한 구조는 아니지만, 유효한 상태를 가지도록 수정하기 위해 아래와 같이 바꾼 것
- interface A | B 식으로 작성할 수 없다는 의미.
- 수정된 코드에서도 볼 수 있듯, type은 여전히 유니온으로 생성 가능

## 스터디 활동

### 소감 및 질문

- 아이템 28 "유효한 상태만 표현하는 타입을 지향하기"를 마무리지으며 인용한 "코드가 길어지거나 표현하기 어렵지만
  결국은 시간을 절약하고 고통을 줄일 수 있습니다."라는 말에 공감한다.

### 사진
![389374148-d39bc80f-0fa4-40c7-9b70-3e97c5d2c6f0](https://github.com/user-attachments/assets/b28d9841-dceb-44db-bac1-dc92e0ceb906)
