# Chapter 7. 연산자 오버로딩과 기타 관례

## 1. 산술 연산자 오버로딩

---

- 코틀린에서 관례를 사용하는 가장 단순한 예는 산술 연산자다.****
    - 자바에서는 원시 타입에 대해서만 산술 연산자를 사용할 수 있고, 추가로 `String`에 대해 `+` 연산자를 사용할 수 있다.

### 1-1. 이항 산술 연산 오버로딩

---

- plus 연산자 구현하기
    
    ```kotlin
    // case 1. 연산자를 자체 함수로 정의하기
    data class Point(val x: Int, val y: Int) {
        operator fun plus(other: Point): Point {
            return Point(x + other.x, y + other.y)
        }
    }
    
    fun main(args: Array<String>) {
        val p1 = Point(10, 20)
        val p2 = Point(30, 40)
        println(p1 + p2) // Point(x=40, y=60)
    }
    
    // case 2. 연산자를 확장 함수로 정의하기
    data class Point(val x: Int, val y: Int)
    
    operator fun Point.plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
    ```
    

- 연산자를 오버로딩하는 함수 앞에는 꼭 `operator`가 있어야 한다.
    - `operator`가 없는데 관례에서 사용하는 함수 이름을 쓴다면 `operator modifier is required...` 오류가 발생한다.
- `operator` 변경자를 추가해 plus 함수를 선언하고 나면 + 기호로 두 Point 객체를 더할 수 있다.
    
    ![7.1 `+` 연산자는 `plus` 함수 호출로 컴파일된다.](./image/7/Untitled.png)
    
    7.1 `+` 연산자는 `plus` 함수 호출로 컴파일된다.
    
- 외부 함수의 클래스에 대한 연산자를 정의할 때는 관례를 따르는 이름의 확장 함수로 구현하는 게 일반적인 패턴이다.
    
    ```kotlin
    operator fun Point.plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
    ```
    

- 오버로딩 가능한 이항 산술 연산자
    
    
    | 식 | 함수 이름 |
    | --- | --- |
    | a * b | times |
    | a / b | div |
    | a % b | rem |
    | a + b | plus |
    | a - b | minus |
- 연산자 우선순위는 표준 숫자 타입에 대한 연산자 우선순위와 같다.
    - 자바를 코틀린에서 호출하는 경우, 함수 이름이 코틀린의 관례에 맞아 떨어지기만 하면 항상 연산자 식을 사용해 그 함수를 호출할 수 있다.
    - 자바에서는 따로 `operator` 표시를 할 수 없으므로 이름과 파라미터 개수만 맞으면 된다.
- 연산자를 정의할 때 두 피연산자가(연산자 함수의 두 파라미터)가 같은 타입일 필요는 없다.
    
    ```kotlin
    operator fun Point.times(scale: Double): Point {
        return Point((x * scale).toInt(), (y * scale).toInt())
    }
    >>> val p = Point(10, 20)
    >>> println(p * 1.5) // Point(x=15, y=30)
    ```
    
- 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치해야만 하는 것도 아니다.
    
    ```kotlin
    operator fun Char.times(count: Int): String {
        return toString().repeat(count)
    }
    >>> println('a' * 3) // aaa
    ```
    
    - 일반 함수와 마찬가지로 연산자 함수도 오버로딩 가능하다. → 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 여럿 만들 수 있다.

### 1-2. 복합 대입 연산자 오버로딩

---

- 복합 대입(compound assignment)연산자: `+=`, `-=` 등의 연산자
    
    ```kotlin
    >>> var point = Point(1, 2)
    >>> point += Point(3, 4)
    >>> println(point) // Point(x=4, y=6)
    ```
    
- 반환 타입이 `Unit`인 `plusAssign` 함수를 정의하면 코틀린은 `+=` 연산자에 그 함수를 사용한다. (`minusAssign`, `timesAssign` 등도 마찬가지)
    
    ```kotlin
    operator fun <T> MutableCollection<T>.plusAssign(element: T) {
        this.add(element)
    }
    ```
    

- 이론적으로는 코드에 있는 `+=`를 `plus`와 `plusAssign` 양쪽으로 컴파일할 수 있다. 어떤 클래스가 이 두 함수를 모두 정의하고 둘 다 `+=`에 사용 가능한 경우 컴파일러는 오류를 보고한다.****
    
    ![7.2 `+=` 연산자는 `plus` 또는 `plusAssign` 함수 호출로 번역할 수 있다.](./image/7/Untitled%201.png)
    
    7.2 `+=` 연산자는 `plus` 또는 `plusAssign` 함수 호출로 번역할 수 있다.
    

- 코틀린 표준 라이브러리는 컬렉션에 대해 두 가지 접근 방법
    - `+` 와 `-` 는 항상 새로운 컬렉션을 반환하며, `+=` 와 `-=` 연산자는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화시킨다.
    - 읽기 전용 컬렉션에서 `+=` 와 `-=` 는 변경을 적용한 복사본을 반환한다.

### 1-3. 단항 연산자 오버로딩

---

- 단항 연산자 오버로딩하는 절차도 이항 연산자와 마찬가지다.
    
    ```kotlin
    data class Point(val x: Int, val y: Int)
    
    operator fun Point.unaryMinus(): Point {
        return Point(-x, -y)
    }
    
    fun main(args: Array<String>) {
        val p = Point(10, 20)
        println(-p)
    }
    ```
    
    ![7.3 단항 `+` 연산자는 `unaryPlus` 호출로 변환된다.](./image/7/Untitled%202.png)
    
    7.3 단항 `+` 연산자는 `unaryPlus` 호출로 변환된다.
    

- 오버로딩할 수 있는 단항 산술 연산자
    
    
    | 식 | 함수 이름 |
    | --- | --- |
    | +a | unaryPlus |
    | -a | unaryMinus |
    | !a | not |
    | ++a, a++ | inc |
    | --a, a-- | dec |
- `inc`, `dec` 함수를 정의해 증가/감소 연산자를 오버로딩하는 경우 컴파일러는 일반적인 값에 대한 전위와 후위 증가/감소 연산자와 같은 의미를 제공한다.
    
    ```kotlin
    operator fun BigDecimal.inc() = this + BigDecimal.ONE
    
    fun main() {
        var bd = BigDecimal.ZERO
        bd = bd.inc()
        println(bd) // 1
        println(++bd) // 2
        bd.dec()
        println(bd) // 2
    }
    ```
    

## 2. 비교 연산자 오버로딩

---

### 2-1. 동등성 연산자: `equals`

---

- 코틀린이 `==` 연산자 호출을 `equals` 메소드 호출로 컴파일한다. `!=` 연산자를 사용하는 식도 `equals` 호출로 컴파일된다.
- `==`와 `!=`는 내부에서 인자가 `null`인지 검사하므로 다른 연산과 달리 널이 될 수 있는 값에도 적용할 수 있다.
    
    ![7.4 동등성 검사 `==`는 `equals` 호출과 널 검사로 컴파일된다.](./image/7/Untitled%203.png)
    
    7.4 동등성 검사 `==`는 `equals` 호출과 널 검사로 컴파일된다.
    
- `data class`는 컴파일러가 자동으로 `equals`를 생성해준다.
    - 직접 `equals`를 구현한다면,
        
        ```kotlin
        class Point(val x: Int, val y: Int) {
            override fun equals(obj: Any?): Boolean {
                if (obj === this) return true
                if (obj !is Point) return false
                return obj.x == x && obj.y == y
            }
        }
        
        fun main(args: Array<String>) {
            println(Point(10, 20) == Point(10, 20)) // true
            println(Point(10, 20) != Point(5, 5)) // true
            println(null == Point(1, 2)) // false
        }
        ```
        
- 식별자 비교 연산자(`===`)를 사용해 `equals`의 파라미터가 수신객체와 같은지 살펴본다 👉 `===`는 오버로딩할 수 없다.
    - 자바의 `==` 연산자와 같다.
    - 서로 같은 객체를 가리키는지(원시 타입인 경우 두 값이 같은지) 비교한다.

- `equals`는 다른 연산자 오버로딩 관례와 달리 `operator`가 아닌 `override`가 붙어 있다.
    - `equals`는 `Any`에 정의된 메서드이므로 `override`가 필요하다.
    - `Any`의 `equals`에는 `operator`가 붙어있지만 그 메서드를 오버라이드하는 (하위 클래스) 메서드 앞에는 `operator` 변경자를 붙이지 않아도 자동으로 상위 클래스의 `operator` 지정이 적용된다.
        
        ![Untitled](./image/7/Untitled%204.png)
        
- `Any`에서 상속받은 `equals`가 확장 함수보다 우선순위가 높기 때문에 `equals`를 확장 함수로 정의할 수는 없다.

### 2-2. 순서 연산자: `compareTo`

---

- 자바에서 정렬이나 최댓값, 최솟값 등 값을 비교해야 하는 알고리즘에 사용할 클래스는 `Comparable` 인터페이스를 구현해야 한다. `Comparable`에 들어있는 `compareTo` 메소드는 한 객체와 다른 객체의 크기를 비교해 정수로 나타내준다. 하지만 자바에는 이 메소드를 짧게 호출할 수 있는 방법이 없다.
- 코틀린도 똑같은 `Comparable` 인터페이스를 지원한다.
    - 코틀린은 `Comparable` 인터페이스 안에 있는 `compareTo` 메소드를 호출하는 관례를 제공한다.
        - 비교 연산자(`<`, `>`, `<=`, `>=`)는 `compareTo` 호출로 컴파일된다.
        
        ![7.5 두 객체를 비교하는 식은 `compareTo`의 결과를 0과 비교하는 코드로 컴파일된다.](./image/7/Untitled%205.png)
        
        7.5 두 객체를 비교하는 식은 `compareTo`의 결과를 0과 비교하는 코드로 컴파일된다.
        
        ```kotlin
        class Person(val firstName: String, val lastName: String) : Comparable<Person> {
            override fun compareTo(other: Person): Int {
                return compareValuesBy(this, other,
                    Person::lastName, Person::firstName)
            }
        }
        ```
        
        - 코틀린 표준 라이브러리의 `compareValuesBy` 함수를 사용해 `compareTo`를 쉽고 간결하게 정의할 수 있다.
            - `compareValuesBy`는 두 객체와 여러 비교 함수를 인자로 받는다.
            - 첫 번째 비교 함수에 두 객체를 넘겨 같지 않으면 그 결과를 즉시 반환하고, 같으면(0) 두 번째 비교 함수로 비교한다. 모든 함수가 0을 반환하면 0을 반환한다.
            - 각 비교 함수는 람다, 프로퍼티, 메서드 참조일 수 있다.
    - 필드를 직접 비교하면 코드는 조금 더 복잡해지지만 비교 속도는 훨씬 더 빨라진다.
    - `Comparable` 인터페이스를 구현하는 모든 자바 클래스를 코틀린에서는 간결한 연산자 구문으로 비교할 수 있다.
        
        ```kotlin
        fun main(args: Array<String>) {
            println("abc" < "bac")
        }
        ```
        

## 3. 컬렉션과 범위에 대해 쓸 수 있는 관례

---

### 3-1. 인덱스로 원소에 접근: `get`과 `set`

---

- 인덱스 연산자를 사용해 원소를 읽는 연산은 `get` 연산자 메서드로 변환되고, 원소를 쓰는 연산은 `set` 연산자 메서드로 변환된다.
    
    ```kotlin
    operator fun Point.get(index: Int): Int {
        return when (index) {
            0 -> x
            1 -> y
            else -> throw kotlin.IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    
    data class MutablePoint(var x: Int, var y: Int)
    operator fun MutablePoint.set(index: Int, value: Int) {
        when (index) {
            0 -> x = value
            1 -> y = value
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    ```
    
    ![7.6 각괄호를 사용한 접근은 `get` 함수 호출로 변환된다.](./image/7/Untitled%206.png)
    
    7.6 각괄호를 사용한 접근은 `get` 함수 호출로 변환된다.
    
    - `get` 메서드의 파라미터로 `Int`가 아닌 타입도 사용할 수 있다.
    
    ![7.7 각괄호를 사용한 대입문은 `set` 함수 호출로 컴파일된다.](./image/7/Untitled%207.png)
    
    7.7 각괄호를 사용한 대입문은 `set` 함수 호출로 컴파일된다.
    

### 3-2. `in` 관례

---

- `in`: 객체가 컬렉션에 들어있는지 검사한다. `in` 연산자와 대응하는 함수는 `contains`다.
    
    ```kotlin
    data class Rectangle(val upperLeft: Point, val lowerRight: Point)
    
    operator fun Rectangle.contains(p: Point): Boolean {
        return p.x in upperLeft.x until lowerRight.x &&
               p.y in upperLeft.y until lowerRight.y
    }
    
    fun main(args: Array<String>) {
        val rect = Rectangle(Point(10, 20), Point(50, 50))
        println(Point(20, 30) in rect) // true
        println(Point(5, 5) in rect) // false
    }
    ```
    
    ![7.8 `in` 연산자는 `contains` 함수 호출로 변환된다.](./image/7/Untitled%208.png)
    
    7.8 `in` 연산자는 `contains` 함수 호출로 변환된다.
    

### 3-3. `rangeTo` 관례

---

- `..` 연산자는 `rangeTo` 함수를 간략하게 표현하는 방법이다.
    
    ![7.9 `..` 연산자는 `rangeTo` 함수 호출로 컴파일된다.](./image/7/Untitled%209.png)
    
    7.9 `..` 연산자는 `rangeTo` 함수 호출로 컴파일된다.
    
    ```kotlin
    fun main(args: Array<String>) {
        val range = 0..8 + 1
        println(range) // 0..9
        range.forEach { print(it) } // 0123456789
    }
    ```
    
- 코틀린 표준 라이브러리에는 모든 `Comparable` 객체에 대해 적용 가능한 `rangeTo` 함수가 들어있다.
    
    ![Untitled](./image/7/Untitled%2010.png)
    
    ```kotlin
    operator fun <T: Comparable<T>> T.rangeTo(that:T): ClosedRange<T>
    ```
    

### 3-4. `for` loop를 위한 `iterator` 관례

---

- `for (x in list) {..}`와 같은 문장은 `list.iterator()`를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 `hasNext`와 `next` 호출을 반복하는 식으로 변환된다.
- 코틀린에서는 `iterator` 메소드를 확장 함수로 정의할 수 있다. 이런 성질로 인해 일반 자바 문자열에 대한 `for` 루프가 가능하다.
    
    ```kotlin
    // iterator 메소드를 확장 함수로 정의
    operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        // 이 객체는 LocalDate 원소에 대한 iterator 를 구현한다.
        object : Iterator<LocalDate> {
            var current = start
            override fun hasNext() =
                // compareTo 관례를 사용해 날짜를 비교한다.
                current <= endInclusive
    
            // 현재 날짜를 저장한 다음에 날짜를 변경한다. 그 후 저장해둔 날짜를 반환한다.
            override fun next() = current.apply {
                current = plusDays(1)
            }
        }
    
    fun main(args: Array<String>) {
        val newYear = LocalDate.ofYearDay(2022, 1)
        val daysOff = newYear.minusDays(1)..newYear
        for (dayOff in daysOff) { println(dayOff) }
    		// 2021-12-31
    		// 2022-01-01
    }
    ```
    

## 4. 구조 분해 선언과 component 함수

---

- 구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.
    
    ```kotlin
    val p = Point(10, 20)
    // = 의 좌변에 여러 변수를 괄호로 묶었다.
    val (x, y) = p // x와 y를 한 번에 초기화한다. x = 10, y = 20
    ```
    
    ![7.10 구조 분해 선언은 `componentN` 함수 호출로 변환된다.](./image/7/Untitled%2011.png)
    
    7.10 구조 분해 선언은 `componentN` 함수 호출로 변환된다.
    
- `data` 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 `componentN` 함수를 만들어준다.
    
    ```kotlin
    // 데이터 타입이 아닌 클래스에서 이런 함수를 어떻게 구현하는지 보여준다.
    class Point(val x: Int, val y: Int) {
        operator fun component1() = x
        operator fun component2() = y
    }
    ```
    

- 구조 분해 선언은 함수에서 여러 값을 반환할 때 유용하다.
    
    ```kotlin
    data class NameComponents(val name: String, val extension: String)
    
    fun splitFilename(fullName: String): NameComponents {
        val result = fullName.split('.', limit = 2)
        // 함수에서 데이터 클래스의 인스턴스를 반환한다.
        return NameComponents(result[0], result[1])
    }
    
    fun main(args: Array<String>) {
        // 구조 분해 선언 구문을 사용해 데이터 클래스를 푼다. 
        val (name, ext) = splitFilename("example.kt")
        println(name)
        println(ext)
    }
    ```
    
- ❗배열이나 컬렉션에도 `componentN` 함수가 있음
    
    ```kotlin
    fun splitFilename(fullName: String): NameComponents {
        val **(name, extension)** = fullName.split('.', limit = 2)
        // 함수에서 데이터 클래스의 인스턴스를 반환한다.
        return NameComponents(name, extension)
    }
    ```
    

- 코틀린 표준 라이브러리에서는 맨 앞의 다섯 원소에 대한 `componentN`을 제공한다.
    - 표준 라이브러리의 `Pair`나 `Triple` 클래스를 사용하면 함수에서 여러 값을 더 간단하게 반환할 수 있다.
        
        ![Untitled](./image/7/Untitled%2012.png)
        

### 4-1. 구조 분해 선언과 루프

---

- 함수 본문 내의 선언문뿐 아니라변수 선언이 들어갈 수 있는 장소라면 어디든 구조 분해 선언을 사용할 수 있다.
    
    ```kotlin
    fun printEntries(map: Map<String, String>) {
        for ((key, value) in map) { //루프 변수에 구조 분해 선언
            println("$key -> $value")
        }
    }
    
    fun main() {
        // 맵 원소를 만들 때도 to 를 사용하면 유용하다.
        val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
        printEntries(map) // Oracle -> Java
                          // JetBrains -> Kotlin
    
        for (entry in map.entries) {
            println("${entry.component1()} -> ${entry.component2()}")
        } // Oracle -> Java
          // JetBrains -> Kotlin
    }
    ```
    
- 코틀린 라이브러리는 `Map.Entry`에 대한 확장 함수로 `component1()`, `component2()`를 제공한다.

## 5. 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

---

- 위임 프로퍼티(delegated property)를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있다. 또한 그 과정에서 접근자 로직을 매번 재구현할 필요도 없다.
- 위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말한다. 이때 작업을 처리하는 도우미 객체를 위임 객체라고 부른다.****

### 5-1. 위임 프로퍼티 소개

---

- 위임 프로퍼티의 일반적인 문법
    
    ```kotlin
    class Foo {
        var p: Type by Delegate() // p 프로퍼티는 접근자 로직을 다른 객체에게 위임한다.
                                  // Delegate 인스턴스를 위임 객체로 사용한다.
    }
    ```
    
- `by` 뒤에 있는 식을 계산해서 위임에 쓰일 객체를 얻는다.
- 컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화한다.
    
    ```kotlin
    class Foo {
        private val delegate = Delegate() // 컴파일러가 생성한 도우미 프로퍼티
        var p: Type
           set(value: Type) = delegate.setValue(..., value)
           get() = delegate.getValue(...)
    }
    
    class Delegate {
        operator fun getValue(...) { ... }
        operator fun setValue(..., value: Type) { ... }
    }
    ```
    
    - 프로퍼티 위임 관례에 따르는 Delegate 클래스는 getValue 와 setValue(변경 가능한 프로퍼티만) 메서드를 제공해야 한다.

### 5-2. 위임 프로퍼티 사용: `by lazy()`를 사용한 프로퍼티 초기화 지연

---

- 지연 초기화(lazy initialization)는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요할 경우 초기화할 때 흔히 쓰이는 패턴이다.
- 초기화 과정에 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.
    
    
- 뒷받침하는 프로퍼티 기법을 사용한 코드: 만들기 성가시고 스레드 안전하지 않다.
    
    ```kotlin
    class Person(val name: String) {
        private var _emails: List<Email>? = null // 데이터를 저장하고 emails의 위임 객체 역활을 하는 _emails 프로퍼티
    
        val emails: List<Email>
            get() {
              if (_emails == null) {
                  _emails = loadEmails(this) // 최초 접근 시 이메일을 가져온다. 
              }
              return _emails!! // 저장해 둔 데이터가 있으면 그 데이터를 반환한다. 
            }
    }
    
    fun main(args: Array<String>) {
        val p = Person("Alice")
        p.emails // 최초로 emails를 읽을 때 단 한번만 이메일을 가져온다. 
        p.emails
    }
    ```
    

👉

- 위임 프로퍼티: 데이터를 저장할 때 쓰이는 뒷받침하는 프로퍼티와 같이 오직 한 번만 초기화됨을 보장하는 게터 로직을 함께 캡슐화해준다.
    
    ```kotlin
    class Person(val name: String) {
        val emails by lazy { loadEmails(this) }
    }
    ```
    
    - `lazy`를 `by` 키워드와 함께 사용해 위임 프로퍼티를 만들 수 있다.
    - `lazy` 함수의 인자는 값을 초기화할 때 호출할 람다다.
    - `lazy` 함수는 기본적으로 스레드 안전하다.
    - 필요에 따라 동기화에 사용할 락을 `lazy` 함수에 전달할 수도 있고, 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 `lazy` 함수가 동기화를 하지 못하게 막을 수도 있다.

### 5-3. 위임 프로퍼티 구현

---

- 통지 기능 예제: 자바 빈 클래스의 필드에 인스턴스를 저장하고 프로퍼티 변경 시 그 인스턴스에게 처리를 위임하는 방식으로 구현한다. 프로퍼티 변경 리스너를 추적해주는 작은 도우미 클래스가 있다.
- 위임 객체: `by` 오른쪽에 오는 객체
    - 코틀린은 위임 객체를 감춰진 프로퍼티에 저장하고, 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 `getValue`와 `setValue`를 호출해준다.

- ex1> 위임 프로퍼티를 통해 프로퍼티 변경 통지 받기
    
    ```kotlin
    open class PropertyChangeAware {
        protected val changeSupport = PropertyChangeSupport(this)
    
        fun addPropertyChangeListener(listener: PropertyChangeListener) {
            changeSupport.addPropertyChangeListener(listener)
        }
    
        fun removePropertyChangeListener(listener: PropertyChangeListener) {
            changeSupport.removePropertyChangeListener(listener)
        }
    }
    
    class ObservableProperty2(var propValue: Int, val changeSupport: PropertyChangeSupport) {
        // getValue, setValue 함수에도 operator 변경자가 붙는다.
        // KProperty 타입의 객체를 사용해 프로퍼티를 표현한다. 그래서 name 프로퍼티를 없앨 수 있다.
        operator fun getValue(p: PersonSalary3, prop: KProperty<*>): Int = propValue
    
        operator fun setValue(p: PersonSalary3, prop: KProperty<*>, newValue: Int) {
            val oldValue = propValue
            propValue = newValue
            changeSupport.firePropertyChange(prop.name, oldValue, newValue)
        }
    }
    
    class PersonSalary3(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
        // 위임 객체를 지정하면 직접 코드를 짜야 했던 여러 작업을 by 키워드를 사용해 코틀린 컴파일러가 자동으로 처리.
        var age: Int by ObservableProperty2(age, changeSupport)
        var salary: Int by ObservableProperty2(salary, changeSupport)
    }
    ```
    
- ex2> 관찰 가능한 프로퍼티 로직을 직접 작성하는 대신 코틀린 표준 라이브러리를 사용한 버전
    
    ```kotlin
    class PersonSalary4(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
        private val observer = { prop: KProperty<*>, oldValue: Int, newValue: Int ->
            changeSupport.firePropertyChange(prop.name, oldValue, newValue)
        }
    
        var age: Int by Delegates.observable(age, observer)
        var salary: Int by Delegates.observable(salary, observer)
    }
    ```
    
    - `by` 오른쪽 식이 꼭 새 인스턴스를 만들 필요는 없다. 함수 호출, 다른 프로퍼티, 다른 식 등이 `by`의 우항에 올 수 있다.
    - 다만 우항을 계산한 결과인 객체는 컴파일러가 호출할 수 있는 올바른 타입의 `getValue`, `setValue` 를 반드시 제공해야 한다.
        - 객체 안에 정의된 메소드이거나 확장 함수일 수 있다.

### 5-4. 위임 프로퍼티 컴파일 규칙

---

```kotlin
class C {
    var prop: Type by MyDelegate()
}
val c = C()
```

- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 `<delegate>`라는 이름으로 부른다.
- 컴파일러는 프로퍼티를 표현하기 위해 `KProperty` 타입의 객체를 사용한다. 이 객체를 `<property>`라고 부른다.
- 컴파일러가 생성하는 코드
    
    ```kotlin
    class C {
        private val <delegate> = MyDelegate()
        var prop: Type
          get() = <delegate>.getValue(this, <property>)
          set(value: Type) = <delegate>.setValue(this, <property>, value)
    }
    ```
    
    - 컴파일러는 모든 프로퍼티 접근자 안에 `getValue`와 `setValue` 호출 코드를 생성해준다.
        
        ![7.11 프로퍼티를 사용하면 `<delegate>`에 있는 `getValue`나 `setValue` 함수가 호출된다.](./image/7/Untitled%2013.png)
        
        7.11 프로퍼티를 사용하면 `<delegate>`에 있는 `getValue`나 `setValue` 함수가 호출된다.
        

- 참고
    - [https://essie-cho.com/kotlin-in-action-07/](https://essie-cho.com/kotlin-in-action-07/)
    - [https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/7](https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/7)
