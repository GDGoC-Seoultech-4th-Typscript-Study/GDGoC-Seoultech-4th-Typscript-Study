# Ch.2 - 타입스크립트의 타입 시스템 (2/2)

>[!tip]
>
> - 타입 시스템이란 무엇인지
> - 타입 시스템은 어떻게 사용해야 하는지
> - 타입시스템에서 무엇을 결정해야 하는지
> - 타입시스템에서 가급적 사용하지 말아야 할 기능

---

## Item 13. 타입과 인터페이스의 차이점 알기

### named type을 정의하는 방법

- 타입을 사용하는 법과 인터페이스를 사용하는 법 두 가지가 있으며, 대부분의 경우 무엇을 사용해도 무관
- 타입과 인터페이스의 차이를 알고 같은 상황에서는 동일한 방법을 사용해 일관성을 유지할 필요
- 하나의 타입에 대해 두 가지 방법을 모두 사용해서 정의할 줄 알아야 함

1. **타입으로 정의**
    
    ```tsx
    type TState = {
    	name: string;
    	capital: string;
    }
    ```
    
2. **인터페이스로 정의**
    
    ```tsx
    interface IState {
    	name: string;
    	capital: string;
    }
    ```
    

### 타입과 인터페이스의 공통점

- 할당 검사
    - 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다.
    - 여기서 “상태”는 "객체의 구조나 속성 값”을 의미하며, 즉 `IState`와 `TState`로 정의한 객체의 속성들,  `name`과 `capital` 같은 프로퍼티가 동일한 방식으로 동작한다는 것을 말한다.
    - 예를 들어, `IState`나 `TState`로 정의된 객체에 추가 속성(`population` 같은)을 넣으면 동일하게 오류가 발생한다.
        
        ```tsx
        const Wyoming: TState = {
          name: 'Wyoming',
          capital: 'Cheyenne',
          population: 500_000
        	  // ... 형식은 'TState' 형식에 할당할 수 없습니다.
        		// 개체 리터럴은 알려진 속성만 지정할 수 있으며
        		// ‘TState' 형식에 'population'이(가) 없습니다.
        }
        ```
        
- 인덱스 시그니처
    - 인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다.
        
        ```tsx
        type TDict = { [key: string]: string };
        interface IDict { [key: string]: string; }
        ```
        
- 함수타입
    - 함수 타입도 인터페이스나 타입으로 정의할 수 있다.
        
        ```tsx
        type TFn = (x: number) => string;
        interface IFn { (x: number): string; }
        
        const toStrT: TFn = x => 'Number ' + x;
        const toStrI: IFn = x => 'Number ' + x;
        ```
        
- 제너릭 지원
    - 타입 별칭과 인터페이스는 모두 제너릭을 지원한다.
    - 제너릭을 사용하면 타입을 동적으로 설정할 수 있다.
        
        ```tsx
        type TPair<T> = {
        	first: T;
        	second: T;
        }
        interface IPair<T> {
        	first: T;
        	second: T;
        }
        ```
        

### 타입과 인터페이스의 차이점

- **타입의 유니온 지원**
    - 타입은 유니온 타입을 표현할 수 있지만, 인터페이스는 할 수 없다.
        
        ```tsx
        type AorB = 'a' | 'b';
        ```
        
    - 인터페이스는 상속을 통해 다른 인터페이스를 확장할 수 있지만, 유니온 타입은 확장할 수 없다.
        
        ```tsx
        interface Base {
            prop1: string;
        }
        
        interface Extended extends Base {
            prop2: number;
        }
        ```
        
        ```tsx
        type C = AorB | 'c'; // 오류: AorB는 더 이상 확장할 수 없음
        ```
        
    - 유니온 타입 확장이 필요할 때는 유니온 타입을 직접 확장하는 대신, 다른 타입과 조합하거나 추가 속성을 부여하는 방법을 사용할 수 있다.
        - 두 개의 데이터구조를 정의하고 하나의 인터페이스에서 매핑
            
            ```tsx
            type Input = { id: number; value: string }; // 입력 데이터 구조
            type Output = { result: string; error?: string }; // 출력 데이터 구조
            
            interface VariableMap {
                [name: string]: Input | Output; // name은 문자열이며 Input 또는 Output 타입
            }
            
            // VariableMap의 예시 사용
            const myMap: VariableMap = {
                userInput: { id: 1, value: "Hello" }, // Input 타입
                processResult: { result: "Success" }, // Output 타입
                errorResult: { result: "Failure", error: "Some error occurred" } // Output 타입
            };
            
            // 접근 예시
            const input = myMap['userInput']; // Input 타입
            const output = myMap['processResult']; // Output 타입
            ```
            
        - 유니온 타입의 객체에 다른 속성을 추가
            
            ```tsx
            type NamedVariable = (Input | Output) & { name: string };
            ```
            
- **타입의 튜플 지원**
    - 타입은 튜플과 배열을 더 간결하게 표현할 수 있다.
        
        ```tsx
        type Pair = [number, number];
        type StringList = string[];
        type NamedNums = [string, ...number[]];
        ```
        
    - 인터페이스로 비슷하게 구현할 수 있지만, 튜플에서 사용할 수 있는 concat 같은 메서드를 지원하지 않는다. 그러므로 튜플은 type 키워드로 구현하는 것이 낫다.
        
        ```tsx
        interface Tuple {
          0: number;
          1: number;
          length: 2;
        }
        
        const t: Tuple = [10, 20];  // 정상
        ```
        
- **인터페이스의 선언병합 지원**
    - 인터페이스는 선언병합(속성을 확장하는 것)을 지원하지만, 타입은 그렇지 않다.
    - 이번 아이템 처음에 등장했던 State 예제에 population 필드를 추가할 때 보강 기법을 사용할 수 있다.
        
        ```tsx
        interface IState {
          name: string;
          capital: string;
        }
        
        interface IState {
          population: number;
        }
        
        const Wyoming: IState = {
          name: 'Wyoming',
          capital: 'Cheyenne',
          population: 500_000
        };
        ```
        
    - 선언 병합은 주로 타입 선언 파일에서 사용되며, API에서 사용자가 새로운 필드를 추가할 수 있도록 유연성을 제공한다. 따라서 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야 한다.

### 타입과 인터페이스의 결정

- **복잡한 타입일 경우:** 타입 별칭(`type`)을 사용하는 것이 좋다.
- **선언 병합이 필요할 경우:** 인터페이스를 사용하는 것이 적합하다.
- **프로젝트의 일관성:** 이미 코드베이스에 일관되게 사용되고 있는 방법이 있다면, 그 방법을 따라야 한다.

---

## Item 14.  타입 연산과 제너릭 사용으로 반복 줄이기

### DRY 원칙

- DRY(don’t repeat yourself): 같은 코드를 반복하지 말아야한다는 원칙
- 코드를 개선해 DRY 원칙을 지키는 예시
    - Before → 비슷한 코드의 반복
        
        ```tsx
        console.log('Cylinder 1 x 1 1,
        'Surface area:6.283185 *1*1+ 6.283185 *1*1,
        'Volume:3.14159 *1*1*1);
        console.log('Cylinder 1x2',
        'Surface area:6.283185 * 1 * 1 + 6.283185 * 2 * 1,
        'Volume:3.14159 *1*2*1);
        console.log('Cylinder 2x1',
        'Surface area:6.283185 *2*1+ 6.283185 *2*1,
        'Volume:3.14159 *2*2*1);
        ```
        
    - After → 함수, 상수,루프의 반복을 제거
        
        ```tsx
        const surfaceArea = (r, h) => 2 * Math.PI * r * (r + h);
        const volume = (r, h) => Math.PI * r * r * h;
        for (const [r, h] of [[1, 1], [1, 2]t [2, 1]]) {
        console.log(
        'Cylinder ${r} x ${h}',
        'Surface area: ${surfaceArea(r, h)}'f
        'Volume: ${volume(rf h)}');
        }
        ```
        

### 타입 중복 문제

- **타입 중복**
    - 이 두 인터페이스는 구조가 유사하지만, 중복된 속성이 존재한다.
        
        ```tsx
        interface Person {
          firstName: string;
          lastName: string;
        }
        
        interface PersonWithBirthDate {
          firstName: string;
          lastName: string;
          birth: Date;
        }
        ```
        
    - 중복된 타입은 코드의 일관성을 해치고 유지보수를 어렵게 만든다.

### **타입 중복을 제거하는 방법**

- **타입에 이름 붙이기**
    - before → 함수의 입력값의 타입이 중복
        
        ```tsx
        function distance(a: {x: number, y: number}, b: {x: number, y: number}) {
          return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
        }
        ```
        
    - after → 입력값의 타입에 이름을 붙여 중복 제거
        
        ```tsx
        interface Point2D {
          x: number;
          y: number;
        }
        
        function distance(a: Point2D, b: Point2D) { /*...*/ }
        ```
        
- **인터페이스 확장**
    - before → 두 타입이 중복된 속성을 가짐
        
        ```tsx
        interface Person {
          firstName: string;
          lastName: string;
        }
        
        interface PersonWithBirthDate {
          firstName: string;
          lastName: string;
          birth: Date;
        }
        ```
        
    - after → 인터페이스를 확장해 속성을 추가함으로써 중복 제거
        
        ```tsx
        interface Person {
          firstName: string;
          lastName: string;
        }
        
        interface PersonWithBirthDate extends Person {
          birth: Date;
        }
        ```
        
- **명명된 타입으로 분리**
    - before → 두 함수가 같은 타입 시그니처를 공유
        
        ```tsx
        function get(url: string； opts: Options): Promise<Response그 {/*...*/}
        function post(url: string, opts: Options): Promise<Response그 {/*...*/}
        ```
        
    - after → 해당 시그니처를 명명된 타입으로 분리
        
        ```tsx
        type HTTPFunction = (url: string, opts: Options) =그 Promise<Response>;
        const get: HTTPFunction = (url, opts) => { /* ... */ };
        const post: HTTPFunction = (url, opts) =>{/*•■•*/};
        ```
        
- **매핑된 타입 이용**
    - before → 두 인터페이스가 공통된 속성을 공유
        
        ```tsx
        interface State {
        	userid: string;
        	pageTitle: string;
        	recentFiles: string[];
        	pageContents: string;
        }
        
        interface TopNavState {
        	userid: string;
        	pageTitle: string;
        	recentFiles: string[];
        }
        ```
        
    - after → 매핑된 타입을 사용하여 `State` 내의 속성이 변경되더라도 `TopNavState`에 자동으로 반영되므로 유지보수가 쉬워짐
        
        ```tsx
        type TopNavState = {
        	[k in 'userid' | 'pageTitle' | 'recentFiles']: State[k]
        }；
        ```
        

---

## Item 15. 동적 데이터에 인덱스 시그니처 사용하기

### 인덱스 시그니처

인덱스 시그니처는 동적 키와 값을 다룰 수 있도록 해주는 타입스크립트의 기능이다. 자바스크립트에서는 문자열 키를 객체에 매핑할 때, 키의 타입이나 값의 타입을 강제하지 않지만 타입스크립트에서는 이를 유연하게 표현하기 위해 **인덱스 시그니처**를 사용할 수 있다.

```tsx
type Rocket = { [property: string]: string };
const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};
```

- **인덱스 시그니처(`[property: string]: string`)의 의미**
    - 키의 이름: `property`는 키의 위치를 표현하는 용도로만 쓰이며, 타입체커에서 무시된다.
    - 키의 타입: 위 예시에서는 문자열(`string`)이어야 하지만, 일반적으로 `number`나 `symbol`도 가능하다.
    - 값의 타입: 위 예시에서는 키에 매핑되는 값은 문자열(`string`)이어야 하지만, 일반적으로는 어떤 것이든 될 수 있다.
- **인덱스 시그니처의 단점**
    - 모든 문자열 키를 허용하기 때문에 잘못된 키도 통과될 수 있다. 예를 들어, `name` 대신 `Name`을 사용해도 오류가 발생하지 않는다.
    - 특정 키가 반드시 있어야 한다는 보장이 없다.  {}도 유효한 Rocket 타입이다.
    - 키마다 서로 다른 값의 타입을 가질 수 없다. 예를 들어 `thrust`는 `number`여야 할 수도 있지만, 모든 값이 문자열로 제한된다.
    - 타입스크립트의 자동 완성 기능이 제대로 동작하지 않을 수 있다.

### 대안: 인터페이스

- 인터페이스 사용
    - 인덱스 시그니처 대신, 명확한 키와 타입을 가진 **인터페이스**를 사용할 수 있다.
        
        ```tsx
        interface Rocket {
        	name: string;
        	variant: string;
        	thrust_kN: number;
        }
        const falconHeavy: Rocket = {
        	name: 'Falcon Heavy1f
        	variant: 'vl',
        	thrust_kN: 15_200
        }；
        ```
        
    - thrust_kN은 `number` 타입이다.
    - 이제 타입스크립트는 모든 필수 필드가 존재하는지 확인한다.
    - 자동완성, 정의로 이동, 이름 바꾸기 등의 언어 서비스도 지원한다.

### 인덱스 시그니처의 적절한 사용 예: 동적 데이터

인덱스 시그니처는 **동적 데이터**를 다룰 때 유용하다.

예를 들어 CSV 파일처럼 헤더 행(row)에 열(column) 이름이 있고, 데이터 행을 열 이름과 값으로 매핑하는 객체로 나타내고 싶은 경우 사용할 수 있다.

- 데이터 매핑 예시
    
    ```tsx
    function parseCSV(input: string): { [columnName: string]: string }[] {
      const lines = input.split('\n');
      const [header, ...rows] = lines;
      const headerColumns = header.split(',');
    
      return rows.map(rowStr => {
        const row: { [columnName: string]: string } = {};
        rowStr.split(',').forEach((cell, i) => {
          row[headerColumns[i]] = cell;
        });
        return row;
      });
    }
    ```
    
    - 여기서는 열 이름이 무엇인지 미리 알 수 없기 때문에 인덱스 시그니처를 사용한다. 하지만, 열 이름이 고정되어 있는 경우, 미리 선언해 둔 타입으로 단언문을 사용한다.
        
        ```tsx
        interface ProductRow {
          productId: string;
          name: string;
          price: string;
        }
        
        declare let csvData: string;
        const products = parseCSV(csvData) as unknown as ProductRow[];
        ```
        
    - 물론 선언해 둔 열들이 런타임에 실제로 일치한다는 보장은 없다. 이 부분이 걱정된다면 값 타입에 undefined를 추가할 수 있다.
        
        ```tsx
        function safeParseCSV(input: string): { 
        	[columnName: string]: string | undefined 
        }[] {
          return parseCSV(input);
        }
        ```
        
    - 이 경우, 각 값을 사용할 때마다 `undefined` 여부를 체크해야 한다.
        
        ```tsx
        const rows = parseCSV(csvData);
        
        const prices: {[produt: string]: number} = {};
        for (const row of rows) {
        	prices[row.productld] = Number(row.price);
        }
        
        const safeRows = safeParseCSV(csvData);
        for (const row of safeRows) {
        	prices[row.productld] = Number(row.price);
        					// 'undefined' 형식을 인덱스 형식으로 사용할 수 없습니다.
        }
        ```
        

---

## Item 16. **number** 인덱스 시그니처보다는 **Array, Tuple, ArrayLike를 사용하기**

자바스크립트에서는 객체의 키가 숫자가 아닌 문자열로 처리된다. 숫자를 키로 사용할 때 자바스크립트는 자동으로 이를 문자열로 변환한다. 이와 같은 동작 때문에 타입스크립트는 `number` 인덱스 시그니처를 지원하지만, 실제로는 런타임에서 문자열로 인식된다.

### 자바스크립트 객체와 배열

- **자바스크립트 객체의 키**
    - 파이썬이나 자바에서 볼 수 있는 ‘해시 가능’ 객체라는 표현이 자바스크립트에는 없다.
    - 만약 더 복잡한 객체를 키로 사용하려고 하면, toString 메서드가 호출되어 객체가 문자열로 변환된다.
        
        ```tsx
        > x = {}
        {}
        > x[[l, 2, 3]] = 2
        2
        > x
        { '1,2,3': 1 }
        ```
        
    - 숫자는 객체의 키로 사용할 수 없으며, 숫자로 작성된 키는 자동으로 문자열로 변환된다.
        
        ```tsx
        > x = {}
        {}
        > x[[l, 2, 3]] = 2
        2
        > x
        { '1,2,3': 1 }
        ```
        
- **자바스크립트의 배열**
    - 배열은 객체이기 때문에 인덱스 역시 문자열로 변환되어 사용된다. 문자열 키를 사용해도 역시 배 열의 요소에 접근할 수 있다.
        
        ```tsx
        > x = [1, 2, 3]
        [ 1, 2, 3 ]
        > x[0]
        1
        
        > x['1']
        2
        ```
        
    - 그러나 배열의 숫자 인덱스는 타입스크립트에서 `number`로 선언된다. 타입스크립트는 이러한 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식한다.
        
        ```tsx
        const xs = [1, 2, 3];
        const x0 = xs[0];   // OK
        const x1 = xs['1']; // 타입스크립트에서는 에러
        ```
        

### 타입스크립트의 배열

- **타입스크립트에서 배열 타입 선언**
    
    ```tsx
    interface Array<T> {
      [n: number]: T;
    }
    ```
    
    - 배열의 키는 런타임에서는 여전히 문자열로 처리되지만, 타입 시스템에서는 이를 `number`로 인식하여 타입 체크 시점에 오류를 잡고 타입 안정성을 보장할 수 있다.
        
        ```tsx
        const xs = [1, 2, 3];
        const x0 = xs [0]; // OK
        const xl = xs['1'];
        					// ~ 인덱스 식이 'number' 형식이 아니므로
        					// 요소에 암시적으로 'any' 형식이 있습니다.
        function get<T>(array: T[], k: string): T {
        	return array[k];
        					// … 인덱스 식이 'number' 형식이 아니므로
        					// 요소에 암시적으로 'any' 형식이 있습니다.
        }
        ```
        
- **배열 순회 시 주의할 점**
    - `for-in` 루프는 배열을 순회하는 데 적합하지 않다. 이 루프는 키가 문자열로 처리되어 혼란을 야기할 수 있다. 대신 `for-of` 또는 `Array.prototype.forEach`를 사용하는 것이 좋다.
    - `for-of` : 인덱스에 신경 쓰지 않는 경우
        
        ```tsx
        for (const x of xs) {
          console.log(x); // number 타입
        }
        ```
        
    - `Array.prototype.forEach` : 인덱스의 타입이 중요한 경우
        
        ```tsx
        xs.forEach((x, i) => {
          console.log(i, x); // i는 number, x는 number 타입
        });
        ```
        

---

## Item 17.  변경 관련된 오류 방지를 위해 **readonly** 사용하기

### 변경 관련 오류: 삼각수 예시

- 오류가 있는 삼각수 출력 코드
    
    ```tsx
    function arraySum(arr: number[]) {
      let sum = 0;
      let num;
      while ((num = arr.pop()) !== undefined) {
        sum += num;
      }
      return sum;
    }
    
    function printTriangles(n: number) {
      const nums = [];
      for (let i = 0; i < n; i++) {
        nums.push(i);
        console.log(arraySum(nums));
      }
    }
    ```
    
    ```tsx
    > printTriangles(5)
    0
    1
    2
    3
    4
    ```
    
    - 위 코드가 의도한 대로 작동하지 않는 이유는 `arraySum` 함수가 `nums` 배열을 변경하지 않는다고 가정하기 때문이다. `arraySum` 함수가 배열을 수정하지 않는다고 생각하고 `nums`를 전달했지만, 사실 `arraySum` 함수는 배열을 수정하는 `pop()` 메서드를 사용하여 배열을 비워내기 때문에 for문의 매 단계에서 배열이 누적되지 않는다.
    - 자바스크립트 배열은 내용을 변경할 수 있기 때문에 타입스크립트에서도 오류 없이 통과한다.

### readonly 접근 제어자 도입

- readonly의 특성
    - 배 열의 요소를 읽을 수 있지만, 쓸 수는 없다.
    - length를 읽을 수 있지만, 바꿀 수는 없다.
    - 배 열을 변경하는 pop을 비롯한 다른 메서드를 호출할 수 없다.
- 수정된 코드
    
    ```tsx
    function arraySum(arr: readonly number[]) {
      let sum = 0;
      let num;
      while ((num = arr.pop()) !== undefined) {
        // 오류 발생: 'readonly number[]' 형식에 'pop' 속성이 없습니다
        sum += num;
      }
      return sum;
    }
    ```
    
    - `readonly`는 해당 배열이 변경되지 않음을 보장한다.
    - `arr` 매개변수는 `readonly number[]`로 선언되어 있어, 배열을 변경할 수 없다. 이제 `pop()` 메서드를 사용할 수 없게 되고, 컴파일러가 오류를 발생시킨다.
    - 매개변수를 readonly로 선언하면 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크하고 함수가 매개변수를 변경하지 않는다고 가정하지만, 명시적인 방법을 사용하는 것이 좋다. 코드에서 배열을 변경하는 부분을 아예 제거하는 식으로 고칠 수 있다.
        
        ```tsx
        function arraySum(arr: readonly number[]) {
        	let sum = 0;
        	for (const num of arr) {
        		sum += num;
        	}
        	return sum;
        }
        ```
        

---

## Item 18. 매핑된 타입을 사용하여 값을 동기화하기

### 산점도 컴포넌트 예시

산점도(Scatter Plot)를 그리기 위한 UI 컴포넌트를 구현할 때는 여러 속성을 관리하게 된다. 예를 들어 `ScatterProps` 인터페이스는 다음과 같은 속성을 포함한다.

- scatterProps interface
    
    ```tsx
    interface ScatterProps {
      // The data
      xs: number[];
      ys: number[];
      
      // Display
      xRange: [number, number];
      yRange: [number, number];
      color: string;
    
      // Events
      onClick: (x: number, y: number, index: number) => void;
    }
    ```
    
    - 산점도를 최적화하려면, 데이터나 디스플레이 속성이 변경될 때만 차트를 다시 그리고, 이벤트 핸들러가 변경될 때는 다시 그릴 필요가 없다.

### 최적화 접근법 1: 보수적 접근

- 보수적 접근
    
    ```tsx
    function shouldUpdate(
      oldProps: ScatterProps,
      newProps: ScatterProps
    ): boolean {
      let k: keyof ScatterProps;
      for (k in oldProps) {
        if (oldProps[k] !== newProps[k]) {
          if (k !== 'onClick') return true;  // onClick은 체크하지 않음
        }
      }
      return false;
    }
    ```
    
    - ‘실패에 닫힌 접근’이라고도 한다.
    - 만약 새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때마다 차트를 다시 그린다.
    - 이 방식은 차트를 너무 자주 다시 그릴 수 있지만, 정확한 렌더링을 보장한다.

### 최적화 접근법 2: 실패에 열린 접근

- 실패에 열린 접근
    
    ```tsx
    function shouldUpdate(
      oldProps: ScatterProps,
      newProps: ScatterProps
    ): boolean {
      return (
        oldProps.xs !== newProps.xs ||
        oldProps.ys !== newProps.ys ||
        oldProps.xRange !== newProps.xRange ||
        oldProps.yRange !== newProps.yRange ||
        oldProps.color !== newProps.color
        // (no check for onClick)
      );
    }
    ```
    
    - 이 방식은 재렌더링을 최소화하지만, 누락된 속성이 있을 경우 차트가 갱신되지 않는 위험이 존재한다.
    - 일반적인 경우에 쓰이는 방법은 아니다.

### 매핑된 타입을 활용한 최적화

앞선 두 가지 최적화 방법 모두 이상적이지 않다. 매핑된 타입은 속성별로 업데이트가 필요한지 여부를 명시할 수 있으므로, 매핑된 타입을 사용해 새로운 속성이 추가될 때 직접 shouldUpdate를 고치도록 하는 게 낫다.

- 매핑된 타입
    
    ```tsx
    const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
      xs: true,
      ys: true,
      xRange: true,
      yRange: true,
      color: true,
      onClick: false,
    };
    
    function shouldUpdate(
      oldProps: ScatterProps,
      newProps: ScatterProps
    ): boolean {
      let k: keyof ScatterProps;
      for (k in oldProps) {
        if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
          return true;
        }
      }
      return false;
    }
    ```
    
    - `REQUIRES_UPDATE` 객체는 각 속성이 업데이트가 필요한지 여부를 명확하게 관리할 수 있도록 한다.
    - `[k in keyof ScatterProps]`은 타입 체커 에게 `REQUIRES_UPDATE`가 `ScatterProps`과 동일한 속성을 가져야 한다는 정보를 제공한다.
    - 여기서 `onClick`은 `false`로 설정되어, 변경되어도 차트가 다시 그려지지 않는다.
- 매핑된 타입의 안정성
    
    ```tsx
    interface ScatterProps {
      onDoubleClick: () => void;
    }
    
    const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
      // 오류 발생: onDoubleClick 속성이 정의되지 않았음
    };
    ```
    
    - 매핑된 타입을 사용하면 속성의 추가 또는 변경 시 타입 체커가 경고를 제공할 수 있다. 예를 들어, 새로운 속성을 추가할 경우 타입 체커가 자동으로 오류를 잡아주기 때문에, 속성 추가나 삭제 시 실수를 방지할 수 있다.

---

## ❓궁금증 

## 💭 하늘 : const와 readonly의 차이

`const`와 `readonly`는 둘 다 값을 변경하지 못하게 제한하는 용도로 사용되지만, 적용 범위와 대상이 다르며 동작 방식에도 차이가 있다. 

### 1. `const`

- **대상**: 변수에 사용
- **적용 범위**: 변수 자체에 대한 재할당을 방지. 즉, `const`로 선언된 변수는 재할당이 불가능
- **변경 가능성**: 객체나 배열 같은 참조 타입의 경우, 내부 속성이나 요소는 변경 가능

- **예시 1: 기본 값**
    
    ```tsx
    const x = 10;
    x = 20;  // 오류: 'x'는 const로 선언되어 재할당할 수 없음
    ```
    
    - `const`로 선언된 변수 `x`는 값이 재할당될 수 없다.
- 예시 2: 참조 타입 (객체, 배열)
    
    ```tsx
    const arr = [1, 2, 3];
    arr[0] = 100;  // 정상: 배열 요소는 변경 가능
    arr = [4, 5, 6];  // 오류: 'arr'에 재할당할 수 없음
    
    const obj = { name: "John" };
    obj.name = "Doe";  // 정상: 객체 속성은 변경 가능
    obj = { name: "Jane" };  // 오류: 'obj'에 재할당할 수 없음
    ```
    
    - `const`는 변수의 참조를 고정하지만, 참조된 객체의 내부는 변경 가능하다.
    - 즉 배열 또는 객체에 재할당은 불가능하지만, 내부의 값이나 속성은 변경할 수 있다.

### 2. `readonly`

- **대상**: 프로퍼티(객체의 속성) 또는 배열의 요소
- **적용 범위**: 값의 변경을 방지. `readonly`로 선언된 객체 속성이나 배열 요소는 변경할 수 없지만, 객체 자체를 참조하는 변수는 변경 가능
- **변경 가능성**: `readonly`는 배열이나 객체 내부의 값을 변경하지 못하게 강제

- **예시 1: 배열**
    
    ```tsx
    const arr: readonly number[] = [1, 2, 3];
    arr[0] = 100;  // 오류: readonly 배열의 요소는 변경할 수 없음
    arr.push(4);  // 오류: readonly 배열에 요소를 추가할 수 없음
    ```
    
    - 여기서 `readonly` 배열은 요소를 변경하거나 추가할 수 없다.
- **예시 2: 객체 속성**
    
    ```tsx
    interface Person {
      readonly name: string;
      age: number;
    }
    
    const person: Person = { name: "John", age: 25 };
    person.age = 30;  // 정상: 'age'는 변경 가능
    person.name = "Doe";  // 오류: readonly 속성은 변경할 수 없음
    ```
    
    - `readonly`로 선언된 객체 속성 `name`은 변경할 수 없다.
    - 하지만 `age`는 `readonly`가 아니므로 변경이 가능하다.

### `const` vs`readonly`

| 특성 | `const` | `readonly` |
| --- | --- | --- |
| **대상** | 변수 | 객체 속성, 배열 요소 |
| **적용 범위** | 재할당 불가 | 값의 변경 불가 |
| **참조 타입** | 참조는 고정되지만 내부는 변경 가능 | 참조된 값(속성, 요소)의 변경 자체가 불가능 |
| **사용 위치** | 함수, 전역 또는 블록 내의 변수 | 인터페이스, 클래스, 배열 요소 |
| **재할당** | 불가능 | 가능 (하지만 readonly 속성/요소는 변경 불가) |
<br/>

## 💭 경인 : ArrayLike를 권장하는 이유는?
Q. 배열은 인덱스 값으로 number가 사용되어야 하지만 문자열일 경우에도 허용을 하는 점이 문제이기 때문에 Array, Tuple을 사용하도록 권장하는데 키를 문자열로 사용하는ArrayLike를 사용하도록 하는 이유가 무엇인지 궁금함
<br/>
A. ArrayLike 객체는 배열처럼 동작하지만 실제 배열은 아니고, 문자열 키를 이용하여 배열의 속성에 접근해야 할 때 유용하게 쓰임.
<br/>

## 💭 정민 : 중요하다고 생각하는 부분
타입 중복은 코드의 중복많큼 많은 문제를 발생시킨다. 따라서 타입에 이름을 붙이거나, 타입을 확장해서 반복을 제거해야한다. 또한, 제너릭 타입은 타입을 위한 함수로 타입을 반복하는 대신 제너릭 타입을 사용해 타입 간에 매핑을 하는 것이 좋다.

## 💭 민혁 : 중요하다고 생각하는 부분
