# timestamp default value issue

## 문제

- test 용 sql script 실행 도중, create table 에서 다음과 같은 오류가 발생했다.

    ```
    nested exception is java.sql.SQLSyntaxErrorException: Invalid default value for 'start_date'
    ```

    [https://stackoverflow.com/questions/35237278/mysql-invalid-default-value-for-timestamp-when-no-default-value-is-given](https://stackoverflow.com/questions/35237278/mysql-invalid-default-value-for-timestamp-when-no-default-value-is-given)


## 원인

- mysql이 기본값이 제공되지 않은 경우 timestamp 필드를 처리하는 방법과 사용자의 SQL 모드 설정이 결합된 결과일 가능성이 높다.
- 기본값을 명시적으로 지정하지 않으면, Null이 아닌 timestamp row의 기본값은 '0000-00-00-00:00:00:00' 다.
- `no_zero_date` sql 모드도 서버에서 명시적으로 또는 엄격한 sql 모드의 일부로 사용하도록 설정되어 있을 수 있다. 이 SQL 모드는 '0000-00-00:00:00'을 기본값으로 설정하거나 이 값을 날짜 필드에 삽입하려는 경우 오류를 생성한다.
- 따라서 테이블에서 timestamp 데이터 유형을 사용할 수 있지만 두 번째 데이터 유형을 null로 만들거나 0 또는 유효 날짜(예: 기간)를 명시적 기본값으로 제공할 수 있다.
- 시작 날짜와 종료 날짜를 이 필드로 표시하고 있으므로 timestamp 대신 datetime을 데이터 형식으로 사용하는 것이 좋다.


## 해결

- schema.sql 파일에 `SET sql_mode = '';` 추가
    - start_date 같은 필드는 DATETIME 으로 바꿔야할지? → index 고려해서 결정하자!

- 참고
    - [https://www.sqlteam.com/articles/timestamps-vs-datetime-data-types](https://www.sqlteam.com/articles/timestamps-vs-datetime-data-types)


- [Notion link](https://www.notion.so/timestamp-default-value-issue-4928e81f787646b9a960ad0ae3d68f39)
