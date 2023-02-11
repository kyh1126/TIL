# Chapter 17. 샤딩 관리

- 샤드 클러스터를 수동으로 관리하는 방법을 다룬다.

## 1. 현재 상태 확인

---

- 데이터의 위치, 샤드 구성, 클러스터 실행 작업을 조회할 수 있는 몇 가지 보조자가 있다.

### 1-1. `sh.status()`를 사용해 요약 정보 얻기

---

- `sh.status()`는 샤드, 데이터베이스, 샤딩된 컬렉션의 개요를 제공한다.
    - 청크의 개수가 적다면 어느 청크가 어디에 위치하는지도 출력한다.
    - 그렇지 않으면 단순히 컬렉션의 샤드 키와 각 샤드가 갖는 청크 개수를 보고한다.

```bash
> sh.status()
--- Sharding Status ---
  sharding version: {
   "_id" : <num>,
   "minCompatibleVersion" : <num>,
   "currentVersion" : <num>,
   "clusterId" : <ObjectId>
}
shards:
 { "_id" : <shard name1>, "host" : <string>, "tags" : [ <string> ... ], "state" : <num> }
 { "_id" : <shard name2>, "host" : <string>, "tags" : [ <string> ... ], "state" : <num> }
 ...
active mongoses:
  <version> : <num>
autosplit:
  Currently enabled: <yes|no>
balancer:
      Currently enabled:  yes
      Currently running:  yes
      Collections with active migrations:
              config.system.sessions started at Fri May 15 2020 17:38:12 GMT-0400 (EDT)
      Failed balancer rounds in last 5 attempts:  0
      Migration Results for the last 24 hours:
             416 : Success
             1 : Failed with error 'aborted', from shardA to shardB
      databases:
       { "_id" : <dbname1>, "primary" : <string>, "partitioned" : <boolean>, "version": <document> }
        <dbname>.<collection>
           shard key: { <shard key> : <1 or hashed> }
           unique: <boolean>
           balancing: <boolean>
           chunks:
              <shard name1> <number of chunks>
              <shard name2> <number of chunks>
              ...
           { <shard key>: <min range1> } -->> { <shard key> : <max range1> } on : <shard name> <last modified timestamp>
           { <shard key>: <min range2> } -->> { <shard key> : <max range2> } on : <shard name> <last modified timestamp>
           ...
           tag: <tag1>  { <shard key> : <min range1> } -->> { <shard key> : <max range1> }
           ...
       { "_id" : <dbname2>, "primary" : <string>, "partitioned" : <boolean>, "version": <document> }
       ...
```

- 청크가 여러 개이면 `sh.status()`는 각 청크를 출력하기보다는 청크 상태를 요약한다.
- 모든 청크를 보려면 `sh.status(true)`를 실행한다.(`true`를 입력하면 상세한 정보를 출력한다)
- `sh.status()`는 `config` 데이터베이스에서 수집한 정보를 모두 보여준다.

### 1-2. 구성 정보 확인하기

---

- 클러스터에 대한 모든 구성 정보는 구성 서버의 `config` 데이터베이스 내 컬렉션에 보관된다.
- 셸 보조자를 사용하면 구성 정보를 더 읽기 쉽게 볼 수 있다. 하지만 클러스터에 대한 메타데이터를 조회하려면 언제든지 `config` 데이터베이스를 직접 쿼리할 수 있다.
    
    ```bash
    > use config
    ```
    
- 일반적으로 `config` 데이터베이스에 들어 있는 데이터는 직접 수정하면 안 된다. 무언가를 수정하고 나면 모든 `mongos` 서버를 재시작해야 변경 사항이 적용된다.

- config.shards: 클러스터 내 모든 샤드를 추적한다.
    
    ```bash
    > db.shards.find()
    {
      "_id" : "shard01",
      "host" : "shard01/localhost:27018,localhost:27019,localhost:27020",
      "state" : 1
    }
    {
      "_id" : "shard02",
      "host" : "shard02/localhost:27021,localhost:27022,localhost:27023",
      "state" : 1
    }
    
    {
      "_id" : "shard03",
      "host" : "shard03/localhost:27024,localhost:27025,localhost:27026",
      "state" : 1
    }
    ```
    
    - 샤드의 "`_id`"는 복제 셋의 이름에서 가져온다. 그러므로 클러스터 안에 있는 각 복제 셋의 이름은 고유해야 한다.
- config.databases: 클러스터가 알고 있는 모든 샤딩 및 비샤딩 데이터베이스를 추적한다.
    
    ```bash
    > db.databases.find()
    { "_id" : "video",
      "primary" : "shard02",
      "partitioned" : true,
      ...
    }
    ```
    
    - `enableSharding`이 데이터베이스에서 실행된 적이 있으면 "`partitioned`"는 `true`가 된다.
    - "`primary`"는 데이터베이스의 '홈 베이스'다.
- config.collections: 모든 샤딩된 컬렉션을 추적한다.
    
    ```bash
    > db.collections.find().pretty()
    {
      "_id": "video.movies",
      ...
      "key" : {
        "imdbId":"hashed"
      },
      "unique" : false,
      ...
    }
    ```
    
    - "`_id`": 컬렉션의 네임스페이스
    - "`key`": 샤드 키. 여기서는 "`imdbId`"에 해시된 샤드 키
    - "`unique`": 샤드 키가 고유한 인덱스는 아님을 나타낸다. 기본적으로 샤드 키는 고유하지 않다.
- config.chunks: 모든 컬렉션 내 모든 청크의 기록을 보관한다.
    
    ```bash
    > db.chunks.find().skip(1).limit(1).pretty()
    {
      "_id" : "video.movies-imdbId_MinKey",
      "lastmod" : Timestamp(2,0),
      "lastmodEpoch" : ObjectId("5db........"),
      "ns" : "video.movies",
      "min" : {
        "imdbid" : {"$minKey":1}
      },
      "max" : {
        "imdbid" : {NumberLong("-72........")
      },
      "shard" : "shard01",
      ... 
    }
    ```
    
    - "`_id`": 청크의 고유 식별자. 일반적으로 네임스페이스, 샤드 키, 하위 청크 범위다.
    - "`ns`": 청크가 위치한 컬렉션
    - "`min`": 청크 범위(경계 포함) 내 최솟값
    - "`max`": 청크 내 모든 값은 이 값보다 작다.
    - "`shard`": 청크가 위치하는 샤드
    - "`lastmod`": 청크 버전 관리를 추적한다.
        - ex> "video.movies-imdbId_MinKey" 청크가 두 개의 청크로 분할됐다면, 이전의 단일 청크와 분할돼서 작아진 "video.movies-imdbId_MinKey" 청크를 구별하는 방법이 필요하다. 따라서 Timestamp 값의 첫 번째 구성 요소는 청크가 새 샤드로 마이그레이션된 횟수를 반영한다. 두 번째 구성 요소는 분할 수를 반영한다.
    - "`lastmodEpoch`": 컬렉션의 생성 시기를 지정하며, 컬렉션이 삭제되고 즉시 재생성될 때 동일한 컬렉션 이름에 대한 요청을 구별하는 데 사용된다.****
- config.changelog: 발생한 분할과 마이그레이션을 모두 기록하므로 클러스터가 무엇을 하고 있는지 추적하는 데 유용하다.
    - 분할
        
        ```bash
        > db.changelog.find({what: "split"}).pretty()
        {
          ....
          "detail" : {
            "before" : {
              ...
              "lastmod":Timestamp(9,1),
              "lastmodEpoch":Objectid("5bdf72c021b6e...")
            },
            "left" : {
              ...
              "lastmod":Timestamp(9,2),
              "lastmodEpoch":Objectid("5bdf72c021b6e...")    
            },
            "right" : {
              ...
              "lastmod":Timestamp(9,3),
              "lastmodEpoch":Objectid("5bdf72c021b6e...")    
            }
          }
        }
        ```
        
        - 출력 결과는 컬렉션의 첫 번째 청크 분할 모습
            - 각각의 새 청크에서 "`lastmod`"의 두 번째 구성 요소가 갱신됐고, 값은 각각 `Timestamp(9,2)`와 `Timestamp(9,3)`이다.
    - 마이그레이션
        - 네 개로 나뉜 체인지로그 도큐먼트가 생긴다.
            - 하나는 마이그레이션의 시작을 명시
            - 하나는 '원본' 샤드
                
                ```bash
                > db.changelog.findOne({what:"moveChunk.to"}) // '원본' 샤드에 의해 생성된 도큐먼트
                {
                  ...
                  "what" : "moveChunk.to",
                  "ns" : "video.movies",
                  ...
                  "details" : {
                    ...
                    "step 1 of 6" : 965,
                    "step 2 of 6" : 608,
                    "step 2 of 6" : 15424,
                    "step 4 of 6" : 0,
                    "step 5 of 6" : 72,
                    "step 6 of 6" : 258,
                    "note" : "success"
                    }
                }
                ```
                
                - "`details`"에 나열된 각 단계에서 시간이 기록되며 "stepN of N" 메세지는 단계의 소요 시간을 밀리초 단위로 보여준다.
                - '원본' 샤드가 `mongos`로부터 `moveChunk` 명령을 받으면 다음과 같은 작업을 수행한다.
                    1. 명령 매개변수를 확인한다.
                    2. 구성 서버가 마이그레이션을 위해 분산된 락을 획득할 수 있는지 확인한다.
                    3. '목적지' 샤드에 접속을 시도한다.
                    4. 데이터를 복사한다. 이는 'the critical section'으로 명시돼 기록된다.
                    5. '목적지' 샤드 및 구성 서버와 조정해 이동을 확정한다.
                - '목적지'와 '원본' 샤드는 "step 4 of 6"부터는 서로 통신이 가능한 상태여야 함. 마지막 단계에서 원본 서버의 네트워크 연결이 비정상적이라면, 마이그레이션을 되돌리거나 계속 진행할 수 없는 상태가 된다. 이럴 때는 `mongod`가 종료된다.
            - 하나는 '목적지' 샤드
                
                ```bash
                > db.changelog.find({what:"moveChunk.from","details.max.imdbId": .....).pretty()
                {
                  ...
                  "what" : "moveChunk.from",
                  "ns" : "video.movies",
                  "detail" : {
                  ...
                    "step 1 of 6" : 0,
                    "step 2 of 6" : 4,
                    "step 2 of 6" : 191,
                    "step 4 of 6" : 17000,
                    "step 5 of 6" : 341,
                    "step 6 of 6" : 39,
                    "to" : "shard01"
                    "from" : "shard02",
                    "note" : "success"
                  }
                }
                ```
                
                - '목적지' 샤드가 '원본' 샤드로부터 명령을 받으면 다음 작업을 수행한다.
                    1. 인덱스를 마이그레이션한다.
                    2. 청크 범위 안에 있는 모든 데이터를 삭제한다.
                    3. 청크 내 모든 도큐먼트를 '목적지' 샤드로 복사한다.
                    4. 복사하는 동안에 도큐먼트에 일어난 작업을 재실행한다.
                    5. '목적지' 샤드가 새로 마이그레이션된 데이터를 대부분의 서버로 복제하도록 기다린다.
                    6. 청크의 메타데이터가 '목적지' 샤드에 위치한다고 바꿔서 마이그레이션을 확정한다.
            - 하나는 마이그레이션에 대한 커밋 정보를 포함한다.
        - '원본' 샤드와 '목적지' 샤드에 대한 도큐먼트는 프로세스에서 단계별로 소요된 시간의 명세를 제공한다.
            - 마이그레이션에서 병목 현상을 일으키는 원인이 디스크인지, 네트워크 혹은 다른 무언가인지 추정할 수 있다.
- config.settings: 현재의 밸런서 설정과 청크 크기를 나타내는 도큐먼트를 포함한다. 컬렉션 내 도큐먼트를 변경해서 밸런서를 켜고 끄거나 청크 크기를 변경할 수 있다.

## 2. 네트워크 연결 추적

---

### 2-1. 연결 통계 정보 얻어오기

---

- `connPoolStats`: 현재 데이터베이스 인스턴스에서 샤드 클러스터나 복제 셋의 다른 멤버로 나가는 열린 연결에 대한 정보를 반환한다.
    - 실행 중인 작업과 간섭을 피하려고 락을 사용하지 않는다 → 정보를 수집할 때 개수가 약간 바뀌어, 호스트와 풀 연결 개수 간에 약간의 차이가 생길 수 있다.

```bash
> db.adminCommnad({"connPoolStats":1})
{
  "numClientConnections" : 10,
  "numAScopedConnections" : 0,
  "totalInUse" : 0,
  "totalAvailable" : 13,
  "totalCreated" : 86,
  "totalRefreshing" : 0,
  "pools" : {
    ...
    "localhost:27017" : {
      "inUse" : 0,
      "available" : 1,
      "created" : 1,
      "refreshing" : 0
    }
    ...
}
```

- "`totalAvailable`": 현재 `mongod`/`mongos` 인스턴스에서 샤드 클러스터나 복제 셋의 다른 구성원으로 나가는 사용 가능한 총 발신 연결 수를 나타낸다.
- "`totalCreated`": 현재 `mongod`/`mongos` 인스턴스가 샤드 클러스터나 복제 셋의 다른 구성원으로 생성한 총 발신 연결 수를 나타낸다.
- "`totalInUse`": 현재 `mongod`/`mongos` 인스턴스에서 현재 사용 중인 샤드 클러스터나 복제 셋의 다른 구성원으로 나가는 총 연결 수를 나타낸다.
- "`totalRefreshing`": 현재 `mongod`/`mongos` 인스턴스에서 현재 새로 새로고침되는 샤드 클러스터나 복제 셋의 다른 구성원으로 나가는 총 발신 연결 수를 나타낸다.
- "`numClientConnections`": 현재 `mongod`/`mongos` 인스턴스에서 샤드 클러스터나 복제 셋의 다른 구성원으로 나가는 활성 및 저장된 동기 연결 수를 나타낸다.
- "`numAScopedConnections`": 현재 `mongod`/`mongos` 인스턴스에서 샤드 클러스터나 복제 셋의 다른 구성원으로 나가는 활성 및 저장된 범위 동기 연결 수를 보고한다.
- "`pools`": 연결 풀별로 그룹화된 연결 통계(사용 중/사용가능/생성됨/새로고침 중)를 나타낸다.
    - DBClient 기반 풀("`pools`" 도큐먼트에서 "`global`" 필드명으로 식별되는 '쓰기 경로')
    - NetworkInterfaceTL 기반 풀('읽기 경로')
- "`hosts`": 호스트별로 그룹화된 연결 통계를 나타낸다. 현재 `mongod`/`mongos` 인스턴스와 샤드 클러스터나 복제 셋의 각 구성원 간의 연결에 대해 보고한다.

### 2-2. 연결 개수 제한하기

---

- 클라이언트가 `mongos`에 접속하면 `mongos`는 클라이언트의 요청을 전달하려고 적어도 하나의 샤드에 연결을 생성한다.
- `mongos` 프로세스가 여러 개이면 샤드가 다룰 수 있는 양보다 더 많은 연결을 생성할 수도 있다. `mongos`는 기본적으로 최대 6만 5536개의 연결을 허용하며, 각각 1만 개의 클라이언트 연결을 갖는 5개의 `mongos` 프로세스가 있다면 하나의 샤드에 5만 개의 연결 생성을 시도한다.
- 이를 방지하려면 `mongos` 명령행 구성에서 `--maxConns` 옵션을 사용해 생성 가능한 연결 개수를 제한하면 된다.
- 다음 공식은 하나의 `mongos`에서 샤드가 다룰 수 있는 연결의 최대 수를 계산하는 데 사용한다.
    
    ```bash
    maxConns = maxConnsPrimary - (numMembersPerReplicaSet x 3)
    
    (other x 3) / numMongosProcesses
    ```
    
    - : 부수적인 역할을 하는 프로세스 개수 / 샤드클러스터에서 mongos의 총 개수
    - `maxConnsPrimary`: 프라이머리의 최대 연결 수. `mongos`로부터의 연결로 샤드를 압도하지 않도록 일반적으로 2만으로 설정한다.
    - `(numMembersPerReplicaSet x 3)`: 프라이머리는 각 세컨더리에 연결을 하나씩 생성하고, 세컨더리는 프라이머리에 두 개의 연결을 생성해서 총 세 개의 연결이 있다.
    - `(other x 3)`: `other`는 모니터링이나 백업 에이전트와 같이 `mongod`에 접속하거나, 직접 셸에 연결하거나, 이동을 위해 다른 샤드로 연결하는 등 부수적인 역할을 하는 프로세스 개수다.
    - `numMongosProcesses`: 샤드 클라스터에서 `mongos`의 총 개수
- ❗`--maxConns`는 단지 mongos가 연결을 이 이상 생성하지 않도록 함
    - 한계에 이를 때 특별히 도움 되는 동작을 하지는 않으며, 그저 요청을 막고 연결이 '해제'되기를 기다린다.

## 3. 서버 관리

---

- 클러스터가 커지면 용량을 추가하거나 구성을 바꿀 필요가 있다.****

### 3-1. 서버 추가

---

- 언제든지 새로운 `mongos` 프로세스를 추가할 수 있다.****
    - `mongos`의 `—-configdb` 옵션이 올바른 구성 서버 셋을 명시하는지 확인하자.
    - `mongos`에 클라이언트가 즉시 연결할 수 있어야 한다.
- 새로운 샤드를 추가하려면 15장에서 봤듯 `addShard` 명령을 사용한다.

### 3-2. 샤드의 서버 변경

---

- 샤드 클러스터를 사용할 때, 각 샤드에서 서버를 변경할 수도 있다.****
    - 샤드의 멤버 구성을 변경하려면 (`mongos`를 통하지 않고) 샤드의 프라이머리에 직접 연결해 복제 셋을 재구성하자.
    - 클러스터 구성은 변경 사항을 가져와서 `config.shards`를 자동으로 수정한다.
- 샤드를 독립 실행형 서버에서 복제 셋으로 변경하기
    - 가장 쉬운 방법은 빈 복제 셋 샤드를 새로 추가하고 독립 실행형 서버 샤드를 제거하는 방법
    - 마이그레이션은 데이터를 새로운 샤드로 옮기는 작업을 한다.

### 3-3. 샤드 제거

---

- 일반적으로 샤드는 클러스터에서 제거되면 안 된다. 주기적으로 샤드를 추가하거나 제거하면 시스템에 필요 이상으로 무리를 준다. 하지만 필요하다면 샤드를 제거할 수 있다.
- 우선 벨런서가 켜진 것을 확인한다.
    - 밸런서: 배출(draining)이라는 프로세스에서, 제거하려는 샤드의 모든 데이터를 다른 샤드로 이동하는 임무가 있다.
- 배출을 시작하려면 `removeShard` 명령을 실행한다. `removeShard`는 샤드명을 가져와서 샤드의 모든 청크를 다른 샤드로 배출한다.
    
    ```bash
    > db.adminCommand({"removeShard":"shard03"})
    {
      ...
      "state" : "started",
      "shard" : "shard03",
      "note" : "you need to drop or movePrimary these database",
      ...
    }
    
    > db.adminCommand({"removeShard":"shard02"})
    {
      ...
      "state" : "ongoing",
      "remaining" : {
        "chunks" : NumberLong(3),
        "dbs" : NumberLong(0)
      },
      ...
    }
    ```
    
    - `removeShard`는 원하는 만큼 여러 번 실행할 수 있다.
    - 청크를 옮기려면 분할해야 할 수도 있으므로, 배출하는 동안 시스템에서 청크 개수를 증가할 수도 있다.
        - ex> 샤드가 5개인 클러스터에서 청크가 다음처럼 분포한다고 가정하자
            
            ```bash
            test-rs0 10
            test-rs1 10
            test-rs2 10
            test-rs3 11
            test-rs4 11
            ```
            
            - 이 클러스터에는 총 52개의 청크가 있다.
            - test-rs3을 제거하면 다음처럼 된다.
                
                ```bash
                test-rs0 15
                test-rs1 15
                test-rs2 15
                test-rs4 15
                ```
                
                - 18개는 test-rs3에서 나왔다.(11개는 시작할 때부터 있었고, 7개는 배출 분할에서 생성됐다)
- 모든 청크가 옮겨졌으면, 제거된 샤드를 프라이머리로 하는 데이터베이스가 있다면 샤드를 제거하기 전에 먼저 제거해야 한다.
    - 제거하려는 샤드가 클러스터 데이터베이스 중 하나의 프라이머리이면 `removeShard`는 "`dbsToMove`" 필드에 데이터베이스를 나열한다.
    - 샤드 제거를 완료하려면 샤드에서 모든 데이터를 마이그레이션한 후 데이터베이스를 새 샤드로 이동하거나, 데이터베이스를 삭제해야 한다.
    - 제거를 완료하려면 `movePrimary` 명령을 사용해 나열된 데이터베이스를 이동한다.
        
        ```bash
        > db.adminCommand({"movePrimary":"video", "to":"shard01"})
        ```
        
    - 데이터베이스를 모두 옮겼으면 `removeShard`를 한 번 더 실행한다.
        - 이 과정은 꼭 필요하지는 않지만 처리가 완료됐는지 확인할 수 있다.
        
        ```bash
        > db.adminCommand({"removeShard":"shard02"})
        {
          ...
          "state" : "completed",
          "shard" : "shard03",
          ...
        }
        ```
        

## 4. 데이터 밸런싱

---

- 일반적으로 몽고 DB는 자동적으로 데이터를 균등하게 분산한다.****
- 자동 밸런싱 작업을 켜고 끄는 방법과 밸런싱 과정에 개입하는 방법을 다룬다.

### 4-1. 밸런서

---

- 대부분의 관리 작업에서는 밸런서를 끄는 작업이 선행된다.
    
    ```bash
    > sh.setBalancerState(false)
    ```
    

- 밸런서가 꺼진 상태에서는 새로운 밸런싱 작업이 시작되지 않는다.
    - 하지만 진행 중에 밸런서를 끄면 밸런싱 작업이 강제로 중지되지 않는다.
    - `config.locks` 컬렉션을 확인해 밸런싱 작업이 아직 진행 중인지 확인해야 한다.
        
        ```bash
        > db.locks.find({"_id" : "balancer"})["state"]
        0 // 꺼져 있음
        ```
        

- 밸런싱 작업은 시스템에 부하를 준다.
    - 목적지 샤드는 원본 샤드 청크 안에 있는 모든 도큐먼트를 쿼리하고 목적지 샤드에 입력해야 한다.
    - 원본 샤드는 도큐먼트를 삭제해야 한다.

- 마이그레이션이 성능 문제를 일으킬 수 있는 두 가지 상황
    1. 핫스팟 샤드 키를 사용하면 지속적으로 마이그레이션이 일어난다.(새로운 청크가 모두 핫스팟에 생성되므로) 시스템은 핫스팟 샤드에서 발생하는 데이터 흐름을 처리할 수 있어야 한다.
    2. 새로운 샤드를 추가하면 밸런서가 해당 샤드를 채우려고 시도할 때 지속적인 마이그레이션이 발생한다.

💡마이그레이션이 애플리케이션 성능에 영향을 미친다고 판단하면 `config.settings` 컬렉션에서 밸런싱 작업할 시간대를 예약하자.

- ex> 밸런싱 작업이 오후 1시에서 4시 사이에 실행되도록 다음 갱신을 실행하자.
    - 1. 먼저 밸런서가 켜져 있는지 확인한 다음 시간대를 설정한다.
        
        ```bash
        > sh.setBalancerState(true)
        ..
        > db.settings.update(
          { _id : "balancer" },
          { $set : { activeWindow : {start : "13:00", stop : "16:00"} } },
          { upsert : true }
        )
        WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
        ```
        
    1. 할당한 시간에 `mongos`가 균형 잡힌 클러스터를 유지할 수 있는지 면밀히 살펴보자.

❗수동 밸런싱 작업을 자동 밸런서와 같이 사용하려 한다면 주의해야 한다.

- 자동 밸런서는 항상 셋의 현재 상태를 기준으로 무엇을 이동할지 결정하며, 셋의 이력을 고려하지 않는다.
    - ex> 각각 500개의 청크를 갖는 shardA와 shardB가 있다고 가정하자.
        - shardA가 많은 쓰기 요청을 받게 돼 밸런서를 끄고 가장 활발한 청크 30개를 shardB로 옮겼다.
        - 이때 다시 밸런서를 켜면 청크 개수를 맞추려고 밸런서는 즉시 30개의 청크를 낚아채서 shardB 에서 shardA로 이동한다.

→ 이를 방지하려면 밸런서를 시작하기 전에 활발하지 않은 청크 30개를 shardB에서 shardA로 옮기면 된다. 그러면 샤드 간 불균형은 없고 밸런서는 청크를 그대로 둔다.

### 4-2. 청크 크기 변경

---

- 청크에는 도큐먼트가 0개부터 수백만 개까지 존재할 수 있다. 일반적으로 청크가 클수록 다른 샤드로 이동하는 데 시간이 많이 걸린다.
    - 기본적으로 청크 크기는 64메가바이트.

- 때로는 64메가바이트 청크로는 마이그레이션이 너무 오래 걸린다. 청크 크기를 줄여서 속도를 높일 수 있다.
    - 셸을 통해서 `mongos`에 연결하고 `config.settings` 컬렉션을 다음처럼 변경하자.
        
        ```bash
        > db.settings.findOne()
        {
          "_id" : "chunksize",
          "value" : 64
        }
        > db.settings.save({"_id": "chunksize", "value": 32})
        WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
        ```
        
    - 기존 청크가 즉시 변경되지는 않는다. 자동 분할은 삽입이나 갱신을 할 때만 발생한다. 따라서 청크 크기를 줄이고 나서 모든 청크가 새로운 크기로 분할되는 데는 시간이 걸린다.
- 분할은 취소할 수 없다. 청크 크기를 늘리면 기존 청크는 새 크기에 이를 때까지 삽입이나 갱신을 통해서만 커진다. 청크 크기의 허용 범위는 1~1024메가바이트다.(경계 포함)

❗이는 모든 컬렉션과 데이터베이스에 영향을 미치는 클러스터 전역 설정임을 알아두자.****

### 4-3. 청크 이동

---

- 청크 내 모든 데이터는 특정 샤드에 자리 잡는다. 그 샤드가 다른 샤드보다 더 많은 청크를 가지면 몽고DB는 일부 청크를 옮겨버린다.
- 수동으로 청크를 이동
    
    ```bash
    > sh.moveChunk("video.movies", {imdbId : 500000}, "shard02")
    { "millis": 4079, "ok": 1 }
    ```
    
    - 옮길 청크를 찾으려면 샤드 키를 사용해야 한다.(여기서는 "imdbId")
        - "imdbId"가 500000인 도큐먼트가 포함된 청크를 shard02로 이동한다.
        - 가장 쉬운 방법은 청크의 하한값으로 지정하는 방법이다.
    - 이 명령은 결과를 반환하기 전에 청크를 이동하기 때문에 실행하는 데 시간이 걸린다.
- 청크가 최대 청크 크기보다 더 크면 `mongos`는 청크를 옮기기를 거부한다.
    
    ```bash
    > sh.moveChunk("video.movies" , {imdbId : NumberLong("83..."}, "shard02")
    {
      "cause" : {
        "chunkTooBig": true,
        ...
        "errMsg" : "chunk too big to move"
      }
    }
    ```
    
    - 청크를 옮기기 전에 `splitAt` 명령을 사용해 수동으로 분할해야 한다.
        
        ```bash
        > db.chunk.find({ns : "video.movies","min.imdbId":NumberLong("6386...")}).pretty()
        {
          ...
          "min" : {"imdbId" : NumberLong("6386...") },
          "max" : {"imdbId" : NumberLong("8345...") },
          ...
        }
        
        > sh.splitAt("video.movies", {"imdbIid": NumberLong("70000000000")})
        
        > db.chunk.find(ns: "video.movies", "min.imdbId": NumberLong("6386...")}).pretty()
        {
          ...
          "min" : {"imdbId" : NumberLong("6386...") },
          "max" : {"imdbId" : NumberLong("70000000000") },
          ...
        }
        ```
        
    - 청크가 작은 조각으로 분할되고 나면 옮길 수 있다. 최대 청크 크기를 늘린 후 옮길 수도 있지만, 큰 청크는 쪼갤 수 있을 때마다 쪼개야 한다.****

### 4-4. 점보 청크

---

- 샤드 키로 "date" 필드를 선택했다고 가정하자.
    - "date" 필드는 '년/월/일' 처럼 보이는 문자열이며, `mongos`는 청크를 하루에 한 개까지만 생성할 수 있다.
    - 갑자기 평소보다 1000배나 많은 트래픽 → 청크는 다른 날보다 훨씬 커지지만 모든 도큐먼트가 샤드 키로 동일한 값을 갖기 때문에 전혀 분할되지 않는다.
    - 청크가 `config.settings`에 설정된 최대 청크 크기보다 커지면 밸런서는 청크를 이동할 수 없다.
- 나눌 수도, 옮길 수도 없는 이 청크를 점보 청크라고 한다.

- 한 샤드가 다른 것들보다 훨씬 빨리 커지는 상황은 점보 청크 문제가 있음을 나타내는 지표
    - 점보 청크가 있는지 `sh.status()`로 확인할 수 있으며, 점보 청크는 `jumbo`라고 표시된다.
        
        ```bash
        > sh.status()
        ...
          {"x": -7} --> {"x": 5 } on : shard0001
          {"x": 5} --> {"x": 6 } on : shard0001 jumbo
          {"x": 6} --> {"x": 7 } on : shard0001 jumbo
          {"x": 7} --> {"x": 339 } on : shard0001
        ...
        ```
        
- `dataSize` 명령으로 청크 크기를 확인할 수 있다.
    - `db.chunks` 컬렉션으로 청크 범위를 확인
        
        ```bash
        > use config
        > var chunks = db.chunks.find({"ns": "acme.analytics"}).toArray()
        ```
        
    - 청크 범위를 사용해 점보 청크일 가능성이 있는 청크를 찾는다.
        
        ```bash
        > use <dbNmae>
        > db.runCommand({"dataSize": "<dbName.collName>",
        ... "keyPattern": {"date": 1}, // shard key
        ... "min": chunks[0].min,
        ... "max": chunks[0].max})
        {
          "size": 33567917,
          "numObjects": 108942,
          "millis": 634,
          "ok": 1,
          ...
        }
        ```
        
    
    ❗`dataSize` 명령이 청크 크기를 알아내려면 청크의 데이터를 스캔해야 한다는 점에 주의하자.
    
    → 가능하면 데이터에 대한 정보를 활용해 검색 범위를 좁히자.
    

- 점보 청크 분산하기
    - 점보 청크 때문에 균형이 깨진 클러스터를 바로잡으려면 샤드 간에 점보 청크를 고르게 분산해야 한다.
    - 이 방법은 복잡한 수작업이긴 하지만, 다운타임이 발생하지 않는다.(많은 데이터를 마이그레이션하므로 느려질 수는 있다)
        - 설명에서 점보 청크가 있는 샤드는 '원본' 샤드로 간주한다. 점보 청크가 마이그레이션할 샤드는 '목적지' 샤드라고 한다. 청크를 옮기려는 '원본' 샤드가 여러 개일 수 있다는 점을 알아두자.
        - 각 '원본' 샤드에 다음 단계를 반복한다.
        1. 밸런서를 종료한다. 이 과정이 진행되는 동안 밸런서의 '도움'은 필요하지 않다.
            
            ```bash
            > sh.setBalancerState(false)
            ```
            
        2. 최대 청크보다 큰 청크를 옮기는 것을 허용하지 않으므로 청크 크기를 일시적을 늘린다.
            
            ```bash
            > use config
            > db.settings.save({"_id": "chunksize", "value": 10000})
            ```
            
        3. `moveChunk` 명령으로 점보 청크를 '원본' 샤드에서 옮긴다.
        4. '목적지' 샤드들과 청크 개수가 비슷해질 때까지 '원본' 샤드에 남아있는 청크에 `splitChunk`를 실행한다.
        5. 청크 크기를 원래 값으로 설정한다.
            
            ```bash
            > db.settings.save({"_id": "chunksize", "value": 64})
            ```
            
        6. 밸런서를 켠다.
            
            ```bash
            > sh.setBalancerState(true)
            ```
            
    
    ❗밸런서가 다시 켜지면 다시 점보 청크를 이동할 수 없게 된다.****
    

- 점보 청크 방지하기
    - 점보 청크를 방지하려면 샤드 키를 더 고르게 분포된 것으로 바꾼다.

### 4-5. 구성 갱신

---

- 때로는 `mongos`가 구성 서버에서 가져온 구성을 올바르게 갱신하지 않는다.
- `flushRouterConfig` 명령으로 수동으로 모든 캐시를 정리한다.
    
    ```bash
     > db.adminCommand({"flushRouterConfig": 1} )
    ```
    
- `flushRouterConfig`가 작동하지 않는다면 모든 `mongos` 혹은 `mongod` 프로세스를 재시작해서 모든 캐시 데이터를 정리하자.

- 참고
    - [https://www.mongodb.com/docs/v6.0/reference/method/sh.status/](https://www.mongodb.com/docs/v6.0/reference/method/sh.status/)
    - [https://github.com/kdh92/kdh92.github.io/blob/main/몽고DB완벽가이드/13.관리.md](https://github.com/kdh92/kdh92.github.io/blob/main/%EB%AA%BD%EA%B3%A0DB%EC%99%84%EB%B2%BD%EA%B0%80%EC%9D%B4%EB%93%9C/13.%EA%B4%80%EB%A6%AC.md)