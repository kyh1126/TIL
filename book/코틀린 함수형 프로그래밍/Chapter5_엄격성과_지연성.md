# 5장. 엄격성과 지연성

- 대부분의 현대적인 프로그래밍 언어와 마찬가지로 코틀린도 기본적으로 엄격한 평가를 수행한다.
    - 엄격하다: 함수를 호출하기 전에 함수 파라미터를 완전히 평가한다는 뜻
- 엄격한 평가에서 식은 그 식이 변수에 바운드되는 순간에 평가된다.
    - 식을 함수 파라미터로 넘기는 경우도 포함된다.

- ex> 트럼프 카드 무더기가 있는데, 홀수 카드를 제거하고 모든 퀸을 뒤집으라는 요구
    
    ```kotlin
    List.of(1, 2, 3, 4)
        .map { it + 10 }.filter { it % 2 == 0 }.map { it * 3 }
    List.of(11, 12, 13, 14)
        .filter { it % 2 == 0 }.map { it * 3 }
    List.of(12, 14)
        .map { it * 3 }
    List.of(36, 42)
    ```
    

- 비엄격성(또는 지연 계산)을 통해 이런 식의 자동 루프 융합을 달성할 수 있다.
- 함수형 프로그래밍에서 비엄격성은 효율을 증가시키고 모듈화를 촉진하는 근본적인 기법이다.

# 1. 엄격한 함수와 엄격하지 않은 함수

---

- 비엄격성은 함수의 특성이다.
    - 함수가 인자 중 하나 이상을 평가하지 않기로 선택한다는 뜻이다.
- 엄격한 함수는 항상 모든 인자를 평가한다.
    
    ```kotlin
    fun square(x: Double): Double = x * x
    ```
    
- 비엄격성
    - 쇼트 서킷 불린 함수 `&&`, `||`
        
        ```kotlin
        >>> false && { println("!!"); true }.invoke() // 아무것도 출력하지 않음
        res0: kotlin.Boolean = false
        
        >>> true || { println("!!"); false }.invoke() // 아무것도 출력하지 않음
        res0: kotlin.Boolean = true
        ```
        
    - 코틀린의 `if` 제어 구조: 모든 인자를 평가하지 않기 때문에 엄격하지 않다.
        
        ```kotlin
        val result = if (input.isEmpty()) exitProcess(-1) else input
        ```
        
        ```kotlin
        fun <A> lazyIf(
            cond: Boolean,
            onTrue: () -> A, // A 타입의 지연 값을 표현하는 함수 파라미터 타입은 () -> A
            onFalse: () -> A
        ): A = if (cond) onTrue() else onFalse()
        
        val y = lazyIf((a < 22),
            { println("a") }, // () -> A를 표현하기 위한 함수 리터럴 구문
            { println("b") }
        )
        ```
        

- 썽크(thunk): 평가하지 않은 식을 나타내는 형태
    - 썽크를 강제로 실행(함수를 호출하면서 빈 인자 목록을 전달)하면 식을 평가한 결과를 얻을 수 있다.
    - 이런 문법을 사용하면, 평가되지 않은 채 함수에 전달된 인자는 함수 본문에서 참조될(정확히는 참조되면서 호출될) 때마다 매번 평가된다.
        
        → 코틀린에서는 (기본적으로는) 인자를 평가한 결과를 캐싱하지 않는다.
        
        ```kotlin
        fun maybeTwice(b: Boolean, i: () -> Int) =
            if (b) i() + i() else 0
        ```
        
    - 결과를 한 번만 계산하고 싶다면 새 값을 대입하면서 `lazy` 내장 함수에 위임함으로써 값을 명시적으로 캐싱할 수 있다.
        
        ```kotlin
        fun maybeTwice2(b: Boolean, i: () -> Int) {
            val j: Int by lazy(i)
            if (b) j + j else 0
        }
        ```
        
        - `if` 문 안에서 j가 참조될 때까지 초기화를 미룬다.
        - 최초 참조 시 계산한 결과를 캐싱함으로써 그 이후 j 참조가 일어날 때는 평가를 더 반복하지 않는다.

<aside>
💡 엄격하지 않은 함수 인자는 이름에 의해 전달(pass by name)되지만, 엄격한 함수의 인자는 값에 의해 전달(pass by value)된다고 말한다.

</aside>

<aside>
💡 지연 초기화

- 지연 초기화: 객체 생성, 값 계산 또는 비용이 많이 드는 다른 처리 등을 최초로 필요할 때까지 미루는 전략
- `lazy` 함수에 람다인 썽크 인자를 넘기면서 호출하면 `Lazy<T>` 타입의 인스턴스가 반환된다.
- `T`: 할당할 변수의 타입
- `Lazy` 객체: 지연 프로퍼티를 구현하는 위임 객체
- 위임: `by` 키워드를 통해 표현된다.

```kotlin
fun expensiveOp(): Int = TODO()

val x: Int by lazy { expensiveOp() } // by 키워드를 사용해 lazy가 반환하는 Lazy<Int>를 x에 바인드함

fun useit() =
    if (x > 10) "hi" // x가 조건문에서 평가되면 expensiveOp가 호출되고 그 결과가 캐시됨
    else if (x == 0) "zero" // expensiveOp를 호출하지 않고 캐시된 값을 사용함
    else ("lo")
```

- 결과는 위임 객체 안에 캐시되고, 그 이후 x 평가는 모두 캐시의 이점을 살려 작동한다.
- 썽크 접근은 기본적으로 동시성 락을 사용해 스레드 안전(thread-safe)하게 돼 있다.
    - `lazy`에 대해 다른 스레드 안전 모드를 지정하면 다른 식으로 동작할 수도 있다.
</aside>

# 2. 확장 예제: 지연 리스트

---

```kotlin
sealed class Stream<out A>

data class Cons<out A>(
    val head: () -> A,
    val tail: () -> Stream<A>
) : Stream<A>()

object Empty : Stream<Nothing>()
```

- ex> Stream의 머리(첫 번째 원소)를 뽑아내는 확장 함수
    
    ```kotlin
    fun <A> Stream<A>.headOption(): Option<A> =
        when (this) {
            is Empty -> None
            is Cons -> Some(head()) // head()를 호출해서 명시적으로 head 썽크를 강제로 평가함
        }
    ```
    

## 2.1 스트림을 메모화하고 재계산 피하기

---

- 메모화(memoization): 과도한 평가를 방지
    - 어떤 식을 최초로 평가한 결과를 캐시에 넣음으로써 비싼 계산을 반복하는 일을 방지한다.

- Cons 데이터 생성자를 직접 사용하면 실제로 이 코드는 expensive(y)를 두 번 계산한다.
    
    ```kotlin
    val tl: Stream<String> = TODO()
    fun expensive(c: String): String = TODO()
    val y: String = TODO()
    
    val x = Cons({ expensive(y) }, { tl })
    val h1 = x.headOption()
    val h2 = x.headOption()
    ```
    
- 스마트 생성자: 어떤 데이터 타입을 만들기 위해 호출할 수 있으며, 몇 가지 추가적인 조건이 성립하게 유지해주거나 '실제' 생성자와는 약간 다른 시그니처를 제공하는 함수
    - 관습적으로 스마트 생성자는 기반 클래스의 동반 객체 안에 위치하며, 이름은 상응하는 데이터 생성자의 이름 첫 글자를 소문자로 바꿔 사용한다.
    
    ```kotlin
    fun <A> cons(hd: () -> A, tl: () -> Stream<A>): Stream<A> {
        val head: A by lazy(hd)
        val tail: Stream<A> by lazy(tl)
        return Cons({ head }, { tail })
    }
    ```
    

<aside>
💡 코틀린이 데이터 생성자를 표현하기 위해 하위 타입 지정(subtyping)을 사용한다.

- 하지만 거의 대부분의 경우 Cons나 Empty 타입을 추론하는 대신 Stream을 추론하게 하고 싶다. 이를 위해 스마트 생성자가 기반 타입을 돌려주도록 하는 게 일반적인 트릭이다.
</aside>

- Stream.of 함수에서 이 두 스마트 생성자를 어떻게 사용하는지 볼 수 있다.
    
    ```kotlin
    fun <A> empty(): Stream<A> = Empty
    
    fun <A> of(vararg xs: A): Stream<A> =
        if (xs.isEmpty()) empty()
        else cons({ xs[0] },
            { of(*xs.sliceArray(1 until xs.size)) })
    ```
    
    - 코틀린이 cons의 인자를 썽크로 감싸는 일을 처리해주지 않으므로, 람다로 둘러싸서 Stream에서 강제로 식을 평가할 때까지 계산이 이뤄지지 않게 해야 한다.

## 2.2 스트림 관찰을 위한 도우미 함수

---

### 연습문제 5.1

---

- Stream을 List로 변환하는 함수를 작성하라. 이 함수는 스트림의 모든 값을 강제 계산해 REPL에서 결과를 관찰할 수 있게 해준다. 스트림을 3장에서 개발한 단일 연결 List로 변환해도 된다. 그리고 이 함수나 다른 도우미 함수를 Stream의 확장 메서드로 작성해도 좋다.
    
    ```kotlin
    fun <A> Stream<A>.toList(): List<A> =
        SOLUTION_HERE()
    ```
    
    - 이 함수를 구현할 때 스택 안전성을 고려하라. List에서 구현한 다른 메서드를 사용하거나 꼬리 재귀 제거를 고려하라.

### 연습문제 5.2

---

- Stream의 맨 앞에서 원소를 n개 반환하는 take(n)과 맨 앞에서 원소를 n개 건너뛴 나머지 스트림을 돌려주는 drop(n)을 작성하라.
    
    ```kotlin
    fun <A> Stream<A>.take(n: Int): Stream<A> =
        SOLUTION_HERE()
    
    fun <A> Stream<A>.drop(n: Int): Stream<A> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.3

---

- 주어진 술어와 일치하는 모든 접두사를 돌려주는 takeWhile을 작성하라.
    
    ```kotlin
    fun <A> Stream<A>.takeWhile(p: (A) -> Boolean): Stream<A> =
        SOLUTION_HERE()
    ```
    
    - REPL에서 스트림을 관찰하기 위해 take와 toList를 함께 사용할 수 있다. 예를 들어 `Stream.of(1, 2, 3).take(2).toList()`를 출력해보라. 단위 테스트에서 단언식을 작성할 때도 이 기법이 유용하다.

# 3. 프로그램 기술과 평가 분리하기

---

- 지연 계산을 사용하면 식에 대한 기술을 그 식을 평가하는 것과 분리할 수 있다.
    - 이를 통해 필요한 '더 커다란' 식을 기술하고 그중 일부만 평가하도록 선택하는 강력한 능력을 가질 수 있다.
- ex> Stream 안에 어떤 Boolean 함수를 만족하는 원소가 있는지 검사하는 exists 함수
    
    ```kotlin
    fun exists(p: (A) -> Boolean): Boolean =
        when (this) {
            is Cons -> p(this.head()) || this.tail().exists(p)
            else -> false
        }
    ```
    
    - 여기서 `||`는 두 번재 인자에 대해 엄격하지 않다. 명시적 재귀를 사용해 구현됐다.
- foldRight를 통해 일반적인 재귀를 구현할 수도 있다.
    
    ```kotlin
    fun <B> foldRight(
        z: () -> B,
        f: (A, () -> B) -> B // () -> B 타입은 f가 두 번째 인자로 받는 값이 이름에 의한 파라미터이며 평가되지 않을 수도 있음을 명시함
    ): B =
        when (this) {
            is Cons -> f(this.head()) {
                tail().foldRight(z, f) // f가 두 번째 인자를 평가하지 않는 경우 재귀가 결코 일어나지 않음
            }
    
            is Empty -> z()
        }
    ```
    
    ```kotlin
    fun exists2(p: (A) -> Boolean): Boolean =
        foldRight({ false }, { a, b -> p(a) || b() })
    ```
    
    - exists 정의는 스트림이 크고 모든 원소에 대한 검사 결과가 `false`인 경우에는 스택 안전하지 않다는 점에 유의하라.

### 연습문제 5.4

---

- Stream의 모든 원소가 주어진 술어를 만족하는지 검사하는 forAll을 구현하라. 여러분의 구현은 술어를 만족하지 않는 값을 만나자마자 순회를 최대한 빨리 중단해야만 한다.
    
    ```kotlin
    fun <A> Stream<A>.forAll(p: (A) -> Boolean): Boolean =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.5

---

- foldRight를 사용해 takeWhile을 구현하라.
    
    ```kotlin
    fun <A> Stream<A>.takeWhile(p: (A) -> Boolean): Stream<A> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.6

---

- 어려움: foldRight를 사용해 headOption을 구현하라.
    
    ```kotlin
    fun <A> Stream<A>.headOption(): Option<A> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.7

---

- foldRight를 사용해 map, filter, append를 구현하라. append 메서드는 인자에 대해 엄격하지 않아야만 한다. 필요하면 앞에서 정의한 함수를 사용할지 고려해보라.
    
    ```kotlin
    fun <A, B> Stream<A>.map(f: (A) -> B): Stream<B> =
        SOLUTION_HERE()
    
    fun <A> Stream<A>.filter(f: (A) -> Boolean): Stream<A> =
        SOLUTION_HERE()
    
    fun <A> Stream<A>.append(sa: () -> Stream<A>): Stream<A> =
        SOLUTION_HERE()
    
    fun <A, B> Stream<A>.flatMap(f: (A) -> Stream<B>): Stream<B> =
        SOLUTION_HERE()
    ```
    

- 이런 구현이 점진적(incremental)이라는 점에 유의하라.
    - 해당 Stream 생성이 실제 이뤄지는 시점은 어떤 다른 계산이 결과 Stream의 원소를 살펴볼 때다.
    - 중간 결과를 완전히 인스턴스화하지 않고도 이런 함수를 하나하나 연쇄적으로 호출할 수 있다.
    
    ```kotlin
    import chapter3.Cons as ConsL
    import chapter3.Nil as NilL
    
    Stream.of(1, 2, 3, 4).map { it + 10 }
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList()
    
    Stream.cons({ 11 }, { Stream.of(2, 3, 4).map { it + 10 } })
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList() // map을 첫 번째 원소에 적용
    
    Stream.of(2, 3, 4).map { it + 10 }
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList() // filter를 첫 번째 원소에 적용함. 술어가 false를 반환함
    
    Stream.cons({ 12 }, { Stream.of(3, 4).map { it + 10 } })
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList() // map을 두 번째 원소에 적용
    
    ConsL(36, Stream.of(3, 4).map { it + 10 }
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList()) // filter를 두 번째 원소에 적용하면, 술어가 참을 반환하고, 두 번째 map을 적용하고, 결과의 첫 번째 원소를 생성함
    
    ConsL(36, Stream.cons({ 13 }, { Stream.of(4).map { it + 10 } })
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList() // map을 세 번째 원소에 적용
    )
    
    ConsL(36, Stream.of(4).map { it + 10 }
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList()) // filter를 세 번째 원소에 적용하고, 술어가 거짓을 반환
    
    ConsL(36, Stream.cons({ 14 }, { Stream.empty<Int>().map { it + 10 } })
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList() // map을 마지막 원소에 적용
    )
    
    ConsL(36, ConsL(42, Stream.empty<Int>().map { it + 10 }
        .filter { it % 2 == 0 }
        .map { it * 3 }.toList())) // filter를 마지막 원소에 적용하면, 술어가 참을 반환하고, 두 번째 map을 적용하고, 결과의 두 번째 원소를 생성함
    
    ConsL(36, ConsL(42, NilL)) // 스트림 끝을 표현하는 Empty에 도달함. 이제 map과 filter가 더 할 일이 없음. 빈 스트림은 Nil로 바뀜
    ```
    

<aside>
💡 임포트 별명

- 코틀린은 as 키워드를 사용한 임포트 별명을 제공한다.

```kotlin
import chapter3.Cons as ConsL
import chapter3.Nil as NilL
```

</aside>

- 스트림을 map, filter 등의 고차 함수를 사용해 로직을 조합할 수 있는 '1급 루프'라고 부르기도 한다.

- 중간 스트림이 인스턴스화되지 않기 때문에 필요보다 더 많이 스트림을 처리하게 될지에 대한 걱정 없이 기존 콤비네이터를 새로운 방식으로 재사용할 수 있다.
    - filter가 전체 스트림을 변환하지만, 이런 변환은 지연 계산으로 이뤄지기 때문에 find는 일치하는 원소를 찾자마자 종료된다.
    
    ```kotlin
    fun find(p: (A) -> Boolean): Option<A> =
        filter(p).headOption()
    ```
    

- 스트림 변환의 점진적인 특성은 메모리 사용 측면에서도 중요하다.
    - 메모리를 가능한 한 빨리 재활용하면 전체적으로 프로그램 실행에 필요한 메모리양이 줄어든다.

# 4. 공재귀 함수를 통해 무한한 데이터 스트림 생성하기

---

- 무한 스트림도 점진적으로 계산이 이뤄진다.
    
    ```kotlin
    fun ones(): Stream<Int> = Stream.cons({ 1 }, { ones() })
    ```
    
    - ones는 무한 시퀀스를 생성한다.
    
    ![5.1 ones 함수는 점진적이기 때문에 요청이 있을 때마다 1을 내놓는 무한한 스트림을 만든다.](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A5%E1%86%B7%E1%84%80%E1%85%A7%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%E1%84%80%E1%85%AA%20%E1%84%8C%E1%85%B5%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%89%E1%85%A5%E1%86%BC%2033e531cbff564d0db8286e8202371a4c/Untitled.png)
    
    5.1 ones 함수는 점진적이기 때문에 요청이 있을 때마다 1을 내놓는 무한한 스트림을 만든다.
    

<aside>
💡 일반적인 재귀 루프를 사용해 스택 안전한 forAll을 작성할 수도 있다.

</aside>

### 연습문제 5.8

---

- ones를 약간 일반화해서 정해진 값으로 이뤄진 무한 Stream을 돌려주는 constant 함수를 작성하라.
    
    ```kotlin
    fun <A> constant(a: A): Stream<A> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.9

---

- n부터 시작해서 n+1, n+2 등을 차례로 내놓는 무한한 정수 스트림을 만들어내는 함수를 작성하라(코틀린에서 `Int` 타입은 32비트 부호가 있는 정수이므로 이 스트림은 약 40억 개의 정수를 주기적으로 반복하면서 양수와 음수를 오간다).
    
    ```kotlin
    fun from(n: Int): Stream<Int> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.10

---

- 0, 1, 1, 2, 3, 5, 9처럼 변하는 무한한 피보나치 수열을 만들어내는 fibs 함수를 작성하라.
    
    ```kotlin
    fun fibs(): Stream<Int> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.11

---

- unfold라는 더 일반적인 스트림 구성 함수를 작성하라. 이 함수는 초기 상태를 첫 번째 인자로 받고, 현재 상태로부터 다음 상태와 스트림상의 다음 값을 만들어내는 함수를 두 번째 인자로 받는다.
    
    ```kotlin
    fun <A, S> unfold(z: S, f: (S) -> Option<Pair<A, S>>): Stream<A> =
        SOLUTION_HERE()
    ```
    

- unfold 함수는 공재귀 함수라고 불리는 함수의 예다.
- 재귀적 함수는 데이터를 소비하는 반면, 공재귀 함수는 데이터를 생성한다.
    - 재귀적 함수는 더 작은 입력에 대해 재귀를 수행하다가 재귀를 끝내는 반면,
    - 공재귀 함수는 생산적인 성질을 유지하는 한 끝나지 않는다.
        - 공재귀: 가드가 있는 재귀(guarded recursion)
        - 생산성: 공종료(cotermination)

### 연습문제 5.12

---

- unfold를 사용해 fibs, from, constant, ones를 구현하라.
    - 재귀적 버전에서 `fun ones(): Stream<Int> = Stream.cons({ 1 }, { ones() })`로 공유를 사용했던 것과 달리 unfold를 사용해 constant와 ones를 정의하면 공유를 쓰지 않게 된다.
    - 재귀 정의는 순회를 하는 동안에도 스트림에 대한 참조를 유지하기 때문에 메모리를 상수로 소비하지만, unfold 기반 구현은 그렇지 않다.
    - 공유 유지는 극히 미묘하며 타입을 통해 추적하기 어려우므로, 스트림을 사용해 프로그래밍을 할 때 일반적으로 의존하는 특성은 아니다.
        - ex> `xs.map { x -> x}`를 호출해도 공유가 깨진다.
    
    ```kotlin
    fun fibs(): Stream<Int> =
        SOLUTION_HERE()
    
    fun from(n: Int): Stream<Int> =
        SOLUTION_HERE()
    
    fun <A> constant(n: A): Stream<A> =
        SOLUTION_HERE()
    
    fun ones(): Stream<Int> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.13

---

- unfold를 사용해 map, take, takeWhile, zipWith, zipAll을 구현하라. zipAll 함수는 두 스트림 중 한쪽에 원소가 남아 있는 한 순회를 계속해야 하며, 각 스트림을 소진했는지 여부를 표현하기 위해 Option을 사용한다.
    
    ```kotlin
    fun <A, B> Stream<A>.map(f: (A) -> B): Stream<B> =
        SOLUTION_HERE()
    
    fun <A> Stream<A>.take(n: Int): Stream<A> =
        SOLUTION_HERE()
    
    fun <A> Stream<A>.takeWhile(p: (A) -> Boolean): Stream<A> =
        SOLUTION_HERE()
    
    fun <A, B, C> Stream<A>.zipWith(
        that: Stream<B>,
        f: (A, B) -> C
    ): Stream<C> =
        SOLUTION_HERE()
    
    fun <A, B> Stream<A>.zipAll(
        that: Stream<B>
    ): Stream<Pair<Option<A>, Option<B>>> =
        SOLUTION_HERE()
    ```
    

### 연습문제 5.14

---

- 어려움: 이전에 작성한 함수를 사용해 startsWith를 구현하라. 이 함수는 어떤 Stream이 다른 Stream의 접두사인지 여부를 검사해야 한다. 예를 들어 `Stream(1, 2, 3).startsWith(Stream(1, 2))`는 `true`다.
    
    ```kotlin
    fun <A> Stream<A>.startsWith(that: Stream<A>): Boolean =
        SOLUTION_HERE()
    ```
    
    - 팁: 이 문제는 이번 장의 앞부분에서 unfold를 사용해 개발한 함수만을 사용해 구현할 수 있다.

### 연습문제 5.15

---

- unfold를 사용해 tails를 구현하라. tails는 주어진 Stream의 모든 접미사를 돌려준다. 이때 원래 Stream과 똑같은 스트림을 가장 먼저 돌려준다. 예를 들어 `Stream.of(1, 2, 3)`에 대해 tails는 `Stream.of(Stream.of(1, 2, 3), Stream.of(2, 3), Stream.of(3), Stream.empty())`를 반환한다.
    
    ```kotlin
    fun <A> Stream<A>.tails(): Stream<Stream<A>> =
        SOLUTION_HERE()
    ```
    

- 이제 작성한 함수들을 사용해 다음과 같이 hasSubsequence를 구현할 수 있다.
    
    ```kotlin
    fun <A> hasSubsequence(s: Stream<A>): Boolean =
        this.tails().exists { it.startsWith(s) }
    ```
    
    - 이 구현은 내포된 루프를 사용하고 각 루프를 일찍 종료시키는 로직을 포함하는 더 복잡한 구현과 똑같은 단계를 수행한다.
- 😃 지연 계산을 사용하면 이 함수를 더 간단한 구성 요소로부터 합성할 수 있고, 그러면서도 여전히 더 특화된(그리고 더 장황한) 구현만큼 효율을 유지할 수 있다.

### 연습문제 5.16

---

- 어려움/선택적: tails를 일반화해서 scanRight를 만들라. scanRight는 foldRight와 마찬가지로 중간 결과로 이뤄진 스트림을 반환한다. 예를 들면 다음과 같다.
    
    ```kotlin
    fun <A, B> Stream<A>.scanRight(z: B, f: (A, () -> B) -> B): Stream<B> =
        SOLUTION_HERE()
    
    >>> Stream.of(1, 2, 3).scanRight(0, { a, b -> a + b }).toList()
    
    res1: chapter3.List<kotlin.Int> =
        Cons(head=6,tail=Cons(head=5,tail=Cons(head=3,tail=Cons(head=0,tail=Nil))))
    ```
    
    - 이 예제는 `List.of(1+2+3+0, 2+3+0, 3+0, 0)`이라는 식과 같다. 여러분의 함수는 중간 결과를 재사용해서 원소가 n개인 Stream을 순회하는 데 걸린 시간이 n에 선형적으로 비례해야만 한다. unfold를 사용해 이 함수를 구현할 수 있을까? 구현할 수 있다면 어떻게 구현할 수 있고, 구현할 수 없다면 왜 구현할 수 없을까? 여러분이 지금까지 작성한 다른 함수를 사용해 이 함수를 구현할 수 있을까?

# 5. 결론

---

- 비엄격성은 효율성 회복보다 훨씬 더 큰 아이디어이기도 하다.
    - 비엄격성은 식에 대한 기술과 그 기술을 언제 어떻게 평가해야 할지를 분리해줌으로써 모듈화를 향상시킬 수 있기 때문이다.
    - 관심사 분리를 통해 식을 기술한 내용을 여러 문맥에서 재사용할 수 있고, 식의 서로 다른 부분을 평가해 서로 다른 결과를 얻을 수 있다.

# 요약

---

- 엄격한 식은 변수에 바인드되는 시점에 평가된다.
    - 간단한 식의 경우 이런 동작을 받아들일 만하지만 복잡한 계산의 경우 최대한 연산을 미뤄야 한다.
- 비엄격성이나 지연 계산은 값을 처음 참조하는 시점까지 계산을 지연시킨다.
    - 이를 통해 값비싼 계산을 꼭 필요할 때 요구에 따라 평가할 수 있다.
- 썽크는 평가가 이뤄지지 않은 식을 표현하는 형태다.
- 식을 썽크로 감싸면 지연 초기화를 달성할 수 있다.
    - 필요하면 나중에 썽크를 강제 실행할 수 있다.
- Stream 데이터 타입을 사용하면 봉인된 Cons와 Empty 타입을 사용하는 지연 리스트 구현을 모델링할 수 있다.
- 메모화는 식을 최초로 평가한 결과를 캐시에 저장함으로써 식이 여러 번 평가되는 것을 방지하는 기법이다.
- 스마트 생성자는 실제 생성자와 약간 다른 시그니처를 제공하는 함수다.
    - 스마트 생성자는 원래 클래스 생성자가 제공하는 요구 조건에 더해 몇 가지 요구 조건을 보장해줄 수 있다.
- 지연 계산을 적용할 때는 기술과 평가라는 관심사를 분리할 수 있다.
    - 이 둘을 분리하면 더 큰 식을 기술하고 필요에 따라 훨씬 작은 부분만 평가할 수 있다.
- 무한 스트림을 데이터를 점진적으로 생성하는 공재귀 함수를 사용해 만들 수도 있다.
    - unfold 함수는 이런 공재귀 스트림 생성자다.
