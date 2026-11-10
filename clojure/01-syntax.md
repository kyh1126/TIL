# 01. Clojure Syntax — 기본 문법과 함수 정의

- 학습일: 2026-11-11
- 공식 교재: [Clojure Syntax](https://clojure.org/guides/learn/syntax)
- 목표: Lisp 표기법으로 작성된 Clojure 코드를 읽고 간단한 함수를 직접 작성한다.
- 실습 결과: **3문제 모두 정답 (100/100)**

## 1. Clojure와 Lisp

Clojure는 JVM에서 실행되는 Lisp 계열의 언어로, Java와 상호운용할 수 있다. 함수와 데이터가 괄호로 표현되는 형태에 익숙해지는 것이 첫 번째 단계다.

## 2. 전위 표기법 (Prefix Notation)

Java는 중위 표기법을 사용한다.

```java
int result = (10 + 20) * 3;
```

Clojure는 함수를 앞에 배치한다.

```clojure
(* (+ 10 20) 3)
;; => 90
```

- `(+ 10 20)` → 30
- `(* 30 3)` → 90
- 기본적으로 `(함수 인자1 인자2 ...)` 형태로 함수를 호출한다.

## 3. 괄호와 자료구조

| 표기 | 의미 | 예시 |
| --- | --- | --- |
| `( )` | 함수 호출·특수 형식·리스트 | `(+ 1 2)` |
| `[ ]` | 벡터·함수 매개변수 | `[10 20 30]` |
| `{ }` | 맵 | `{:name "Kim" :age 30}` |
| `#{ }` | 집합 | `#{1 2 3}` |

`(1 2 3)`은 인용 없이 평가하면 첫 요소 `1`을 함수처럼 호출하려 하므로 오류가 발생한다. 리스트 데이터를 표현하려면 인용(quote)을 사용할 수 있다.

```clojure
'(1 2 3)
;; => (1 2 3)
```

맵의 `:name`은 문자열이 아닌 **키워드(keyword)** 다.

```clojure
(def user {:id 1 :name "Kim" :active true})
(:name user)
;; => "Kim"
```

## 4. 함수 정의 — defn

Java:

```java
public int calculateTotal(int price, int quantity) {
    return price * quantity;
}
```

Clojure:

```clojure
(defn calculate-total [price quantity]
  (* price quantity))

(calculate-total 10000 3)
;; => 30000
```

- `defn`: 함수 정의
- `calculate-total`: 함수 이름 (하이픈 명명법이 일반적)
- `[price quantity]`: 매개변수 목록
- 함수의 마지막 표현식 평가 결과가 반환되므로 Java의 `return`이 필요 없다.

## 5. 직접 푼 실습 문제

### 문제 1 — 중첩 연산 결과

```clojure
(* (+ 10 5) (- 8 3))
;; => 75
```

**제출 답안: 75 (정답)**

### 문제 2 — 할인 계산 결과

```clojure
(defn calculate-discount [price rate]
  (- price (* price rate)))

(calculate-discount 20000 0.1)
;; => 18000.0
```

**제출 답안: 18000 (수학적 결과 정답)**

- `0.1`은 부동소수점 값이므로 실제 REPL에서는 `18000.0`으로 표시된다.
- 금액 계산 로직을 마이그레이션할 때는 숫자 정밀도와 자료형을 명확히 검증해야 한다.

### 문제 3 — Java 함수를 Clojure로 변환

**직접 작성한 정답:**

```clojure
(defn calculate-payable [price quantity coupon]
  (- (* price quantity) coupon))
```

총액에서 쿠폰 금액을 차감하며, 마지막 표현식의 값이 반환된다.

## 6. Java / Spring Boot 전환 시 기억할 점

- **문법 변환 ≠ 의미적 동등성:** 입출력뿐 아니라 예외, 상태 변경, DB·외부 연동 부작용도 확인한다.
- **숫자 타입:** 부동소수점·정수·BigDecimal의 차이를 고려한다.
- **키워드와 문자열:** `:name`과 `"name"`은 서로 다른 키다.
- **기존 동작 테스트:** 기능을 새로 작성하기 전에 현재 Clojure 구현의 동작을 검증 가능한 테스트로 고정한다.

## 7. 복습 체크리스트

- [x] 전위 표기법 해석
- [x] 중첩 함수 호출 평가
- [x] `defn`으로 함수 정의
- [x] Java 계산 함수를 Clojure로 작성
- [ ] `let` 지역 바인딩과 스코프 (2강)

## 공식 참고 자료

- [Clojure Learn](https://clojure.org/guides/learn/clojure)
- [Syntax](https://clojure.org/guides/learn/syntax)
