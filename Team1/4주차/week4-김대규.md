# TypeScript 4주차 학습 기록

## Item 19: 추론 가능한 타입을 사용해 장황한 코드 방지하기

1. **타입스크립트의 타입 추론**:

   - 타입스크립트는 코드를 작성할 때, 명시적으로 타입을 정의하지 않아도 변수를 선언하는 시점에 값의 타입을 추론함.
   - 이로 인해 타입 구문을 줄일 수 있어 코드의 가독성과 간결성을 높일 수 있음.

   ```typescript
   const axis1: string = "x"; // 명시적 타입 지정
   const axis2 = "y"; // 추론된 타입: 'y'
   ```

   - `axis2`는 "y"라는 문자열 값이 할당되었기 때문에, 타입스크립트가 이를 리터럴 타입으로 추론함.

2. **복잡한 객체 타입 추론**:

   - 객체를 다룰 때도 타입스크립트는 객체의 구조를 분석하여 자동으로 타입을 추론함.
   - 이를 통해 명시적인 타입 정의 없이도 안정적으로 코드를 작성할 수 있음.

   ```typescript
   interface Product {
     id: number;
     name: string;
     price: number;
   }

   function logProduct(product: Product) {
     const { id, name, price } = product; // 비구조화 할당
     console.log(id, name, price); // 모든 변수의 타입이 추론됨
   }
   ```

   - 비구조화 할당문으로 지역 변수(`id`, `name`, `price`)의 타입이 자동으로 추론됨.

3. **명시적 타입 구문의 필요성**:

   - 타입스크립트의 타입 추론은 매우 강력하지만, 때로는 명시적 타입 정의가 필요한 상황이 존재함.
   - 예를 들어, 함수의 매개변수나 반환값의 타입은 코드의 의도를 더 명확히 드러내기 위해 명시적으로 정의해야 할 수 있음.
   - 특히, 정보가 부족한 경우 추론이 잘못될 수 있음.

   ```typescript
   function calculateTotalPrice(price: number, quantity: number): number {
     return price * quantity;
   }
   ```

   - 위 예제에서 함수의 반환 타입은 `number`로 추론되지만, 이를 명시적으로 작성하면 코드의 의도가 명확해지고 유지 보수가 용이해짐.

---

## Item 20: 다형성 타입 대신 유니온 타입 사용하기

1. **다형성 타입의 한계**:

   - 다형성 타입은 매우 포괄적이며, 다양한 타입을 받아들이는 함수나 클래스를 정의할 때 유용함.
   - 하지만 지나치게 포괄적인 경우 예기치 않은 동작을 초래할 수 있음.

   ```typescript
   function identity<T>(value: T): T {
     return value;
   }
   ```

   - 위 함수는 모든 타입을 받아들일 수 있으므로 사용에 주의가 필요함.

2. **유니온 타입의 장점**:

   - 특정 상황에서 사용할 수 있는 타입을 제한하여 코드의 명확성과 안정성을 높일 수 있음.

   ```typescript
   function getLength(value: string | string[]): number {
     return Array.isArray(value) ? value.length : value.length;
   }
   ```

   - 이 함수는 문자열이나 문자열 배열만 입력받을 수 있음.
   - 이를 통해 함수가 처리할 수 있는 입력값의 범위를 명확히 정의함.

3. **타입 가드 활용**:

   - 조건문이나 타입 가드를 사용해 유니온 타입의 값을 세부적으로 처리할 수 있음.

   ```typescript
   function processInput(input: string | number) {
     if (typeof input === "string") {
       console.log(input.toUpperCase());
     } else {
       console.log(input.toFixed(2));
     }
   }
   ```

   - 타입 가드를 통해 각 타입에 적합한 로직을 구현할 수 있음.

---

## Item 21: 타입 간격 좁히기

1. **타입 좁히기 개념**:

   - 타입스크립트는 코드 실행 흐름에 따라 타입을 점진적으로 좁히며, 이를 통해 정확한 타입을 추론함.
   - 타입 단언문이나 조건문을 활용하면 명시적으로 타입을 좁힐 수 있음.

   ```typescript
   function handleEvent(event: MouseEvent | KeyboardEvent) {
     if ("key" in event) {
       console.log(event.key); // KeyboardEvent에서만 유효
     } else {
       console.log(event.clientX); // MouseEvent에서만 유효
     }
   }
   ```

2. **타입 단언문**:

   - 때로는 타입스크립트가 타입을 정확히 추론하지 못하는 경우가 있음.
   - 이럴 때 타입 단언문을 사용하여 타입을 명시적으로 지정할 수 있음.

   ```typescript
   const canvas = document.getElementById("canvas") as HTMLCanvasElement;
   const ctx = canvas.getContext("2d")!;
   ```

---

## Item 22: 타입스크립트와 자바스크립트의 호환성

1. **타입스크립트와 자바스크립트의 차이**:

   - 타입스크립트는 자바스크립트를 기반으로 하지만, 정적 타입 시스템을 제공하여 런타임 오류를 줄임.
   - 하지만 자바스크립트와의 호환성을 유지하기 위해, 때로는 타입스크립트가 런타임 오류를 잡지 못하는 경우가 있음.

2. **런타임 오류 방지**:

   - 타입 구문을 추가하거나 타입 가드를 활용해 런타임 오류를 방지할 수 있음.

   ```typescript
   interface Shape {
     kind: "circle" | "square";
     radius?: number;
     side?: number;
   }

   function calculateArea(shape: Shape): number {
     if (shape.kind === "circle") {
       return Math.PI * shape.radius! ** 2;
     } else {
       return shape.side! * shape.side!;
     }
   }
   ```

   - `!` 연산자는 null과 undefined를 제외하겠다는 의미로, 이를 통해 코드 안정성을 높임.

---

## 궁금했던 점

**질문**: "함수 매개변수에 항상 명시적으로 타입을 지정해야 하는가?"  
**해결**: 함수 매개변수 타입은 코드의 복잡성에 따라 다르게 접근해야 함. 간단한 함수의 경우 타입스크립트의 추론을 신뢰할 수 있지만, 복잡한 데이터 구조나 외부로 노출되는 API 함수에서는 명시적으로 타입을 지정하는 것이 좋음. 이는 코드의 의도를 명확히 드러내고, 협업 시 오류를 줄이는 데 도움을 줌.

---
