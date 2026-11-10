# 04. Hashed Collections — Map, Set과 중첩 데이터

- 학습일: 2026-11-11
- 학습 자료: [공식 Hashed Collections 가이드](https://clojure.org/guides/learn/hashed_colls)
- 실습: **4/4 정답 (100점)**
- 목표: Map/Set 불변성, 중첩 Map 조회, Set의 중복 제거를 이해한다.

## 1. Map과 Keyword

```clojure
(def user {:id 1 :name "Kim" :age 30})
(get user :name)               ;; => "Kim"
(:name user)                   ;; => "Kim"
(get user :email "unknown")    ;; => "unknown"
```

- `:name`은 Keyword이며 문자열 `"name"`과 다른 키다.
- `get`의 기본값은 **키가 없을 때** 반환된다. 값이 `nil`인 기존 키에는 `nil`이 반환된다.

```clojure
(get {:email nil} :email "unknown") ;; => nil
(contains? {:email nil} :email)     ;; => true
```

## 2. assoc와 dissoc: 불변 Map

```clojure
(def user {:name "Kim" :age 30})
(def updated (assoc user :age 31))
(:age user)      ;; => 30
(:age updated)   ;; => 31
(dissoc user :age) ;; => {:name "Kim"}
```

`assoc`와 `dissoc`는 원본을 변경하지 않고 새 Map을 반환한다.

## 3. Set, conj, disj

```clojure
(def roles #{:admin :user})
(def updated (conj roles :admin :viewer))
(count updated)               ;; => 3
(contains? updated :viewer)   ;; => true
(disj roles :admin)           ;; => #{:user}
```

Set은 중복된 값을 허용하지 않으며, `conj`와 `disj`로 원소를 추가하거나 제거한 새로운 Set을 반환한다.

## 4. 중첩 Map: get-in과 assoc-in

```clojure
(def order {:id 1 :payment {:status :pending}})
(def paid (assoc-in order [:payment :status] :completed))
(get-in order [:payment :status]) ;; => :pending
(get-in paid [:payment :status])  ;; => :completed
```

`assoc-in`도 원본 Map을 변경하지 않는다.

## 5. 실습 채점 — 4문제 모두 정답

### 1번: Map 불변성

```clojure
(def user {:name "Kim" :age 30})
(assoc user :age 31)
(:age user) ;; => 30
```

- 제출: **30, Map은 불변이다.**
- 결과: **정답**. 반환된 새 Map을 사용하지 않았으므로 원본은 유지된다.

### 2번: 누락된 키의 기본값

```clojure
(get {:name "Kim"} :age "unknown")
;; => "unknown"
```

- 제출: **"unknown"**
- 결과: **정답**.

### 3번: Set 중복 제거

```clojure
(def roles #{:admin :user})
(def updated (conj roles :admin :viewer))
(count updated) ;; => 3
```

- 제출: **3**
- 결과: **정답**. `:admin`은 중복 추가되지 않는다.

### 4번: 중첩 Map을 조회한 결과 Vector

```clojure
(def order {:id 1 :payment {:status :pending}})
(def paid (assoc-in order [:payment :status] :completed))
[(get-in order [:payment :status])
 (get-in paid [:payment :status])]
;; => [:pending :completed]
```

- 제출: **[:pending :completed]**, “단순 조회 및 변경 후 조회의 vector 자료구조.”
- 결과: **정답**. 최종 Vector는 Keyword 값 두 개를 원소로 가진다.

## 6. 3강 오개념 복습: 중첩과 평탄화

3강에서는 `[xs ys]`를 평탄화된 Vector로 잘못 해석했다.

```clojure
(def xs [10 20])
(def ys [10 20 30 40])
[xs ys]       ;; => [[10 20] [10 20 30 40]]
(into xs ys)  ;; => [10 20 10 20 30 40]
```

4강에서는 Map의 **중첩 조회 경로**와 최종 반환 Vector를 정확히 구분했다. 다만 중첩 Vector 자체를 직접 평가하는 문제는 후속 학습에서 다시 확인한다.

## 7. 실무 마이그레이션 체크리스트

- [x] `assoc` / `assoc-in`의 불변성
- [x] Map 기본값과 없는 키 이해
- [x] Set의 중복 제거와 불변성
- [x] 중첩 조회 결과 Vector 파악
- [ ] 중첩 Vector와 평탄화 추가 검증
- [ ] `contains?`가 Map에서는 **키**, Set에서는 **원소**, Vector에서는 **인덱스** 존재를 검사한다는 점 복습

Java DTO, `HashMap`, JSON으로 전환할 때 **원본 상태 변경 여부·자료구조의 중첩 깊이·반환 데이터**를 테스트로 확인한다.

## 8. 다음 학습

5강: `map`, `filter`, `reduce`, `->` / `->>`, Lazy Evaluation.

## 참고 문서

- [Clojure 공식 Hashed Collections](https://clojure.org/guides/learn/hashed_colls)
- [Clojure Data Structures](https://clojure.org/reference/data_structures)
