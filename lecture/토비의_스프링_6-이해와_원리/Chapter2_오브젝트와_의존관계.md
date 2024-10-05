# 2. 오브젝트와 의존관계

## 오브젝트와 의존관계

---

- 스프링의 관심은 오브젝트이다.

- 오브젝트: OOP, 객체, 클래스?
    - 클래스와 오브젝트 → 두 가지를 구분하는게 중요하다.
        - 클래스와 오브젝트는 다르다.
    - 자바의 오브젝트: 클래스의 인스턴스 또는 배열(Array)

- 의존관계(dependency)
    - 의존관계는 반드시 두 개 이상의 대상이 존재하고, 하나가 다른 하나를 사용, 호출, 생성, 인스턴스화, 전송 등을 할 때 의존관계에 있다고 이야가힌다.
    - 클래스 사이에 의존관계가 있을 때 의존 대상이 변경되면 이를 사용하는 클래스의 코드도 영향을 받는다.
    - 오브젝트 사이에 의존관계는 실행되는 런타임에 의존관계가 만들어지고 실행 기능에 영향을 받지만 클래스 레벨의 의존관계와는 다를 수 있다.
    - ex> A → B: A가 B에 의존한다.
        
        ![image.png](./image/2/image.png)
        
        - Client의 기능이 제대로 동작하려면 Supplier가 필요
        - Client가 Supplier를 사용, 호출, 생성, 인스턴스화, 전송
        - 클래스 레벨(코드 레벨)의 의존관계: Supplier가 변경되면 Client 코드가 영향을 받는다.

## 관심사의 분리

---

- 관심사는 동일한 이유로 변경되는 코드의 집합
- API를 이용해서 환율정보를 가져오고 JSON을 코드에 매핑하는 관심과, Payment를 준비하는 로직은 관심이 다르다.
    - 변경의 이유와 시점을 살펴보고 이를 분리한다.

### 메소드 추출(extract method) 리팩터링

---

- 환율정보를 가져오고 JSON을 코드에 매핑하는 관심을 분리해보자.
    
    ```java
    public class PaymentService {
        public Payment prepare(Long orderId, String currency, BigDecimal foreignCurrencyAmount) throws IOException {
            // 환율 가져오기
            BigDecimal exRate = getExRate(currency);
            BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
            LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
    
            return new Payment(orderId, currency, foreignCurrencyAmount, exRate, convertedAmount, validUntil);
        }
    
        private BigDecimal getExRate(String currency) throws IOException {
            URL url = new URL("https://open.er-api.com/v6/latest/" + currency);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String response = br.lines().collect(Collectors.joining());
            br.close();
    
            ObjectMapper mapper = new ObjectMapper();
            ExRateData data = mapper.readValue(response, ExRateData.class);
            BigDecimal exRate = data.rates().get("KRW");
            return exRate;
        }
    }
    ```
    
    ![image.png](./image/2/image%201.png)
    

### 리팩터링

---

- 꼭 필요하지 않은, 코드만 봐도 알 수 있는 것들에 대한 주석은 지운다.
- `PaymentService` 주석 삭제
    - `환율 가져오기`는 저 URL에서 뭘 가져오는지 알 수 없으므로 남기고 나머지 주석은 삭제한다.
        
        ```java
        ...
                // 금액 계산
                BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
        
                // 유효 시간 계산
                LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
        ...
        ```
        
    - 메소드 추출 리팩터링 후, 메서드명이 잘 나타내주므로 `환율 가져오기` 주석 삭제한다.

## 상속을 통한 확장

---

- Payment의 변경 없이 환율 정보를 가져오는 방법을 확장하게 만들려면 상속을 이용할 수 있다.
    - 어떻게 `Payment`를 준비할 것인가와 어떻게 환율을 가져올 것인가라는 관심사가 클래스 레벨에서 분리된다.
    - 환율 정보를 담은 오브젝트인 ExRate를 생성하는 책임을 서브 클래스에게 위임하는 방식이다.
        - PaymentService: 추상클래스로 만든다.
            
            ```java
            public abstract class PaymentService {
                public Payment prepare(Long orderId, String currency, BigDecimal foreignCurrencyAmount) throws IOException {
                    BigDecimal exRate = getExRate(currency);
                    BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
                    LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
            
                    return new Payment(orderId, currency, foreignCurrencyAmount, exRate, convertedAmount, validUntil);
                }
            
                abstract BigDecimal getExRate(String currency) throws IOException;
            }
            ```
            
        - WebApiExRatePaymentService: 상속 후 환율 가져오는 추상메소드를 구현한다.
            
            ```java
            public class WebApiExRatePaymentService extends PaymentService {
                @Override
                BigDecimal getExRate(String currency) throws IOException {
                    {
                        URL url = new URL("https://open.er-api.com/v6/latest/" + currency);
                        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                        BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
                        String response = br.lines().collect(Collectors.joining());
                        br.close();
            
                        ObjectMapper mapper = new ObjectMapper();
                        ExRateData data = mapper.readValue(response, ExRateData.class);
                        BigDecimal exRate = data.rates().get("KRW");
                        return exRate;
                    }
                }
            }
            ```
            
        - Client: main 메소드 가짐
            
            ```java
            public class Client {
                public static void main(String[] args) throws IOException {
                    PaymentService paymentService = new WebApiExRatePaymentService();
                    Payment payment = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
                    System.out.println(payment);
                }
            }
            ```
            
        
        → 재사용성이 좋아짐
        

- 객체지향 디자인 패턴에 나오는 팩토리 메소드 패턴을 이용한다.
    - 팩토리 메소드 패턴: 상속을 통해 기능을 확장하게 하는 패턴

- 만약 USD일 땐 고정환율을 제공하자 요구사항이 추가되면 새 PaymentService 추가하면 된다.
    - SimpleExRatePaymentService: 상속 후 환율 가져오는 추상메소드를 구현한다.
        
        ```java
        public class SimpleExRatePaymentService extends PaymentService {
            @Override
            BigDecimal getExRate(String currency) throws IOException {
                if (currency.equals("USD")) return BigDecimal.valueOf(1000);
        
                throw new IllegalArgumentException("지원되지 않는 통화입니다");
            }
        }
        ```
        
    
    😃 상속하는 클래스들은 보통 뒤에 부모 클래스명을 붙인다.
    
    ![image.png](./image/2/image%202.png)
    

- 상속을 통해서 관심사가 다른 코드를 분리해내고, 서로 독립적으로 변경 또는 확장할 수 있도록 만드는 것은 간단하면서도 매우 효과적인 방법이다.
- 하지만 이 방법은 상속을 사용했다는 것이 단점이다.
    - 자바는 다중 상속을 허용하지 않는다.
        - 따라서 또 다른 관심사를 분리할 경우 확장을 이용하기가 매우 어렵다.
    - 또, 상속은 상하위 클래스의 관계에 매우 밀접하다.
        - 상위 클래스의 변경에 따라 하위 클래스를 모두 변경해야 하는 상황이 생길 수도 있다.
    
    → 디자인 패턴에 일부 적용된 특별한 목적을 위해서 사용되는 경우가 아니라면, 상속을 통한 확장은 관심사의 분리에 사용하기엔 분명한 한계가 있다.
    

## 클래스의 분리

---

- 관심사에 따라 클래스를 분리해서 각각 독립적으로 구성할 수 있다.
- 결과적으로 클래스 레벨에 사용 의존관계가 만들어지기 때문에 강한 코드 수준의 결합이 생긴다. 실제로 사용할 클래스가 변경되면 이를 이용하는 쪽의 코드도 따라서 변경이 되어야 한다. 상속을 통한 확장처럼 유연한 변경도 불가능해진다.

- PaymentService: Provider 클래스를 통해 환율을 가져온다.
    
    ```java
    public class PaymentService {
        public Payment prepare(Long orderId, String currency, BigDecimal foreignCurrencyAmount) throws IOException {
            WebApiExRateProvider exRateProvider = new WebApiExRateProvider();
    
            BigDecimal exRate = exRateProvider.getWebExRate(currency);
            BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
            LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
    
            return new Payment(orderId, currency, foreignCurrencyAmount, exRate, convertedAmount, validUntil);
        }
    }
    ```
    
- WebApiExRateProvider
    
    ```java
    public class WebApiExRateProvider {
        BigDecimal getWebExRate(String currency) throws IOException {
            {
                URL url = new URL("https://open.er-api.com/v6/latest/" + currency);
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
                String response = br.lines().collect(Collectors.joining());
                br.close();
    
                ObjectMapper mapper = new ObjectMapper();
                ExRateData data = mapper.readValue(response, ExRateData.class);
                return data.rates().get("KRW");
            }
        }
    }
    ```
    

😃 더 구체적인 클래스명, 메소드명을 줄 수 있다.

![image.png](./image/2/image%203.png)

- 상속한 것이 아니기 때문에 사용하는 클래스의 메소드 이름과 구조도 제각각일 수 있다. 그래서 클래스가 변경되면 많은 코드가 따라서 변경되어야 한다.
    - 클래스가 다르다는 것을 제외하면 관심사의 분리가 잘 된 방법이 아니다.

## 인터페이스 도입

---

- 독립적인 인터페이스를 정의하고 PaymentService가 사용할 메소드 이름을 정해둔다. 이를 각 클래스가 구현하게 만들면 이를 사용하는 쪽에서 의존하는 클래스가 변경되더라도 사용하는 메소드 이름의 변경이 일어나지 않는다.

- PaymentService: 인터페이스 타입을 통해 환율을 가져온다.
    
    ```java
    public class PaymentService {
        private final ExRateProvider exRateProvider;
    
        public PaymentService() {
            exRateProvider = new WebApiExRateProvider();
        }
    
        public Payment prepare(Long orderId, String currency, BigDecimal foreignCurrencyAmount) throws IOException {
            BigDecimal exRate = exRateProvider.getExRate(currency);
            BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
            LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
    
            return new Payment(orderId, currency, foreignCurrencyAmount, exRate, convertedAmount, validUntil);
        }
    }
    ```
    
- ExRateProvider
    
    ```java
    public interface ExRateProvider {
        BigDecimal getExRate(String currency) throws IOException;
    }
    ```
    
- WebApiExRateProvider
    
    ```java
    public class WebApiExRateProvider implements ExRateProvider {
        @Override
        public BigDecimal getExRate(String currency) throws IOException {
            {
                URL url = new URL("https://open.er-api.com/v6/latest/" + currency);
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
                String response = br.lines().collect(Collectors.joining());
                br.close();
    
                ObjectMapper mapper = new ObjectMapper();
                ExRateData data = mapper.readValue(response, ExRateData.class);
                return data.rates().get("KRW");
            }
        }
    }
    ```
    

![image.png](./image/2/image%204.png)

- 하지만, 클래스의 인스턴스를 만드는 생성자를 호출하는 코드에는 클래스 이름이 등장하기 때문에 사용하는 환율 정보를 가져오는 클래스가 변경되면 PaymentService의 코드도 일부분이지만 따라서 변경되어야 한다.
- 여전히 상속을 통한 확장만큼의 유연성도 가지지 못한다.

## 관계설정 책임의 분리

---

- 앞에서 ExRateProvider 인터페이스를 사용하도록 작성했겠지만 구현 클래스에 대한 정보를 가지고 있다면 PaymentService는 여전히 ExRateProvider를 구현한 특정 클래스에 의존하는 코드가 된다.
- 자신이 어떤 클래스의 오브젝트를 사용할지를 결정한다면 관계설정 책임을 직접 가지고 있는 것이다.
- 이 관계설정 책임을 자신을 호출하는 앞의 오브젝트에게 넘길 수 있다. 이렇게 되면 코드 레벨의 의존관계에서 자유로워진다. 이후에는 오직 인터페이스에만 의존하는 코드가 되기 때문에 어떤 구현 클래스의 오브젝트를 사용하게 되더라도 PaymentService의 코드가 변경되지 않는다.

- PaymentService: 넘겨받은 ExRateProvider를 통해 환율을 가져온다.
    
    ```java
    public class PaymentService {
        private final ExRateProvider exRateProvider;
    
        public PaymentService(ExRateProvider exRateProvider) {
            this.exRateProvider = exRateProvider;
        }
    
        public Payment prepare(Long orderId, String currency, BigDecimal foreignCurrencyAmount) throws IOException {
            BigDecimal exRate = exRateProvider.getExRate(currency);
            BigDecimal convertedAmount = foreignCurrencyAmount.multiply(exRate);
            LocalDateTime validUntil = LocalDateTime.now().plusMinutes(30);
    
            return new Payment(orderId, currency, foreignCurrencyAmount, exRate, convertedAmount, validUntil);
        }
    }
    ```
    
- Client
    
    ```java
    public class Client {
        public static void main(String[] args) throws IOException {
            PaymentService paymentService = new PaymentService(new WebApiExRateProvider());
            Payment payment = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println(payment);
        }
    }
    ```
    

![image.png](./image/2/image%205.png)

- 관계설정 책임을 가진 앞의 클래스(Client)는 생성자를 통해서 어떤 클래스의 오브젝트를 사용할지 결정한 것을 전달해주면 된다.

## 오브젝트 팩토리

---

- Client는 클라이언트로서의 책임과 PaymentService와 ExRateProvider 오브젝트 사이의 관계설정 책임을 두 가지를 가지고 있다. 관심사의 분리가 필요하다.
- 클라이언트의 관계설정 책임을 가진 코드를 ObjectFactory라는 이름으로 분리한다. ObjectFactory는 사용할 클래스를 선정하고 오브젝트를 만들면서 의존관계가 있다면 이를 생성자에 전달해서 만드는 기능을 담당한다.

- ObjectFactory: 한 메소드에 관계 설정과 객체 생성 후 반환하는 관심사가 둘 다 들어있지 않게 구성한다.
    
    ```java
    public class ObjectFactory {
        public PaymentService paymentService() {
            return new PaymentService(exRateProvider());
        }
    
        private WebApiExRateProvider exRateProvider() {
            return new WebApiExRateProvider();
        }
    }
    ```
    
- Client
    
    ```java
    public class Client {
        public static void main(String[] args) throws IOException {
            ObjectFactory objectFactory = new ObjectFactory();
            PaymentService paymentService = objectFactory.paymentService();
    
            Payment payment = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println(payment);
        }
    }
    ```
    

![image.png](./image/2/image%206.png)

## 원칙과 패턴

---

- 객체지향 설계원칙과 객체지향 디자인 패턴

### 개방-폐쇄 원칙(Open-Closed Principle)

---

- 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.
- 클래스가 기능을 확장할 때 클래스의 코드는 변경되지 않아야 한다.

### 높은 응집도와 낮은 결합도(High Coherence and low coupling)

---

- 응집도가 높다는 것은 하나의 모듈이 하나의 책임 또는 관심사에 집중되어 있다는 뜻. 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다.
- 책임과 관심사가 다른 모듈과는 낮은 결합도, 즉 느슨하게 연결된 형태를 유지하는 것이 바람직하다.

### 전략 패턴(Strategy Pattern)

---

- 자신의 기능 맥락(Context)에서, 필요에 따라서 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라서 바꿔서 사용할 수 있게 하는 디자인 패턴
- `Collections.sort()`는 정렬에 사용할 전략 오브젝트를 전달 받아서 사용한다.

### 제어의 역전(Inversion of Control)

---

- 제어권 이전을 통한 제어관계 역전 - 프레임워크의 기본 동작 원리

## 스프링 컨테이너와 의존관계 주입

---

- 스프링의 BeanFactory가 앞에서 만든 ObjectFactory가 제공하던 기능을 대체한다. BeanFactory는 ObjectFactory의 구성 정보를 참고해서 동작하게 만든다.
    - 스프링 컨테이너는 빈(bean)이라고 불리는 애플리케이션을 구성하는 오브젝트를 관리하는 기능을 담당한다.

- ObjectFactory: `@Configuration` + `@Bean`으로 구성
    
    ```java
    @Configuration
    public class ObjectFactory {
        @Bean
        public PaymentService paymentService() {
            return new PaymentService(exRateProvider());
        }
    
        @Bean
        public ExRateProvider exRateProvider() {
            return new SimpleExRateProvider();
        }
    }
    ```
    
- Client
    
    ```java
    public class Client {
        public static void main(String[] args) throws IOException {
            BeanFactory beanFactory = new AnnotationConfigApplicationContext(ObjectFactory.class);
            PaymentService paymentService = beanFactory.getBean(PaymentService.class);
    
            Payment payment = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println(payment);
        }
    }
    ```
    

![image.png](./image/2/image%207.png)

### 의존관계 주입(Dependency Injection)

---

- IoC는 스프링의 동작원리를 정확하게 설명하기에는 너무 일반적인 프레임워크 동작원리를 설명하는 용어이다.
- 그래서 스프링과 같이 오브젝트의 의존관계에 대한 책임을 스프링과 같은 외부 오브젝트가 담당하도록 만드는 것을 설명하는, 의존관계 주입 패턴, 줄여서 DI라고 불리는 용어가 마틴 파울러에 의해서 제안되었고 스프링 개발자들 사이에서, 또 이 원칙을 따라서 프레임워크를 만들거나 개발 방식을 설명하는 다른 언어와 기술에서도 넓게 사용되고 있다.
- 스프링이 처음 등장했던 시기에는 IoC라는 용어를 주로 사용했기 때문에 이후 DI를 사용하면서도 IoC라는 용어도 같이 쓰이기도 한다. 스프링은 IoC/DI 컨테이너라는 식으로 설명하는 문서도 많이 있다.

### 컨테이너

---

- 애플리케이션을 구성하는 오브젝트를 만들어서 담아두고 필요할 때 사용하도록 제공하는 기능을 담당하는 것을 컨테이너라고 부른다. 보통 오브젝트를 보관하는 것 뿐 아니라 생명주기까지 담당한다. 스프링 컨테이너는 빈이라고 부르는 오브젝트를 생성하고 의존관계를 설정하는 것까지 담당한다.
- 스프링 컨테이너는 빈 오브젝트의 생명주기를 담당하는 기능도 제공한다.

## 구성정보를 가져오는 다른 방법

---

- `@Configuration`, `@Bean` 애노테이션이 붙은 구성정보 클래스와 메소드를 통해서 만들어질 오브젝트와 의존관계를 정의하는 코드는, 같은 구성정보를 제공받을 수 있다면 다양한 다른 방법으로 정의할 수 있다.
- 예전에 많이 사용하던 XML을 이용하는 방법과 `@Component` 애노테이션이 붙은 클래스를 모두 찾아보는 빈 스캐닝 방식과 생성자 파라미터를 보고 의존 빈 오브젝트를 선택하는데 필요한 타입 정보를 가져오는 방식을 사용할 수도 있다. 빈 정보를 스캐닝에 의해서 동적으로 만들어내는 경우에는 `@ComponentScan` 애노테이션이 사용된다.

- ObjectFactory: `@ComponentScan` 역할을 담당한다.
    
    ```java
    @Configuration
    @ComponentScan
    public class ObjectFactory {
        /*@Bean
        public PaymentService paymentService() {
            return new PaymentService(exRateProvider());
        }
    
        @Bean
        public ExRateProvider exRateProvider() {
            return new SimpleExRateProvider();
        }*/
    }
    ```
    
- SimpleExRateProvider / WebApiExRateProvider: 사용할 구현체에 `@Component`를 붙인다.

![image.png](./image/2/image%208.png)

- 실제로는 빈 스캐닝 방식과 `@Configuration`/`@Bean`을 가진 구성정보 클래스 두 가지 방식을 혼합해서 사용한다.

## 싱글톤 레지스트리

---

- 싱글톤 패턴은 다음과 같은 단점을 가진다.
    - `private` 생성자를 가지고 있기 때문에 상속할 수 없다.
    - 싱글톤은 테스트하기 힘들다.
    - 서버 환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
    - 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.

- 스프링은 하나의 오브젝트만 만들어져야 한다는 필요를 충족하면서도 싱글톤 패턴을 사용해서 만들었을 때의 단점을 가지지 않도록 컨테이너 레벨에서 하나의 오브젝트만 만들어지는 것을 보장하는 기능을 제공한다. 이렇게 싱글톤 오브젝트를 만들고 관리하는 스프링 컨테이너를 싱글톤을 등록하고 관리한다는 의미에서 싱글톤 레지스트리라고 부르기도 한다.
- 스프링의 빈이 생성되고 적용되는 범위를 빈의 스코프(scope)라고 부른다. 스프링은 기본적으로 빈 오브젝트가 싱글톤 스코프를 가지도록 한다. 필요에 따라 여러 개의 오브젝트가 만들어지도록 할 수도 있다.

## DI와 디자인 패턴 (1)

---

- 디자인 패턴을 구분하는 두 가지 방식이 있다.
- 하나는 사용 목적(purpose)이고 다른 하나는 스코프(scope)이다.
- 스코프에 의해서 분류하면 클래스 패턴과 오브젝트 패턴으로 나눌 수 있다.
- 클래스 패턴은 상속(Inheritance)을 이용해서 확장성을 가진 패턴으로 만들어지고, 오브젝트 패턴은 합성(Composition)을 이용한다. 대부분의 디자인 패턴은 오브젝트 패턴이다. 가능하면 오브젝트 합성을 상속보다 더 선호하라는 디자인 패턴의 기본 객체지향 원리를 따른 것이다.
    - 오브젝트 합성을 이용하는 디자인 패턴을 적용할 때 스프링의 의존관계 주입을 사용.
- 전략 패턴은 오브젝트 합성을 이용한다.

![image.png](./image/2/image%209.png)

- 환율 정보가 필요할 때 매번 Web API를 호출해야 할까?
    - 환율 정보가 필요한 기능 증가
    - 응답시간
    - 환율 변동 주기
    
    → 환율 정보 캐시(Cache)의 도입
    

- 데코레이터 패턴: 오브젝트에 부가적인 기능/책임을 동적으로 부여하는 디자인 패턴이다.
- WebApiExRateProvider에 캐시 기능을 추가
    
    ![image.png](./image/2/image%2010.png)
    

## DI와 디자인 패턴 (2)

---

- CachedExRateProvider
    
    ```java
    public class CachedExRateProvider implements ExRateProvider {
        private final ExRateProvider target;
    
        private BigDecimal cachedExRate;
        private LocalDateTime cachedExpiryTime;
    
        public CachedExRateProvider(ExRateProvider target) {
            this.target = target;
        }
    
        @Override
        public BigDecimal getExRate(String currency) throws IOException {
            if (cachedExRate == null || cachedExpiryTime.isBefore(LocalDateTime.now())) {
                cachedExRate = this.target.getExRate(currency);
                cachedExpiryTime = LocalDateTime.now().plusSeconds(3);
    
                System.out.println("Cache Updated");
            }
    
            return cachedExRate;
        }
    }
    ```
    
- Client
    
    ```java
    public class Client {
        public static void main(String[] args) throws IOException, InterruptedException {
            BeanFactory beanFactory = new AnnotationConfigApplicationContext(ObjectFactory.class);
            PaymentService paymentService = beanFactory.getBean(PaymentService.class);
    
            Payment payment1 = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println("Payment1:" + payment1);
    
            TimeUnit.SECONDS.sleep(1);
    
            Payment payment2 = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println("Payment2:" + payment2);
    
            TimeUnit.SECONDS.sleep(2);
    
            Payment payment3 = paymentService.prepare(100L, "USD", BigDecimal.valueOf(50.7));
            System.out.println("Payment3:" + payment3);
        }
    }
    ```
    
- ObjectFactory
    
    ```java
    @Configuration
    public class ObjectFactory {
        @Bean
        public PaymentService paymentService() {
            return new PaymentService(cachedExRateProvider());
        }
    
        @Bean
        public ExRateProvider cachedExRateProvider() {
            return new CachedExRateProvider(exRateProvider());
        }
    
        @Bean
        public ExRateProvider exRateProvider() {
            return new WebApiExRateProvider();
        }
    }
    ```
    

## 의존성 역전 원칙(DIP)

---

1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
2. 추상화는 구체적인 사항에 의존해서는 안 된다. 구체적인 사항은 추상화에 의존해야 한다.
- 🥲

![image.png](./image/2/image%2011.png)

![image.png](./image/2/image%2012.png)

👉

- DIP는 먼저 인터페이스를 통해서 추상화에 의존하도록 코드를 만들어야 한다.
- 그리고 인터페이스 소유권의 역전도 필요하다.

- Separatae Interface 패턴
    - 인터페이스와 그 구현을 별개의 패키지에 위치시키는 패턴이다.
    - 인터페이스를 이를 구현한 클래스와 같은 패키지가 아닌 이 인터페이스를 사용하는 클라이언트와 같은 패키지에 위치하게 한다. 만약 이를 사용하는 클래스가 여럿인 경우에는 별개의 패키지로 인터페이스를 구분해둘 수 있다.
- 😃

![image.png](./image/2/image%2013.png)

![image.png](./image/2/image%2014.png)
