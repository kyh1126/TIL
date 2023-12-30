# Chapter 10. 애노테이션과 리플렉션

---

- 어떤 함수를 호출하려면 그 함수가 정의된 클래스의 이름과 함수 이름, 파라미터 이름 등을 알아야만 했다. 애노테이션과 리플렉션을 사용하면 그런 제약을 벗어나서 미리 알지 못하는 임의의 클래스를 다룰 수 있다. 애노테이션을 사용하면 라이브러리가 요구하는 의미를 클래스에게 부여할 수 있고, 리플렉션을 사용하면 실행 시점에 컴파일러 내부 구조를 분석할 수 있다.****

## 1. 애노테이션 선언과 적용

---

### 1-1. 애노테이션 적용

---

- 애노테이션을 적용하려면 적용하려는 대상 앞에 애노테이션을 붙이면 된다. 애노테이션은 `@`과 애노테이션 이름으로 이뤄진다.****
    - ex> `@Deprecated`:코틀린에서는 `replaceWith` 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제시할 수 있고, API 사용자는 그 패턴을 보고 지원이 종료될 API 기능을 더 쉽게 새 버전으로 포팅할 수 있다.
        
        ```kotlin
        @Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
        fun remove(index: Int) {...}
        ```
        

- 애노테이션의 인자로는 원시 타입의 값, 문자열, `enum`, 클래스 참조, 다른 애노테이션 클래스, 그리고 지금까지 말한 요소들로 이뤄진 배열이 들어갈 수 있다. 애노테이션 인자를 지정하는 문법은 자바와 약간 다르다.
    - 클래스를 애노테이션 인자로 지정할 때는 `@MyAnnotation(MyClass::class)`처럼 `::class`를 클래스 이름 뒤에 뒤에 넣어야 한다.
    - 다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션의 이름 앞에 `@`를 넣지 않아야 한다. 예를 들어 방금 살펴본 예제의 `ReplaceWith`는 애노테이션이다. 하지만 `Deprecated` 애노테이션의 인자로 들어가므로 `ReplaceWith` 앞에 `@`를 사용하지 않는다.
        
        ```kotlin
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(
            name = "order_id", referencedColumnName = "orderId",
            insertable = false, updatable = false, // read-only
            foreignKey = ForeignKey(value = ConstraintMode.NO_CONSTRAINT)
        )
        var confirmOrder: ConfirmOrder? = null
        ```
        
    - 배열을 인자로 지정하려면 `@RequestMapping(path = arrayOf("/foo", "/bar"))`처럼 `arrayOf` 함수를 사용한다. 자바에서 선언한 애노테이션 클래스를 사용한다면 `value`라는 이름의 파라미터가 필요에 따라 자동으로 가변 길이 인자로 변환된다. 따라서 그런 경우에는 `@JavaAnnotationWithArrayValue("abc", "foo", "bar")`처럼 `arrayOf` 함수를 쓰지 않아도 된다.

- 애노테이션 인자를 컴파일 시점에 알 수 있어야 한다. 따라서 임의의 프로퍼티를 인자로 지정할 수는 없다. 프로퍼티를 애노테이션 인자로 사용하려면 그 앞에 `const` 변경자를 붙여야 한다.
    - 컴파일러는 `const`가 붙은 프로퍼티를 컴파일 시점 상수로 취급한다.
    
    ```kotlin
    const val TEST_TIMEOUT = 100L
    
    @Test(timeout = TEST_TIMEOUT)
    fun testMethod() { ... }
    ```
    

### 1-2. 애노테이션 대상

---

- 코틀린 소스코드에서 한 선언을 컴파일한 결과가 여러 자바 선언과 대응하는 경우가 자주 있다. 그리고 이때 코틀린 선언과 대응하는 여러 자바 선언에 각각 애노테이션을 붙여야 할 때가 있다.
    - 1️⃣ 코틀린 프로퍼티는 기본적으로 자바 필드와 게터 메소드 선언과 대응한다.
    - 2️⃣ 프로퍼티가 변경 가능하면 세터에 대응하는 자바 세터 메소드와 세터 파라미터가 추가된다.
    - 3️⃣ 주 생성자에서 프로퍼티를 선언하면 이런 접근자 메소드와 파라미터 외에 자바 생성자 파라미터와도 대응이 된다.

- 사용 지점 대상(use-site target) 선언으로 애노테이션을 붙일 요소를 정할 수 있다.
    - 사용 지점 대상은 `@` 기호와 애노테이션 이름 사이에 붙으며, 애노테이션 이름과는 콜론(`:`)으로 분리된다.
        
        ![10.1 사용 지점 대상 지정 문법](./image/10/Untitled.png)
        
        10.1 사용 지점 대상 지정 문법
        
        - `@Rule` 어노테이션을 프로퍼티 게터에 적용하라는 뜻
    - 규칙을 지정하려면 공개(`public`) 필드나 메소드 앞에 `@Rule`을 붙여야 한다.
        - `@Rule` 은 필드에 적용되지만 코틀린의 필드는 기본적으로 비공개이기 때문에 이런 예외가 생긴다 → `@get:Rule`(프로퍼티가 아니라 게터에 애노테이션이 붙는다)
            - This `@Rule` 'folder' must be public

- 자바에 선언된 애노테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우 기본적으로 프로퍼티의 필드에 그 애노테이션이 붙는다 👉 코틀린으로 애노테이션을 선언하면 프로퍼티에 직접 적용할 수 있는 애노테이션을 만들 수 있다.
- 사용 지점 대상을 지정할 때 지원하는 대상 목록
    
    
    | 대상 | 설명 |
    | --- | --- |
    | property | 프로퍼티 전체. 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다. |
    | field | 프로퍼티에 의해 생성되는 (뒷받침하는) 필드 |
    | get | 프로퍼티 게터 |
    | set | 프로퍼티 세터 |
    | receiver | 확장 함수나 프로퍼티의 수신 객체 파라미터 |
    | param | 생성자 파라미터 |
    | setparam | 세터 파라미터 |
    | delegate | 위임 프로퍼티의 위임 인스턴스를 담아둔 필드 |
    | file | 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스 |
    - `file` 대상을 사용하는 애노테이션은 `package` 선언 앞에서 파일의 최상위 수준에만 적용할 수 있다.****
        - ex> `@JvmName`: 파일에 있는 최상위 선언을 담는 클래스의 이름을 바꿔준다. `@file:JvmName("StringFunctions")`

- ❗자바 API 를 애노테이션으로 제어하기
    - `@JvmName`: 코틀린 선언이 만들어내는 자바 필드나 메소드 이름을 변경한다.
    - `@JvmStatic`: 메소드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메소드로 노출된다.
    - `@JvmOverloads`: 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성해준다.
    - `@JvmField`: 프로퍼티에 사용하면 게터나 세터가 없는 공개된(public) 자바 필드로 프로퍼티를 노출시킨다.

### 1-3. 애노테이션을 활용한 JSON 직렬화 제어

---

- 직렬화(serialization): 객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것
- 역직렬화(deserialization): 텍스트나 이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어 낸다.
    
    ![10.2 Person 인스턴스의 직렬화와 역직렬화](./image/10/Untitled%201.png)
    
    10.2 Person 인스턴스의 직렬화와 역직렬화
    

- 제이키드 라이브러리
    
    ```kotlin
    package _10_annotationAndReflection._1_annotation
    
    import _10_annotationAndReflection.jkid.JsonExclude
    import _10_annotationAndReflection.jkid.JsonName
    import _10_annotationAndReflection.jkid.deserialization.deserialize
    import _10_annotationAndReflection.jkid.serialization.serialize
    
    // @JsonName(name="first_name")과 같이 표시할 수도 있다.
    data class Person(
        @JsonName("alias") val name: String,
        @JsonExclude val age: Int? = null
    )
    
    fun main() {
        val person = Person("Jenny", 31)
        // 직렬화
        println(serialize(person))
    
    //    val json = """{"name":"Jenny", "age":31}"""
        val json = """{"alias":"Jenny"}"""
        // 역직렬화
        println(deserialize<Person>(json))
    }
    ```
    
    - `@JsonExclude`: 직렬화나 역직렬화 시 그 프로퍼티를 무시할 수 있다.****
    - `@JsonName`: 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름을 쓰게 할 수 있다.

### 1-4. 애노테이션 선언

---

- 애노테이션 클래스는 오직 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 내부에 아무 코드도 들어있을 수 없다. 컴파일러는 애노테이션 클래스에서 본문을 정의하지 못하게 막는다.
    
    ```kotlin
    annotation class JsonExclude
    ```
    
- 파라미터가 있는 애노테이션을 정의하려면 애노테이션 클래스의 주 생성자에 파라미터를 선언해야 한다.
    
    ```kotlin
    annotation class JsonName(val name: String)
    ```
    
    - 다만 애노테이션 클래스에서는 모든 파라미터 앞에 `val`을 붙여야만 한다.

- 자바 애노테이션에서는 `value`라는 메소드를 썼다. (ex> 제이키드 애노테이션)
    
    ```kotlin
    public @interface JsonName {
        String value(); // 자바에서는 name 프로퍼티 대신 value 프로퍼티를 사용한다.
    }
    ```
    
- 자바 👉 어떤 애노테이션을 적용할 때 `value`를 제외한 모든 애트리뷰트에는 이름을 명시해야 한다.
- 코틀린 👉 일반적인 생성자 호출과 같다.

### 1-5. 메타 애노테이션: 애노테이션을 처리하는 방법 제어

---

- 자바와 마찬가지로 코틀린 애노테이션 클래스에도 애노테이션을 붙일 수 있다.
- 메타애노테이션(meta-annotation): 애노테이션 클래스에 적용할 수 있는 애노테이션
    - `@Target`: 표준 라이브러리에 있는 메타애노테이션 중 가장 흔히 쓰이는 메타애노테이션. 애노테이션을 적용할 수 있는 요소의 유형을 지정한다. 지정하지 않으면 모든 선언에 적용할 수 있는 애노테이션이 된다.
    
    ```kotlin
    @Target(AnnotaionTarget.PROPERTY)
    annotation class JsonExclude
    ```
    

<aside>
💡 `@Retention`: 정의 중인 애노테이션 클래스를 소스 수준에서만 유지할지 `.class` 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타애노테이션. 자바 컴파일러는 기본적으로 애노테이션을 `.class` 파일에는 저장하지만 런타임에는 사용할 수 없게 한다. 대부분의 애노테이션은 런타임에도 사용할 수 있어야 하므로 코틀린에서는 기본적으로 애노테이션의 `@Retention`을 `RUNTIME`으로 지정한다.****

</aside>

### 1-6. 애노테이션 파라미터로 클래스 사용

---

- 어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능이 필요할 때 👉 클래스 참조를 파라미터로 하는 애노테이션 클래스를 선언하면 그런 기능을 사용할 수 있다.
    
    ```kotlin
    package _10_annotationAndReflection._1_annotation
    
    import _10_annotationAndReflection.jkid.DeserializeInterface
    
    interface Company {
        val name: String
    }
    data class CompanyImpl(override val name: String) : Company
    
    // CompanyImpl 은 out Any 의 하위 타입이다.
    data class Person2(
        val name: String,
        @DeserializeInterface(CompanyImpl::class) val company: Company
    )
    
    // 클래스 참조를 인자로 받는 애노테이션 정의
    @Target(AnnotationTarget.PROPERTY)
    annotation class DeserializeInterface(val targetClass: KClass<out Any>)
    ```
    
    - 일반적으로 클래스를 가리키려면 클래스 이름 뒤에 `::class` 키워드를 붙여야 한다.
- `KClass`: 자바 `java.lang.Class`타입과 같은 역할을 하는 코틀린 타입
    - 코틀린 클래스에 대한 참조를 저장할 때 `KClass` 타입을 사용한다.
- `KClass`의 타입 파라미터는 이 `KClass`의 인스턴스가 가리키는 코틀린 타입을 지정한다.
    - ex> `CompanyImpl::class`의 타입 → `KClass<CompanyImpl>`
        
        ![10.4 애노테이션에 인자로 전달한 `CompanyImpl::class`의 타입인 `KClass<CompanyImpl>`은 애노테이션의 파라미터 타입인 `KClass<out Any>`의 하위 타입이다.](./image/10/Untitled%202.png)
        
        10.4 애노테이션에 인자로 전달한 `CompanyImpl::class`의 타입인 `KClass<CompanyImpl>`은 애노테이션의 파라미터 타입인 `KClass<out Any>`의 하위 타입이다.
        
    - ex> fn-product 내 SQS 메시지 속성 `enum`
        
        ```kotlin
        fun <T: Any> **cast**(any: Any, clazz: Class<out T>): T = clazz.cast(any)
        
        private val messageGroupId = MESSAGE_GROUP_ID.value
        val convertedValue = **cast**(messageGroupId, MESSAGE_GROUP_ID.type)
        ```
        
        ![Untitled](./image/10/Untitled%203.png)
        
        ⬇️
        
        ```kotlin
        fun <T : Any> **cast**(any: Any, clazz: KClass<out T>): T =
            clazz.javaObjectType.cast(any)
        
        private val messageGroupId = MESSAGE_GROUP_ID.value
        val convertedValue = **cast**(messageGroupId, MESSAGE_GROUP_ID.type)
        ```
        
        ![Untitled](./image/10/Untitled%204.png)
        

### 1-7. 애노테이션 파라미터로 제네릭 클래스 받기

---

- ex> 제이키드 애노테이션 `@CustomSerializer`: 커스텀 직렬화 클래스에 대한 참조를 인자로 받는다.****
    
    ```kotlin
    package _10_annotationAndReflection._1_annotation
    
    import _10_annotationAndReflection.jkid.CustomSerializer
    import _10_annotationAndReflection.jkid.DateSerializer
    import java.util.*
    
    // 기본적으로 제이키드는 원시 타입이 아닌 프로퍼티를 중첩된 객체로 직렬화한다.
    // 직렬화하는 로직 직접 제공
    data class Person3(
        val name: String,
        @CustomSerializer(DateSerializer::class) val birthDate: Date
    )
    
    // 제네릭 클래스라서 참조하려면 항상 타입 인자를 제공해야 한다.
    interface ValueSerializer<T> {
        fun toJsonValue(value: T): Any?
        fun fromJsonValue(jsonValue: Any?): T
    }
    
    @Target(AnnotationTarget.PROPERTY) 
    annotation class CustomSerializer(
        val serializerClass: KClass<out ValueSerializer<*>>
    )
    ```
    
    - CustomSerializer 애노테이션이 어떤 타입에 대해 쓰일지 알 수 없으므로 ValueSerializer 에 스타 프로젝션을 썼다.
        
        ![10.5 애노테이션 파라미터 serializerClass 의 타입. ValueSerializer 를 확장하는 클래스에 대한 참조만 올바른 인자로 인정된다.](./image/10/Untitled%205.png)
        
        10.5 애노테이션 파라미터 serializerClass 의 타입. ValueSerializer 를 확장하는 클래스에 대한 참조만 올바른 인자로 인정된다.
        
- 클래스를 인자로 받아야 하면 애노테이션 파라미터 타입에 `KClass<out 허용할 클래스 이름>`을 쓴다.
- 제네릭 클래스를 인자로 받아야 하면 `KClass<out 허용할 클래스 이름<*>>`처럼 허용할 클래스의 이름 뒤에 스타 프로젝션을 덧붙인다.

## 2. 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

---

- 리플렉션: 실행 시점에 (동적으로) 객체의 프로퍼티와 메소드에 접근할 수 있게 해주는 방법
- 보통 객체의 메소드나 프로퍼티에 접근할 때는 프로그램 소스코드 안에 구체적인 선언이 있는 메소드나 프로퍼티 이름을 사용하며, 컴파일러는 그런 이름이 실제로 가리키는 선언을 컴파일 시점에 (정적으로) 찾아내서 해당하는 선언이 실제 존재함을 보장한다.
- 하지만 타입과 관계없이 객체를 다뤄야 하거나 객체가 제공하는 메소드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우가 있다 👉 리플렉션을 사용해야 한다.
    - JSON 직렬화 라이브러리가 그런 경우다.
    - 직렬화 라이브러리는 어떤 객체든 JSON으로 변환할 수 있어야 하고, 실행 시점이 되기 전까지는 라이브러리가 직렬화할 프로퍼티나 클래스에 대한 정보를 알 수 없다.

- 코틀린에서 리플렉션을 사용하려면 두 가지 서로 다른 리플렉션 API 를 다뤄야 한다.
    - 1️⃣ 자바가 `java.lang.reflect` 패키지를 통해 제공하는 표준 리플렉션이다. 코틀린 클래스는 일반 자바 바이트코드로 컴파일되므로 자바 리플렉션 API 도 코틀린 클래스를 컴파일한 바이트코드를 완벽히 지원한다.
    - 2️⃣ 코틀린이 `kotlin.reflect` 패키지를 통해 제공하는 코틀린 리플렉션 API 다. 이 API는 자바에는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션을 제공한다. 하지만 현재 코틀린 리플렉션 API 는 자바 리플렉션 API 를 완전히 대체할 수 있는 복잡한 기능을 제공하지는 않는다.****

<aside>
💡 코틀린 리플렉션 패키지의 메이븐 그룹/아티팩트 ID: `implementation("org.jetbrains.kotlin:kotlin-reflect")`

</aside>

### 2-1. 코틀린 리플렉션 API: `KClass`, `KCallable`, `KFunction`, `KProperty`

---

- 코틀린 리플렉션 API 를 사용할 때 처음 접하게 되는 것은 클래스를 표현하는 `KClass`다.
- MyClass`::class`라는 식을 쓰면 `KClass`의 인스턴스를 얻을 수 있다.
- 일단 자바 클래스를 얻었으면 `.kotlin` 확장 프로퍼티를 통해 자바에서 코틀린 리플렉션 API 로 옮겨올 수 있다.
    
    ```kotlin
    package _10_annotationAndReflection._2_reflection
    
    import kotlin.reflect.full.memberProperties
    
    class Person(val name: String, val age: Int)
    
    fun main() {
        val person = Person("Alice", 29)
        val kClass = person.javaClass.kotlin
        println(kClass.simpleName) // Person
    
        kClass.memberProperties.forEach { print(it.name + " ") } // age name
    }
    
    // JvmClassMapping.kt
    public val <T : Any> Class<T>.kotlin: KClass<T>
        @JvmName("getKotlinClass")
        get() = Reflection.getOrCreateKotlinClass(this) as KClass<T>
    ```
    
    - 이 예제는 클래스 이름과 그 클래스에 들어있는 프로퍼티 이름을 출력하고 member Properties를 통해 클래스와 모든 조상 클래스 내부에 정의된 비확장 프로퍼티를 모두 가져온다.

- `KClass`: 클래스의 내부를 살펴볼 때 사용할 수 있는 다양한 메소드를 볼 수 있다.
    
    ```kotlin
    interface KClass<T : Any> {
        val simpleName: String?
        val qualifiedName: String?
        val members: Collection<KCallable<*>>
        val constructors: Collection<KFunction<T>>
        val nestedClasses: Collection<KClass<*>>
        ...
    }
    ```
    
    ![Untitled](./image/10/Untitled%206.png)
    
- `KCallable`: 함수와 프로퍼티를 아우르는 공통 상위 인터페이스
    - `call`: 함수나 프로퍼티의 게터를 호출할 수 있다. 타입 안정성을 보장해주지는 않는다.
        
        ```kotlin
        import kotlin.reflect.full.memberProperties
        
        fun foo(x: Int) = println(x)
        
        fun main() {
            val kFunction = ::foo
        
        //    kFunction.invoke(42) // 42
            kFunction.call(42)
            // Exception in thread "main" kotlin.reflect.jvm.internal.KotlinReflectionInternalError: Function 'foo' (JVM signature: foo(I)V) not resolved in package _10_annotationAndReflection._2_reflection: no members found
        }
        
        ->
        
        import _10_annotationAndReflection._2_reflection.KotlinReflectionApiEx1.foo
        import kotlin.reflect.full.memberProperties
        
        object KotlinReflectionApiEx1 {
            fun foo(x: Int) = println(x)
        }
        
        fun main() {
            val kFunction = ::foo
        
            kFunction.call(42) // 42
        }
        ```
        
    
    ```kotlin
    interface KCallable<out R> {
        fun call(vararg args: Any?): R
        ...
    }
    ```
    
    ![Untitled](./image/10/Untitled%207.png)
    
- `KFunction1<Int, Unit>`: 파라미터와 반환 값 타입 정보가 들어있다.
    - `invoke`: `KFunction1` 인터페이스를 통해 함수를 호출. 인자 개수나 타입이 맞아 떨어지지 않으면 컴파일이 안 된다 👉 `KFunction`의 인자 타입과 반환 타입을 모두 다 안다면 `invoke` 메소드를 호출하는 게 낫다.
        
        ```kotlin
        fun foo(x: Int) = println(x)
        
        fun main() {
            val kFunction = ::foo
        
            kFunction.invoke(42) // 42
        }
        ```
        
- `KProperty1<Person, Int>`: 제네릭 클래스. 첫 번째 타입 파라미터는 수신 객체 타입, 두 번째 타입 파라미터는 프로퍼티 타입을 표현한다.
    - `call`: 프로퍼티의 게터를 호출한다.
        
        ```kotlin
        package _10_annotationAndReflection._2_reflection
        
        import _10_annotationAndReflection._2_reflection.KotlinReflectionApiEx1.counter
        import kotlin.reflect.full.memberProperties
        
        class Person(val name: String, val age: Int)
        
        object KotlinReflectionApiEx1 {
            var counter = 0
        }
        
        fun main() {
            val kProperty = ::counter
            kProperty.setter.call(21)
            println(kProperty.get()) // 21
        }
        ```
        
    - 프로퍼티 인터페이스는 프로퍼티 값을 얻는 더 좋은 방법으로 `get` 메소드를 제공한다.
    - 멤버 프로퍼티는 `KProperty1` 인스턴스로 표현된다. 어떤 객체에 속해 있는 프로퍼티이므로 멤버 프로퍼티의 값을 가져오려면 `get` 메소드에게 프로퍼티를 얻고자 하는 객체 인스턴스를 넘겨야 한다.
        - 인자가 1개인 `get` 메소드가 들어있다.
        
        ```kotlin
        class Person(val name: String, val age: Int)
        
        fun main() {
            val person = Person("Alice", 29)
        
            val memberProperty = Person::age
            println(memberProperty.get(person)) // 29
        }
        ```
        
        ![Untitled](./image/10/Untitled%208.png)
        
- 최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 접근할 수 있고 함수의 로컬 변수에는 접근할 수 없다.
- `KClass`, `KFunction`, `KParameter`는 모두 `KAnnotatedElement`를 확장한다.
    
    ![Untitled](./image/10/Untitled%209.png)
    

- 💡 실행 시점에 소스코드 요소에 접근하기 위해 사용할 수 있는 인터페이스의 계층 구조
    
    ![10.6 코틀린 리플렉션 API 의 인터페이스 계층 구조](./image/10/Untitled%2010.png)
    
    10.6 코틀린 리플렉션 API 의 인터페이스 계층 구조
    
    - `KClass`: 클래스와 객체를 표현할 때 쓰인다.
    - `KProperty`: 모든 프로퍼티를 표현할 수 있다. / `KMutableProperty`: `var`로 정의한 변경 가능한 프로퍼티를 표현한다.
        - 선언된 `getter`와 `setter` 인터페이스로 프로퍼티 접근자를 함수처럼 다룰 수 있다 → `KFunction`을 확장한다.

### 2-2. 리플렉션을 사용한 객체 직렬화 구현

---

- 직렬화 함수의 기능을 살펴보자. 기본적으로 직렬화 함수는 객체의 모든 프로퍼티를 직렬화한다.
    
    ```kotlin
    private fun StringBuilder.serializeObject(obj: Any) {
        val kClass = obj.javaClass.kotlin // 객체의 KClass를 얻는다.
        val properties = kClass.memberProperties // 클래스의 모든 프로퍼티를 얻는다.
        properties.joinToStringBuilder(this, prefix = "{", postfix = "}") { prop ->
            serializeString(prop.name) // 프로퍼티 이름을 얻는다.
            append(": ")
            serializePropertyValue(prop.get(obj)) // 프로퍼티 값을 얻는다.
        }
    }
    ```
    
    - 이 예제에서는 어떤 객체의 클래스에 정의된 모든 프로퍼티를 열거하기 때문에 정확히 각 프로퍼티가 어떤 타입인지 알 수 없다 → prop 변수의 타입은 `KProperty1<Any, *>`이며, prop.get(obj) 메소드를 호출은 `Any` 타입의 값을 반환한다.

### 2-3. 애노테이션을 활용한 직렬화 제어

---

- JSON 직렬화 과정을 제어하는 과정 중에서 `@JsonExclude`를 사용하여 특정 필드들을 제외하고 싶을 경우가 있다.(제이키드)
    
    ```kotlin
    // 인자로 전달받은 타입에 해당하는 애노테이션이 있으면 그 애노테이션을 반환한다.
    inline fun <reified T> KAnnotatedElement.findAnnotation(): T? = 
        annotations.filterIsInstance<T>().firstOrNull()
    
    private fun StringBuilder.serializeObject(obj: Any) {
        obj.javaClass.kotlin.memberProperties
            .filter { it.findAnnotation<JsonExclude>() == null }
            .joinToStringBuilder(this, prefix = "{", postfix = "}") {
                serializeProperty(it, obj)
            }
    }
    ```
    
- `KAnnotatedElement` 인터페이스는 `annotations` 프로퍼티가 있다.
    - `annotations`: 소스코드상에서 해당 요소에 적용된 (`@Retention`을 `RUNTIME`으로 지정한) 모든 애노테이션 인스턴스의 컬렉션이다.
    - `KProperty`는 `KAnnotatedElement`를 확장하므로 `property.annotations`를 통해 프로퍼티의 모든 애노테이션을 얻을 수 있다.
    - `KAnnotatedElement.findAnnotation`: 인자로 전달받은 타입에 해당하는 애노테이션이 있으면 그 애노테이션을 반환한다.

- 클래스와 객체는 모두 `KClass`로 표현된다.
    - 객체에는 `object` 선언에 의해 생성된 싱글턴을 가리키는 `objectInstance`라는 프로퍼티가 있다 👉 `objectInstance` 프로퍼티에 싱글턴 인스턴스가 들어있다. createInstance 를 호출할 필요가 없다.
        
        ![Untitled](./image/10/Untitled%2011.png)
        

### 2-4. JSON 파싱과 객체 역직렬화

---

- JSON 파싱: 렉서, 파서, 역직렬화기
    
    ![Untitled](./image/10/Untitled%2012.png)
    

### 2-5. 최종 역직렬화 단계: `callBy()`, 리플렉션을 사용해 객체 만들기

---

- `KCallable.call`은 인자 리스트를 받아서 함수나 생성자를 호출해준다. 디폴트 파라미터 값을 지원하지 않는다는 한계가 있다.
- `KCallable.callBy`: 디폴트 파라미터 값을 지원. 파라미터와 파라미터에 해당하는 값을 연결해주는 맵을 인자로 받는다. 맵에서 파라미터를 찾을 수 없는데, 파라미터 디폴트 값이 정의돼 있다면 그 디폴트 값을 사용한다. `args` 맵에 들어있는 각 값의 타입이 생성자의 파라미터 타입과 일치해야 한다. 그렇지 않으면 `IllegalArgumentException`이 발생한다.
    - `callBy`: 생성자 파라미터와 그 값을 연결해주는 맵을 넘기면 객체의 주 생성자를 호출할 수 있다.
        
        ```kotlin
        package _10_annotationAndReflection._2_reflection
        
        import kotlin.reflect.KClass
        import kotlin.reflect.KParameter
        import kotlin.reflect.full.primaryConstructor
        
        class ClassInfo<T : Any>(cls: KClass<T>) {
            private val constructor = cls.primaryConstructor!!
            fun createInstance(arguments: Map<KParameter, Any?>): T {
        //        ensureAllParametersPresent(arguments)
                return constructor.callBy(arguments)
            }
        }
        ```
        
    
    ```kotlin
    public actual interface KCallable<out R> : KAnnotatedElement {
        public actual val name: String
        public val parameters: List<KParameter>
        public val returnType: KType
        @SinceKotlin("1.1")
        public val typeParameters: List<KTypeParameter>
    
        public fun call(vararg args: Any?): R
    
        public fun callBy(args: Map<KParameter, Any?>): R
    
        @SinceKotlin("1.1")
        public val visibility: KVisibility?
        @SinceKotlin("1.1")
        public val isFinal: Boolean
        @SinceKotlin("1.1")
        public val isOpen: Boolean
        @SinceKotlin("1.1")
        public val isAbstract: Boolean
        @SinceKotlin("1.3")
        public val isSuspend: Boolean
    }
    ```
    

- `KParameter.isOptional`: `true` → 파라미터에 디폴트 값이 있다.
    
    ```kotlin
    fun createInstance(arguments: Map<KParameter, Any?>): T {
        val firstParameter = arguments.keys.first()
        firstParameter.isOptional
    }
    ```
    
    ```kotlin
    public interface KParameter : KAnnotatedElement {
        public val index: Int
        public val name: String?
        public val type: KType
        public val kind: Kind
    
        public enum class Kind {
            INSTANCE,
            EXTENSION_RECEIVER,
            VALUE,
        }
    
        /**
         * `true` if this parameter is optional and can be omitted when making a call via [KCallable.callBy], or `false` otherwise.
         *
         * A parameter is optional in any of the two cases:
         * 1. The default value is provided at the declaration of this parameter.
         * 2. The parameter is declared in a member function and one of the corresponding parameters in the super functions is optional.
         */
        public val isOptional: Boolean
        @SinceKotlin("1.1")
        public val isVararg: Boolean
    }
    ```
    
- `KType.isMarkedNullable`: `true` → 파라미터가 널이 될 수 있는 값 (디폴트 파라미터 값으로 `null`을 사용한다)
    
    ```kotlin
    fun createInstance(arguments: Map<KParameter, Any?>): T {
        val firstParameter = arguments.keys.first()
        val type = firstParameter.type
        type.isMarkedNullable
    }
    ```
    
    ```kotlin
    public actual interface KType : KAnnotatedElement {
        @SinceKotlin("1.1")
        public actual val classifier: KClassifier?
        @SinceKotlin("1.1")
        public actual val arguments: List<KTypeProjection>
        public actual val isMarkedNullable: Boolean
    }
    ```
    

- 참고
    - [https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/10](https://incheol-jung.gitbook.io/docs/study/kotlin-in-action/10)
    - [https://github.com/3tudy/kotlin/blob/master/Kotlin in Action/chapter10/chapter10.md](https://github.com/3tudy/kotlin/blob/master/Kotlin%20in%20Action/chapter10/chapter10.md)
    - `annotation class`: [https://kotlinlang.org/docs/whatsnew13.html#nested-declarations-in-annotation-classes](https://kotlinlang.org/docs/whatsnew13.html#nested-declarations-in-annotation-classes)
