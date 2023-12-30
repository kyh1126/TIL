# 부록E. 코루틴과 Async/Await

## ****E.1 코루틴이란?****

---

- 코루틴: 컴퓨터 프로그램 구성 요소 중 하나로 비선점형(non-preemptive) 멀티태스킹을 수행하는 일반화한 서브루틴이다. 코루틴은 실행을 일시 중단하고 재개할 수 있는 여러 진입 지점을 허용한다.

👉 서로 협력해서 실행을 주고받으면서 작동하는 여러 서브루틴을 말한다.

- 대표적인 코루틴으로는 제네레이터를 예로 들 수 있다. 일반적으로 `yield` 키워드를 사용해 caller 쪽으로 제어를 넘겼다가 caller 가 코루틴을 호출하면 다시 `yield` 전까지 실행 후 제어를 caller 쪽으로 돌려준다.****

### 서브 루틴

---

- 여러 명령어를 모아 이름을 부여해서 반복 호출할 수 있게 정의한 프로그램 구성 요소. 쉽게 말해서, 함수.
- 서브루틴에 진입하기
    - 함수(또는 메소드)를 호출하면 그때마다 활성 레코드라는 것이 스택에 할당되면서 서브루틴 내부의 로컬 변수 등이 초기화된다.
    - `return`을 통해 서브루틴이 실행을 중단하고 **제어**를 호출한 쪽에게 돌려주는 지점을 여러개로 설정할 수도 있다.
    - 서브루틴에서 반환되고 나면 활성 레코드가 스택에서 사려져 실행 중이던 모든 상태를 잃어버린다.

### 비선점형 멀티태스킹

---

- 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 만들 수 없는 멀티태스킹을 말한다.

## ****E.2 코틀린의 코루틴 지원: 일반적인 코루틴****

---

- 코틀린은 특정 코루틴을 언어가 지원하는 형태가 아니라, 코루틴을 구현할 수 있는 기본 도구를 언어가 제공하는 형태다.

### E.2.1 여러 가지 코루틴

---

- 1️⃣ `kotlinx.coroutines.CoroutineScope.launch`: `launch`는 코루틴을 잡으로 반환하며, 만들어진 코루틴은 기본적으로 즉시 실행된다. 원하면 `launch`가 반환한 `Job`의 `cancel()`을 호출해 코루틴 실행을 중단시킬 수 있다.
    
    ```kotlin
    import kotlinx.coroutines.*
    import java.time.ZonedDateTime
    import java.time.temporal.ChronoUnit
    
    fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)
    
    fun log(msg:String) = println("${now()}:${Thread.currentThread()}:${msg}")
    
    fun launchInGlobalScope() {
        GlobalScope.launch {    // GlobalScope에서는 메인 스레드가 실행 중인 동안만 
                                // 코루틴 동작을 보장해준다.
            log("coroutine started.")
        }
    }
    
    fun main() {
        log("main() started.")
        launchInGlobalScope()   // main 함수와는 다른 thread에서 실행된다.
        log("launchInGlobalScope() executed")
        Thread.sleep(5000L)     // 이 딜레이(sleep)이 없으면 코루틴은 실행되지 않는다.
        log("main() terminated")
    }
    ```
    
- GlobalScope에서는 메인 스레드가 실행 중일 때만 코루틴의 동작을 보장한다.
- 하지만 메인 스레드가 멈추지 않고 그대로 프로그램 종료까지 이어지면 코루틴은 실행되지 않는다.
- 이를 방지하기 위해선 비동기적으로 `launch`를 실행하거나, `launch`가 모두 실행될 때까지 기다려야 한다.

- 2️⃣ `runBlocking()`: 코루틴의 실행이 끝날 때까지 현재 스레드를 블록시키는 함수. `runBlocking()`은 `CoroutineScope`의 확장 함수가 아닌 일반 함수이기 때문에 별도의 코루틴 스코프 객체 없이 사용 가능하다.
    
    ```kotlin
    import kotlinx.coroutines.*
    import java.time.ZonedDateTime
    import java.time.temporal.ChronoUnit
    
    fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)
    
    fun log(msg:String) = println("${now()}:${Thread.currentThread()}:${msg}")
    
    fun runBlockingExample() {
        runBlocking {
            launch {
                log("GlobalScope.launch started.")
            }
        }
    }
    
    fun main() {
        log("main() started.")
        runBlockingExample()
        log("runBlockingExample() executed")
        Thread.sleep(5000L)
        log("main() terminated")
    }
    ```
    
- 모든 실행이 메인 스레드에서 이루어진다.****

- 3️⃣ `kotlinx.coroutines.CoroutineScope.async`: `async`는 사실상 `launch`와 같은 일을 한다. 유일한 차이는 `launch`가 `Job`을 반환하는 반면 `async`는 `Deffered`를 반환한다는 점뿐이다. `Deffered`는 `Job`을 상속한 클래스이기 때문에 `launch` 대신 `async`를 사용할 수 있다.
    
    ```kotlin
    fun sumAll() {
        runBlocking {
            val d1 = async { delay(1000L); 1 }
            log("after async(d1)")
            val d2 = async { delay(2000L); 2 }
            log("after async(d2)")
            val d3 = async { delay(3000L); 3 }
            log("after async(d3)")
    
            log("1+2+3 = ${d1.await() + d2.await() + d3.await()}")
            log("after await all & add")
        }
    }
    ```
    
    - d1, d2, d3가 모두 계산되고 결과 6을 얻을 때까지 3초가 걸린다.(6초가 아니다)
    - 모든 함수들이 메인 스레드 안에서 실행되었다.
- `Deffered`의 타입 파라미터는 `Deffered` 코루틴이 계산을 하고 돌려주는 값의 타입이다.****
- `async`
    - 코드 블록을 비동기로 실행할 수 있다.
    - `async`가 반환하는 `Deffered`의 `await`을 사용해서 코루틴이 결과 값을 내놓을 때까지 기다렸다가 결과값을 얻어낼 수 있다.

### E.2.2 코루틴 컨텍스트와 디스패처

---

- `CoroutineScope`는 `CoroutineContext` 필드를 `launch` 등의 확장 함수 내부에서 사용하기 위한 매개체 역할만을 담당한다.
- `CoroutineContext`는 실제로 코루틴이 실행 중인 여러 작업(Job 타입)과 디스패처를 저장하는 일종의 맵이라 할 수 있다. 코틀린 런타임은 이 `CoroutineContext`를 사용해서 다음에 실행할 작업을 선정하고, 어떻게 스레드에 배정할지에 대한 방법을 결정한다.

### E.2.3 코루틴 빌더와 일시 중단 함수

---

- 코루틴 빌더: `launch`나 `async`, `runBlocking`은 모두 코루틴 빌더라고 불린다. 코루틴을 만들어주는 함수
    - `produce`: 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다. 이 함수는 `ReceiveChannel<>`을 반환한다. 그 채널로부터 메시지를 전달받아 사용할 수 있다.
    - `actor`: 정해진 채널로 메시지를 받아 처리하는 액터를 코루틴으로 만든다. 이 함수가 반환하는 `Sendchannel<>` 채널의 `send()` 메소드를 통해 액터에게 메시지를 보낼 수 있다. `delay()`와 `yield()`는 **일시 중단**suspending 함수라고 불린다.
- 일시 중단 함수들
    - `withContext`: 다른 컨텍스트로 코루틴을 전환한다.****
    - `withTimeout`: 코루틴이 정해진 시간 안에 실행되지 않으면 예외를 발생시키게 한다.****
    - `withTimeoutOrNull`: 코루틴이 정해진 시간 안에 실행되지 않으면 `null`을 결과로 돌려준다.****
    - `awaitAll`: 모든 작업의 성공을 기다린다. 작업 중 어느 하나가 예외로 실패하면 `awaitAll`도 그 예외로 실패한다.****
    - `joinAll`: 모든 작업이 끝날 때까지 현재 작업을 일시 중단시킨다.****

## E.3 `suspend` 키워드와 코틀린의 일시 중단 함수 컴파일 방법

---

- 일시 중단 함수를 코루틴이나 일시 중단 함수가 아닌 함수에서 호출하는 것은 컴파일러 수준에서 금지된다. 코틀린은 일시 중단 함수를 만들 수 있도록 `suspend`라는 키워드를 제공한다.
- 함수 정의의 `fun` 앞에 `suspend`를 넣으면 일시 중단 함수를 만들 수 있다.
    
    ```kotlin
    suspend fun yieldThreeTimes() {
        ...
    }
    ```
    

### `suspend` 함수는 어떻게 작동하는 것인가?

---

- 일시 중단 함수안에서 `yield()`를 해야 하는 경우 필요한 동작들:
    - 코루틴에 진입할 때와 코루틴에서 나갈 때 코루틴이 실행 중이던 상태를 저장하고 복구하는 등의 작업을 할 수 있어야 한다. → CPS 변환과 상태 기계를 활용
    - 현재 실행 중이던 위치를 저장하고 다시 코루틴이 재개될 때 해당 위치부터 실행을 재개할 수 있어야 한다. → CPS 변환과 상태 기계를 활용
    - 다음에 어떤 코루틴을 실행할지 결정한다. → 코루틴 컨텍스트의 디스패처가 수행한다.
- CPS(continuation passing style) 변환: 프로그램의 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수(Continuation)로 뽑고, 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만드는 소스코드 변환기술이다.

## E.4 코루틴 빌더 만들기

---

### E.4.1 제네레이터 빌더 사용법

---

- 제네레이터 빌더가 있을 때
    
    ```kotlin
    fun idMaker() = generate<Int, Unit> {
    	var index = 0
    	while(index < 3)
    		yield(index++)
    }
    
    fun main(args: Array<String>) {
    	val gen = idMaker()
    	println(gen.next(Unit)) // 0
    	println(gen.next(Unit)) // 1
    	println(gen.next(Unit)) // 2
    	println(gen.next(Unit)) // null
    }
    ```
    

### E.4.2 제네레이터 빌더 구현

---

- next(T) 를 호출해 R 타입의 값을 돌려받아 처리할 수 있게 해주는 제너레이터
    
    ```kotlin
    interface Generator<out R, in T> {
    	fun next(param: T): R? // 제네레이터가 끝나면 null을 돌려주므로 ?가 붙음
    }
    ```
    
- generator 가 받는 블록 안에서 `yield`를 호출하고 싶을 경우
    - GeneratorBuilder 클래스
        
        ```kotlin
        @RestrictsSuspension // suspend 가 붙은 함수에만 이 클래스를 수신 객체로 지정할 수 있게 한다.
        interface GeneratorBuilder<in T, R> {
            suspend fun yield(value: T): R
            suspend fun yieldAll(generator: Generator<T, R>, param: R)
        }
        ```
        
    - 블록의 타입: `block: suspend GeneratorBuilder<T, R>.(R) -> Unit`
        
        ```kotlin
        fun <T, R> generate(block: suspend GeneratorBuilder<T, R>.(R) -> Unit): Generator<T, R> {
            // GeneratorCoroutine객체, Generator타입, GeneratorBuilder이기도하다
            val coroutine = GeneratorCoroutine<T, R>()
            // 새 suspend함수 만들기, R을 받고, block을 호출한다(GeneratorCoroutine과 R을 넘겨준다), 숨겨진 Continuation을 마지막 인자로 넘겨준다
            val initial: suspend (R) -> Unit = { result -> block(coroutine, result) }
            // 코루틴이 실행되면서 처리할 부분
            coroutine.nextStep = { param -> initial.startCoroutine(param, coroutine) }
            // 바로 실행하지 않음. 사용하는 위치에서 next()호출을 한다(제네레이터)
            return coroutine
        }
        ```
        
- GeneratorCoroutine
    
    ```kotlin
    internal class GeneratorCoroutine<T, R> : Generator<T, R>, GeneratorBuilder<T, R> {
        lateinit var nextStep: (R) -> Unit
        private var lastValue: T? = null
        private var lastException: Throwable? = null
    
        // Generator<T, R> 구현
    
        override fun next(param: R): T? {
            nextStep(param)
            lastException?.let { throw it }
            return lastValue
        }
    
        //GeneratorBuilder<T,R> 구현
    
        override suspend fun yield(value: T): R = suspendCoroutineUninterceptedOrReturn { cont ->
            lastValue = value
            nextStep = { cont.resume(it) }
            COROUTINE_SUSPENDED
        }
    
        override suspend fun yieldAll(generator: Generator<T, R>, param: R) {
            suspendCoroutineUninterceptedOrReturn sc@{ cont ->
                lastValue = generator.next(param)
                if (lastValue == null) return@sc Unit
                nextStep = { param ->
                    lastValue = generator.next(param)
                    if (lastValue == null) cont.resume(Unit)
                    COROUTINE_SUSPENDED
                }
            }
        }
    
        override val context: CoroutineContext
            get() = EmptyCoroutineContext
    
        override fun resumeWith(result: Result<Unit>) {
            result
                // 코루틴이 잘 돌았으면 next 에서 null 을 반환하게 nextValue 를 설정한다.
                .onSuccess { lastValue = null }
                // 코루틴에서 오류가 났으면 예외를 설정한다.
                .onFailure { lastException = it }
        }
    }
    ```
    
- 전체 코드가 실행되는 과정
    1. 메인 함수가 실행된다.
    2. 메인에서 idMaker()를 실행한다.
    3. idMaker()은 generate()를 실행한다
        1. Generate()는 GeneratorCoroutine을 만든다. 이 GeneratorCoroutine는 코루틴 이자 제네레이터이자 제네레이터 빌더 역할을 함께 수행한다.
        2. GeneratorCoroutine타입 객체인 coroutine의 nextStep을는 generate()에 넘어간 블록을 갖고 코루틴을 시작하는 코드를 넣는다.
        3. Coroutine을 반환한다.
    4. 이 시점에서는 실제로는 아무 코루틴도 만들어지거나 실행되지 않았다는 점에 유의하라.
    5. 메인 함수에서 제네레이터의 next()를 호출한다.
    6. 제네레이터의 next()는 nextStep()을 호출한다. 그 안에는 코루틴을 시작하는 코드가 들어있다. 코루틴이 시작되면 디스패처는 적절한 스레드를 선택(우리 generator는 현재 스래드를 그냥 계속 사용한다)하고 initial 에 저장된 suspend 람다에 들어가 있는 블록을 시작한다.

- 메인함수랑 ↔  제네레이터 코루틴 두 가지 루틴이 서로 제어를 주고받는 상태가 된다.
    
    
    | 메인함수 | 제네레이터 (코루틴) |
    | --- | --- |
    | 1) 첫 번째 next()호출
    2) nextSstep(0에 있는 startCoroutine실행 | (코루틴 시작됨) |
    | (일시 중단 상태) | 1. var index = 0 
       while(index<3)을 실행하면서 index가 3보다 작으므로     
        루프 안에 들어감
    2. yield(index++)에 yield(0) 실행
    3. yield안에서 nextValue에 0 지정
    4. yield안에서 nextStep을 지정(Continuation을 재개하는 코드가 들어감)
    5. yield가 COROUTINE_SUSPENDED를 반환함에 따라 런타임은 제어를 메인 함수에게 돌려줌 |
    | 3) nextValue에 저장된 0을 반화(첫 번째 next() 끝)하면 그 값을 출력함
    4) 두번째 next() 호출
    5) nextStep()에서 Continuation실행(코루틴의 Continuation이므로 코루틴으로 제어가 넘어감) | (일시 중단 상태) |
    | (일시 중단 상태) | 6) 코루틴의 Continuation이 실행됨
    7) 지난번 yield에서 저장한 Continuation의 다음 지점은 ++연산자에 의해 index를 증가시키는 연산임
    8) Index를 증가시키고 다시 while로 돌아가서 index값 검사
    9) Index가 1이므로 3보다 작아서 루프 내부로 들어가 yield실행 |
    | 양쪽 루틴이 같은 로직 반복 | 양쪽 루틴이 같은 로직 반복 |
    | (일시 중단 상태) | 6’) 코루틴의 Continuation실행됨
    8’) Index++에 의해 Index가 3이 됨에 따라 while로프에서 나옴
    10) 블록에서 return(return문 없지만 블록 끝에 도달)
    11) startCoroutine에서 지정한 Continuation이 호출됨
    12) 런타임은 GeneratorCoroutine의 resumeWith를 호출
    13) Continuation에서 오류가 발생하지 않았으므로 result.onSuccess에서 nextValue에 null 설정되고 코루틴 종료 |
    | next()에서 nextValue인 null을 반환하고 그 null값을 출력 |  |
    | 메인 함수 종료 |  |

- launch 구현
    
    ```kotlin
    fun launch(context: CoroutineContext = EmptyCoroutineContext, block: suspend () -> Unit) = 
    	block.startCoroutine(Continuation(context) { result ->
    			result.onFailure {exception ->
    				val currentThread = Thread.currentThread()
    				currentThread.uncaughtExceptionHandler(currentThread, exception)
    		}
    	}
    ```
    
    - 즉시 코루틴 실행
    - 익명 클래스를 사용해 Continuation클래스를 직접 만들어서 startCoroutine에 전달
    - 블록을 코루틴으로 실행하고 나면 아무 처리도 필요 없기 때문에 result의 onSuccess쪽에는 아무 콜백을 지정하지 않고, 예외로 실패한 경우에만 예외를 다시 던진다
    - result는 코루틴(launch에 전달한 블록)이 반환하는 값을 코틀린 런타임이 Continuation에 전달해 준다는 점에 유의하기
