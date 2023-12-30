# Chapter 9. 제네릭스

## 1. 제네릭 타입 파라미터

---

- 코틀린 컴파일러는 보통 타입과 마찬가지로 타입 인자도 추론할 수 있다.
    
    ```kotlin
    val authors = listOf("Dmitry", "Svetlana")
    ```
    
- 빈 리스트를 만들어야 한다면 타입 인자를 추론할 근거가 없기 때문에 직접 타입 인자를 명시해야 한다.
    - 변수의 타입을 지정
        
        ```kotlin
        val readers: MutableList<String> = mutableListOf()
        ```
        
    - 변수를 만드는 함수의 타입 인자를 지정
        
        ```kotlin
        val readers = mutableListOf<String>()
        ```
        
- 자바와 달리 코틀린에서는 제네릭 타입의 타입 인자를 프로그래머가 명시하거나 컴파일러가 추론할 수 있어야 한다.
    - 자바는 맨 처음에 제네릭 지원이 없었고 자바 1.5에 뒤늦게 제네릭을 도입했기 때문에 이전 버전과 호환성을 유지하기 위해 타입 인자가 없는 제네릭 타입(raw)을 허용한다.

### 1-1. 제네릭 함수와 프로퍼티

---

- 리스트를 다루는 함수를 작성한다면 어떤 특정 타입을 저장하는 리스트뿐 아니라 모든 리스트를 다룰 수 있는 함수를 원할 것이다. 이럴때 제네릭 함수를 작성해야 한다.
    
    ![9.1 제네릭 함수인 `slice`는 T 를 타입 파라미터로 받는다.](./image/9/Untitled.png)
    
    9.1 제네릭 함수인 `slice`는 T 를 타입 파라미터로 받는다.
    
- 이런 함수를 구체적인 리스트에 대해 호출할 때 타입 인자를 명시적으로 지정할 수 있다. 하지만 실제로는 대부분 컴파일러가 타입 인자를 추론할 수 있으므로 그럴 필요가 없다.
    
    ```kotlin
    val letters = ('a'..'z').toList()
    
    println(letters.slice<Char>(0..2)) // 타입 인자를 명시적으로 지정한다.
    println(letters.slice(0..2)) // 컴파일러는 여기서 T가 Char라는 사실을 추론한다.
    ```
    

### 1-2. 제네릭 클래스 선언

---

- 자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호(<>)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다. 타입 파라미터를 이름 뒤에 붙이고 나면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다.
    
    ```kotlin
    interface List<T> { // List 인터페이스에 T라는 타입 파라미터를 정의한다.
        operator fun get(index: Int): T // 인터페이스 안에서 T를 일반 타입처럼 사용할 수 있다.
    }
    ```
    
- 제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다.
    - 구체적인 타입을 넘길 수도 있고 (하위 클래스도 제네릭 클래스라면) 타입 파라미터로 받은 타입을 넘길수도 있다.
    
    ```kotlin
    class StringList: List<String> { // 이 클래스는 구체적인 타입 인자로 String 을 지정해 List 를 구현한다.
        override fun get(index: Int): String = ...
    }
    class ArrayList<T>: List<T> { // ArrayList 의 제네릭 타입 파라미터 T 를 List 의 타입 인자로 넘긴다.
        override fun get(index: Int): T = ...
    }
    ```
    
- 클래스가 자기 자신을 타입 인자로 참조할 수도 있다. ex> 구현하는 클래스
    - 비교 가능한 모든 값은 자신을 같은 타입의 다른 값과 비교하는 방법을 제공해야만 한다.
    
    ```kotlin
    interface Comparable<T> {
        fun compareTo(other: T): Int
    }
    class String: Comparable<String> {
        override fun compareTo(other: String): Int = /*...*/
    }
    ```
    

### 1-3. 타입 파라미터 제약

---

- 타입 파라미터 제약: 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능
- 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한(upper bound)으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입이어야 한다.
    - 제약을 가하려면 타입 파라미터 이름 뒤에 콜론(`:`)을 표시하고 그 뒤에 상한 타입을 적으면 된다.
    
    ![9.2 타입 파라미터 뒤에 상한을 지정함으로써 제약을 정의할 수 있다.](./image/9/Untitled%201.png)
    
    9.2 타입 파라미터 뒤에 상한을 지정함으로써 제약을 정의할 수 있다.
    
- 타입 파라미터 T 에 대한 상한을 정하고 나면 T 타입의 값을 그 상한 타입의 값으로 취급할 수 있다.
    
    ```kotlin
    fun <T : Number> oneHalf(value: T): Double { // Number를 타입 파라미터 상한으로 정한다. 
        return value.toDouble() / 2.0 // Number 클래스에 정의된 메소드를 호출한다. 
    }
    >>> println(oneHalf(3)) // 1.5
    ```
    
- 타입 파라미터에 여러 제약을 가하기
    
    ```kotlin
    fun <T> ensureTrailingPeriod(seq: T) where T : CharSequence, T : Appendable { // 타입 파라미터 제약 목록
        if (!seq.endsWith('.')) { // CharSequence 인터페이스의 확장 함수를 호출
            seq.append('.') // Appendable 인터페이스의 메소드를 호출
        }
    }
    
    fun main() {
        val helloWorld = StringBuilder("Hello World")
        ensureTrailingPeriod(helloWorld)
        println(helloWorld) // Hello World.
    }
    ```
    
    → 타입 인자가 `CharSequence`와 `Appendable` 인터페이스를 반드시 구현해야 한다.
    

### 1-4. 타입 파라미터를 널이 될 수 없는 타입으로 한정

---

- 아무런 상한을 정하지 않은 타입 파라미터는 결과적으로 `Any?`를 상한으로 정한 파라미터와 같다.
    
    ```kotlin
    class Processor<T> {
        fun process(value: T) {
            value?.hashCode() // "value"는 널이 될 수 있다. 따라서 안전한 호출을 사용해야 한다.
        }
    }
    ```
    
- `<T : Any>`라는 제약은 T 타입이 항상 널이 될 수 없는 타입이 되도록 보장한다.
    
    ```kotlin
    class Processor<T : Any> {
        fun process(value: T) {
            value.hashCode()
        }
    }
    ```
    
- 타입 파라미터를 널이 될 수 없는 타입으로 제약하기만 하면 타입 인자로 널이 될 수 있는 타입이 들어오는 일을 막을 수 있다.****

## 2. 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

---

- JVM 의 제네릭스는 보통 타입 소거를 사용해 구현된다 → 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다.
    
    ![Untitled](./image/9/Untitled%202.png)
    

### 2-1. 실행 시점의 제네릭: 타입 검사와 캐스트

---

- 자바와 마찬가지로 코틀린 제네릭 타입 인자 정보는 런타임에 지워진다 → 제네릭 클래스 인스턴스가 그 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다.
    
    ```kotlin
    val list1: List<String> = listOf("a", "b")
    val list2: List<Int> = listOf(1, 2, 3)
    ```
    
    ![9.3 실행 시점에 list1 이나 list2 가 정수의 리스트로 선언됐다는 사실을 알 수 없다. 각 객체는 단지 `List` 일 뿐이다.](./image/9/Untitled%203.png)
    
    9.3 실행 시점에 list1 이나 list2 가 정수의 리스트로 선언됐다는 사실을 알 수 없다. 각 객체는 단지 `List` 일 뿐이다.
    
    - 컴파일러는 두 리스트를 서로 다른 타입으로 인식하지만 실행 시점에 그 둘은 완전히 같은 타입의 객체다.
- 타입 소거로 인해 생기는 한계 → `is` 검사에서 타입 인자로 지정한 타입을 검사할 수는 없다.
    
    ```kotlin
    if (value is List<String>) {...}
    // ERROR: Cannot check for instance of erased type
    ```
    
- 제네릭 타입 소거 장점: 저장해야 하는 타입 정보의 크기가 줄어들어서 전반적인 메모리 사용량이 줄어든다.
- 스타 프로젝션(`*`): 타입 파라미터가 2개 이상이라면 모든 타입 파라미터에 `*`를 포함시켜야 한다 → 인자를 알 수 없는 제네릭 타입을 표현할 때 사용

- 실행 시점에는 제네릭 타입의 타입 인자를 알 수 없으므로 캐스팅은 항상 성공한다. 그런 타입 캐스팅을 사용하면 컴파일러가 "`unchecked cast`(검사할 수 없는 캐스팅)"이라는 경고를 해준다. 하지만 컴파일러는 단순히 경고만 하고 컴파일을 진행한다.
    
    ```kotlin
    fun printSum(c: Collection<*>) {
        val intList = c as? List<Int> ?: throw IllegalArgumentException("List is expected")
        println(intList.sum())
    }
    
    fun main() {
        printSum(listOf(1, 2, 3)) // 6
    }
    ```
    
    ![Untitled](./image/9/Untitled%204.png)
    
- 하지만 잘못된 타입의 원소가 들어있는 리스트를 전달하면 실행 시점에 `ClassCastException`이 발생한다.
    
    ```kotlin
    fun main() {
        printSum(listOf("a", "b"))
        // Exception in thread "main" java.lang.ClassCastException: java.lang.String incompatible with java.lang.Number
    }
    ```
    
    - 어떤 값이 `List<Int>`인지 검사할 수는 없으므로 `IllegalArgumentException`이 발생하지는 않는다 → `as?` 캐스트가 성공하고 문자열 리스트에 대해 `sum` 함수가 호출된다. `sum`이 실행되는 도중에 예외가 발생한다.
- 코틀린 컴파일러는 컴파일 시점에 타입 정보가 주어진 경우에는 `is` 검사를 수행하게 허용할 수 있을 정도로 똑똑하다.
    
    ```kotlin
    fun printSum(c: Collection<Int>) {
        if (c is List<Int>) {
            println(c.sum())
        }
    }
    
    fun main() {
        printSum(listOf(1, 2, 3)) // 6
    }
    ```
    
- 일반적으로 코틀린 컴파일러는 우리에게 안전하지 못한 검사와 수행할 수 있는 검사를 알려주기 위해 최대한 노력한다. 따라서 컴파일러 경고의 의미와 어떤 연산이 안전한지에 대해 알아야 한다.****
    - 안전하지 못한 `is` 검사는 금지
    - 위험한 `as` 캐스팅은 경고를 출력

### 2-2. 실체화한 타입 파라미터를 사용한 함수 선언

---

- 제네릭 함수의 타입 인자도 제네릭 함수가 호출되도 그 함수의 본문에서는 호출시 쓰인 타입 인자를 알 수 없다.
    
    ```kotlin
    fun <T> isA(value: Any) = value is T
    // Error: Cannot check for instance of erased type: T
    ```
    
- 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.
    
    ```kotlin
    inline fun <refied T> isA(value: Any) = value is T
    
    fun main() {
        println(isA<String>("abc")) // true
        println(isA<String>(123)) // false
    }
    ```
    
    ![Untitled](./image/9/Untitled%205.png)
    
- `filterIsInstance`: 인자로 받은 컬렉션의 원소 중에서 타입 인자로 지정한 클래스의 인스턴스만을 모아서 만든 리스트를 반환한다.
    
    ![Untitled](./image/9/Untitled%206.png)
    
    ![Untitled](./image/9/Untitled%207.png)
    
    ```kotlin
    fun main() {
        val items = listOf("one", 2, "three")
        println(items.filterIsInstance<String>()) // [one, three]
    }
    ```
    
- 코틀린에서, 일반적인 경우 런타임에 제네릭 파라미터를 확인할 방법은 없다. [[stackoverflow 동일 이슈](https://stackoverflow.com/questions/36569421/kotlin-how-to-work-with-list-casts-unchecked-cast-kotlin-collections-listkot#answer-36570969)]
    
    ```kotlin
    @Suppress("UNCHECKED_CAST")
    inline fun <reified T : Any> List<*>.checkItemsAre() =
        if (all { it is T })
            this as List<T>
        else null
    ```
    
    → 타입 인자를 실행 시점에 알 수 있다.
    

- 인라인 함수에서만 실체화한 타입 인자를 쓸 수 있는 이유
    - 컴파일러는 인라인 함수의 본문을 구현한 바이트코드를 그 함수가 호출되는 모든 지점에 삽입한다.
    - 컴파일러는 실체화한 타입 인자를 사용해 인라인 함수를 호출하는 각 부분의 정확한 타입 인자를 알 수 있다.
        
        ```kotlin
        inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
            val destination = mutableListOf<T>()
            for (element in this) {
                // 각 원소가 타입 인자로 지정한 클래스의 인스턴스인지 검사할 수 있다.
                if (element is T) { // 인라인 시 바이트코드 생성: if (element is **String**)
                    destination.add(element)
                }
            }
            return destination
        }
        ```
        

<aside>
💡 이 경우에는 함수를 `inline`으로 만드는 이유가 성능 향상이 아니라 실체화한 타입 파라미터를 사용하기 위함이다.

</aside>

### 2-3. 실체화한 타입 파라미터로 클래스 참조 대신

---

- `java.lang.Class` 타입 인자를 파라미터로 받는 API 에 대한 코틀린 어댑터를 구축하는 경우 실체화한 타입 파라미터를 자주 사용한다.
    
    ```kotlin
    val serviceImpl = ServiceLoader.load(Service::class.java)
    ```
    
- `::class.java` 구문은 코틀린 클래스에 대응하는 `java.lang.Class` 참조를 얻는 방법을 보여준다. [[참고](https://www.notion.so/java-lang-Class-class-java-vs-javaClass-ccb4d3be07c04c7caecb11dc359501b3?pvs=21)]
    - [부끄러운 삽질기 공개](https://www.notion.so/non-nullable-type-class-71848e3c774247caaf29b58117f0512d?pvs=21)...
- 👉 구체화한 타입 파라미터를 사용해 다시 작성
    
    ```kotlin
    inline fun <reified T> loadService(): ServiceLoader<T> {
        // T::class 로 타입 파라미터의 클래스를 가져온다.
        return ServiceLoader.load(T::class.java)
    }
    
    fun main() {
        val serviceImpl = loadService<Service>()
    }
    ```
    

### 2-4. 실체화한 타입 파라미터의 제약

---

- 다음과 같은 경우에 실체화한 타입 파라미터를 사용할 수 있다.
    - 타입 검사와 캐스팅(`is`, `!is`, `as`, `as?`)
    - 10장에서 설명할 코틀린 리플렉션 API(`::class`)
    - 코틀린 타입에 대응하는 `java.lang.Class`를 얻기(`::class.java`)
    - 다른 함수를 호출할 때 타입 인자로 사용
- 하지만 다음과 같은 일은 할 수 없다.
    - 타입 파라미터 클래스의 인스턴스 생성하기
    - 타입 파라미터 클래스의 동반 객체 메소드 호출하기
    - 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
    - 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 `refied`로 지정하기****
        - 실체화한 타입 파라미터를 인라인 함수에만 사용할 수 있으므로 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 모든 람다와 함께 인라이닝된다.
            - 람다 내부에서 타입 파라미터를 사용하는 방식에 따라서는 람다를 인라이닝할 수 없는 경우가 생기기도 하고
            - 성능 문제로 람다를 인라이닝하고 싶지 않을 수도 있다.
                
                → `noinline` 변경자를 함수 타입 파라미터에 붙여서 인라이닝을 금지할 수 있다.
                

## 3. 변성: 제네릭과 하위 타입

---

- 변성 개념은 `List<String>`와 `List<Any>`와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.****
- 변성을 잘 활용하면 사용에 불편하지 않으면서 타입 안전성을 보장하는 API를 만들 수 있다.

### 3-1. 변성이 있는 이유: 인자를 함수에 넘기기

---

- `List<Any>` 타입의 파라미터를 받는 함수에 `List<String>`을 넘기면 안전할까? `String` 클래스는 `Any`를 확장하므로 `Any` 타입 값을 파라미터로 받는 함수에 `String`값을 넘겨도 절대로 안전하다. 하지만 `Any`와 `String`이 `List` 인터페이스의 타입 인자로 들어가는 경우 그렇게 자신 있게 안전성을 말할 수 없다 → 코틀린 컴파일러는 실제 이런 함수 호출을 금지한다.
    
    ```kotlin
    >>> val strings = mutableListOf("abc", "bac")
    >>> addAnswer(strings) // 이 줄이 컴파일된다면
    >>> println(strings.maxBy { it.length })
    // 실행 시점에 예외가 발생할 것이다.
    // ClassCastException: Integer cannot be cast to String
    ```
    
    ![Untitled](./image/9/Untitled%208.png)
    
    - 이 예제는 `MutableList<Any>`가 필요한 곳에 `MutableList<String>`을 넘기면 안 된다는 사실을 보여준다.****

👉 어떤 함수가 리스트의 원소를 추가하거나 변경한다면 타입 불일치가 생길 수 있어서 `List<Any>`대신 `List<String>`을 넘길 수 없다. 하지만 원소 추가나 변경이 없는 경우에는 `List<String>`을 `List<Any>`대신 넘겨도 안전하다.

### 3-2. 클래스, 타입, 하위 타입

---

- 타입과 클래스의 차이
    - 제네릭 클래스가 아닌 클래스에서는 클래스 이름을 바로 타입으로 쓸 수 있다.
        
        ```kotlin
        var x: String
        var x: String?
        ```
        
        → 모든 코틀린 클래스가 적어도 둘 이상의 타입을 구성할 수 있다.
        
- 제네릭 클래스에서는
    - 올바른 타입을 얻으려면 제네릭 타입의 타입 파라미터를 구체적인 타입 인자로 바꿔줘야 한다.
        
        ```kotlin
        // List 는 타입이 아니다.
        // 하지만 타입 인자를 치환한 다음은 모두 제대로 된 타입이다.
        List<Int>
        List<String?>
        ```
        
        - 각각의 제네릭 클래스는 무수히 많은 타입을 만들어낼 수 있다.

- 하위 타입(subtype): 어떤 타입 `A`의 값이 필요한 모든 장소에 어떤 타입 `B`의 값을 넣어도 아무 문제가 없다면 `B`는 타입 `A`의 하위 타입이다. (`B`가 `A`보다 구체적)
    - `Int`는 `Number`의 하위 타입
    - 모든 타입이 자신의 하위 타입
    
    ![9.4 A 가 필요한 모든 곳에 B 를 사용할 수 있으면 B 는 A 의 하위 타입이다.](./image/9/Untitled%209.png)
    
    9.4 A 가 필요한 모든 곳에 B 를 사용할 수 있으면 B 는 A 의 하위 타입이다.
    
- 상위 타입: 하위 타입의 반대다.
    - A 타입이 B 타입의 하위 타입이라면 B 는 A 의 상위 타입이다.
- 하위 타입은 하위 클래스(subclass)와 근본적으로 같다.
    - `Int` 클래스는 `Number`의 하위 클래스 → `Int`는 `Number`의 하위 타입
- 무공변(invariant): 제네릭 타입을 인스턴스화 할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 타입 관계가 성립하지 않는 제네릭 타입 👉 자바에서는 모든 클래스가 무공변이다.
    - A 와 B 가 서로 다르기만 하면 `MutableList<A>`는 항상 `MutableList<B>`의 하위 타입이 아니다.
- 공변적(covariant): 코틀린의 `List` 인터페이스는 읽기 전용 컬렉션 → `A`가 `B`의 하위 타입이면 `List<A>`는 `List<B>`의 하위 타입이다.

### 3-3. 공변성: 하위 타입 관계를 유지

---

- A 가 B 의 하위 타입일 때 `Producer<A>`가 `Producer<B>`의 하위 타입이면 `Producer`는 공변적이다 → 하위 타입 관계가 유지된다.
    - ex> Cat가 Animal의 하위 타입이기 때문에 `Producer<Cat>`은 `Producer<Animal>`의 하위 타입이다.
- 코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 `out`을 넣어야 한다.
    
    ```kotlin
    interface Producer<out T> {  // 클래스가 T에 대해 공변적이라고 선언한다. 
        fun produce(): T
    }
    ```
    
- 클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도 그 클래스의 인스턴스를 함수 인자나 반환값으로 사용할 수 있다.
    
    ```kotlin
    open class Animal {
        fun feed() { ... }
    }
    class Herd<T : Animal> {  // 이 타입 파라미터를 무공변성으로 지정한다. 
        val size: Int get() = ...
        operator fun get(i: Int): T { ... }
    }
    fun feedAll(animals: Herd<Animal>) {
        for (i in 0 until animals.size) {
            animals[i].feed()
        }
    }
    ```
    
    ```kotlin
    // 사용자 코드가 고양이 무리를 만들어서 관리한다. 
    class Cat : Animal() {   
        fun cleanLitter() { ... }
    }
    fun takeCareOfCats(cats: Herd<Cat>) {
        for (i in 0 until cats.size) {
            cats[i].cleanLitter()
            // feedAll(cats)           
        }
    }
    ```
    
    - feedAll 함수에게 고양이 무리를 넘기면 타입 불일치(type mismatch) 오류를 볼 수 있다. Herd 클래스의 `T` 타입 파라미터에 대해 아무 변성도 지정하지 않았기 때문에 고양이 무리는 동물 무리의 하위 클래스가 아니다.
        
        ![Untitled](./image/9/Untitled%2010.png)
        
    
    👉 Herd 클래스는 `List`와 비슷한 API 를 제공하며 동물을 그 클래스에 추가하거나 무리안의 동물을 다른 동물로 바꿀 수는 없다. 따라서 Herd 를 공변적인 클래스로 만들고 호출 코드를 적절히 바꿀 수 있다.
    
    ```kotlin
    class Herd<out T : Animal> {  
       ...
    }
    fun takeCareOfCats(cats: Herd<Cat>) {
        for (i in 0 until cats.size) {
            cats[i].cleanLitter()
        }
        feedAll(cats)  
    }
    ```
    

- 타입 파라미터를 공변적으로 지정하면 클래스 내부에서 그 파라미터를 사용하는 방법을 제한한다.
    - 타입 안전성을 보장하기 위해 공변적 파라미터는 항상 아웃(`out`) 위치에만 있어야 한다.
        - 클래스가 `T` 타입의 값을 생산할 수는 있지만 `T` 타입의 값을 소비할수는 없다.

- 클래스 멤버를 선언할 때 타입 파라미터를 사용할 수 있는 지점은 모두 인(`in`)과 아웃(`out`)위치로 나뉜다.
    - `T`가 함수의 반환 타입에 쓰인다면 `T`는 아웃 위치에 있다. 그 함수는 `T` 타입의 값을 생산한다.
    - `T`가 함수의 파라미터 타입에 쓰인다면 `T`는 인 위치에 있다. 그런 함수는 `T` 타입의 값을 소비한다.
        
        ![9.6 함수 파라미터 타입은 인 위치, 함수 반환 타입은 아웃 위치에 있다.](./image/9/Untitled%2011.png)
        
        9.6 함수 파라미터 타입은 인 위치, 함수 반환 타입은 아웃 위치에 있다.
        

- 클래스 타입 파라미터 `T` 앞에 `out` 키워드를 붙이면 클래스 안에서 `T`를 사용하는 메소드가 아웃 위치에서만 `T`를 사용하게 허용하고 인 위치에서는 `T`를 사용하지 못하게 막는다.
    
    ```kotlin
    class Herd<out T: Animal> {
        val size: Int get() = ...
        operator fun get(i: Int): T {...} // T를 반환 타입으로 사용한다.
    }
    ```
    
    ![Untitled](./image/9/Untitled%2012.png)
    
    - 컴파일러는 타입 파라미터가 쓰이는 위치를 제한한다.
    - 위치 규칙은 외부에서 볼 수 있는(`public`, `protected`, `internal`) 클래스 API 에만 적용할 수 있다.
        
        → 변성 규칙은 클래스 외부의 사용자가 클래스를 잘못 사용하는 일을 막기위한 것이므로 클래스 내부 구현에는 적용되지 않는다.
        

<aside>
💡 `private` 메소드 파라미터, 생성자 파라미터는 인, 아웃 어느쪽도 아니다.

</aside>

### 3-4. 반공변성: 뒤집힌 하위 타입 관계

---

- 반공변 클래스의 하위 타입 관계는 공변 클래스의 경우와 반대다.
    
    ```kotlin
    interface Comparator<in T> {
        fun compare(e1: T, e2: T): Int {...}    // T를 in 위치에 사용한다.
    }
    ```
    
    - 인터페이스의 메소드는 T 타입의 값을 소비하기만 한다 → `T`가 인(`in`) 위치에서만 쓰인다.
- 타입 B 가 타입 A 의 하위 타입인 경우 `Consumer<A>`가 `Consumer<B>`의 하위 타입인 관계가 성립하면 제네릭 클래스 `Consumer<T>`는 타입 인자 `T`에 대해 반공변이다.
    
    ![9.7 공변성 타입 `Producer<T>`에서는 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지되지만 반공변성 타입 `Consumer<T>`에서는 타입 인자의 하위 타입 관계가 제네릭 타입으로 오면서 뒤집힌다.](./image/9/Untitled%2013.png)
    
    9.7 공변성 타입 `Producer<T>`에서는 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지되지만 반공변성 타입 `Consumer<T>`에서는 타입 인자의 하위 타입 관계가 제네릭 타입으로 오면서 뒤집힌다.
    
    | 공변성(covariance) | 반공변성(contravariance) | 무공변성(invariance) |
    | --- | --- | --- |
    | Producer<out T> | Consumer<in T> | MutableList<T> |
    | 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지된다. | 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다. | 하위 타입 관계가 성립하지 않는다. |
    | Producer<Cat>은 Producer<Animal>의 하위 타입이다. | Consumer<Animal>은 Consumer<Cat>의 하위 타입이다. |  |
    | T를 아웃 위치에서만 사용할 수 있다. | T를 인 위치에서만 사용할 수 있다. | T를 아무 위치에서나 사용할 수 있다. |

### 3-5. 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

---

- 선언 지점 변성(declaration site variance): 클래스를 선언하면서 변성을 지정하면 그 클래스를 사용하는 모든 장소에 변성 지정자가 영향을 끼치므로 편리하다.
- 사용 지점 변성(use-site variance): 자바에서는 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시해야 한다.

- 코틀린도 사용 지점 변성을 지원한다 👉 클래스 안에서 어떤 타입 파라미터가 공변적이거나 반공변적인지 선언할 수 없는 경우에도 특정 타입 파라미터가 나타나는 지점에서 변성을 정할 수 있다.
    
    ```kotlin
    fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
        for (item in source) {
            destination.add(item)
        }
    }
    ```
    
    - 이 경우 두 컬렉션의 원소 타입이 정확하게 일치할 필요가 없다.
        - ex> 문자열이 원소인 컬렉션에서 객체의 컬렉션으로 원소를 복사해도 아무 문제가 없다.
    - 1️⃣ 여러 다른 리스트 타입에 대해 작동하게 만들기
        
        ```kotlin
        // source 의 원소 타입은 destination 원소 타입의 하위 타입이어야 한다.
        fun <T : R, R> copyData2(source: MutableList<T>, destination: MutableList<R>) {
            for (item in source) {
                destination.add(item)
            }
        }
        
        fun main() {
            val ints = mutableListOf(1, 2, 3)
            val anyItems = mutableListOf<Any>()
            // Int 가 Any 의 하위 타입이므로 이 함수를 호출할 수 있다.
            copyData2(ints, anyItems)
            println(anyItems) // [1, 2, 3]
        }
        ```
        
    - 2️⃣ 함수 구현이 아웃(`out`) 위치 / 인(`in`) 위치에 있는 타입 파라미터를 사용하는 메소드만 호출한다면 그런 정보를 바탕으로 함수 정의 시 타입 파라미터에 변성 변경자를 추가할 수 있다. → 타입 프로젝션
        
        ```kotlin
        // out 키워드를 타입을 사용하는 위치 앞에 붙이면 T 타입을 in 위치에 사용하는 메소드를 호출하지 않는다는 뜻이다.
        fun <T> copyData3(source: MutableList<out T>, destination: MutableList<T>) {
            for (item in source) {
                destination.add(item)
            }
        }
        
        // in 키워드를 붙여 그 파라미터를 더 상위 타입으로 대치할 수 있다. source 리스트 원소의 상위 타입을 destination 리스트 원소 타입으로 허용한다.
        fun <T> copyData4(source: MutableList<T>, destination: MutableList<in T>) {
            for (item in source) {
                destination.add(item)
            }
        }
        ```
        
- 타입 프로젝션: source 를 일반적인 `MutableList`가 아니라 `MutableList`를 프로젝션을 한(제약을 가한) 타입으로 만든다.

### 3-6. 스타 프로젝션: 타입 인자 대신 `*` 사용

---

- 스타 프로젝션: 제네릭 타입 인자 정보가 없음을 표현. `List<*>`(원소 타입이 알려지지 않은 리스트)
    - `MutableList<*>`는 `MutableList<Any?>`와 같지 않다.
        - `MutableList<Any?>`는 모든 타입의 원소를 담을 수 있다는 사실을 알 수 있는 리스트다.
        - 반면 `MutableList<*>`는 어떤 정해진 구체적인 타입의 원소만을 담는 리스트지만 그 원소의 타입을 정확히 모른다는 사실을 표현한다.
        
        ```kotlin
        >>> val list: MutableList<Any?> = mutableListOf('a', 1, "qwe")
        >>> val chars = mutableListOf('a', 'b', 'c')
        >>> val unknownElements: MutableList<*> =                
        ...         if (Random().nextBoolean()) list else chars
        >>> unknownElements.add(42) // 컴파일러는 이 메소드 호출을 금지한다.                              
        // Error: Out-projected type 'MutableList<*>' prohibits the use of 'fun add(element: E): Boolean'
        >>> println(unknownElements.first()) // 원소를 가져와도 안전하다. first()는 Any? 타입의 원소를 반환한다. 
        a
        ```
        
- 타입 파라미터를 시그니처에서 전혀 언급하지 않거나 데이터를 읽기는 하지만 그 타입에는 관심이 없는 경우와 같이 타입 인자 정보가 중요하지 않을 때도 스타 프로젝션 구문을 사용할 수 있다.
    
    ```kotlin
    fun printFirst(list: List<*>) {  // 모든 리스트를 인자로 받을 수 있다. 
        if (list.isNotEmpty()) { // isNotEmpty()에서는 제네릭 타입 파라미터를 사용하지 않는다. 
            println(list.first()) // first()는 이제 Any?를 반환하지만 여기서는 그 타입만으로 충분하다. 
        }
    }
    >>> printFirst(listOf("Svetlana", "Dmitry")) // Svetlana
    ```
    

- 참고
    - [https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/9](https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/9)
    - [https://github.com/3tudy/kotlin/blob/master/Kotlin in Action/chapter09/chapter09.md](https://github.com/3tudy/kotlin/blob/master/Kotlin%20in%20Action/chapter09/chapter09.md)
