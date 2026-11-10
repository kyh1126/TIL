# 07. Exception Handling — try, catch, finally, throw, ex-info와 ex-data

- 학습일: 2026-11-11
- 학습 자료: [Clojure Flow Control](https://clojure.org/guides/learn/flow), [Special Forms](https://clojure.org/reference/special_forms)
- 목표: Java 예외와 Clojure 예외 처리의 공통점·차이점을 이해하고 도메인 예외 데이터를 Java로 보존한다.
- 실습 결과: **최초 4/5 정답 → 보충 1/2 정답 → 최종 1/1 정답 및 설명 통과**.

## 1. try / catch / finally: 반환값과 부작용

```clojure
(try
  (/ 20 0)
  (catch ArithmeticException e
    :divide-by-zero))
;; => :divide-by-zero
```

- `try`도 값을 반환하는 표현식이다. 정상이면 본문 마지막 값, 예외를 잡으면 선택된 `catch`의 마지막 값을 반환한다.
- `finally`는 예외 발생 여부와 관계없이 실행된다(정상적인 제어 흐름 기준).
- `finally`의 마지막 표현식 값은 일반적으로 `try` / `catch`의 반환값을 덮어쓰지 않는다.
- 단, `finally`가 새 예외를 던지면 기존 예외 또는 정상 결과에 영향을 준다.

```clojure
(try
  (+ 2 3)
  (catch Exception e :failed)
  (finally (println "cleanup")))
;; "cleanup" 출력, 전체 반환값 5
```

## 2. throw — JVM 예외 객체 던지기

```clojure
(throw (IllegalArgumentException. "invalid amount"))
```

Java의 `throw new IllegalArgumentException("invalid amount");`에 대응한다. `IllegalArgumentException.`의 끝 점은 Java 생성자 호출 문법이다.

## 3. ex-info / ex-data — 구조화된 도메인 예외

```clojure
(try
  (throw
    (ex-info
      "Invalid amount"
      {:code :invalid-amount
       :amount -10}))
  (catch clojure.lang.ExceptionInfo e
    [(.getMessage e)
     (:code (ex-data e))]))
;; => ["Invalid amount" :invalid-amount]
```

- `ex-info`는 `clojure.lang.ExceptionInfo` 객체를 **생성**한다. 자동으로 던지지는 않는다.
- `throw`를 사용해 예외를 발생시킨다.
- `ex-data`는 예외에 포함된 Map을 읽는다.
- `(.getMessage e)`는 Java 인스턴스 메서드 호출 문법이며, Java의 `e.getMessage()`에 대응한다.

## 4. catch 분기와 클래스 상속

```clojure
(try
  (throw (IllegalArgumentException. "bad"))
  (catch RuntimeException e :runtime)
  (catch Exception e :general))
;; => :runtime
```

`IllegalArgumentException`은 `RuntimeException`의 하위 클래스이므로 먼저 일치하는 `catch`의 결과를 반환한다. 구체적인 타입을 상위 타입보다 먼저 배치하는 습관이 유용하다.

## 5. Java/Spring Boot 마이그레이션: 예외 데이터 의미 보존

```clojure
(defn validate-payment [amount]
  (if (pos? amount)
    {:status :ok :amount amount}
    (throw
      (ex-info "Invalid amount"
               {:type :payment/invalid-amount
                :amount amount}))))
```

Java로 옮길 때는 단순히 메시지만 보존하지 말고, `ex-data`에 포함된 `:type`과 `:amount`가 예외 핸들러, 로깅, HTTP 응답 등에 활용되는지 확인해야 한다.

## 6. 최초 실습 — 4/5 정답

### 1번: ArithmeticException (정답)

```clojure
(try
  (/ 20 0)
  (catch ArithmeticException e :divide-by-zero))
;; => :divide-by-zero
```

제출: `:divide-by-zero` — **정답**.

### 2번: finally 실행과 반환값 (정답)

```clojure
(try
  (+ 2 3)
  (catch Exception e :failed)
  (finally (println "cleanup")))
;; => 5 ("cleanup" 출력)
```

제출: `"cleanup" / 5` — **정답**.

### 3번: ex-info / ex-data (정답)

```clojure
(try
  (throw (ex-info "Invalid amount"
                  {:code :invalid-amount :amount -10}))
  (catch clojure.lang.ExceptionInfo e
    [(.getMessage e) (:code (ex-data e))]))
;; => ["Invalid amount" :invalid-amount]
```

제출: `["Invalid amount" :invalid-amount]` — **정답**.

### 4번: catch 상속 관계 (정답)

```clojure
(try
  (throw (IllegalArgumentException. "bad"))
  (catch RuntimeException e :runtime)
  (catch Exception e :general))
;; => :runtime
```

제출: `:runtime, IllegalArgumentException은 RuntimeException 타입임.` — **정답**.

### 5번: validate-payment 직접 작성 (오답)

최초 제출:

```clojure
(defn validate-payment [amount]
  (if pos? {:status :ok :amount amount})
  (throw (ex-info "Invalid amount" {:type :payment/invalid-amount :amount amount}))
```

문제점:
1. `pos?`는 **함수 객체**이므로 호출하지 않아도 참으로 평가된다. 조건 검사는 `(pos? amount)`여야 한다.
2. `throw`를 `if` 바깥에 두면, 문법을 완성하더라도 앞의 `if` 결과와 무관하게 결국 예외를 던진다.
3. 제출 코드에는 `defn`을 닫는 괄호도 부족하다.

수정:

```clojure
(defn validate-payment [amount]
  (if (pos? amount)
    {:status :ok :amount amount}
    (throw
      (ex-info "Invalid amount"
               {:type :payment/invalid-amount
                :amount amount}))))
```

## 7. 보충 실습 — 1/2 정답

### A: 함수 객체와 조건 평가 (오답)

```clojure
(if pos? :positive :negative)
;; => :positive
```

제출: `:negative / 함수 호출을 하지 않았음.` — **오답**.

- 함수를 호출하지 않았다는 분석은 맞지만, 그 사실이 조건을 거짓으로 만들지는 않는다.
- Clojure에서 거짓은 `false`와 `nil`뿐이며 **함수 객체도 참**이다.

### B: validate-refund 작성 (정답)

제출 코드:

```clojure
(defn validate-refund [amount]
  (if (pos? amount)
    {:status :valid :amount amount}
    (throw (ex-info "Invalid refund"
                    {:type :refund/invalid-amount
                     :amount amount}))))
```

**정답**. `pos?` 함수 호출, `if`의 양쪽 분기, `throw` / `ex-info` 데이터, 괄호가 모두 맞다.

## 8. 최종 확인 — 1/1 정답 (통과)

```clojure
(defn is-valid? [amount]
  (pos? amount))

[(if is-valid? :A :B)
 (if (is-valid? -100) :C :D)
 (if (is-valid? 100) :E :F)]
;; => [:A :D :E]
```

제출: **`[:A :D :E] / 오직 false, nil만이 거짓으로 평가되기 때문이다. 함수 객체는 참으로 평가된다.`**

- **정답**. 조건에 함수 객체를 전달한 경우와 함수 호출의 Boolean 결과를 전달한 경우를 구분했다.
- 최초 오류였던 함수 호출 및 분기 이해가 최종 확인에서 교정됐다.

## 9. 마이그레이션 체크리스트

- [x] `try` / `catch` / `finally`의 실행 시점과 반환값
- [x] 예외 객체 **생성**과 `throw` **발생** 구분
- [x] `ex-info` / `ex-data`의 도메인 데이터 보존 필요성
- [x] `RuntimeException`과 하위 예외의 `catch` 매칭
- [x] `if`의 참·거짓 분기에 `throw`를 올바르게 배치
- [x] 함수 객체와 함수 호출 결과의 Truthiness 차이 이해
- [ ] Java 전환 시 예외 매핑, 트랜잭션 롤백, HTTP 오류 응답 계약을 테스트로 검증
- [ ] Checked Exception 및 Java Interop 관련 추가 학습

## 10. 다음 강의

8강: Java Interop — 생성자, 인스턴스/정적 메서드, `import`, `doto`, Java 객체와 Clojure 컬렉션.

## 참고 문서

- [Clojure Flow Control](https://clojure.org/guides/learn/flow)
- [Clojure Special Forms](https://clojure.org/reference/special_forms)
- [Clojure Java Interop](https://clojure.org/reference/java_interop)
