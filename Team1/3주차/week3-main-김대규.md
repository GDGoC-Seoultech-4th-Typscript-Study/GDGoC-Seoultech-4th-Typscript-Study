## 1. 타입과 인터페이스의 차이점 알기

타입스크립트에서는 타입과 인터페이스를 사용해 명명된 타입(named type)을 정의할 수 있음. 두 가지는 유사한 기능을 제공하지만, 사용 시 차이점을 이해하고 일관성을 유지하는 것이 중요함.

### 공통점

- **타입 정의**: 타입(`type`)과 인터페이스(`interface`)는 모두 객체 구조와 값을 명확히 정의할 수 있음.
- **인덱스 시그니처**: `[key: string]: type`과 같은 방식으로 객체의 동적 키와 값을 정의할 수 있음.
- **함수 타입**: 함수의 타입을 정의할 수 있으며, 이때 인터페이스와 타입 모두 활용할 수 있음.
- **제너릭 지원**: 타입과 인터페이스 모두 제너릭을 통해 타입을 동적으로 설정할 수 있음.
- **확장 가능성**: 인터페이스와 타입은 서로 확장 가능하며, 클래스에서도 사용할 수 있음.

### 차이점

- **유니온 타입**: 타입은 유니온 타입을 표현할 수 있지만, 인터페이스는 유니온 타입 확장이 불가함. 복잡한 타입이 필요할 경우 타입이 더 유리함.

```tsx
type AorB = "a" | "b";
```

- **선언 병합**: 인터페이스는 선언 병합(Declaration Merging)을 지원하여 보강(Augmentation)이 가능하지만, 타입에서는 불가능함.
  ```tsx
  interface IState {
    name: string;
    capital: string;
  }

  interface IState {
    population: number;
  }

  const Wyoming: IState = {
    name: "Wyoming",
    capital: "Cheyenne",
    population: 500_000,
  };
  ```
- **튜플 및 배열 지원**: 타입은 튜플과 배열을 간결하게 표현할 수 있으며, 인터페이스는 배열과 같이 메서드 사용에 제한이 있을 수 있어 튜플이나 배열의 경우 타입으로 구현하는 것이 좋음.
  ```tsx
  type Pair = [number, number];
  type StringList = string[];
  type NamedNums = [string, ...number[]];
  ```

## 2. 타입 연산과 제너릭 사용으로 반복 줄이기

코드의 중복을 줄이고 유지보수를 간편하게 하기 위해 DRY(Don't Repeat Yourself) 원칙을 준수함. 타입스크립트에서도 타입을 활용해 반복을 줄일 수 있음.

- **타입에 이름 붙이기**: 타입이나 함수의 입력값에 대해 반복이 발생하는 경우, 시그니처를 분리해 이름을 붙여 중복을 제거할 수 있음.
  ```tsx
  interface Point2D {
    x: number;
    y: number;
  }

  function distance(a: Point2D, b: Point2D) { /*...*/
  ```
- **인터페이스 확장**: 인터페이스 확장을 통해 중복된 속성을 가진 타입을 효과적으로 관리할 수 있음.
  ```tsx
  interface Person {
    firstName: string;
    lastName: string;
  }

  interface PersonWithBirthDate extends Person {
    birth: Date;
  }
  ```
- **제너릭 사용**: 제너릭은 타입을 동적으로 처리할 수 있는 타입 매개변수로, 반복되는 코드를 줄이는 데 유용함. 예를 들어, `type Pair<T> = { first: T; second: T; }`와 같이 제너릭을 사용하면 타입 재사용이 가능함.
  ```tsx
  type TPair<T> = {
    first: T;
    second: T;
  };

  interface IPair<T> {
    first: T;
    second: T;
  }
  ```
- **매핑된 타입**: `Pick`과 같이 매핑된 타입을 사용하여 중복된 부분을 제거하고 동적으로 타입을 매핑할 수 있음.
  ```tsx
  type TopNavState = {
    [k in "userid" | "pageTitle" | "recentFiles"]: State[k];
  };
  ```

## 3. 동적 데이터에 인덱스 시그니처 사용하기

동적 키와 값을 다룰 때는 인덱스 시그니처를 사용함. 인덱스 시그니처는 객체의 키가 문자열이거나 동적으로 변할 때 사용됨. 예를 들어, `[property: string]: string`과 같은 형식으로 키와 값을 정의할 수 있음.

### 인덱스 시그니처의 장단점

- **장점**: 런타임까지 키의 타입을 예측할 수 없을 때 유용하게 사용할 수 있으며, 특히 CSV 파일 등에서 헤더와 데이터를 매핑할 때 활용할 수 있음.
  ```tsx
  function parseCSV(input: string): { [columnName: string]: string }[] {
    const lines = input.split("\n");
    const [header, ...rows] = lines;
    const headerColumns = header.split(",");

    return rows.map((rowStr) => {
      const row: { [columnName: string]: string } = {};
      rowStr.split(",").forEach((cell, i) => {
        row[headerColumns[i]] = cell;
      });
      return row;
    });
  }
  ```
- **단점**:
  - 모든 키를 허용하기 때문에 실수로 잘못된 키를 포함할 수 있음.
  - 각 키에 대해 타입을 다르게 지정할 수 없고, 자동 완성 기능도 제대로 동작하지 않음.
  - 특정 키가 반드시 있어야 한다는 보장이 없음. 이러한 한계를 보완하기 위해 인덱스 시그니처 대신 인터페이스 사용을 고려할 수 있음.

## 4. Array, Tuple, ArrayLike로 동적 데이터 표현하기

JavaScript 객체는 키를 문자열로 처리하기 때문에, 타입스크립트에서는 숫자 인덱스를 `number`로 인식할 수 있지만 실제로는 문자열로 변환되어 사용됨. 이와 같은 혼란을 줄이기 위해 타입스크립트에서는 숫자 키를 사용하는 경우 Array, Tuple, ArrayLike 타입을 사용할 것을 권장함.

### 사용 예시

- **배열 순회**: 배열 순회 시에도 `for-in` 대신 `for-of` 또는 `Array.prototype.forEach` 등을 사용하여 인덱스 오류를 방지하는 것이 좋음.
  ```tsx
  const xs = [1, 2, 3];

  for (const x of xs) {
    console.log(x); // number 타입
  }

  xs.forEach((x, i) => {
    console.log(i, x); // i는 number, x는 number 타입
  });
  ```

## 5. 변경 관련된 오류 방지를 위한 readonly 사용하기

**readonly**는 값이 수정되지 않도록 보장하며, 특정 값이나 타입을 안전하게 유지하기 위해 사용함. `readonly`를 통해 값이 변경되는 실수를 방지할 수 있음.

- **readonly 배열과 객체**: 배열과 객체의 요소가 읽기 전용으로 지정되어 수정할 수 없음. 예를 들어 `readonly number[]`는 변경할 수 없는 숫자 배열을 의미함.
  ```tsx
  function arraySum(arr: readonly number[]): number {
    let sum = 0;
    for (const num of arr) {
      sum += num;
    }
    return sum;
  }
  ```
- **readonly와 const**: `const`는 변수의 재할당을 방지하고 참조는 허용하지만 내부 속성의 변경은 허용함. 반면 `readonly`는 참조된 객체 자체의 변경을 허용하지 않음.
  ```tsx
  const arr: readonly number[] = [1, 2, 3];
  ```

## 6. 매핑된 타입을 사용하여 값을 동기화하기

UI 컴포넌트에서 속성 값의 동기화가 필요한 경우 매핑된 타입을 사용하면 유용함. 매핑된 타입은 객체의 속성을 동적으로 정의할 수 있게 하여, 유지보수에 유리하고 새로운 속성 추가 시 타입 체커가 자동으로 검증할 수 있게 함.

### 최적화 접근법

1. **보수적 접근**: 모든 속성이 변경될 때마다 다시 그리도록 하는 접근으로, 정확하지만 불필요하게 자주 렌더링할 수 있음.
2. **실패에 열린 접근**: 일부 속성 변경 시 렌더링을 제한하는 방법으로, 렌더링 횟수를 줄일 수 있지만 특정 상황에서 누락될 가능성이 있음.
3. **매핑된 타입 사용**: 매핑된 타입은 속성별로 업데이트가 필요한지 여부를 명시할 수 있으므로, 매핑된 타입을 사용해 새로운 속성이 추가될 때 자동으로 업데이트 여부를 검증할 수 있음.

```tsx
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps): boolean {
  for (let k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

이처럼 매핑된 타입을 사용하여 필요할 때만 업데이트하고, 불필요한 렌더링을 줄일 수 있음.

---

## 궁금했던 점

인덱스 시그니쳐가 잘 이해가 되지 않아서 추가적으로 공부해봄

### 인덱스 시그니처에 대한 정리

인덱스 시그니처(Index Signature)는 TypeScript에서 객체의 키와 값의 타입을 동적으로 정의할 수 있는 기능임. 객체의 속성 이름과 그에 대응하는 값의 타입을 사전에 정확히 알 수 없을 때 유용하게 사용됨.

### 인덱스 시그니처의 기본 형태

인덱스 시그니처는 다음과 같은 형태로 정의함:

```tsx
interface SomeObject {
  [key: KeyType]: ValueType;
}
```

- `KeyType`은 인덱스의 타입으로, `string` 또는 `number`, `symbol`를 사용할 수 있음.
- `ValueType`은 해당 키에 할당될 값의 타입을 나타냄.

### 사용 예시

예를 들어, 다양한 속성을 가진 사용자 객체를 정의해야 하는 경우를 생각해볼 수 있음. 각 사용자는 여러 속성을 가질 수 있으며, 각 속성의 값은 문자열이어야 함. 이러한 경우 인덱스 시그니처를 사용하여 다음과 같이 표현할 수 있음:

```tsx
typescript
코드 복사
interface User {
  [propertyName: string]: string;
}

let user: User = {
  name: "John Doe",
  email: "john.doe@example.com",
  age: "30" // `age`는 문자열 타입이어야 함.
};

```

위 예시에서 `User` 인터페이스는 어떤 문자열 키에도 문자열 값이 할당될 수 있음을 명시함. 따라서 `user` 객체는 `name`, `email`, `age` 등 어떤 속성도 가질 수 있으며, 모든 값은 문자열이어야 함.

### 주의사항

인덱스 시그니처를 사용할 때 몇 가지 주의할 점이 있음:

1. **키 타입 제한**: `KeyType`은 `string` 또는 `number`, `symbol`만 가능함. `boolean` 등 다른 타입을 사용할 경우 에러가 발생함.

   ```tsx
   typescript
   코드 복사
   interface InvalidKeyType {
     [key: boolean]: string; // 오류 발생
   }

   ```

2. **동일한 키 타입 중복 선언 불가**: 동일한 키 타입의 인덱스 시그니처를 여러 번 선언할 수 없음.

   ```tsx
   typescript
   코드 복사
   interface DuplicateKeyType {
     [key: string]: string;
     [key: string]: number; // 오류 발생
   }

   ```

3. **일반 속성과의 공존**: 인덱스 시그니처와 일반 속성을 함께 사용할 수 있지만, 일반 속성의 타입은 인덱스 시그니처의 값 타입과 호환되어야 함.

   ```tsx
   typescript
   코드 복사
   interface MixedProperties {
     [key: string]: string;
     fixedProperty: string; // 허용됨
     anotherProperty: number; // 오류 발생
   }
   ```

4. **타입 안전성 감소**: 인덱스 시그니처는 유연성을 제공하지만, 사용 시 객체의 속성에 대한 타입 안전성이 감소할 수 있음. 가능한 한 구체적인 타입을 사용하는 것이 좋음.

### 결론

인덱스 시그니처는 객체의 구조가 유연하면서도 타입 안정성을 유지해야 할 때 매우 유용함. 이를 통해 더 동적이고 유연한 애플리케이션을 구축할 수 있으며, TypeScript의 강력한 타입 시스템을 최대한 활용할 수 있음

---

## 인증 사진

![IMG_4693.JPG](https://prod-files-secure.s3.us-west-2.amazonaws.com/1d0b7a97-ea6f-46d5-ac9a-a1cd951a497c/62491113-d206-43a5-b2df-5cd682f3fce0/IMG_4693.jpg)
