# Chapter 3. 사용자 및 권한

## 1. 사용자 식별

- MySQL 의 사용자는 다른 DBMS 와는 조금 다르게 사용자의 계정 뿐만 아니라 사용자의 접속 지점도 계정의 일부가 된다. 따라서 계정을 언급할 땐 아이디와 호스트를 함께 명시해야 한다.
    - 사용자의 접속 지점: 클라이언트가 실행된 호스트명이나 도메인 또는 IP 주소
- 아이디와 IP 주소는 역따옴표(```)나 홑따옴표(`'`)로 감싸서 표현된다.
    - ex> `'jenny'@'127.0.0.1'`: MySQL 서버가 기동 중인 로컬 호스트에서 jenny 라는 아이디로 접속할 때만 사용될 수 있는 계정
- 모든 외부 컴퓨터에서 접속 가능한 사용자 계정: 사용자 계정의 호스트 부분을 `%` 문자로 대체하면 된다.
    - `%` 문자: 모든 IP 또는 모든 호스트명을 의미

- 다음과 같은 2개의 사용자 계정이 있는 MySQL 서버가 있다면 ?
    
    ```sql
    'svc_id'@'192.168.0.10' (이 계정의 비밀번호는 123)
    'svc_id'@'%' (이 계정의 비밀번호는 abc)
    ```
    
    - 권한이나 계정 정보에 대해 MySQL 은 범위가 가장 작은 것을 항상 먼저 선택한다.
        - 즉, `'svc_id'@'192.168.0.10'`인 계정을 선택한다.


## 2. 사용자 계정 관리

### 2-1. 시스템 계정과 일반 계정

---

- MySQL 8.0 부터 시스템 유저 권한의 유무에 따라 계정이 구분된다.
    - 시스템 계정
        - DB 서버 관리자를 위한 계정
        - DB 서버 관리와 관련된 중요 작업
            - 계정 관리(계정 생성 및 삭제, 계정의 권한 부여 및 제거)
            - 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
            - 스토어드 프로그램 생성 시 DEFINER 를 타 사용자로 설정
        - SYSTEM_USER 권한을 할당
    - 일반 계정
        - 응용 프로그램이나 개발자를 위한 계정
        - SYSTEM_USER 권한을 부여하지 않는다.

- 내장된 계정: `'root'@'localhost'`를 제외한 3개의 계정은 내부적으로 각기 다른 목적으로 사용되므로 삭제되지 않도록 주의
    - `'mysql.sys'@'localhost'`: 8.0부터 기본 내장된 sys 스키마의 객체(뷰, 함수, 프로시저)들의 DEFINER 로 사용되는 계정이다.
    - `'mysql.session'@'localhost'`: MySQL 플러그인이 서버로 접근할 때 사용되는 계정
    - `'mysql.infoschema'@'localhost'`: `information_schema`에 정의된 뷰의 DEFINER 로 사용되는 계정
    - → 처음부터 잠겨있는 상태(`account_locked`)이므로 의도적으로 잠긴 계정을 풀지 않는 한 악의적인 용도로 사용할 수 없으므로 보안을 걱정하지는 않아도 된다.
        
        ```sql
        mysql> SELECT user, host, account_locked FROM mysql.user WHERE user LIKE 'mysql%';
        +------------------+-----------+----------------+
        | user             | host      | account_locked |
        +------------------+-----------+----------------+
        | mysql.infoschema | localhost | Y              |
        | mysql.session    | localhost | Y              |
        | mysql.sys        | localhost | Y              |
        +------------------+-----------+----------------+
        3 rows in set (0.00 sec)
        ```
        

### 2-2. 계정 생성

---

- MySQL 8.0 버전부터는 계정의 생성은 `CREATE USER` 명령으로, 권한 부여는 `GRANT` 명령으로 구분해서 실행하도록 바뀌었다.
    - 5.7 까지는 `GRANT` 명령으로 권한의 부여와 동시에 계정 생성이 가능했다.
- 계정 생성시 설정 가능 옵션
    - 계정의 인증 방식과 비밀번호
    - 비밀번호 관련 옵션(유효 기간, 이력 개수, 재사용 불가 기간)
    - 기본 역할 (Role)
    - SSL 옵션
    - 계정 잠금 여부
- 일반적으로 많이 사용되는 옵션을 가진 `CREATE USER` 명령
    
    ```sql
    mysql> CREATE USER 'user'@'%'
             **IDENTIFIED WITH** 'mysql_native_password' BY 'password'
             **REQUIRED** NONE
             **PASSWORD EXPIRE** INTERVAL 30 DAY
             **ACCOUNT** UNLOCK
             **PASSWORD HISTORY** DEFAULT
             **PASSWORD REUSE INTERVAL** DEFAULT
             **PASSWORD REQUIRE** CURRENT DEFAULT;
    ```
    
- `IDENTIFIED WITH`
    - 사용자의 인증 방식과 비밀번호를 설정한다.
    - Native Pluggable Authentication: 단순히 비밀번호에 대한 해시(SHA-1) 값을 저장해두고, 클라이언트가 보낸 값과 해시값이 일치하는지 비교하는 방식
        - 5.7 버전까지 기본으로 사용되던 방식
    - Caching SHA-2 Pluggable Authentication
        - 8.0 부터 기본 인증 방식
        - 내부적으로 Salt 키를 활용하며, 수천 번의 해시 계산을 수행해서 결과를 만들어 내기 때문에 동일한 키 값에 대해서도 결과가 달라진다 → 성능이 매우 떨어지는 걸 보완하기 위해 해시 결과값을 메모리에 캐시해서 사용한다.
        - 이 인증 방식을 사용하려면 SSL/TLS 또는 RSA 키 페어를 반드시 사용해야 하기 때문에 클라이언트에서 접속할 때 SSL 옵션을 활성화해야 한다.
    - PAM Pluggable Authentication
        - 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증 방식
        - MySQL 엔터프라이즈 에디션에서만 사용 가능
    - LDAP Pluggable Authentication
        - LDAP 을 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식
        - MySQL 엔터프라이즈 에디션에서만 사용 가능
    - 기존 5.7 까지의 연결 방식과는 다른 방식으로 접속해야 하기 때문에, 보안 수준은 좀 낮아지겠지만 기존 버전과의 호환성을 고려한다면 Native Authentication 방식으로 계정을 생성해야 할 수도 있다.
        - 8.0 에서도 Native Authentication 을 기본 인증 방식으로 설정하려면 `my.cnf`설정 추가
            
            ```bash
            default_authentication_plugin=mysql_native_password
            ```
            
- `REQUIRE`
    - MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다.
        - 옵션을 SSL 로 설정하지 않았더라도 Caching SHA-2 Pluggable Authentication 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속할 수 있게 된다.
- `PASSWORD EXPIRE`
    - 비밀번호의 유효 기간을 설정하는 옵션
    - 설정 가능 옵션
        - `PASSWORD EXPIRE`: 계정의 생성과 동시에 비밀번호를 만료 처리
        - `PASSWORD EXPIRE NEVER`: 비밀번호의 만료 기간 없음
        - `PASSWORD EXPIRE DEFAULT`: `default_password_lifetime` 시스템 변수에 저장된 값을 사용하여 비밀번호 유효 기간을 설정
        - `PASSWORD EXPIRE INTERVAL n DAY`: 비밀번호 유효 기간을 `n`일자로 설정
- `PASSWORD HISTORY`
    - 한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션
    - 설정 가능 옵션
        - `PASSWORD HISTORY DEFAULT`: `password_history` 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하며, 저장된 이력이 있는 비밀번호는 재사용할 수 없다.
        - `PASSWORD HISTORY n`: 비밀번호 이력을 최근 `n`개까지만 저장하며, 저장된 이력에 남아 있는 비밀번호는 재사용할 수 없다.
    - 이전에 사용했던 비밀번호 확인하기
        
        ```sql
        mysql> SELECT * FROM mysql.password_history;
        +------------------+-----------+--------------------+----------------+
        | Host             | User      | Password_timestamp | Password       |
        +------------------+-----------+--------------------+----------------+
        | ...
        ```
        
- `PASSWORD REUSE INTERVAL`
    - 한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션
    - 설정 가능 옵션
        - `PASSWORD REUSE INTERVAL DEFAULT`: `password_reuse_interval` 변수에 저장된 기간으로 설정
        - `PASSWORD REUSE INTERVAL n`: `n`일자 이후에 비밀번호를 재사용 할 수 있게 설정
- `PASSWORD REQUIRE`
    - 비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호(만료된 비밀번호)를 필요로 할지 말지를 결정하는 옵션
    - 설정 가능 옵션
        - `PASSWORD REQUIRE DEFAULT`: `password_require_current` 시스템 변수의 값으로 설정
        - `PASSWORD REQUIRE CURRENT`: 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
        - `PASSWORD REQUIRE OPTIONAL`: 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- `ACCOUNT LOCK / UNLOCK`
    - 계정 생성 시 또는 `ALTER USER` 명령으로 계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 여부를 결정
    - 설정 가능 옵션
        - `ACCOUNT LOCK`: 계정을 사용하지 못하게 잠금
        - `ACCOUNT UNLOCK`: 잠긴 계정을 다시 사용 가능 상태로 잠금 해제


## 3. 비밀번호 관리

### 3-1. 고수준 비밀번호

---

- MySQL 서버의 비밀번호는 유효기간이나 이력 관리를 통한 재사용 금지 기능 뿐 아니라 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 있다.
- MySQL 서버에서 유효성 체크 규칙을 적용하려면 기본적으로 내장된 `validate_password` 컴포넌트를 이용하면 된다.
    - `validate_password` 컴포넌트는 MySQL 서버 프로그램에 내장돼 있다.
    
    ```bash
    # validate_password 컴포넌트 설치
    mysql> INSTALL COMPONENT 'file://component_validate_password';
    Query OK, 0 rows affected (0.60 sec)
    
    # 설치된 컴포넌트 확인
    mysql> SELECT * FROM mysql.component;
    +--------------+--------------------+------------------------------------+
    | component_id | component_group_id | component_urn                      |
    +--------------+--------------------+------------------------------------+
    |            1 |                  1 | file://component_validate_password |
    +--------------+--------------------+------------------------------------+
    
    # 컴포넌트에서 제공하는 시스템 변수 확인
    mysql> SHOW GLOBAL VARIABLES LIKE 'validate_password%';
    +--------------------------------------+--------+
    | Variable_name                        | Value  |
    +--------------------------------------+--------+
    | validate_password.check_user_name    | ON     |
    | validate_password.dictionary_file    |        |
    | validate_password.length             | 8      |
    | validate_password.**mixed_case_count**   | 1      |
    | validate_password.**number_count**       | 1      |
    | validate_password.policy             | MEDIUM |
    | validate_password.**special_char_count** | 1      |
    +--------------------------------------+--------+
    ```
    
    - 비밀번호 번호 정책
        - `LOW`: 비밀번호의 길이만 검증 (`validate_password.length`)
        - `MEDIUM`: DEFAULT, 비밀번호의 길이와 문자(숫자, 대소문자, 특수문자)의 배합을 검증
        - `STRONG`: `MEDIUM` 레벨의 검증을 모두 수행하며, 금칙어 포함 여부까지 검증
            - `validate_password.dictionary_file`: 금칙어들이 저장된 사전 파일. 한 줄에 하나씩 기록해서 저장한 텍스트 파일로 작성. ex> 금칙어 파일 [샘플](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10k-most-common.txt)
    - 금칙어 파일 등록
        
        ```sql
        mysql> SET GLOBAL validate_password.dictionary_file='prohibitive_word.data';
        mysql> SET GLOBAL validate_password.policy='STRONG';
        ```
        

### 3-2. 이중 비밀번호

---

- 이중 비밀번호: 2개의 비밀번호 중 하나만 일치하면 로그인이 통과되는 것을 의미
    - 프라이머리: 최근에 설정된 비밀번호
    - 세컨더리: 이전 비밀번호
- 이중 비밀번호 사용: 비밀번호 변경 구문에 `RETAIN CURRENT PASSWORD` 옵션만 추가
    - 고급 비밀번호에서 validate_password 켰더니🙂... `/etc/my.cnf` 수정!
        
        ```sql
        # Default Homebrew MySQL server config
        | mysql.infoschema | localhost | Y              |
        # Default Homebrew MySQL server config
        [mysqld]
        # Only allow connections from localhost
        bind-address = 127.0.0.1
        mysqlx-bind-address = 127.0.0.1
        
        default_authentication_plugin=mysql_native_password
        
        #### Real MySQL 8.0 51p ---
        max_connections=100
        
        #### InnoDB ---------------
        innodb_sort_buffer_size=5M
        
        innodb_log_files_in_group=2
        innodb_log_file_size=1024M
        
        innodb_buffer_pool_size=200M
        innodb_buffer_pool_instances=1
        innodb_io_capacity=100
        innodb_io_capacity_max=40
        
        #### Password validate ---
        **validate_password.policy=LOW**
        ```
        
    
    ```sql
    # 비밀번호를 "old_password" 로 설정
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password'
    
    # 프라이머리 비밀번호를 "new_password"로 설정하고, 이전 비밀번호는 세컨더리 비밀번호로 설정한다.
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
    ```
    
- 세컨더리 비밀번호 삭제
    
    ```sql
    mysql> ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
    ```
    
    - 모든 서비스가 재배포 되었다면 보안을 위해 세컨더리 비밀번호를 삭제하는 걸 추천한다.


## 4. 권한 (Privilege)

- MySQL 8.0부터 정적 권한(기존(5.7) 권한), 동적 권한의 개념이 추가 되었다. [링크](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_super)
    - 정적 권한: MySQL 서버의 소스코드에 고정적으로 명시돼 있는 권한
        - 글로벌 권한: DB나 테이블 이외의 객체에 적용되는 권한
            - `GRANT` 명령에서 특정 객체를 명시하지 말아야 한다.
        - 객체 권한: DB나 테이블을 제어하는 데 필요한 권한
            - `GRANT` 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야 한다.
    - 동적 권한: MySQL 서버가 시작되면서 동적으로 생성하는 권한
- MySQL 5.7 버전까지는 SUPER 권한이 DB 관리를 위해 꼭 필요한 권한이었지만, 8.0 부터는 SUPER 권한은 잘게 쪼개어져 동적 권한으로 분산됐다.
    - 백업 관리자와 복제 관리자 개별로 꼭 필요한 권한만 부여할 수 있게 되었다.

- `GRANT` 명령: 사용자에게 권한을 부여
    
    ```sql
    mysql> GRANT privilege_list ON db.table TO 'user'@'host';
    ```
    
    - 8.0부터 존재하지 않는 사용자에 `GRANT` 명령어를 실행하면 에러가 발생하기 때문에 권한을 부여하기 전에 사용자를 반드시 생성해야 한다.
- 범위 별 권한 부여하기
    
    ```sql
    # privilege_list 에는 구분자(,)를 써서 여러 권한을 동시에 명시할 수 있다.
    GRANT previlege1, previlege2, previlege3 ON db.table TO 'user'@'host';
    
    # 글로벌 권한
    -- 글로벌 권한은 특정 DB나 테이블에 부여할 수 없기 때문에 항상 *.* 을 사용한다.
    GRANT SUPER ON *.* TO 'user'@'localhost';
    
    # DB 권한
    -- DB 권한은 모든 DB, 특정 DB에 권한을 부여할 수 있기 때문에 *.* 나 employees.* 모두 사용할 수 있다.
    GRANT EVENT ON *.* TO 'user'@'localhost';
    GRANT EVENT ON employees.* TO 'user'@'localhost';
    
    # 테이블 권한
    -- 테이블 권한은 모든 DB, 특정 DB, 특정 DB 의 특정 테이블에 대해서도 권한을 부여할 수 있기 때문에 *.* 나 employees.*, employees.department 모두 사용할 수 있다.
    GRANT SELECT,INSERT,UPDATE,DELETE ON *.* TO 'user'@'localhost';
    GRANT SELECT,INSERT,UPDATE,DELETE ON employees.* TO 'user'@'localhost';
    GRANT SELECT,INSERT,UPDATE,DELETE ON employees.department TO 'user'@'localhost';
    ```
    
    - 칼럼 단위의 권한이 하나라도 설정되면 나머지 모든 테이블의 모든 칼럼에 대해서도 권한 체크를 하기 때문에 전체적인 성능에 영향을 미칠수 있어, 잘 사용하지 않는다.

- 권한 확인1: `SHOW GRANTS` 명령
    
    ```sql
    mysql> SHOW GRANTS FOR 'root'@'localhost';
    ```
    
- 권한 확인2: mysql DB 의 권한 관련 테이블 참조
    
    ```sql
    # 정적 권한
    select * from mysql.user -- 계정정보 & 계정이나 역할에 부여된 글로벌 권한
    select * from mysql.db -- 계정이나 역할에 DB단위로 부여된 권한
    select * from mysql.tables_priv -- 계정이나 역할에 테이블 단위로 부여된 권한
    select * from mysql.columns_priv -- 계정이나 역할에 컬럼 단위로 부여된 권한
    select * from mysql.procs_priv -- 계정이나 역할에 스토어드 프로그램 단위로 부여된 권한
    # 동적 권한
    mysql> select * from mysql.global_grants -- 계정이나 역할에 부여되는 동적 글로벌 권한
    ```
    

## 5. 역할 (Role)

- MySQL 8.0 버전부터 권한을 묶어서 역할(Role)을 사용할 수 있게 됐다.

- `CREATE ROLE` 명령: 역할 생성
- 처음 계정을 생성하면 아무 권한이 존재하지 않지만 `GRANT` 명령으로 역할을 부여하면 된다.
    
    ```sql
    mysql> CREATE USER 'jenny'@'127.0.0.1' IDENTIFIED BY '12345678';
    mysql> CREATE USER 'extra'@'127.0.0.1' IDENTIFIED BY '12345678';
    
    mysql> CREATE ROLE role_emp_read, role_emp_write;
    
    mysql> SELECT user, host, account_locked FROM mysql.user;
    +------------------+-----------+----------------+
    | user             | host      | account_locked |
    +------------------+-----------+----------------+
    | role_emp_read    | %         | Y              |
    | role_emp_write   | %         | Y              |
    | extra            | 127.0.0.1 | N              |
    | jenny            | 127.0.0.1 | N              |
    | mysql.infoschema | localhost | Y              |
    | mysql.session    | localhost | Y              |
    | mysql.sys        | localhost | Y              |
    | root             | localhost | N              |
    +------------------+-----------+----------------+
    # employees DB 조회, 변경 권한을 역할에 준다.
    mysql> GRANT SELECT ON employees.* TO role_emp_read;
    mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
    
    mysql> GRANT role_emp_read TO 'jenny'@127.0.0.1;
    mysql> GRANT role_emp_write TO 'extra'@127.0.0.1;
    ```
    
- 계정에 역할 부여
    - 역할 부여해도 계정의 활성화된 역할을 조회해 보면 없다고 나온다. 계정 로그인 후 `SET ROLE` 명령을 통해 역할을 활성화해야 한다.
        
        ```sql
        > mysql -ujenny -p -h '127.0.0.1'
        
        mysql> SELECT current_role();
        +----------------+
        | current_role() |
        +----------------+
        | NONE           |
        +----------------+
        mysql> SHOW GRANTS;
        +---------------------------------------------------+
        | Grants for jenny@127.0.0.1                        |
        +---------------------------------------------------+
        | GRANT USAGE ON *.* TO `jenny`@`127.0.0.1`         |
        | GRANT `role_emp_read`@`%` TO `jenny`@`127.0.0.1`  |
        +---------------------------------------------------+
        ```
        
        - MySQL 서버의 역할이 자동으로 활성화되지 않게 설정돼 있기 때문
    - `activate_all_roles_on_login` 시스템 변수: 사용자가 MySQL 서버에 로그인 시 역할을 자동으로 활성화할지 여부값
        
        ```sql
        # SET GLOBAL activate_all_roles_on_login=ON 설정으로 대체 가능
        mysql> SET ROLE role_emp_read;
        
        mysql> SELECT current_role();
        +---------------------+
        | current_role()      |
        +---------------------+
        | `role_emp_read`@`%` |
        +---------------------+
        
        mysql> SELECT user, host, account_locked FROM mysql.user;
        ERROR 1142 (42000): SELECT command denied to user 'jenny'@'localhost' for table 'user'
        
        # ㅠㅠ.. 다시 root 계정으로
        mysql> SELECT user, host, account_locked FROM mysql.user;
        +------------------+-----------+----------------+
        | user             | host      | account_locked |
        +------------------+-----------+----------------+
        | role_emp_read    | %         | Y              |
        | role_emp_write   | %         | Y              |
        | extra            | 127.0.0.1 | N              |
        | jenny            | 127.0.0.1 | N              |
        | mysql.infoschema | localhost | Y              |
        | mysql.session    | localhost | Y              |
        | mysql.sys        | localhost | Y              |
        | root             | localhost | N              |
        +------------------+-----------+----------------+
        ```
        
- 역할과 계정의 차이
    1. 서버 내부적으로 역할과 계정은 아무런 차이가 없다.(`mysql.user`)
    2. `CREATE ROLE` 명령으로 생성된 역할은 `account_locked` 컬럼의 값이 `Y`로 설정되서 로그인 용도로 사용할 수 없게 된다.
    - 3. 역할의 호스트 부분은 아무런 영향이 없다.
        - 호스트 부분을 명시하지 않은 경우에는 자동으로 ‘모든 호스트(`%`)’가 자동으로 추가된다.
        - 역할을 다른 계정에 부여하지 않고 직접 로그인하는 용도로 사용한다면(실제 계정처럼 사용한다면) 의미가 있다.
    1. 역할과 계정은 실제로 DB에서도 구분하기 힘들기 때문에 `role_`과 같은 prefix를 붙여 구분할 수 있도록 하는 것이 좋다.

- 계정의 기본 역할 또는 역할에 부여된 역할 그래프 관계 확인: `SHOW GRANTS` 명령, mysql DB 의 권한 관련 테이블 참조
    
    ```sql
    # 계정별 기본 역할
    mysql> SELECT * FROM mysql.default_roles;
    
    # 역할에 부여된 역할 관계 그래프
    mysql> SELECT * FROM mysql.role_edges;
    +-----------+----------------+-----------+---------+-------------------+
    | FROM_HOST | FROM_USER      | TO_HOST   | TO_USER | WITH_ADMIN_OPTION |
    +-----------+----------------+-----------+---------+-------------------+
    | %         | role_emp_read  | 127.0.0.1 | jenny   | N                 |
    | %         | role_emp_write | 127.0.0.1 | extra   | N                 |
    +-----------+----------------+-----------+---------+-------------------+
    ```
    

- 참고
    - [https://velog.io/@emawlrdl/CH3.-사용자-및-권한](https://velog.io/@emawlrdl/CH3.-%EC%82%AC%EC%9A%A9%EC%9E%90-%EB%B0%8F-%EA%B6%8C%ED%95%9C)
