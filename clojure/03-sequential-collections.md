# 03. Sequential Collections — Vector, List, Sequence

- 학습일: 2026-11-11
- 공식 교재: [Learn Clojure — Sequential Collections](https://clojure.org/guides/learn/sequential_colls)
- 추가 참고: [Clojure Data Structures](https://clojure.org/reference/data_structures)
- 목표: 순차 컬렉션의 불변성, `conj` 추가 위치, 중첩 컬렉션, `first` / `rest` / `next`를 이해한다.
- 실습: **최초 2/4**, **보충 2/3**. List에 원소를 추가하는 순서와 불변성 관련 결과 예측은 개선됐지만, 중첩 Vector 표현은 추가 복습이 필요하다.

## 1. Vector와 List

### Vector — 대괄호

```clojure
(def numbers [10 20 30])
(get numbers 0)
;; => 10

(count numbers)
;; => 3
```

- 인덱스를 사용해 요소에 접근하는 순차 컬렉션이다.
- `conj`가 Vector 끝에 요소를 추가한 **새 Vector**를 반환한다.

### List — 인용한 소괄호

```clojure
(def numbers-list '(10 20 30))
(first numbers-list)
;; => 10
```

- List는 앞쪽 원소 처리에 적합하다.
- Clojure에서 `(10 20 30)`은 평가될 때 함수 호출처럼 해석되므로, 데이터로 된 List는 `'(10 20 30)`처럼 인용한다.
- `conj`는 List의 **맨 앞**에 원소를 추가한 새 List를 반환한다.

### 차이 비교

| 구분 | Vector | List |
| --- | --- | --- |
| 표기 | `[1 2 3]` | `'(1 2 3)` |
| 접근 | 인덱스 접근 가능 | 순차적 탐색에 적합 |
| `(conj coll 4)` | `[1 2 3 4]` | `(4 1 2 3)` |
| 원본 변경 | 없음 | 없음 |

## 2. conj와 불변성

```clojure
(def nums [1 2 3])
(conj nums 4)
nums
;; => [1 2 3]
```

`conj`의 반환값을 사용하지 않았으므로 `nums`가 가리키는 Vector 값은 변하지 않는다.

```clojure
(def updated (conj nums 4))
nums
;; => [1 2 3]
updated
;; => [1 2 3 4]
```

Clojure의 영속적 컬렉션은 새 값을 반환하면서 내부 구조를 공유할 수 있다. Java의 변경 가능한 `ArrayList.add`를 그대로 대응시키면 기존 동작과 달라질 수 있다.

### 같은 conj, 다른 추가 방향

```clojure
(conj [1 2 3] 4 5)
;; => [1 2 3 4 5]

(conj '(1 2 3) 4 5)
;; => (5 4 1 2 3)
```

List에는 4가 먼저 앞에 들어가고, 그 다음 5가 **다시 앞에** 들어가므로 최종 결과가 뒤집혀 보인다.

## 3. Sequence, first, rest, next

```clojure
(def v [10 20 30])
(first v)
;; => 10

(rest v)
;; => (20 30)

(next v)
;; => (20 30)
```

끝에 도달했을 때 차이:

```clojure
(rest [100])
;; => ()

(next [100])
;; => nil
```

- `rest`: 남은 원소의 시퀀스를 반환. 남은 것이 없으면 빈 시퀀스.
- `next`: 남은 원소의 시퀀스를 반환. 남은 것이 없으면 `nil`.
- Clojure에서는 **빈 시퀀스도 참(truthy)** 이므로, 남은 원소 존재 여부를 검사할 때 `next`를 유용하게 쓸 수 있다.
- Sequence는 Vector나 List 그 자체와 동일한 구체 타입이 아니라 순차 탐색을 위한 추상화다.

## 4. 최초 실습 — 2/4 정답

### 문제 1 — 불변 Vector (오답)

```clojure
(def nums [1 2 3])
(conj nums 4)
nums
;; => [1 2 3]
```

- 최초 제출: `[1 2 3 4]`
- **정답: `[1 2 3]`**
- 교정: `conj`가 반환한 새 Vector를 보관하지 않으면 원본 `nums`는 변하지 않는다.

### 문제 2 — List에 여러 원소 추가 (오답)

```clojure
(conj '(1 2 3) 4 5)
;; => (5 4 1 2 3)
```

- 최초 제출: `(4 5 1 2 3)`
- **정답: `(5 4 1 2 3)`**
- 교정: `4`가 먼저 맨 앞에 추가되고, `5`가 다시 맨 앞에 추가된다.

### 문제 3 — rest와 next (정답)

```clojure
(rest [100]) ; => ()
(next [100]) ; => nil
```

- 제출: 빈 시퀀스와 `nil`을 각각 반환한다고 설명.
- **정답**.

### 문제 4 — 불변 Vector 새 값 사용 (정답)

직접 작성한 코드:

```clojure
(def numbers [10 20 30])
(def updated (conj numbers 40))
```

- **정답**. 원본은 유지하고 `updated`에 새 Vector를 바인딩한다.

## 5. 보충 실습 — 2/3 정답

### 복습 A — 중첩 Vector (오답, 개념 보완)

```clojure
(def xs [10 20])
(def ys (conj xs 30 40))
[xs ys]
;; => [[10 20] [10 20 30 40]]
```

- 보충 제출: `[10 20 10 20 30 40]`
- **정답: `[[10 20] [10 20 30 40]]`**.
- `[xs ys]`는 **Vector 두 개를 원소로 가지는 Vector**이다. 두 Vector를 자동으로 평탄화하지 않는다.
- 제출한 평탄화 결과가 필요하다면 `(into xs ys)`처럼 명시적인 함수를 사용한다.

```clojure
(into xs ys)
;; => [10 20 10 20 30 40]

(count [xs ys])
;; => 2

(count (into xs ys))
;; => 6
```

### 복습 B — List conj 순서 (정답)

```clojure
(conj '(10 20) 30 40)
;; => (40 30 10 20)
```

- 제출: `(40 30 10 20)` — **정답**.
- 최초 실습에서 혼동했던 다중 인자의 추가 순서를 교정했다.

### 복습 C — 불변성과 let (정답, 이유 보충 필요)

```clojure
(let [xs [1 2]]
  (conj xs 3)
  (count xs))
;; => 2
```

- 제출: `2` — **반환값 정답**.
- 이유: `conj`가 생성한 새 Vector를 사용하지 않았고, `let`의 마지막 표현식인 `(count xs)`가 원래 Vector의 요소 수를 반환한다.

## 6. Java / Spring Boot 마이그레이션 포인트

- `conj` 결과를 버린 경우 원본 컬렉션이 변경되지 않았음을 보존한다.
- Vector/List에 `conj`를 적용할 때 **추가 순서**가 달라지는지 확인한다.
- `[xs ys]`와 `(into xs ys)`를 혼동하면 중첩 데이터 스키마와 API JSON 배열 형태가 달라진다.
- `nil`과 빈 컬렉션, 빈 시퀀스를 구분한다. JSON 직렬화나 Java의 `null` / `Optional` 변환 때 의미가 바뀔 수 있다.
- 단순히 타입만 치환하지 말고 실제 실행 결과에 대한 테스트를 작성한다.

## 7. 학습 상태와 다음 목표

- [x] Vector의 기본 사용과 인덱스
- [x] List의 `conj` 추가 방향 교정
- [x] `rest`와 `next`의 빈 결과 차이
- [x] 원본 컬렉션이 변하지 않는다는 불변성 개념 복습
- [ ] **중첩 Vector와 평탄화 결과**를 구분하는 문제 추가 확인
- [ ] 답안에서 반환값뿐 아니라 이유를 설명하는 습관
- [ ] 4강 Hashed Collections — Map과 Set

## 참고

- [Sequential Collections 공식 가이드](https://clojure.org/guides/learn/sequential_colls)
- [Clojure Data Structures](https://clojure.org/reference/data_structures)
