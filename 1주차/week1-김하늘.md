# [Effective TypeScript] Ch.1 - 타입스크립트 알아보기

## item 1. 타입스크립트와 자바스크립트의 관계 이해하기

### TypeScript

- MicroSoft에서 개발한 JavaScript의 슈퍼셋 언어
- 확장자로 .ts(.tsx)를 사용하며, 컴파일 결과로 JavaScript 코드를 출력하고 실행 또한 JavaScript로 이루어짐
- 런타임에 오류를 발생시킬 코드를 미리 찾아냄으로써 JavaScript의 타입 불안정성을 해결하는 것

### JavaScript와 TypeScript의 관계

- 모든 자바스크립트 프로그램은 타입스크랩트 프로그램이지만 어떤 타입스크랩트 프로그램은 자바스크랩트 프로그램이 아니다.
- 타입 구문을 사용하는 순간부터 자바스크립트는 타입스크립트 영역
    
    ```tsx
    function greet(who: string) {
    console.log('Hello', who);
    }
    ```
    
    위와 같이 타입을 명시하는 타입스크립트 문법을 사용한 코드를 자바스크립트 엔진(ex:node.js)로 구동하면 다음와 같은 syntax error가 발생한다. 
    *function greet(who: string) {								
                                    ^
    SyntaxError: Unexpected token :*
    
- 타입 스크립트 컴파일러는 일반 자바스크립트 프로그램에도 유용
    
    ```jsx
    let city = 'new york city';
    console, log (city. tollppercase());
    ```
    
    위 코드에는 타입 구문이 없지만, 타입스크립트의 타입 체커는 다음과 같이 문제점을 찾아낼 수 있다. 
    *let city = 'new york city';
    console. log(city.tollppercase());
    // 'toUppercase' 속성이 'string’ 형식에 없습니다.
    // ‘ tollppe rCase1 을(를) 사용하시겠습니까?*
    city 변수가 문자열이라는 것을 알려 주지 않아도 타입스크립트는 초깃값으로 부터 타입을 추론할 수 있는 것이다.
    

## item 2. 타입스크립트 설정 이해하기

### 설정

- 타입스크립트의 설정들은 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분
- 언어 자체의 핵심 요소들을 제어하는 설정도 존재
- 타입스크립트 설정은 커맨드 라인을 이용하기보다는 tsconfig.json을 사용하기

### noImplicitAny

- 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
- noImplicitAny 해제
    
    ```tsx
    function add(a, b) {
    return a + b;
    }
    ```
    
    - 위 코드는 noImplicitAny가 해제되어 있을 때에는 유효하다. 이 때 타입스크립트는 매개변수의 타입을 추론하며, 타입체커는 다음과 같은 피드백을 제공한다. any를 코드에 넣지 않았지만 any 타입으로 간주되기 때문에 이를 ‘암시적 any’라고 부른다.
    *function add(a, b) {
    // ~ 'a' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
    // ~ 'b' 매개변수에는 암시적으로 'any' 형식이 포함됩니다.*
    - any 타입을 매개변수에 사용하면 타입체커는 무력해진다.
- noImplicitAny 설정
    
    ```tsx
    function add(a: number, b: number) {
    return a + b;
    }
    ```
    
    - noImplicitAny를 설정한 상태에서 오류를 발생시키지 않으려면 위와 같이 명시적으로 타입을 사용함으로써 해결한다.
    - 타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 타입을 명시한다.
    - 새 프로젝트를 시작한다면 처음부터 noImplicitAny를 설정하여 코드를 작성할 때마다 타입을 명시하도록 해야 하며, 그러면 타입스크립트가 문제를 발견하기 수월해지고 코드의 가독성이 좋아지며 개발자의 생산성이 향상될 수 있다.

### strictNullChecks

- null과 undefined가 모든 타입에서 허용되는지 확인하는설정
- strictNullChecks 해제
    
    ```tsx
    const x: number = null; //정상, null은 유효한 값입니다.
    ```
    
    - 위 코드는 strictNullChecks가 해제되어 있을 때에는 유효하다.
- strictNullChecks 설정
    
    ```tsx
    const x: number | null = null;
    ```
    
    - 만약 null을 허용하려고 한다면, 위와 같이 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.
    - strictNullChecks는 null과 undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 한다.
    - strictNullChecks를 설정하려면 noImplicitAny를 먼저 설정해야 한다.

## item 3. 코드 생성과 타입이 관계없음을 이해하기

1. 최신 타입스크립트/자바스크립트를 브라우저 에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일
2. 코드의 타입 오류를 체크

타입스크립트 컴파일러가 수행하는 두 가지 역할을 되짚어 보면 타입스크립트가 할 수 있는 일과 할 수 없는 일을 짐작할 수 있다.

### 타입 오류가 있는 코드의 컴파일 가능

- 컴파일은 타입 체크와 독립적으로 동작하기 때문에 타입 오류가 있는 코드도 컴파일이 가능하다.
- 타입스크립트 오류는 C나 자바 같은 언어들의 경고(warning)와 비슷하다. 문제가 될 만한 부분을
알려 주지만 그렇다고 빌드를 멈추지는 않는다.

### 런타임 중 타입 체크 불가

```tsx
interface Square {
width: number;
}
interface Rectangle extends Square {
height: number;
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
if (shape instanceof Rectangle) {
										// 'Rectangle'은(는) 형식만 참조하지만,
										// 여기서는 값으로 사용되고 있습니다.
return shape.width * shape.height;
										// ’Shape' 형식에 'height' 속성이 없습니다.
  } else {
    return shape.width * shape.width;
  }
}
```

- instanceof 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다.
- 런타임에 타입 정보를 유지하는 방법이 필요
    - height 속성이 존재하는지 체크
        
        ```tsx
        function calculateArea(shape: Shape) {
          if ('height' in shape) {
            shape; // 타입이 Rectangle
            return shape.width * shape.height;
          } else {
            shape; // 타입이 Square
            return shape.width * shape.width;
          }
        }
        ```
        
    - 태그 기법
        
        ```tsx
        interface Square {
        	kind: 'square';
        	width: number;
        }
        interface Rectangle {
        	kind: ‘rectangle’;
        	height: number;
        	width: number;
        }
        type Shape = Square | Rectangle;
        function calculateArea(shape: Shape) {
        	if (shape.kind === 1 rectangle') {
        		shape; // 타입이 Rectangle
        		return shape.width * shape.height;
        	} else {
        		shape; // 타입이 Square
        		return shape.width * shape.width;
        	}
        }
        ```
        
    - 타입을 클래스로 만들기
        
        ```tsx
        class Square {
        	constr나ctor(public width: number) {}
        }
        class Rectangle extends Square {
        	constructor(public width: number, public height: number) {
        	super(width);
        	}
        }
        type Shape = Square | Rectangle;
        function calculateArea(shape: Shape) {
        	if (shape instanceof Rectangle) {
        		shape; // 타입이 Rectangle
        		return shape.width * shape.height;
        	} 
        	else {
        		shape; // 타입이 Square
        		return shape.width * shape.width; // 정상
        	}
        }
        ```
        

## item 4. 구조적 타이핑에 익숙해지기

타입스크립트는 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 자바스크립트의 동작을 그대로 모델링한다. 그런데, 타입 체커의 타입에 대한 이해도가 사람과 조금 다르기 때문에 가끔 예상치 못한 결과가 나오기도 한다.

### 2D 벡터 타입을 다루는 경우

2D 벡터 인터페이스와 벡터의 길이를 계산하는 함수는 다음과 같다.

```tsx
interface Vector2D {
	x: number;
	y: number;
}

interface NamedVector {
name: string;
x: number;
y: number;
}

function calculateLength(v: Vector2D) {
	return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

NamedVector는 number 타입의 x와 y 속성이 있기 때문에, 즉 NamedVector의 구조가 Vector2D와 호환되기 때문에 calculateLength 호출이 가능하다. NamedVector를 위한 별도의 calculateLength를 구현할 필요가 없는 것이다. 이를 구조적 타이핑이라고 한다.

```tsx
const v: NamedVector = { x: 3, y: 4, name: 'Zee1 };
calculateLength(v); // 정상,결과는 5
```

### 3D 벡터 타입을 다루는 경우

3D 벡터와 calculateLenth()를 사용해 벡터의 길이를 정규화하는 함수를 만들어보자.

```tsx
interface Vector3D {
	x: number;
	y: number;
	z: number;
}

function normalize(v: Vector3D) {
	const length = calculateLength(v);
	return {
	x: v.x / length,
	**y: v.y / length,
	z: v.z / length,
	};
}
```

{x: 3, y: 4, z: 5}에 대해 normalize 함수를 사용하면 1보다 긴 길이를 출력한다. calculateLength는 2D 벡터를 기반으로 연산하는데 normalize가 3D 벡터로 연산되는 과정에서 z가 정규화에서 무시된 것이다. calculateLength가 2D벡터를 받도록 선언되었음에도 불구하고 3D 벡터를 받는 데 문제가 없었던 이유는 무엇일까? Vector3D와 호환되는 {x, y, z} 객체로 calculateLength를 호출하면  구조적 타이핑 관점에서 x와 y가 있어서 Vector2D와 호환되기 때문이다.

```tsx
normalize({x: 3, y: 4, z: 5});
 // 결과 { x: 0.6, y: 0.8, z: 1 }, 1보다 큰 값 반환
```

## item 5. any 타입 지양하기

일부 특별한 경우를 제외하고는 any를 사용하면 타입스크립트의 수많은 장점을 누릴 수 없게 되므로, 부득이하게 any를 사용하더라도 그 위험성을 알고 있어야 한다. 다음 예시를 가지고 any 타입의 위험성을 알아본다.

```tsx
let age: number;
age = '12'; // ~ '"12"' 형식은 ’number' 형식에 할당할 수 없습니다.

age = '12' as any; // OK
```

### 타입 안정성 부재

age는 number 타입으로 선언되었지만 as any를 사용하면 string 타입을 할당할 수 있게 된다. 타입 체커는 선언에 따라 number타입으로 판단할 것이다. 다음 예시 작업을 수행한다고 가정하자.  코드는 런타임에 정상적으로 작동하지만 실행결과 age는 "121"로 의도된 것과 다르게 작동할 것이다.

```tsx
age += 1;
```

### 함수 시그니처 무시

함수를 작성할 때는 시그니처를 명시해야 한다. 호출하는 쪽은 약속된 타입의 입력을 제공하고 함수는 약속된 타입의 출력을 반환한다. 그러나 any 타입을 사용하면 이런 약속을 어길 수 있다.

```tsx
function calculateAge(birthDate: Date): number {
// ...
}
let birthDate: any = '1990—01-19';
calculateAge(birthDate); // 정상
```

### 코드 리팩터링 때 버그 감춤

```tsx
interface ComponentProps {
	onSelectltem: (id: number) => void;
}

function renderselector(props: ComponentProps) {/*...*/}

let selectedld: number = 0;

function handleSelectItem(item: any) {
	selectedld = item.id;
}

renderSelector({onSelectltem: handleSelectltem});
```

이 경우 타입 체크를 모두 통과한다. `handleSelectItem`()에서  `item`의 타입이 모든 타입을 허용하는 `any`이기 때문에 `item.id`에 접근할 때 타입 체크에서는 문제가 발생하지 않기 때문이다. 하지만 실행 시 id를 전달
받으면, 타입 체커를 통과함에도 불구하고 런타임에는 오류가 발생할 것이다. `id`는 숫자일 뿐 객체가 아니기 때문이다. any가 아니라 구체적인 타입을 사용했다면, 타입 체커가 오류를 발견했을 것이다.

### 타입시스템의 신뢰도 하락
- 보통 타입 체커가 실수를 잡아주고 코드의 신뢰도를 향상시킨다.
- 그러나 런타임에 타입 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없고, 대규모 팀에 타입스크립트를 도입하려는 상황에서 타입 체커를 신뢰할 수 없다면 큰 문제가 될 수 있다.
- any 타입을 쓰지 않으면 런타임 에 발견될 오류를 미 리 잡을 수 있고 신뢰도를 높일 수 있다.

