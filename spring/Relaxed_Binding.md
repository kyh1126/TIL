# Relaxed Binding

- SpringBoot 는 Environment properties 를 유연한 규칙으로 바인딩한다.
- 스프링 부트(v2.5.2) 레퍼런스 참고: [https://docs.spring.io/spring-boot/docs/2.5.2/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables](https://docs.spring.io/spring-boot/docs/2.5.2/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)

### Relaxed Binding 종류

---

- `my.main-project.person.first-name`: Kebab case, `.properties`와 `.yml` 파일 안에서 추천되는 사용 방식이다.
- `my.main-project.person.firstName`: 표준 camel case 구문.
- `my.main-project.person.first_name`: 언더스코어 표기법, `.properties`와 `.yml` 파일 안에서 대체 형식이다.
- `MY_MAINPROJECT_PERSON_FIRSTNAME`: 대문자 형식, 시스템 환경 변수들에 추천되는 사용 방식이다.

### Property source 별 Relaxed binding 규칙들

---

- Properties 파일들: Camel case, kebab case, 또는 언더스코어 표기법. List 의 경우 `[]`(표준 list 구문)이나 comma 구분된 값들
- YAML 파일들: Camel case, kebab case, 또는 언더스코어 표기법. List 의 경우 표준 YAML list 구문이나 comma 구분된 값들
    - ex1>

        ```yaml
        ports:
        - protocol: TCP
          port: 6379

        # is equivalent to
        ports: [{"protocol": "TCP", "port": 6379}]
        ```

    - ex2>

        ```yaml
        metadata:
          name: test-network-policy
          namespace: default

        # is equivalent to
        metadata: {"name": "test-network-policy", "namespace": "default"}
        ```

    - ex3>

        ```yaml
        metadata:
          - name: test-network-policy
          - namespace: default

        # is equivalent to
        metadata: [
          {"name": "test-network-policy"},
          {"namespace": "default"}
        ]
        ```

- Environment 변수들: 언더스코어를 구분자로 쓴 대문자 형식. List 의 경우 언더스코어로 둘러싸인 숫자값들 → 아래에서 더 살펴볼 예정
- System properties: Camel case, kebab case, 또는 언더스코어 표기법. List 의 경우 `[]`(표준 list 구문)이나 comma 구분된 값들

> properties 들은 소문자의 kebab 포맷을 추천한다 ex> `my.person.first-name`=Rod

- 환경 변수들의 Binding
    - 대부분의 운영 체제는 환경 변수에 사용할 수 있는 이름에 대해 엄격한 규칙을 적용한다.
        - Linux 셸 변수: 문자(a ~ z 또는 A ~ Z), 숫자(0 ~ 9) 또는 밑줄 문자(_)만 포함될 수 있다.
        - 일반적으로 Unix 셸 변수의 이름도 대문자로 표시된다.
    - 스프링 부트의 Relaxed 바인딩 규칙은 가능한 한 이러한 이름지정 제한과 호환되도록 설계되었다.
    - 표준 양식의 property 이름을 환경 변수 이름으로 변환하려면 다음 규칙을 따르자.
        - 점(`.`)을 언더스코어(`_`)로 변환한다.
        - 모든 대쉬(-)들을 제거한다.
        - 대문자로 변환한다.
    - ex> configuration property `spring.main.log-startup-info`는 `SPRING_MAIN_LOGSTARTUPINFO`환경 변수로 바뀐다.
    - 환경 변수는 object 목록에 바인딩할 때도 사용될 수 있다. List 에 바인딩하려면 변수 이름에서 요소 번호를 밑줄로 둘러싸야 합니다.
        - ex> configuration property `my.service[0].other`는 `MY_SERVICE_0_OTHER` 환경 변수로 사용된다.
    