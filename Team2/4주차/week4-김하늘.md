# Ch.3 - 타입 추론 (1/2)

<aside>
💡

학습목표

- 타입스크립트가 어떻게 타입을 추론하는지
- 언제 타입 선언을 작성해야 하는지
- 타입 추론이 가능하더라도 명시적으로 타입 선언을 작성하는 것이 필요한 상황은 언제인지
</aside>

---

## Item 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

### 타입구문 생략하기

타입스크립트에는 개발자가 일일이 변수와 함수에 타입을 지정하지 않아도 **타입 추론**을 통해 코드의 타입을 자동으로 판단할 수 있다. 타입 추론이 된다면 명시적 타입 구문은 필요하지 않다. 코드의 모든 변수에 타입을 선언하는 것은 비생산적이며 형편없는 스타일로 여겨진다.

- **primary 변수에서의 타입 추론**
    - 편집기에서 `x` 위에 마우스를 올리면 타입이 `number`로 추론되었음을 확인가능하다.
        
        ```tsx
        // 명시적 타입 구문 사용
        let x: number = 12;
        
        // 타입 추론 사용
        let x = 12;
        ```
        
- **객체와 배열에서의 타입 추론**
    - 타입스크립트는 객체와 배열의 타입도 추론할 수 있다.
    - 예를 들어, 다음 두 코드에서 `person` 객체의 타입은 동일하게 추론된다.
        
        ```tsx
        // 명시적 타입 구문 사용
        const person: {
            name: string;
            born: { where: string; when: string };
            died: { where: string; when: string };
        } = {
            name: 'Sojourner Truth',
            born: { where: 'Swartekill, NY', when: 'c.1797' },
            died: { where: 'Battle Creek, MI', when: 'Nov. 26, 1883' }
        };
        
        // 타입 추론 사용
        const person = {
            name: 'Sojourner Truth',
            born: { where: 'Swartekill, NY', when: 'c.1797' },
            died: { where: 'Battle Creek, MI', when: 'Nov. 26, 1883' }
        };
        ```
        
- **함수 반환 타입의 추론**
    - 다음 예제처럼 배열의 경우도 입력을 받아 연산을 하는 함수가 어떤 타입을 반환하는지 추론한다.
        
        ```tsx
        // 타입 추론 사용
        function square(nums: number[]) {
        		return nums.map(x => x * x);
        }
        const squares = square( [1, 2, 3, 4]); // 타입은 number[]
        ```
        

### 타입 추론이 용이한 상황

타입이 추론되면 리팩터링이 용이해진다. Product 타입과 기록을 위한 함수를 가정해보자.

- 초기 `Product` 타입과 `logProduct` 함수
    - 초기 `Product`의 `id` 속성의 타입을 `number`로 명시하고, 이 타입을 바탕으로 `logProduct` 함수를 생성했다.
    - 속성의 타입을 명시해 생성한 타입과 함수
        
        ```tsx
        interface Product {
          id: number;
          name: string;
          price: number;
        }
        
        function logProduct(product: Product) {
          const id: number = product.id;
          const name: string = product.name;
          const price: number = product.price;
          console.log(id, name, price);
        }
        ```
        
- `Product` 타입의 변경
    - 그런데 `id`에 문자도 들어 있을 수 있음을 나중에 알게 되어 이를 `string`으로 변경했다.
    - 그러면 `logProduct` 내의 `id` 변수 선언에 있는 타입과 맞지 않기 때문에 오류가 발생한다.
        
        ```tsx
        interface Product {
          id: string;
          name: string;
          price: number;
        }
        
        function logProduct(product: Product) {
          const id: number = product.id;
        	  // 'string' 형식은 'number' 형식에 할당할 수 없습니다.
          const name: string = product.name;
          const price: number = product.price;
          console.log(id, name, price);
        }
        ```
        
- 타입 추론을 통한 리팩터링
    - 이 문제를 해결하려면 `logProduct` 함수에서 **비구조화 할당문**을 사용해 모든 속성의 타입을 자동으로 추론하도록 바꾸는 것이 좋다.
        
        ```tsx
        function logProduct(product: Product) {
          const { id, name, price } = product;
          console.log(id, name, price);
        }
        ```
        
    - 비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다.
    - 이 경우, TypeScript가 자동으로 `id`, `name`, `price`의 타입을 추론하므로, `Product`의 타입이 변경되더라도 `logProduct` 내에서 별도로 타입을 수정할 필요가 없다.

### 타입 명시가 필요한 상황 - 함수의 반환

TypeScript는 함수 매개변수의 타입을 추론할 수 있지만, 때로는 명시적 타입 구문을 통해 오류를 미리 예방할 수 있다. 타입 추론이 가능할지라도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는 게 좋다. 주식 시세를 조회하는 함수의 예시를 가정하자.

- **주식 시세를 조회하는 함수 `getQuote`**
    
    ```tsx
    function getQuote(ticker: string) {
      return fetch(`https://quotes.example.com/?q=${ticker}`)
        .then(response => response.json());
    }
    ```
    
    - `getQuote` 함수가 주어진 `ticker`에 대한 시세를 조회하고, `fetch` API를 통해 그 결과를 반환한다.
    - `fetch`는 항상 `Promise`를 반환하므로, TypeScript 이 함수도 `Promise`를 반환한다고 추론하여 이 함수는 `Promise<any>` 타입을 반환한다고 인식할 것이다.
- **문제 발생: 캐시 추가**
    - 이미 조회한 종목을 다시 요청하지 않도록 캐시를 추가한다고 가정하자.
    - 캐시 기능을 추가하면 코드가 약간 변경된다. 캐시된 값을 먼저 반환하고, 그렇지 않으면 새로운 데이터를 가져오는 방식으로 동작한다.
    
    ```tsx
    const cache: { [ticker: string]: number } = {};
    
    function getQuote(ticker: string) {
      if (ticker in cache) {
        return cache[ticker];
      }
      return fetch(`https://quotes.example.com/?q=${ticker}`)
        .then(response => response.json())
        .then(quote => {
          cache[ticker] = quote;
          return quote;
        });
    }
    ```
    
    - 이 코드에는 오류가 있다.  `getQuote` 함수는 항상 `Promise<number>`를 반환해야 하기 때문에, `if` 구문에서 `cache[ticker]`대신 `Promise.resolve(cache[ticker])`와 같이 `Promise` 객체가 되어야 한다.
    - 그런데 실행해 보면 오류는 getQuote 내부가 아닌 getQuote를 호출한코드에서 발생한다.
        
        ```tsx
        getQuote('MSFT').then(considerBuying);
        		// 'number | Promise<any>' 형식에 'then' 속성이 없습니다.
        		// 'number' 형식에 'then' 속성이 없습니다.
        ```
        
    - 이때 의도된 반환 타입(Promise<number>)을 명시하면 정확한 위치에 오류가 표시된다.
        
        ```tsx
        const cache: { [ticker: string]: number } = {};
        
        function getQuote(ticker: string) {
          if (ticker in cache) {
            return cache[ticker];
        		    // 'number' 형식은 'Promise<number>' 형식에 할당할 수 없습니다.
          }
          //..
        }
        ```
        
        - 반환 타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않는다.
- **반환 타입 명시해야하는 이유**
    1. 오류의 위치를 제대로 표시
    2. 명확한 의도 전달
    3. 명명된 타입 사용

---

## Item 20.  다른 타입에는 다른 변수 사용하기

타입스크립트에서는 자바스크립트와 달리 한 변수를 여러 타입으로 사용하기 어려우며, 타입 일관성을 유지하는 것이 중요하다

```tsx
let id = "12-34-56";
fetchProduct(id); // string으로 사용
id = 123456;
fetchProductBySerialNumber(id); // number로 사용
```

```tsx
let id = "12-34-56";
fetchProduct(id);
id = 123456; // '123456' 형식은 'string' 형식에 할당할 수 없습니다.
fetchProductBySerialNumber(id); // 'string' 형식의 인수는 'number' 형식의 매개변수에 할당될 수 없습니다.
```

### 해결 - 별도 변수 도입하기

`id` 변수를 `string | number`로 지정해 오류를 해결할 수는 있겠지만, id를 사용할 때마다 값이 어떤 타입 인지 확인해야 하기 때문에 더 복잡할 수 있다. 다른 타입에는 별도의 변수를 사용하는 것이 바람직하다.

```tsx
const id = "12-34-56";
fetchProduct(id);       // string 타입의 id 사용

const serial = 123456;
fetchProductBySerialNumber(serial); // number 타입의 serial 사용
```

### 별도 변수를 사용해야 하는 이유

- **** 서로 관련이 없는 두 개의 값을 분리한다 (id와 serial)
- **** 변수명을 더 구체적으로 지을 수 있다
- **** 타입 추론을 향상시키며, 타입 구문이 불필요해진다
- **** 타입이 좀 더 간결해진다 (string |number 대신 string과 number를 사용)
- let 대신 const로 변수를 선언하게 된다

---

## Item 21. 타입 넓히기

타입 넓히기는 타입스크립트가 상수의 값을 초기화할 때 해당 값에 가능한 가장 좁은 타입을 적용하는 대신, 좀 더 넓은 타입으로 추론하는 과정을 말한다.

### 타입 넓히기 개념

- 상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야
- 즉, 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추
- 이때 타입스크립트는 `x`의 타입을 `'x'`로 좁히는 대신, `string`으로 넓히기를 적용하여 추론
    
    ```tsx
    let x = 'x';
    
    x = 'Four score and seven years ago...'; // 넓히기 발생, 이후에 다른 문자열을 할당하는 것도 가능
    ```
    

### 코드 예제: `getComponent` 함수

- 3D 벡터에 대해 `x`, `y`, `z` 축 중 하나의 값을 얻는 함수
    
    ```tsx
    interface Vector3 { x: number; y: number; z: number; }
    function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
      return vector[axis];
    }
    
    let x = 'x';
    let vec = { x: 10, y: 20, z: 30 };
    getComponent(vec, x); // 'string' 형식의 인수는 '"x" | "y" | "z"' 형식의 매개변수에 할당할 수 없습니다.
    ```
    
    - `getComponent` 함수는 `axis` 파라미터에 `'x' | 'y' | 'z'` 타입을 기대하지만, `x` 변수는 타입 넓히기 때문에 `string`으로 추론되어 오류가 발생

### 타입 넓히기를 제어하는 방법

1. **`const` 사용하기**
    
    ```tsx
    const x = 'x'; // 타입은 "x"
    let vec = { x: 10, y: 20, z: 30 };
    getComponent(vec, x); // 정상
    ```
    
    - `let` 대신 `const`를 사용하여 변수를 선언하면 타입 넓히기를 방지 가능
    - `const`로 선언된 변수는 재할당이 불가능하기 때문에 타입스크립트는 변수를 가장 좁은 리터럴 타입으로 추론
2. **명시적 타입 구문 사용하기**
    
    ```tsx
    const v: { x: 1 | 3 | 5 } = { x: 1 }; // 타입이 { x: 1 | 3 | 5 }
    ```
    
    - 변수의 타입을 명시하여 타입스크립트가 넓히기를 수행하지 않도록 제어 가능
3. **`as const` 단언문 사용하기**
    
    ```tsx
    const v1 = { x: 1, y: 2 }; // 타입은 { x: number; y: number; }
    const v2 = { x: 1 as const, y: 2 }; // 타입은 { x: 1; y: number; }
    const v3 = { x: 1, y: 2 } as const; // 타입은 { readonly x: 1; readonly y: 2; }
    ```
    
    - `as const` 단언문은 변수를 리터럴 타입으로 지정하는 데 사용
    - `as const`를 사용하면 타입스크립트는 가장 좁은 타입으로 추론

---

## Item 22.  타입 좁히기

타입 좁히기는 TypeScript가 넓은 타입을 좁은 타입으로 축소하는 과정을 의미한다. 예를 들어, `HTMLElement | null`과 같은 넓은 타입에서 `HTMLElement`나 `null`로 구체화하는 것을 말한다.

### 코드 예제: null 체크

- null 체크
    
    ```tsx
    const el = document.getElementById('foo'); // 타입: HTMLElement | null
    
    if (el) {
      el; // 타입이 HTMLElement로 좁혀짐
      el.innerHTML = 'Party Time <blink>';
    } else {
      el; // 타입이 null
      alert('No element #foo');
    }
    ```
    
    - 위 코드에서는 `el`이 `null`이 아닌 경우에만 `HTMLElement` 타입으로 좁혀진다.
    - 즉, 첫 번재 블록에서 HTMLElement | null 타입의 null을 제외하므로 더 좁은 타입이 되어 작업이 쉬워진다.

### 타입 좁히기의 방법

1. **예외 던지기**
    
    ```tsx
    const el = document.getElementById('foo'); // 타입: HTMLElement | null
    
    if (!el) throw new Error('Unable to find #foo');
    el; // 이제 타입은 HTMLElement
    el.innerHTML = 'Party Time <blink>';
    ```
    
2. **`instanceof`를 이용하기**
    
    ```tsx
    function contains(text: string, search: string | RegExp) {
      if (search instanceof RegExp) {
        search; // 타입이 RegExp로 좁혀짐
        return !!search.exec(text);
      }
      search; // 타입이 string으로 좁혀짐
      return text.includes(search);
    }
    ```
    
3. **속성 체크를 이용하기**
    
    ```tsx
    interface A { a: number }
    interface B { b: number }
    
    function pickAB(ab: A | B) {
      if ('a' in ab) {
        ab; // 타입이 A로 좁혀짐
      } else {
        ab; // 타입이 B로 좁혀짐
      }
      ab; // 타입이 A | B
    }
    ```
    
4. **`Array.isArray`를 이용하기**
    
    ```tsx
    function contains(text: string, terms: string | string[]) {
      const termList = Array.isArray(terms) ? terms : [terms];
      termList; // 타입이 string[]
      // ...
    }
    ```
    
5. **명시적 태그를 활용하기**
    
    ```tsx
    interface UploadEvent { type: 'upload'; filename: string; contents: string }
    interface DownloadEvent { type: 'download'; filename: string }
    
    type AppEvent = UploadEvent | DownloadEvent;
    
    function handleEvent(e: AppEvent) {
      switch (e.type) {
        case 'download':
          e; // 타입이 DownloadEvent로 좁혀짐
          break;
        case 'upload':
          e; // 타입이 UploadEvent로 좁혀짐
          break;
      }
    }
    ```
    

---

## 궁금한 점 : let 대신 const로 변수를 선언하는 것의 장점

타입이 다른 두 개 이상의 변수가 필요할 때는 서로 관련이 없는 두 개의 값을 분리하고 별도의 변수를 도입하는 것이 좋다. 교재는 이렇게 하는 것의 장점 중 하나로 ‘let 대신 const로 변수를 선언할 수 있음’을 들고 있는데, 왜 let 대신 const를 쓸 수 있게 되는 건지, const로 선언하는 것이 어떤 점에서 let보다 유용한지 알아보자.

### `let`의 쓰임

변수가 이후에 변경될 가능성이 있는 경우에는 `let`을 사용한다.

- ex1) 반복문에서의 카운터 변수
    
    ```tsx
    for (let i = 0; i < 10; i++) {
        console.log(i); // `i`는 매 반복마다 값이 바뀜.
    }
    ```
    
- ex2) 동적으로 값이 변경되는 경우
    
    ```tsx
    let status = "loading";
    status = "completed"; // 상태에 따라 값을 업데이트할 필요가 있는 경우.
    ```
    

### `const`를 쓸 수 있게 되는 이유

변수에 다른 값을 할당하기 위해서는 변수가 재할당 가능해야 하므로 `let`으로 선언해야 한다. 그러나 어떤 변수를 다른 타입으로 재할당하는 대신 별도의 변수를 도입하면 변경가능하지 않아도 되므로 const로 선언할 수 있다.

### `const`로 선언하는 것의 장점

타입스크립트 코드에서 변수의 값을 변경할 필요가 없다면 `const`가 더 안전하고 권장되는 방법이다.

- **불변성 유지**
    - `const`는 값을 재할당할 수 없게 하므로, 변수의 값이 실수로 변경되는 것을 방지한다.
    - 함수나 객체를 전달할 때 특히 유용하며, 코드의 안정성을 높일 수 있다.
- **코드 가독성 향상**
    - 변수가 선언된 후 변하지 않는다는 점이 명확하기 때문에, 읽는 사람 입장에서도 해당 변수의 상태가 어디서든 동일하다는 것을 쉽게 파악할 수 있다.
    - 변수가 변경될 수 없으므로 타입을 확인할 필요가 줄어들어 가독성이 높아진다.
- **최적화 가능성**
    - `const` 변수는 재할당이 불가능하기 때문에  타입스크립트는 이 값이 변하지 않음을 알기 때문에 더욱 정확하게 타입을 추론할 수 있다.