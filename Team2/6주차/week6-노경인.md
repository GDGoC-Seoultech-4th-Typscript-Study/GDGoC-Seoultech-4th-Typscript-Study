
## 아이템 28. 유효한 상태만 표현하는 타입을 지향하기

- 유효한 상태만 표현할 수 있는 타입을 만드는 것이 중요

```jsx
interface State {
	pageText: string;
	isLoading: boolean;
	error?: string;
}

function renderPage(state: State) {
	if (state.error) {
		return 'Error! Unable to load ${currentPage}: ${state.error}';
	} else if (state.isLoading) {
		return 'Loading ${currentPage}.••';
	}
	return '<h1>${currentPage}</h1>\n${state.pageText}';
}
```

→ 해당 renderPage 메소드에 대한 오류가 발생을 했을 때, 해당 오류가 isLoading인 상태라서 페이지 렌더링이 안되는 건지, 아니면 다른 오류가 발생하여 렌더링이 안되는 건지 알 수 없는 상황이 발생 ⇒ “분기 조건이 명확히 분리되지 않음”. : ‘무효한 상태’ 발생

상태 값의 두 가지 속성이 동시에 정보가 부족하거나 두 가지 속성이 충돌하는 상황이 생김. ⇒ State가 두 상황이 동시에 존재할 수 있는 상황(무효한 상태)을 허용하고 있음.

이런 상황이 발생하면 오류가 발생했을 때, 원인을 알기 힘들기 때문에 유효한 상태에 대해서만 표현하는 타입을 지향해야함 → 시간을 절약 가능 (오류 발생 시, 해결을 빠르게 가능)

- 무효한 상태를 발생 시키지 않기 위한 코드

```jsx
interface RequestPending {
	state: 'pending1;
}

interface RequestError {
	state: 'error';
	error: string;
}

interface Requestsuccess {
	state: 'ok';
	pageText: string;
}

type Requeststate = RequestPending | RequestError | Requestsuccess;

interface State {
	currentPage: string;
	requests: {[page: string]: Requeststate};
}
```

→ 전체적인 코드의 길이는 길어질 수 있지만, 문제가 발생한 상황을 해결하는데 더욱 시간을 절약할 수 있다는 장점이 존재하기 때문에, 많은 분기를 설정하여 이에 대한 코드를 짜주는 습관을 갖는 것이 좋음

## 아이템 29. 사용할 때는 너그럽게, 생성할 때는 엄격하게

> *함수 시그니처에서 함수의 매개변수는 타입의 범위를 넓게 지정 해도 되지만, 결과를 반환하는 return 값에 대해서는 타입의 범위가 구체적이어야 함.*
> 

- 매개변수로 들어올 수 있는 값은 굉장히 다양하기 때문에, 유니온을 사용해서 여러 가지 경우의 수를 제공하는 게 좋음. ⇒ 자유로워짐.
- 하지만 return 값에 대해서는 선택적 속성을 가지면 해당 메서드를 사용하기 어렵게 만듬. ⇒ 사용하기 편리한 api일 수록 반환 타입이 엄격해야함.

```jsx
interface CameraOptions {
	center?: LngLat;
	zoom?: number;
	bearing?: number;
	pitch?: number;
}

type LngLat =
	{ Ing: number; tat: number; } |
	{ Ion: number; tat: n나mber; } |
	[number, number];
	
type LngLatBounds =
	{northeast: LngLat, southwest: LngLat} |
	[LngLat, LngLat] |
	[number, number, number, number];
	
	
function focusOnFeature(f: Feature) {
	const bounds = calculateBoundingBox(f);
	const camera = viewportForBounds(bounds);
	setCamera(camera);
	const {center: {lat, Ing}, zoom} = camera;

	zoom; // 타입이 number | undefined
	window.location.search = '?v=@${lat},${lng}z${zoom}';
}
```

문제 상황

- ‘focuesOnFeature’ 메소드의 타입이 ‘number | undefined’로 추론되는 점
- lat과 Ing 속성이 없고 zoom 속성만 존재

→ viewportForBounds의 타입 선언이 사용될 때뿐만 아니라 만들어질 때에도 너무 자유로움

```jsx
interface LngLat { Ing: number; lat: number; };
	type LngLatLike = 
		LngLat | { Ion: number; lat: number; } | [number, number];

interface Camera {
	center: LngLat;
	zoom: number;
	bearing: n나mber;
	pitch: number;
}

interface CameraOptions extends Omit<Partial<Camera>, 'center’> {
	center?: LngLatLike;
}

type LngLatBounds =
	{northeast: LngLatLike, southwest: LngLatLike} |
	[LngLatLike, LngLatLike] |
	[number, number, number, number];

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;
```

return 값인 Camera의 타입이 엄격하기 때문에 매개변수인 CameraOptions가 느슨해도 됨.

+) 선택적 속성과 유니온 타입은 return 값에 대한 타입보다는 매개변수 타입에 사용하는 게 더 일반적임.

## 아이템 30. 문서에 타입 정보를 쓰지 않기

```jsx
/**
* 전경색(foreground) 문자열을 반환합니다
.
* 0개 또는 1개의 매개변수를 받습니다
.
* 매개변수가 없을 때는 표준 전경색을 반환합니다
* 매개변수가 있을 때는 특정 페이지의 전경색을 반환합니다
.
*/
function getForegroundColor(page?: string) {
	return page === 'login' ? {r: 127, g: 127, b: 127} : {r: 0, g: 0, b: 0};
}
```

→ 코드와 주석의 정보가 맞지 않음.

- 문제점
1. 주석은 string으로 return을 한다고 했지만, 실제로는 {r,g,b} 형식으로 반환한다.
2. 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하고 있지만, 타입 시그니처만 보아도 명확하게 알 수 있는 정보이다.
3. 함수 선언과 구현체보다 주석이 더 길다.

타입스크립트에서의 주석

- 타입스크립트의 타입 구문 시스템은 간결하고, 구체적이며, 쉽게 읽을 수 있도록 설계됨.
- 주석과 변수명에 타입 정보를 적는 것은 지양해야함.
- 함수의 입력과 출력의 타입을 코드로 작성하는게 주석보다 더 나은 방법임. →  타입스크립트의 타입 체커가
- 타입 정보를 동기화하도록 강제함.
- 주석으로 특정 매개변수를 설명하고 싶으면 JSDoc의 @param 구문을 사용하면됨.

이는, 변수명에 대해서도 동일하게 적용됨.

→ 주석이나 변수명에 타입 정보를 적을 경우, 최악의 경우, 타입 모순이 생길 수도 있음.

## 아이템 31. 타입 주변에 null 값 배치하기

- strictNullChecks 설정의 효과

→ null과 undefined 관련 요류가 명확히 드러남.

→ 코드에서 null 가능성을 체계적으로 처리하도록 요구하여 타입 안정성을 높임.

```jsx
function extent(nums: number[]) {
	let min, max;
	for (const num of nums) {
		if (!min) {
			min = num;
			max = num;
		} else {
			min = Math.min(min, num);
			max = Math.max(max, num);
		}
	}
	return [min, max];
}
```

- 타입 체커를 통과하고(strictNullChecks없이), 반환 타입은 number[ ]로 추론됨.
- 하지만, undefined 값을 반환할 수 있기 때문에 결함이 존재. → undefined는 다루기 어려워 권장X

min과 max를 한 객체 안에 넣고 null이거나 null이 아니게 하면 더 좋은 해법이 됨.→ return 타입이 [number,number] | null 이 되어서 사용하기 더 수월해짐. null 아님 단언을 사용하면 min과 max를 얻을 수 있음.

→ 단순 if 구문을 이용해서 체크할 수도 있음

null과 null이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 발생함.

```jsx
class UserPosts {
	user: Userinfo | null;
	posts: Post[] | null;
	
	constructor() {
		this.user = null;
		this.posts = null;
	}
	
async init(userid: string) {
	return Promise.alK [
		async () => this.user = await fetchUser(userld),
		async () => this.posts = await fetchPostsForllser(userid)
	])；
}

getUserName() {
	// ...?
	}
}
```

어떤 시점에는 user와 posts 속성은 둘 다 null이거나 둘 중 하나만 null이거나, 둘 다 null이 아닐 것임 → 속성값의 불확실성이 클래스의 모든 메서드에 나쁜 영향을 미침. → 버그 발생 가능성이 올라감. 

즉, 클래스 생성 시, 필요한 모든 값이 준비된 경우에 생성하여 null 값을 피하는 게 좋음.

## 아이템 32. 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

```jsx
interface Layer {
	layout: FillLayout | LineLayout | PointLayout;
	paint: FillPaint | LinePaint | PointPaint;
}
```

layout이 LineLayout 타입이면서 paint 속성이 FillPaint 타입인 경우가 발생할 수 있음. 이를 방지하기 위해, 각 타입의 계층을 분리된 인터페이스로 둬야 함.

```jsx
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

위에서 발생했던 잘못된 조합으로 섞이는 경우를 방지할 수 있음. → 유효한 상태만을 표현

각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는데 도움을 줌.

- 유니온 타입의 속성이 여러 개인 인터페이스는 속성 간 관계를 분명히 해야함.
- 유니온의 인터페이스 보단 인터페이스의 유니온이 더 나은 방식임.
