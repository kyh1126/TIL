# 02. Functions — let, fn과 렉시컬 스코프

- 학습일: 2026-11-11
- 교재: [Clojure Learn — Functions](https://clojure.org/guides/learn/functions)
- 참고: [Clojure Special Forms — let](https://clojure.org/reference/special_forms)
- 목표: `defn`, `let`, `fn`을 읽고 작성하며, 같은 이름의 바인딩이 중첩될 때 어떤 값을 참조하는지 이해한다.
- 실습 결과: **2/3 정답 (67점 / 100점)** — 스코프·섀도잉 재학습 필요

## 1. let — 지역 바인딩

Java 코드:

```java
public int calculate(int price, int quantity) {
    int subtotal = price * quantity;
    int discount = 1000;
    return subtotal - discount;
}
```

Clojure 코드:

```clojure
(defn calculate [price quantity]
  (let [subtotal (* price quantity)
        discount 1000]
    (- subtotal discount)))
```

- `let`의 대괄호 안에는 `이름 값` 순서로 지역 바인딩을 만든다.
- 뒤의 바인딩은 앞에서 정의한 지역 바인딩을 참조할 수 있다.
- 본문에서 마지막으로 평가된 표현식이 `let`의 결과가 된다.
- `let`으로 만든 지역 바인딩의 값을 재할당할 수 없다.

## 2. Scope — 재할당과 섀도잉은 다르다

### 재할당 불가

```clojure
(let [x 10]
  (set! x 20))
;; 컴파일 오류: let 지역 바인딩을 set!으로 변경할 수 없음
```

### 새 스코프에서 같은 이름을 바인딩하는 것은 가능

```clojure
(let [x 10
      y (+ x 5)]
  (let [x 100]
    (+ x y)))
;; => 115
```

1. 바깥쪽 `x = 10`.
2. 바깥쪽 `y = (+ 10 5) = 15`.
3. 안쪽 `let`은 **새 `x = 100` 바인딩을 생성**한다. 기존 `x`를 수정하지 않는다.
4. 안쪽 본문에서 `x`는 가장 가까운 `100`, `y`는 바깥의 `15`를 참조한다.
5. 최종 결과는 `(+ 100 15) = 115`.

이처럼 같은 이름의 바깥쪽 바인딩을 안쪽 바인딩이 가리는 것을 **섀도잉(Shadowing)** 이라고 한다.

| 구분 | 의미 | Clojure let에서 |
| --- | --- | --- |
| 재할당 | 기존 바인딩의 값을 수정 | 불가 |
| 섀도잉 | 새로운 렉시컬 스코프에서 같은 이름을 바인딩 | 가능 |

**Java와 비교:** Java는 같은 메서드의 중첩 블록에서 기존 지역 변수와 동일한 이름을 다시 선언하는 것이 금지될 수 있으므로, Java의 지역 변수 선언 규칙을 Clojure에 그대로 적용하면 안 된다.

## 3. fn — 익명 함수

```clojure
(fn [x] (* x 2))
;; 이름 없는 함수

(def double-value (fn [x] (* x 2)))
(double-value 5)
;; => 10
```

단순 익명 함수는 축약 문법 `#()`를 사용할 수 있다.

```clojure
#(* % 2)    ; %: 첫 번째 인자
#(+ %1 %2)  ; %1, %2: 첫 번째, 두 번째 인자
```

Java의 람다와 비슷하게 함수를 값처럼 전달할 수 있다.

## 4. 실습 답안과 피드백

### 문제 1 — let 계산

```clojure
(let [price 10000
      quantity 3
      subtotal (* price quantity)
      discount 5000]
  (- subtotal discount))
;; => 25000
```

- 제출 답안: `25000`
- 판정: **정답**
- 이해한 개념: 앞선 바인딩 참조, 마지막 표현식 반환

### 문제 2 — 중첩 let의 스코프

```clojure
(let [x 10
      y (+ x 5)]
  (let [x 100]
    (+ x y)))
;; => 115
```

- 제출 답안: "에러, let은 지역변수 재할당하면 에러난다고 했었음."
- 판정: **오답**
- 오개념: 동일한 이름의 **새 바인딩 생성**을 **기존 값의 재할당**으로 혼동했다.
- 올바른 이해: 안쪽 `let`은 바깥쪽 `x`의 값을 바꾸지 않고, 내부에서 참조할 새로운 `x`를 정의한다. `y`는 기존 `15`다.

### 문제 3 — Java 함수를 Clojure로 변환

**직접 작성한 코드(정답):**

```clojure
(defn calculate-final-price [price quantity discount]
  (let [subtotal (* price quantity)
        finalPrice (- subtotal discount)]
    finalPrice))
```

- 판정: **정답** (문법과 계산 흐름 모두 정상).
- 개선 권장: Clojure 관례상 `finalPrice`보다 `final-price`를 선호한다.

**권장 스타일:**

```clojure
(defn calculate-final-price [price quantity discount]
  (let [subtotal (* price quantity)
        final-price (- subtotal discount)]
    final-price))
```

## 5. Java/Spring Boot 마이그레이션 관점

- 중첩된 `let`에서 **심볼이 실제로 어떤 스코프의 값을 참조하는지** 확인한다.
- 변수 이름만 보고 전역/지역 상태가 변경된다고 판단하지 않는다.
- `let`의 새 바인딩을 Java로 옮길 때는 별도 스코프 또는 명확히 구분된 변수명으로 표현해 의미를 보존한다.
- `fn`과 함수 반환값·클로저가 기존 상태나 부작용과 얽혀 있는지 확인한다.

## 6. 보충 복습 문제 (아직 제출 전)

### 복습 A

```clojure
(let [x 5
      y (* x 2)]
  (let [x 20]
    (+ x y)))
```

반환값과 그 이유를 설명한다.

### 복습 B

```clojure
(let [x 10]
  (let [x (+ x 5)]
    x))
```

안쪽 `x`의 초기값 계산에서 어느 `x`를 읽는지 설명한다.

### 복습 C

```clojure
(let [x 10]
  (let [x 100]
    x)
  x)
```

마지막 표현식의 반환값과 이유를 설명한다.

## 7. 학습 체크리스트

- [x] `let` 기본 계산 및 바인딩 순서
- [x] `defn`으로 함수 작성
- [x] `let`과 마지막 표현식으로 값 반환
- [ ] **섀도잉 vs 재할당**을 예측 문제에서 정확히 구분
- [ ] `fn` 익명 함수 직접 작성
- [ ] 복습 문제를 통과한 뒤 3강 Sequential Collections로 이동

## 참고

- [Learn Clojure — Functions](https://clojure.org/guides/learn/functions)
- [Clojure Special Forms](https://clojure.org/reference/special_forms)

## 8. 보충 복습 채점 — 3/3 정답 (100점)

처음 혼동했던 **재할당 vs 섀도잉** 개념을 세 문제 모두 맞히며 교정했습니다.

- **복습 A: 30 (정답)** — 바깥쪽 `y = 10`, 안쪽 `x = 20`이므로 `(+ x y)`는 30.
- **복습 B: 15 (정답)** — 안쪽 `x`의 초기값 `(+ x 5)`는 바깥쪽 `x = 10`을 읽고, 새 `x = 15`를 바인딩.
- **복습 C: 10 (정답)** — 안쪽 `let`의 결과 100이 아니라 바깥쪽 `let`의 마지막 표현식 `x = 10`이 반환됨.

**핵심 정리**

- 재할당은 이미 있는 지역 바인딩의 값을 바꾸는 것이며 `let`에서는 불가.
- 섀도잉은 새 렉시컬 스코프에서 같은 이름의 새 바인딩을 만들어 바깥쪽 이름을 가리는 것으로 가능.
- 최종 반환값은 해당 표현식의 마지막 평가 결과.

**학습 상태:** 2강 스코프 복습 통과. 3강 Sequential Collections 진도 진행. 익명 함수 `fn` 직접 작성은 추후 검증.
