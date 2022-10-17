# Chapter 2. 몽고DB 기본

- 몽고DB의 기본 개념
    - 도큐먼트: 몽고DB 데이터의 기본 단위. 관계형 데이터베이스의 행과 유사하다
    - 컬렉션: 동적 스키마가 있는 테이블과 같다.
    - 몽고DB의 단일 인스턴스는 자체적인 컬렉션을 갖는 여러 개의 독립적인 데이터베이스를 호스팅한다.
    - 모든 도큐먼트는 컬렉션 내에서 고유한 특수키인 "`_id`"를 가진다.
    - 몽고DB는 몽고 셸 이라는 간단하지만 강력한 도구와 함께 배포된다.
        - mongo 셸은 몽고DB 인스턴스를 관리하고 몽고DB 쿼리 언어로 데이터를 조작하기 위한 내장 지원을 제공한다.
        - 또한 사용자가 다양한 목적으로 자신의 스크립트를 만들고 로드할 수 있는 완전한 기능의 자바스크립트 해석기다.

## 1. 도큐먼트

---

- 몽고DB의 핵심: 정렬된 키와 연결된 값의 집합으로 이뤄진 도큐먼트
    - 몽고DB의 도큐먼트: 관계형 데이터베이스의 행에 대응된다.
- 대부분의 언어는 맵, 해시, 딕셔너리와 같이 도큐먼트를 자연스럽게 표현하는 자료구조를 가진다.
    - ex> 자바스크립트는 객체로 표현된다.
        
        ```jsx
        {"greeting" : "Hello, world!"}
        ```
        

- 도큐먼트의 키는 문자열이다. 다음 예외 몇 가지를 제외하면 어떤 UTF-8 문자든 쓸 수 있다.
    - 키는 `\0`(`null` 문자)을 포함하지 않는다. `\0`은 키의 끝을 나타내는 데 사용된다.
    - `.`과 `$` 문자는 몇 가지 특별한 속성을 가지며 이후 장에서 설명할 특정 상황에만 사용해야 한다.
        - 이 문자들은 보통 예약어로 취급해야 하며 부적절하게 사용하면 드라이버에서 경고를 발생한다.

- 몽고DB는 데이터형과 대소문자를 구별한다.
    - 다음 두 도큐먼트는 서로 다르다.
        
        ```json
        {"count": 5}
        {"count": "5"}
        
        {"count": 5}
        {"Count": 5}
        ```
        

- 몽고DB에서는 키가 중복될 수 없음
    - 다음 도큐먼트는 올바른 도큐먼트가 아니다.
        
        ```json
        {"greeting" : "Hello, world!", "greeting" : "Hello, MongoDB!"}
        ```
        

## 2. 컬렉션

---

- 컬렉션: 도큐먼트의 모음이다.
    - 관계형 데이터베이스의 테이블에 대응된다.

### 2-1. 동적 스키마

---

- 컬렉션은 동적 스키마를 갖는다.
    - 하나의 컬렉션 내 도큐먼트들이 모두 다른 구조를 가질 수 있다는 의미
        
        ```json
        // 다음 도큐먼트들을 하나의 컬렉션에 저장할 수 있다.
        {"greeting" : "Hello, world!", "vies" : 3}
        {"signoff" : "Good night, and good luck"}
        ```
        

- 도큐먼트들의 키, 키의 개수, 데이터형의 값은 모두 다르다.

❓’왜 별도의 컬렉션이 필요하지?’, ’왜 하나 이상의 컬렉션이 필요할까?’

- 같은 컬렉션에 다른 종류의 도큐먼트를 저장하면 개발자와 관리자에게 번거로운 일이 생길 수도 있다.
    - 각 쿼리가 특정 스키마를 고수하는 도큐먼트를 반환하는지, 혹은 쿼리한 코드가 다른 구조의 도큐먼트를 다룰 수 있는지 확실히 확인하자.
    - ex> 블로그 게시물을 쿼리한 데이터 중 작성자 데이터만 제거하려면 상당히 번거롭다.
- 컬렉션별로 목록을 뽑으면 한 컬렉션 내 특정 데이터형별로 쿼리해서 목록을 뽑을 때마다 훨씬 빠르다.
    - ex> "요약", "전체", "뚱뚱이 원숭이"와 같은 컬렉션형을 값으로 하는 "type" 키가 도큐먼트 내에 있다면, 단일 컬렉션에 들어 있는 값을 찾기보다 세 컬렉션 중에서 올바른 컬렉션을 쿼리하는 편이 훨씬 빠르다.
- 같은 종류의 데이터를 하나의 컬렉션에 모아두면 데이터 지역성에도 좋다.
    - 블로그 게시물 여러 개를 뽑는 경우, 게시물과 저자 정보가 섞인 컬렉션보다 게시물만 들어 있는 컬렉션에서 뽑을 때 디스크 탐색 시간이 더 짧다.
- 인덱스를 만들면 도큐먼트는 특정 구조를 가져야 한다 → 인덱스는 컬렉션별로 정의한다.
    - 같은 유형의 도큐먼트를 하나의 컬렉션에 넣음으로써 컬렉션을 효율적으로 인덱싱할 수 있다.

- 애플리케이션 스키마는 기본적으로 필요하지는 않지만 정의하면 좋다.****
    - 몽고DB의 도큐먼트 유효성 검사 기능과 객체-도큐먼트 매핑 라이브러리를 이용하며, 이는 많은 프로그래밍 언어에서 사용 가능하다.

### 2-2. 네이밍

---

- 컬렉션은 이름으로 식별된다. 컬렉션명은 어떤 UTF-8 문자열이든 쓸 수 있지만 몇 가지 제약 조건이 있다.
    - 빈 문자열(`" "`)은 유효한 컬렉션명이 아니다.
    - `\0`(`null` 문자)은 컬렉션명의 끝을 나타내는 문자이므로 컬렉션명에 사용할 수 없다.
    - `system.`으로 시작하는 컬렉션명은 시스템 컬렉션에서 사용하는 예약어이므로 사용할 수 없다.
        - ex> `system.users` 컬렉션에는 데이터베이스 사용자 정보가, `system.namespaces` 컬렉션에는 데이터베이스 내 모든 컬렉션의 정보가 들어 잇다.
    - 사용자가 만든 컬렉션은 이름에 예약어인 `$`를 포함할 수 없다.
        - 시스템에서 생성한 몇몇 컬렉션에서 `$` 문자를 사용하므로 데이터베이스에서 사용하는 다양한 드라이버가 `$` 문자를 포함하는 컬렉션명을 지원하기는 한다 → 이런 컬렉션에 접근할 때가 아니라면 `$`를 컬렉션명에 사용해서는 안 된다.

- 서브컬렉션
    - 서브컬렉션의 네임스페이스에 `.` 문자를 사용해 컬렉션을 체계화한다.****
        - ex> 블로그 기능이 있는 애플리케이션은 blog.posts와 blog.authors라는 컬렉션을 가질 수 있다.
            - 이는 단지 체계화를 위함이며 blog 컬렉션이나 자식 컬렉션과는 아무런 관계가 없다.
    - 큰 파일을 저장하는 프로토콜인 GridFS는 콘텐츠 데이터와 별도로 메타데이터를 저장하는 데 서브컬렉션을 사용한다.
    - 대부분의 드라이버는 특정 컬렉션의 서브컬렉션에 접근하는 몇 가지 편리한 문법을 제공한다.
        - ex> 데이터베이스 셸에서 `db.blog`는 blog 컬렉션을, `db.blog.posts`는 `blog.posts` 컬렉션을 보여준다.

## 3. 데이터베이스

---

- 몽고DB는 컬렉션에 도큐먼트를 그룹화할 뿐 아니라 데이터베이스에 컬렉션을 그룹지어 놓는다.
- 몽고DB의 단일 인스턴스는 여러 데이터베이스를 호스팅할 수 있으며, 각 데이터베이스를 완전히 독립적으로 취급할 수 있다.
    - 한 애플리케이션의 데이터를 동일한 데이터베이스에 저장하는 것은 좋은 방식이다.

- 데이터베이스는 컬렉션과 마찬가지로 이름으로 식별된다. 데이터베이스 이름에는 어떤 UTF-8 문자열이든 쓸 수 있지만 몇 가지 제약 조건이 있다.
    - 빈 문자열(`" "`)은 유효한 데이터베이스 이름이 아니다.
    - 데이터베이스 이름은 다음 문자를 포함할 수 없다. `/`, `\`, `.`, `' '`, `*`, `<`, `>`, `:`, `|`, `?`, `$`, (단일 공간), `\0`(`null` 문자)
    - 데이터베이스 이름은 대소문자를 구별한다.
    - 데이터베이스 이름은 최대 64바이트다.
- 직접 접근할 수는 있지만 특별한 의미론(semantics)을 갖는 예약된 데이터베이스 이름도 있다.
    - admin
        - 인증(authentication)과 권한 부여(authorization) 역할을 한다.
        - 일부 관리 작업을 하려면 이 데이터베이스에 대한 접근이 필요하다.
    - local
        - 단일 서버에 대한 데이터를 저장한다.
        - 복제 셋(replica set)에서 local은 복제(replication) 프로세스에 사용된 데이터를 저장한다.
            - local 데이터베이스 자체는 복제되지 않는다.
    - config
        - 샤딩된 몽고DB 클러스터는 config 데이터베이스를 사용해 각 샤드의 정보를 저장한다.

- 컬렉션을 저장하는 데이터베이스의 이름을 컬렉션명 앞에 붙이면 올바른 컬렉션명인 네임스페이스를 얻는다.
    - ex> cms 데이터베이스의 blog.posts 컬렉션을 사용 → 컬렉션의 네임스페이스는 cms.blog.posts

## 4. 몽고DB 시작

---

- 서버를 시작하려면 원하는 유닉스 명령행 환경에서 `mongod` 실행 파일을 실행한다.
    
    ```bash
    # 몽고DB 설치
    $ brew tap mongodb/brew
    ..
    
    $ brew install mongodb-community
    ..
    
    $ brew services start mongodb/brew/mongodb-community
    ..
    ==> Successfully started `mongodb-community` (label: homebrew.mxcl.mongodb-community)
    
    $ mongod --version
    db version v6.0.1
    Build Info: {
        "version": "6.0.1",
        "gitVersion": "32f0f9c88dc44a2c8073a5bd47cf779d4bfdee6b",
        "modules": [],
        "allocator": "system",
        "environment": {
            "distarch": "aarch64",
            "target_arch": "aarch64"
        }
    }
    ```
    
- `mongod`는 인수(argument) 없이 실행하면 기본 데이터 디렉토리로 `/data/db`(윈도우에서는 `\data\db\`)를 사용한다.
    - 데이터 디렉터리가 존재하지 않거나 쓰기 권한이 없을 때는 서버가 실행되지 않는다.
- 시작할 때 서버는 버전과 시스템 정보를 출력한 후 클라이언트의 연결을 기다린다. 몽고DB는 기본적으로 27017번 포트에서 소켓 연결을 기다린다.

## 5. 몽고DB 셸 소개

---

- 몽고DB는 명령행에서 몽고DB 인스턴스와 상호작용하는 자바스크립트 셸을 제공한다.
- 셸: 관리 기능이나, 실행 중인 인스턴스를 점검하거나 간단한 기능을 시험하는 데 매우 유용하다.

### 5-1. 셸 실행

---

- `mongo`를 실행해 셸을 시작 → 레거시 `mongo` 셸은 6.0 이상 버전부터 서버 바이너리와 함께 제공되지 않는다.
    
    ```bash
    $ mongo
    zsh: command not found: mongo
    
    $ echo 'export PATH="/opt/homebrew/opt/mongodb-community@6.0/bin:$PATH"' >> ~/.zshrc
    $ source ~/.zshrc
    ```
    
    - `mongosh` 설치
        
        ```bash
        $ brew install mongosh
        Warning: mongosh 1.6.0 is already installed and up-to-date.
        ```
        
- `mongosh`를 실행해 셸을 시작한다.
    
    ```bash
    $ mongosh
    Current Mongosh Log ID:	634c21b48c01c60d77b30d11
    Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.6.0
    Using MongoDB:		6.0.1
    Using Mongosh:		1.6.0
    
    For mongosh info see: https://docs.mongodb.com/mongodb-shell/
    
    To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
    You can opt-out by running the disableTelemetry() command.
    
    ------
       The server generated these startup warnings when booting
       2022-10-16T23:52:58.063+09:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
    ------
    
    ------
       Enable MongoDB's free cloud-based monitoring service, which will then receive and display
       metrics about your deployment (disk utilization, CPU, operation statistics, etc).
    
       The monitoring data will be available on a MongoDB website with a unique URL accessible to you
       and anyone you share the URL with. MongoDB may use this information to make product
       improvements and to suggest MongoDB products and deployment options to you.
    
       To enable free monitoring, run the following command: db.enableFreeMonitoring()
       To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
    ------
    
    test>
    ```
    
    - 셸은 시작하면 자동으로 로컬 장비에서 실행 중인 몽고DB 서버에 접속을 시도

- 셸은 완전한 자바스크립트 해석기이며 임의의 자바스크립트 프로그램을 실행한다. 또한 표준 자바스크립트 라이브러리의 모든 기능을 활용할 수 있다. 자바스크립트 함수를 정의하고 호출할 수도 있다. 여러 줄의 명령도 작성할 수 있다.
    
    ```bash
    test> x = 200;
    200
    test> x/5;
    40
    
    test> Math.sin(Math.PI / 2)
    1
    test> new Date("20109/1/1");
    ISODate("+020108-12-31T15:00:00.000Z")
    test> "Hello, World!".replace("World", "MongoDB");
    Hello, MongoDB!
    
    test> function factorial (n) {
    ... if (n <= 1) return 1;
    ... return n*factorial(n - 1);
    ... }
    [Function: factorial]
    test> factorial(5);
    120
    ```
    

### 5-2. 몽고DB 클라이언트

---

- 셸은 시작할 때 몽고DB 서버의 test 데이터베이스에 연결하고, 데이터베이스 연결을 전역 변수 db에 할당한다.

- 현재 db에 할당된 데이터베이스를 확인하려면 `db`를 입력한다.
    
    ```bash
    test> db
    test
    ```
    
- 데이터베이스 선택
    
    ```bash
    test> use video
    switched to db video
    video> # db 변수는 video 데이터베이스를 가리킨다.
    ```
    
- db 변수에서 컬렉션에 접근
    
    ```bash
    video> db.movies # 현재 데이터베이스의 movies 컬렉션을 반환한다.
    video.movies
    video>
    ```
    

### 5-3. 셸 기본 작업

---

- 생성
    - `insertOne`: 컬렉션에 도큐먼트를 추가한다.
        
        ```bash
        # 도큐먼트를 나타내는 자바스크립트 객체인 movie라는 지역 변수를 생성한다.
        test> movie = {"title" : "Star Wars: Episode IV - A New Hope",
        ... "director" : "George Lucas",
        ... "year" : 1977}
        {
          title: 'Star Wars: Episode IV - A New Hope',
          director: 'George Lucas',
          year: 1977
        }
        
        # movies 컬렉션에 저장
        test> db.movies.insertOne(movie)
        {
          acknowledged: true,
          insertedId: ObjectId("634c25b48c01c60d77b30d12")
        }
        ```
        
- 읽기
    - `findOne`: 컬렉션에서 단일 도큐먼트를 읽음
        
        ```bash
        test> db.movies.findOne()
        {
          _id: ObjectId("634c25b48c01c60d77b30d12"),
          title: 'Star Wars: Episode IV - A New Hope',
          director: 'George Lucas',
          year: 1977
        }
        ```
        
    - `find`: 일치하는 도큐먼트를 20개까지 자동으로 출력하지만 그 이상도 가져올 수 있다.
        
        ```bash
        test> db.movies.find().pretty()
        [
          {
            _id: ObjectId("634c25b48c01c60d77b30d12"),
            title: 'Star Wars: Episode IV - A New Hope',
            director: 'George Lucas',
            year: 1977
          }
        ]
        ```
        
    
    → `find`와 `findOne`은 컬렉션을 쿼리하는 데 사용한다. 쿼리 도큐먼트 형태로 조건 전달도 가능하다.
    
- 갱신
    - `updateOne`: 매개변수는 최소 두 개다. 첫 번째는 수정할 도큐먼트를 찾는 기준이고, 두 번째는 갱신 작업을 설명하는 도큐먼트다. 갱신하려면 갱신 연산자인 `set`을 이용한다.
        
        ```bash
        test> db.movies.updateOne({title : "Star Wars: Episode IV - A New Hope"},
        ... {$set : {reviews: []}}) # 도큐먼트에 새 키 값으로 리뷰 배열을 추가한다.
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        test> db.movies.find().pretty()
        [
          {
            _id: ObjectId("634c25b48c01c60d77b30d12"),
            title: 'Star Wars: Episode IV - A New Hope',
            director: 'George Lucas',
            year: 1977,
            reviews: []
          }
        ]
        ```
        
- 삭제
    - `deleteOne`과 `deleteMany`는 도큐먼트를 데이터베이스에서 영구적으로 삭제한다. 필터 도큐먼트로 삭제 조건을 지정한다.
        
        ```bash
        test> db.movies.deleteOne({title : "Star Wars: Episode IV - A New Hope"})
        { acknowledged: true, deletedCount: 1 }
        ```
        

## 6. 데이터형

---

### 6-1. 기본 데이터형

---

- 몽고DB는 JSON의 키/값 쌍 성질을 유지하면서 추가적인 데이터형을 지원한다.
    - JSON 데이터형: `null`, 불리언, 숫자, 문자열, 배열, 객체만 지원

- 가장 일반적인 데이터형
    - `null`: null 값과 존재하지 않는 필드를 표현하는 데 사용
        
        ```json
        {"x" : null}
        ```
        
    - 불리언: 참과 거짓 값에 사용
        
        ```json
        {"x" : true}
        ```
        
    - 숫자: 셸은 74비트 부동소수점 수를 기본으로 사용. 4바이트 혹은 8바이트의 부호 정수는 각각 `NumberInt`, `NumberLong` 클래스를 사용한다.
        
        ```json
        {"x" : 3.14}
        {"x" : 3}
        {"x" : NumberInt("3")}
        {"x" : NumberLong("3")}
        ```
        
    - 문자열: 어떤 UTF-8 문자열이든 문자열형으로 표현할 수 있다.
        
        ```json
        {"x" : "foobar"}
        ```
        
    - 날짜: 몽고DB는 1970년 1월 1일부터의 시간을 1/1000초 단위로 나타내는 64비트 정수로 날짜를 저장한다. 표준 시간대(time zone)는 저장하지 않는다.
        
        ```json
        {"x" : new Date()}
        ```
        
    - 정규 표현식: 쿼리는 자바스크립트의 정규 표현식 문법을 사용할 수 있다.
        
        ```json
        {"x" : /foobar/i}
        ```
        
    - 배열: 값의 셋이나 리스트를 배열로 표현할 수 있다.
        
        ```json
        {"x" : ["a", "b", "c"]}
        ```
        
    - 내장 도큐먼트: 부모 도큐먼트의 값으로 내장된 도큐먼트 전체를 포함할 수 있다.
        
        ```json
        {"x" : {"foo" : "bar"}}
        ```
        
    - 객체 ID: 도큐먼트용 12바이트 ID다.
        
        ```json
        {"x" : ObjectId()}
        ```
        

- 상대적으로 덜 사용되는 데이터형 목록
    - 이진 데이터: 임의의 바이트 문자열이며 셸에서는 조작이 불가능하다. 데이터베이스에 UTF-8이 아닌 문자열을 저장하는 유일한 방법이다.
    - 코드: 쿼리와 도큐먼트는 임의의 자바스크립트 코드를 포함할 수 있다.
        
        ```json
        {"x" : function() { /* ... */}}
        ```
        

### 6-2. 날짜

---

- 새로운 `Date` 객체를 생성할 때는 항상 `Date()`가 아닌 `new Date()`를 호출해야 한다.
    - 함수로 생성자를 호출하면(new가 포함되지 않은 것) 실제 `Date` 객체가 아닌 날짜의 문자열 표현을 반환한다.
        
        ```json
        test> Date()
        Mon Oct 17 2022 01:04:35 GMT+0900 (대한민국 표준시)
        test> x = Date()
        Mon Oct 17 2022 01:04:42 GMT+0900 (대한민국 표준시)
        test> x
        Mon Oct 17 2022 01:04:42 GMT+0900 (대한민국 표준시)
        
        test> x = new Date()
        ISODate("2022-10-16T16:04:50.891Z")
        test> x
        ISODate("2022-10-16T16:04:50.891Z")
        test> new Date()
        ISODate("2022-10-16T16:05:14.695Z")
        ```
        
- 셸에서는 날짜가 현지 시간대 설정을 이용해 표시된다.

### 6-3. 배열

---

- 배열은 정렬 연산(리스트, 스택, 큐)과 비정렬 연산(셋)에 호환성 있게 사용 가능한 값이다.
    
    ```json
    test> x = {"things" : ["pie", 3.14]}
    ```
    
    - 배열은 서로 다른 데이터형을 값으로 포함할 수 있다.
    - 배열에 쿼리하거나 배열의 내용을 이용해 인덱스를 만들 수 있다.
    - 몽고DB에서는 배열 내부에 도달해서 원자적으로 배열의 내용을 수정할 수 있다.

### 6-4. 내장 도큐먼트

---

- 내장 도큐먼트: 도큐먼트는 키에 대한 값이 될 수 있다.
    
    ```json
    test> x = {
    ...   "name" : "John Doe",
    ...   "address" : {
    ...     "street" : "123 Park Street",
    ...     "city" : "Anytown",
    ...     "state" : "NY"
    ...   }
    ... }
    {
      name: 'John Doe',
      address: { street: '123 Park Street', city: 'Anytown', state: 'NY' }
    }
    ```
    
- 배열과 마찬가지로 몽고DB는 내장 도큐먼트의 구조를 ‘이해’하고, 인덱스를 구성하고, 쿼리하며, 갱신하기 위해 내장 도큐먼트 내부에 접근한다.
- 하지만 몽고DB에서는 더 많은 데이터 반복이 생길 수 있다는 단점이 있다.
    - 관계형 데이터베이스에서 address 가 분리된 테이블에 있고 주소의 오타를 고쳐야 한다고 가정하자.
        - people과 address를 조인하면 같은 주소를 갖는 모든 사람의 주소를 수정할 수 있다.
    - 몽고DB에서는 각 사람의 도큐먼트에서 오타를 수정해야 한다.

### 6-5. `_id`와 `ObjectId`

---

- `_id`: 몽고DB에 저장된 모든 도큐먼트는 "`_id`" 키를 가진다. "`ObjectId`"가 기본이다.
    - 컬렉션 내 모든 도큐먼트가 고유하게 식별되게 한다.

- ObjectId
    - "`_id`"의 기본 데이터형이다.
    - ObjectId 클래스는 가벼우면서도, 여러 장비에 걸쳐 전역적으로 고유하게(유일하게) 생성하기 쉽게 설계됐다.
    - 전통적인 것이 아닌 ObjectId를 사용하는 주요 이유는 몽고DB의 분산 특성 때문이다.
        - 여러 서버에 걸쳐 자동 증가하는 기본 키를 동기화하는 작업은 어렵고 시간이 걸린다.
        - 몽고DB는 분산 데이터베이스로 설계됐기 때문에 샤딩된 환경에서 고유 식별자를 생성하는 것이 매우 중요했다.
    - ObjectId는 12바이트 스토리지를 사용하며 24자리 16진수 문자열 표현이 가능하다. 바이트당 2자리를 사용한다.
        - 첫 4바이트는 1970년 1월 1일부터의 시간을 1/1000초 단위로 저장하는 타임스탬프다.
            - 타임스탬프가 맨 처음에 온다는 것은 ObjectId가 대략 입력 순서대로 정렬된다는 의미다. 이는 확실히 보장되지는 않지만, ObjectId를 효율적으로 인덱싱하는 등 다소 멋진 특성이 있다.
        - 다음 5바이트는 랜덤 값이다.
        - 마지막 3바이트는 단순히 증분하는 숫자로, 1초 내 단일 프로세스의 유일성을 보장한다.
    - 고유한 ObjectId는 프로세스당 1초에 1677만 7216개까지 생성된다.

- _id 자동 생성
    - 도큐먼트를 입력할 때 "`_id`" 키를 명시하지 않으면 입력된 도큐먼트에 키가 자동으로 추가된다.
    - 이는 몽고DB 서버에서 관리할 수 있지만 일반적으로는 클라이언트 쪽 드라이버에서 관리한다.

## 7. 몽고DB 셸 사용

---

- `mongosh`
    
    ```bash
    # 다음과 같다.
    mongosh "mongodb://localhost:27017"
    ```
    

- 다른 장비나 포트에 `mongosh`를 연결하려면 셸을 시작할 때 호스트명, 포트를 명시해야 한다.
    
    ```bash
    # localhost에 디폴트 포트가 아닌거로 접근할 때
    mongosh --port 30000
    
    # 원격 호스트의 몽고DB 인스턴스에 접근할 때
    mongosh "mongodb://mongodb0.example.com:28015"
    
    # 특정 Database에 연결할 때, 특정하지 않을 경우 test Database로 연결
    mongosh "mongodb://localhost:27017/db1"
    ```
    

- 시작한 후 원하는 대에 `new Mongo(호스트명)`를 실행함으로써 `mongod`에 연결한다.
    
    ```bash
    test> conn = new Mongo("localhost:27017")
    mongodb://localhost:27017/?directConnection=true&serverSelectionTimeoutMS=2000
    
    test> db = conn.getDB("myDB")
    myDB
    ```
    

### 7-1. 셸 활용 팁

---

- `help`를 입력하면 셸에 내장된 도움말을 볼 수 있다.
    
    ```bash
    myDB> help
    
      Shell Help:
    
        use                                        Set current database
        show                                       'show databases'/'show dbs': Print a list of all available databases.
                                                   'show collections'/'show tables': Print a list of all collections for current database.
                                                   'show profile': Prints system.profile information.
                                                   'show users': Print a list of all users for current database.
                                                   'show roles': Print a list of all roles for current database.
                                                   'show log <type>': log for current connection, if type is not set uses 'global'
                                                   'show logs': Print all logs.
    
        exit                                       Quit the MongoDB shell with exit/exit()/.exit
        quit                                       Quit the MongoDB shell with quit/quit()
        Mongo                                      Create a new connection and return the Mongo object. Usage: new Mongo(URI, options [optional])
        connect                                    Create a new connection and return the Database object. Usage: connect(URI, username [optional], password [optional])
        it                                         result of the last line evaluated; use to further iterate
        version                                    Shell version
        load                                       Loads and runs a JavaScript file into the current shell environment
        enableTelemetry                            Enables collection of anonymous usage data to improve the mongosh CLI
        disableTelemetry                           Disables collection of anonymous usage data to improve the mongosh CLI
        passwordPrompt                             Prompts the user for a password
        sleep                                      Sleep for the specified number of milliseconds
        print                                      Prints the contents of an object to the output
        printjson                                  Alias for print()
        cls                                        Clears the screen like console.clear()
        isInteractive                              Returns whether the shell will enter or has entered interactive mode
    
      For more information on usage: https://docs.mongodb.com/manual/reference/method
    ```
    
- `db.help()`: 데이터베이스 수준의 도움말
    
    ```bash
    myDB> db.help()
    
      Database Class:
    
        getMongo                                   Returns the current database connection
        getName                                    Returns the name of the DB
        getCollectionNames                         Returns an array containing the names of all collections in the current database.
        getCollectionInfos                         Returns an array of documents with collection information, i.e. collection name and options, for the current database.
        runCommand                                 Runs an arbitrary command on the database.
        adminCommand                               Runs an arbitrary command against the admin database.
        aggregate                                  Runs a specified admin/diagnostic pipeline which does not require an underlying collection.
        getSiblingDB                               Returns another database without modifying the db variable in the shell environment.
        getCollection                              Returns a collection or a view object that is functionally equivalent to using the db.<collectionName>.
        dropDatabase                               Removes the current database, deleting the associated data files.
        createUser                                 Creates a new user for the database on which the method is run. db.createUser() returns a duplicate user error if the user already exists on the database.
        updateUser                                 Updates the user’s profile on the database on which you run the method. An update to a field completely replaces the previous field’s values. This includes updates to the user’s roles array.
        changeUserPassword                         Updates a user’s password. Run the method in the database where the user is defined, i.e. the database you created the user.
        logout                                     Ends the current authentication session. This function has no effect if the current session is not authenticated.
        dropUser                                   Removes the user from the current database.
        dropAllUsers                               Removes all users from the current database.
        auth                                       Allows a user to authenticate to the database from within the shell.
        grantRolesToUser                           Grants additional roles to a user.
        revokeRolesFromUser                        Removes a one or more roles from a user on the current database.
        getUser                                    Returns user information for a specified user. Run this method on the user’s database. The user must exist on the database on which the method runs.
        getUsers                                   Returns information for all the users in the database.
        createCollection                           Create new collection
        createView                                 Create new view
        createRole                                 Creates a new role.
        updateRole                                 Updates the role’s profile on the database on which you run the method. An update to a field completely replaces the previous field’s values.
        dropRole                                   Removes the role from the current database.
        dropAllRoles                               Removes all roles from the current database.
        grantRolesToRole                           Grants additional roles to a role.
        revokeRolesFromRole                        Removes a one or more roles from a role on the current database.
        grantPrivilegesToRole                      Grants additional privileges to a role.
        revokePrivilegesFromRole                   Removes a one or more privileges from a role on the current database.
        getRole                                    Returns role information for a specified role. Run this method on the role’s database. The role must exist on the database on which the method runs.
        getRoles                                   Returns information for all the roles in the database.
        currentOp                                  Calls the currentOp command. Returns a document that contains information on in-progress operations for the database instance. The db.currentOp() method wraps the database command currentOp.
        killOp                                     Calls the killOp command. Terminates an operation as specified by the operation ID. To find operations and their corresponding IDs, see $currentOp or db.currentOp().
        shutdownServer                             Calls the shutdown command. Shuts down the current mongod or mongos process cleanly and safely. You must issue the db.shutdownServer() operation against the admin database.
        fsyncLock                                  Calls the fsync command. Forces the mongod to flush all pending write operations to disk and locks the entire mongod instance to prevent additional writes until the user releases the lock with a corresponding db.fsyncUnlock() command.
        fsyncUnlock                                Calls the fsyncUnlock command. Reduces the lock taken by db.fsyncLock() on a mongod instance by 1.
        version                                    returns the db version. uses the buildinfo command
        serverBits                                 returns the db serverBits. uses the buildInfo command
        isMaster                                   Calls the isMaster command
        hello                                      Calls the hello command
        serverBuildInfo                            returns the db serverBuildInfo. uses the buildInfo command
        serverStatus                               returns the server stats. uses the serverStatus command
        stats                                      returns the db stats. uses the dbStats command
        hostInfo                                   Calls the hostInfo command
        serverCmdLineOpts                          returns the db serverCmdLineOpts. uses the getCmdLineOpts command
        rotateCertificates                         Calls the rotateCertificates command
        printCollectionStats                       Prints the collection.stats for each collection in the db.
        getFreeMonitoringStatus                    Calls the getFreeMonitoringStatus command
        disableFreeMonitoring                      returns the db disableFreeMonitoring. uses the setFreeMonitoring command
        enableFreeMonitoring                       returns the db enableFreeMonitoring. uses the setFreeMonitoring command
        getProfilingStatus                         returns the db getProfilingStatus. uses the profile command
        setProfilingLevel                          returns the db setProfilingLevel. uses the profile command
        setLogLevel                                returns the db setLogLevel. uses the setParameter command
        getLogComponents                           returns the db getLogComponents. uses the getParameter command
        cloneDatabase                              deprecated, non-functional
        cloneCollection                            deprecated, non-functional
        copyDatabase                               deprecated, non-functional
        commandHelp                                returns the db commandHelp. uses the passed in command with help: true
        listCommands                               Calls the listCommands command
        getLastErrorObj                            Calls the getLastError command
        getLastError                               Calls the getLastError command
        printShardingStatus                        Calls sh.status(verbose)
        printSecondaryReplicationInfo              Prints secondary replicaset information
        getReplicationInfo                         Returns replication information
        printReplicationInfo                       Formats sh.getReplicationInfo
        printSlaveReplicationInfo                  DEPRECATED. Use db.printSecondaryReplicationInfo
        setSecondaryOk                             This method is deprecated. Use db.getMongo().setReadPref() instead
        watch                                      Opens a change stream cursor on the database
        sql                                        (Experimental) Runs a SQL query against Atlas Data Lake. Note: this is an experimental feature that may be subject to change in future releases.
    ```
    
- `db.foo.help()`: 컬렉션 수준의 도움말
    
    ```bash
    myDB> db.foo.help()
    
      Collection Class:
    
        aggregate                                  Calculates aggregate values for the data in a collection or a view.
        bulkWrite                                  Performs multiple write operations with controls for order of execution.
        count                                      Returns the count of documents that would match a find() query for the collection or view.
        countDocuments                             Returns the count of documents that match the query for a collection or view.
        deleteMany                                 Removes all documents that match the filter from a collection.
        deleteOne                                  Removes a single document from a collection.
        distinct                                   Finds the distinct values for a specified field across a single collection or view and returns the results in an array.
        estimatedDocumentCount                     Returns the count of all documents in a collection or view.
        find                                       Selects documents in a collection or view.
        findAndModify                              Modifies and returns a single document.
        findOne                                    Selects documents in a collection or view.
        renameCollection                           Renames a collection.
        findOneAndDelete                           Deletes a single document based on the filter and sort criteria, returning the deleted document.
        findOneAndReplace                          Modifies and replaces a single document based on the filter and sort criteria.
        findOneAndUpdate                           Updates a single document based on the filter and sort criteria.
        insert                                     Inserts a document or documents into a collection.
        insertMany                                 Inserts multiple documents into a collection.
        insertOne                                  Inserts a document into a collection.
        isCapped                                   Checks if a collection is capped
        remove                                     Removes documents from a collection.
        replaceOne                                 Replaces a single document within the collection based on the filter.
        update                                     Modifies an existing document or documents in a collection.
        updateMany                                 Updates all documents that match the specified filter for a collection.
        updateOne                                  Updates a single document within the collection based on the filter.
        compactStructuredEncryptionData            Compacts structured encryption data
        convertToCapped                            calls {convertToCapped:'coll', size:maxBytes}} command
        createIndexes                              Creates one or more indexes on a collection
        createIndex                                Creates one index on a collection
        ensureIndex                                Creates one index on a collection
        getIndexes                                 Returns an array that holds a list of documents that identify and describe the existing indexes on the collection.
        getIndexSpecs                              Alias for getIndexes. Returns an array that holds a list of documents that identify and describe the existing indexes on the collection.
        getIndices                                 Alias for getIndexes. Returns an array that holds a list of documents that identify and describe the existing indexes on the collection.
        getIndexKeys                               Return an array of key patterns for indexes defined on collection
        dropIndexes                                Drops the specified index or indexes (except the index on the _id field) from a collection.
        dropIndex                                  Drops or removes the specified index from a collection.
        totalIndexSize                             Reports the total size used by the indexes on a collection.
        reIndex                                    Rebuilds all existing indexes on a collection.
        getDB                                      Get current database.
        getMongo                                   Returns the Mongo object.
        dataSize                                   This method provides a wrapper around the size output of the collStats (i.e. db.collection.stats()) command.
        storageSize                                The total amount of storage allocated to this collection for document storage.
        totalSize                                  The total size in bytes of the data in the collection plus the size of every index on the collection.
        drop                                       Removes a collection or view from the database.
        exists                                     Returns collection infos if the collection exists or null otherwise.
        getFullName                                Returns the name of the collection prefixed with the database name.
        getName                                    Returns the name of the collection.
        runCommand                                 Runs a db command with the given name where the first param is the collection name.
        explain                                    Returns information on the query plan.
        stats                                      Returns statistics about the collection.
        latencyStats                               returns the $latencyStats aggregation for the collection. Takes an options document with an optional boolean 'histograms' field.
        initializeOrderedBulkOp                    Initializes an ordered bulk command. Returns an instance of Bulk
        initializeUnorderedBulkOp                  Initializes an unordered bulk command. Returns an instance of Bulk
        getPlanCache                               Returns an interface to access the query plan cache for a collection. The interface provides methods to view and clear the query plan cache.
        mapReduce                                  Calls the mapReduce command
        validate                                   Calls the validate command. Default full value is false
        getShardVersion                            Calls the getShardVersion command
        getShardDistribution                       Prints the data distribution statistics for a sharded collection.
        watch                                      Opens a change stream cursor on the collection
        hideIndex                                  Hides an existing index from the query planner.
        unhideIndex                                Unhides an existing index from the query planner.
    ```
    

- 함수의 기능을 알고 싶으면 함수명을 괄호 없이 입력하면 된다 → 자바스크립트 소스코드가 출력된다.
    
    ```bash
    myDB> db.movies.updateOne
    [Function: updateOne] AsyncFunction {
      apiVersions: [ 1, Infinity ],
      serverVersions: [ '3.2.0', '999.999.999' ],
      returnsPromise: true,
      topologies: [ 'ReplSet', 'Sharded', 'LoadBalanced', 'Standalone' ],
      returnType: { type: 'unknown', attributes: {} },
      deprecated: false,
      platforms: [ 'Compass', 'Browser', 'CLI' ],
      isDirectShellCommand: false,
      acceptsRawInput: false,
      shellCommandCompleter: undefined,
      help: [Function (anonymous)] Help
    }
    ```
    

### 7-2. 셸에서 스크립트 실행하기

---

- 자바스크립트 파일을 셸로 전달해 실행할 수도 있다. mongo 셸은 각 스크립트를 실행하고 빠져나온다.
    
    ```bash
    $ mongosh script1.js script2.js script3.js
    Current Mongosh Log ID:	634d877c189f8c65c96b0764
    Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.6.0
    Using MongoDB:		6.0.1
    Using Mongosh:		1.6.0
    
    For mongosh info see: https://docs.mongodb.com/mongodb-shell/
    
    ------
       The server generated these startup warnings when booting
       2022-10-18T01:32:11.422+09:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
    ------
    
    ------
       Enable MongoDB's free cloud-based monitoring service, which will then receive and display
       metrics about your deployment (disk utilization, CPU, operation statistics, etc).
    
       The monitoring data will be available on a MongoDB website with a unique URL accessible to you
       and anyone you share the URL with. MongoDB may use this information to make product
       improvements and to suggest MongoDB products and deployment options to you.
    
       To enable free monitoring, run the following command: db.enableFreeMonitoring()
       To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
    ------
    
    Loading file: script1.js
    ..
    ```
    

- 셸 보조자와 대응하는 자바스크립트 용법
    
    
    | 셸 보조자 | 같은 의미의 자바스크립트 |
    | --- | --- |
    | use video | db.getSisterDB("video") |
    | show dbs | db.getMongo().getDBs() |
    | show collections | db.getCollectionNames() |
- 스크립트를 사용해 셸에 변수를 입력할 수도 있다.
    
    ```bash
    // defineConnectTo.js
    /**
     * 데이터베이스에 연결하고 db를 설정
     */
    var connectTo = function(port, dbname) {
        if (!port) {
            port = 27017;
        }
    
        if (!dbname) {
            dbname = "test";
        }
    
        db = connect("localhost:"+port+"/"+dbname);
        return db;
    }
    
    $ mongosh
    ..
    
    test> typeof connectTo
    undefined
    test> load('defineConnectTo.js') # 스크립트를 셸에 로드
    true
    test> typeof connectTo
    function
    test>
    ```
    
- 기본적으로 셸은 셸을 시작한 디렉터리에서 스크립트를 찾는다.(셸을 시작한 디렉터리가 어느 것인지 확인하려면 `run("pwd")` 명령어를 사용한다)
    - `mongosh` 내 네이티브 메소드들 ([reference](https://www.mongodb.com/docs/v6.0/reference/method/js-native/#std-label-native-in-mongosh))
        
        ```bash
        test> process.cwd()
        /Users/kim-yoonhee
        ```
        

### 7-3. `.mongorc.js` 만들기

---

- 자주 로드되는 스크립트를 (셸이 시작할 때마다 실행되는) `.mongorc.js` 파일에 넣을 수 있다.
- `mongosh`의 경우, `.mongoshrc.js`다.
    - ex> 홈 디렉터리에 파일을 만들고 셸을 시작할 때마다 문구가 뜬다.
        
        ```bash
        // .mongoshrc.js
        var compliment = ["attractive", "intelligent", "like Batman"];
        var index = Math.floor(Math.random()*3);
        
        print("Hello, you're looking particularly "+compliment[index]+" today!");
        
        $ mongosh
        ..
        Hello, you're looking particularly attractive today!
        ```
        

- 이 스크립트로 사용하고 싶은 전역 변수를 설정하고, 긴 별칭을 짧게 만들고, 내장 함수를 재정의한다.
- 일반적인 용법 중 하나는 더 '위험한' 셸 보조자를 제거하는 것이다.
    - `dropDatabase`나 `deleteIndexes` 같은 함수가 아무것도 수행하지 않게 재정의하거나 모두 선언 해제한다.
        
        ```jsx
        var no = function() {
            print("Mot on my watch.");
        };
        
        // 데이터베이스 삭제 방지
        db.dropDatabase = DB.prototype.dropDatabase = no;
        
        // 컬렉션 삭제 방지
        DBCollection.prototype.drop = no;
        
        // 인덱스 삭제 방지
        DBCollection.prototype.dropIndex = no;
        
        // 인덱스 삭제 방지
        DBCollection.prototype.dropIndexes = no;
        ```
        
- 셸을 시작할 때 `—-norc` 옵션을 사용해 `.mongoshrc.js` 로딩을 비활성화할 수 있다.

- 참고
    - [https://www.mongodb.com/docs/mongodb-shell/install/](https://www.mongodb.com/docs/mongodb-shell/install/)
    - [https://devbksheen.tistory.com/m/entry/몽고DB의-데이터형](https://devbksheen.tistory.com/m/entry/%EB%AA%BD%EA%B3%A0DB%EC%9D%98-%EB%8D%B0%EC%9D%B4%ED%84%B0%ED%98%95)
    - [https://jackjeong.tistory.com/m/166](https://jackjeong.tistory.com/m/166)