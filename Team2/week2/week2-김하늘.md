# Ch.2 - 타입스크립트의 타입 시스템 (1/2)

<aside>
💡학습목표

- 타입 시스템이란 무엇인지
- 타입 시스템은 어떻게 사용해야 하는지
- 타입시스템에서 무엇을 결정해야 하는지
- 타입시스템에서 가급적 사용하지 말아야 할 기능
</aside>

---

## item 6. 편집기를 사용하여 타입 시스템 탐색하기

### 추론 타입 확인하기

- 타입스크립트가 심벌의 타입을 어떻게 판단하고 있는지 확인
- 타입스크립트가 함수의 타입을 어떻게 판단하고 있는지 확인
- 조건문의 분기에서 값의 타입이 어떻게 변하는지 확인
- 객체에서 개별 속성의 추론 타입 확인

### 타입 오류 살펴보기

- id에 해당하거나 기본값인 HTMLElement를 반환하는 함수의 예시
    - 2개의 오류가 있는 코드
        
        ```tsx
        function getElement(elOrid: string | HTMLElement | null): HTMLElement {
          if (typeof elOrid === 'object') {
            return elOrid;
            // 오류: 'HTMLElement | null' 형식은 'HTMLElement' 형식에 할당할 수 없습니다.
          } else if (elOrid === null) {
            return document.body;
          } else {
            const el = document.getElementById(elOrid);
            return el;
            // 오류: 'HTMLElement | null' 형식은 'HTMLElement' 형식에 할당할 수 없습니다.
          }
        }
        ```
        
        첫번째 if문의 의도는 HTMLElement라는 객체를 골라내는 것이지만, 자바스크립트에서 typeof null은 "object"이므로 elOrld가 분기문 내에서 null일 가능성이 있다. 두 번째 오류는 document.getElementByld가null을 반환할 가능성이 있어서 발생한다.
        
    - 수정된 코드
        
        ```tsx
        function getElement(elOrid: string | HTMLElement | null): HTMLElement {
          if (elOrid === null) {
            return document.body; // elOrid가 null이면 document.body를 반환
          } else if (typeof elOrid === 'object') {
            return elOrid; // elOrid가 HTMLElement일 경우 그대로 반환
          } else {
            const el = document.getElementById(elOrid);
            if (el === null) {
              throw new Error(`Element with id ${elOrid} not found`);
            }
            return el; // el이 null이 아니라면 반환
          }
        ```
        
        첫번째 오류는 처음에 null 체크를 추가해서 바로잡는다. 두번째 오류 역시 null 체크를 추가하고 예외를 던져야 한다.
        

---

## item 7.  타입이 값들의 집합이라고 생각하기

### 타입체커의 관점에서 타입

- **타입**: 할당 가능한 값들의 집합
- **타입 체커의 역할**: 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것
- **‘할당 가능한’**: 집합의 관점에서 ‘~의 원소(값과 타입의 관계)’ 또는 ‘~의 부분 집합(두 타입 의 관계)’을 의미
    
    ```tsx
    type AB = 'A' | 'B';
    type AB12 = 'A' | 'B' | 12;
    
    const a: AB = 'A'; 
    	// 정상, ‘A1 는 집합 {'A1, 'B1}의 원소입니다.
    const c: AB = 'C;
    	// ~ '"C" 형식은 'AB' 형식에 할당할 수 없습니다.
    	
    const ab: AB = Math.random() < 0.5 ? 'A' : 'B';
    	//정상, {"A", "B"}는 {"A", "B"}의 부분 집합입니다.
    const abl2: AB12 = ab; 
    	// 정상, {"A", "B"}는 {"A", "B", 12}의 부분 집합입니다.
    	
    declare let twelve: AB12;
    const back: AB = twelve;
    	// — 'AB12' 형식은 'AB' 형식에 할당할 수 없습니다.
    	// '12' 형식은 'AB' 형식에 할당할 수 없습니다.
    ```
    

### Never & Unit(Literal) & Union 타입

- **Never**
아무 값도 포함하지 않는 공집합, 아무런 값도 할당할 수 없음
    
    ```tsx
    const x: never = 12;
    	// ~ '12' 형식은 'never' 형식에 할당할 수 없습니다.
    ```
    
- **Unit(Literal)**
한 가지 값만 포함하는 타입, 리터럴 타입으로 불리기도 함
    
    ```tsx
    type A = 'A';
    type B = 'B*;
    type Twelve = 12;
    ```
    
- **Union**
값 집합들의 합집합, 두 개 혹은 그 이상의 값을 포함하도록 묶인 타입
    
    ```tsx
    type AB = 'A' | 'B';
    type AB12 = 'A' | 'B' | 12;
    ```
    

### 인터페이스로 설명하는 타입

- 원소들을 일일이 추가해 타입을 만드는 방법
    
    ```tsx
    type Int = 1 | 2 | 3 | 4 | 5 // | ...
    ```
    
- 인터페이스로 요소를 서술하는 방법
    
    ```tsx
    interface Identified {
    id: string;
    }
    ```
    
    위 인터페이스가 타입 범위 내의 값들에 대한 설명이라고 생각해 보자. 어떤 객체가 string으로 할당 가능한 id 속성을 가지고 있다면 그 객체는 Identified 이다.
    

### 연산

타입은 '속성'이 아닌 '값의 범위'를 정의한다. 
즉 타입스크립트에서 타입을 정의할 때, 우리는 인터페이스의 속성(예: `name: string`)을 정의하지만, 실제로 타입은 그 속성들의 조합으로 표현되는 가능한 값의 집합을 정의하는 것이다.

```tsx
interface Person {
name: string;
}
interface Lifespan {
birth: Date;
death?: Date;
}
type PersonSpan = Person & Lifespan;
type PersonOrLifespan = Person | Lifespan;
```

- **& 연산자**
    - 두 타입의 인터섹션(intersection, 교집합)을 계산
    - Person과 Lifespan 인터페이스는 공통으로 가지는 속성이 없기 때문에 PersonSpan 타입을 공집합으로 예상하기 쉬우나, 속성의 교집합이 아니라 두 타입 모두를 만족하는 값의 범위를 의미
    - Person과 Lifespan을 둘 다 가지는 값은 인터섹션 타입
        
        ```tsx
        const ps: PersonSpan = {
        name: ’Alan Turing',
        birth: new Date('1912/06/23'),
        death: new Date(11954/06/07'),
        }； //
        정상
        ```
        
- **| 연산자**
    - 두 타입 중 하나에 속하는 값을 계산
    - `Person`이나 `Lifespan` 중 하나만 만족하는 값도 유니온 타입에 포함
        
        ```tsx
        const personOrLifespan1: PersonOrLifespan = { name: 'Alan Turing' }; // Person
        const personOrLifespan2: PersonOrLifespan = { birth: new Date('1912-06-23') }; // Lifespan
        ```
        
- **keyof**
    - 타입의 모든 속성 이름을 키의 집합으로 반환하는 연산자
        
        ```tsx
        type PersonKeys = keyof Person; // "name"
        type LifespanKeys = keyof Lifespan; // "birth" | "death"
        ```
        
    - `keyof (A & B)`: `A`와 `B`의 속성을 모두 포함하므로 `keyof A`와 `keyof B`의 유니온이 된다. 즉, 두 타입의 속성 키를 모두 사용할 수 있다.
        
        ```tsx
        type K1 = keyof (Person & Lifespan); // "name" | "birth" | "death"
        ```
        
    - `keyof (A | B)`: `A`와 `B` 중 하나만 만족하면 되므로, 공통된 속성만 가져오게 된다. 공통 속성이 없으면 `never`이 된다.
        
        ```tsx
        type K2 = keyof (Person | Lifespan); // never
        ```
        

---

## item 8. 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌（symbol）은 타입 공간이나 값 공간 중의 한 곳에 존재한다

- 타입 공간의 심벌
    
    ```tsx
    interface Cylinder {
    radius: number;
    height: number;
    }
    ```
    
    - `interface Cylinder`에서 `Cylinder`는 타입으로 쓰인다.
- 값 공간의 심벌
    
    ```tsx
    const Cylinder = (radius: number, height: number) => ({「adius, height});
    ```
    
    - `const Cylinder`에서 `Cylinder`는 타입으로 쓰인다.
- 가능한 오류
    
    ```tsx
    function calculateVolume(shape: unknown) {
    	if (shape instanceof Cylinder) {
    		shape.radius
    			// '{}' 형식에 'radius' 속성이 없습니다.
    	}
    }
    ```
    
    - instanceof를 이용해 shape가 Cylinder 타입인지 체크 시도
    - instanceof는 자바스크립트의 런타임 연산자이고,값에 대해서 연산 수행
    - instanceof Cylinder는 타입이 아니라 자바스크립트의 런타임 값인 `const Cylinder`에 대해 동작

---

## item 9. 타입 단언보다는 타입 선언을 사용하기

### 변수에 값을 할당하고 타입을 부여하는 방법

1. **변수 선언 시 타입 명시하기**
    - 변수를 선언할 때 타입을 명시적으로 지정하는 방식
    - 타입스크립트가 이 변수에 할당된 값이 선언된 타입과 일치하는지 검사
        
        ```tsx
        interface Person { 
          name: string; 
        }
        
        const alice: Person = { name: 'Alice' }; // 타입은 Person
        ```
        
2. **타입 단언 사용하기**
    - 타입 단언(Type Assertion)을 사용하여, 타입스크립트에게 “이 값이 `Person` 타입이라고 믿어라"라고 명령하는 방식
    - 타입스크립트가 추론한
    타입이 있더라도 Person 타입으로 간주
        
        ```tsx
        const bob = { name: 'Bob' } as Person; // 타입은 Person
        ```
        

### **타입 단언보다 타입 선언을 사용하기**

- 할당되는 값이 해당 인터페이스를 만족하는지 검사
    
    ```tsx
    const alice: Person = {};
    // 'Person' 유형에 필요한 'name' 속성이 '{}' 유형에 없습니다.
    const bob = {} as Person; // 오류 없음
    ```
    
- 잉여 속성 체크 동작
    
    ```tsx
    const alice: Person = {
    	name: 'Alice',
    	occupation: 'TypeScript developer'
    	// 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Person' 형식에 'occupation'이（가） 없습니다.
    }；
    const bob = {
    	name: 'Bob',
    	occupation: 'JavaScript developer'
    } as Person; // 오류 없음
    ```
    

---

## item 10. 객체 래퍼 타입 피하기

### 기본형 타입과 객체 래퍼 타입

- 기본형 타입
    - 불변(immutable)이며, 메서드를 가지지 않는다는 점에서 객체와 구분
    - `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
- 객체 래퍼 타입
    - 메서드를 가지는 기본형 타입의 ‘객체’ 타입이 정의
    - 기본형 `string` → 객체 타입 `String`
    - 기본형 `number` → 객체 타입 `Number`
    - 기본형 `boolean` → 객체 타입 `Boolean`
    - 기본형 `symbol` → 객체 타입 `Symbol`
    - 기본형 `bigint` → 객체 타입 `BigInt`

### 객체 래퍼 타입과 기본형의 차이

- 기본형 값과 객체 래퍼 타입은 동일하게 동작하지 않는다.
- `new String("hello") === "hello"`는 false
- `new String()`으로 생성한 것은 객체, `"hello"`는 기본형 값
- 객체 래퍼 타입은 자신과만 동일하다는 점이 다르다. 즉, 두 `new String("hello")` 객체도 서로 다른 객체로 취급되어 동일하지 않다.

### 타입스크립트에서 기본형과 객체 래퍼 타입

- 타입스크립트는 기본형과 객체 래퍼 타입을 구분
- string은 String에 할당할 수 있지만, String은 string에 할당할 수 없다.
    
    ```tsx
    function isGreeting(phrase: String) {
      return ['hello', 'good day'].includes(phrase); // 오류 발생
    }
    ```
    

---

## item 11. 잉여 속성 체크의 한계 인지하기

### 잉여 속성 체크

- 잉여 속성 체크가 있는 경우
    - 객체 리터럴을 타입이 명시된 변수에 할당할 때 타입스크립트는 그 객체가 타입에 맞는 속성들만 포함하는지 확인하는데, 이때 잉여 속성 체크를 통해 정의되지 않은 속성이 포함된 경우 오류를 발생
        
        ```tsx
        interface Room {
          numDoors: number;
          ceilingHeightFt: number;
        }
        
        const r: Room = {
          numDoors: 1,
          ceilingHeightFt: 10,
          elephant: 'present',  // 오류 발생: Room에 'elephant' 속성이 없음
        };
        ```
        
- 잉여 속성 체크가 없는 경우
    - 객체 리터럴을 바로 할당할 때는 잉여 속성 체크가 수행되지만, **임시 변수**를 사용하면 잉여 속성 체크를 건너뛰게 된다. 즉, 할당 가능한 타입이면 잉여 속성도 허용된다.
        
        ```tsx
        const obj = {
          numDoors: 1,
          ceilingHeightFt: 10,
          elephant: 'present',  // 객체에 존재하는 속성
        };
        
        const r: Room = obj;  // 정상
        ```
        

### 잉여 속성 체크의 한계

- 객체 리터럴에만 적용
    - 이미 정의된 변수나 타입 단언문을 사용하면 이 체크가 동작하지 않는다.
        
        ```tsx
        const o: Options = { darkmode: true, title: 'Ski Free' };  // 오류 발생
        
        const intermediate = { darkmode: true, title: 'Ski Free' };
        const o: Options = intermediate;  // 정상 (잉여 속성 체크가 생략됨)
        ```
        
- 타입 단언 사용 시 잉여 속성 체크 우회
    - 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 앞에서다룬 Room이나 Options 예제 같은 문제점을 방지할 수  있다.
    - 단언문보다 선언문을 사용해야 하는 단적인 이유 중 하나
    
    ```tsx
    const o = { darkmode: true, title: 'Ski Free' } as Options;  // 정상
    ```
    

---

## item 12. 함수 표현식에 타입 적용하기

### 함수 문장과 함수 표현식

자바스크립트에서는 함수 ‘문장(statement)’과 함수 ‘표현식(expression)’을 다르게 인식

- **함수 문장(Function Statement)**
    - `function` 키워드를 사용하여 선언된 함수. 선언과 동시에 실행 가능.
        
        ```tsx
        function rollDice(sides: number): number {
          return Math.floor(Math.random() * sides) + 1;
        }
        ```
        
- **함수 표현식(Function Expression)**
    - 변수에 함수를 할당하는 형태
        
        ```tsx
        const rollDice = function (sides: number): number {
          return Math.floor(Math.random() * sides) + 1;
        };
        
        const rollDice = (sides: number): number => Math.floor(Math.random() * sides) + 1;
        ```
        
    - 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문에 타입스크립트에서는 함수 표현식을 사용하는 것이 좋다.
        
        ```tsx
        type DiceRollFn = (sides: number) => number;
        const rollDice: DiceRollFn = sides =>{/*•••*/};
        ```
        

### 함수 표현식에 타입을 적용하는 이유

- **불필요한 코드의 반복 줄임**
    - 여러 개의 함수가 동일한 시그니처를 가질 때, 타입을 반복해서 작성하지 않고 타입을 하나의 타입으로 선언해서 재사용 가능하다.
        
        ```tsx
        function add(a: number,b: number) { return a + b; }
        function sub(a: number,b: number) { return a - b; }
        function mul(a: number,b: number) { return a * b; }
        function div(a: number,b: number) { return a / b; }
        ```
        
        ```tsx
        type BinaryFn = (a: number, b: number) => number;
        
        const add: BinaryFn = (a, b) => a + b;
        const sub: BinaryFn = (a, b) => a - b;
        const mul: BinaryFn = (a, b) => a * b;
        const div: BinaryFn = (a, b) => a / b;
        ```
        
- **typeof를 사용한 함수 타입 추론**
    - `typeof` 키워드를 사용하여 기존 함수의 타입을 가져올 수 있다. 이를 통해 기존 함수와 동일한 시그니처를 가진 새 함수를 선언할 수 있다.
    - `typeof fetch`를 사용하여 `checkedFetch`가 `fetch` 함수와 동일한 타입 시그니처를 갖도록 지정한다. 즉, `input`과 `init`의 타입은 `fetch`와 동일하게 자동 추론된다. 이 방식은 불필요하게 타입을 반복 선언하지 않고, 타입스크립트가 알아서 타입을 추론하게 함으로써 코드의 간결성과 안전성을 높인다.
        
        ```tsx
        declare function fetch(
        	input: Requestinfo, init?: Requestlnit
        ): Promise<Response>;
        
        //function statement
        async function checkedFetch(input: RequestInfo, init?: RequestInit) {
          const response = await fetch(input, init);
          if (!response.ok) {
            throw new Error('Request failed: ' + response.status);
          }
          return response;
        }
        
        //function expression
        const checkedFetch: typeof fetch = async (input, init) => {
          const response = await fetch(input, init);
          if (!response.ok) {
            throw new Error('Request failed: ' + response.status);
          }
          return response;
        };
        ```