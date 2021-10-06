# Hibernate5 Naming 전략

## Naming Strategy

### 1. Implicit Naming Strategy

---

- 명시적으로(`@Table(name = )`, `@Column(name = )`) 이름을 주지 않은 경우의 네이밍 전략
- default 전략: JPA 에 정의한 변수나 클래스명을 그대로 적용하는 것
- spring boot 기본 전략 - `application.yml`
    
    ```yaml
    spring.jpa.hibernate.naming.implicit-strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
    ```
    

### 2. Physical Naming Strategy

---

- Implicit Naming Strategy 나 명시적으로 이름을 준 엔티티, 필드 변경이 있을 때 유용한 전략이다. 항상 가장 마지막에 덮어써진다.
    - 예를 들어 엔티티의 일부 필드는 `@Column(name = )`로 명시적 이름을 줬고, 나머지는 Implicit Naming Strategy 가 적용된 상태일 경우,
    - 정책 변경으로 인해 모든 필드 네이밍 형태를 통일 시켜야하는 경우, Physical Naming Strategy 이 없다면 개발자는 모든 `@Column(name = )`과 Implicit Naming Strategy를 변경해줘야한다.
- Implicit Naming Strategy 을 준다고 하더라도 Physical Naming Strategy 로 덮어쓴다.
- default 전략: camel case를 underscore 형태로 변경
- spring boot 기본 전략 - `application.yml`
    
    ```yaml
    spring.jpa.hibernate.naming.physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
    ```
    
- 종류
    - `SpringPhysicalNamingStrategy`(default): camel case를 underscore 형태로 변경
    - `PhysicalNamingStrategyStandardImpl`: 변수 이름을 그대로 사용

- 참고
    - [https://prolog.techcourse.co.kr/posts/1182](https://prolog.techcourse.co.kr/posts/1182)