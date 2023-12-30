# Chapter 3. 도큐먼트 생성, 갱신, 삭제

- docker 로 mongo 재설치
    
    ```bash
    $ cd ~
    $ docker pull mongo
    $ mkdir mongodb
    
    # 몽고DB 컨테이너의 볼륨을 로컬 디렉터리와 마운트시키지 않으면 컨테이너를 삭제할 때
    # 컨테이너에 저장되어있는 데이터도 삭제되기 때문에 복구할 수 없다.
    # 몽고DB의 기본 데이터 디렉터리는 /data/db
    # -v ~/mongodb:data/db을 사용해 로컬의 ~/mongodb 디렉터리와 마운트시킴
    $ docker run --name mongodb -v ~/mongodb:/data/db -d -p 27017:27017 mongo
    ab32e314d71559a912b796b0cb7b8c88ee8655da307ef3121204fee4c97820a4
    
    $ docker ps
    CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                               NAMES
    ab32e314d715   mongo          "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:27017->27017/tcp            mongodb
    
    $ docker exec -it mongodb bash
    root@ab32e314d715:/# mongod --version
    db version v6.0.2
    Build Info: {
        "version": "6.0.2",
        "gitVersion": "94fb7dfc8b974f1f5343e7ea394d0d9deedba50e",
        "openSSLVersion": "OpenSSL 1.1.1f  31 Mar 2020",
        "modules": [],
        "allocator": "tcmalloc",
        "environment": {
            "distmod": "ubuntu2004",
            "distarch": "aarch64",
            "target_arch": "aarch64"
        }
    }
    
    root@ab32e314d715:/# mongosh
    Current Mongosh Log ID:	635beb2154ccc42b82b1a853
    Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.6.0
    Using MongoDB:		6.0.2
    Using Mongosh:		1.6.0
    
    For mongosh info see: https://docs.mongodb.com/mongodb-shell/
    
    To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
    You can opt-out by running the disableTelemetry() command.
    
    ------
       The server generated these startup warnings when booting
       2022-10-28T14:45:15.812+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
       2022-10-28T14:45:15.812+00:00: vm.max_map_count is too low
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
    

## 1. 도큐먼트 삽입

---

- 삽입: 몽고DB에 데이터를 추가하는 기본 방법
- `insertOne` 메서드: 도큐먼트를 삽입
    
    ```bash
    test> db.movies.insertOne({"title" : "Stand by Me"})
    {
      acknowledged: true,
      insertedId: ObjectId("635beca8ecaec72e722b80e7")
    }
    
    test> db.movies.findOne()
    { _id: ObjectId("635beca8ecaec72e722b80e7"), title: 'Stand by Me' }
    ```
    
    - 도큐먼트에 "`_id`" 키가 추가되고(제공하지 않는 경우) 도큐먼트가 몽고DB에 저장된다.

### 1-1. `insertMany`

---

- `insertMany`: 여러 도큐먼트를 컬렉션에 삽입. 도큐먼트 배열을 데이터베이스에 전달한다.
    - 도큐먼트를 대량 삽입하므로 훨씬 더 효율적이다.

```bash
test> db.movies.drop()
true

test> db.movies.insertMany([{"title" : "Ghostbusters"},
...                             {"title" : "E.T."},
...                             {"title" : "Blade Runner"}]);
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("635beeffecaec72e722b80e8"),
    '1': ObjectId("635beeffecaec72e722b80e9"),
    '2': ObjectId("635beeffecaec72e722b80ea")
  }
}

test> db.movies.find()
[
  { _id: ObjectId("635beeffecaec72e722b80e8"), title: 'Ghostbusters' },
  { _id: ObjectId("635beeffecaec72e722b80e9"), title: 'E.T.' },
  { _id: ObjectId("635beeffecaec72e722b80ea"), title: 'Blade Runner' }
]
```

- `insertMany`는 여러 도큐먼트를 단일 컬렉션에 삽입할 때 유용하다.

- 한 번에 일괄 삽입할 수 있는 데이터의 크기에는 제한이 있다.
    - 48MB보다 큰 메시지를 허용하지 않음
        - 삽입된 데이터를 48MB 크기의 일괄 삽입 여러 개로 분할한다.

- 배열 중간에 있는 도큐먼트에서 특정 유형의 오류가 발생하는 경우, 정렬 연산을 선택했는지 혹은 비정렬 연산을 선택했는지에 따라 발생하는 상황이 달라진다.
    - 옵션 도큐먼트에 "`ordered`" 키에 `true`를 지정 (default)
        - 도큐먼트가 제공된 순서대로 삽입됨
        - 삽입 오류를 생성하면, 배열에서 해당 지점을 벗어난 도큐먼트는 삽입되지 않는다.
            
            ```bash
            # "_id"가 동일한 도큐먼트를 두 개 이상 삽입할 수 없으므로 세 번째 도큐먼트가 오류를 발생시킨다.
            test> db.movies.insertMany([
            ... {"_id" : 0, "title" : "Top Gun"},
            ... {"_id" : 1, "title" : "Back to the Future"},
            ... {"_id" : 1, "title" : "Gremlins"},
            ... {"_id" : 2, "title" : "Aliens"}])
            Uncaught:
            MongoBulkWriteError: E11000 duplicate key error collection: test.movies index: _id_ dup key: { _id: 1 }
            Result: BulkWriteResult {
              result: {
                ok: 1,
                writeErrors: [
                  WriteError {
                    err: {
                      index: 2,
                      code: 11000,
                      errmsg: 'E11000 duplicate key error collection: test.movies index: _id_ dup key: { _id: 1 }',
                      errInfo: undefined,
                      op: { _id: 1, title: 'Gremlins' }
                    }
                  }
                ],
                writeConcernErrors: [],
                insertedIds: [
                  { index: 0, _id: 0 },
                  { index: 1, _id: 1 },
                  { index: 2, _id: 1 },
                  { index: 3, _id: 2 }
                ],
                nInserted: 2,
                nUpserted: 0,
                nMatched: 0,
                nModified: 0,
                nRemoved: 0,
                upserted: []
              }
            }
            ```
            
    - `false`
        - 몽고DB가 성능을 개선하려고 삽입을 재배열할 수 있다.
        - 일부 삽입이 오류를 발생시키는지 여부에 관계없이 모든 도큐먼트 삽입을 시도한다.
            
            ```bash
            # 세 번째 도큐먼트만 중복된 "_id" 오류 때문에 삽입에 실패했다.
            test> db.movies.insertMany([
            ... {"_id" : 3, "title" : "Sixteen Candles"},
            ... {"_id" : 4, "title" : "The Terminator"},
            ... {"_id" : 4, "title" : "The Princess Bride"},
            ... {"_id" : 5, "title" : "Scarface"}],
            ... {"ordered" : false})
            Uncaught:
            MongoBulkWriteError: E11000 duplicate key error collection: test.movies index: _id_ dup key: { _id: 4 }
            Result: BulkWriteResult {
              result: {
                ok: 1,
                writeErrors: [
                  WriteError {
                    err: {
                      index: 2,
                      code: 11000,
                      errmsg: 'E11000 duplicate key error collection: test.movies index: _id_ dup key: { _id: 4 }',
                      errInfo: undefined,
                      op: { _id: 4, title: 'The Princess Bride' }
                    }
                  }
                ],
                writeConcernErrors: [],
                insertedIds: [
                  { index: 0, _id: 3 },
                  { index: 1, _id: 4 },
                  { index: 2, _id: 4 },
                  { index: 3, _id: 5 }
                ],
                nInserted: 3,
                nUpserted: 0,
                nMatched: 0,
                nModified: 0,
                nRemoved: 0,
                upserted: []
              }
            }
            ```
            

### 1-2. 삽입 유효성 검사

---

- 몽고DB는 삽입된 데이터에 최소한의 검사를 수행한다.
    - "`_id`" 필드가 존재하지 않으면 새로 추가
    - 모든 도큐먼트는 16MB보다 작아야 하므로 크기를 검사
        - doc라는 도큐먼트의 Binary JSON(BSON) 크기(바이트 단위)를 보려면 `bsonsize(doc)`를 실행한다.
            
            ```bash
            test> doc = {subject: "MongoDB", name: "The definitive guide of MongoDB 3e"}
            { subject: 'MongoDB', name: 'The definitive guide of MongoDB 3e' }
            
            test> bsonsize(doc)
            71
            ```
            

### 1-3. 삽입

---

- 도큐먼트를 몽고DB에 삽입
    - 몽고DB 3.0 이전 버전: `insert`
    - 몽고DB 3.0 ~ : `insertOne`, `insertMany`뿐 아니라 다른 여러 방법

## 2. 도큐먼트 삭제

---

- `deleteOne`: 필터와 일치하는 첫 번째 도큐먼트를 삭제한다.
    
    ```bash
    test> db.movies.find()
    [
      { _id: ObjectId("635beeffecaec72e722b80e8"), title: 'Ghostbusters' },
      { _id: ObjectId("635beeffecaec72e722b80e9"), title: 'E.T.' },
      { _id: ObjectId("635beeffecaec72e722b80ea"), title: 'Blade Runner' },
      { _id: 0, title: 'Top Gun' },
      { _id: 1, title: 'Back to the Future' },
      { _id: 3, title: 'Sixteen Candles' },
      { _id: 4, title: 'The Terminator' },
      { _id: 5, title: 'Scarface' }
    ]
    
    test> db.movies.deleteOne({"_id" : 4})
    { acknowledged: true, deletedCount: 1 }
    
    test> db.movies.find()
    [
      { _id: ObjectId("635beeffecaec72e722b80e8"), title: 'Ghostbusters' },
      { _id: ObjectId("635beeffecaec72e722b80e9"), title: 'E.T.' },
      { _id: ObjectId("635beeffecaec72e722b80ea"), title: 'Blade Runner' },
      { _id: 0, title: 'Top Gun' },
      { _id: 1, title: 'Back to the Future' },
      { _id: 3, title: 'Sixteen Candles' },
      { _id: 5, title: 'Scarface' }
    ]
    ```
    
- `deleteMany`: 필터와 일치하는 모든 도큐먼트를 삭제
    
    ```bash
    test> db.movies.drop()
    true
    
    test> db.movies.insertMany([
    ... {"_id" : 0, "title" : "Top Gun", "year" : 1986},
    ... {"_id" : 1, "title" : "Back to the Future", "year" : 1985},
    ... {"_id" : 3, "title" : "Sixteen Candles", "year" : 1984},
    ... {"_id" : 4, "title" : "The Terminator", "year" : 1984},
    ... {"_id" : 5, "title" : "Scarface", "year" : 1983}])
    {
      acknowledged: true,
      insertedIds: { '0': 0, '1': 1, '2': 3, '3': 4, '4': 5 }
    }
    
    test> db.movies.deleteMany({"year" : 1984})
    { acknowledged: true, deletedCount: 2 }
    
    test> db.movies.find()
    [
      { _id: 0, title: 'Top Gun', year: 1986 },
      { _id: 1, title: 'Back to the Future', year: 1985 },
      { _id: 5, title: 'Scarface', year: 1983 }
    ]
    
    test> db.movies.deleteMany({})
    { acknowledged: true, deletedCount: 3 }
    ```
    

- 도큐먼트를 삭제
    - 몽고DB 3.0 이전 버전: `remove`
    - 몽고DB 3.0 ~ : `deleteOne`과 `deleteMany` 등 여러 메서드

### 2-1. `drop`

---

- `drop`: 전체 컬렉션을 삭제. 그리고 빈 컬렉션에 인덱스를 재생성한다.
    
    ```bash
    test> db.movies.drop()
    true
    ```
    

<aside>
💡 이전에 백업된 데이터를 복원하는 방법 외에 `delete` 또는 `drop` 작업을 취소하거나 삭제된 도큐먼트를 복구하는 방법은 없다.

</aside>

## 3. 도큐먼트 갱신

---

- `updateOne`
- `updateMany`
- `replaceOne`

- 갱신은 원자적으로 이뤄진다.
    - 요청 두 개가 동시에 발생하면 서버에 먼저 도착한 요청이 적용된 후 다음 요청이 적용된다.
    - 기본 동작을 원치 않으면 도큐먼트 버저닝 패턴을 고려하자.

### 3-1. 도큐먼트 치환

---

- `replaceOne`: 도큐먼트를 새로운 것으로 완전히 치환한다.
    - 대대적인 스키마 마이그레이션에 유용하다.
- ex> 사용자 도큐먼트를 다음과 같이 큰 규모로 변경한다고 가정
    
    ```bash
    test> db.users.insertOne({"name": "joe", "friends": 32, "enemies": 2});
    
    test> db.users.findOne()
    {
      _id: ObjectId("635e227cd6ae359ae0982415"),
      name: 'joe',
      friends: 32,
      enemies: 2
    }"
    ```
    
    - "friends"와 "enemies" 필드를 "relationship"라는 서브도큐먼트로 옮겨보자.
    - 셸에서 도큐먼트의 구조를 수정한 후 `replaceOne`을 사용해 데이터베이스의 버전을 교체한다.
        
        ```bash
        test> var joe = db.users.findOne({"name":"joe"});
        
        test> joe.relationship = {"friends": joe.friends, "enemies": joe.enemies};
        { friends: 32, enemies: 2 }
        
        test> joe.username = joe.name;
        joe
        
        test> delete joe.friends;
        true
        test> delete joe.enemies;
        true
        test> delete joe.name;
        true
        
        test> db.users.replaceOne({"name":"joe"}, joe);
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        ```
        
    - `findOne` 함수를 실행하면 갱신된 도큐먼트의 구조를 보여준다.
        
        ```bash
        test> db.users.findOne()
        {
          _id: ObjectId("635e227cd6ae359ae0982415"),
          relationship: { friends: 32, enemies: 2 },
          username: 'joe'
        }
        ```
        

### 3-2. 갱신 연산자

---

- 부분 갱신에는 원자적 갱신 연산자를 사용한다.
    - 갱신 연산자: 키를 변경, 추가, 제거하고, 심지어 배열과 내장 도큐먼트를 조작하는 복잡한 갱신 연산을 지정하는 데 사용하는 특수키

- "`$inc`" 제한자 → 아래에 자세히 나온다.
    
    ```bash
    test> db.analytics.insertOne({"url": "www.example.com", "pageviews": 52});
    
    test> db.analytics.updateOne({"url": "www.example.com"},
    ...{"$inc": {"pageviews": 1}})
    {
      acknowledged: true,
      insertedId: null,
      matchedCount: 1,
      modifiedCount: 1,
      upsertedCount: 0
    }
    
    test> db.analytics.findOne()
    {
      _id: ObjectId("635e26c6d6ae359ae0982417"),
      url: 'www.example.com',
      pageviews: 53
    }
    ```
    

- "`$set`" 제한자 사용하기: 필드 값을 설정한다. 필드가 존재하지 않으면 새 필드가 생성된다.
    - 스키마를 갱신하거나 사용자 정의 키를 추가할 때 편리하다.
        
        ```bash
        test> db.users.drop();
        true
        
        test> db.users.insertOne(
        ...     {
        ...         "name": "joe",
        ...         "age": 30,
        ...         "sex": "male",
        ...         "location": "Wisconsin"
        ...     }
        ... );
        
        test> db.users.updateOne(
        ...     {"name": "joe"},
        ...     {"$set": {"favorite book": "War and Peace"}}
        ... );
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        
        test> db.users.findOne()
        {
          _id: ObjectId("635e27d2d6ae359ae0982418"),
          name: 'joe',
          age: 30,
          sex: 'male',
          location: 'Wisconsin',
          'favorite book': 'War and Peace'
        }
        
        test> db.users.updateOne(
        ...     {"name": "joe"},
        ...     {"$set": {"favorite book": "Green Eggs and Ham"}}
        ... );
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        ```
        
    - 키의 데이터형도 변경할 수 있다.
        
        ```bash
        test> db.users.updateOne(
        ...     {"name": "joe"},
        ...     {"$set": {"favorite book":["Cat's Cradle", "Foundation Trilogy", "Ender's Game"]}}
        ... );
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        
        test> db.users.findOne()
        {
          _id: ObjectId("635e27d2d6ae359ae0982418"),
          name: 'joe',
          age: 30,
          sex: 'male',
          location: 'Wisconsin',
          'favorite book': [ "Cat's Cradle", 'Foundation Trilogy', "Ender's Game" ]
        }
        ```
        
    - "`$unset`": 키와 값을 모두 제거한다.
        
        ```bash
        test> db.users.updateOne(
        ...     {"name": "joe"},
        ...     {"$unset": {"favorite book":1}}
        ... );
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        
        test> db.users.findOne()
        {
          _id: ObjectId("635e27d2d6ae359ae0982418"),
          name: 'joe',
          age: 30,
          sex: 'male',
          location: 'Wisconsin'
        }
        ```
        
    - 내장 도큐먼트 내부의 데이터를 변경
        
        ```bash
        test> db.blog.posts.insertOne(
        ... {
        ...     "title": "A Blog Post",
        ...     "content": "Hello MongoDB!",
        ...     "author": {
        ...         "name": "joe",
        ...         "email": "joe@example.com",
        ...     }
        ... });
        
        test> db.blog.posts.updateOne({"author.name": "joe"},
        ... {"$set": {"author.name": "joe schmoe"}})
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 1,
          upsertedCount: 0
        }
        
        test> db.blog.posts.findOne()
        {
          _id: ObjectId("635e2d44d6ae359ae0982419"),
          title: 'A Blog Post',
          content: 'Hello MongoDB!',
          author: { name: 'joe schmoe', email: 'joe@example.com' }
        }
        ```
        
    
    <aside>
    💡 키를 추가, 변경, 삭제할 때는 항상 `$` 제한자를 사용해야 한다.
    
    </aside>
    

- 증가와 감소
    - "`$inc`" 연산자: 이미 존재하는 키의 값을 변경하거나 새 키를 생성하는데 사용한다.
        - 분석, 분위기, 투표 등과 같이 자주 변하는 수치 값을 갱신하는 데 매우 유용하다.
            
            ```bash
            # 게임을 저장하고 점수를 갱신하는 게임 컬렉션
            test> db.games.insertOne({"game": "pinball", "user": "joe"})
            
            # 플레이어의 점수가 증가. 기본 단위가 50
            test> db.games.updateOne({"game": "pinball", "user": "joe"},
            ... {"$inc": {"score": 50}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            test> db.games.findOne()
            {
              _id: ObjectId("6367b875750d5a5b553def82"),
              game: 'pinball',
              user: 'joe',
              score: 50 # "$inc"에 의해 생성되고 50만큼 증가했다.
            }
            ```
            
        - "`$set`"과 비슷하지만 숫자를 증감하기 위해 설계됐다. `int`, `long`, `double`, `decimal` 타입 값에만 사용할 수 있다.
            - 여러 언어에서 숫자로 자동 변환되는 데이터형의 값에는 사용할 수 없다. ex> `null`, 불리언, 문자열로 나타낸 숫자
            
            ```bash
            test> db.strcounts.insert({"count" : "1"})
            DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
            {
              acknowledged: true,
              insertedIds: { '0': ObjectId("6367baa5750d5a5b553def83") }
            }
            
            test> db.strcounts.update({}, {"$inc" : {"count" : 1}})
            DeprecationWarning: Collection.update() is deprecated. Use updateOne, updateMany, or bulkWrite.
            MongoServerError: Cannot apply $inc to a value of non-numeric type. {_id: ObjectId('6367baa5750d5a5b553def83')} has the field 'count' of non-numeric type string
            ```
            
        - "`$inc`"의 키 값은 무조건 숫자여야 한다. 숫자가 아닌 값은 증감할 수 없기 때문이다.
            
            ```bash
            test> db.strcounts.update({}, {"$inc" : {"count" : "1"}})
            MongoServerError: Cannot increment with non-numeric argument: {count: "1"}
            ```
            

- 배열 연산자
    - 배열을 다루는 데 갱신 연산자를 사용할 수 있다. 리스트에 대한 인덱스를 지정할 수 있을 뿐 아니라 셋처럼 이중으로 쓸 수 있다.
    - 요소 추가하기
        - "`$push`": 배열이 이미 존재하면 배열 끝에 요소를 추가하고, 존재하지 않으면 새로운 배열을 생성한다.
            
            ```bash
            test> db.blog.posts.findOne()
            {
              _id: ObjectId("635e2d44d6ae359ae0982419"),
              title: 'A Blog Post',
              content: 'Hello MongoDB!',
              author: { name: 'joe schmoe', email: 'joe@example.com' }
            }
            
            # 블로그 게시물에 배열 형태의 "comments" 키를 삽입
            test> db.blog.posts.updateOne({"title": "A Blog Post"},
            ... {"$push": {"comments" :
            ...     {"name" : "joe", "email" : "joe@example.com",
            ...     "content" : "nice post."}}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            test> db.blog.posts.findOne()
            {
              _id: ObjectId("635e2d44d6ae359ae0982419"),
              title: 'A Blog Post',
              content: 'Hello MongoDB!',
              author: { name: 'joe schmoe', email: 'joe@example.com' },
              comments: [ { name: 'joe', email: 'joe@example.com', content: 'nice post.' } ]
            }
            
            # 댓글을 더 추가
            test> test> db.blog.posts.updateOne({"title": "A Blog Post"},
            ... {"$push" : {"comments" :
            ...     {"name" : "bob", "email" : "bob@example.com",
            ...     "content" : "good post."}}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            test> db.blog.posts.findOne()
            {
              _id: ObjectId("635e2d44d6ae359ae0982419"),
              title: 'A Blog Post',
              content: 'Hello MongoDB!',
              author: { name: 'joe schmoe', email: 'joe@example.com' },
              comments: [
                { name: 'joe', email: 'joe@example.com', content: 'nice post.' },
                { name: 'bob', email: 'bob@example.com', content: 'good post.' }
              ]
            }
            ```
            
        - 몽고DB 쿼리 언어는 "`$push`"를 포함해 일부 연산자에 제한자를 제공한다.
            - "`$each`" 제한자: 작업 한 번으로 값을 여러 개 추가할 수 있다.
                
                ```bash
                test> db.stock.ticker.insertOne({"_id" : "GOOG"})
                { acknowledged: true, insertedId: 'GOOG' }
                
                # 배열에 새로운 요소 세 개를 추가한다.
                test> db.stock.ticker.updateOne({"_id" : "GOOG"},
                ... {"$push" : {"hourly" : {"$each" : [562.776, 562.790, 559.123]}}})
                {
                  acknowledged: true,
                  insertedId: null,
                  matchedCount: 1,
                  modifiedCount: 1,
                  upsertedCount: 0
                }
                
                test> db.stock.ticker.findOne()
                { _id: 'GOOG', hourly: [ 562.776, 562.79, 559.123 ] }
                ```
                
            - "`$slice`": 배열을 특정 길이로 늘림. 배열이 특정 크기 이상으로 늘어나지 않게 하고 효과적으로 '`top N`' 목록을 만들 수 있다. 도큐먼트 내에 큐를 생성하는 데 사용할 수 있다.
                
                ```bash
                # 배열에 추가할 수 있는 요소의 개수를 10개로 제한한다.
                test> db.movies.updateOne({"genre" : "horror"},
                ... {"$push" : {"top10" : {"$each" : ["Nightmare on Elm Street", "Saw"],
                ...                        "$slice" : -10}}})
                ```
                
                - 추가 후에 배열 요소의 개수가 10보다 작으면 모든 요소가 유지되고, 10보다 크면 마지막 10개 요소만 유지된다.
                
                ![Untitled](./image/3/Untitled%201.png)
                
            - "`$sort`": 트리밍하기 전에 적용할 수 있다.(정렬)
                
                ```bash
                # "rating" 필드로 배열의 모든 요소를 정렬한 후 처음 10개의 요소를 유지한다.
                test> db.movies.updateOne({"genre" : "horror"},
                ... {"$push" : {"top10" : {"$each" : [
                ...         {"name" : "Nightmare on Elm Street", "rating" : 6.6},
                ...         {"name" : "Saw", "rating" : 4.3}
                ...     ],
                ...     "$slice" : -10,
                ...     "$sort" : {"rating" : -1}}}})
                ```
                
                ![Untitled](./image/3/Untitled%202.png)
                
        
        <aside>
        💡 "`$slice`"나 "`$sort`"를 배열상에서 "`$push`"와 함께 쓰려면 반드시 "`$each`"도 사용해야 한다.
        
        </aside>
        
    - 배열을 집합으로 사용하기
        - "`$ne`": 특정 값이 배열에 존재하지 않을 때 해당 값을 추가하면서, 배열을 집합처럼 처리
            
            ```bash
            test> db.papers.insertMany( [
            ...     {"authors cited": ["Richie", "Richie2"], "quantity": 30},
            ...     {"authors cited": ["bolts"], "quantity": 50},
            ...     {"authors cited": ["washers"], "quantity": 10}
            ...  ] )
            {
              acknowledged: true,
              insertedIds: {
                '0': ObjectId("6367c629750d5a5b553def8b"),
                '1': ObjectId("6367c629750d5a5b553def8c"),
                '2': ObjectId("6367c629750d5a5b553def8d")
              }
            }
            
            # 인용 목록에 저자가 존재하지 않을 때만 해당 저자를 추가
            test> db.papers.updateOne({"authors cited" : {"$ne" : "Richie"}},
            ... {$push : {"authors cited" : "Richie"}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            test> db.papers.find()
            [
              {
                _id: ObjectId("6367c629750d5a5b553def8b"),
                'authors cited': [ 'Richie', 'Richie2' ],
                quantity: 30
              },
              {
                _id: ObjectId("6367c629750d5a5b553def8c"),
                'authors cited': [ 'bolts', 'Richie' ],
                quantity: 50
              },
              {
                _id: ObjectId("6367c629750d5a5b553def8d"),
                'authors cited': [ 'washers' ],
                quantity: 10
              }
            ]
            ```
            
        - "`$addToSet`": "`$ne`"가 작동하지 않을 때나 "`$addToSet`"을 사용하면 무슨 일이 일어났는지 더 잘 알 수 있을 때 유용하다.
            
            ```bash
            test> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
            {
              _id: ObjectId("4b2d75476cc613d5ee930164"),
              username: 'joe',
              emails: [ 'joe@example.com', 'joe@gmail.com', 'joe@yahoo.com' ]
            }
            
            # 다른 주소를 추가
            # 중복을 피할 수 있다.
            test> db.users.updateOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")},
            ...  {"$addToSet" : {"emails" : "joe@gmail.com"}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 0,
              upsertedCount: 0
            }
            test> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
            {
              _id: ObjectId("4b2d75476cc613d5ee930164"),
              username: 'joe',
              emails: [ 'joe@example.com', 'joe@gmail.com', 'joe@yahoo.com' ]
            }
            
            test> db.users.updateOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")}, {"$addToSet" : {"emails" : "joe@hotmail.com"}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            test> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
            {
              _id: ObjectId("4b2d75476cc613d5ee930164"),
              username: 'joe',
              emails: [
                'joe@example.com',
                'joe@gmail.com',
                'joe@yahoo.com',
                'joe@hotmail.com'
              ]
            }
            ```
            
        - 고유한 값을 여러 개 추가하려면 "`$addToSet`"과 "`$each`"를 결합해 사용한다. "`$ne`"/"`$push`" 조합으로는 할 수 없는 작업이다.
            
            ```bash
            test> db.users.updateOne(
            ...     {"_id" : ObjectId("4b2d75476cc613d5ee930164")},
            ...     {"$addToSet" : {"emails" : {"$each" : ["joe@php.net", "joe@example.com", "joe@python.org"]}}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            test> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
            {
              _id: ObjectId("4b2d75476cc613d5ee930164"),
              username: 'joe',
              emails: [
                'joe@example.com',
                'joe@gmail.com',
                'joe@yahoo.com',
                'joe@hotmail.com',
                'joe@php.net',
                'joe@python.org'
              ]
            }
            ```
            
    - 요소 제거하기
        - "`$pop`": 배열의 양쪽 끝에서 요소를 제거 (배열을 큐나 스택처럼 사용)
            - 마지막부터 요소 제거: `{"$pop": {"key": 1}}`
            - 처음부터 요소 제거: `{"$pop": {"key": -1}}`
        - "`$pull`": 주어진 조건에 맞는 배열 요소를 제거
            
            ```bash
            test> db.lists.insertOne({"todo" : ["dishes", "laundry", "dry cleaning"]})
            
            # 세탁 -> 목록에서 제거
            test> db.lists.updateOne({}, {"$pull" : {"todo" : "laundry"}})
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 1,
              modifiedCount: 1,
              upsertedCount: 0
            }
            
            # 도큐먼트에서 조건과 일치하는 요소를 모두 제거한다.
            test> db.lists.find()
            [
              {
                _id: ObjectId("6367cb1e750d5a5b553def8e"),
                todo: [ 'dishes', 'dry cleaning' ]
              }
            ]
            ```
            
        
        <aside>
        💡 배열 연산자는 배열을 값으로 갖는 키에만 사용한다.
        
        </aside>
        
    - 배열의 위치 기반 변경
        - 위치를 이용
            - 배열 인덱스는 기준이 0이며, 배열 요소는 인덱스를 도큐먼트의 키처럼 사용한다.
                
                ```bash
                test> db.blog.posts.insertOne(
                ... {
                ...   "_id": ObjectId("4b329a216cc613d5ee930192"),
                ...   "content": "...",
                ...   "comments": [
                ...     {"comment": "good!", "author": "John", "votes": 0},
                ...     {"comment": "too short!", "author": "Claire", "votes": 3},
                ...     {"comment": "free", "author": "Alice", "votes": -5},
                ...     {"comment": "vacation", "author": "Lynn", "votes": -7}
                ...   ]
                ... })
                
                # 첫 번째 댓글의 투표수를 증가시키기
                test> db.blog.posts.updateOne({"_id" : ObjectId("4b329a216cc613d5ee930192")}, {"$inc" : {"comments.0.votes" : 1}})
                {
                  acknowledged: true,
                  insertedId: null,
                  matchedCount: 1,
                  modifiedCount: 1,
                  upsertedCount: 0
                }
                
                test> db.blog.posts.findOne({"_id": ObjectId("4b329a216cc613d5ee930192")})
                {
                  _id: ObjectId("4b329a216cc613d5ee930192"),
                  content: '...',
                  comments: [
                    { comment: 'good!', author: 'John', votes: 1 },
                    { comment: 'too short!', author: 'Claire', votes: 3 },
                    { comment: 'free', author: 'Alice', votes: -5 },
                    { comment: 'vacation', author: 'Lynn', votes: -7 }
                  ]
                }
                ```
                
        - 위치 연산자("`$`" 문자)를 사용
            - 몽고DB에서는 쿼리 도큐먼트와 일치하는 배열 요소 및 요소의 위치를 알아내서 갱신하는 위치 연산자 "`$`"를 제공한다.
                
                ```bash
                test> db.blog.posts.updateOne({"comments.author" : "John"}, {"$set" : {"comments.$.author" : "Jim"}})
                {
                  acknowledged: true,
                  insertedId: null,
                  matchedCount: 1,
                  modifiedCount: 1,
                  upsertedCount: 0
                }
                
                test> db.blog.posts.findOne({"_id": ObjectId("4b329a216cc613d5ee930192")})
                {
                  _id: ObjectId("4b329a216cc613d5ee930192"),
                  content: '...',
                  comments: [
                    { comment: 'good!', author: 'Jim', votes: 1 },
                    { comment: 'too short!', author: 'Claire', votes: 3 },
                    { comment: 'free', author: 'Alice', votes: -5 },
                    { comment: 'vacation', author: 'Lynn', votes: -7 }
                  ]
                }
                ```
                
    - 배열 필터를 이용한 갱신
        - `arrayFilters`: 개별 배열 요소를 갱신하는 배열 필터. 특정 조건에 맞는 배열 요소를 갱신할 수 있다.
            - ex> 반대표가 5표 이상인 댓글을 숨기기
                
                ```bash
                db.blog.updateOne(
                    {"post" : post_id },
                    { $set: { "comments.$[elem].hidden" : true } },
                    {
                      arrayFilters: [ { "elem.votes": { $lte: -5 } }]
                    }
                )
                ```
                
                - "comments" 배열의 각 일치 요소에 대한 식별자로 elem을 정의한다. elem이 식별한 댓글의 votes값이 -5 이하면 "comments" 도큐먼트에 "hidden" 필드를 추가하고 값을 true로 설정한다.

### 3-3. 갱신 입력

---

- 갱신 입력은 특수한 형태를 갖는 갱신이다.
    - 갱신 조건에 맞는 도큐먼트가 존재하지 않을 때는 쿼리 도큐먼트와 갱신 도큐먼트를 합쳐서 새로운 도큐먼트를 생성한다.
    - 조건에 맞는 도큐먼트가 발견되면 일반적인 갱신을 수행한다.

- 갱신 입력을 사용하면 코드를 줄이고 경쟁 상태를 피할 수 있다. 더 빠르고 원자적이다.
    
    ```bash
    test> db.analytics.updateOne({"url" : "/blog"}, {"$inc" : {"pageviews" : 1}}, {"upsert" : true})
    {
      acknowledged: true,
      insertedId: ObjectId("6367d1bfe061103153fc2184"),
      matchedCount: 0,
      modifiedCount: 0,
      upsertedCount: 1
    }
    
    test> db.analytics.findOne({"_id": ObjectId("6367d1bfe061103153fc2184")})
    {
      _id: ObjectId("6367d1bfe061103153fc2184"),
      url: '/blog',
      pageviews: 1
    }
    ```
    
    - 새로운 도큐먼트는 조건 도큐먼트에 도큐먼트 제한자를 적용해 만들어진다.
        
        ```bash
        test> db.users.updateOne({"rep" : 25}, {"$inc" : {"rep" : 3}}, {"upsert" : true})
        {
          acknowledged: true,
          insertedId: ObjectId("6367d266e061103153fc21a9"),
          matchedCount: 0,
          modifiedCount: 0,
          upsertedCount: 1
        }
        
        test> db.users.findOne({"_id": ObjectId("6367d266e061103153fc21a9")})
        { _id: ObjectId("6367d266e061103153fc21a9"), rep: 28 }
        ```
        

- "`$setOnInsert`": 도큐먼트가 삽입될 때 필드 값을 설정하는 데만 사용하는 연산자
    - 검증 예제
        
        ```bash
        test> db.users.updateOne({}, {"$setOnInsert" : {"createdAt" : new Date()}}, {"upsert" : true})
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 0,
          upsertedCount: 0
        }
        # 변화가 없음.. updateMany도 마찬가지!
        test> db.users.find()
        [
          {
            _id: ObjectId("635e27d2d6ae359ae0982418"),
            name: 'joe',
            age: 30,
            sex: 'male',
            location: 'Wisconsin'
          },
          {
            _id: ObjectId("4b2d75476cc613d5ee930164"),
            username: 'joe',
            emails: [
              'joe@example.com',
              'joe@gmail.com',
              'joe@yahoo.com',
              'joe@hotmail.com',
              'joe@php.net',
              'joe@python.org'
            ]
          },
          { _id: ObjectId("6367d266e061103153fc21a9"), rep: 28 }
        ]
        ```
        
    
    ```bash
    
    # _id 지정하여 실행
    test> db.users.updateOne({"_id": ObjectId("5727b4ac223502483c7f3ace")}, {"$setOnInsert" : {"createdAt" : new Date()}}, {"upsert" : true})
    {
      acknowledged: true,
      insertedId: ObjectId("5727b4ac223502483c7f3ace"),
      matchedCount: 0,
      modifiedCount: 0,
      upsertedCount: 1
    }
    
    test> db.users.findOne({"_id": ObjectId("5727b4ac223502483c7f3ace")})
    {
      _id: ObjectId("5727b4ac223502483c7f3ace"),
      createdAt: ISODate("2022-11-06T15:40:44.263Z")
    }
    ```
    
    - 다시 갱신하면 기존 도큐먼트를 찾고, 아무것도 입력되지 않으며, "createdAt" 필드는 변경되지 않는다.
        
        ```bash
        test> db.users.updateOne({"_id": ObjectId("5727b4ac223502483c7f3ace")}, {"$setOnInsert" : {"createdAt" : new Date()}}, {"upsert" : true})
        {
          acknowledged: true,
          insertedId: null,
          matchedCount: 1,
          modifiedCount: 0,
          upsertedCount: 0
        }
        ```
        

- 저장 셀 보조자
    - `save`: 도큐먼트가 존재하지 않으면 도큐먼트를 삽입하고, 존재하면 도큐먼트를 갱신하게 하는 셸 함수
        - 셀 함수는 매개변수가 하나이며 도큐먼트를 넘겨받는다.
        - 5.0 버전부터는 없어짐 ([레퍼런스](https://www.mongodb.com/docs/manual/reference/method/js-collection/))
        - 도큐먼트가 "`_id`" 키를 포함하면 `save`는 갱신 입력을 실행하고, 포함하지 않으면 삽입을 실행한다.
            
            ```bash
            test> db.testcalls.insertOne({"aaaa" : "..."})
            
            test> var x = db.testcalls.findOne()
            
            test> x.num = 42
            42
            test> db.testcalls.save(x)
            TypeError: db.testcalls.save is not a function
            
            # save가 없다면
            test> db.testcol.replaceOne({"_id" : x._id}, x)
            {
              acknowledged: true,
              insertedId: null,
              matchedCount: 0,
              modifiedCount: 0,
              upsertedCount: 0
            }
            ```
            

### 3-4. 다중 도큐먼트 갱신

---

- `updateMany`: 조건에 맞는 도큐먼트를 모두 수정
    - 스키마를 변경하거나 특정 사용자에게 새로운 정보를 추가할 때 쓰기 좋다.
    
    ```bash
    test> db.users.insertMany([ { birthday: "10/13/1978" }, { birthday: "10/13/1978" }, { birthday: "10/13/1978" }])
    {
      acknowledged: true,
      insertedIds: {
        '0': ObjectId("6367da34750d5a5b553def96"),
        '1': ObjectId("6367da34750d5a5b553def97"),
        '2': ObjectId("6367da34750d5a5b553def98")
      }
    }
    
    test> db.users.updateMany({"birthday" : "10/13/1978"}, {"$set" : {"gift" : "Happy Birthday!"}})
    {
      acknowledged: true,
      insertedId: null,
      matchedCount: 3,
      modifiedCount: 3,
      upsertedCount: 0
    }
    
    test> db.users.find({"birthday" : "10/13/1978"})
    [
      {
        _id: ObjectId("6367da34750d5a5b553def96"),
        birthday: '10/13/1978',
        gift: 'Happy Birthday!'
      },
      {
        _id: ObjectId("6367da34750d5a5b553def97"),
        birthday: '10/13/1978',
        gift: 'Happy Birthday!'
      },
      {
        _id: ObjectId("6367da34750d5a5b553def98"),
        birthday: '10/13/1978',
        gift: 'Happy Birthday!'
      }
    ]
    ```
    

### 3-5. 갱신한 도큐먼트 반환

---

- `findOneAndXX`: `updateOne`과 같은 메서드와의 주요 차이점은 사용자가 수정된 도큐먼트의 값을 원자적으로 얻을 수 있다는 점이다.

![Untitled](./image/3/Untitled%203.png)

- `findOneAndUpdate`: 한 번의 연산으로 항목을 반환하고 갱신할 수 있다. 기본적으로 도큐먼트의 상태를 수정하기 전에 반환한다.
    - 사전준비
        
        ```bash
        test> db.processes.insertOne({
        ...     "_id" : ObjectId("4b3e7a18005cab32be6291f7"),
        ...     "priority" : 1,
        ...     "status" : "READY"
        ... })
        ```
        
    
    ```bash
    test> db.processes.findOneAndUpdate({"status" : "READY"},
    ... {"$set" : {"status" : "RUNNING"}},
    ... {"sort" : {"priority" : -1}})
    {
      _id: ObjectId("4b3e7a18005cab32be6291f7"),
      priority: 1,
      status: 'READY'
    }
    ```
    
    - 옵션 도큐먼트의 "`returnNewDocument`" 필드를 `true`로 설정하면 갱신된 도큐먼트를 반환한다.
        
        ```bash
        test> db.processes.findOneAndUpdate({"status" : "READY"},
        ... {"$set" : {"status" : "RUNNING"}},
        ... {"sort" : {"priority" : -1}, "returnNewDocument" : true})
        {
          _id: ObjectId("4b3e7a18005cab32be6291f7"),
          priority: 1,
          status: 'RUNNING'
        }
        ```
        
- `findOneAndDelete`: 삭제된 도큐먼트를 반환한다.

- 참고
    - [https://devbksheen.tistory.com/m/entry/몽고DB-시작하기](https://devbksheen.tistory.com/m/entry/%EB%AA%BD%EA%B3%A0DB-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)
    - [https://jackjeong.tistory.com/m/167](https://jackjeong.tistory.com/m/167)
    - [https://github.com/yongtaelim/book-study/blob/master/mongo/src/main/kotlin/com/jessyt/mongo/chapter03/README.md](https://github.com/yongtaelim/book-study/blob/master/mongo/src/main/kotlin/com/jessyt/mongo/chapter03/README.md)
    - [https://github.com/WouNice/Groceries/blob/master/MongoDB权威指南/Part 1 MongoDB 入门/第 3 章 创建、更新和删除文档.md](https://github.com/WouNice/Groceries/blob/master/MongoDB%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97/Part%201%20MongoDB%20%E5%85%A5%E9%97%A8/%E7%AC%AC%203%20%E7%AB%A0%20%E5%88%9B%E5%BB%BA%E3%80%81%E6%9B%B4%E6%96%B0%E5%92%8C%E5%88%A0%E9%99%A4%E6%96%87%E6%A1%A3.md)
