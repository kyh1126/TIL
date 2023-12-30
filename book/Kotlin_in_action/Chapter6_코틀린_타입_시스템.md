# Chapter 6. 코틀린 타입 시스템

## 1. 널 가능성

---

- 널 가능성(nullability): NPE 를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성
- 코틀린을 비롯한 최신 언어에서 `null`에 대한 접근 방법은 가능한 한 이 문제를 실행시점에서 컴파일 시점으로 옮기는 것이다.****
    
    → 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.
    

### 1-1. 널이 될 수 있는 타입

---

- 코틀린 타입 시스템이 `null`이 될 수 있는 타입을 명시적으로 지원한다.
- 타입 이름 뒤에 `?`를 붙이면 그 타입의 변수나 프로퍼티에 `null` 참조를 저장할 수 있다.
    
    ![6.1 널이 될 수 있는 타입의 변수에는 `null` 참조를 저장할 수 있다.](./image/6/Untitled.png)
    
    6.1 널이 될 수 있는 타입의 변수에는 `null` 참조를 저장할 수 있다.
    
    - `?`가 없는 타입은 그 변수가 `null` 참조를 저장할 수 없다.

👉 모든 타입은 기본적으로 널이 될 수 없는 타입이다.

- 1️⃣ 널이 될 수 있는 타입의 변수가 있다면 그에 대해 수행할 수 있는 연산이 제한된다.
    - ex> 널이 될 수 있는 변수에 대해 `변수.메서드()`처럼 메소드를 직접 호출할 수는 없다.
- 2️⃣ 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없다.
    
    ![Untitled](./image/6/Untitled%201.png)
    
- 3️⃣ 널이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 파라미터를 받는 함수에 전달할 수 없다.
    
    ```kotlin
    fun strLen(s: String) = s.length
    >>> val x: String? = null
    >>> strLen(x)
    ERROR: Type mismatch: inferred type is String? but String was expected
    ```
    

- 널이 될 수 있는 타입의 값으로 👉  `null`과 비교하는 것
    - 일단 `null`과 비교하고 나면 컴파일러는 그 사실을 기억하고 `null`이 아님이 확실한 영역에서는 해당 값을 널이 될 수 없는 타입의 값처럼 사용할 수 있다.
        
        ![Untitled](./image/6/Untitled%202.png)
        

### 1-2. 타입의 의미

---

- 자바의 타입 시스템이 널을 제대로 다루지 못한다
    - 자바에서 `String` 타입의 변수에는 `String`이나 `null`이라는 두 가지 종류의 값이 들어갈 수 있다.
    - `instanceof` 연산자도 `null`이 `String`이 아니라고 답한다.
- `NullPointerException` 오류를 다루는 다른 방법
    - 자바에도 `NullPointerException`문제를 해결하는 데 도움을 주는 도구가 있다.
    - 예를 들어 애노테이션을 사용해 값이 널이 될 수 있는지 여부를 표시(`@Nullable`이나 `@NotNull`)하기도 한다.
        - 하지만 그런 도구는(IDE 에 종속된) 표준 자바 컴파일 절차의 일부가 아니기 때문에 일관성 있게 적용된다는 보장을 할 수 없다.
        - 오류가 발생할 위치를 정확하게 찾기 위해 적절한 위치에 애노테이션을 추가하는 일도 쉽지는 않다.
    
    ❗이 문제를 해결하는 다른 방법은 `null` 값을 코드에서 절대로 쓰지 않는 것이다.
    
    - `null` 대신 자바 8에 새로 도입된 `Optional` 타입 등의 `null`을 감싸는 특별한 래퍼 타입을 활용할 수 있다. 하지만 이도 몇 가지 단점이 있다.
        - 코드가 더 지저분해지고 래퍼가 추가됨에 따라 실행 시점에 성능이 저하되며 전체 에코시스템에서 일관성 있게 활용하기 어렵다.****
        - 📚 [12 recipes for using the Optional class as it’s meant to be used](https://blogs.oracle.com/javamagazine/post/12-recipes-for-using-the-optional-class-as-its-meant-to-be-used)

- 실행 시점에 널이 될 수 있는 타입이나 널이 될 수없는 타입의 객체는 같다.
    - 널이  될 수 있는 타입은 널이 될 수 없는 타입을 감싼 래퍼 타입이 아니다.
- 모든 검사는 컴파일 타임에 수행된다
    
    🤭 코틀린에서는 널이 될 수 있는 타입을 처리하는 데 별도의 실행 시점 부가 비용이 들지 않는다.
    

### 1-3. 안전한 호출 연산자: `?.`

---

- `?.`은 `null` 검사와 메소드 호출을 한 번의 연산으로 수행한다.
    - 예를 들어 `s?.toUpperCase()`는 훨씬 더 복잡한 `if(s!=null) s.toUpperCase() else null`과 같다.
- 호출하려는 값이 `null`이면 이 호출은 무시되고 `null`이 결과 값이 된다.
    
    ![6.2 안전한 호출 연산자는 널이 아닌 값에 대해서만 메소드를 호출한다.](./image/6/Untitled%203.png)
    
    6.2 안전한 호출 연산자는 널이 아닌 값에 대해서만 메소드를 호출한다.
    

- 객체 그래프에서 널이 될 수 있는 중간 객체가 여럿 있다면 한 식 안에서 안전한 호출을 연쇄해서 함께 사용하면 편할 때가 자주 있다.
    
    ```kotlin
    class Address(val streetAddress: String, val zipCode: Int,
                  val city: String, val country: String)
    
    class Company(val name: String, val address: Address?)
    
    class Person(val name: String, val company: Company?)
    
    fun Person.countryName(): String {
        val country = this.company?.address?.country
        return if (country != null) country else "Unknown"
    }
    
    fun main(args: Array<String>) {
        val person = Person("Dmitry", null)
        println(person.countryName())
    }
    ```
    

### 1-4. 엘비스 연산자: `?:`

---

- 엘비스(elvis) 연산자: 코틀린은 `null` 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공한다.
- 코틀린에서는 `return`이나 `throw` 등의 연산도 식이다 → 엘비스 연산자의 우항에 `return`, `throw` 등의 연산을 넣을 수 있고, 엘비스 연산자를 더욱 편하게 사용할 수 있다. 👉 이런 패턴은 함수의 전제 조건을 검사하는 경우 특히 유용하다.
    - 좌항이 `null`이면 함수가 즉시 어떤 값을 반환하거나 예외를 던진다.
    
    ```kotlin
    class Address(val streetAddress: String, val zipCode: Int,
                  val city: String, val country: String)
    
    class Company(val name: String, val address: Address?)
    
    class Person(val name: String, val company: Company?)
    
    fun printShippingLabel(person: Person) {
        val address = person.company?.address ?: throw IllegalArgumentException("No address")
        with (address) {
            println(streetAddress)
            println("$zipCode $city, $country")
        }
    }
    
    fun main(args: Array<String>) {
        val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
        val jetbrains = Company("JetBrains", address)
        val person = Person("Dmitry", jetbrains)
        printShippingLabel(person)
        printShippingLabel(Person("jenny", null))
    }
    ```
    
    - printShippingLabel 함수는 모든 정보가 제대로 있으면 주소를 출력한다. 주소가 없으면 그냥 `NullPointerException`을 던지는 대신에 의미 있는 오류를 발생시킨다.****

### 1-5. 안전한 캐스트: `as?`

---

- `as`: 대상 값을 `as`로 지정한 타입으로 바꿀 수 없으면 `ClassCastException`이 발생한다.
- `as?`는 값을 대상 타입으로 변환할 수 없으면 `null`을 반환한다. 👉 이 패턴을 사용하면 파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 캐스트할 수 있고, 타입이 맞지 않으면 쉽게 `false`를 반환할 수 있다.****
    
    ![6.4 타입 캐스트 연산자는 값을 주어진 타입으로 변환하려 시도하고 타입이 맞지 않으면 `null`을 반환한다.](./image/6/Untitled%204.png)
    
    6.4 타입 캐스트 연산자는 값을 주어진 타입으로 변환하려 시도하고 타입이 맞지 않으면 `null`을 반환한다.
    
    ```kotlin
    class Person(val firstName: String, val lastName: String) {
       override fun equals(o: Any?): Boolean {
          val otherPerson = o as? Person ?: return false
    
          return otherPerson.firstName == firstName &&
                 otherPerson.lastName == lastName
       }
    
       override fun hashCode(): Int =
          firstName.hashCode() * 37 + lastName.hashCode()
    }
    
    fun main(args: Array<String>) {
        val p1 = Person("Dmitry", "Jemerov")
        val p2 = Person("Dmitry", "Jemerov")
        println(p1 == p2)
        println(p1.equals(42))
    }
    ```
    

### 1-6. 널 아님 단언: `!!`

---

- 어떤 값이든 널이 될 수 없는 타입으로 (강제로) 바꿀 수 있다.
- 실제 널에 대해 `!!`를 적용하면 NPE 가 발생한다.
    
    ![6.5 널 아님 단언을 사용하면 값이 널이 아닐 때 NPE 를 던질 수 있다.](./image/6/Untitled%205.png)
    
    6.5 널 아님 단언을 사용하면 값이 널이 아닐 때 NPE 를 던질 수 있다.
    
    ```kotlin
    fun ignoreNulls(s: String?) {
        val sNotNull: String = s!!
        println(sNotNull.length)
    }
    
    fun main() {
        ignoreNulls(null)
    }
    // Exception in thread "main" java.lang.NullPointerException
    //    at _6_typeSystem._1_nullability.NotNullAssertionEx5Kt.ignoreNulls(NotNullAssertionEx5.kt:7)
    //    at _6_typeSystem._1_nullability.NotNullAssertionEx5Kt.main(NotNullAssertionEx5.kt:12)
    //    at _6_typeSystem._1_nullability.NotNullAssertionEx5Kt.main(NotNullAssertionEx5.kt)
    ```
    
- 어떤 함수가 값이 널인지 검사한 다음에 다른 함수를 호출한다고 해도 컴파일러는 호출된 함수 안에서 안전하게 그 값을 사용할 수 있음을 인식할 수 없다.
- `!!`를 널에 대해 사용해서 발생하는 예외의 스택 트레이스(stack trace)에는 어떤 파일의 몇 번째 줄인지에 대한 정보는 들어있지만 어떤식에서 예외가 발생했는지에 대한 정보는 들어있지 않다.
    - 👉 어떤 값이 널이었는지 확실히 하기 위해 여러 `!!` 단언문을 한 줄에 함께 쓰는 일을 피하라.
        
        ```kotlin
        person.company!!.address!!.country // 이런 식으로 코드를 작성하지 말라.
        ```
        

<aside>
💡 컴파일러가 검증할 수 없는 단언을 사용하기 보다는 더 나은 방법을 찾아보라는 설계의도가 담겨있다.

</aside>

### 1-7. `let` 함수

---

- `let` 함수를 사용하면 널이 될 수 있는 식을 더 쉽게 다룰 수 있다.
    - `let` 함수를 안전한 호출 연산자(`?.`)와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있다.
- Function 선택: [https://kotlinlang.org/docs/scope-functions.html#function-selection](https://kotlinlang.org/docs/scope-functions.html#function-selection)

- `let` 함수는 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다.
    - 널이 될 수 있는 값에 대해 안전한 호출 구문을 사용해 `let`을 호출하되 널이 될 수 없는 타입을 인자로 받는 람다를 `let`에 전달한다.
        
        ![6.6 `let`을 안전하게 호출하면 수신 객체가 널이 아닌 경우 람다를 실행해준다.](./image/6/Untitled%206.png)
        
        6.6 `let`을 안전하게 호출하면 수신 객체가 널이 아닌 경우 람다를 실행해준다.
        
        ```kotlin
        fun sendEmailTo(email: String) {
            println("Sending email to $email")
        }
        
        fun main(args: Array<String>) {
            var email: String? = "yole@example.com"
            email?.let { sendEmailTo(it) } // Sending email to yole@example.com
            email = null
            email?.let { sendEmailTo(it) }
        }
        ```
        
- `let`을 쓰면 긴 식의 결과를 저장하는 변수를 따로 만들 필요가 없다.
    
    ```kotlin
    // before
    val person: Person? = getTheBestPersonInTheWorld()
    if (person != null) sendEmailTo(person.email)
    
    //after
    getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
    ```
    

- 여러 값이 널인지 검사해야 한다면 `let` 호출을 중첩시켜서 처리할 수 있다.
    
    😭 그렇게 `let`을 중첩시켜 처리하면 코드가 복잡해져서 알아보기 어려워진다.
    
    😃 그런 경우 일반적인 `if`를 사용해 모든 값을 한꺼번에 검사하는 편이 낫다.
    

### 1-8. 나중에 초기화할 프로퍼티

---

- 코틀린에서 클래스 안의 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화할지 않고 특별한 메소드 안에서 초기화할 수는 없다.
    - ex> jUnit 의 `@Before`
- 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.
    - 게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다.
- 그런 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용할 수밖에 없다.
    - 하지만 널이 될 수 있는 타입을 사용하면 모든 프로퍼티 접근에 널 검사를 넣거나 `!!` 연산자를 써야 한다.

- 널 아님 단언을 사용해 널이 될 수 있는 프로퍼티 접근하기 → 이 코드는 보기 나쁘다. 특히 프로퍼티를 여러 번 사용해야 하면 코드가 더 못생겨진다.
    
    ```kotlin
    class MyService {
        fun performAction(): String = "foo"
    }
    
    class MyTest {
        private var myService: MyService? = null // null로 초기화하기 위해 널이 될 수 있는 타입인 프로퍼티를 선언한다. 
    
        @Before fun setUp() {
            myService = MyService() // setUp 메소드 안에서 진짜 초깃값을 지정한다. 
        }
    
        @Test fun testAction() {
            Assert.assertEquals("foo",
                myService!!.performAction()) // 반드시 널 가능성에 신경 써야 한다. !!나 ?을 꼭 써야 한다. 
        }
    }
    ```
    
- 이를 해결하기 위해 myService 프로퍼티를 나중에 초기화(late-initialized)할 수 있다 👉 `lateinit` 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.
    
    ```kotlin
    class MyService {
        fun performAction(): String = "foo"
    }
    
    class MyTest {
        private lateinit var myService: MyService // 초기화하지 않고 널이 될 수 없는 프로퍼티를 선언한다. 
    
        @Before fun setUp() {
            myService = MyService() 
        }
    
        @Test fun testAction() {
            Assert.assertEquals("foo",
                myService.performAction()) // 널 검사를 수행하지 않고 프로퍼티를 사용한다. 
         }
    }
    ```
    

- 나중에 초기화하는 프로퍼티는 항상 `var`여야 한다.
    - `val` 프로퍼티는 `final` 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 한다.
- 널이 될 수 없는 타입이라 해도 더 이상 생성자 안에서 초기화할 필요가 없다.****
    - 그 프로퍼티를 초기화하기 전에 프로퍼티에 접근하면 “lateinit property myService has not been initialized”이라는 예외가 발생한다.
    
    → 단순한 `NullPointerException`이 발생하는 것보다 훨씬 좋다.
    

### 1-9. 널이 될 수 있는 타입 확장

---

- 널이 될 수 있는 타입에 대한 확장 함수를 정의하면 `null` 값을 다루는 강력한 도구로 활용할 수 있다.
    - 이런 처리는 확장 함수에서만 가능하다.
    - 일반 멤버 호출은 객체 인스턴스를 통해 디스패치 되므로 그 인스턴스가 널인지 여부를 검사하지 않는다.
    
    ```kotlin
    fun verifyUserInput(input: String?) {
        // Doesn't need Safe call operator
        // isNullOrBlank : nullable type's extension function
        if (input.isNullOrBlank()) {
            println("Please fill in the required fields")
        }
    }
    
    fun main() {
        verifyUserInput(" ") // Please fill in the required fields
        verifyUserInput(null) // Please fill in the required fields
    }
    ```
    
    ![6.7 널이 될 수 있는 타입의 확장 함수는 안전한 호출 없이도 호출 가능하다.](./image/6/Untitled%207.png)
    
    6.7 널이 될 수 있는 타입의 확장 함수는 안전한 호출 없이도 호출 가능하다.
    
    - 그 함수의 내부에서 `this`는 `null`이 될 수 있다.

<aside>
💡 처음에는 널이 될 수 없는 타입에 대한 확장 함수를 정의하라.

</aside>

### 1-10. 타입 파라미터의 널 가능성

---

- 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다.
- 타입 파라미터 `T`를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 `?`가 없더라도 `T`는 `null`이 될 수 있는 타입이다.
    
    ```kotlin
    fun <T> printHashCode(t: T) { // T의 타입은 Any?로 추론된다.
        println(t?.hashCode())
    }
    ```
    

### 1-11. 널 가능성과 자바

---

- 플랫폼 타입: 자바의 타입 중 코틀린이 `null` 관련 정보를 알 수 없는 타입
    
    ![6.9 자바 타입은 코틀린에서 플랫폼 타입으로 표현된다. 플랫폼 타입은 널이 될 수 있는 타입이나 널이 될 수 없는 타입 모두로 사용할 수 있다.](./image/6/Untitled%208.png)
    
    6.9 자바 타입은 코틀린에서 플랫폼 타입으로 표현된다. 플랫폼 타입은 널이 될 수 있는 타입이나 널이 될 수 없는 타입 모두로 사용할 수 있다.
    
    - 코틀린에서 플랫폼 타입을 선언할 수는 없다. 자바 코드에서 가져온 타입만 플랫폼 타입이 된다.
    - `!` 표기는 `String!` 타입의 `null` 가능성에 대해 아무 정보도 없다는 뜻이다.
        
        ```kotlin
        >>> val i: Int = person.name
        // ERROR: Type mismatch: inferred type is String! but Int was expected
        ```
        
    - 자바에서 가져온 `null` 값을 널이 될 수 없는 코틀린 변수에 대입하면 실행 시점에 대입이 이뤄질 때 예외가 발생한다.
- 상속
    - 코틀린에서 자바 메소드를 오버라이드할 때 그 메소드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야 한다.

## 2. 코틀린의 원시 타입

---

### 2-1. 원시타입: `Int`, `Boolean` 등

---

- 원시 타입의 변수에는 그 값이 직접 들어가지만, 참조 타입의 변수에는 메모리상의 객체 위치가 들어간다.
    - 자바는 참조 타입이 필요한 경우 특별한 래퍼 타입(`Integer` 등)으로 원시 타입 값을 감싸서 사용한다.
- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용한다.
    - 래퍼 타입을 따로 구분하지 않으면 편리하다. 더 나아가 코틀린에서는 숫자 타입 등 원시 타입의 값에 대해 메소드를 호출할 수 있다.
        
        ```kotlin
        fun showProgress(progress: Int) {
        		val percent = progress.coerceIn(0, 100)
        		println("We're ${percent}% done!")
        }
        ```
        

👉 실행 시점에 숫자 타입이 가능한 한 가장 효율적인 방식으로 표현된다. 대부분의 경우 코틀린의 `Int` 타입은 자바 `int` 타입으로 컴파일 된다.

- 자바 원시 타입에 해당하는 타입
    - 정수 타입 : `Byte`, `Short`, `Int`, `Long`
    - 부동소수점 수 타입 : `Float`, `Double`
    - 문자 타입 : `Char`
    - 불리언 타입 : `Boolean`****

### 2-2. 널이 될 수 있는 원시 타입: `Int?`, `Boolean?` 등

---

- `null` 참조를 자바의 참조 타입의 변수에만 대입할 수 있기 때문에 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없다. 따라서 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.
    
    ```kotlin
    data class Person(val name: String, val age: Int? = null) {
        fun isOlderThan(other: Person): Boolean? {
            if (age == null || other.age == null)
                return null
            return age > other.age
        }
    }
    
    fun main(args: Array<String>) {
        println(Person("Sam", 35).isOlderThan(Person("Jenny", 33))) // false
        println(Person("Sam", 35).isOlderThan(Person("Jenny"))) // null
    }
    ```
    
    - Person 클래스에 선언된 age 프로퍼티의 값은 `Integer`로 저장된다. 코틀린에서 적절한 타입을 찾으려면 그 변수나 프로퍼티에 널이 들어갈 수 있는지만 고민하면 된다.****
- 리스트는 래퍼인 `Integer` 타입으로 이뤄진 리스트다 👉 JVM 은 타입 인자로 원시 타입을 허용하지 않는다. 따라서 자바나 코틀린 모두에서 제네릭 클래스는 항상 박스 타입을 사용해야 한다.
    
    ```kotlin
    val listOfInts = listOf(1, 2, 3)
    ```
    

### 2-3. 숫자 변환

---

- 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다. 결과 타입이 허용하는 숫자의 범위가 원래 타입의 범위보다 넓은 경우 조차도 자동 변환은 불가능하다.
    
    ```kotlin
    val i = 1
    val l: long = i // "Error: type mismatch" 컴파일 오류 발생
    val l2: long = i.toLong()
    ```
    
- 코틀린은 모든 원시 타입에 대한 변환 함수를 제공한다.
    - 그런 변환 함수의 이름은 `toByte()`, `toShort()`, `toChar()` 등과 같다.
    
    → 어떤 타입을 더 표현 범위가 넓은 타입으로 변환하는 함수도 있고(`Int.toLong()`), 타입을 범위가 더 표현 범위가 좁은 타입으로 변환하면서 값을 벗어나는 경우에는 일부를 잘라내는 함수(`Long.toInt()`)도 있다.
    
- 숫자 리터럴을 타입이 알려진 변수에 대입하거나 함수에게 인자로 넘기면 컴파일러가 필요한 변환을 자동으로 넣어준다.
    
    ```kotlin
    fun foo(l: Long) = println(l)
    
    fun main() {
        val b: Byte = 1
        val ll = b + 1L
        foo(42) // 컴파일러는 42를 Long 값으로 해석한다.
    }
    ```
    
- 문자열을 숫자로 변환하기
    - 코틀린 표준 라이브러리는 문자열을 원시 타입으로 변환하는 여러 함수를 제공한다. (`toInt`, `toByte`, `toBoolean` 등)
        
        ```kotlin
        >>> println("42".toInt()) 
        42
        ```
        
    - 이런 함수는 문자열의 내용을 각 원시 타입을 표기하는 문자열로 파싱한다. 파싱에 실패하면 `NumberFormatException`이 발생한다.****

### 2-4. `Any`, `Any?`: 최상위 타입

---

- 자바에서 `Object`가 클래스 계층의 최상위 타입이듯 코틀린에서는 `Any` 타입이 모든 널이 될 수 없는 타입의 조상 타입이다.
- 코틀린에서는 `Any`가 `Int` 등의 원시 타입을 포함한 모든 타입의 조상 타입이다.
    
    ```kotlin
    val answer: Any = 42 // Any가 참조 타입이기 때문에 42가 박싱된다.
    ```
    
- `null`을 포함하는 모든 값을 대입할 변수를 선언하려면 `Any?`를 사용해야 한다.

- 내부적으로 `Any` 타입은 자바의 `Object`에 대응한다.
    - 하지만 `toString`, `equals`, `hashcode` 외 `java.lang.Object`에 있는 다른 메소드(`wait`, `notify` 등)는 `Any`에서 사용할 수 없다.
    - 그런 메소드를 호출하고 싶다면 `java.lang.Object` 타입으로 값을 캐스트해야 한다.

### 2-5. `Unit` 타입: 코틀린의 void

---

- 코틀린의 `Unit` 타입은 자바 `void`와 같은 기능을 한다.
    - 관심을 가질 만한 내용을 전혀 반환하지 않는 함수의 반환 타입으로 `Unit`을 쓸 수 있다.
        
        ```kotlin
        fun f(): Unit { ... }
        ```
        
- 이는 반환 타입 선언 없이 정의한 블록이 본문인 함수와 같다.
    
    ```kotlin
    fun f() { ... }
    ```
    

- 코틀린의 `Unit`이 자바 `void`와 다른 점은 무엇일까?
    - `Unit`은 모든 기능을 갖는 일반적인 타입이며, `void`와 달리 `Unit`을 타입 인자로 쓸 수 있다. `Unit` 타입에 속한 값은 단 하나뿐이며, 그 이름도 `Unit`이다. `Unit` 타입의 함수는 `Unit` 값을 묵시적으로 반환한다.
    - 제네릭 파라미터를 반환하는 함수를 오버라이드하면서 반환 타입으로 `Unit`을 쓸 때 유용하다.
        
        ```kotlin
        interface Processor<T> {
            fun process() : T
        }
        
        class NoResultProcessor : Processor<Unit> {
        		override fun process() { // Unit을 반환하지만 타입을 저장할 필요는 없다.
                // 업무 처리 코드
                // 여기서 return을 명시할 필요가 없다. 
        		}
        }
        ```
        
- 함수형 프로그래밍에서 전통적으로 `Unit`은 '단 하나의 인스턴스만 갖는 타입'을 의미해 왔고 바로 그 유일한 인스턴스의 유무가 자바 `void`와 코틀린 `Unit`을 구분하는 가장 큰 차이다.****
    - 어쩌면 자바 등의 명령형 프로그래밍 언어에서 관례적으로 사용해온 `Void`라는 이름을 사용할 수도 있겠지만, 코틀린에는 `Nothing`이라는 전혀 다른 기능을 하는 타입이 하나 존재한다.

### 2-6. `Nothing` 타입: 이 함수는 결코 정상적으로 끝나지 않는다

---

- 코틀린에는 결코 성공적으로 값을 돌려주는 일이 없으므로 '반환 값'이라는 개념 자체가 의미 없는 함수가 일부 존재한다.
    
    ```kotlin
    fun fail(message: String) : Nothing {
        throw IllegalStateException(message)
    }
    
    val address = company.address ?: fail("No address")
    println(address.city)
    ```
    
- `Nothing` 타입은 아무 값도 포함하지 않는다. → `Nothing`은 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터만 쓸 수 있다.****
- 컴파일러는 `Nothing`이 반환 타입인 함수가 결코 정상 종료되지 않음을 알고 그 함수를 호출하는 코드를 분석할 때 사용한다.

## 3. 컬렉션과 배열

---

### 3-1. 널 가능성과 컬렉션

---

- 컬렉션 안에 널 값을 넣을 수 있는지 여부는 어떤 변수의 값이 널이 될 수 있는지 여부와 마찬가지로 중요하다.
    - 변수 타입 뒤에 `?`를 붙이면 그 변수에 널을 저장할 수 있다는 뜻인 것처럼 타입 인자로 쓰인 타입에도 같은 표시를 사용할 수 있다.
        
        ```kotlin
        fun addValidNumbers(numbers: List<Int?>) {
            var sumOfValidNumbers = 0
            var invalidNumbers = 0
            for (number in numbers) {
                if (number != null) {
                    sumOfValidNumbers += number
                } else {
                    invalidNumbers++
                }
            }
            println("Sum of valid numbers : $sumOfValidNumbers")
            println("Invalid numbers : $invalidNumbers")
        }
        
        fun main() {
            val reader = BufferedReader(StringReader("1\nabc\n42"))
            val numbers = readNumbers(reader)
            addValidNumbers(numbers)
        }
        ```
        
    - 리스트의 원소에 접근하면 `Int?` 타입의 값을 얻는다. 따라서 그 값을 산술식에 사용하기 전에 널 여부를 검사해야 한다.
- 널이 될 수 있는 값으로 이뤄진 컬렉션으로 널 값을 걸러내는 경우가 자주 있어서 코틀린 표준 라이브러리는 그런 일을 하는 `filterNotNull`이라는 함수를 제공한다.
    
    ```kotlin
    fun addValidNumbers2(numbers: List<Int?>) {
        var validNumbers = numbers.filterNotNull()
    
        println("Sum of valid numbers : ${validNumbers.sum()}")
        println("Invalid numbers : ${numbers.size - validNumbers.size}")
    }
    
    fun main() {
        val reader = BufferedReader(StringReader("1\nabc\n42"))
        val numbers = readNumbers(reader)
        addValidNumbers(numbers)
    }
    ```
    

### 3-2. 읽기 전용과 변경 가능한 컬렉션

---

- 코틀린에서는 컬렉션안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다.
    - `kotlin.collections.Collection`: 코틀린 컬렉션을 다룰 때 사용하는 가장 기초적인 인터페이스****
    - `kotlin.collections.MutableCollection`: 컬렉션의 데이터 수정 인터페이스
        - 일반 인터페이스인 `kotlin.collections.Collection`을 확장하면서 원소를 추가하거나, 삭제하거나, 컬렉션 안의 원소를 모두 지우는 등의 메소드를 더 제공한다.
        
        ![6.11 `MutableCollection`은 `Collection`을 확장하면서 컬렉션 내용을 변경하는 메소드를 더 제공한다.](./image/6/Untitled%209.png)
        
        6.11 `MutableCollection`은 `Collection`을 확장하면서 컬렉션 내용을 변경하는 메소드를 더 제공한다.
        

### 3-3. 코틀린 컬렉션과 자바

---

- 모든 코틀린 컬렉션은 그에 상응하는 자바 컬렉션 인터페이스의 인스턴스라는 점은 사실이다.
- 하지만 코틀린은 모든 자바 컬렉션 인터페이스마다 읽기 전용 인터페이스와 변경 가능한 인터페이스라는 두 가지 표현을 제공한다.
    
    ![6.13 코틀린 컬렉션 인터페이스의 계층 구조. 자바 클래스 `ArrayList`와 `HashSet`은 코틀린의 변경 가능 인터페이스를 확장한다.](./image/6/Untitled%2010.png)
    
    6.13 코틀린 컬렉션 인터페이스의 계층 구조. 자바 클래스 `ArrayList`와 `HashSet`은 코틀린의 변경 가능 인터페이스를 확장한다.
    
    ![6.1 컬렉션 생성 함수](./image/6/Untitled%2011.png)
    
    6.1 컬렉션 생성 함수
    
- 이런 성질로 인해 컬렉션의 변경 가능성과 관련해 중요한 문제가 생긴다 👉 자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않으므로, 코틀린에서 읽기 전용 `Collection`으로 선언된 객체라도 자바 코드에서는 그 컬렉션 객체의 내용을 변경할수 있다.
    
    ```kotlin
    /* Java */
    // CollectionUtils.java
    public class CollectionUtils {
        public static List<String> uppercaseAll(List<String> items) {
            for (int i = 0; i < items.size(); i++) {
                items.set(i, items.get(i).toUpperCase());
            }
            return items;
        }
    }
    
    // Kotlin
    // collections.kt
    fun printInUppercase(list: List<String>) {
        println(CollectionUtils.uppercaseAll(list))
        println(list.first())
    }
    ```
    

### 3-4. 컬렉션을 플랫폼 타입으로 다루기

---

- 컬렉션 타입이 시그니처에 들어간 자바 메소드 구현을 코틀린에서 사용할 컬렉션 타입에 반영할 때 선택 사항
    - 컬렉션이 널이 될 수 있는가?
    - 컬렉션의 원소가 널이 될 수 있는가?
    - 오버라이드하는 메소드가 컬렉션을 변경할 수 있는가?

👉 선택을 제대로 하려면 자바 인터페이스나 클래스가 어떤 맥락에서 사용되는지 정확히 알아야 한다.

### 3-5. 객체의 배열과 원시 타입의 배열

---

- 코틀린 배열: 타입 파라미터를 받는 클래스다. 배열의 원소 타입은 바로 그 타입 파라미터에 의해 정해진다.
    
    ```kotlin
    fun main(args: Array<String>) {
        for (i in args.indices) {
            println("Argument $i is: ${args[i]}")
        }
    }
    ```
    
- 코틀린에서 배열을 만드는 방법
    - `arrayOf` 함수에 원소를 넘기면 배열을 만들 수 있다.
    - `arrayOfNulls` 함수에 정수 값을 인자로 넘기면 모든 원소가 `null`이고 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있다. 물론 원소 타입이 널이 될 수 있는 타입인 경우에만 이 함수를 쓸 수 있다.
    - `Array` 생성자는 배열 크기와 람다를 인자로 받아서 람다를 호출해서 각 배열 원소를 초기화해준다.
        - `arrayOf`를 쓰지 않고 각 원소가 널이 아닌 배열을 만들어야 하는 경우 이 생성자를 사용한다.
        
        ```kotlin
        val letters = Array<String>(26) { i -> ('a' + i).toString() }
        println(letters.joinToString("")) // abcdefghijklmnopqrstuvwxyz
        ```
        

- 코틀린에서는 배열을 인자로 받는 자바 함수를 호출하거나 `vararg` 파라미터를 받는 코틀린 함수를 호출하기 위해 가장 자주 배열을 만든다.
    - `toTypedArray` 메소드를 사용하면 쉽게 컬렉션을 배열로 바꿀 수 있다.
    
    ```kotlin
    val strings = listOf("a", "b", "c")
    println("%s/%s/%s".format(*strings.toTypedArray())) // a/b/c
    ```
    

- 코틀린은 원시 타입의 배열을 표현하는 별도 클래스를 각 원시 타입마다 하나씩 제공한다.
    - ex> `IntArray`: `Int` 타입의 배열.
    - 코틀린은 `ByteArray`, `CharArray`, `BooleanArray` 등의 원시 타입 배열을 제공한다.
    
    👉 자바 원시 타입 배열인 `int[]`, `byte[]`, `char[]` 등으로 컴파일 된다. 박싱하지 않고 가장 효율적인 방식으로 저장된다.
    
- 원시 타입의 배열을 만드는 방법
    - 각 배열 타입의 생성자는 `size` 인자를 받아서 해당 원시 타입의 디폴트 값으로 초기화된 `size` 크기의 배열을 반환한다.
        
        ```kotlin
        val fiveZeros = IntArray(5)
        ```
        
    - 팩토리 함수(`IntArray`를 생성하는 `intArrayOf` 등)는 여러 값을 가변 인자로 받아서 그런 값이 들어간 배열을 반환한다.
        
        ```kotlin
        val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0)
        ```
        
    - (일반 배열과 마찬가지로) 크기와 람다를 인자로 받는 생성자를 사용한다.
        
        ```kotlin
        val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
        println(squares.joinToString())
        ```
        

- `forEachIndexed`: 배열의 모든 원소를 갖고 인자로 받은 람다를 호출해준다.
    
    ```kotlin
    val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
    println(squares.joinToString())
    
    squares.forEachIndexed { index, element ->
        println("Argument $index is: $element")
    }
    // Argument 0 is: 1
    // Argument 1 is: 4
    // Argument 2 is: 9
    // Argument 3 is: 16
    // Argument 4 is: 25
    ```
    

- 참고
    - [https://essie-cho.com/kotlin-in-action-06/](https://essie-cho.com/kotlin-in-action-06/)
    - [https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/6](https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/6)
