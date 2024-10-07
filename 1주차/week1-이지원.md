# TypeScript Study
> Week 1, <이펙티브 타입스크립트> 북스터디

## 1장 타입스크립트 알아보기

- 포함 관계: JS ⊂ TS

### 간단히 비교하기
| | JS | TS |
|---|---|---|
| 확장자 | .js .jsx | .ts .tsx |
| Type System | 동적(Dynamic) | 정적(Static) |
| 컴파일 과정 | X | O |
|  |  |  | 

### TypeScript 설정 관련
` $ tsc --{settings} {filename}`  
or `tsconfig.json` file 수정을 통해 설정 가능  

- `noImplicitAny` 타입이 정의되지 않은 변수(Any로 판단되는 것)가 있다면 오류로 표현. 모든 경우에 타입을 명시하도록 하는 역할
- `strictNullChecks` null, undefined 값을 **가질 수 있는 타입**만이 해당 값을 가지도록 허용

## TypeScript 타입과 컴파일러
### 컴파일러가 하는 일
- 최신 JS/TS가 브라우저에서 동작하도록 __구버전의 JS로 바꾼다__ (원 문장: Transpile한다)
- 타입 오류를 체크한다  
- TS는 타입 오류가 있는 코드라도 컴파일이 가능하다. 원하지 않을 경우 `noEmitOnError` 설정으로 조정 가능하다.
> 작성된 코드를 구버전의 JS로 바꾸는 과정은 컴파일 과정을 한번 더 거쳐야 하는 코드이다.

### 타입
- 런타임에는 타입 체크가 불가능하다.
- 타입은 TS -> JS로 변환되는 시점에 제거된다.
- 타입 연산은 타입 체커를 통과하더라도 런타임에 영향을 주지 않는다.
- 함수 overloading 불가능
- 런타임 성능과는 무관하다.


### 예시
아래 코드는 런타임 오류를 발생시키는 코드이다. 
```ts
interface Square { width: number; }
interface Rectangle extends Square { height: number; }
type Shape = Square | Rectangle; // 

function Area(shape: Shape) {
    if (shape instanceof Rectangle) { return shape.width * shape.height; }
    else { return shape.width * shape.width }
}
```
- `instanceof` 체크는 런타임에 일어난다.
- 그러나 `Rectangle`은 type이기 때문에 런타임 시점에 타입 정보가 유지되지 않는다.
> `interface`는 type으로 사용된다.

### 그럼 어떻게 타입 정보를 유지하나요
[^0]
- 위의 예시에서는 Area() 함수에서 `if ('height' in shape)`를 통해 shape 타입에 height라는 property가 있는지 체크한다.[^1]
- tagged union 사용하기
- 타입을 클래스로 만들어 **값**이면서 동시에 **타입**이게 하기  


## 구조적 타이핑
- duck typing: 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다.[^2]

```ts
interface Vector2D {
    x: number;
    y: number;
}

function calLen(v: Vector2D) {
    return Math.sqrt(v.x * v.x + v.y * v.y)
}

interface NamedVec {
    name: String;
    x: number;
    y: number;
}

const v: NamedVec = {x: 3, y: 4, name: 'Zee'}
```
- 위 예시에서 `NamedVec`은 "구조적으로" `Vector2D`와 일치하기 때문에 `Vector2D`를 인자로 받는 `calLen()` 함수에 넘겨줄 수 있다.
- 추가적인 `name`이라는 property가 있지만, `Vector2D`가 요구하는 `x`, `y`가 있기 때문에 가능한 것

### 구조적 타이핑이 문제가 되는 경우
```ts
function normalize(v: Vector3D) {
    const len = calLen(v);
    return {
        x: v.x/len,
        y: v.y/len,
        z: v.z/len,
    };
}
```
- 위 예시에서 `calLen()`은 `Vector2D`type을 인자로 받지만, `Vector3D`를 넘겨도 오류가 발생하지 않는다.
- 이는 타입스크립트의 타입은 열려 있기 때문이다.

```ts
function calculateLengthLl(v: Vector3D) { 
    let length = 0;
    for (const axis of Object.keys(v)) { 
    const coord = v[axis]; // Error
    ...
```
- axis: x, y, z
- `v`는 "어떤 속성이든 가질 수 있다"
- 그렇기 때문에 `Vector3D`가 무조건 `v`라고 확정할 수 없고, 그렇기 때문에 암시적으로 any 타입일 것이라고 설명한다.

## any 타입 지양
### 왜?
- any 타입은 "타입 안전성이 없다"
- IDE 등에서 자동 완성 기능이 지원되지 않는다
- 타입 체커가 발견하지 못하지만 런타임에는 오류가 발생하는 경우를 만들 수 있다.
- 상태와 같은 복잡한 객체를 정의할 때, 의도된 설계를 알기 어려워질 수 있다.
<br></br>

# Summary
- 기본적으로 변수를 선언할 떄 type을 같이 명시하는 습관을 들이자.
- Typescript의 type은 런타임에 제거되므로, 런타임에도 타입 체크가 필요하다면 다른 방법들을 사용해서 타입 정보를 유지하도록 만들어야 한다.
- 꼭 필요한 경우가 아니라면 any type은 사용하지 말자.

## 잘 이해되지 않는 부분
[^0] 해당 이야기는 **런타임에 타입을 유지해야 하는 경우**에만 해당하는 건지?

[^1] 이게 결국 런타임엔 값을 체크하기 때문에, Rectangle이라는 값에 height라는 값이 있는지 확인하는 과정인지?

[^2] 어떤 함수의 매개변수 값이 제대로 존재하면 타입이 무엇인지는 관계 없이 정상적으로 작동한다?