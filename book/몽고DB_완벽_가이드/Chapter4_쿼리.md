# Chapter 4. 쿼리

## 1. find 소개

---

- `find` 함수: 쿼리에 사용한다.
    - 쿼리: 컬렉션에서 도큐먼트의 서브셋(빈 컬렉션부터 컬렉션 전체까지)을 반환한다.
    - 첫 매개변수에 따라 어떤 도큐먼트를 가져올지 결정된다. 빈 쿼리 도큐먼트(즉 `{}`)는 컬렉션 내 모든 것과 일치한다.
        
        ```bash
        # 컬렉션 c 내 모든 도큐먼트를 반환한다.
        db.c.find()
        ```
        

### 1-1. 반환받을 키 지정

---

- 도큐먼트 내 키/값 정보가 모두 필요하지 않을 수 있다 → `find`(또는 `findOne`)의 두 번째 매개변수에 원하는 키를 지정하면 된다.
    
    ```bash
    # 사용자 정보 컬렉션에서 "username"과 "email" 키의 값만 원할 때
    test> db.users.find({}, {"username" : 1, "email" : 1})
    [
      {
        _id: ObjectId("4ba0f0dfd22aa494fd523620"),
        username: 'joe',
        email: 'joe@example.com'
      }
    ]
    ```
    
    - "`_id`" 키는 지정하지 않아도 항상 반환된다.
- 두 번째 매개변수를 사용해서 특정 키/값 쌍을 제외한 결과를 얻을 수도 있다.
    
    ```bash
    test> db.users.find({}, {"fatal_weakness" : 0})
    [
      {
        _id: ObjectId("4ba0f0dfd22aa494fd523620"),
        username: 'joe',
        email: 'joe@example.com'
      }
    ]
    
    test> db.users.find({}, {"username" : 1, "_id" : 0})
    [ { username: 'joe' } ]
    ```
    

### 1-2. 제약 사항

---

- 데이터베이스에서 쿼리 도큐먼트 값은 반드시 상수여야 한다.
    - 도큐먼트 내 다른 키의 값을 참조할 수 없음을 의미한다.
        
        ```bash
        # 재고 도큐먼트에 재고 수량 키 "in_stock"과 판매 수량 키 "num_sold"가 있으면 키 값을 다음과 같은 쿼리로 비교할 수 없다.
        > db.stock.find({"in_stock" : "this.num_sold"}) // 작동하지 않음
        ```
        

## 2. 쿼리 조건

---

### 2-1. 쿼리 조건절

---

- 비교연산자: "`$lt`"(<), "`$lte`"(<=), "`$gt`"(>), "`$gte`"(>=)
    
    ```bash
    # "age"가 18 이상, 30 이하인 도큐먼트를 모두 찾아낸다.
    > db.users.find({"age" : {"$gte" : 18, "$lte" : 30}})
    
    # 2007년 1월 1일 이전에 등록한 사람
    > start = new Date("01/01/2007")
    > db.users.find({"registered" : {"$lt" : start}})
    ```
    

- "`$ne`": 키 값이 특정 값과 일치하지 않는 도큐먼트를 찾음. 모든 데이터형에 사용할 수 있다.
    
    ```bash
    # 사용자명이 "joe"가 아닌 사용자
    > db.users.find({"username" : {"$ne" : "joe"}})
    ```
    

### 2-2. OR 쿼리

---

- "`$in`": 하나의 키를 다양한 값과 비교하는 쿼리에 사용한다. 서로 다른 데이터형도 쓸 수 있다.
    
    ```bash
    # 추첨 당첨자를 뽑는 상황에서 당첨 번호가 725, 542, 390
    > db.raffle.find({"ticket_no" : {"$in" : [725, 542, 390]}})
    
    # "user_id"가 12345이거나 "joe"인 도큐먼트를 찾는다.
    > db.users.find({"user_id" : {"$in" : [12345, "joe"]}})
    ```
    
- "`$nin`": 배열 내 조건과 일치하지 않는 도큐먼트를 반환한다.
    
    ```bash
    # 당첨 번호를 가지지 않는 사람을 모두 반환한다.
    > db.raffle.find({"ticket_no" : {"$nin" : [725, 542, 390]}})
    ```
    
- "`$or`": 더 일반적이며, 여러 키를 주어진 값과 비교하는 쿼리에 사용한다. 가능한 조건들의 배열을 취한다. 다른 조건절도 포함할 수 있다.
    
    ```bash
    # "ticket_no"가 725이거나 "winner"가 true인 도큐먼트
    > db.raffle.find({"$or" : [{"ticket_no" : 725}, {"winner" : true}]})
    
    # "ticket_no"가 세 번호 중 적어도 하나와 일치하거나 "winner"가 true인 경우
    > db.raffle.find({"$or" : [{"ticket_no" : {"$in" : [725, 542, 390]}},
    ...                        {"winner" : true}]})
    ```
    

<aside>
💡 `AND` 쿼리에서는 최소한의 인수로 최적의 결과(범위를 좁힌 결과)를 추려내야 한다. `OR` 쿼리는 반대인데, 첫 번째 인수가 일치하는 도큐먼트가 많을수록 효율적이다.

</aside>

<aside>
💡 "`$or`" 연산자가 항상 작동하는 동안에는 가능한 한 "`$in`"을을 사용하자. 쿼리 옵티마이저는 "`$in`"을 더 효율적으로 다룬다

</aside>

### 2-3. `$not`

---

- `$not`: 메타 조건절이며 어떤 조건에도 적용할 수 있다. 정규 표현식과 함께 사용해 주어진 패턴과 일치하지 않는 도큐먼트를 찾을 때 특히 유용하다.
    
    ```bash
    # "$mod" 연산자로 도큐먼트를 쿼리하는 예제
    # "id_num"의 값이 1, 6, 11, 16 등인 사용자
    > db.users.find({"id_num" : {"$mod" : [5, 1]}})
    
    # "id_num"이 2, 3, 4, 5, 7, 8, 9, 10, 12 등인 사용자
    > db.users.find({"id_num" : {"$not" : {"$mod" : [5, 1]}}})
    ```
    

## 3. 형 특정 쿼리

---

### 3-1. `null`

---

- `null`은 스스로와 일치하는 것을 찾는다.
    
    ```bash
    test> db.c.find()
    [
      { _id: ObjectId("4ba0f0dfd22aa494fd523621"), y: null },
      { _id: ObjectId("4ba0f0dfd22aa494fd523622"), y: 1 },
      { _id: ObjectId("4ba0f148d22aa494fd523623"), y: 2 }
    ]
    
    test> db.c.find({"y" : null})
    [ { _id: ObjectId("4ba0f0dfd22aa494fd523621"), y: null } ]
    ```
    
- `null`은 '존재하지 않음'과도 일치한다 → 해당 키를 갖지 않는 도큐먼트도 반환한다.
    
    ```bash
    test> db.c.find({"z" : null})
    [
      { _id: ObjectId("4ba0f0dfd22aa494fd523621"), y: null },
      { _id: ObjectId("4ba0f0dfd22aa494fd523622"), y: 1 },
      { _id: ObjectId("4ba0f148d22aa494fd523623"), y: 2 }
    ]
    ```
    
- 값이 `null`인 키만 찾고 싶으면 키가 `null`인 값을 쿼리하고, "`$exists`" 조건절을 사용해 `null` 존재 여부를 확인하면 된다.
    
    ```bash
    test> db.c.find({"z" : {"$eq" : null, "$exists" : true}})
    ```
    

### 3-2. 정규 표현식

---

- "`$regex`": 패턴 일치 문자열을 위한 정규식 기능을 제공한다. 정규 표현식 플래그(예를 들면 `i`)는 사용할 수 있지만 꼭 필요하지는 않다.
    
    ```bash
    # 이름이 Joe나 joe인 사용자
    > db.users.find( {"name" : {"$regex" : /joe/i } })
    
    # joe뿐 아니라 joey도 찾고 싶다면
    > db.users.find({"name" : /joey?/i})
    ```
    
- 몽고DB는 정규 표현식 일치에 펄 호환 정규 표현식(PCRE) 라이브러리를 사용

### 3-3. 배열에 쿼리하기

---

- 배열 요소 쿼리는 스칼라 쿼리와 같은 방식으로 동작하도록 설계됐다.
    
    ```bash
    test> db.food.insertOne({"fruit" : ["apple", "banana", "peach"]})
    
    test> db.food.find({"fruit" : "banana"})
    [
      {
        _id: ObjectId("63691f7e6922ecc5ea5063a1"),
        fruit: [ 'apple', 'banana', 'peach' ]
      }
    ]
    ```
    

- `$all` 연산자
    - 2개 이상의 배열 요소가 일치하는 배열을 찾으려면 "`$all`"을 사용한다. 순서는 중요하지 않다.
        
        ```bash
        test> db.food.insertOne({"_id" : 1, "fruit" : ["apple", "banana", "peach"]})
        test> db.food.insertOne({"_id" : 2, "fruit" : ["apple", "kumquat", "orange"]})
        test> db.food.insertOne({"_id" : 3, "fruit" : ["cherry", "banana", "apple"]})
        
        # "apple"과 "banana" 요소
        test> db.food.find({fruit : {$all : ["apple", "banana"]}})
        [
          { _id: 1, fruit: [ 'apple', 'banana', 'peach' ] },
          { _id: 3, fruit: [ 'cherry', 'banana', 'apple' ] }
        ]
        ```
        

- 전체 배열과 정확하게 일치하는 도큐먼트를 쿼리할 수도 있다.
    
    ```bash
    test> db.food.find({"fruit" : ["apple", "banana", "peach"]})
    [
      { _id: 1, fruit: [ 'apple', 'banana', 'peach' ] }
    ]
    
    # 일치하지 않는다.
    test> db.food.find({"fruit" : ["apple", "banana"]})
    test> db.food.find({"fruit" : ["banana", "apple", "peach"]})
    ```
    
- 배열 내 특정 요소를 쿼리하려면 key.index 구문을 이용해 순서를 지정한다.
    
    ```bash
    test> db.food.find({"fruit.2" : "peach"})
    [
      { _id: 1, fruit: [ 'apple', 'banana', 'peach' ] }
    ]
    ```
    
    - 배열의 인덱스는 항상 0에서 시작

- `$size` 연산자
    - 특정 크기의 배열을 쿼리하는 유용한 조건절. 다른 `$조건절`과 결합해 사용할 수 없다.
        
        ```bash
        test> db.food.find({"fruit" : {"$size" : 3}})
        [
          { _id: 1, fruit: [ 'apple', 'banana', 'peach' ] },
          { _id: 2, fruit: [ 'apple', 'kumquat', 'orange' ] },
          { _id: 3, fruit: [ 'cherry', 'banana', 'apple' ] }
        ]
        ```
        

- `$slice` 연산자
    - 배열 요소의 부분집합을 반환받을 수 있다.
        
        ```bash
        # 블로그 게시물에서 먼저 달린 댓글 열 개를 반환
        > db.blog.posts.findOne(criteria, {"comments" : {"$slice" : 10}})
        
        # 나중에 달린 댓글 열 개를 반환
        > db.blog.posts.findOne(criteria, {"comments" : {"$slice" : -10}})
        ```
        
    - 오프셋과 요소 개수를 지정해 원하는 범위 안에 있는 결과를 반환할 수 있다. 현재 있는 요소까지만 반환한다.
        
        ```bash
        # 처음 23개를 건너뛰고, 24번째 요소부터 33번째 요소까지 반환
        > db.blog.posts.findOne(criteria, {"comments" : {"$slice" : [23, 10]}})
        ```
        
    - 특별히 명시하지 않는 한 "`$slice`" 연산자는 도큐먼트 내 모든 키를 반환한다. 명시하지 않은 키는 반환하지 않는 다른 키 명시자들과는 다르다.

- 일치하는 배열 요소의 반환
    - `$` 연산자를 사용하면 일치하는 요소를 반환받을 수 있다.
        
        ```bash
        # 도큐먼트에서 첫 번째로 일치하는 댓글만 반환함
        test> db.blog.posts.find({"comments.name" : "bob"}, {"comments.$" : 1})
        [
          {
            _id: ObjectId("635e2d44d6ae359ae0982419"),
            comments: [
              { name: 'bob', email: 'bob@example.com', content: 'good post.' }
            ]
          }
        ]
        ```
        

- 배열 및 범위 쿼리의 상호작용
    
    ```bash
    test> db.test.find()
    [
      { _id: ObjectId("636925ee6922ecc5ea5063a2"), x: 5 },
      { _id: ObjectId("636925ee6922ecc5ea5063a3"), x: 15 },
      { _id: ObjectId("636925ee6922ecc5ea5063a4"), x: 25 },
      { _id: ObjectId("636925ee6922ecc5ea5063a5"), x: [ 5, 25 ] }
    ]
    
    # "x"의 값이 10과 20 사이인 도큐먼트 -> 배열에 대한 범위 쿼리가 본질적으로 쓸모없어진다.
    test> db.test.find({"x" : {"$gt" : 10, "$lt" : 20}})
    [
      { _id: ObjectId("636925ee6922ecc5ea5063a3"), x: 15 },
      { _id: ObjectId("636925ee6922ecc5ea5063a5"), x: [ 5, 25 ] } # 5와 25 둘 다 10과 20 사이는 아니지만, 25는 첫 번째 절과 일치하고 5는 두 번째 절과 일치하기 때문에 반환되었다.
    ]
    ```
    
    - "`$elemMatch`" 연산자를 사용하면 몽고DB는 두 절을 하나의 배열 요소와 비교한다.
        - 비배열 요소를 일치시키지 않는다는 함정이 있다.
            
            ```bash
            > db.test.find({"x" : {"$elemMatch" : {"$gt" : 10, "$lt" : 20}}})
            > // 결과 없음
            ```
            
    - 쿼리하는 필드에 인덱스가 있다면 `min` 함수와 `max` 함수를 사용해 "`$gt`"와 "`$lt`" 값 사이로 인덱스 범위를 제한해 쿼리할 수 있다.
        
        ```bash
        test> db.test.createIndexes( [{ "x" : 1 }] )
        [ 'x_1' ]
        
        test> db.test.find({"x" : {"$gt" : 10, "$lt" : 20}}).min({"x" : 10}).max({"x" : 20}).hint({"x" : 1})
        [ { _id: ObjectId("636925ee6922ecc5ea5063a3"), x: 15 } ]
        ```
        
    - 배열을 포함하는 도큐먼트에 범위 쿼리를 할 때 `min`함수와 `max`함수를 사용하면 좋다. 배열에 대한 "`$gt`"/"`$lt`" 쿼리의 인덱스 한계는 비효율적이다.
        - 어떤 값이든 허용하므로 범위 내 값뿐 아니라 모든 인덱스 항목을 검색한다.

### 3-4. 내장 도큐먼트에 쿼리하기

---

- 도큐먼트 전체를 대상으로 하는 방식
    - 일반적인 쿼리와 동일하게 작동한다.
        
        ```bash
        {
            "name" : {
                "first" : "Joe",
                "last" : "Schmoe"
            },
            "age" : 45
        }
        
        # Joe Schmoe라는 사람을 쿼리
        > db.people.find({"name" : {"first" : "Joe", "last" : "Schmoe"}})
        ```
        
        - 서브도큐먼트 전체에 쿼리하려면 서브도큐먼트와 정확히 일치해야 한다. 또한 이런 쿼리를 순서를 따진다.
- 도큐먼트 내 키/값 쌍 각각을 대상으로 하는 방식
    - 정확히 일치시키는 방법이 아니므로 스키마가 변경되더라도 모든 쿼리가 정상적으로 작동한다.
        
        ```bash
        > db.people.find({"name.first" : "Joe", "name.last" : "Schmoe"})
        ```
        
    - 점 표기법은 쿼리 도큐먼트와 다른 도큐먼트 타입의 큰 차이점이다.
        - 쿼리 도큐먼트는 점을 포함할 수 있고, 이는 내장 도큐먼트 내 항목에 접근할 수 있다는 의미다.
        - 입력하는 도큐먼트에 `.` 문자를 사용할 수 없는 이유이기도 하다.
    - 모든 키를 지정하지 않고도 조건을 정확하게 묶으려면 "`$elemMatch`"를 사용한다.
        - 조건을 부분적으로 지정해 배열 내에서 하나의 내장 도큐먼트를 찾게 해준다.
            
            ```bash
            > db.blog.find()
            {
                "content" : "...",
                "comments" : [
                    {
                        "author" : "joe",
                        "score" : 3,
                        "comment" : "nice post"
                    },
                    {
                        "author" : "mary",
                        "score" : 6,
                        "comment" : "terrible post"
                    }
                ]
            }
            
            # 블로그 게시물에서 5점 이상을 받은 joe의 댓글을 찾는다.
            > db.blog.find({"comments" : {"$elemMatch" :
            ... {"author" : "joe", "score" : {"$gte" : 5}}}})
            ```
            

## 4. `$where` 쿼리

---

- "`$where`": 도큐먼트 내 두 키의 값을 비교하는 쿼리에 가장 자주 쓰인다.
    
    ```bash
    > db.foo.insertOne({"apple" : 1, "banana" : 6, "peach" : 3})
    > db.foo.insertOne({"apple" : 8, "spinach" : 4, "watermelon" : 4})
    
    # 두 필드의 값이 동일한 도큐먼트를 반환
    > db.foo.find({"$where" : function () {
    ... for (var current in this) {
    ...     for (var other in this) {
    ...         if (current != other && this[current] == this[other]) {
    ...             return true;
    ...         }
    ...     }
    ... }
    ... return false;
    ... }});
    [
      {
        _id: ObjectId("63692c956922ecc5ea5063a7"),
        apple: 8,
        spinach: 4,
        watermelon: 4
      }
    ]
    ```
    
- "`$where`" 쿼리는 일반 쿼리보다 훨씬 느리니 반드시 필요한 경우가 아니면 사용하지 말자.
    - 실행 시 각 도큐먼트는 BSON에서 자바스크립트 객체로 변환되기 때문에 오래걸린다.
    - 또한 "`$where`" 절에는 인덱스를 쓸 수 없다.

- 몽고DB 3.6에는 몽고DB 쿼리 언어로 집계 표현식을 사용할 수 있도록 `$expr` 연산자가 추가됐다.
    - 자바스크립트를 실행하지 않아 더 빨리 쿼리할 수 있으므로 가능한 한 `$where` 대신 `$expr`을 사용하자.

## 5. 커서

---

- 데이터베이스는 커서를 사용해 `find`의 결과를 반환한다.
- 일반적으로 클라이언트 측의 커서 구현체는 쿼리의 최종 결과를 강력히 제어하게 해준다.
    - 결과 개수를 제한하거나, 결과 중 몇 개를 건너뛰거나, 여러 키를 조합한 결과를 어떤 방향으로든 정렬하는 등 다양하게 조작할 수 있다.
        
        ```bash
        test> for(i=0; i<100; i++) {
        ...     db.collection.insertOne({x : i});
        ... }
        {
          acknowledged: true,
          insertedId: ObjectId("63692e516922ecc5ea50640b")
        }
        
        test> var cursor = db.collection.find();
        ```
        
        - 결과를 한 번에 하나씩 볼 수 있다는 장점이 있다.
        - 이 방법은 컬렉션 내에 무엇이 있는지 보는 데 사용하며, 셸에서 실제 프로그래밍을 하는 데는 적합하지 않다.
    - 결과를 얻으려면 커서의 `next` 메서드를 사용하고, 다른 결과가 있는지 확인하려면 `hasNext`를 사용한다.
        
        ```bash
        # 결과를 확인하는 반복문
        test> while (cursor.hasNext()) {
        ...     obj = cursor.next();
        ...     // 사용자 정의 작업 수행
        ... }
        { _id: ObjectId("63692e516922ecc5ea50640b"), x: 99 }
        
        # forEach 반복문
        test> cursor.forEach(function(x) {
        ...     print(x);
        ... });
        { _id: ObjectId("63692e506922ecc5ea5063a8"), x: 0 }
        { _id: ObjectId("63692e506922ecc5ea5063a9"), x: 1 }
        ...
        { _id: ObjectId("63692e516922ecc5ea50640b"), x: 99 }
        
        test> cursor.forEach(function (x) { print(x); });
        MongoCursorExhaustedError: Cursor is exhausted
        ```
        
    - find를 호출할 때 셸이 데이터베이스를 즉시 쿼리하지는 않으며 결과를 요청하는 쿼리를 보낼 때까지 기다린다.
        - 쿼리하기 전에 옵션을 추가할 수 있다. 어떤 순서로든 이어 쓸 수 있다.
            
            ```bash
            # 모두 동일하게 작동한다.
            > var cursor = db.f.find().sort({"x" : 1}).limit(1).skip(10);
            > var cursor = db.f.find().limit(1).sort({"x" : 1}).skip(10);
            > var cursor = db.f.find().skip(10).limit(1).sort({"x" : 1});
            
            # 이때 비로소 쿼리가 서버로 전송된다.
            > cursor.hasNext()
            ```
            

### 5-1. 제한, 건너뛰기, 정렬

---

- 가장 일반적인 쿼리 옵션으로는 반환받는 결과 개수를 제한하거나, 몇 개의 결과를 건너뛰거나, 결과를 정렬하는 옵션이 있다.
    - 결과 개수를 제한하려면 `find` 호출에 `limit` 함수를 연결한다.
        
        ```bash
        # 3개의 결과만 반환
        > db.c.find().limit(3)
        ```
        
        - 컬렉션에서 쿼리 조건에 맞는 결과가 3개보다 적으면, 조건에 맞는 도큐먼트 개수만큼만 반환한다. `limit`은 상한만 설정하고 하한은 설정하지 않는다.
    - `skip`은 `limit`와 유사하게 작동한다.
        
        ```bash
        # 조건에 맞는 결과 중 처음 3개를 건너뛴 나머지를 반환
        > db.c.find().skip(3)
        ```
        
    - `sort`는 객체를 매개변수로 받는다. 정렬 방향은 1(오름차순) 또는 -1(내림차순)이다.
        
        ```bash
        # "username"은 오름차순으로, "age"는 내림차순으로 정렬
        > db.c.find().sort({"username" : 1, "age" : -1})
        ```
        

- 페이지를 나눌 때 편리하다.
    
    ```bash
    # 가격을 내림차순으로 정렬해 한 페이지당 50개씩 결과를 보여주기
    > db.stock.find({"desc" : "mp3"}).limit(50).sort({"price" : -1})
    
    # 다음 페이지
    > db.stock.find({"desc" : "mp3"}).limit(50).skip(50).sort({"price" : -1})
    ```
    
    - 큰 수를 건너뛰면 비효율적이다.

- 비교 순서
    1. `null`
    2. 숫자(`int`, `long`, `double`, `decimal`)
    3. 문자열
    4. 객체/도큐먼트
    5. 배열
    6. 이진 데이터
    7. 객체 ID
    8. 불리언
    9. 날짜
    10. 타임스탬프
    11. 정규 표현식

### 5-2. 많은 수의 건너뛰기 피하기

---

- 대부분의 데이터베이스는 `skip`을 위해 인덱스 안에 메타데이터를 저장하지만 몽고DB는 아직 해당 기능을 지원하지 않는다 → 많은 수의 건너뛰기는 피해야 한다.

- `skip`을 사용하지 않고 페이지 나누기
    
    ```bash
    // 사용하지 말자. 많은 수의 건너뛰기 때문에 느려진다.
    > var page1 = db.foo.find().limit(100)
    > var page2 = db.foo.find().skip(100).limit(100)
    > var page3 = db.foo.find().skip(100).limit(100)
    ```
    
    - "date"를 내림차순으로 정렬해 도큐먼트를 표시
        
        ```bash
        # 첫 페이지를 구한다.
        > var page1 = db.foo.find().sort({"date" : -1}).limit(100)
        
        # 마지막 도큐먼트의 "date" 값을 사용해 다음 페이지를 가져온다.
        var latest = null;
        
        // 첫 페이지 보여주기
        while (page1.hasNext()) {
            latest = page1.next();
            display(latest);
        }
        
        // 다음 페이지 가져오기
        var page2 = db.foo.find({"date" : {"$lt" : latest.date}});
        page2.sort({"date" : -1}).limit(100);
        ```
        
- 랜덤으로 도큐먼트 찾기
    
    ```bash
    // 사용하지 말자.
    > var total = db.foo.count()
    > var random = Math.floor(Math.random()*total)
    > db.foo.find().skip(random).limit(1)
    ```
    
    - 도큐먼트를 입력할 때 랜덤 키를 별도로 추가하는 방법
        
        ```bash
        > db.people.insertOne({"name" : "joe", "random" : Math.random()})
        > db.people.insertOne({"name" : "john", "random" : Math.random()})
        > db.people.insertOne({"name" : "jim", "random" : Math.random()})
        
        # 랜덤 수를 계산해 쿼리 조건으로 사용한다.
        > var random = Math.random()
        > result = db.people.findOne({"random" : {"$gt" : random}})
        # "random" 값보다 클 때는 빈 결과를 반환한다. 도큐먼트를 다른 방향으로 반환함으로써 간단히 방지할 수 있다.
        > if (result == null) {
        ...     result = db.people.findOne({"random" : {"$lte" : random}})
        ... }
        ```
        

### 5-3. 종료되지 않는 커서

---

- 커서
    - 클라이언트가 보는 커서
    - 클라이언트 커서가 나타내는 데이터베이스 커서 (서버 측 커서)

- 서버 측에서 보면 커서는 메모리와 리소스를 점유한다.
    - 커서가 더는 가져올 결과가 없거나 클라이언트로부터 종료 요청을 받으면 데이터베이스를 점유하고 있던 리소스를 해제한다.
    - 그러면 데이터베이스가 리소스를 다른 작업에 사용할 수 있으므로 커서도 신속하게 해제해야 한다.

- 서버 커서를 종료하는(그리고 이후의 작업도 정리하는) 몇 가지 조건
    1. 커서는 조건에 일치하는 결과를 모두 살펴본 후에는 스스로 정리한다.
    2. 커서가 클라이언트측에서 유효 영역을 벗어나면 드라이버는 데이터베이스에 메시지를 보내 커서를 종료해도 된다고 알린다.
    3. 사용자가 아직 결과를 다 살펴보지 않았고, 커서가 여전히 유효 영역 내에 있더라도 10분 동안 활동이 없으면 데이터베이스 커서는 자동으로 죽는다.

- 종종 커서를 오래 남겨두고 싶을 때
    - 많은 드라이버는 데이터베이스가 커서를 타임아웃시키지 못하게 하는 `immortal`이라는 함수를 제공한다.
    - 커서는 데이터베이스에 남아 서버가 재시작할 때까지 리소스를 차지한다.

- 참고
    - [https://jackjeong.tistory.com/m/173](https://jackjeong.tistory.com/m/173)
    - [https://github.com/yongtaelim/book-study/tree/master/mongo/src/main/kotlin/com/jessyt/mongo/chapter04](https://github.com/yongtaelim/book-study/tree/master/mongo/src/main/kotlin/com/jessyt/mongo/chapter04)
    - [https://github.com/WouNice/Groceries/blob/master/MongoDB权威指南/Part 1 MongoDB 入门/第 4 章 查询.md](https://github.com/WouNice/Groceries/blob/master/MongoDB%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97/Part%201%20MongoDB%20%E5%85%A5%E9%97%A8/%E7%AC%AC%204%20%E7%AB%A0%20%E6%9F%A5%E8%AF%A2.md)