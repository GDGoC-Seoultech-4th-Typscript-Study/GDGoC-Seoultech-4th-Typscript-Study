# Ch.4 - 타입 설계 (1/2)

<aside>
💡

학습목표

- 유효한 상태만 허용하는 타입 설계하는 법
- 입력 타입은 유연하고 출력 타입은 엄격하게 설계하는 법
- 주석보다 타입 시스템을 활용하여 오류를 줄이는 방법
- 의도하지 않은 null 참조 오류를 방지하는 방법
- 런타임에 타입의 범위를 좁히는 태그된 유니온을 사용하는 방법
</aside>

---

## Item 28. 유효한 상태만 표현하는 타입을 지향하기

잘 설계된 타입은 직관적이고 유지보수가 용이한 코드를 작성할 수 있게 한다. 반대로, 설계가 잘못된 타입은 코드의 복잡성을 증가시키고, 오류를 유발할 가능성을 높인다. 따라서 **유효한 상태만 표현하는 타입**을 만드는 것이 중요하다.

### 문제 상황: 잘못된 상태 표현

- 잘못된 설계의 예: 웹 애플리케이션에서 페이지 상태를 표현하는 코드
    
    ```tsx
    interface State {
      pageText: string;
      isLoading: boolean;
      error?: string;
    }
    ```
    
    - `isLoading`이 `true`인데 `error`도 설정되어 있다면, 로딩 중인지 오류 상태인지 명확하지 않다.
    - 특정 조건에서 상태가 충돌하거나 필요한 정보가 부족할 수 있다.
- 문제점
    1. **모호한 상태:** `renderPage`에서 상태 분기를 명확히 처리하기 어렵습니다.
    2. **변경 중 버그:** `changePage`에서 상태를 업데이트하는 동안, 이전 오류 메시지가 화면에 표시될 가능성이 있습니다.
    3. **동시 요청 충돌:** 페이지 로딩 중 다른 페이지로 전환하면, 결과가 비정상적으로 동작할 수 있습니다.

### 개선: 유효한 상태만 표현하는 타입

- 개선된 설계
    
    ```tsx
    interface RequestPending {
      state: 'pending';
    }
    interface RequestError {
      state: 'error';
      error: string;
    }
    interface RequestSuccess {
      state: 'ok';
      pageText: string;
    }
    
    type RequestState = RequestPending | RequestError | RequestSuccess;
    
    interface State {
      currentPage: string;
      requests: { [page: string]: RequestState };
    }
    ```
    
    - 태그된 유니온(구별된 유니온)을 사용하여 상태를 명확히 표현
    - 이 설계는 무효한 상태를 제거하며, 모든 상태를 명확히 구분
- 개선된 `renderPage` 함수
    
    ```tsx
    function renderPage(state: State) {
      const { currentPage } = state;
      const requestState = state.requests[currentPage];
    
      switch (requestState.state) {
        case 'pending':
          return `Loading ${currentPage}...`;
        case 'error':
          return `Error! Unable to load ${currentPage}: ${requestState.error}`;
        case 'ok':
          return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
      }
    }
    ```
    
- 개선된 `changePage` 함수
    
    ```tsx
    async function changePage(state: State, newPage: string) {
      state.requests[newPage] = { state: 'pending' };
      state.currentPage = newPage;
    
      try {
        const response = await fetch(getUrlForPage(newPage));
        if (!response.ok) {
          throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
        }
        const pageText = await response.text();
        state.requests[newPage] = { state: 'ok', pageText };
      } catch (e) {
        state.requests[newPage] = { state: 'error', error: String(e) };
      }
    }
    ```
    

---

## Item 29.  사용할 때는 너그럽게, 생성할 때는 엄격하게

### 함수 타입의 설계 원칙

1. **매개변수 타입**은 호출의 편리성을 위해 **범위를 넓게** 설계해야
2. **반환 타입**은 반환된 값을 안전하게 사용할 수 있도록 **엄격하고 구체적으로** 설계해야
- 예제: 3D 매핑 API
    
    ```tsx
    declare function setCamera(camera: CameraOptions): void;
    declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;
    ```
    

### 타입 설계: 매개변수 타입의 유연성과 반환 타입의 구체성

- CameraOptions와 LngLat 타입 정의
    
    ```tsx
    interface CameraOptions {
      center?: LngLat;
      zoom?: number;
      bearing?: number;
      pitch?: number;
    }
    
    type LngLat =
      | { lng: number; lat: number }
      | { lon: number; lat: number }
      | [number, number];
    ```
    
    - **CameraOptions**
        - 모든 속성은 선택적(optional)
        - 일부 값만 수정 가능하도록 유연성을 제공
    - **LngLat**
        - 다양한 형태({lng, lat}, {lon, lat}, [lng, lat])를 지원하여 사용성을 향상
- LngLatBounds 정의
    
    ```tsx
    type LngLatBounds =
      | { northeast: LngLat; southwest: LngLat }
      | [LngLat, LngLat]
      | [number, number, number, number];
    ```
    
    - 경계(`bounds`)를 표현할 때, 이름 있는 객체, 좌표 쌍, 숫자 튜플을 모두 허용하여 편의성을 향상

### 문제점: 타입의 불명확성과 개선 방안

- 문제 예제:
    
    ```tsx
    function focusOnFeature(f: Feature) {
      const bounds = calculateBoundingBox(f);
      const camera = viewportForBounds(bounds);
      setCamera(camera);
      
      const { center: { lat, lng }, zoom } = camera; // 오류 발생
      window.location.search = `?v=@${lat},${lng}z${zoom}`;
    }
    ```
    
    - `viewportForBounds`의 반환 타입이 너무 자유롭기 때문에 `lat`, `lng` 속성이 없는 경우 발생.
    - `zoom`의 타입도 `number | undefined`로 추론되어 불편함이 발생.
- 개선된 예제:
    
    ```tsx
    interface Camera {
      center: LngLat;
      zoom: number;
      bearing: number;
      pitch: number;
    }
    
    interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
      center?: LngLatLike;
    }
    
    type LngLatLike = LngLat | { lon: number; lat: number } | [number, number];
    ```
    
    - 반환 타입을 `Camera`로 정의하여 **엄격한 형태**를 보장.
    - 매개변수 타입은 `CameraOptions`로 정의하여 호출의 유연성을 유지.

---

## Item 30. 문서에 타입 정보를 쓰지 않기

### 잘못된 예제 코드

- 잘못된 예제 코드
    
    ```tsx
    /**
     * 전경색(foreground) 문자열을 반환합니다.
     * 0개 또는 1개의 매개변수를 받습니다.
     * 매개변수가 없을 때는 표준 전경색을 반환합니다.
     * 매개변수가 있을 때는 특정 페이지의 전경색을 반환합니다.
     */
    function getForegroundColor(page?: string) {
        return page === 'login' ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
    }
    ```
    
    - **반환 타입 오류**: 주석은 문자열을 반환한다고 되어 있지만, 실제 코드는 `{ r, g, b }` 객체를 반환합니다.
    - **불필요한 정보**: 함수의 매개변수 수와 같은 정보는 코드만 봐도 알 수 있으며, 주석으로 반복할 필요가 없습니다.
    - **장황함**: 주석이 구현체보다 길고 복잡합니다.

### 개선된 예제

- 개선된 예제
    
    ```tsx
    /**
     * 애플리케이션 또는 특정 페이지의 전경색을 가져옵니다.
     * @param page - 페이지 이름
     * @returns 전경색 객체 { r, g, b }
     */
    function getForegroundColor(page?: string): { r: number; g: number; b: number } {
        return page === 'login' ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
    }
    ```
    
    - **JSDoc 사용**: 필요한 경우 `@param`이나 `@returns` 태그를 사용하여 매개변수와 반환값을 설명합니다.
    - **타입 시스템 활용**: 반환 타입을 코드에 명시하여, 타입 정보를 중복으로 적는 일을 피합니다.

---

## Item 31. 타입 주변에 **null** 값 배치하기

### Null 처리의 어려움

- `strictNullChecks` 옵션을 활성화하면, null과 undefined 관련 오류가 코드 전반에서 발생할 수 있음.
- 모든 변수가 null이 될 가능성을 코드로 명시하지 않으면, 많은 오류가 나타남.
- 한 변수의 null 여부가 다른 변수의 null 여부와 암묵적으로 연결되는 경우, 코드가 복잡해지고 버그를 유발할 수 있음.

### **예제: `extent` 함수 개선**

- **문제 상황**: 숫자 배열의 최솟값과 최댓값을 계산하는 `extent` 함수가 제대로 동작하지 않음.
    
    ```tsx
    function extent(nums: number[]) {
      let min, max;
      for (const num of nums) {
        if (!min) { // 최솟값이 0이면 조건이 false로 평가됨
          min = num;
          max = num;
        } else {
          min = Math.min(min, num); // 오류: min이 undefined일 가능성 존재
          max = Math.max(max, num);
        }
      }
      return [min, max]; // 반환 타입이 [number | undefined, number | undefined]
    }
    ```
    
    - `0` 값이 올바르게 처리되지 않음.
    - 빈 배열을 처리할 때 `[undefined, undefined]`를 반환하여 문제가 발생.
    - `min`과 `max`가 항상 동시에 null 또는 null이 아니지만, 이를 타입으로 표현하지 못함.
- 해결책: Null 값을 명시적으로 처리
    
    ```tsx
    function extent(nums: number[]): [number, number] | null {
      let result: [number, number] | null = null;
      for (const num of nums) {
        if (!result) {
          result = [num, num];
        } else {
          result = [Math.min(result[0], num), Math.max(result[1], num)];
        }
      }
      return result;
    }
    ```
    
    - 반환 타입을 `[number, number] | null`로 설정하여 null 여부를 명확히 함.
    - 한 번에 두 값을 동시에 초기화하는 방식으로 설계를 단순화.

---

## Item 32.  유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

### 문제점: 유니온 타입의 속성을 가지는 인터페이스

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

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- `layout`이 `LineLayout`인데 `paint`가 `FillPaint`라면 의미가 불분명
- 이런 조합은 오류를 유발하고 다루기 어려움

### 해결책1: 인터페이스의 유니온으로 표현

- 예시: 인터페이스 분리
    
    ```tsx
    interface FillLayer {
      layout: FillLayout;
      paint: FillPaint;
    }
    interface LineLayer {
      layout: LineLayout;
      paint: LinePaint;
    }
    interface PointLayer {
      layout: PointLayout;
      paint: PointPaint;
    }
    
    type Layer = FillLayer | LineLayer | PointLayer;
    ```
    
    - 속성을 분리하고 각 타입에 대해 별도의 인터페이스를 정의

### 해결책2: 태그된 유니온 사용

- 태그(`type` 속성)를 추가하면 런타임에 어떤 타입인지 명확히 알 수 있음
- 타입스크립트의 **제어 흐름 분석**과도 잘 맞음
- 예시:
    
    ```tsx
    interface FillLayer {
      type: 'fill';
      layout: FillLayout;
      paint: FillPaint;
    }
    interface LineLayer {
      type: 'line';
      layout: LineLayout;
      paint: LinePaint;
    }
    interface PointLayer {
      type: 'point';
      layout: PointLayout;
      paint: PointPaint;
    }
    
    type Layer = FillLayer | LineLayer | PointLayer;
    
    function drawLayer(layer: Layer) {
      if (layer.type === 'fill') {
        // 타입이 FillLayer로 좁혀짐
        const { paint, layout } = layer;
      } else if (layer.type === 'line') {
        // 타입이 LineLayer로 좁혀짐
        const { paint, layout } = layer;
      } else {
        // 타입이 PointLayer로 좁혀짐
        const { paint, layout } = layer;
      }
    }
    ```
    
    - `type` 속성 덕분에 코드 내에서 특정 타입으로 좁혀서 작업할 수 있음
    - 타입스크립트가 타입 간 관계를 자동으로 추론하고 체크할 수 있음

---

## 궁금한 점 - **유니온 타입**과 **인터페이스의 유니온**의 설계 차이

### 1. **유니온 타입의 설계**

유니온 타입에서는 여러 타입이 결합되어 **각 속성의 값**이 유니온으로 구성

- 예시: 유니온 타입
    
    ```tsx
    interface Layer {
      layout: FillLayout | LineLayout | PointLayout;
      paint: FillPaint | LinePaint | PointPaint;
    }
    ```
    
    - `layout`과 `paint` 속성 간의 관계가 강제되지 않음.
    - 잘못된 조합(`layout`이 `FillLayout`이고 `paint`가 `LinePaint`인 경우 등)을 허용하기 쉬움.
    - **타입 간의 관계를 모델링하기 어려움**.

### 2. 인터페이스의 유니온 설계

인터페이스의 유니온은 각 타입의 속성을 별도의 인터페이스로 분리하고, 이를 유니온으로 합침

- 예시: 인터페이스의 유니온
    
    ```tsx
    interface FillLayer {
      type: 'fill';
      layout: FillLayout;
      paint: FillPaint;
    }
    
    interface LineLayer {
      type: 'line';
      layout: LineLayout;
      paint: LinePaint;
    }
    
    interface PointLayer {
      type: 'point';
      layout: PointLayout;
      paint: PointPaint;
    }
    
    type Layer = FillLayer | LineLayer | PointLayer;
    ```
    
    - 각 타입(`FillLayer`, `LineLayer`, `PointLayer`)이 **명확하게 정의**되고, 서로 잘못된 조합이 발생하지 않음.
    - `type` 속성을 통해 **태그된 유니온**으로 특정 타입을 좁힐 수 있어 코드 가독성과 타입 안정성이 높아짐.

### 3. **차이점 비교**

| 특징 | 유니온 타입 | 인터페이스의 유니온 |
| --- | --- | --- |
| **타입 간 관계 표현** | 속성 간의 관계를 명시적으로 표현하기 어려움. | 속성 간의 관계를 명확하게 모델링 가능. |
| **잘못된 조합 방지** | 잘못된 조합을 방지하지 못함. | 인터페이스별로 속성이 고정되므로 잘못된 조합 방지. |
| **타입 좁히기 지원** | 타입 좁히기가 어렵거나 복잡해짐. | 태그(`type`)를 이용해 쉽게 좁힐 수 있음. |
| **확장성** | 새로운 타입 추가 시 기존 코드 수정이 필요할 수 있음. | 새로운 인터페이스를 추가하기 쉽고 코드 수정이 적음. |
| **복잡성** | 간단하지만 관계가 복잡하면 오류가 생길 가능성이 큼. | 초기 설정은 복잡할 수 있지만 안전한 설계. |
