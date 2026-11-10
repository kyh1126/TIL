# 06. Flow Control — if, when, cond, Truthiness와 nil

- 학습일: 2026-11-11
- 공식 참고: [Clojure Learn — Flow Control](https://clojure.org/guides/learn/flow), [Special Forms](https://clojure.org/reference/special_forms)
- 학습 목표: Clojure의 조건 분기, 진릿값 판정, 빈 컬렉션과 `nil`을 구별하고 Java로 의미를 보존해 옮긴다.
- 평가: **최초 실습 3/5 정답 → 보충 2/3 정답 → 최종 확인 1/1 정답 (통과)**

## 1. if — 표현식의 결과 반환

```clojure
(if (>= 10000 5000) "APPROVED" "REJECTED")
;; => "APPROVED"

(if false "YES")
;; => nil
```

- `if`는 조건이 참일 때 두 번째 표현식, 거짓일 때 세 번째 표현식의 값을 반환한다.
- else 표현식을 생략할 수 있으며, 조건이 거짓이면 `nil`이다.
- Java의 `if` 문과 다르게 Clojure에서는 `if` 자체가 값을 반환한다.

## 2. Truthiness — false와 nil만 거짓

| 값 | Clojure 조건식 평가 |
| --- | --- |
| `nil` | 거짓 |
| `false` | 거짓 |
| `0` | 참 |
| `""` | 참 |
| `[]` | 참 |
| `{}` | 참 |
| `()` | 참 |

```clojure
(if [] "PRESENT" "EMPTY")
;; => "PRESENT"

(if (empty? []) "EMPTY" "PRESENT")
;; => "EMPTY"
```

컬렉션이 비어 있다는 사실과 그 컬렉션 값이 거짓인지는 서로 다른 개념이다. Java에서는 조건에 반드시 `boolean` 표현식을 사용하며, 빈 목록 판단에는 `isEmpty()`를 사용한다.

## 3. when — 참일 때만 본문 실행

```clojure
(when true
  (println "executed")
  :success)
;; executed 출력, 반환값 :success

(when false
  (println "executed")
  :success)
;; 출력 없음, 반환값 nil
```

조건이 참이면 여러 표현식을 순서대로 평가하고 마지막 값을 반환한다. 거짓이면 본문을 실행하지 않고 `nil`을 반환한다. 마지막 표현식의 값이 반환되는 것은 **본문이 실행될 때에만** 해당한다.

## 4. cond — 위에서부터 첫 번째 참인 분기 선택

```clojure
(let [age 20]
  (cond
    (>= age 21) :adult
    (>= age 18) :young-adult
    :else :minor))
;; => :young-adult
```

`:else`는 별도의 특수 키워드 구문이 아니라, 참으로 평가되는 Keyword를 마지막 조건으로 사용한 관례다. 일치하는 조건이 없고 기본 분기도 없다면 `nil`을 반환한다.

## 5. nil과 누락된 Map 키

```clojure
(def user {:profile {:email nil}})

(get-in user [:profile :email])
;; => nil

(contains? (:profile user) :email)
;; => true

(contains? (:profile user) :phone)
;; => false

(nil? nil)
;; => true

(some? false)
;; => true
```

`nil` 값이 들어 있는 키와 누락된 키는 다르다. Java의 `Map.containsKey()`와 비슷하게 `contains?`를 활용한다.

## 6. and / or — 단락 평가와 실제 반환값

```clojure
(and true :ok)
;; => :ok

(or nil :fallback)
;; => :fallback
```

`and`와 `or`는 단락 평가하며 항상 Boolean만 반환하는 것은 아니다. Java의 `&&` / `||`가 Boolean 결과를 반환한다는 점과 차이가 있다.

## 7. seq, rest, empty?의 차이 — 핵심 오개념 교정

```clojure
(rest [1])
;; => ()

(seq (rest [1]))
;; => nil

(seq [])
;; => nil

(empty? [])
;; => true
```

- `rest`는 남은 요소가 없어도 **빈 시퀀스**를 반환하며, Clojure에서 빈 시퀀스는 참으로 평가된다.
- `seq`는 요소가 없는 컬렉션에 대해 **nil**을 반환한다.
- `empty?`는 컬렉션이 비었는지 Boolean으로 반환한다.
- 즉, **`[]` / `()`와 `nil`은 조건식에서 서로 다르게 동작한다**.

## 8. 최초 실습 — 3/5 정답

### 문제 1: 0의 진릿값

```clojure
(if 0 "YES" "NO")
;; => "YES"
```

제출 `"YES"` — **정답**.

### 문제 2: 빈 Vector의 진릿값

```clojure
(if [] "PRESENT" "EMPTY")
;; => "PRESENT"
```

제출 `"EMPTY"` — **오답**. 빈 Vector `[]`도 참이다.

### 문제 3: when이 false일 때

```clojure
(when false
  (println "executed")
  :success)
;; 출력 없음, 반환값 nil
```

제출 `"실행안됨. :success"` — **부분 정답**. 본문이 실행되지 않는 것은 알았지만, 반환값을 `:success`로 잘못 예측했다. 조건이 거짓이면 `nil`이다.

### 문제 4: cond

```clojure
(let [age 20]
  (cond
    (>= age 21) :adult
    (>= age 18) :young-adult
    :else :minor))
;; => :young-adult
```

제출 `:young-adult` — **정답**.

### 문제 5: 중첩 Map과 nil

```clojure
(def user {:profile {:email nil}})
[(get-in user [:profile :email])
 (contains? (:profile user) :email)
 (contains? (:profile user) :phone)]
;; => [nil true false]
```

제출 `[nil true false]` — **정답**. 중첩 Map의 조회 결과를 Vector로 정확히 해석했다.

## 9. 1차 보충 실습 — 2/3 정답

### A: 빈 Map

```clojure
(if {} :yes :no)
;; => :yes
```

제출 `:yes` — **정답**.

### B: seq와 when

```clojure
(when (seq [])
  :processed)
;; => nil
```

제출 `:processed` — **오답**. `seq`는 빈 Vector에 대해 `nil`을 반환하므로 `when` 본문은 실행되지 않는다.

### C: empty?와 when

```clojure
(when (empty? [])
  (println "empty")
  :done)
;; "empty" 출력, 반환값 :done
```

제출 `"실행됨 / :done"` — **정답**.

## 10. 최종 확인 — 1/1 정답, 개념 설명까지 통과

```clojure
[(when (rest [1])
   :A)
 (when (seq (rest [1]))
   :B)]
;; => [:A nil]
```

제출: **`[:A nil]` — "seq는 빈 컬렉션일 때 nil을 반환하고, when은 조건이 거짓이면 nil을 반환한다."**

- **정답**. 첫 번째 `when`은 빈 시퀀스 `()`를 참으로 평가하여 `:A`를 반환한다.
- 두 번째 `when`의 조건은 `(seq ())`이므로 `nil`이 되어 반환값 역시 `nil`이다.
- 최초 실습에서 보인 **빈 컬렉션 = 거짓** 혼동과 `when` 반환값 오개념을 최종적으로 교정했다.

## 11. Java / Spring Boot 마이그레이션 관점

- [x] Clojure 조건문에서 거짓인 값은 `nil`과 `false`뿐임을 이해
- [x] 빈 Vector/Map/시퀀스의 Truthiness 이해
- [x] `when` 조건이 거짓일 때 `nil`을 반환함을 이해
- [x] `seq`와 `rest`의 빈 결과 차이를 설명하고 실습으로 검증
- [x] Map에서 **누락된 키**와 **값이 nil인 키** 구분
- [ ] `and` / `or`가 실제 값 자체를 반환하는 경우 추가 실습
- [ ] Clojure의 조건식과 반환값을 Java `boolean` / `null` 및 Optional 등으로 옮길 때 동등성 테스트 수행

## 12. 다음 학습

7강: Exception Handling — `try`, `catch`, `finally`, `throw`, `ex-info` / `ex-data`.

## 공식 참고 문서

- [Learn Clojure — Flow Control](https://clojure.org/guides/learn/flow)
- [Clojure Special Forms](https://clojure.org/reference/special_forms)
