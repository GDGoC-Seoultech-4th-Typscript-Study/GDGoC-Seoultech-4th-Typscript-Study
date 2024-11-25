#### **Item 23: 한꺼번에 객체 생성하기**

1. **한꺼번에 생성해야 하는 이유**

   - 타입스크립트에서는 객체를 생성할 때 속성을 한꺼번에 정의하면 타입 체커가 올바르게 작동한다.
   - 속성을 나눠 추가하는 방식은 타입 체커가 객체의 구조를 이해하지 못하게 되어 오류를 발생시킨다.
     ```typescript
     const pt = {};
     pt.x = 3; // 오류: '{}' 형식에 'x' 속성이 없습니다.
     pt.y = 4; // 오류: '{}' 형식에 'y' 속성이 없습니다.
     ```

2. **해결 방법**

   - 객체를 생성할 때 한 번에 속성을 정의하면 타입 체커가 오류를 발생시키지 않는다.
     ```typescript
     const pt = { x: 3, y: 4 };
     ```
   - 타입을 명시적으로 선언하거나 `as` 단언문을 사용해 타입 체커를 통과시킬 수도 있다.
     ```typescript
     const pt = {} as { x: number; y: number };
     pt.x = 3;
     pt.y = 4; // 정상
     ```

3. **객체 전개 연산자의 활용**

   - 작은 객체를 조합하여 큰 객체를 생성할 때는 객체 전개 연산자를 사용한다.
     ```typescript
     const pt = { x: 3, y: 4 };
     const id = { name: "Pythagoras" };
     const namedPoint = { ...pt, ...id }; // 정상
     ```
   - 조건부 속성 추가:
     ```typescript
     const optionalProperties = condition ? { optionalKey: value } : {};
     const combined = { ...baseObject, ...optionalProperties };
     ```

4. **유니온 타입과 선택적 필드 처리**

   - 조건부 속성 추가 시 유니온 타입이 생성되는 문제를 해결하기 위해 헬퍼 함수를 사용할 수 있다:
     ```typescript
     function addOptional<T extends object, U extends object>(
       base: T,
       optional: U | null
     ): T & Partial<U> {
       return { ...base, ...optional };
     }
     const result = addOptional(
       { name: "Alice" },
       condition ? { age: 30 } : null
     );
     ```

5. **요약**
   - 객체 생성 시 속성을 나눠 추가하면 오류 발생 가능.
   - 객체 전개 연산자를 활용해 조건부 속성 추가.
   - 유니온 타입 대신 선택적 필드를 활용하여 타입 안정성 유지.

---

#### **Item 24: 일관성 있는 별칭 사용하기**

1. **별칭의 기본 개념**

   - 별칭은 동일한 데이터를 참조하지만 다른 이름으로 접근하기 위해 생성한다.
   - 별칭을 통해 수정된 값은 원본 데이터에도 영향을 미친다.
     ```typescript
     const obj = { key: [1, 2, 3] };
     const alias = obj.key;
     alias[0] = 10;
     console.log(obj.key); // [10, 2, 3]
     ```

2. **별칭 남용의 문제점**

   - 별칭을 무분별하게 사용하면 제어 흐름 분석이 어려워지고 코드 오류가 발생할 가능성이 높아진다.
     ```typescript
     function process(polygon: { bbox?: [number, number] }) {
       const box = polygon.bbox;
       if (polygon.bbox) {
         // box의 타입은 여전히 'BoundingBox | undefined'로 남아 오류 발생.
       }
     }
     ```

3. **비구조화를 활용한 해결 방법**

   - 객체 비구조화를 통해 별칭 없이도 속성에 간결하게 접근 가능:
     ```typescript
     const { key } = obj;
     ```

4. **별칭 사용 시의 주의사항**

   - 속성보다 지역 변수를 활용해 타입 정제를 유지하는 것이 안전하다:
     ```typescript
     const box = polygon.bbox;
     if (box) {
       if (point.x < box.x[0] || point.x > box.x[1]) return false;
     }
     ```

5. **요약**
   - 별칭은 최소한으로 사용하고 일관성을 유지.
   - 객체 비구조화를 통해 가독성을 높이고 오류를 줄인다.
   - 속성보다 지역 변수를 활용하여 타입 정제를 유지.

---

#### **Item 25: 비동기 코드에는 콜백 대신 async 함수 사용하기**

1. **콜백의 한계**

   - 콜백 방식은 중첩이 깊어지며 "콜백 지옥"을 유발한다.
     ```typescript
     fetch(url1, (response1) => {
       fetch(url2, (response2) => {
         fetch(url3, (response3) => {
           console.log(response3);
         });
       });
     });
     ```

2. **`Promise`의 도입**

   - `Promise`는 비동기 흐름을 단순화하며, 중첩된 콜백을 해결한다:
     ```typescript
     fetch(url1)
       .then((response1) => fetch(url2))
       .then((response2) => fetch(url3))
       .then((response3) => console.log(response3));
     ```

3. **`async/await`의 활용**

   - `async`와 `await`를 사용하면 더욱 간결하고 직관적인 코드를 작성할 수 있다:
     ```typescript
     async function fetchData() {
       const response1 = await fetch(url1);
       const response2 = await fetch(url2);
       console.log(response1, response2);
     }
     ```

4. **병렬 작업**

   - `Promise.all`을 사용하면 여러 비동기 작업을 병렬로 실행 가능:
     ```typescript
     const [data1, data2] = await Promise.all([fetch(url1), fetch(url2)]);
     ```

5. **요약**
   - 콜백보다 `async/await`가 가독성과 유지보수성에서 유리.
   - 병렬 작업은 `Promise.all`을 활용해 성능 향상.

---

#### **Item 26: 타입 추론에 문맥이 어떻게 사용되는지 이해하기**

1. **문맥 기반 타입 추론의 중요성**

   - 타입스크립트는 값만이 아니라 사용되는 문맥까지 고려해 타입을 추론한다.
   - 변수로 분리하면 예상치 못한 타입 오류가 발생할 수 있다:
     ```typescript
     type Language = "JavaScript" | "TypeScript";
     function setLanguage(lang: Language) {}
     let lang = "JavaScript"; // string으로 추론
     setLanguage(lang); // 오류 발생
     ```

2. **문제 해결 방법**

   - 타입 선언 추가:
     ```typescript
     let lang: Language = "JavaScript";
     ```
   - 상수 사용:
     ```typescript
     const lang = "JavaScript";
     ```

3. **튜플 및 객체 사용 시 주의점**

   - 튜플을 사용할 경우 `as const`를 활용하거나 명시적으로 선언해야 한다:
     ```typescript
     const loc = [10, 20] as const;
     function panTo(coords: readonly [number, number]) {}
     panTo(loc);
     ```

4. **콜백 사용 시 주의점**

   - 문맥이 소실되지 않도록 매개변수 타입을 명시적으로 선언해야 한다.

5. **요약**
   - 타입 선언과 `as const`를 통해 문맥을 명확히 하여 타입 오류 방지.
   - 콜백 함수의 타입 명시를 통해 타입 안전성을 유지.

---

#### **Item 27: 함수형 기법과 라이브러리로 타입 흐름 유지하기**

1. **함수형 프로그래밍의 이점**

   - 함수형 프로그래밍 기법은 데이터 처리와 타입 관리를 간결하고 명확하게 만든다.
   - 로대시(Lodash)와 같은 라이브러리를 활용하면 타입 정보가 잘 유지된다:
     ```typescript
     import _ from "lodash";
     const processed = _.map(data, (item) => item.property);
     ```

2. **활용 예시**

   - 데이터 필터링 및 매핑:
     ```typescript
     const filtered = data
       .filter((item) => item.isActive)
       .map((item) => item.name);
     ```

3. **요약**
   - 함수형 프로그래밍은 타입 안정성과 가독성을 동시에 제공.
   - 로대시 등 라이브러리를 적극 활용하여 생산성을 높인다.

---

### 깨달은 점

- **비동기 작업 최적화**  
  `async/await`와 `Promise.all`의 차이를 이해하면서, 병렬 작업과 직렬 작업의 성능 차이를 알게 되었다. 이를 통해 병렬로 처리할 수 있는 작업은 적극적으로 병렬화를 도입해야 한다고 느꼈다.
- **타입 선언의 중요성**  
  명시적인 타입 선언과 문맥 기반 타입 추론의 차이를 배우며, 타입스크립트의 타입 안정성을 최대한 활용하려면 선언을 명확히 해야 한다는 점을 깨달았다.
