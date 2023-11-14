# 4장. 예외를 사용하지 않고 오류 다루기

- 왜 예외를 던지는 게 바람직하지 않은 동작일까?
    - 예외가 던져지면, 제어가 프로그램의 현재 지점으로부터 벗어난 다른 쪽에 위임되면서 호출 스택을 거슬러 올라가며 전파된다.
    - 제어 상실(loss of control): 예외가 처리되지 않아 프로그램이 중단되거나, 호출 스택의 위에 있는 어떤 코드가 예외를 잡아서 예외를 처리하는 것 중 하나를 뜻한다.
- 함수형 프로그래밍에서 예외적인 경우를 어떻게 처리할 수 있을까?
    - 실패와 예외를 일반적인 값으로 표현할 수 있다.
- 기존 함수형 언어로부터 이런 타입을 포팅한 코틀린 함수형 프로그래밍 라이브러리로 애로우(Arrow)가 있다.
    - 애로우에서는 현재 `Option`이 사용 중단 예정(deprecated)이지만, 함수형 프로그래밍 커뮤니티에서는 여전히 `Option`을 많이 쓰고 있으므로 이 책에 `Option`을 포함시켰다.

# 1. 예외를 던지는 것의 문제점

---

- 예외를 던지는 함수
    
    ```kotlin
    fun failingFn(i: Int): Int {
        val y: Int = throw Exception("boom") // Int 타입의 선언이 예외를 던짐
        return try {
            val x = 42 + 5
            x + y
        } catch (e: Exception) {
            43 // 도달할 수 없는 코드여서 43이 반환되지 않음
        }
    }
    ```
    
- y가 참조 투명하지 않음을 증명할 수 있다.
    - 참조 투명성(RT)인 모든 식은 그 식이 가리키는 값으로 치환할 수 있고, 이런 치환이 프로그램의 의미를 보존한다.
    
    ```kotlin
    fun failingFn2(i: Int): Int =
        try {
            val x = 42 + 5
            x + (throw Exception("boom!")) as Int // Exception을 던지는 코드에 임의의 타입을 지정할 수 있음. 여기서는 Int를 지정함
        } catch (e: Exception) {
            43 // 예외가 잡히고 43이 반환됨
        }
    ```
    
    - `try` 블록 안에서 예외가 발생하기 때문에 결과가 달라진다.

- 예외에는 크게 두 가지 문제가 있다.
    - 예외는 RT를 깨고 문맥 의존성을 만든다.
        - 이로 인해 치환 모델이 제공하는 단순한 추론을 벗어나며, 혼란스러운 예외 기반의 코드를 작성할 수 있게 된다.
        - 함수형 프로그래밍에서는 오류를 회복할 수 없는 극단적인 상황이 아닌 한, 예외를 던지는 일을 피한다.
    - 예외는 타입 안전하지 않다.
        - 예외가 발생하는지 검사하지 않으면, 예외가 던져진다는 사실을 실행 시점이 돼서야 감지할 수 있다.

😀 오류 코드를 사용하는 대신, '값이 정의될 수도 있음'을 표현하는 새로운 제네릭 타입을 도입하고 고차 함수를 사용해 오류를 처리하고 전파하는 일반적인 패턴을 캡슐화한다.

# 2. 예외에 대한 문제가 있는 대안

---

- 예외를 사용할 법한 실질적인 상황을 가정하고 예외 대신 어떤 방법을 사용할 수 있는지 살펴보자.
- 부분 함수(partial function)의 예
    
    ```kotlin
    fun mean(xs: List<Double>): Double =
        if (xs.isEmpty())
            throw ArithmeticException("mean of empty list!") // xs가 비어 있으면 ArithmeticException을 던짐
        else xs.sum() / length(xs) // xs가 비어 있지 않으면 올바른 결과를 반환
    ```
    
    - 부분 함수: 입력 중 일부에 대해 결과가 정의되지 않는 함수

## 2.1 센티넬 값

---

- 첫 번째 접근 방법은 예외를 던지는 대신에 `Double` 타입의 가짜 값을 반환하는 것
- 우리는 다음과 같은 이유로 이를 거부한다.
    - 오류가 조용히 전파될 수 있다.
        - 호출한 쪽에서 조건 검사를 잊어버릴 수 있는데, 컴파일러가 이에 대해 경고하지 않으므로 그 이후의 코드가 제대로 작동하지 않을 수 있다.
    - 호출 지점에서 명시적으로 `if`를 사용해 '실제' 결과가 반환됐는지 검사해야 하므로 보일러플레이트 코드가 상당히 많이 필요하다.
    - 다형적인 코드에 적용할 수 없다.
        - 몇몇 출력 타입에 대해 적절한 센티넬 값이 존재하지 않는 경우도 있다.
            - 기본 타입이 아닌 값에만 사용할 수 있는 `null`을 여기에 쓸 수는 없다.
    - 호출하는 쪽에서 특정 정책이나 호출 규약을 지키도록 요구한다.

## 2.2 디폴트 값 제공

---

- 두 번째 대안은 입력을 처리할 수 없는 경우에 돌려줄 디폴트 값을 호출자가 추가 인자로 제공하게 하는 것이다.
    
    ```kotlin
    fun mean(xs: List<Double>, onEmpty: Double) =
        if (xs.isEmpty()) onEmpty // xs가 비어 있는 경우, 디폴트 값을 돌려줌
        else xs.sum() / xs.size() // xs가 비어 있지 않은 경우 올바른 결과를 돌려줌
    ```
    
    - 이렇게 하면 mean이 전함수(total function)가 된다.
    - 하지만 mean 함수를 직접 호출하는 쪽에서 함수 결과가 정의되지 않는 경우가 언제인지 이해해야만 하며, 디폴트 값이 `Double`로 한정된다는 단점이 있다.
- 우리에게는 정의되지 않은 경우를 어떻게 처리할지 결정하는 것을 최대한 연기해 가장 적합한 수준에서 다룰 수 있게 해주는 방법이 필요하다.

# 3. `Option`으로 성공 상황 인코딩하기

---

- 바람직하다고 생각하는 접근 방법: 함수의 반환 타입에 그 함수가 항상 결과를 제공하는지 여부를 명시적으로 표현하는 것
    - 호출하는 쪽으로 오류 처리 전략을 미루는 것

```kotlin
sealed class Option<out A>

data class Some<out A>(val get: A) : Option<A>()

object None : Option<Nothing>()
```

- `Option`에는 두 가지 경우가 있다.
    - 정의되지 않음. None으로 이 경우를 표현한다.
    - 정의됨. Some으로 이 경우를 표현한다.

- Option을 사용하면 다음과 같이 mean을 정의할 수 있다.
    
    ```kotlin
    fun mean(xs: List<Double>): Option<Double> =
        if (xs.isEmpty()) None // xs가 비어 있으면 None 값을 반환
        else Some(xs.sum() / xs.size()) // 올바른 결과를 감싼 Some 값을 반환
    ```
    
    - 이제는 반환 타입이 결과가 항상 정의되지 않을 수도 있음을 반영한다.

## 3.1 `Option` 사용 패턴

---

- Option이 쓰이는 몇 가지 상황
    - getOption을 사용해 주어진 키를 맵에서 찾는다.
    - 리스트나 다른 이터러블(iterable)에 대해 정의된 firstOrNone과 lastOrNone은 시퀀스가 비어 있지 않을 때 첫 번째 원소나 마지막 원소가 들어 있는 Option을 반환한다.

<aside>
💡 널이 될 수 있는 타입과 `Option` 비교

- 코틀린은 Option 개념을 도입하지 않기로 결정했다. 코틀린을 만든 이들은 가벼운 래퍼(wrapper)를 인스턴스화하는 것을 성능상 비용이라고 언급한다.
    - 코틀린에서는 `null` 값을 다루는 대안을 널이 될 수 있는 타입(nullable type)이라는 개념으로 도입했다.
</aside>

### `Option`에 대한 기본 함수

---

- 여기서는 가능하면 확장 메서드를 사용한다.
    - 확장 메서드를 사용하면 fn(obj, arg1) 대신 obj.fn(arg1)이나 obj fn arg1으로 함수를 호출할 수 있다.

```kotlin
fun <A, B> Option<A>.map(f: (A) -> B): Option<B> =
    SOLUTION_HERE() // Option이 None이 아닌 경우 f를 적용해 A 타입 값을 B 타입으로 변환함

fun <A, B> Option<A>.flatMap(f: (A) -> Option<B>): Option<B> =
    SOLUTION_HERE() // Option이 None이 아닌 경우, 실패할 수도 있는 f를 적용해 A 타입 값을 B 타입으로 변환함

fun <A> Option<A>.getOrElse(default: () -> A): A =
    SOLUTION_HERE() // Option이 None인 경우 디폴트 값을 반환함

fun <A> Option<A>.orElse(ob: () -> Option<A>): Option<A> =
    SOLUTION_HERE() // Option이 None인 경우 디폴트 옵션을 반환함

fun <A> Option<A>.filter(f: (A) -> Boolean): Option<A> =
    SOLUTION_HERE() // 술어 f를 만족하지 않으면 Some을 None으로 변환함
```

### 연습문제 4.1

---

- 앞에 있는 Option에 대한 모든 함수를 구현하라. 각 함수를 구현할 때 각 함수의 의미가 무엇이고 어떤 상황에서 각 함수를 사용할지 생각해보라. 나중에 각 함수를 언제 사용할지 살펴본다. 다음은 이 연습문제를 풀기 위한 몇 가지 힌트다.
    - 매칭을 사용해도 좋다. 하지만 map과 getOrElse 이외의 모든 함수를 매칭 없이 구현할 수 있다.
    - map과 flatMap의 경우 타입 시그니처만으로 구현을 결정할 수 있다.
    - getOrElse는 Option이 Some인 경우 결과를 반환하지만 Option이 None인 경우 주어진 디폴트 값을 반환한다.
    - orElse는 첫 번째 Option의 값이 정의된 경우(즉, Some인 경우) 그 Option을 반환한다. 그렇지 않은 경우 두 번째 Option을 반환한다.

### 기본 `Option` 함수의 사용 시나리오

---

- 패턴 매칭으로 문제를 해결하기 전에 여기서 본 함수들이 캡슐화해주는 패턴을 인식할 수 있는지 살펴보라.
    
    ```kotlin
    data class Employee(
        val name: String,
        val department: String,
        val manager: Option<String>
    )
    
    fun lookupByName(name: String): Option<Employee> = TODO()
    
    fun timDepartment(): Option<String> =
        lookupByName("Tim").map { it.department }
    ```
    
    ![4.2 map을 사용해 Option의 내용 변환하기](./image/4/Untitled.png)
    
    4.2 map을 사용해 Option의 내용 변환하기
    
    - lookupByName("Tim")이 None을 반환하면 map은 나머지 계산을 중단하고 it.department 함수를 호출하지 않는다.
- manager에 대해서도 그냥 map을 쓰면 다루기 힘든 Option<Option<String>>이 생긴다.
    
    ```kotlin
    val unwieldy: Option<Option<String>> =
        lookupByName("Tim").map { it.manager }
    ```
    
    ```kotlin
    val manager: String = lookupByName("Tim")
        .flatMap { it.manager }
        .getOrElse { "Unemployed" }
    ```
    
    ![4.3 flatMap을 사용해 Option<Option<>> 내용 변환하고 펼치기](./image/4/Untitled%201.png)
    
    4.3 flatMap을 사용해 Option<Option<>> 내용 변환하고 펼치기
    

### 연습문제 4.2

---

- flatMap을 사용해 variance 함수를 구현하라. 시퀀스의 평균이 m이면, 분산(variance)은 시퀀스의 원소를 x라 할 때 x-m을 제곱한 값의 평균이다. 코드로 쓰면 (x - m).pow(2)라 할 수 있다. 리스트 4.2에서 만든 mean 메서드를 사용해 이 함수를 구현할 수 있다.
    
    ```kotlin
    fun mean(xs: List<Double>): Option<Double> =
        if (xs.isEmpty()) None
        else Some(xs.sum() / xs.size())
    
    fun variance(xs: List<Double>): Option<Double> =
        SOLUTION_HERE()
    ```
    
    - flatMap을 사용하면 각 단계가 실패할 수도 있는 여러 단계로 이뤄진 계산을 구성할 수 있다.
    - filter를 사용하면 성공적인 값이 술어를 만족시키지 않을 때 성공을 실패로 변환할 수 있다.
        
        ```kotlin
        val dept: String = lookupByName("Tim")
            .map { it.department }
            .filter { it != "Accounts" }
            .getOrElse { "Unemployed" }
        ```
        

- 오류를 공통 값으로 반환하는 것이 편리할 수도 있다.
    - 고차 함수를 사용하면 예외를 사용할 때와 마찬가지로 오류 처리 로직을 통합할 수 있다.

## 3.2 Option 합성, 끌어올리기 및 예외 기반 API 감싸기

---

```kotlin
fun <A, B> lift(f: (A) -> B): (Option<A>) -> Option<B> =
    { oa -> oa.map(f) }
```

- 우리 주변에 널려 있는 모든 함수를 단일 Option 값 문맥 안에서 작동하는 함수로 변환할 수 있다.

```kotlin
val absO: (Option<Double>) -> Option<Double> =
    lift { kotlin.math.abs(it) }
```

- 어떤 함수든 이런 끌어올림이 가능하다.
    
    ![4.4 간단한 함수를 Option 타입을 받고 반환하도록 끌어올리기](./image/4/Untitled%202.png)
    
    4.4 간단한 함수를 Option 타입을 받고 반환하도록 끌어올리기
    

- ex> 자동차 보험 회사의 할인율(보험료 산정 기준)을 계산하는 함수
    
    ```kotlin
    /**
     * 두 가지 핵심 요소로부터 연간 보험 할인율을 계산하는 최고 기밀 공식
     */
    fun insuranceRateQuote(
        age: Int,
        numberOfSpeedingTickets: Int
    ): Double = TODO()
    ```
    
    - 사용자가 웹 폼에 입력한 나이와 속도 위반 딱지 수가 단순한 문자열로 들어오기 때문에 파싱에 실패할 수도 있다.(`NumberFormatException`)
    
    ```kotlin
    fun parseInsuranceRateQuote(
        age: String,
        speedingTickets: String
    ): Option<Double> {
    
        val optAge: Option<Int> = catches { age.toInt() }
    
        val optTickets: Option<Int> =
            catches { speedingTickets.toInt() }
    
        //return insuranceRateQuote(optAge, optTickets) // 타입이 불일치해서 타입 검사를 통과하지 못함
    }
    
    fun <A> catches(a: () -> A): Option<A> = // a를 평가하는 과정에서 발생하는 예외를 모두 잡아내 None으로 변환하기 위해 엄격하지 않은 A를 받음
        try {
            Some(a()) // Some의 내부에서 엄격하지 않은 파라미터 a를 호출
        } catch (e: Throwable) { // 오류 e에 대한 정보를 버림. 4.4절에서 Either를 사용해 개선할 예정
            None
        }
    ```
    
    - 하지만 문제가 있다. optAge와 optTicket을 Option<Int>로 파싱한 다음, 현재 두 Int를 받게 돼 있는 insuranceRateQuote를 어떻게 호출할 수 있을까?
    - insuranceRateQuote를 끌어올려서 두 가지 선택적인 값을 처리하게 만들어야 한다.

### 연습문제 4.3

---

- 두 Option 값을 이항 함수를 통해 조합하는 제네릭 함수 map2를 작성하라. 두 Option 중 어느 하나라도 None이면 반환값도 None이다. 다음은 map2의 시그니처다.
    
    ```kotlin
    fun <A, B, C> map2(a: Option<A>, b: Option<B>, f: (A, B) -> C): Option<C> =
        SOLUTION_HERE()
    ```
    

- 한 가지 단점이 있다. 두 Option 중 어느 하나 또는 둘 다 None이면, 전체적으로 None이 반환되기 때문에 두 옵션 중 어느 쪽이 실패했는지에 대한 정보가 사라진다.

### 연습문제 4.4

---

- 원소가 Option인 리스트를 원소가 리스트인 Option으로 합쳐주는 sequence 함수를 작성하라. 반환되는 Option의 원소는 원래 리스트에서 Some인 값들만 모은 리스트다. 원래 리스트 안에 None이 단 하나라도 있으면 결괏값이 None이어야 하며, 그렇지 않으면 모든 정상 값이 모인 리스트가 들어있는 Some이 결괏값이어야 한다. 시그니처는 다음과 같다.
    
    ```kotlin
    fun <A> sequence(xs: List<Option<A>>): Option<List<A>> =
        SOLUTION_HERE()
    ```
    
- 이 문제는 객체지향 스타일로 함수를 작성하는 게 적합하지 않다고 확실히 알 수 있는 경우다. 이 함수는 List의 메서드가 돼서는 안 되고(List는 Option에 대해 알 필요가 없어야만 한다), Option의 메서드가 될 수도 없다. 따라서 이 함수를 Option의 동반 객체 안에 넣어야 한다.

- ex> Option<Int>로 파싱하고 싶은 String 값으로 이뤄진 리스트가 있다면 어떻게 해야 할까?
    
    ```kotlin
    fun parseInts(xs: List<String>): Option<List<Int>> =
        sequence(xs.map { str -> catches { str.toInt() } })
    ```
    
    - 이 코드는 리스트를 두 번(한 번은 String을 Option<Int>로 변환하기 위해, 다른 한 번은 Option<Int> 값을 Option<List<Int>>로 변환하기 위해) 순회하기 때문에 효율적이지 않다.

### 연습문제 4.5

---

- traverse 함수를 구현하라. map을 한 다음에 sequence를 하면 간단하지만, 리스트를 단 한 번만 순회하는 더 효율적인 구현을 시도해보라. 코드를 작성하고 나면 sequence를 traverse를 사용해 구현하라.
    
    ```kotlin
    fun <A, B> traverse(
        xa: List<A>,
        f: (A) -> Option<B>
    ): Option<List<B>> =
        SOLUTION_HERE()
    ```
    

## 3.3 `Option`과 `for` 컴프리헨션 사용하기

---

- 여러 언어에서 `for` 컴프리헨션 또는 모나드 컴프리헨션을 제공한다.
    - `flatMap`과 `map` 함수를 연쇄적으로 호출해 최종 값을 얻는 코드에 대해 제공하는 문법 설탕 요소
    - `for` 컴프리헨션을 쓸 수 있으면 함수 호출을 내포시켜가면서 처리하는 것보다 훨씬 더 사용하기 즐겁고 간결한 (명령형 코드를 닮은) 구문을 사용할 수 있다.

- 코틀린은 기본 설치 상태에서 `for` 컴프리헨션을 제공하지 않는다.
    - 애로우에서는 다양한 데이터 타입과 함께 쓸 수 있고 다른 언어가 제공하는 `for` 컴프리헨션과 비슷하게 작동하는 fx 블록으로 `for` 컴프리헨션을 구현했다.
    - AS-IS
        
        ```kotlin
        fun <A, B, C> map2(
            oa: Option<A>,
            ob: Option<B>,
            f: (A, B) -> C
        ): Option<C> =
            oa.flatMap { a ->
                ob.map { b ->
                    f(a, b)
                }
            }
        ```
        
    - TO-BE
        
        ```kotlin
        fun <A, B, C> map2(
            oa: Option<A>,
            ob: Option<B>,
            f: (A, B) -> C
        ): Option<C> =
            Option.fx {
                val a = oa.bind()
                val b = ob.bind()
                f(a, b)
            }
        ```
        
        - 컴파일러는 `bind()` 문장을 각각의 `Option`에 대한 flatMap 호출로 바꾸고, 마지막 식을 결과를 돌려주는 `map` 호출로 바꾼다.

# 4. 성공과 실패 조건을 `Either`로 인코딩하기

---

- `Either`를 쓰면 실패의 이유를 추적할 수 있다.
    
    ```kotlin
    sealed class Either<out E, out A>
    
    data class Left<out E>(val value: E) : Either<E, Nothing>()
    
    data class Right<out A>(val value: A) : Either<Nothing, A>()
    ```
    
    - `Either`를 사용할 때는 관습적으로 `Right` 데이터 생성자를 사용해 성공을 표현하고, `Left`로 실패를 표현한다.

- ex> mean
    - AS-IS
        
        ```kotlin
        fun mean(xs: List<Double>): Double =
                if (xs.isEmpty())
                    throw ArithmeticException("mean of empty list!")
                else xs.sum() / length(xs)
        ```
        
    - TO-BE
        
        ```kotlin
        fun mean(xs: List<Double>): Either<String, Double> =
            if (xs.isEmpty())
                Left("mean of empty list!")
            else Right(xs.sum() / xs.size())
        ```
        

- 새 `Either` 생성을 돕기 위해 다시 `catches`라는 함수를 작성할 것이다.
    - 던져진 예외를 잡아서 값으로 변환하는 과정의 공통 패턴을 분리해준다.
    
    ```kotlin
    fun <A> catches(a: () -> A): Either<Exception, A> =
        try {
            Right(a())
        } catch (e: Exception) {
            Left(e)
        }
    ```
    

### 연습문제 4.6

---

- Right 값에 대해 활용할 수 있는 map, flatMap, orElse, map2를 구현하라.
    
    ```kotlin
    fun <E, A, B> Either<E, A>.map(f: (A) -> B): Either<E, B> =
        SOLUTION_HERE()
    
    fun <E, A, B> Either<E, A>.flatMap(f: (A) -> Either<E, B>): Either<E, B> =
        SOLUTION_HERE()
    
    fun <E, A> Either<E, A>.orElse(f: () -> Either<E, A>): Either<E, A> =
        SOLUTION_HERE()
    
    fun <E, A, B, C> map2(
        ae: Either<E, A>,
        be: Either<E, B>,
        f: (A, B) -> C
    ): Either<E, C> =
        SOLUTION_HERE()
    ```
    

## 4.1 `Either`를 `for` 컴프리헨션에서 사용하기

---

```kotlin
suspend fun String.parseToInt(): arrow.core.Either<Throwable, Int> = // parseToInt 확장 메서드를 String에 추가
    arrow.core.Either.catch { this.toInt() } // Either.catch 메서드를 사용해 Either<Throwable, Int>를 만듬

suspend fun parseInsuranceRateQuote( // 메서드 앞에 suspend가 붙으면 이 메서드의 하위 처리가 블록시킬 수 있다는 뜻임
    age: String,
    numberOfSpeedingTickets: String
): arrow.core.Either<Throwable, Double> {
    val ae = age.parseToInt() // 확장 메서드를 사용해 Either를 만듬
    val te = numberOfSpeedingTickets.parseToInt()
    return arrow.core.Either.fx { // Either.fx를 사용해 for 컴프리헨션을 시작
        val a = ae.bind() // bind()를 사용해 오른쪽으로 기울어진 Either에 대해 flatMap()을 사용함
        val t = te.bind()
        insuranceRateQuote(a, t) // 성공 시 마지막으로 insuranceRateQuote를 계산한 값을 Either.Right에 전달
    }
}
```

- 일시 중단 함수(suspending function): 일반 코틀린 함수 앞에 `suspend`라는 변경자가 붙은 함수로, 이 함수가 오래 실행되는 하위 프로세스로 인해 일시 중단될 수도 있다는 사실을 표시한다.

### 연습문제 4.7

---

- Either에 대한 sequence와 traverse를 구현하라. 두 함수는 오류가 생긴 경우 최초로 발생한 오류를 반환해야 한다.
    
    ```kotlin
    fun <E, A, B> traverse(
        xs: List<A>,
        f: (A) -> Either<E, B>
    ): Either<E, List<B>> =
        SOLUTION_HERE()
    
    fun <E, A> sequence(es: List<Either<E, A>>): Either<E, List<A>> =
        SOLUTION_HERE()
    ```
    

```kotlin
data class Name(val value: String)
data class Age(val value: Int)
data class Person(val name: Name, val age: Age)

fun mkName(name: String): Either<String, Name> =
    if (name.isBlank()) Left("Name is empty.")
    else Right(Name(name))

fun mkAge(age: Int): Either<String, Age> =
    if (age < 0) Left("Age is out of range.")
    else Right(Age(age))

fun mkPerson(name: String, age: Int): Either<String, Person> =
    map2(mkName(name), mkAge(age)) { n, a -> Person(n, a) }
```

### 연습문제 4.8

---

- 4.8에서는 이름과 나이 모두 잘못되더라도 map2가 오류를 하나만 보고할 수 있다. 두 오류를 모두 보고하게 하려면 어디를 바꿔야 할까? map2나 mkPerson의 시그니처를 바꿔야 할까, 아니면 Either보다 이 추가 요구 사항을 더 잘 다룰 수 있는 추가 구조를 포함하는 새로운 데이터 타입을 만들어야 할까? 이 데이터 타입에 대해 orElse, traverse, sequence는 어떻게 다르게 동작해야 할까?

# 요약

---

- 예외를 던지는 것은 바람직하지 않고 참조 투명성을 깬다.
- 복원이 불가능한 극단적인 경우에만 예외를 던져야 한다.
- 예외적인 경우를 캡슐화하는 데이터 타입을 사용하면 순수 함수형 오류 처리를 달성할 수 있다.
- 성공을 `Some`으로 인코딩하고 실패를 비어 있는 `None`으로 인코딩하고 싶을 때 `Option` 데이터 타입이 편리하다.
- `Either` 데이터 타입을 사용하면 성공을 `Right`로, 실패를 `Left`로 인코딩할 수 있다.
- 예외를 던지지 않는 함수를 끌어올려서 `Option`이나 `Either` 타입을 따르게 할 수 있다.
- 일련의 `Option`과 `Either` 연산은 맨 처음 실패에 직면했을 때 중단될 수 있다.
- `for` 컴프리헨션은 일련의 `map`, `flatMap` 콤비네이터 호출을 매끄럽게 표현하도록 해주는 언어 요소다.
- 코틀린에서 사용할 수 있는 함수형 프로그래밍 보조 라이브러리인 애로우 라이브러리에는 코드를 단순화하기 위해 바인딩 메서드를 사용해 `for` 컴프리헨션을 처리해주는 `Either`가 있다.
