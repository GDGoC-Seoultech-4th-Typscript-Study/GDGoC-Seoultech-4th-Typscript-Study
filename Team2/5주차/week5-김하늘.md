# Ch.3 - 타입 추론 (2/2)

>[!tip]
>
> - 타입 시스템이란 무엇인지
> - 타입 시스템은 어떻게 사용해야 하는지
> - 타입시스템에서 무엇을 결정해야 하는지
> - 타입시스템에서 가급적 사용하지 말아야 할 기능

---

## Item 23. 한꺼번에 객체 생성하기

타입스크립트는 객체를 생성할 때 속성을 한꺼번에 정의하는 것이 타입 추론에 유리하다. 속성을 하나씩 추가하면 타입스크립트의 타입 체커에서 오류가 발생할 수 있다. 이를 방지하고 안전한 객체를 생성하기 위해 다양한 방법을 사용할 수 있다.

### 1.객체 생성 시 속성 한꺼번에 정의하기

- 자바스크립트에서는 아래와 같이 속성을 하나씩 추가할 수 있다.
    
    ```tsx
    const pt = {};
    pt.x = 3;
    pt.y = 4;
    ```
    
- 타입스크립트에서는 위 코드가 오류를 발생시킨다.
    
    ```tsx
    const pt = {};
    pt.x = 3; // 오류: '{}' 형식에 'x' 속성이 없습니다.
    pt.y = 4; // 오류: '{}' 형식에 'y' 속성이 없습니다.
    ```
    
- 해결방법 - 객체를 생성할 때 한꺼번에 정의한다.
    
    ```tsx
    const pt: Point = {
      x: 3,
      y: 4,
    }; // 정상
    ```
    

### 2. **타입 단언문을 사용해 임시 해결**

- 속성을 나누어 추가해야 한다면, `as`를 사용해 타입 단언문으로 해결할 수 있다
    
    ```tsx
    const pt = {} as Point;
    pt.x = 3;
    pt.y = 4; // 정상
    ```
    

### 3. **작은 객체를 조합해서 큰 객체 생성**

- 작은 객체들을 조합하여 큰 객체를 생성할 때도 여러 단계를 거치는 것은 피해야한다.
    
    ```tsx
    const pt = { x: 3, y: 4 };
    const id = { name: 'Pythagoras' };
    const namedPoint = {};
    Object.assign(namedPoint, pt, id);
    
    [namedPoint.name](http://namedpoint.name/);
    // 오류: '{}' 형식에 'name' 속성이 없습니다.
    ```
    
- 해결 방법 - 객체 전개 연산자 `{ ...a, ...b }`를 사용하여 한꺼번에 생성한다.
    
    ```tsx
    const namedPoint = { ...pt, ...id };
    namedPoint.name; // 정상
    ```
    

### 4. 조건부 속성 추가

- 객체 전개 연산자를 사용하면 조건부로 속성을 추가할 수 있다.
    
    ```tsx
    declare let hasMiddle: boolean;
    
    const firstLast = { first: 'Harry', last: 'Truman' };
    const president = { 
      ...firstLast, 
      ...(hasMiddle ? { middle: 'S' } : {}) 
    };
    
    // 타입 추론 결과:
    // const president: { middle?: string; first: string; last: string }
    ```
    

### 5. **유니온 타입과 선택적 필드**

- 조건부 속성이 포함된 객체는 유니온 타입으로 추론될 수 있다.
    
    ```tsx
    declare let hasDates: boolean;
    
    const nameTitle = { name: 'Khufu', title: 'Pharaoh' };
    const pharaoh = {
      ...nameTitle,
      ...(hasDates ? { start: -2589, end: -2566 } : {})
    };
    
    // 타입 추론 결과:
    // const pharaoh: { name: string; title: string } | { start: number; end: number; name: string; title: string }
    ```
    
- 위처럼 `start`와 `end`가 선택적 필드이기를 원할 경우, 유니온 타입이 아니라 선택적 필드로 표현해야 한다. 이를 위해 **헬퍼 함수**를 활용할 수 있다.
    
    ```tsx
    function addOptional<T extends object, U extends object>(
      a: T,
      b: U | null
    ): T & Partial<U> {
      return { ...a, ...b };
    }
    
    const pharaoh = addOptional(
      nameTitle,
      hasDates ? { start: -2589, end: -2566 } : null
    );
    
    pharaoh.start; // 정상, 타입: number | undefined
    ```
    

---

## Item 24.  일관성 있는 별칭 사용하기

타입스크립트에서 별칭(alias)을 사용하는 경우, 별칭이 남발되면 코드 가독성을 떨어뜨리고, 제어 흐름 분석에 혼란을 줄 수 있다. 별칭 사용 시에는 **일관성 있는 이름 사용**과 **타입 정제 흐름 방해 최소화**를 고려해야 한다.

### 별칭 사용과 동작 방식

- 객체의 속성에 별칭을 할당하면, 별칭을 통해 변경한 값은 원래 속성에도 영향을 미친다.
- 예시: 별칭 생성과 값 변경
    
    ```tsx
    const borough = { name: 'Brooklyn', location: [40.688, -73.979] };
    const loc = borough.location; // 별칭 생성
    loc[0] = 0; // 별칭을 통한 값 변경
    console.log(borough.location); // [0, -73.979]
    ```
    

### **별칭의 문제점 - 제어 흐름 분석 방해**

- 별칭을 남용하면 컴파일러의 타입 정제를 방해할 수 있다.
- 예제: 별칭으로 타입 정제 방해
    - 별칭을 남용하면 컴파일러의 타입 정제를 방해할 수 있다.
        
        ```tsx
        function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
          const box = polygon.bbox; // 별칭 생성
          if (polygon.bbox) { // polygon.bbox로 타입 정제
            // box는 타입이 여전히 BoundingBox | undefined
            if (pt.x < box.x[0] || pt.x > box.x[1] || 
                pt.y < box.y[0] || pt.y > box.y[1]) {
              return false;
            }
          }
        }
        ```
        

### 일관된 별칭 사용

- 별칭 사용 시에는 객체 속성에 대한 별칭과 이름을 일관되게 유지해야 제어 흐름 분석이 제대로 이루어진다.
- 예제: `box` 별칭 일관 사용
    
    ```tsx
    function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
      const box = polygon.bbox;
      if (box) { // 별칭으로 타입 정제
        if (pt.x < box.x[0] || pt.x > box.x[1] || 
            pt.y < box.y[0] || pt.y > box.y[1]) {
          return false;
        }
      }
    }
    ```
    

### 일관된 별칭 사용

- 비구조화를 통해 속성 이름을 일관되게 유지할 수 있다.
- 예제: 비구조화로 속성 이름 일관성 유지
    
    ```tsx
    function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
      const { bbox } = polygon;
      if (bbox) {
        const { x, y } = bbox;
        if (pt.x < x[0] || pt.x > x[1] || 
            pt.y < y[0] || pt.y > y[1]) {
          return false;
        }
      }
    }
    ```
    

---

## Item 25. 비동기 코드에는 콜백 대신 **async** 함수 사용하기

### 콜백의 한계

- **콜백 지옥**
    
    ```tsx
    fetchURL(urll, function(responsel) {
      fetchURL(url2, function(response2) {
        fetchURL(url3, function(response3) {
          console.log(1);
        });
        console.log(2);
      });
      console.log(3);
    });
    console.log(4);
    // 로그: 4 → 3 → 2 → 1
    ```
    
    - 코드 중첩이 심해지고 가독성이 떨어짐
    - 코드의 순서와 실행 순서 불일치로 인해 직관적이지 않음
    - 병렬 실행이나 오류 처리 추가 시 더 복잡해짐

### 대안: 프로미스(Promise)

- 프로미스(promise) 개념을 도입
    
    ```tsx
    const pagelPromise = fetch(urll);
    
    pagelPromise
      .then(responsel => fetch(url2))
      .then(response2 => fetch(url3))
      .then(response3 => {
        // 작업
      })
      .catch(error => {
        // 오류 처리
      });
    ```
    
    - 코드 중첩을 줄이고, 실행 순서를 코드 순서와 일치시킴
    - 병렬 처리 및 고급 처리(e.g., `Promise.all`)를 더 쉽게 작성 가능

### 대안: async/await

- async와 await 키워드 도입
    
    ```tsx
    async function fetchPages() {
      try {
        const responsel = await fetch(urll);
        const response2 = await fetch(url2);
        const response3 = await fetch(url3);
      } catch (e) {
        console.error(e);
      }
    }
    ```
    
    - 프로미스를 더욱 간결하고 직관적으로 작성할 수 있도록 개선한 방식
    - `await` 키워드는 프로미스가 처리(resolve)될 때까지 실행을 멈춤.
    - 프로미스가 거절(reject)되면 **예외(Exception)**를 던지므로 `try/catch`로 오류 처리 가능.

---

## Item 26. 타입 추론에 문맥이 어떻게 사용되는지 이해하기

TypeScript는 단순히 값의 형태뿐만 아니라 **문맥**을 고려하여 타입을 추론한다. 그러나 문맥을 분리하면 예상치 못한 타입 오류가 발생할 수 있다. 이러한 문제를 이해하고 적절히 해결하는 방법을 살펴본다.

### 문맥 기반 타입 추론

- TypeScript는 값이 사용되는 문맥을 기반으로 타입을 추론한다.
- 예시: `setLanguage` 함수
    
    ```tsx
    type Language = 'JavaScript' | 'TypeScript' | 'Python';
    function setLanguage(language: Language) { /* ... */ }
    
    // 정상: 인라인으로 문자열 리터럴 전달
    setLanguage('JavaScript');
    
    // 오류: 변수로 분리된 경우
    let language = 'JavaScript';
    setLanguage(language);
    // ❌ 'string' 형식은 'Language' 형식에 할당될 수 없습니다.
    ```
    
    - `let language = 'JavaScript';`에서 TypeScript는 `language`를 `string`으로 추론
    - `setLanguage`의 매개변수는 `Language` 타입만 허용하므로 오류가 발생

### 해결 방법

1. **변수에 명시적으로 타입 선언 추가**
    
    ```tsx
    let language: Language = 'JavaScript';
    setLanguage(language); // 정상
    ```
    
    - 변수 타입을 명시하면, 할당 오류를 방지
    - 오타 검출에도 유리
2. **`const`를 사용하여 문자열 리터럴 타입 추론**
    
    ```tsx
    const language = 'JavaScript';
    setLanguage(language); // 정상
    ```
    
    - `const`를 사용하면 TypeScript는 `language`를 `'JavaScript'` 리터럴 타입으로 추론

### **튜플 사용 시 주의점**

- **문제: 배열로 정의된 변수**
    
    ```tsx
    function panTo(where: [number, number]) { /* ... */ }
    
    // 정상
    panTo([10, 20]);
    
    // 오류
    const loc = [10, 20];
    panTo(loc);
    // ❌ 'number[]' 형식은 '[number, number]' 형식에 할당될 수 없습니다.
    ```
    
    - `[10, 20]`은 튜플로 판단되지만, `loc`는 일반 배열(`number[]`)로 추론
- **해결 방법 1:** 명시적인 타입 선언
    
    ```tsx
    const loc: [number, number] = [10, 20];
    panTo(loc); // 정상
    ```
    
- **해결 방법 2:** `as const`를 사용하여 상수 문맥 제공
    
    ```tsx
    const loc = [10, 20] as const;
    panTo(loc); // 정상
    ```
    
    - `as const`는 `loc`의 타입을 `readonly [10, 20]`으로 추론
    - 이 경우 함수의 시그니처를 `readonly`로 변경해야
        
        ```tsx
        function panTo(where: readonly [number, number]) { /* ... */ }
        ```
        

### 객체 사용 시 주의점

- **문제: 객체의 프로퍼티 타입**
    
    ```tsx
    type Language = 'JavaScript' | 'TypeScript' | 'Python';
    interface GovernedLanguage {
      language: Language;
      organization: string;
    }
    
    function complain(language: GovernedLanguage) { /* ... */ }
    
    // 정상
    complain({ language: 'TypeScript', organization: 'Microsoft' });
    
    // 오류
    const ts = { language: 'TypeScript', organization: 'Microsoft' };
    complain(ts);
    // ❌ 'string' 형식은 'Language' 형식에 할당될 수 없습니다.
    ```
    
    - `ts.language`가 `string`으로 추론되어 오류가 발생
- **해결 방법 1:** 명시적인 타입 선언
    
    ```tsx
    const ts: GovernedLanguage = { language: 'TypeScript', organization: 'Microsoft' };
    complain(ts); // 정상
    ```
    
- **해결 방법 2:** `as const`를 사용하여 상수 문맥 제공
    
    ```tsx
    const ts = { language: 'TypeScript', organization: 'Microsoft' } as const;
    complain(ts); // 정상
    ```
    

### 콜백 사용 시 주의점

- **문제: 콜백에서 타입 누락**
    
    ```tsx
    function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
      fn(Math.random(), Math.random());
    }
    
    // 정상: 인라인 콜백
    callWithRandomNumbers((a, b) => console.log(a + b));
    
    // 오류: 콜백을 변수로 분리
    const fn = (a, b) => console.log(a + b);
    // ❌ 'a', 'b' 매개변수에 암시적으로 'any'가 포함됩니다.
    callWithRandomNumbers(fn);
    ```
    
    - 콜백 함수에서 매개변수 타입 추론이 문맥에 의존하는 경우 문제가 발생
- **해결 방법 1:** 매개변수에 타입 명시
    
    ```tsx
    const fn = (a: number, b: number) => console.log(a + b);
    callWithRandomNumbers(fn); // 정상
    ```
    
- **해결 방법 2:** 함수 전체에 타입 선언 추가
    
    ```tsx
    const fn: (n1: number, n2: number) => void = (a, b) => console.log(a + b);
    callWithRandomNumbers(fn); // 정상
    ```
    

---

## Item 27.  함수형 기법과 라이브러리로 타입 흐름 유지하기

### **배경**

- 자바스크립트는 다른 언어처럼 표준 라이브러리가 풍부하지 않음.
- jQuery, Underscore, Lodash, Ramda와 같은 라이브러리가 이를 보완.
- 특히 Lodash와 Ramda는 함수형 프로그래밍을 자바스크립트 환경에 도입.
- 타입스크립트와 조합하면 함수형 기법의 장점이 더욱 극대화됨:
    - **타입 흐름 유지**: 데이터 가공 시 타입 정보를 잃지 않음.
    - **코드 가독성**: 루프 대신 함수형 기법을 활용해 코드가 간결해짐.

### **예제: CSV 데이터 파싱**

**1. 절차형 코드**

```tsx
typescript
Copy code
const csvData = "name,age\nAlice,30\nBob,25";
const rawRows = csvData.split('\n');
const headers = rawRows[0].split(',');
const rows = rawRows.slice(1).map(rowStr => {
  const row: Record<string, string> = {};
  rowStr.split(',').forEach((val, j) => {
    row[headers[j]] = val;
  });
  return row;
});

```

- **문제점**: 타입을 수동으로 관리해야 함.

**2. 함수형 기법을 활용한 코드**

```tsx
typescript
Copy code
const rows = rawRows.slice(1).map(rowStr =>
  rowStr.split(',').reduce((row, val, i) => {
    row[headers[i]] = val;
    return row;
  }, {} as Record<string, string>)
);

```

- **문제점**: 코드가 간결해졌지만, 가독성이 떨어질 수 있음.

**3. Lodash를 사용한 코드**

```tsx
typescript
Copy code
import _ from 'lodash';

const rows = rawRows.slice(1).map(rowStr =>
  _.zipObject(headers, rowStr.split(','))
);

```

- **장점**:
    - 코드가 간결하고 가독성 높음.
    - 타입스크립트와 함께 사용하면 추가 타입 정의 없이도 오류 없이 동작.

### **타입 유지와 함수형 기법의 장점**

- **타입 유지**:
    - 함수형 라이브러리는 기존 데이터 타입을 수정하지 않고 새로운 값을 반환.
    - 안전한 타입 흐름 유지 가능.
- **효율적인 데이터 처리**:
    - Lodash와 같은 라이브러리를 사용하면 데이터 가공 시 반복적인 타입 정의를 줄일 수 있음.

---

## 더 알아볼 점 : 비동기 코드와 async/await의 조합

### Promise.all과 async/await의 조합

`async/await`는 비동기 코드를 더 직관적이고 동기적인 코드처럼 작성할 수 있게 해주는 구문이고, `Promise.all`은 여러 개의 비동기 작업을 병렬로 처리하는 데 유용한 메서드이다. 이 두 가지를 결합하면, 여러 비동기 작업을 병렬로 실행하면서도 코드를 깔끔하게 유지할 수 있다.

- 예시: `Promise.all`과 `async/await`의 조합
    
    ```tsx
    async function fetchData() {
        const url1 = "https://api.example.com/data1";
        const url2 = "https://api.example.com/data2";
    
        try {
            // 여러 비동기 작업을 병렬로 실행
            const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
            
            // 데이터 처리
            const jsonData1 = await data1.json();
            const jsonData2 = await data2.json();
    
            console.log(jsonData1, jsonData2);
        } catch (error) {
            console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
        }
    }
    
    fetchData();
    ```
    

### 병렬 실행과 직렬 실행의 차이

- **병렬 실행**
    - `Promise.all`을 사용하면 여러 비동기 작업을 동시에 시작하여, 모든 작업이 완료될 때까지 기다린다. 각 작업은 서로 독립적으로 실행되므로, 작업이 완료되는 순서에 관계없이 기다리지 않고 동시에 처리된다.
        
        ```tsx
        async function fetchData() {
            const url1 = "https://api.example.com/data1";
            const url2 = "https://api.example.com/data2";
        
            try {
                // 여러 비동기 작업을 병렬로 실행
                const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
                
                // 데이터 처리
                const jsonData1 = await data1.json();
                const jsonData2 = await data2.json();
        
                console.log(jsonData1, jsonData2);
            } catch (error) {
                console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
            }
        }
        
        fetchData();
        ```
        
- **직렬 실행**
    - `await`를 사용하면 각 비동기 작업이 순차적으로 실행된다. 이전 작업이 완료되어야만 다음 작업이 시작되므로, 병렬 실행보다 시간이 더 걸릴 수 있다.
        
        ```tsx
        async function fetchData() {
            const url1 = "https://api.example.com/data1";
            const url2 = "https://api.example.com/data2";
        
            try {
                // 여러 비동기 작업을 병렬로 실행
                const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
                
                // 데이터 처리
                const jsonData1 = await data1.json();
                const jsonData2 = await data2.json();
        
                console.log(jsonData1, jsonData2);
            } catch (error) {
                console.error("데이터를 가져오는 데 오류가 발생했습니다:", error);
            }
        }
        
        fetchData();
        ```
