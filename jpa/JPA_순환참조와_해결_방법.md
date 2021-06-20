# JPA 순환참조와 해결 방법

## 순환참조 발생

- JPA 의 `@OneToMany`는 디폴트로 맵핑된 데이터에 대해 `FetchType.LAZY` 를 사용하게 된다.
    - 예를 들어, User 라는 Entity 와 Account 라는 Entity 가 서로 양방향 참조 (1:N)를 하고 있다고 해보자.

        ```java
        public class User {
            @Id
            private long user_id;

            .. 생략

            @OneToMany(mappedBy = "user")
            private List<Account> accounts;
        }
        ```

        ```java
        public class Account {
            @Id
            private long id;

            .. 생략

            @ManyToOne
            @JoinColumn(name = "user_id")
            private User user;
        }
        ```

- JUnit 테스트에서는 AccountRepository 를 사용하여 return 된 List의 size() 가 1인지만 체크했다.
    - 그 List 에 포함된 Account 를 꺼내서 따로 다른 작업을 하지 않았기 때문에 Account 가 참조하고 있는 User entity 는 전혀 건드리지 않았다.
- 하지만, Controller 를 통해 Account Entity 를 Response 로 내보내고 브라우저에 json 형태로 뿌려주기 위해서는 Account entity 가 참조하고 있는 User entity 도 함께 불러오게 된다.
    - 여기서 문제가 발생한다.
    - User entity 는 다시 Account entity 를 참조하기 때문에 또 Account entity 를 불러오는 것이다.
    - Repository 에서는 Account entity 한개를 return 했지만 Controller 를 통해 Response 되어 json 으로 표시될 때는 이렇게 두 개의 entity 가 계속해서 서로를 불러오면서 페이지 가득 똑같은 데이터가 중복되어 노출된 것이다.

- JPA 의 양방향 참조로 인해서 RESTFul API 서버를 구현하는데 있어서 문제가 생기는데, 응답에 Entity 를 담아서 보낼 경우
- JsonSerializer 가 toString() 호출 → property 들을 매핑하는 과정에서 무한 순환 참조가 일어나게 되는 문제이다.


## 해결 방법

- `@JsonIgnore`: 이 어노테이션을 붙이면 json 데이터에 해당 프로퍼티는 `null`로 들어가게 된다.
    - json serialize 과정에서 `null`로 세팅된다.
    - 즉, 데이터에 아예 포함이 안되게 된다.
- `@JsonManagedReference`와 `@JsonBackReference`: 본질적으로 순환참조를 방어하기 위한 Annotation.
    - 부모 클래스(User entity)에 `@JsonManagedReference`,
    - 자식 클래스측(Account entity)에 `@JsonBackReference`을 추가해주면 된다.
- DTO 사용: 문제가 발생하게 된 주 원인은 '양방향 매핑'이기도 하지만, 더 정확하게는 entity 자체를 response 로 리턴한데 있다.
    - entity 자체를 return 하지 말고, dto 객체를 만들어 필요한 데이터만 옮겨담아 client 로 리턴하면 순환참조와 관련된 문제는 애초에 방어할수 있다.
- 맵핑 재설정 : 사람마다 다르지만 양방향 맵핑이 꼭 필요한지 다시 한번 생각해볼 필요가 있다.
    - 만약 양쪽에서 접근 할 필요가 없다면 단방향 맵핑을 하면 자연스레 순환참조가 해결된다.


- [Notion link](https://www.notion.so/JPA-7a1d8e2a81ea4df68447dbc1c079f1d5)
