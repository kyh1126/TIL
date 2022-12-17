# Chapter 8. 트랜잭션

- 몽고DB는 여러 작업, 컬렉션, 데이터베이스, 도큐먼트 및 샤드에서 ACID 호환 트랜잭션을 지원한다.

## 1. 트랜잭션 소개

---

- 트랜잭션: 읽기나 쓰기 작업이 가능한 데이터베이스의 작업을 하나 이상 포함하는 데이터베이스의 논리적 처리 단위
    - 작업이 성공하든 실패하든 부분적으로는 완료되지 않는다.
    - 트랜잭션을 사용하려면 몽고DB 버전이 4.2 이상이어야 하며 몽고DB 드라이버를 몽고DB 4.2 이상에 맞게 갱신해야 한다.

### 1-1. ACID의 정의

---

- Atomicity (원자성): 트랜잭션 내 모든 작업이 적용되거나 아무 작업도 적용되지 않도록 한다.
    - 트랜잭션은 부분적으로 적용될 수 없다.
    - 커밋되거나 중단된다.
- Consistency (일관성): 트랜잭션이 성공하면 데이터베이스가 하나의 일관성 있는 상태에서 다음 일관성 있는 상태로 이동하도록 한다.
- Isolation (고립성): 여러 트랜잭션이 데이터베이스에서 동시에 실행되도록 허용하는 속성이다.
    - 트랜잭션이 다른 트랜잭션의 부분 결과를 보지 않도록 보장한다.
    - 여러 병렬 트랜잭션이 각 트랜잭션을 순차적으로 실행할 때와 동일한 결과를 얻게 된다.
- Durability (영속성): 트랜잭션이 커밋될 때 시스템 오류가 발생하더라도 모든 데이터가 유지되도록 한다.

- ACID를 준수한다: 속성을 모두 충족하고 성공적인 트랜잭션만 처리될 때

- 몽고DB는 복제 셋과(또는) 샤드 전체에 ACID 호환 트랜잭션이 있는 분산 데이터베이스다.

## 2. 트랜잭션 사용법

---

- 몽고DB는 트랜잭션을 사용하기 위한 두 가지 API를 제공
    - 코어 API: 관계형 데이터베이스와 유사한 구문
        - 대부분의 오류에 재시도 로직을 제공하지 않으며 개발자가 작업에 대한 로직, 트랜잭션 커밋 함수, 필요한 재시도 및 오류 로직을 모두 작성해야 한다.
    - 콜백 API: 트랜잭션 사용에 권장되는 접근 방식
        - 지정된 논리 세션과 관련된 트랜잭션 시작, 콜백 함수로 제공한 함수 실행, 트랜잭션 커밋(또는 오류 시 중단)을 포함해 코어 API에 비해 많은 기능을 래핑하는 단일 함수를 제공한다.
        - 커밋 오류를 처리하는 재시도 로직도 포함한다.

- 개발자는 트랜잭션에서 사용할 논리 세션을 시작해야 하며, 트랜잭션의 작업이 특정 논리 세션과 연결돼야 한다.(즉 세션을 각 작업에 전달)
    - 몽고DB 논리 세션은 전체 몽고DB 배포 컨텍스트에서 작업의 시간과 순서를 추적한다.
    - 논리 세션 또는 서버 세션은 몽고DB에서 재시도 가능한 쓰기와 인과적 일관성(causal consistency)을 지원하기 위해 클라이언트 세션에서 사용하는 기본 프레임워크의 일부다.

- 애플리케이션이 복잡하고 추가 코드 작성이 필요할 때 코어 API보다 콜백 API를 권장한다.
    
    
    | 코어 API | 콜백 API |
    | --- | --- |
    | 트랜잭션을 시작하고 커밋하려면 명시적인 호출이 필요하다. | 트랜잭션을 시작하고 지정된 작업을 실행한 후 커밋(또는 오류 시 중단)한다. |
    | TransientTransactionError 및 UnknownTransactionCommitResult에 대한 오류 처리 로직을 통합하지 않고, 대신 사용자 지정 오류 처리를 통합하는 유연성을 제공한다. | TransientTransactionError 및 UnknownTransactionCommitResult에 대한 오류 처리 로직을 자동으로 통합한다. |
    | 특정 트랜잭션을 위해 API로 전달되는 명시적 논리 세션이 필요하다. | 특정 트랜잭션을 위해 API로 전달되는 명시적 논리 세션이 필요하다. |

- ex> 전자상거래 사이트에서는 주문이 이뤄지고 해당 품목이 판매되면 재고에서 제거된다.
    
    ```bash
    # 단일 트랜잭션에 서로 다른 컬렉션의 두 도큐먼트가 포함된다.
    orders.insertOne({ "sku": "abc123", "qty": 100 }, session=session)
    inventory.update_one({ "sku": "abc123", "qty": { "$gte": 100 } }, session=session)
    ```
    
    - 코어 API
        
        ```python
        # Define the uriString using the DNS Seedlist Connection Format 
        # for the connection
        uri = 'mongodb+srv://server.example.com/'
        client = MongoClient(uriString)
        
        my_wc_majority = WriteConcern('majority', wtimeout=1000)
        
        # Prerequisite / Step 0: Create collections, if they don't already exist. 
        # CRUD operations in transactions must be on existing collections.
        
        client.get_database( "webshop",
                             write_concern=my_wc_majority).orders.insert_one({"sku":
                             "abc123", "qty":0})
        client.get_database( "webshop",
                             write_concern=my_wc_majority).inventory.insert_one(
                             {"sku": "abc123", "qty": 1000})
        
        # Step 1: Define the operations and their sequence within the transaction
        def update_orders_and_inventory(my_session):
            orders = session.client.webshop.orders
            inventory = session.client.webshop.inventory
        
            with session.start_transaction(
                    read_concern=ReadConcern("snapshot"),
                    write_concern=WriteConcern(w="majority"),
                    read_preference=ReadPreference.PRIMARY):
        
                orders.insert_one({"sku": "abc123", "qty": 100}, session=my_session)
                inventory.update_one({"sku": "abc123", "qty": {"$gte": 100}},
                                     {"$inc": {"qty": -100}}, session=my_session)
                commit_with_retry(my_session)
        
        # Step 2: Attempt to run and commit transaction with retry logic
        def commit_with_retry(session):
            while True:
                try:
                    # Commit uses write concern set at transaction start.
                    session.commit_transaction()
                    print("Transaction committed.")
                    break
                except (ConnectionFailure, OperationFailure) as exc:
                    # Can retry commit
                    if exc.has_error_label("UnknownTransactionCommitResult"):
                        print("UnknownTransactionCommitResult, retrying "
                              "commit operation ...")
                        continue
                    else:
                        print("Error during commit ...")
                        raise
        
        # Step 3: Attempt with retry logic to run the transaction function txn_func
        def run_transaction_with_retry(txn_func, session):
            while True:
                try:
                    txn_func(session)  # performs transaction
                    break
                except (ConnectionFailure, OperationFailure) as exc:
                    # If transient error, retry the whole transaction
                    if exc.has_error_label("TransientTransactionError"):
                        print("TransientTransactionError, retrying transaction ...")
                        continue
                    else:
                        raise
        
        # Step 4: Start a session.
        with client.start_session() as my_session:
        
        # Step 5: Call the function 'run_transaction_with_retry' passing it the function
        # to call 'update_orders_and_inventory' and the session 'my_session' to associate
        # with this transaction.
        
            try:
                run_transaction_with_retry(update_orders_and_inventory, my_session)
            except Exception as exc:
                # Do something with error. The error handling code is not
                # implemented for you with the Core API.
                raise
        ```
        
    - 콜백 API
        
        ```python
        # Define the uriString using the DNS Seedlist Connection Format 
        # for the connection
        uriString = 'mongodb+srv://server.example.com/'
        client = MongoClient(uriString)
        
        my_wc_majority = WriteConcern('majority', wtimeout=1000)
        
        # Prerequisite / Step 0: Create collections, if they don't already exist.
        # CRUD operations in transactions must be on existing collections.
        
        client.get_database( "webshop",
                             write_concern=my_wc_majority).orders.insert_one({"sku":
                             "abc123", "qty":0})
        client.get_database( "webshop",
                             write_concern=my_wc_majority).inventory.insert_one(
                             {"sku": "abc123", "qty": 1000})
        
        # Step 1: Define the callback that specifies the sequence of operations to
        # perform inside the transactions.
        
        def callback(my_session):
            orders = my_session.client.webshop.orders
            inventory = my_session.client.webshop.inventory
        
            # Important:: You must pass the session variable 'my_session' to 
            # the operations.
        
            orders.insert_one({"sku": "abc123", "qty": 100}, session=my_session)
            inventory.update_one({"sku": "abc123", "qty": {"$gte": 100}},
                                 {"$inc": {"qty": -100}}, session=my_session)
        
        #. Step 2: Start a client session.
        
        with client.start_session() as session:
        # Step 3: Use with_transaction to start a transaction, execute the callback,
        # and commit (or abort on error).
        
            session.with_transaction(callback,
                                     read_concern=ReadConcern('local'),
                                     write_concern=my_write_concern_majority,
                                     read_preference=ReadPreference.PRIMARY)
        ```
        

<aside>
💡 트랜잭션에서 컬렉션 생성, 삭제 또는 인덱스 작업이 허용되지 않는다.

</aside>

## 3. 애플리케이션을 위한 트랜잭션 제한 조정

---

- 애플리케이션이 트랜잭션을 최적으로 사용하도록 매개변수를 조정하자.

### 3-1. 타이밍과 Oplog 크기 제한

---

- 시간 제한
    - 특정 트랜잭션이 실행될 수 있는 시간, 트랜잭션이 락을 획득하려고 대기하는 시간, 모든 트랜잭션이 실행될 최대 길이를 제어
    - 트랜잭션의 최대 실행 시간은 기본적으로 1분 이하다.
        - mongod 인스턴스 레벨에서 `transactionLifeTimeLimitSeconds`에 의해 제어되는 제한을 수정해 증가시킬 수 있다.
    - 주기적으로 실행되는 정리 프로세스에 의해 중단된다.
        - 60초와 `transactionLifeTimeLimitSeconds/2` 중 더 낮은 값을 주기로 실행된다.
    - 트랜잭션에 시간 제한을 명시적으로 설정하려면 commitTransaction에 `maxTimeMS`를 지정하는 것이 좋다.
        - 설정하지 않으면 `transactionLifeTimeLimitSeconds`가 사용된다.
        - 설정했지만 `transactionLifeTimeLimitSeconds`를 초과하는 경우 `transactionLifeTimeLimitSeconds`가 대신 사용된다.
    - 트랜잭션의 작업에 필요한 락을 획득하기 위해 트랜잭션이 대기하는 최대 시간은 기본적으로 5밀리세컨드다.
        - `maxTransactionLockRequestTimeoutMillis`에 의해 제어되는 제한을 수정해 늘릴 수 있다.
        - 이 시간 내에 락을 획득할 수 없으면 트랜잭션은 중단된다.
        - 0: 필요한 모든 락을 즉시 획득할 수 없으면 트랜잭션이 중단된다.
        - -1: 작업별 제한 시간이 `maxTimeMS`에 지정된 대로 사용된다.
        - 0보다 큰 숫자: 트랜잭션이 필요한 락을 획득하려고 시도하는 (지정된) 기간으로 해당 시간까지의 대기 시간을 구성한다.
- Oplog 크기 제한
    - 몽고DB는 트랜잭션의 쓰기 작업에 필요한 만큼 oplog 항목을 생성한다.
    - 각 oplog 항목은 BSON 도큐먼트 크기 제한인 16메가바이트 이햐여야 한다.

- 유연성 있는 모델과 스키마 설계 패턴과 같은 모범 사례를 사용하면 대부분의 상황에서 트랜잭션을 사용하지 않아도 된다.

<aside>
💡 트랜잭션은 애플리케이션에서 드물게 사용하는 것이 좋은 강력한 기능이다.

</aside>

- 참고
    - [https://github.com/arkss/TIL/blob/master/MONGODB/02_몽고DB개발/08_트랜잭션.md](https://github.com/arkss/TIL/blob/master/MONGODB/02_%EB%AA%BD%EA%B3%A0DB%EA%B0%9C%EB%B0%9C/08_%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98.md)
    - [https://jackjeong.tistory.com/m/179](https://jackjeong.tistory.com/m/179)
    - [https://github.com/mongodb-the-definitive-guide-3e/mongodb-the-definitive-guide-3e/blob/master/chapter8/ch8_1.md](https://github.com/mongodb-the-definitive-guide-3e/mongodb-the-definitive-guide-3e/blob/master/chapter8/ch8_1.md)