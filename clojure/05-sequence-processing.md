# 05. Sequence Processing — map, filter, reduce, Threading Macros와 Lazy Evaluation

- 학습일: 2026-11-11
- 공식 참고: [Sequences](https://clojure.org/guides/learn/sequences), [Threading Macros](https://clojure.org/guides/threading_macros), [Lazy Sequences](https://clojure.org/reference/lazy)
- 목적: Clojure 컬렉션 가공 코드를 읽고 Java Stream으로 의미를 보존하며 옮긴다.
- **실습 결과: 5/5 정답 (100점).** 문제 5는 반환값은 정답이지만 요청된 동작 원리 설명이 누락되어 보충 학습으로 남김.

## 1. map — 각 요소 변환

```clojure
(map #(* % 2) [10 20 30])
;; => (20 40 60)
```

- 형태: `(map 함수 컬렉션)`
- `#(* % 2)`는 익명 함수 `(fn [x] (* x 2))`의 축약형이다.
- `map` 함수의 결과는 보통 **지연 시퀀스**다. 자료구조로서의 Map(`{:id 1}`)과 혼동하지 않는다.
- Vector가 필요하면 `(vec (map inc [1 2 3]))` → `[2 3 4]`.

Java 비유:

```java
List<Integer> doubled = numbers.stream()
    .map(n -> n * 2)
    .toList();
```

`toList()`의 결과 자료구조와 Clojure의 지연 시퀀스는 같지 않다.

## 2. filter — 조건에 맞는 요소만 선택

```clojure
(filter even? [1 2 3 4 5 6])
;; => (2 4 6)

(filter #(>= % 100) [50 100 150 200])
;; => (100 150 200)
```

- `even?`와 같은 술어(Predicate)를 전달한다.
- `filter`는 선택된 요소를 포함하는 지연 시퀀스를 반환한다.
- 원본 컬렉션을 변경하지 않는다.

## 3. reduce — 누적 계산

```clojure
(reduce + 0 [10 20 30])
;; => 60

(reduce
  (fn [total amount] (+ total amount))
  0
  [10 20 30])
;; => 60

(reduce + 0 [])
;; => 0
```

- 형태: `(reduce 누적함수 초기값 컬렉션)`
- 집계 함수는 이전 누적값과 현재 요소를 받는다.
- 초기값이 명시되면 빈 컬렉션의 처리도 예측 가능하다.

## 4. -> 와 ->> — Threading Macros

`->`는 이전 결과를 **다음 호출의 첫 번째 인자**에 삽입한다.

```clojure
(-> {:name "Kim" :balance 10000}
    (update :balance #(+ % 5000))
    (assoc :active true))
;; => {:name "Kim", :balance 15000, :active true}
```

위 코드는 다음 호출 구조에 대응한다.

```clojure
(assoc
  (update {:name "Kim" :balance 10000}
          :balance #(+ % 5000))
  :active true)
```

`->>`는 이전 결과를 **다음 호출의 마지막 인자**에 삽입한다.

```clojure
(->> [1 2 3 4 5 6]
     (filter odd?)
     (map #(* % 10))
     (reduce + 0))
;; => 90
```

Threading Macro는 **멀티스레딩이 아니라 값을 함수 사이로 전달하는 매크로**다. `map`, `filter`, `reduce`는 컬렉션을 마지막 인자로 받기 때문에 `->>`와 함께 자주 쓴다.

## 5. Lazy Evaluation과 무한 시퀀스

```clojure
(def result
  (map #(* % 2) (range)))

(take 3 result)
;; => (0 2 4)
```

- 인자 없는 `(range)`는 끝이 없는 숫자 시퀀스를 나타낼 수 있다.
- `map`은 결과를 지연 시퀀스로 만들어, 소비 시점에 필요한 부분을 평가한다.
- `take`는 지정한 개수만 결과로 제공한다. **무한한 데이터를 모두 메모리에 보관한다는 뜻이 아니다.**
- 단, 지연 시퀀스의 **청크 평가(chunking)** 때문에 실제 변환 함수 호출 횟수가 출력 요소 수와 정확히 일치한다고 가정하면 안 된다.
- 무한 시퀀스 전체를 `doall` 또는 `vec`으로 실체화하려 하면 종료되지 않거나 메모리가 부족해질 수 있다.

Java Stream과의 유사점:

```java
List<Integer> firstThree = Stream.iterate(0, n -> n + 1)
    .map(n -> n * 2)
    .limit(3)
    .toList();
// [0, 2, 4]
```

Java의 Stream은 일반적으로 한 번 소비하면 재사용할 수 없는 반면, Clojure의 지연 시퀀스는 소비·캐시·재사용 등 세부 동작이 다르다. 실무 전환 시 이 차이를 고려한다.

## 6. 실무 주문 집계 예제

```clojure
(def orders
  [{:id 1 :status :paid    :amount 10000}
   {:id 2 :status :pending :amount 20000}
   {:id 3 :status :paid    :amount 30000}])

(defn calculate-paid-total [orders]
  (->> orders
       (filter #(= (:status %) :paid))
       (map :amount)
       (reduce + 0)))

(calculate-paid-total orders)
;; => 40000
```

Java로 옮길 때는 데이터가 `nil`일 수 있는지, 금액 타입과 합계 오버플로·정밀도 문제가 없는지, 평가 시점에 DB·네트워크 부작용이 있는지 확인한다.

## 7. 사용자 실습 채점 — 5/5 정답

### 문제 1 — map + 중첩 Vector

```clojure
(def xs [1 2 3])
(def ys (map inc xs))
[xs (vec ys)]
;; => [[1 2 3] [2 3 4]]
```

- 제출: `[[1 2 3] [2 3 4]]` — **정답**.
- 3강에서 혼동했던 **중첩 Vector vs 평탄화**를 올바르게 구분했다. `[xs (vec ys)]`의 원소는 두 개의 Vector다.

### 문제 2 — filter → map → reduce

```clojure
(->> [1 2 3 4 5 6]
     (filter odd?)
     (map #(* % 10))
     (reduce + 0))
;; => 90
```

- 제출: `90` — **정답**.
- 중간 결과: `(1 3 5)` → `(10 30 50)` → `90`.

### 문제 3 — Thread-first

```clojure
(-> {:name "Kim" :balance 10000}
    (update :balance #(+ % 5000))
    (assoc :active true))
;; => {:name "Kim", :balance 15000, :active true}
```

- 제출: `{:name "Kim" :balance 15000 :active true}` — **정답**.
- Map의 출력 키 순서는 의미적 동등성에 영향을 주지 않는다.

### 문제 4 — 짝수 합계 함수 직접 작성

**제출한 코드:**

```clojure
(defn sum-even-numbers [numbers]
  (->> numbers
    (filter even?)
    (reduce + 0)))
```

- **정답**. `defn`, `->>`, `filter`, `reduce`를 정확히 사용했다.
- `(sum-even-numbers [1 2 3 4 5 6])` → `12`.
- `(sum-even-numbers [])` → `0`.
- 들여쓰기를 정리하면 가독성이 좋아진다.

### 문제 5 — 무한 시퀀스 + 지연 평가

```clojure
(def result (map #(* % 2) (range)))
(take 3 result)
;; => (0 2 4)
```

- 제출: `(0 2 4)` — **반환값 정답**.
- 보완점: 요청했던 **무한 시퀀스에서도 반환값을 얻는 이유**는 설명하지 않았다.
- 해설: `range`가 종료되지 않는 시퀀스를 표현하더라도 `map`과 `take`가 지연 처리하므로 전체 결과를 완성할 필요 없이 앞부분을 가져올 수 있다. 내부적으로 청크 단위 평가가 가능하다.

## 8. Java / Spring Boot 마이그레이션 주의사항

- [x] `map`의 반환값이 Vector가 아닌 지연 시퀀스일 수 있음을 구분
- [x] `filter`, `map`, `reduce` 파이프라인 해석
- [x] `->` / `->>` 인자 위치 이해
- [x] 중첩 Vector 재평가 정답
- [ ] 지연 평가의 실행 시점·청크 평가·부작용 직접 설명
- [ ] 지연 시퀀스가 DB 커넥션, 트랜잭션, 외부 리소스의 수명과 충돌하지 않는지 확인
- [ ] 금액 누적의 자료형, `nil`, 예외 처리 의미 보존

## 9. 다음 학습

6강 — `if`, `when`, `cond`, `nil` / Truthiness와 흐름 제어.

## 공식 참고 문서

- [Clojure Sequences](https://clojure.org/guides/learn/sequences)
- [Threading Macros](https://clojure.org/guides/threading_macros)
- [Clojure Lazy Sequences](https://clojure.org/reference/lazy)
