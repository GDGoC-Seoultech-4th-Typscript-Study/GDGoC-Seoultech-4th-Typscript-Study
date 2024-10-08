# 이펙티브 타입스크립트 1장


## 1 ) 타입스크립트와 자바스크립트의 관계

***“타입스크립트는 자바스크립트의 상위 집합”*** 

: 문법 오류가 없는 자바스크립트 프로그램은 유효한 타입스크립트 프로그램이다.


자바스크립트 파일 → . js ( or . jsx) 확장자 사용 vs 타입스크립트 파일 → . ts ( or . tsx ) 확장자 사용

- 타입스크립트는 초깃값으로부터 ‘***타입 추론***’ 을 하는데 , 이는 타입스크립트에서 중요한 부분이다.
- 타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다.
- 타입 스크립트는 오류가 발생하지는 않지만 의도와 다르게 동작하는 코드에서 몇 가지 문제를 찾아내기도 한다.

```jsx
const states = [
{name: 'Alabama1, capital: 'Montgomery1},
{name: 'Alaska', capital: 'Juneau'},
{name: 'Arizona'； capital: ‘Phoenix’},
// ...
]；
for (const state of states) {
console.log(state.capitol); // capital' 속성이 ... 형식에 없습니다.'capitol'을(를) 사용하시겠습니까?
/* 
undefined
undefined
undefined 
*/
```

타입을 명시적으로 선언하지 않을 경우, 잘못된 해결책을 제시

```jsx
interface State {
	name: string;
	capital: string;
}
const states: State[] = [
	{name: 'Alabama',capitol:'Montgomery'},
	{name: 'Alaska1', capitol:'Juneau'},
	{name: 'Arizona', capitol: 'Phoenix'},
] // capital' 속성이 ... 형식에 없습니다. 'capitol'을(를) 사용하시겠습니까?

for (const state of states) {
	console.log(state.capital);
}
```

타입을 명시적으로 선언한 경우, 올바른 해결책을 제시

---

## 2 ) 타입스크립트 설정

- 타입스크립트 설정하기
    - 타입스크립트 컴파일러는 매우 많은 설정을 가지고 있는데, 해당 설정을 할 수 있는 방법은 크게 2가지가 있음
    1. 커맨드 라인에서 사용
    
    ```bash
    $ tsc —noImplicitAny program.ts
    ```
    
    1. *tsconfig.json* 설정 파일 ( 권장 )
    
    ```jsx
    }
    	"compileroptions": {
    		"noImplicitAny": true
    	}
    }
    
    // *tsc --init* 을 통해 설정 파일 생성
    ```
    
- 언어 핵심 요소 제어 설정
    1. **noImplicitAny** 
        - 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
        - 타입스크립트를 효과적으로 사용하기 위해 설정 필요
    2. **strictNullChecks**
        - null과 undefined가 모든 타입에서 허용되는지 확인하는설정
        - null과 undefined 관련 오류를 잡기 위해 많은 도움이 됨

*+)  타입스크립트에서 엄격한 체크를 하고 싶다면 **strict** 설정을 고려해야 함*

---

## 3 ) 코드 생성과 타입이 관계없음

- 타입스크립트 컴파일러의 두 가지 역할
    - 최신 타입스크립트 / 자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 **트랜스파일**
    - 코드의 **타입 오류** **체크**

### 타입 오류가 있는 코드도 컴파일이 가능

- 컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일 가능

### 런타임에는 타입 체크 불가능

```jsx
interface Square {
	width: number;
}

interface Rectangle extends Square {
	height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		return shape.width * shape.height;
	} else {
		return shape.width * shape.width;
	}
}
```

- 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거됨
    - Rectangle은 타입이기 때문에 런타임 시점에 제거되기 때문에 shape의 타입을 명확히 할 수 없음 → 런타임에 타입을 유지하는 방법이 필요
        - height 속성이 존재하는 지 체크 → **속성 체크**는 런타임에 접근 가능한 값에만 관련, 타입 체커 역시도 shape의 타입을 Rectangle로 보정
            
            ```jsx
            function calculateArea(shape: Shape) {
            	if ('height' in shape) {
            		shape; 
            		return shape.width * shape.height;
            } else {
            		shape; 
            		return shape.width * shape.width;
            	}
            }
            ```
            
        - **태그 기법 :** 런타임에 접근 가능한 타입 정보를 명시적으로 저장
            
            ```jsx
            interface Square {
            kind: 'square';
            width: number;
            }
            
            interface Rectangle {
            kind: ‘rectangle’;
            height: number;
            width: number;
            }
            
            type Shape = Square | Rectangle; // ‘**태그된 유니온**(tagged union)’의 한 예
            
            function calculateArea(shape: Shape) {
            	if (shape.kind === 1 rectangle') {
            		shape; 
            		return shape.width * shape.height;
            	} else {
            		shape; 
            		return shape.width * shape.width;
            	}
            }
            ```
            
    - 타입을 클래스로 만들어 타입 (런타임 접근 불가)과 값 (런타임 접근 가능) 을 둘 다 사용 → Square과 Rectangle을 클래스로 만듦

### 타입 연산은 런타임에 영향을 주지 않음

as number : 타입을 number로 만드는 타입 연산을 위한 코드

```jsx
function asNumber(val: number | string): number {
	return val as number; // val이라는 문자열(or number)을 number 타입으로 변환
}
```

→ 위와 같은 타입 연산은 런타임에 영향X

### 타입스크립트 타입으로는 함수를 오버로드할 수 없음

- 하나의 함수에 대해 여러 개의 **선언문**을 작성할 수 있지만, **구현체**는 하나만 존재
    - **선언문**
        
        ```jsx
        function add(a: number, b: number): number;
        function add(a: string, b: string): string;
        ```
        
    - **구현체**
        
        ```jsx
        function add(a, b) {
        	return a + b;
        }
        ```
        

---

## 4 ) 구조적 타이핑

- 덕 타이핑 (duck typing) : 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식 → 자바스크립트

```jsx
interface Vector2D {
	x: number;
	y: n나mber;
}

function calculateLength(v: Vector2D) {
	return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
	name: string;
	x: number;
	y: number;
} 
```

NamedVector는 number 타입의 x와 y 속성이 있기 때문에 (Vector2D와 ***구조적으로*** 같음) calculateLength 함수로 호출 가능 → NamedVector를 위한 별도 calculateLength를 구현할 필요 없음.

---

## 5 ) any 타입

any 타입을 사용할 경우에 다양한 문제들이 발생할 수 있음

1. 타입 안정성 부족 → 변수의 타입이 number로 선언이 됐는데, string 값이 들어와 오류가 발생해도 이를 찾아내기 어려움
2. 함수 시그니처 무시 → 함수는 호출하는 쪽에서 약속된 타입의 입력값을 제공하고, 함수는 약속된 타입의 출력값을 반환해야 하는데 이를 무시하여 문제가 발생할 수 있음
3. 타입 설계를 감춤 → 객체 정의를 하려면 속성의 타입을 작성해야 하는데, any 타입을 사용하면 상태 객체 설계를 감춤

- any 타입을 사용하면 타입스크립트를 사용하는 목적을 없앨 수 있기 때문에 사용을 지양하는 게 좋음

---

### 이해가 어려운 부분

- 구조적 타이핑의 문제점 부분이 이해가 잘 되지 않음
