# 08. Java Interop — Clojure에서 Java 객체 사용하기

- 학습일: 2026-11-11
- 공식 문서: [Clojure Java Interop](https://clojure.org/reference/java_interop)
- 학습 목표: Java 생성자·메서드·정적 멤버 호출과 Clojure 불변 컬렉션/Java 변경 가능 객체의 차이를 이해하고 Java/Spring Boot 마이그레이션에 적용한다.
- 평가 결과: **최초 실습 4/5 정답 → 보충 1/1 정답, 통과**
- 기록 범위: 8강 핵심 내용, 실제 실습 답안과 오답 교정. 부가 질문은 포함하지 않는다.

## 1. Java 생성자 호출

```clojure
(def names (java.util.ArrayList.))
(String. "hello")
(new java.util.ArrayList)
```

- 클래스 이름 뒤의 점 `.`은 Java 생성자를 호출하는 문법이다.
- `new`도 생성자 호출에 사용할 수 있다.
- Clojure에서 만든 Java 객체도 해당 Java 클래스의 변경 가능성 등의 특성을 그대로 가진다.

Java 비교:

```java
ArrayList<String> names = new ArrayList<>();
String greeting = new String("hello");
```

## 2. 인스턴스 메서드 호출

```clojure
(.toUpperCase "java")
;; => "JAVA"

(.substring "Clojure" 0 3)
;; => "Clo"

(.length "Clojure")
;; => 7
```

- `(.method object args...)` 형태로 인스턴스 메서드를 호출한다.
- Java `text.toUpperCase()`에 대응하는 Clojure 코드는 `(.toUpperCase text)`이다.

## 3. 정적 멤버와 import

```clojure
(Math/max 10 20)
;; => 20

(Integer/parseInt "123")
;; => 123

(java.time.LocalDate/parse "2026-11-11")
;; => LocalDate 인스턴스
```

Namespace에 Java 클래스를 import할 수 있다.

```clojure
(ns example.service
  (:import [java.time LocalDate]
           [java.util ArrayList]))

(def date (LocalDate/parse "2026-11-11"))
(.getYear date)
;; => 2026 (정수)
```

정적 메서드는 `ClassName/method`, 정적 필드는 `ClassName/FIELD` 형태로 접근한다.

## 4. Clojure 불변 컬렉션과 Java 변경 가능 객체

Clojure Vector에 `conj`를 호출하면 새 Vector가 반환되며 원본은 변하지 않는다.

```clojure
(def xs [10])
(conj xs 20)
xs
;; => [10]
```

그러나 Java `ArrayList`의 `.add`는 기존 객체를 직접 변경한다.

```clojure
(def xs (java.util.ArrayList.))
(.add xs 10)
(.add xs 20)
[(vec xs) (.size xs)]
;; => [[10 20] 2]
```

Java 객체를 `vec`로 변환하면 별도의 Clojure Vector를 만든다.

```clojure
(let [xs (java.util.ArrayList.)]
  (.add xs 10)
  (let [ys (conj (vec xs) 20)]
    [(vec xs) ys]))
;; => [[10] [10 20]]
```

## 5. doto — 원래 객체에 연속 작업 수행

```clojure
(def names
  (doto (java.util.ArrayList.)
    (.add "Kim")
    (.add "Lee")))

(vec names)
;; => ["Kim" "Lee"]
```

- `doto`는 첫 번째 표현식으로 만든 객체를 후속 호출의 대상으로 사용하고, 원래 객체를 반환한다.
- 각 `.add`의 반환값은 객체를 변경하는 데 사용되지만, 다음 호출로 전달되지 않는다.
- `->`는 이전 단계의 **반환값**을 다음 단계 첫 인자로 전달하므로 동작이 다르다.

예를 들어 아래의 코드는 `.add`가 Boolean을 반환하기 때문에 의도한 체이닝에 실패한다.

```clojure
(-> (java.util.ArrayList.)
    (.add "Kim")
    (.add "Lee"))
;; 두 번째 .add의 대상이 Boolean이 되어 오류
```

## 6. 타입 힌트(Type Hint)와 Reflection

```clojure
(defn upper [^String value]
  (.toUpperCase value))
```

`^String` 타입 힌트는 컴파일러가 Java 메서드 호출 대상을 파악하는 데 도움을 주며, 불필요한 Reflection을 줄일 수 있다. 실제 사용 여부는 컴파일 경고, 성능 측정, 메서드 오버로드를 함께 검토한다.

## 7. 최초 실습 채점 — 4/5

### 1번: 정적 메서드와 인스턴스 메서드

```clojure
[(Math/max 10 20)
 (.toUpperCase "java")]
;; => [20 "JAVA"]
```

- 제출: `[20 "JAVA"]`
- **정답**.

### 2번: LocalDate.getYear 반환 타입 — 최초 오답

```clojure
(.getYear
  (java.time.LocalDate/parse "2026-11-11"))
;; => 2026
```

- 최초 제출: `"2026"`
- **정답: `2026` (숫자)**.
- 교정: Java `LocalDate.getYear()`는 `int`를 반환하므로 Clojure 결과도 문자열이 아닌 숫자다. 입력이 문자열이더라도 메서드의 반환 타입이 결정적이다.
- 문자열이 필요할 때는 `(str (.getYear date))`를 사용한다.

### 3번: Java ArrayList의 상태 변경

```clojure
(let [xs (java.util.ArrayList.)]
  (.add xs 10)
  (.add xs 20)
  [(vec xs) (.size xs)])
;; => [[10 20] 2]
```

- 제출: `[[10 20] 2]`
- **정답**.

### 4번: ArrayList와 Clojure 불변 Vector

```clojure
(let [xs (java.util.ArrayList.)]
  (.add xs 10)
  (let [ys (conj (vec xs) 20)]
    [(vec xs) ys]))
;; => [[10] [10 20]]
```

- 제출: `[[10] [10 20]]`
- **정답**. 원본 Java List가 변경되지 않고 별도의 Vector가 생성되었다. 중첩 Vector 형태도 정확하다.

### 5번: doto를 사용한 함수 작성

**제출 코드:**

```clojure
(defn create-names []
  (doto (java.util.ArrayList.)
    (.add "Kim")
    (.add "Lee")))
```

- **정답**. 함수는 `["Kim" "Lee"]`로 변환할 수 있는 Java `ArrayList` 객체를 반환한다.

## 8. 보충 실습 — 1/1 통과

문제:

```clojure
(let [date (java.time.LocalDate/parse "2026-11-11")]
  [(.getYear date)
   (str (.getYear date))
   (.toString date)])
```

- 보충 제출: **`[2026 "2026" "2026-11-11"]`**
- **정답**.
- 첫 요소는 정수, 둘째와 셋째 요소는 문자열이다.
- 최초 2번의 숫자와 문자열 혼동이 교정됐음을 확인했다.

## 9. Java/Spring Boot 마이그레이션 체크리스트

- [x] Java 생성자, 인스턴스·정적 메서드 호출 문법을 구분
- [x] `doto`의 원본 객체 반환과 `->`의 반환값 전달을 구분
- [x] Java `ArrayList`의 변경 가능성과 Clojure Vector 불변성을 구분
- [x] 메서드의 Java 반환 타입과 Clojure 값 표현을 구분 (보충 실습으로 확인)
- [x] 중첩 Vector 결과를 정확하게 표현
- [ ] Java Interop 메서드 오버로드 및 Type Hint의 실전 사례
- [ ] 레거시 프로젝트의 Java 객체 공유·리소스 수명·API 직렬화 동작 검증

## 10. 다음 강의

9강: 상태 관리 — `atom`, `deref` / `@`, `swap!`, `reset!`와 Java 동시성 비교.

## 참고

- [Clojure Java Interop 공식 문서](https://clojure.org/reference/java_interop)
- [Clojure Vars and Environments](https://clojure.org/reference/vars)
