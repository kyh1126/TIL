# Chapter 11. DSL 만들기

## 1. API 에서 DSL 로

---

- 깔끔한 API
    - 코드를 읽는 사람들이 어떤 일이 벌어질지 명확하게 이해할 수 있어야 한다.
    - 불필요한 구문이나 번잡한 준비 코드가 적고 코드가 간결해야 한다.

- 코틀린이 간결한 구문을 어떻게 지원하는가?
    
    
    | 일반 구문 | 간결한 구문 | 사용한 언어 특성 |
    | --- | --- | --- |
    | StringUtil.capitalizes(s) | s.capitalize() | 확장 함수 |
    | 1.to(“one”) | 1 to “one” | 중위 호출 |
    | set.add(2) | set += 2 | 연산자 오버로딩 |
    | map.get(“key”) | map[“key”] | get 메소드에 대한 관례 |
    | file.use({ f -> f.read() }) | file.use { it.read() } | 람다를 괄호 밖으로 빼내는 관례 |
    | sb.append(“yes”) | with (sb) { append(“yes”) append(“no”)} | 수신 객체 지정 람다 |
- 코틀린 DSL 도 온전히 컴파일 시점에 타입이 정해진다. 컴파일 시점 오류 감지, IDE 지원 등 모든 정적 타입 지정 언어의 장점을 코틀린 DSL 을 사용할 때도 누릴 수 있다.

### 1-1. 영역 특화 언어라는 개념

---

- 범용 프로그래밍 언어: 컴퓨터로 풀 수 있는 모든 문제를 충분히 풀 수 있는 기능을 제공
    - 명령적이다.
        - 어떤 연산을 완수하기 위해 필요한 각 단계를 순서대로 정확히 기술
- 영역 특화 언어: 특정 과업 또는 영역에 초점을 맞추고 그 영역에 필요하지 않은 기능을 없앰
    - 선언적이다.
        - 원하는 결과를 기술하기만 하고 그 결과를 달성하기 위해 필요한 세부 실행은 언어를 해석하는 엔진에 맡긴다.
    - 범용 언어로 만든 호스트 애플리케이션과 함께 조합하기가 어렵다. 자체 문법이 있기 때문에 다른 언어의 프로그램 안에 직접 포함시킬 수가 없다.

### 1-2. 내부 DSL

---

- 외부 DSL: 독립적인 문법 구조를 가졌다.
    
    ```sql
    // SQL을 이용한 쿼리문 작성
    SELECT Country.name, COUNT(Customer.id)
      FROM Country
      JOIN Customer ON Country.id = Customer.country_id
     GROUP BY Country.name
     ORDER BY COUNT(Customer.id) DESC
     LIMIT 1
    ```
    
- 내부 DSL: 범용 언어로 작성된 프로그램의 일부. 범용 언어와 동일한 문법을 사용한다.
    
    → 내부 DSL 은 완전히 다른 언어가 아니라 DSL 의 핵심 장점을 유지하면서 주 언어를 특별한 방법으로 사용하는 것이다.
    
    ```kotlin
    // 코틀린과 익스포즈드 프레임워크를 사용해 같은 쿼리 작성
    // SQL 질의가 돌려주는 결과 집합을 코틀린 객체로 변환하기 위해 노력할 필요가 없다.
    (Country join Customer)
        .slice(Coutry.name, Count(Customer.id))
        .selectAll()
        .groupBy(Country.name)
        .orderBy(Count(Customer.id), isAsc = false)
        .limit(1)
    ```
    

### 1-3. DSL 의 구조

---

- DSL 과 일반 API 를 구분하는 방법
    - DSL 은 구조 또는 문법이 존재한다.
    - DSL 에서는 여러 함수 호출을 조합해서 연산을 만들며, 타입 검사기는 여러 함수 호출이 바르게 조합됐는지를 검사한다.
    - 같은 문맥을 함수 호출 시마다 반복하지 않고도 재사용할 수 있다.
        
        ```kotlin
        // gradle 에서 디펜던시 설정 -> 람다 중첩을 이용해 구조를 만듦
        dependencies {
            compile("junit:junit:4.11")
            compile("com.google.inject:guice:4.1.0")
        }
        
        // VS
        
        // 일반 명령-질의 API를 통해 디펜던시 설정 -> 코드에 중복이 많다
        project.dependencies.add("compile", "junit:junit:4.11")
        project.dependencies.add("compile", "com.google.inject:guice:4.1.0")
        ```
        

### 1-4. 내부 DSL 로 HTML 만들기

---

- `kotlinx.html` 라이브러리 ([https://github.com/Kotlin/kotlinx.html](https://github.com/Kotlin/kotlinx.html))
    
    ```kotlin
    // 칸(cell)이 하나인 표를 만드는 코드
    fun createSimpleTable() = createHtml().
      table {
        tr {
          td { +"cell" }
        }
      }
    ```
    
- 코틀린 코드로 HTML 을 만들려는 이유
    - 코틀린 버전은 타입 안정성을 보장한다.
        - `td`는 `tr` 안에서만 사용할 수 있다. 그렇게 하지 않으면 컴파일이 되지 않는다.
    - 코틀린 코드를 원하는대로 사용할 수 있다.
        
        ```kotlin
        // 표를 정의하면서 동적으로(맵에 들어있는 원소에 따라) 표의 칸을 생성할 수 있다.
        fun createAnotherTable() = createHTML().table {
            val numbers = mapOf(1 to "one", 2 to two)
            for ((num, string) in numbers) {
              tr {
                td { +"$num" }
                td { +string }
              }
            }
        }
        ```
        

## 2. 구조화된 API 구축: DSL 에서 수신 객체 지정 DSL 사용

---

### 2-1. 수신 객체 지정 람다와 확장 함수 타입

---

- 1️⃣ 일반 람다를 받는 `buildString`: 한 `StringBuilder` 객체에 여러 내용을 추가할 수 있다.
    
    ```kotlin
    fun buildString(builderAction: (StringBuilder) -> Unit): String { // 함수 타입인 파라미터 정의
        val sb = StringBuilder()
        builderAction(sb) // 람다 인자로 StringBuilder 인스턴스를 넘긴다.
        return sb.toString()
    }
    
    fun main() {
        val s = buildString {
            it.append("Hello, ") // it 은 StringBuilder 인스턴스를 가리킨다.
            it.append("World!")
        }
        println(s) // Hello, World!
    }
    ```
    
- 2️⃣ 수신 객체 지정 람다: 메소드 이름 앞에 `it.`을 일일이 넣지 않고 더 간단하게 호출
    - 람다의 인자 중 하나에게 수신 객체라는 상태를 부여하면 이름과 마침표를 명시하지 않아도 그 인자의 멤버를 바로 사용할 수 있다.
        - 클래스 멤버 안에서 보통 그렇듯이 모호한 경우가 아니라면 `this.`를 명시할 필요가 없다.
        
        ![11.1 수신 객체 타입이 `String`이며 파라미터로 두 `Int`를 받고 `Unit`을 반환하는 확장 함수 타입 정의](./image/11/Untitled.png)
        
        11.1 수신 객체 타입이 `String`이며 파라미터로 두 `Int`를 받고 `Unit`을 반환하는 확장 함수 타입 정의
        
    
    ```kotlin
    // 수신 객체 지정 람다를 사용해 buildString() 정의
    fun buildString2(builderAction: StringBuilder.() -> Unit): String { // 수신 객체가 있는 함수 타입(확장 함수 타입)의 파라미터 선언
      val sb = StringBuilder()
      sb.builderAction() // StringBuilder 인스턴스를 수신 객체로 넘김.
      return sb.toString()
    }
    
    fun main() {
        val s2 = buildString2 {
            this.append("Hello, ") // "this" 키워드는 StringBuilder 인스턴스를 가리킨다.
            append("World!")  // this 를 생략해도 묵시적으로 StringBuilder 인스턴스가 수신 객체로 취급된다.
        }
        println(s) // Hello, World!
    }
    ```
    
    ![11.2 buildString 함수(수신 객체 지정 람다)의 인자는 확장 함수 타입(builderAction)의 파라미터와 대응한다. 호출된 람다 본문 안에서는 수신 객체(sb)가 묵시적 수신 객체(`this`)가 된다.](./image/11/Untitled%201.png)
    
    11.2 buildString 함수(수신 객체 지정 람다)의 인자는 확장 함수 타입(builderAction)의 파라미터와 대응한다. 호출된 람다 본문 안에서는 수신 객체(sb)가 묵시적 수신 객체(`this`)가 된다.
    
- 3️⃣ 확장 함수 타입의 변수를 정의: 변수를 마치 확장 함수처럼 호출하거나 수신 객체 지정 람다를 요구하는 함수에게 인자로 넘길 수 있다.
    
    ```kotlin
    val appendExc1: StringBuilder.() -> Unit = { this.append("!") }
    
    fun main() {
        val stringBuilder = StringBuilder("Hi")
        stringBuilder.appendExc1()
        println(stringBuilder) // Hi!
        println(buildString2(appendExc1)) // !
    }
    ```
    

- 기본적으로 `apply`와 `with`는 모두 자신이 제공받은 수신 객체로 확장 함수 타입의 람다를 호출한다.
    - `apply`: 수신 객체 타입에 대한 확장 함수로 선언됐기 때문에 수신 객체의 메소드처럼 불리며, 수신 객체를 묵시적 인자(`this`)로 받는다. 수신 객체를 다시 반환한다.
        
        ```kotlin
        inline fun <T> T.apply(block: T.() -> Unit): T {
            block() // apply 안에 들어간 T람다가 T의 확장함수가 된다.
            return this
        }
        ```
        
    - `with`: 수신 객체를 첫 번째 파라미터로 받는다. 람다를 호출해 얻은 결과를 반환한다.
        
        ```kotlin
        inline fun <T, R> with(receiver: T, block: T.() -> R): R =
            recevier.block() // 람다를 호출해 얻은 결과를 반환한다.
        ```
        

### 2-2. 수신 객체 지정 람다를 HTML 빌더 안에서 사용

---

- HTML 빌더: HTML 을 만들기 위한 코틀린 DSL
    
    ```kotlin
    fun createSimpleTable() = createHTML().
      table { 
        tr { 
          td { +"cell" }
        }
      }
    ```
    
    ```kotlin
    open class Tag(val name: String) {
        private val children = mutableListOf<Tag>()    // 모든 중첩 태그를 저장
        protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
            child.init()    //  자식 태그 초기화
            children.add(child)   // 자식 태그에 대한 참조 저장
        }
    
        override fun toString() =
            "<$name>${children.joinToString("")}</$name>"
    }
    
    fun table(init: TABLE.() -> Unit) = TABLE().apply(init)
    
    class TABLE : Tag("table") {
        fun tr(init: TR.() -> Unit) = doInit(TR(), init)     // TR 태그 인스턴스를 새로 만들고 초기환 후 TABLE 태그의 자식으로 등록
    }
    
    class TR : Tag("tr") {
        fun td(init: TD.() -> Unit) = doInit(TD(), init) // // TD 태그의 새 인스턴스를 만들어서 TR 태그의 자식으로 등록
    }
    
    class TD : Tag("td")
    
    fun createTable() =
        table {
            tr {
                td {
                }
            }
        }
    
    fun main() {
        println(createTable()) // <table><tr><td></td></tr></table>
    }
    ```
    

### 2-3. 코틀린 빌더: 추상화와 재사용을 가능하게 하는 도구

---

- 내부 DSL 을 사용하면 일반 코드와 마찬가지로 반복되는 내부 DSL 코드 조각을 새 함수로 묶어서 재사용할 수 있다.
- [부트스트랩 라이브러리](https://getbootstrap.com/) 에서 제공하는 드롭다운 메뉴가 있는 HTML 페이지를 코틀린 빌더를 사용해 생성
    - 부트스트랩 라이브러리 버전
        
        ```html
        <div class="dropdown">
          <button class="btn dropdown-toggle">
            Dropdown
            <span class="caret"></span>
          </button>
          <ul class="dropdown-menu">
            <li><a href="#">Action</a></li>
            <li><a href="#">Another action</a></li>
            <li role="separator" class="divider"></li>
            <li class="dropdown-header">Header</li>
            <li><a href="#">Separated link</a></li>
          </ul>
        </div>
        ```
        
    - `kotlin.html`(코틀린 빌더) 버전
        
        ```kotlin
        fun buildDropdown() = createHTML().div(classes = "dropdown") {
            button(classes = "btn dropdown-toggle") {
                +"Dropdown"
                span(classes = "caret")
            }
            ul(classes = "dropdown-menu") {
                li { a("#") { +"Action" } }
                li { a("#") { +"Another action" } }
                li { role = "separator"; classes = setOf("divider") }
                li { classes = setOf("dropdown-header"); +"Header" }
                li { a("#") { +"Separated link" } }
            }
        }
        ```
        
    - 반복되는 로직을 별도의 함수로 분리 → 코드를 더 읽기 쉽게
        - div, button 은 모두 일반 함수
        
        ```kotlin
        fun dropdownExample() = createHTML().dropdown {
          dropdownButton { +"Dropdown" }
          dropdownMenu {
             item("#", "Action")
             item("#", "Another action")
             divider()
             dropdownHeader("Header")
             item("#", "Separated link")
          }
        }
        ```
        

## 3. `invoke` 관례를 사용한 더 유연한 블록 중첩

---

- `invoke` 관례를 사용하면 객체를 함수처럼 호출할 수 있다.

### 3-1. `invoke` 관례: 함수처럼 호출할 수 있는 객체

---

- 관례: 특별한 이름이 붙은 함수를 일반 메소드 호출 구문으로 호출하지 않고 더 간단한 다른 구문으로 호출할 수 있게 지원하는 기능 ex> 객체에 대해 인덱스 연산을 사용할 수 있게 해주는 `get`, `invoke`
- `invoke` 관례
    - `get`과 달리 각괄호 대신 괄호를 사용한다.
    - `operator` 변경자가 붙은 `invoke` 메소드 정의가 들어있는 클래스의 객체를 함수처럼 호출할 수 있다.
        
        ```kotlin
        class Greeter(val greeting: String) {
            operator fun invoke(name: String) {
                println("$greeting, $name!")
            }
        }
        
        fun main() {
            val bavarianGreeter = Greeter("Servus")
            // Greeter 인스턴스를 함수처럼 호출한다.
            // 내부적으로 bavarianGreeter.invoke("Dmitry") 으로 컴파일된다.
            bavarianGreeter("Dmitry") // Servus, Dmitry!
        
            Greeter("Servus")("Dmitry") // 동일 결과가 나온다.
        }
        ```
        
        - 디컴파일
            
            ```java
            import kotlin.Metadata;
            
            public final class Greeter {
               @NotNull
               private final String greeting;
            
               public final void invoke(@NotNull String name) {
                  Intrinsics.checkNotNullParameter(name, "name");
                  String var2 = this.greeting + ", " + name + '!';
                  System.out.println(var2);
               }
            
               @NotNull
               public final String getGreeting() {
                  return this.greeting;
               }
            
               public Greeter(@NotNull String greeting) {
                  Intrinsics.checkNotNullParameter(greeting, "greeting");
                  super();
                  this.greeting = greeting;
               }
            }
            
            public final class InvokeEx3Kt {
               public static final void main() {
                  Greeter bavarianGreeter = new Greeter("Servus");
                  bavarianGreeter.invoke("Dmitry");
               }
            
               // $FF: synthetic method
               public static void main(String[] var0) {
                  main();
               }
            }
            ```
            
    - 원하는 대로 파라미터 개수나 타입을 지정할 수 있다.
    - 여러 파라미터 타입을 지원하기 위해 `invoke`를 오버로딩할 수도 있다.

### 3-2. `invoke` 관례와 함수형 타입

---

- 인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스(`Function1` 등)를 구현하는 클래스로 컴파일된다.
    
    ```kotlin
    interface Function2<in P1, in P2, out R> {
        operator fun invoke(p1: P1, p2: P2): R
    }
    ```
    
- 람다를 함수처럼 호출하면 이 관례에 따라 `invoke` 메소드 호출로 변환된다.
    
    ```kotlin
    data class Issue(
        val id: String,
        val project: String,
        val type: String,
        val priority: String,
        val description: String
    )
    
    class ImportantIssuesPredicate(val project: String) : (Issue) -> Boolean {  // 함수타입을 부모 클래스로 사용한다.
        override fun invoke(issue: Issue): Boolean { // invoke 메소드를 구현한다.
            return issue.project == project && issue.isImportant()
        }
    
        private fun Issue.isImportant(): Boolean {
            return type == "Bug" &&
                (priority == "Major" || priority == "Critical")
        }
    }
    
    fun main() {
        val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
        val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal", "Intention: convert several calls on the same receiver to with/apply")
    
        val predicate = ImportantIssuesPredicate("IDEA")
        for (issue in listOf(i1, i2).filter(predicate)) { // 술어를 filter() 에게 넘긴다.
            println(issue.id) // IDEA-154446
        }
    }
    ```
    
    - `filter`는 `T`타입 element 를 입력으로 받아 `Boolean`을 반환하는 함수형 변수를 입력으로 받고, `Boolean`이 `true` 인 경우 반환할 새 list 에 해당 값을 포함시킨다.
        
        ```kotlin
        public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
            return filterTo(ArrayList<T>(), predicate)
        }
        
        public inline fun <T, C : MutableCollection<in T>> Iterable<T>.filterTo(destination: C, predicate: (T) -> Boolean): C {
            for (element in this) if (predicate(element)) destination.add(element)
            return destination
        }
        ```
        

- 람다를 여러 메소드로 나누고 각 메소드에 뜻을 명확히 알 수 있는 이름을 붙이고 싶다.
    
    👉 람다를 함수 타입 인터페이스를 구현하는 클래스로 변환하고 그 클래스의 `invoke` 메소드를 오버라이드
    
    - 장점: 람다 본문에서 따로 분리해낸 메소드가 영향을 끼치는 영역을 최소화할 수 있다.

### 3-3. DSL 의 `invoke` 관례: 그레이들에서 의존관계 정의

---

- 중첩된 블록 구조를 허용하는 한편 넓게 펼쳐진 형태의 함수 호출 구조도 함께 제공하는 API
    
    ```kotlin
    dependencies.compile("junit:junit:4.11")
    dependencies {
        compile("junit:junit:4.11")
    }
    ```
    

- `dependencies`는 `DependencyHandler` 클래스의 인스턴스
- `DependencyHandler` 구현 👉 `invoke` 안에서 `DependencyHandler` 타입의 객체를 따로 명시하지 않고 `compile()`을 호출할 수 있다.
    
    ```kotlin
    class DependencyHandler {
        // 일반적인 명령형 API 정의
        fun compile(coordinate: String) { // 일반적인 명령형 API를 정의
            println("Added dependency on $coordinate")
        }
    
        // invoke를 정의해 DSL 스타일 API를 제공
        operator fun invoke(body: DependencyHandler.() -> Unit) { // invoke 를 정의해 DSL 스타일 API 를 제공
            body() // == this.body()  
        }
    }
    
    fun main() {
        val dependencies = DependencyHandler()
        dependencies.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0") // 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0
    
        dependencies {
            compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
        } // 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0
    }
    ```
    
    - 두 번째 호출은 결과적으로 다음과 같이 변환된다.
        
        ```kotlin
        dependencies.invoke({
            this.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
        })
        ```
        
    - `dependencies`를 함수처럼 호출하면서 람다를 인자로 넘긴다.
    - 람다의 타입은 확장 함수 타입(수신 객체를 지정한 함수 타입)이며, 지정한 수신 객체 타입은 `DpendencyHandler`다.
    - `invoke` 메소드는 이 수신 객체 지정 람다를 호출한다.
    - `invoke`가 `DependencyHandler`의 메소드이므로 이 메소드 내부에서 묵시적 수신 객체 `this`는 `DependencyHandler` 객체다.

## 4. 실전 코틀린 DSL

---

### 4-1. 중위 호출 연쇄: 테스트 프레임워크의 `should`

---

- 내부 DSL 의 핵심 특징: 깔끔한 구문
- DSL 을 깔끔하게 만들려면 코드에 쓰이는 기호의 수를 줄여야 한다.

- 코틀린테스트 DSL 에서 중위 호출 활용
    
    ```kotlin
    s should startWith("kot")
    ```
    
    ```kotlin
    infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this) // infix : 두개의 변수 가운데 오는 함수, this == T
    ```
    
    ```kotlin
    // Matcher 선언
    interface Matcher<T> {
        fun test(value: T)
    }
    
    class startWith(val prefix: String) : Matcher<String> { // 클래스의 첫 글자가 대문자가 아닌 케이스 (dsl 에서는 종종 명명규칙을 벗어나야 할 때가 있다)
        override fun test(value: String) {
            if (!value.startsWith(prefix)) {
                throw AssertionError("String $value does not start with $prefix")
            }
        }
    }
    
    fun main() {
        val s = "Apple"
        s should startWith("A")
        s should startWith("B") // Exception in thread "main" java.lang.AssertionError: String Apple does not start with B
    }
    ```
    
    👉
    
    ```kotlin
    fun main() {
        "kotlin" should start with "kot"
    }
    ```
    
    ```kotlin
    // 중위호출과 object로 정의한 싱글턴 객체 인스턴스를 조합하면 dsl 에 복잡한 문법을 도입할 수 있다.
    object start // 함수에 데이터를 넘기기 위해서가 아니라 dsl 문법을 정의하기 위해 사용
    
    // start를 인자로 넘겨서 결과로 startWrapper 인스턴스를 받을 수 있다.
    infix fun String.should(x: start): StartWrapper = StartWrapper(this)
    
    // startWrapper 클래스에는 검사를 실행하기 위해 인자로 받는 with 라는 멤버를 가진다.
    class StartWrapper(val value: String) {
        infix fun with(prefix: String) = if (!value.startsWith(prefix)) {
            throw AssertionError("String $value does not start with $prefix")
        } else Unit
    
    }
    ```
    

### 4-2. 원시 타입에 대한 확장 함수 정의: 날짜 처리

---

- 다음과 같은 API 를 구현해보자
    
    ```kotlin
    val yesterday = 1.days.ago
    val tomorrow = 1.days.fromNow
    ```
    
- 자바 8의 `java.time` API 와 코틀린을 사용
    
    ```kotlin
    import java.time.Period
    import java.time.LocalDate
    
    // days는 Int 타입의 확장 프로퍼티
    val Int.days: Period 
        get() = Period.ofDays(this) // this는 상수의 값을 가리킴
    
    // LocalDate 클래스에는 코틀린의 - 연산자 관례와 일치하는 인자가 하나뿐인 minus 메소드가 들어가 있음 (public LocalDate minus(TemporalAmount var1))
    // 따라서 -, + 는 코틀린에서 제공하는 확장함수가 아닌 LocalData라는 JDK 클래스에 있는 관례에 의해 minus, plus 메소드가 호출되는 것
    val Period.ago: LocalDate
        get() = LocalDate.now() - this
    
    val Period.fromNow: LocalDate
        get() = LocalDate.now() + this
    
    fun main() {
        println(1.days.ago) // 2022-04-08
        println(1.days.fromNow) // 2022-04-10
    }
    ```
    

### 4-3. 멤버 확장 함수: SQL 을 위한 내부 DSL

---

- 클래스 안에서 확장 함수와 확장 프로퍼티를 선언하는 것
    - 정의한 확장 함수나 확장 프로퍼티는 그들이 선언된 클래스의 멤버인 동시에 그들이 확장하는 다른 타입의 멤버이기도 하다. → 멤버 확장
    - 멤버 확장으로 정의하는 이유: 메소드가 적용되는 범위를 제한하기 위함

- 익스포즈드 프레임워크에서 SQL 로 테이블을 다루기
    
    ```kotlin
    // 코틀린 익스포즈드에서 테이블 선언
    object Country : Table() {
        val id = integer("id").autoIncrement().primaryKey()
        val name = varchar("name", 50)
    }
    
    // Table 밖에서는 이 함수들을 호출할 수 없다.
    class Table {
        fun integer(name: String): Column<Int>
        fun varchar(name: String, length: Int): Column<String>
        fun <T> Column<T>.primaryKey(): Column<T>
        fun Column<Int>.autoIncrement(): Column<Int> // 수신객체타입을 Int 로 제한
        //...
    }
    ```
    
- 각 칼럼의 속성을 지정하는 방법 👉 멤버 확장
    
    ```kotlin
    val id = integer("id").autoIncrement().primaryKey()
    
    class Table {
        fun <T> Column<T>.primaryKey(): Column<T> // 이 칼럼을 테이블의 기본 키(PK)로 지정한다.
        fun Column<Int>.autoIncrement(): Column<Int> // 숫자 타입의 칼럼만 자동 증가 칼럼으로 설정할 수 있다.
        // ...
    }
    ```
    
    - `Column<Int>.autoIncrement()`
        - 확장 함수의 다른 멋진 속성 → 수신 객체 타입을 제한하는 기능

### 4-4. 안코: 안드로이드 UI 를 동적으로 생성하기

---

- 반복되는 로직을 추출해서 재활용할 수 있다.
    
    ```kotlin
    fun Activity.showAreYouSureAlert(process: () -> Unit) {
        alert(title = "Are you sure?",
            message = "Are you really sure?") {
            positiveButton("Yes") { process() }
            negativeButton("No") { cancel() }
        }
    }
    ```
    
    ```kotlin
    fun Context.alert(
        message: String,
        title: String,
        init: AlertDialogBuilder.() -> Unit
    )
    
    class AlertDialogBuilder {
        fun positiveButton(text: String, callback: DialogInterface.() -> Unit)
        fun negativeButton(text: String, callback: DialogInterface.() -> Unit)
        // ...
    }
    ```
    
    ```kotlin
    verticalLayout {
        val email = editText {
            hint = "Email"
        }
        val password = editText {
            hint = "Password"
            transformationMethod = PasswordTransformationMethod.getInstance()
        }
        button("Log In") {
            onClick {
                logIn(email.text, password.text)
            }
        }
    }
    ```
    

- 참고
    - [https://worker-investor.tistory.com/64](https://worker-investor.tistory.com/64)
