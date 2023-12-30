# Chapter 5. 인덱싱

## 1. 인덱싱 소개

---

- 데이터베이스 인덱스는 책의 인덱스와 유사하다.
    - 데이터베이스는 전체 내용을 살펴보는 대신 지름길을 택해, 특정 내용을 가리키는 정렬된 리스트를 확인한다.

- 컬렉션 스캔: 인덱스를 사용하지 않는 쿼리
    - 서버가 쿼리 결과를 찾으려면 '전체 내용을 살펴봐야 함'

- ex> 1백만 건(혹은 인내심이 있다면 1천만 건이나 1억 건)의 도큐먼트를 갖는 컬렉션을 생성해보자.
    
    ```bash
    test> for (i=0; i<1000000; i++) {
    ...     db.users.insertOne(
    ...         {
    ...              "i" : i,
    ...              "username" : "user"+i,
    ...              "age" : Math.floor(Math.random()*120),
    ...              "created" : new Date()
    ...         }
    ...     );
    ... }
    {
      acknowledged: true,
      insertedId: ObjectId("636faf8e42454e67365a47b4")
    }
    ```
    

- `explain` 함수를 이용해 쿼리가 실행될 때 몽고DB가 무엇을 하는지 확인할 수 있다.
    - `explain`는 명령을 감싸는 커서 보조자 메서드와 사용하면 좋다.
    - `explain` 커서 메서드는 다양한 CRUD 작업의 실행 정보를 제공한다.
    - `executionStats` 모드는 인덱스를 이용한 쿼리의 효과를 이해하는 데 도움이 된다.
        
        ```bash
        test> db.users.find({"username": "user101"}).explain("executionStats")
        {
          explainVersion: '1',
          queryPlanner: {
            namespace: 'test.users',
            indexFilterSet: false,
            parsedQuery: { username: { '$eq': 'user101' } },
            queryHash: '7D9BB680',
            planCacheKey: '7D9BB680',
            maxIndexedOrSolutionsReached: false,
            maxIndexedAndSolutionsReached: false,
            maxScansToExplodeReached: false,
            winningPlan: {
              stage: 'COLLSCAN',
              filter: { username: { '$eq': 'user101' } },
              direction: 'forward'
            },
            rejectedPlans: []
          },
          executionStats: {
            executionSuccess: true,
            nReturned: 1,
            executionTimeMillis: 675,
            totalKeysExamined: 0,
            totalDocsExamined: 1000000,
            executionStages: {
              stage: 'COLLSCAN',
              filter: { username: { '$eq': 'user101' } },
              nReturned: 1,
              executionTimeMillisEstimate: 111,
              works: 1000002,
              advanced: 1,
              needTime: 1000000,
              needYield: 0,
              saveState: 1000,
              restoreState: 1000,
              isEOF: 1,
              direction: 'forward',
              docsExamined: 1000000
            }
          },
          command: { find: 'users', filter: { username: 'user101' }, '$db': 'test' },
          serverInfo: {
            host: 'ab32e314d715',
            port: 27017,
            version: '6.0.2',
            gitVersion: '94fb7dfc8b974f1f5343e7ea394d0d9deedba50e'
          },
          serverParameters: {
            internalQueryFacetBufferSizeBytes: 104857600,
            internalQueryFacetMaxOutputDocSizeBytes: 104857600,
            internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
            internalDocumentSourceGroupMaxMemoryBytes: 104857600,
            internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
            internalQueryProhibitBlockingMergeOnMongoS: 0,
            internalQueryMaxAddToSetBytes: 104857600,
            internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
          },
          ok: 1
        }
        ```
        
        - `totalDocsExamined`: 몽고DB가 쿼리를 실행하면서 살펴본 도큐먼트 개수
        - `executionTimeMillis`: 쿼리하는 데 걸린 시간 (밀리초 단위)
        - `nReturned`: 반환받은 결과의 개수

<aside>
💡 몽고DB가 쿼리에 효율적으로 응답하게 하려면 애플리케이션의 모든 쿼리 패턴에 인덱스를 사용해야 한다.

</aside>

### 1-1. 인덱스 생성

---

- 인덱스를 만들려면 `createIndex` 컬렉션 메서드를 사용한다.
    
    ```bash
    test> db.users.createIndex({"username" : 1})
    username_1
    # {
    #     "createdCollectionAutomatically" : false,
    #     "numIndexesBefore" : 1,
    #     "numIndexesAfter" : 2,
    #     "ok" : 1
    # }
    ```
    
    - `createIndex` 호출이 몇 초 후에 반횐되지 않으면, 다른 셸에서 `db.currentOp()`를 실행하거나 `mongod`의 로그를 확인해 인덱스 구축의 진행률을 체크해보자.

- 인덱스 구축이 완료되면 다시 처음처럼 쿼리
    
    ```bash
    test> db.users.find({"username": "user101"}).explain("executionStats")
    {
      explainVersion: '1',
      queryPlanner: {
        namespace: 'test.users',
        indexFilterSet: false,
        parsedQuery: { username: { '$eq': 'user101' } },
        queryHash: '7D9BB680',
        planCacheKey: '24069050',
        maxIndexedOrSolutionsReached: false,
        maxIndexedAndSolutionsReached: false,
        maxScansToExplodeReached: false,
        winningPlan: {
          stage: 'FETCH',
          inputStage: {
            stage: 'IXSCAN',
            keyPattern: { username: 1 },
            indexName: 'username_1',
            isMultiKey: false,
            multiKeyPaths: { username: [] },
            isUnique: false,
            isSparse: false,
            isPartial: false,
            indexVersion: 2,
            direction: 'forward',
            indexBounds: { username: [ '["user101", "user101"]' ] }
          }
        },
        rejectedPlans: []
      },
      executionStats: {
        executionSuccess: true,
        nReturned: 1,
        executionTimeMillis: 7,
        totalKeysExamined: 1,
        totalDocsExamined: 1,
        executionStages: {
          stage: 'FETCH',
          nReturned: 1,
          executionTimeMillisEstimate: 0,
          works: 2,
          advanced: 1,
          needTime: 0,
          needYield: 0,
          saveState: 0,
          restoreState: 0,
          isEOF: 1,
          docsExamined: 1,
          alreadyHasObj: 0,
          inputStage: {
            stage: 'IXSCAN',
            nReturned: 1,
            executionTimeMillisEstimate: 0,
            works: 2,
            advanced: 1,
            needTime: 0,
            needYield: 0,
            saveState: 0,
            restoreState: 0,
            isEOF: 1,
            keyPattern: { username: 1 },
            indexName: 'username_1',
            isMultiKey: false,
            multiKeyPaths: { username: [] },
            isUnique: false,
            isSparse: false,
            isPartial: false,
            indexVersion: 2,
            direction: 'forward',
            indexBounds: { username: [ '["user101", "user101"]' ] },
            keysExamined: 1,
            seeks: 1,
            dupsTested: 0,
            dupsDropped: 0
          }
        }
      },
      command: { find: 'users', filter: { username: 'user101' }, '$db': 'test' },
      serverInfo: {
        host: 'ab32e314d715',
        port: 27017,
        version: '6.0.2',
        gitVersion: '94fb7dfc8b974f1f5343e7ea394d0d9deedba50e'
      },
      serverParameters: {
        internalQueryFacetBufferSizeBytes: 104857600,
        internalQueryFacetMaxOutputDocSizeBytes: 104857600,
        internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
        internalDocumentSourceGroupMaxMemoryBytes: 104857600,
        internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
        internalQueryProhibitBlockingMergeOnMongoS: 0,
        internalQueryMaxAddToSetBytes: 104857600,
        internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
      },
      ok: 1
    }
    ```
    

👉

- 인덱스는 쿼리 시간에 놀라운 차이를 만든다.****
- 인덱싱된 필드를 변경하는 쓰기(삽입, 갱신, 삭제) 작업은 더 오래 걸린다.
    - 데이터가 변경될 때마다 도큐먼트뿐 아니라 모든 인덱스를 갱신해야 하기 때문이다.

### 1-2. 복합 인덱스 소개

---

- 인덱스는 가능한 한 효율적으로 쿼리하려는 목적으로 사용한다.
- 상당수의 쿼리 패턴은 두 개 이상의 키를 기반으로 인덱스를 작성해야 한다.
- 인덱스가 앞 부분에 놓일 때만 정렬에 도움이 된다.
    
    ```bash
    # "username" 인덱스는 다음 정렬에 큰 도움이 되지 않는다.
    db.users.find().sort({"age" : 1, "username" : 1})
    ```
    
    - 명령은 먼저 "age"로 정렬한 후에 "username" 으로 정렬

- 복합 인덱스: 2개 이상의 필드로 구성된 인덱스
    
    ```bash
    test> db.users.createIndex({"age" : 1, "username" : 1})
    age_1_username_1
    ```
    
    - 쿼리에서 정렬 방향이 여러 개이거나 검색 조건에 여러 개의 키가 있을 때 유용하다.

- 정렬 없는(순차 정렬(natural order)이라 한다) 쿼리
    
    ```bash
    test> db.users.find({}, {"_id" : 0, "i" : 0, "created" : 0})
    [
      { username: 'user0', age: 118 },
      { username: 'user1', age: 66 },
      { username: 'user2', age: 56 },
    ...
    ]
    ```
    
- 인덱스는 다음과 같은 형태로 표현된다.
    
    ```bash
    test> db.users.find({}, {"_id" : 0, "i" : 0, "created" : 0}).sort({"age" : 1, "username" : 1})
    [
      { username: 'user100089', age: 0 },
      { username: 'user10020', age: 0 },
      { username: 'user100227', age: 0 },
    ...
      { username: 'user100230', age: 1 },
      { username: 'user100316', age: 1 },
      { username: 'user100401', age: 1 },
    ...
    ]
    ```
    
    - 각 인덱스 항목은 나이와 사용자명을 포함하고 레코드 식별자를 가리킨다.
- 레코드 식별자는 내부에서 스토리지 엔진에 의해 사용되며 도큐먼트 데이터를 찾는다.

- 인덱스를 사용하는 방법
    1. 단일 값을 찾는 동등 쿼리
        
        `db.users.find({"age" : 21}).sort({"username" :-1})`
        
        - `{"age" : 21}`과 일치하는 마지막 항목부터 순서대로 인덱스를 탐색한다.
        
        <aside>
        💡 몽고DB는 인덱스를 어느 방향으로도 쉽게 탐색하므로 정렬 방향은 문제가 되지 않음
        
        </aside>
        
    2. 범위 쿼리이며 여러 값이 일치하는 도큐먼트를 찾아낸다
        
        `db.users.find({"age" : {"$gte" : 21, "$lte" : 30}})`
        
        - 인덱스에 있는 첫 번째 키인 "age"를 다음처럼 사용해 일치하는 도큐먼트를 반환받는다.
        
        <aside>
        💡 몽고DB가 인덱스를 사용해 쿼리하면 일반적으로 인덱스 순서에 따라 도큐먼트 결과를 반환한다.
        
        </aside>
        
    3. 다중값 쿼리 (정렬을 포함)
        
        `db.users.find({"age" : {"$gte" : 21, "$lte" :30}}).sort({"username" : 1})`
        
        - 검색 조건에 맞는 인덱스를 사용한다.
        - 인덱스는 사용자명을 정렬된 순서로 반환하지 않으며, 쿼리는 사용자명에 따라 정렬된 결과를 요청한다 → 결과를 반환하기 전에 메모리에서 정렬해야 함을 의미한다. (이전 쿼리보다 비효율적)
            - 속도는 검색 조건과 일치하는 결과가 얼마나 많은지에 따라 다르다.
            - 결과가 32메가바이트 이상이면 몽고DB는 데이터가 너무 많아 정렬을 거부한다는 오류를 내보낸다. (4.4 버전부터는 100메가바이트)****
                
                ```bash
                Error: error: {
                    "ok" : 0,
                    "errmsg" : "Executor error during find command: OperationFailed:
                Sort operation used more than the maximum 33554432 bytes of RAM. Add
                an index, or specify a smaller limit.",
                    "code" : 96,
                    "codeName" : "OperationFailed"
                }
                ```
                
            
            <aside>
            💡 오류를 피하고 싶다면 [정렬 작업을 지원하는 인덱스](https://www.mongodb.com/docs/manual/reference/method/cursor.sort/#sort-index-use)를 생성하고 `sort`에 `limit`를 함께 사용해 결과를 32메가바이트 이하로 줄이자.
            
            </aside>
            
        - 같은 키를 역순으로 한 `{"username" : 1, "age" : 1}` 인덱스 또한 사용할 수 있다.
            - 모든 인덱스 항목을 탐색하지만 원하는 순서로 되돌린다. 인덱스의 "age" 부분을 이용해서 일치하는 도큐먼트를 가져온다.
            - 거대한 인메모리 정렬이 필요하지 않다는 장점이 있다.
            - 하지만 일치하는 값을 모두 찾으려면 전체 인덱스를 훑어야 한다.

<aside>
💡 복합 인덱스를 구성할 때는 정렬 키를 첫 번째에 놓으면 좋다.

</aside>

### 1-3. 몽고DB가 인덱스를 선택하는 방법

---

- 쿼리가 들어오면 몽고DB는 쿼리 모양을 확인한다.
    - 모양은 검색할 필드와 정렬 여부 등 추가 정보와 관련 있다.
    
    👉 시스템은 이 정보를 기반으로 쿼리를 충족하는 데 사용할 인덱스 후보 집합을 식별한다.
    

- ex> 쿼리가 들어오고 인덱스 5개 중 3개가 쿼리 후보로 식별됐다.
    - 몽고DB는 각 인덱스 후보에 하나씩 총 3개의 쿼리 플랜을 만들고, 각각 다른 인덱스를 사용하는 3개의 병렬 스레드에서 쿼리를 실행한다. (어떤 스레드에서 가장 빨리 결과를 반환하는지 확인하기 위함)
    - 가장 먼저 목표 상태에 도달하는 쿼리 플랜이 승자 → 앞으로 동일한 모양을 가진 쿼리에 사용할 인덱스로 선택된다.

- 플랜은 일정 기간(시범 기간(trial period)이라 한다) 동안 서로 경쟁하며, 각 레이스의 결과로 전체 승리 플랜을 산출한다.
    - 쿼리 스레드가 레이스에서 이기려면, 모든 쿼리 결과를 가장 먼저 반환하거나 결과에 대한 시범 횟수를 정렬 순서로 가장 먼저 반환해야 한다.
        
        ![5-1. 몽고DB 쿼리 플래너가 인덱스를 선택하는 방법(레이스로 시각화)](Chapter%205%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%83%E1%85%A6%E1%86%A8%E1%84%89%E1%85%B5%E1%86%BC%202adf2b0981944d8d9ac06aee3f78e1ff/5-1.png)
        
        5-1. 몽고DB 쿼리 플래너가 인덱스를 선택하는 방법(레이스로 시각화)
        
    - 승리한 플랜은 차후 모양이 같은 쿼리에 사용하기 위해 캐시에 저장된다.
    - 인덱스를 다시 작성하거나, 인덱스를 추가하거나 삭제하면 플랜이 캐시에서 제거된다.
    - 쿼리 플랜 캐시는 명시적으로 지울 수 있으며, `mongod` 프로세스를 다시 시작할 때도 삭제된다.

### 1-4. 복합 인덱스 사용

---

### 인덱스의 선택성을 고려한다

---

- 특정 쿼리 패턴에서 스캔할 레코드 개수를 인덱스가 얼마나 최소화하는지에 관심 있다.

- ex> 약 백만 개의 레코드가 포함된 학생 데이터셋
    
    ```bash
    # 데이터셋의 도큐먼트
    {
        "_id" : ObjectId("585d817db4743f74e2da067c"),
        "student_id" : 0,
        "scores" : [
        {
            "type" : "exam",
            "score" : 38.05000060199827
        },
        {
            "type" : "quiz",
            "score" : 79.45079445008987
        },
        {
            "type" : "homework",
            "score" : 74.50150548699534
        },
        {
            "type" : "homework",
            "score" : 74.68381684615845
        }
        ],
        "class_id" : 127
    }
    
    # 인덱스 두 개
    > db.students.createIndex({"class_id": 1})
    > db.students.createIndex({student_id: 1, class_id: 1})
    
    # 쿼리
    > db.students.find({student_id:{$gt:500000}, class_id:54})
    ...          .sort({student_id:1})
    ...          .explain("executionStats")
    ```
    
- [BEFORE] `explain` 메서드가 제공하는 실행 통계(execution stats): 몽고DB가 인덱스를 어떻게 사용해서 쿼리를 충족했는지 알려준다.
    
    ```bash
    {
      "queryPlanner": {
        "plannerVersion": 1,
        "namespace": "school.students",
        "indexFilterSet": false,
        "parsedQuery": {
          "$and": [
            {
              "class_id": {
                "$eq": 54
              }
            },
            {
              "student_id": {
                "$gt": 500000
              }
            }
          ]
        },
        "winningPlan": {
          "stage": "FETCH",
          "inputStage": {
            "stage": "IXSCAN",
            "keyPattern": {
              "student_id": 1,
              "class_id": 1
            },
            "indexName": "student_id_1_class_id_1",
            "isMultiKey": false,
            "multiKeyPaths": {
              "student_id": [ ],
              "class_id": [ ]
            },
            "isUnique": false,
            "isSparse": false,
            "isPartial": false,
            "indexVersion": 2,
            "direction": "forward",
            "indexBounds": {
              "student_id": [
                "(500000.0, inf.0]"
              ],
              "class_id": [
                "[54.0, 54.0]"
              ]
            }
          }
        },
        "rejectedPlans": [
          {
            "stage": "SORT",
            "sortPattern": {
              "student_id": 1
            },
            "inputStage": {
              "stage": "SORT_KEY_GENERATOR",
              "inputStage": {
                "stage": "FETCH",
                "filter": {
                  "student_id": {
                    "$gt": 500000
                  }
                },
                "inputStage": {
                  "stage": "IXSCAN",
                  "keyPattern": {
                    "class_id": 1
                  },
                  "indexName": "class_id_1",
                  "isMultiKey": false,
                  "multiKeyPaths": {
                  "class_id": [ ]
                  },
                  "isUnique": false,
                  "isSparse": false,
                  "isPartial": false,
                  "indexVersion": 2,
                  "direction": "forward",
                  "indexBounds": {
                    "class_id": [
                      "[54.0, 54.0]"
                    ]
                  }
                }
              }
            }
          }
        ]
      },
      "executionStats": {
        "executionSuccess": true,
        "nReturned": 9903,
        "executionTimeMillis": 4325,
        "totalKeysExamined": 850477,
        "totalDocsExamined": 9903,
        "executionStages": {
          "stage": "FETCH",
          "nReturned": 9903,
          "executionTimeMillisEstimate": 3485,
          "works": 850478,
          "advanced": 9903,
          "needTime": 840574,
          "needYield": 0,
          "saveState": 6861,
          "restoreState": 6861,
          "isEOF": 1,
          "invalidates": 0,
          "docsExamined": 9903,
          "alreadyHasObj": 0,
          "inputStage": {
            "stage": "IXSCAN",
            "nReturned": 9903,
            "executionTimeMillisEstimate": 2834,
            "works": 850478,
            "advanced": 9903,
            "needTime": 840574,
            "needYield": 0,
            "saveState": 6861,
            "restoreState": 6861,
            "isEOF": 1,
            "invalidates": 0,
            "keyPattern": {
              "student_id": 1,
              "class_id": 1
            },
            "indexName": "student_id_1_class_id_1",
            "isMultiKey": false,
            "multiKeyPaths": {
              "student_id": [ ],
              "class_id": [ ]
            },
            "isUnique": false,
            "isSparse": false,
            "isPartial": false,
            "indexVersion": 2,
            "direction": "forward",
            "indexBounds": {
              "student_id": [
                "(500000.0, inf.0]"
              ],
              "class_id": [
                "[54.0, 54.0]"
              ]
            },
            "keysExamined": 850477,
            "seeks": 840575,
            "dupsTested": 0,
            "dupsDropped": 0,
            "seenInvalidated": 0
          }
        }
      },
      "serverInfo": {
        "host": "SGB-MBP.local",
        "port": 27017,
        "version": "3.4.1",
        "gitVersion": "5e103c4f5583e2566a45d740225dc250baacfbd7"
      },
      "ok": 1
    }
    ```
    
- "`executionStats`" 필드: 선정된 쿼리 플랜에 대해 완료된 쿼리 실행을 설명하는 통계를 포함한다.
    - "`totalKeysExamined`": 몽고DB가 결과 셋을 생성하기 위해 인덱스 내에서 몇 개의 키를 통과했는지 나타낸다.
        - "`nReturns`"와 비교하면 몽고DB가 쿼리와 일치하는 도큐먼트를 찾으려고 얼마나 많은 인덱스를 통과했는지 알 수 있다.
        
        😵 예제에서는 일치하는 도큐먼트 9903개를 찾으려고 인덱스 키 85만 477개를 검사 → 쿼리를 충족하는 데 사용된 인덱스가 선택적이지 않았음을 의미한다. "`executionTimeMillis`" 필드에 표시됐듯 쿼리를 실행하는 데 4.3초가 넘게 걸렸다. 
        
    - "`winningPlan`": 선정된 쿼리 플랜
        - 선정된 플랜은 "student_id"와 "class_id"를 기반으로 복합 인덱스를 사용했다.
            
            ```bash
            "winningPlan": {
              "stage": "FETCH",
              "inputStage": {
                "stage": "IXSCAN",
                "keyPattern": {
                  "student_id": 1,
                  "class_id": 1
                },
            ```
            
        - 쿼리 플랜은 단계 트리로 표시한다. 각 단계에는 하위 단계 개수에 따라 하나 이상의 입력 단계가 있을 수 있다.
            - 입력 단계는 도큐먼트나 인덱스 키를 상위 단계에 제공한다.
                - 예제에서는 입력 단계인 인덱스 스캔이 하나 있었고, 해당 스캔은 쿼리와 일치하는 도큐먼트에 대한 레코드 ID를 상위 단계인 "`FETCH`" 단계에 제공했다.
            - 그러면 "`FETCH`" 단계는 도큐먼트를 검색하고 클라이언트가 요청하면 일괄적으로 반환한다.
    - "`rejectedPlans`": 실패한 쿼리 플랜
        - 실패한 쿼리 플랜은 "class_id"를 기반으로 하는 인덱스를 사용. 이후에 인메모리 정렬을 수행했다.
        - 쿼리 플랜에 "`SORT`" 단계가 표시된다면 → 정렬할 때 인덱스를 사용할 수 없었으며 대신 인메모리 정렬을 했다는 의미
            
            ```bash
            "rejectedPlans": [
              {
                "stage": "SORT",
                "sortPattern": {
                  "student_id": 1
                },
            ```
            

- 실패한 쿼리 플랜에 있는 "class_id"만을 기반으로 인덱스를 사용하는 것이 좋다.
    - 실행 중인 다중값 쿼리는 "student_id"가 50만보다 큰 레코드를 요청
        
        → 컬렉션에 있는 레코드의 약 절반
        

- 커서 `hint` 메서드: 모양이나 이름을 지정함으로써 사용할 인덱스를 지정할 수 있다.
    
    ```bash
    # hint를 사용
    > db.students.find({student_id:{$gt:500000}, class_id:54})
    ...          .sort({student_id:1})
    ...          .hint({class_id:1})
    ...          .explain("executionStats")
    ```
    
- 인덱스 필터: 쿼리, 정렬, 프로젝션 사양의 조합인 쿼리 모양을 사용한다.
    - `planCacheSetFilter` 함수를 인덱스 필터와 함께 사용하면, 쿼리 옵티마이저가 인덱스 필터에 저장된 인덱스만 고려하도록 제한할 수 있다.
    - 쿼리 모양에 대한 인덱스 필터가 있을 때는 몽고DB가 `hint`를 무시한다.
    - 인덱스 필터는 `mongod` 서버 프로세스 동안만 지속되며 종료 후에는 지속되지 않는다.
- [AFTER] 출력된 결과: 약 85만 개의 인덱스 키를 스캔 → 약 2만 개, 4.3초 → 272밀리초
    
    ```bash
    {
      "queryPlanner": {
        "plannerVersion": 1,
        "namespace": "school.students",
        "indexFilterSet": false,
        "parsedQuery": {
          "$and": [
            {
              "class_id": {
                "$eq": 54
              }
            },
            {
              "student_id": {
                "$gt": 500000
              }
            }
          ]
        },
        "winningPlan": {
          "stage": "SORT",
          "sortPattern": {
            "student_id": 1
          },
          "inputStage": {
            "stage": "SORT_KEY_GENERATOR",
            "inputStage": {
              "stage": "FETCH",
              "filter": {
                "student_id": {
                  "$gt": 500000
                }
              },
              "inputStage": {
                "stage": "IXSCAN",
                "keyPattern": {
                  "class_id": 1
                },
                "indexName": "class_id_1",
                "isMultiKey": false,
                "multiKeyPaths": {
                  "class_id": [ ]
                },
                "isUnique": false,
                "isSparse": false,
                "isPartial": false,
                "indexVersion": 2,
                "direction": "forward",
                "indexBounds": {
                  "class_id": [
                    "[54.0, 54.0]"
                  ]
                }
              }
            }
          }
        },
        "rejectedPlans": [ ]
      },
      "executionStats": {
        "executionSuccess": true,
        "nReturned": 9903,
        "executionTimeMillis": 272,
        "totalKeysExamined": 20076,
        "totalDocsExamined": 20076,
        "executionStages": {
          "stage": "SORT",
          "nReturned": 9903,
          "executionTimeMillisEstimate": 248,
          "works": 29982,
          "advanced": 9903,
          "needTime": 20078,
          "needYield": 0,
          "saveState": 242,
          "restoreState": 242,
          "isEOF": 1,
          "invalidates": 0,
          "sortPattern": {
            "student_id": 1
          },
          "memUsage": 2386623,
          "memLimit": 33554432,
          "inputStage": {
            "stage": "SORT_KEY_GENERATOR",
            "nReturned": 9903,
            "executionTimeMillisEstimate": 203,
            "works": 20078,
            "advanced": 9903,
            "needTime": 10174,
            "needYield": 0,
            "saveState": 242,
            "restoreState": 242,
            "isEOF": 1,
            "invalidates": 0,
            "inputStage": {
              "stage": "FETCH",
              "filter": {
                "student_id": {
                  "$gt": 500000
                }
              },
              "nReturned": 9903,
              "executionTimeMillisEstimate": 192,
              "works": 20077,
              "advanced": 9903,
              "needTime": 10173,
              "needYield": 0,
              "saveState": 242,
              "restoreState": 242,
              "isEOF": 1,
              "invalidates": 0,
              "docsExamined": 20076,
              "alreadyHasObj": 0,
              "inputStage": {
                "stage": "IXSCAN",
                "nReturned": 20076,
                "executionTimeMillisEstimate": 45,
                "works": 20077,
                "advanced": 20076,
                "needTime": 0,
                "needYield": 0,
                "saveState": 242,
                "restoreState": 242,
                "isEOF": 1,
                "invalidates": 0,
                "keyPattern": {
                  "class_id": 1
                },
                "indexName": "class_id_1",
                "isMultiKey": false,
                "multiKeyPaths": {
                  "class_id": [ ]
                },
                "isUnique": false,
                "isSparse": false,
                "isPartial": false,
                "indexVersion": 2,
                "direction": "forward",
                "indexBounds": {
                  "class_id": [
                    "[54.0, 54.0]"
                  ]
                },
                "keysExamined": 20076,
                "seeks": 1,
                "dupsTested": 0,
                "dupsDropped": 0,
                "seenInvalidated": 0
              }
            }
          }
        }
      },
      "serverInfo": {
        "host": "SGB-MBP.local",
        "port": 27017,
        "version": "3.4.1",
        "gitVersion": "5e103c4f5583e2566a45d740225dc250baacfbd7"
      },
      "ok": 1
    }
    ```
    

### 더 나은 인덱스를 설계

---

- 우리가 정말로 보고 싶은 것은 "`totalKeysExamined`"와 매우 유사한 "`nReturned`"이다. 또한 `hint`를 사용하지 않고 쿼리를 효율적으로 실행하기를 원한다.
- 일반적으로 동등 필터를 사용할 필드가 다중값 필터를 사용할 필드보다 앞에 오도록 복합 인덱스를 설계해야 한다.
    
    ```bash
    > db.students.createIndex({class_id:1, student_id:1})
    ```
    
    - 새로운 인덱스가 준비된 상태에서 쿼리를 다시 실행하면 이번에는 `hint`가 필요하지 않다.
        
        ```bash
        ...
        "executionStats": {
          "executionSuccess": true,
          "nReturned": 9903,
          "executionTimeMillis": 37,
          "totalKeysExamined": 9903,
          "totalDocsExamined": 9903,
          "executionStages": {
            "stage": "FETCH",
            "nReturned": 9903,
            "executionTimeMillisEstimate": 36,
            "works": 9904,
            "advanced": 9903,
            "needTime": 0,
            "needYield": 0,
            "saveState": 81,
            "restoreState": 81,
            "isEOF": 1,
            "invalidates": 0,
            "docsExamined": 9903,
            "alreadyHasObj": 0,
            "inputStage": {
              "stage": "IXSCAN",
              "nReturned": 9903,
              "executionTimeMillisEstimate": 0,
              "works": 9904,
              "advanced": 9903,
              "needTime": 0,
              "needYield": 0,
              "saveState": 81,
              "restoreState": 81,
              "isEOF": 1,
              "invalidates": 0,
              "keyPattern": {
                "class_id": 1,
                "student_id": 1
              },
              "indexName": "class_id_1_student_id_1",
              "isMultiKey": false,
              "multiKeyPaths": {
                "class_id": [ ],
                "student_id": [ ]
              },
              "isUnique": false,
              "isSparse": false,
              "isPartial": false,
              "indexVersion": 2,
              "direction": "forward",
              "indexBounds": {
                "class_id": [
                  "[54.0, 54.0]"
                ],
                "student_id": [
                  "(500000.0, inf.0]"
                ]
              },
              "keysExamined": 9903,
              "seeks": 1,
              "dupsTested": 0,
              "dupsDropped": 0,
              "seenInvalidated": 0
            }
          }
        },
        ```
        

- 복합 인덱스를 설계할 때
    - 동등 필터에 대한 키를 맨 앞에 표시해야 한다.
    - 정렬에 사용되는 키는 다중값 필드 앞에 표시해야 한다.
    - 다중값 필터에 대한 키는 마지막에 표시해야 한다.

- 키 방향 선택하기
    - 복합 정렬을 서로 다른 방향으로 최적화하려면 방향에 맞는 인덱스를 사용해야 한다.
        
        `{"age": 1, "username": -1}`
        
    - 애플리케이션이 `{"age": 1, "username": 1}`을 이용해 정렬을 최적화해야 한다면 해당 방향으로 두 번째 인덱스를 생성한다.
    - 역방향 인덱스(각 방향에 -1을 곱한다)는 서로 동등하다.
        - `{"age": 1, "username": -1}`과 `{"age": -1, "username": 1}`은 동일한 쿼리를 충족한다.
- 커버드 쿼리 사용하기
    - 쿼리가 단지 인덱스에 포함된 필드를 찾는 중이라면 도큐먼트를 가져올 필요가 없다.
    - 인덱스가 쿼리가 요구하는 값을 모두 포함하면, 쿼리가 커버드된다고 한다.
    - 쿼리가 확실히 인덱스만 사용하게 하려면 "`_id`" 필드를 반환받지 않도록 반환받을 키를 지정해야 한다.
    - 커버드 쿼리에 `explain`을 실행하면 결과에 "`FETCH`" 단계의 하위 단계가 아닌 "`IXSCAN`" 단계가 있으며, "`executionStats`"에서는 "`totalDocsExamined`"의 값이 0이 된다.
- 암시적 인덱스
    - 복합 인덱스는 '이중 임무'를 수행할 수 있으며 쿼리마다 다른 인덱스처럼 동작할 수 있다.
        - `{"age": 1, "username": 1}`로 인덱스를 가지면 "age" 필드는 `{"age": 1}`로만 인덱스를 가질 때와 동일한 방법으로 정렬된다.
    - 인덱스가 N 개의 키를 가진다면 키들의 앞부분은 '공짜' 인덱스가 된다.
        - ex> `{"a": 1, "b": 1, "c": 1 ..., "z": 1}`과 같은 인덱스가 있다면 사실상 `{"a": 1}`, `{"a": 1, "b": 1}`, `{"a": 1, "b": 1, "c": 1}` 등으로 인덱스를 가진다.

### 1-5. `$` 연산자의 인덱스 사용법

---

- 비효율적인 연산자
    - 일반적으로 부정 조건은 비효율적이다.
    - `$ne`: 인덱스를 사용하긴 하지만 잘 활용하지는 못한다. "`$ne`"로 지정된 항목을 제외한 모든 인덱스 항목을 살펴봐야 하므로 기본적으로 전체 인덱스를 살펴봐야 한다.
        - ex> "i"라는 필드에 인덱스를 갖는 컬렉션에서, 탐색된 인덱스 범위: 쿼리는 3보다 작은 인덱스 항목과 3보다 큰 인덱스 항목을 모두 조사한다.
            
            ```bash
            db.example.find({"i" : {"$ne" : 3}}).explain()
            {
                "queryPlanner" : {
                    ...,
                    "parsedQuery" : {
                        "i" : {
                            "$ne" : "3"
                        }
                    },
                    "winningPlan" : {
                    {
                        ...,
                        "indexBounds" : {
                            "i" : [
                                [
                                    {
                                        "$minElement" : 1
                                    },
                                    3
                                ],
                                [
                                    3,
                                    {
                                        "$maxElement" : 1
                                    }
                                    ]
                                ]
                            }
                        }
                    },
                    "rejectedPlans" : [ ]
                },
                "serverInfo" : {
                    ...,
                }
            }
            ```
            
    - `$not`: 종종 인덱스를 사용. "`$not`"은 기초적인 범위와 정규 표현식을 반대로 뒤집을 수 있다(ex> `{"key" :{"$lt" : 7}}`을 `{"key" : {"$gte" : 7}}`로). 하지만 "`$not`"을 사용하는 쿼리 대부분은 테이블 스캔을 수행한다.
    - `$nin`: 항상 테이블 스캔을 수행한다.
- 범위
    - 다중 필드로 인덱스를 설계할 때는 완전 일치가 사용될 필드를 첫 번째에, 범위가 사용될 필드를 마지막에 놓자 → 쿼리가 첫 번째 인덱스 키와 정확히 일치하는 값을 찾은 후 두 번째 인덱스 범위 안에서 검색****
        - ex> 특정한 나이와 사용자명의 범위에 `{"age" : 1, "username" : 1}` 인덱스를 사용해 쿼리: "age" : 47로 건너뛰고 곧이어 "user5"와 "user8" 사이의 사용자명 범위 내에서 검색한다.
            
            ```bash
            > db.users.find({"age" : 47, "username" :
            ... {"$gt" : "user5", "$lt" : "user8"}}).explain('executionStats')
            {
                "queryPlanner" : {
                    "plannerVersion" : 1,
                    "namespace" : "test.users",
                    "indexFilterSet" : false,
                    "parsedQuery" : {
                        "$and" : [
                            {
                                "age" : {
                                    "$eq" : 47
                                }
                            },
                            {
                                "username" : {
                                    "$lt" : "user8"
                                }
                            },
                            {
                                "username" : {
                                    "$gt" : "user5"
                                }
                            }
                        ]
                    },
                    "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                            "stage" : "IXSCAN",
                            "keyPattern" : {
                                "age" : 1,
                                "username" : 1
                            },
                            "indexName" : "age_1_username_1",
                            "isMultiKey" : false,
                            "multiKeyPaths" : {
                                "age" : [ ],
                                "username" : [ ]
                            },
                            "isUnique" : false,
                            "isSparse" : false,
                            "isPartial" : false,
                            "indexVersion" : 2,
                            "direction" : "forward",
                            "indexBounds" : {
                                "age" : [
                                    "[47.0, 47.0]"
                                ],
                                "username" : [
                                    "(\"user5\", \"user8\")"
                                ]
                            }
                        }
                    },
                    "rejectedPlans" : [
                        {
                            "stage" : "FETCH",
                            "filter" : {
                                "age" : {
                                    "$eq" : 47
                                }
                            },
                            "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                    "username" : 1
                                },
                                "indexName" : "username_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                    "username" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                    "username" : [
                                        "(\"user5\", \"user8\")"
                                    ]
                                }
                            }
                        }
                    ]
                },
                "executionStats" : {
                    "executionSuccess" : true,
                    "nReturned" : 2742,
                    "executionTimeMillis" : 5,
                    "totalKeysExamined" : 2742,
                    "totalDocsExamined" : 2742,
                    "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 2742,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 2743,
                        "advanced" : 2742,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 23,
                        "restoreState" : 23,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 2742,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                            "stage" : "IXSCAN",
                            "nReturned" : 2742,
                            "executionTimeMillisEstimate" : 0,
                            "works" : 2743,
                            "advanced" : 2742,
                            "needTime" : 0,
                            "needYield" : 0,
                            "saveState" : 23,
                            "restoreState" : 23,
                            "isEOF" : 1,
                            "invalidates" : 0,
                            "keyPattern" : {
                                "age" : 1,
                                "username" : 1
                            },
                            "indexName" : "age_1_username_1",
                            "isMultiKey" : false,
                            "multiKeyPaths" : {
                                "age" : [ ],
                                "username" : [ ]
                            },
                            "isUnique" : false,
                            "isSparse" : false,
                            "isPartial" : false,
                            "indexVersion" : 2,
                            "direction" : "forward",
                            "indexBounds" : {
                                "age" : [
                                    "[47.0, 47.0]"
                                ],
                                "username" : [
                                    "(\"user5\", \"user8\")"
                                ]
                            },
                            "keysExamined" : 2742,
                            "seeks" : 1,
                            "dupsTested" : 0,
                            "dupsDropped" : 0,
                            "seenInvalidated" : 0
                        }
                    }
                },
                "serverInfo" : {
                    "host" : "eoinbrazil-laptop-osx",
                    "port" : 27017,
                    "version" : "4.0.12",
                    "gitVersion" : "5776e3cbf9e7afe86e6b29e22520ffb6766e95d4"
                },
                "ok" : 1
            }
            ```
            
        - ex> `{"username" : 1, "age" : 1}`로 인덱스를 사용: 쿼리가 "user5"와 "user8" 사이의 사용자를 모두 살펴보고 "age" : 47인 사용자를 뽑아내야 하므로 쿼리 플랜을 변경 👉 이전 인덱스를 사용할 때의 100배가 되는 인덱스 항목을 살펴보도록 강제한다.
            
            ```bash
            > db.users.find({"age" : 47, "username" : {"$gt" : "user5", "$lt" : "user8"}})
                            .explain('executionStats')
            {
                "queryPlanner" : {
                    "plannerVersion" : 1,
                    "namespace" : "test.users",
                    "indexFilterSet" : false,
                    "parsedQuery" : {
                        "$and" : [
                            {
                                "age" : {
                                    "$eq" : 47
                                }
                            },
                            {
                                "username" : {
                                    "$lt" : "user8"
                                }
                            },
                            {
                                "username" : {
                                    "$gt" : "user5"
                                }
                            }
                        ]
                    },
                    "winningPlan" : {
                        "stage" : "FETCH",
                        "filter" : {
                            "age" : {
                                "$eq" : 47
                            }
                        },
                        "inputStage" : {
                            "stage" : "IXSCAN",
                            "keyPattern" : {
                                "username" : 1
                            },
                            "indexName" : "username_1",
                            "isMultiKey" : false,
                            "multiKeyPaths" : {
                                "username" : [ ]
                            },
                            "isUnique" : false,
                            "isSparse" : false,
                            "isPartial" : false,
                            "indexVersion" : 2,
                            "direction" : "forward",
                            "indexBounds" : {
                                "username" : [
                                    "(\"user5\", \"user8\")"
                                ]
                            }
                        }
                    },
                    "rejectedPlans" : [
                        {
                            "stage" : "FETCH",
                            "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                    "username" : 1,
                                    "age" : 1
                                },
                                "indexName" : "username_1_age_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                    "username" : [ ],
                                    "age" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                    "username" : [
                                        "(\"user5\", \"user8\")"
                                    ],
                                    "age" : [
                                        "[47.0, 47.0]"
                                    ]
                                }
                            }
                        }
                    ]
                },
                "executionStats" : {
                    "executionSuccess" : true,
                    "nReturned" : 2742,
                    "executionTimeMillis" : 369,
                    "totalKeysExamined" : 333332,
                    "totalDocsExamined" : 333332,
                    "executionStages" : {
                        "stage" : "FETCH",
                        "filter" : {
                            "age" : {
                                "$eq" : 47
                            }
                        },
                        "nReturned" : 2742,
                        "executionTimeMillisEstimate" : 312,
                        "works" : 333333,
                        "advanced" : 2742,
                        "needTime" : 330590,
                        "needYield" : 0,
                        "saveState" : 2697,
                        "restoreState" : 2697,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 333332,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                            "stage" : "IXSCAN",
                            "nReturned" : 333332,
                            "executionTimeMillisEstimate" : 117,
                            "works" : 333333,
                            "advanced" : 333332,
                            "needTime" : 0,
                            "needYield" : 0,
                            "saveState" : 2697,
                            "restoreState" : 2697,
                            "isEOF" : 1,
                            "invalidates" : 0,
                            "keyPattern" : {
                                "username" : 1
                            },
                            "indexName" : "username_1",
                            "isMultiKey" : false,
                            "multiKeyPaths" : {
                                "username" : [ ]
                            },
                            "isUnique" : false,
                            "isSparse" : false,
                            "isPartial" : false,
                            "indexVersion" : 2,
                            "direction" : "forward",
                            "indexBounds" : {
                                "username" : [
                                    "(\"user5\", \"user8\")"
                                ]
                            },
                            "keysExamined" : 333332,
                            "seeks" : 1,
                            "dupsTested" : 0,
                            "dupsDropped" : 0,
                            "seenInvalidated" : 0
                        }
                    }
                },
                "serverInfo" : {
                    "host" : "eoinbrazil-laptop-osx",
                    "port" : 27017,
                    "version" : "4.0.12",
                    "gitVersion" : "5776e3cbf9e7afe86e6b29e22520ffb6766e95d4"
                },
                "ok" : 1
            }
            ```
            
- OR 쿼리
    - 몽고DB는 쿼리당 하나의 인덱스만 사용할 수 있다. 유일한 예외는 "`$or`"다.
    - "`$or`"는 두 개의 쿼리를 수행하고 결과를 합치므로 "`$or`"은 "`$or`" 절마다 하나씩 인덱스를 사용할 수 있다.
        
        ```bash
        db.foo.find({"$or" : [{"x" : 123}, {"y" : 456}]}).explain()
        {
            "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "foo.foo",
                "indexFilterSet" : false,
                "parsedQuery" : {
                    "$or" : [
                        {
                            "x" : {
                                "$eq" : 123
                            }
                        },
                        {
                            "y" : {
                                "$eq" : 456
                            }
                        }
                    ]
                },
                "winningPlan" : {
                    "stage" : "SUBPLAN",
                    "inputStage" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                            "stage" : "OR",
                            "inputStages" : [
                                {
                                    "stage" : "IXSCAN",
                                    "keyPattern" : {
                                        "x" : 1
                                    },
                                    "indexName" : "x_1",
                                    "isMultiKey" : false,
                                    "multiKeyPaths" : {
                                        "x" : [ ]
                                    },
                                    "isUnique" : false,
                                    "isSparse" : false,
                                    "isPartial" : false,
                                    "indexVersion" : 2,
                                    "direction" : "forward",
                                    "indexBounds" : {
                                        "x" : [
                                            "[123.0, 123.0]"
                                        ]
                                    }
                                },
                                {
                                    "stage" : "IXSCAN",
                                    "keyPattern" : {
                                        "y" : 1
                                    },
                                    "indexName" : "y_1",
                                    "isMultiKey" : false,
                                    "multiKeyPaths" : {
                                        "y" : [ ]
                                    },
                                    "isUnique" : false,
                                    "isSparse" : false,
                                    "isPartial" : false,
                                    "indexVersion" : 2,
                                    "direction" : "forward",
                                    "indexBounds" : {
                                        "y" : [
                                            "[456.0, 456.0]"
                                        ]
                                    }
                                }
                            ]
                        }
                    }
                },
                "rejectedPlans" : [ ]
            },
            "serverInfo" : {
            ...,
            },
            "ok" : 1
        }
        ```
        
    - 일반적으로 두 번 쿼리해서 결과를 병합하면 한 번 쿼리할 때보다 훨씬 비효율적이다.
    - 가능하면 "`$or`"보다는 "`$in`"을 사용하자.

### 1-6. 객체 및 배열 인덱싱

---

- 몽고DB는 도큐먼트 내부에 도달해서 내장 필드와 배열에 인덱스를 생성하도록 허용한다.

- 내장 도큐먼트 인덱싱하기
    - 인덱스 → 내장 도큐먼트 키에 생성될 수 있다. 서브필드에 인덱스를 만들어 해당 필드를 이용하는 쿼리의 속도를 높일 수 있다.
        
        ```bash
        # 각 도큐먼트가 한 명의 사용자를 나타낸다면, 각 사용자의 위치가 명시된 내장 도큐먼트가 있다.
        {
            "username" : "sid",
            "loc" : {
                "ip" : "1.2.3.4",
                "city" : "Springfield", # "loc"의 서브필드
                "state" : "NY"
            }
        }
        ```
        
    - `db.users.createIndex({"loc.city": 1})`
- 배열 인덱싱하기
    - 배열에도 인덱스를 생성할 수 있다. 인덱스를 사용하면 배열의 특정 요소를 효율적으로 찾을 수 있다.
    - `db.blog.createIndex({"comments.date" : 1})`
    - 배열을 인덱싱하면 배열의 각 요소에 인덱스 항목을 생성 → 단일값 인덱스보다 더 부담
- 다중키 인덱스가 미치는 영향
    - 어떤 도큐먼트가 배열 필드를 인덱스 키로 가지면 인덱스는 즉시 다중키 인덱스로 표시된다.
    - 다중키 인덱스가 사용됐다면 "`isMultiKey`" 필드는 `true`다.
        - 인덱스는 일단 다중키로 표시되면 필드 내 배열을 포함하는 도큐먼트가 모두 제거되더라도 비다중키가 될 수 없다.
        - 비다중키가 되게 하려면 인덱스를 삭제하고 다시 생성해야만 한다.
    - 다중키 인덱스는 비다중키 인덱스보다 약간 느릴 수 있다.****
        - 하나의 도큐먼트를 여러 개의 인덱스 항목이 가리킬 수 있으므로 몽고DB는 결과를 반환하기 전에 중복을 제거해야 한다.

### 1-7. 인덱스 카디널리티

---

- 카디널리티는 컬렉션의 한 필드에 대해 고윳값이 얼마나 많은지 나타낸다.
    - "gender"나 "newsletter opt-out"과 같은 필드는 가질 수 있는 값이 두 가지뿐 → 매우 낮은 카디널리티
    - "username"이나 "email" 같은 필드는 컬렉션의 각 도큐먼트마다 유일한 값 → 높은 카디널리티
    - "age"나 "zip code"와 같은 필드 → 카디널리티가 중간쯤

- 일반적으로 필드의 카디널리티가 높을수록 인덱싱이 더욱 도움이 된다.****
    - 인덱스가 검색 범위를 훨씬 작은 결과 셋으로 빠르게 좁힐 수 있기 때문이다.

<aside>
💡 높은 카디널리티 키를 낮은 카디널리티 키보다 앞에 놓자.

</aside>

## 2. `explain` 출력

---

- `explain`은 쿼리에 대한 많은 정보를 제공하며, 느린 쿼리를 위한 중요한 진단 도구다.
    
    ```bash
    > test.users.find({"age" : 42}).explain('executionStats')
    {
        "queryPlanner" : {
            "plannerVersion" : 1,
            "namespace" : "test.users",
            "indexFilterSet" : false,
            "parsedQuery" : {
                "age" : {
                    "$eq" : 42
                }
            },
            "winningPlan" : {
                "stage" : "FETCH",
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "keyPattern" : {
                        "age" : 1,
                        "username" : 1
                    },
                    "indexName" : "age_1_username_1",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "age" : [ ],
                        "username" : [ ]
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "age" : [
                            "[42.0, 42.0]"
                        ],
                        "username" : [
                            "[MinKey, MaxKey]"
                        ]
                    }
                }
            },
            "rejectedPlans" : [ ]
        },
        "executionStats" : {
            "executionSuccess" : true,
            "nReturned" : 8449,
            "executionTimeMillis" : 15,
            "totalKeysExamined" : 8449,
            "totalDocsExamined" : 8449,
            "executionStages" : {
                "stage" : "FETCH",
                "nReturned" : 8449,
                "executionTimeMillisEstimate" : 10,
                "works" : 8450,
                "advanced" : 8449,
                "needTime" : 0,
                "needYield" : 0,
                "saveState" : 66,
                "restoreState" : 66,
                "isEOF" : 1,
                "invalidates" : 0,
                "docsExamined" : 8449,
                "alreadyHasObj" : 0,
                "inputStage" : {
                    "stage" : "IXSCAN",
                    "nReturned" : 8449,
                    "executionTimeMillisEstimate" : 0,
                    "works" : 8450,
                    "advanced" : 8449,
                    "needTime" : 0,
                    "needYield" : 0,
                    "saveState" : 66,
                    "restoreState" : 66,
                    "isEOF" : 1,
                    "invalidates" : 0,
                    "keyPattern" : {
                        "age" : 1,
                        "username" : 1
                    },
                    "indexName" : "age_1_username_1",
                    "isMultiKey" : false,
                    "multiKeyPaths" : {
                        "age" : [ ],
                        "username" : [ ]
                    },
                    "isUnique" : false,
                    "isSparse" : false,
                    "isPartial" : false,
                    "indexVersion" : 2,
                    "direction" : "forward",
                    "indexBounds" : {
                        "age" : [
                            "[42.0, 42.0]"
                        ],
                        "username" : [
                            "[MinKey, MaxKey]"
                        ]
                    },
                    "keysExamined" : 8449,
                    "seeks" : 1,
                    "dupsTested" : 0,
                    "dupsDropped" : 0,
                    "seenInvalidated" : 0
                }
            }
        },
        "serverInfo" : {
            "host" : "eoinbrazil-laptop-osx",
            "port" : 27017,
            "version" : "4.0.12",
            "gitVersion" : "5776e3cbf9e7afe86e6b29e22520ffb6766e95d4"
        },
        "ok" : 1
    }
    ```
    
- 쿼리가 "`COLLSCAN`"을 사용하면 인덱스를 사용하지 않음을 알 수 있다.

- `keyPattern`: 어떤 인덱스가 사용됐는지 나타낸다.
- `nReturned`: 실제로 반환된 도큐먼트 개수
- `totalKeysExamined`: 검색한 인덱스 항목 개수
- `totalDocsExamined`: 검색한 도큐먼트 개수
- `nscannedObjects`: 스캔한 도큐먼트 개수
- `executionTimeMillis`: 서버가 요청을 받고 응답을 보낸 시점까지 쿼리가 얼마나 빨리 실행됐는지. 최고로 뽑힌 플랜이 아니라 모든 플랜이 실행되기까지 걸린 시간을 반영한다.
- `isMultiKey`: 다중키 인덱스 사용 여부
- `stage`: 인덱스를 사용해 쿼리할 수 있었는지 여부. "`COLLSCAN`"은 인덱스로 쿼리할 수 없어 컬렉션 스캔을 수행했음을 뜻한다.
- `needYields`: 쓰기 요청을 처리하도록 쿼리가 양보(일시 중지)한 횟수. 대기 중인 쓰기가 있다면 쿼리는 일시적으로 락을 해제하고 쓰기가 처리되게 한다.
- `indexBounds`: 인덱스가 어떻게 사용됐는지 설명하며 탐색한 인덱스의 범위를 제공한다.

## 3. 인덱스를 생성하지 않는 경우

---

- 인덱스는 데이터의 일부를 조회할 때 가장 효율적이며 어떤 쿼리는 인덱스가 없는 게 더 빠르다.
    - 인덱스는 컬렉션에서 가져와야 하는 부분이 많을수록 비효율적 → 인덱스를 하나 사용하려면 두 번의 조회를 해야 하기 때문이다.
    - 한 번은 인덱스 항목을 살펴보고, 또 한 번은 도큐먼트를 가리키는 인덱스의 포인터를 따라간다.
- 인덱스가 도움이 될지 혹은 방해가 될지 알 수 있는 공식은 없다.
- 대체로 쿼리가 컬렉션의 30% 이상을 반환하는 경우 인덱스는 종종 쿼리 속도를 높인다.
    
    
    | 인덱스가 적합한 경우 | 컬렉션 스캔이 적합한 경우 |
    | --- | --- |
    | 큰 컬렉션 | 작은 컬렉션 |
    | 큰 도큐먼트 | 작은 도큐먼트 |
    | 선택적 쿼리 | 비선택적 쿼리 |

## 4. 인덱스 종류

---

### 4-1. 고유 인덱스

---

- 고유 인덱스는 각 값이 인덱스에 최대 한 번 나타나도록 보장한다.
    - ex> 여러 도큐먼트에서 "firstname" 키에 동일한 값을 가질 수 없도록 하려면 "firstname" 필드가 있는 도큐먼트에 대해서만 "`partialFilterExpression`"으로 고유 인덱스를 만들면 된다.
        
        ```bash
        > db.users.createIndex({"firstname" : 1},
        ... {"unique" : true, "partialFilterExpression":{
             "firstname": {$exists: true } } } )
        {
            "createdCollectionAutomatically" : false,
            "numIndexesBefore" : 3,
            "numIndexesAfter" : 4,
            "ok" : 1
        }
        ```
        
    - 중복키 예외를 발생시킴
        
        ```bash
        > db.users.insert({firstname: "bob"})
        WriteResult({ "nInserted" : 1 })
        > db.users.insert({firstname: "bob"})
        WriteResult({
          "nInserted" : 0,
          "writeError" : {
            "code" : 11000,
            "errmsg" : "E11000 duplicate key error collection: test.users index:
                       firstname_1 dup key: { : \"bob\" }"
          }
        })
        ```
        

- 고유 인덱스인 "`_id`"의 인덱스는 컬렉션을 생성하면 항상 자동으로 생성된다. 삭제할 수 없다.

- 복합 고유 인덱스
    - 개별 키는 같은 값을 가질 수 있지만 인덱스 항목의 모든 키에 걸친 값의 조합은 인덱스에서 최대 한 번만 나타난다.

- 중복 제거하기
    - 기존 컬렉션에 고유 인덱스를 구축할 때 중복된 값이 있으면 실패한다.
        
        ```bash
        > db.users.createIndex({"age" : 1}, {"unique" : true})
        WriteResult({
            "nInserted" : 0,
            "writeError" : {
                "code" : 11000,
                "errmsg" : "E11000 duplicate key error collection:
                           test.users index: age_1 dup key: { : 12 }"
            }
        })
        ```
        

### 4-2. 부분 인덱스

---

- 고유 인덱스는 `null`을 값으로 취급하므로, 키가 없는 도큐먼트가 여러 개인 고유 인덱스를 만들 수 없다.
    - 하지만 오직 키가 존재할 때만 고유 인덱스가 적용되도록 할 때가 많다.
    
    → 고유한 필드가 존재하거나 필드가 아예 존재하지 않으면 "`unique`"와 "`partial`"을 결합할 수 있다.
    

- 부분 인덱스를 만들려면 "`partialFilterExpression`" 옵션을 포함시킨다.
    
    ```bash
    # 이메일 주소는 선택 항목이지만 입력할 경우 고유해야 한다
    > db.users.ensureIndex({"email" : 1}, {"unique" : true, "partialFilterExpression" : { email: { $exists: true } }})
    ```
    
- 부분 인덱스는 반드시 고유할 필요는 없다.
    - 고유하지 않은 부분 인덱스를 만들려면 "`unique`" 옵션을 제외시키기만 하면 된다.

<aside>
💡 필드가 없는 도큐먼트가 필요할 때는 `hint`를 사용함으로써 테이블 스캔을 하도록 강제할 수 있다.

</aside>

## 5. 인덱스 관리

---

- 데이터베이스의 인덱스 정보는 모두 `system.indexes` 컬렉션에 저장된다.****
    - 예약된 컬렉션
    - `createIndex`, `createIndexes`, `dropIndexes`와 같은 데이터베이스 명령으로만 조작할 수 있다.
- `db.컬렉션명.getIndexes()`: 특정 컬렉션의 모든 인덱스 정보를 확인
    
    ```bash
    > db.students.getIndexes()
    [
        {
            "v" : 2,
            "key" : {
                "_id" : 1
            },
        "name" : "_id_",
        "ns" : "school.students"
        },
        {
        "v" : 2,
        "key" : {
            "class_id" : 1
        },
        "name" : "class_id_1",
        "ns" : "school.students"
        },
        {
        "v" : 2,
        "key" : {
            "student_id" : 1,
            "class_id" : 1
        },
        "name" : "student_id_1_class_id_1",
        "ns" : "school.students"
        }
    ]
    ```
    

### 5-1. 인덱스 식별

---

- 컬렉션 내 각 인덱스는 고유하게 식별하는 이름이 있다.
- 인덱스명은 서버에서 인덱스를 삭제하거나 조작하는 데 사용된다.
- `createIndex` 옵션으로 원하는 이름을 지정할 수 있다.
    
    ```bash
    > db.soup.createIndex({"a" : 1, "b" : 1, "c" : 1, ..., "z" : 1},
    ... {"name" : "alphabet"})
    ```
    

### 5-2. 인덱스 변경

---

- `dropIndex`: 인덱스를 제거
    
    ```bash
    > db.people.dropIndex("x_1_y_1")
    { "nIndexesWas" : 3, "ok" : 1 }
    ```
    

- 새로운 인덱스를 구축하려면 시간이 오래 걸리고 리소스가 많이 필요하다.
    - 몽고DB 4.2 이후 버전에서는 인덱스를 최대한 빨리 구축하기 위해, 구축이 완료될 때까지 데이터베이스의 모든 읽기와 쓰기를 중단한다.
- 데이터베이스가 읽기와 쓰기에 어느 정도 응답하게 하려면 인덱스를 구축할 때 "`background`" 옵션을 사용하자.
    - 이는 인덱스 구축이 종종 다른 작업에 양보하도록 강제하지만 여전히 애플리케이션에 큰 영향을 준다.
    - 포그라운드 인덱싱보다 훨씬 느리다.

- 몽고DB 4.2는 하이브리드 인덱스 구축이라는 새로운 접근 방식을 도입했다.
    - 인덱스 구축 프로세스의 시작과 끝에만 락을 가짐
    - 프로세스의 나머지 부분은 읽기 및 쓰기 작업을 인터리빙한다.
    
    → 포그라운드, 백그라운드 인덱싱을 모두 대체한다.
    

- 기존 도큐먼트에 인덱스를 생성하면 인덱스를 먼저 생성한 후 모든 도큐먼트를 끼워 넣을 때보다 조금 빠르다.

- 참고
    - [https://github.com/arkss/TIL/blob/master/MONGODB/02_몽고DB개발/05_인덱싱.md](https://github.com/arkss/TIL/blob/master/MONGODB/02_%EB%AA%BD%EA%B3%A0DB%EA%B0%9C%EB%B0%9C/05_%EC%9D%B8%EB%8D%B1%EC%8B%B1.md)
    - [https://jackjeong.tistory.com/m/174](https://jackjeong.tistory.com/m/174)
    - [https://github.com/WouNice/General-Notes/blob/master/MongoDB权威指南/Part 2 设计应用程序/第 5 章 索引.md](https://github.com/WouNice/General-Notes/blob/master/MongoDB%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97/Part%202%20%E8%AE%BE%E8%AE%A1%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F/%E7%AC%AC%205%20%E7%AB%A0%20%E7%B4%A2%E5%BC%95.md)
