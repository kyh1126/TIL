# Profiles 란?

- 레퍼런스: [https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/features.html#features.profiles](https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/features.html#features.profiles)

## Profiles

---

- Spring Profiles는 애플리케이션 configuration 의 일부를 분리하고 특정 환경에서만 사용할 수 있도록 하는 방법을 제공한다.
- `@Component`, `@Configuration`, `@ConfigurationProperties`에 `@Profile`로 로드될 때 제한을 줄 수 있다.

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {
    // ...
}
```

- `spring.profiles.active` `Environment` 속성: 활성시킬 profile 들을 지정할 수 있다.
    - `application.properties`

        ```yaml
        spring.profiles.active=dev,hsqldb
        ```

    - `application.yml`

        ```yaml
        spring:
          profiles:
            active: "dev,hsqldb"
        ```

### 여러 Profiles active 하기

---

- 스프링 프레임워크 v5.1 이상은 더 복잡한 프로필을 문자열 표현식로 지정하는 추가 기능을 제공한다.

    ```java
    @Profile("test", "local")
    @Bean("amazonDynamoDB")
    @Primary
    fun localAmazonDynamoDB(): AmazonDynamoDB {
        log.info("Start Local Amazon DynamoDB Client")
        ...
    }
    ```

- 여러 profiles 가 아닌 경우

    ```java
    @Profile("!test & !local")
    @Bean
    @Primary
    fun amazonDynamoDB(): AmazonDynamoDB {
        log.info("Start AWS Amazon DynamoDB Client")
        ...
    }
    ```


## Config 파일 processing

### 스프링 부트 2.4 이전

---

- YAML 에서는 구분자(`---`)를 이용해서 문서를 구분짓는다.
- 프로파일을 선언하지 않으면, `application.yml` 을 비롯해서 `default` 프로파일을 사용한다.
    - 구분자(`---`) 로 구분되는 첫번째 문서

- `application-jenny.yml`

    ```yaml
    jenny:
      root-uri: https://github.com/kyh1126

    ---
    spring.profiles: develop
    jenny:
      root-uri: https://github.com/kyh1126/dev

    ---
    spring.profiles: beta
    jenny:
      root-uri: https://github.com/kyh1126/beta
    ```

- `application.yml`: 위의 YAML 파일 같이 읽어오기

    ```yaml
    spring.profiles:
      include:
        - jenny
    ```

### 스프링 부트 2.4 이상

---

- `spring.profiles` 속성 Deprecated → `spring.config.activate.on-profile`를 사용해야 한다.
- 구분자(`#---`)로 구분되는 제일 마지막 문서로 덮어써 버린다.
- 스프링 부트 2.4 부터 변경된 구성파일 처리방식이 마음에 들지 않는다면, `spring.config.use-legacy-processing: true` 선언하자.
- Profile Groups 등장
    - 레퍼런스: [https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/features.html#features.profiles.groups](https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/features.html#features.profiles.groups)

- `jenny.yml`

    ```yaml
    jenny:
      root-uri: https://github.com/kyh1126

    ---
    spring.config.activate.on-profile: jenny-develop
    jenny:
      root-uri: https://github.com/kyh1126/dev

    ---
    spring.config.activate.on-profile: jenny-beta
    jenny:
      root-uri: https://github.com/kyh1126/beta
    ```

- `application.yml`: 위의 YAML 파일 같이 읽어오기

    ```yaml
    spring:
      config:
        import: classpath:jenny.yml
      profiles:
        include:
          - jenny
        group:
          develop: jenny-develop
          beta: jenny-beta
    ```

- 참고
    - [http://honeymon.io/tech/2021/01/16/spring-boot-config-data-migration.html](http://honeymon.io/tech/2021/01/16/spring-boot-config-data-migration.html)
    - [https://stackoverflow.com/questions/38133808/spring-multiple-profiles-active](https://stackoverflow.com/questions/38133808/spring-multiple-profiles-active)


- [Notion link](https://jennyuni.notion.site/Profiles-f3eec612d1e34626963f996ae1fb3a11)
