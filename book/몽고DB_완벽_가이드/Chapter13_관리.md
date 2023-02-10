# Chapter 13. 관리

## 1. 독립 실행형 모드에서 멤버 시작

---

- 많은 유지 보수 작업은 (쓰기와 관련되므로) 세컨더리에서 수행될 수 없으며, 애플리케이션 성능에 영향을 미치기 때문에 프라이머리에서 수행되면 안 된다.
- 그러므로 다음 절에서는 독립 실행형 모드 서버 시작을 자주 언급한다.
    - 멤버가 복제 셋의 멤버가 아닌 독립 실행형 서버로 재시작함을 의미한다.

- 독립 실행형 모드에서 멤버를 시작하려면 먼저 명령행 인수를 확인해야 한다.
    
    ```bash
    > db.serverCmdLineOpts()
    {
      "argv" : [ "mongod","-f","/var/lib/mongod.conf"],
      "parsed":{
            "replSet":"mySet",
            "port":"27017",
            "dbpath":"/var/lib/db"
            },
      "ok" : 1
    }
    ```
    
    - 복제 셋의 secondary 에서 해보면..
        
        ```bash
        mdbDefGuide [direct: secondary] test> db.serverCmdLineOpts()
        {
          argv: [ 'mongod', '--replSet', 'mdbDefGuide', '--bind_ip_all' ],
          parsed: { net: { bindIp: '*' }, replication: { replSet: 'mdbDefGuide' } },
          ok: 1,
          '$clusterTime': {
            clusterTime: Timestamp({ t: 1671350315, i: 1 }),
            signature: {
              hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
              keyId: Long("0")
            }
          },
          operationTime: Timestamp({ t: 1671350315, i: 1 })
        }
        ```
        
    - 이 서버에서 유지 보수를 수행하려면 `replSet` 옵션 없이 서버를 재시작하면 된다.
        - 일반적인 독립 실행형 `mongod`처럼 읽기와 쓰기가 가능하게 해준다.
        - 복제 셋에 있는 다른 서버에서 이 서버와 통신하기를 원치 않으므로, (다른 멤버들이 서버를 발견하지 못하도록) 서버가 다른 포트로 수신하게 한다.
        - 서버 데이터를 조작하려고 이런 방식으로 시작하므로 dbpath는 그대로 유지한다.

1. 먼저 mongo 셸에서 서버를 종료한다.
    
    ```bash
    mdbDefGuide [direct: secondary] test> db.shutdownServer()
    MongoNetworkError: connection 2 to 127.0.0.1:27017 closed
    test> %
     ✘ kim-yoonhee  ~ 
    ```
    
2. 운영체제 셸에서 `replSet` 매개변수 없이 다른 포트에서 `mongod`를 시작한다.
    
    ```bash
     ✘ kim-yoonhee  ~  mongosh --port 30000 --dbpath /var/lib/db
    ```
    

→ 독립 실행형 서버로 구동되며 연결을 위한 포트는 30000번이다.

- 복제 셋의 다른 멤버는 27017번 포트로 이 서버에 접속을 시도하고, 곧 서버가 다운됐다고 추정하게 된다.

- 서버에서 유지 보수 수행을 마치면, 서버를 종료하고 원래 옵션으로 재시작할 수 있다.****
    - 서버는 유지 보수 모드 기간 동안 놓친 연산들을 복제해 복제 셋의 다른 멤버와 자동으로 동기화한다.

## 2. 복제 셋 구성

---

- 복제 셋 구성은 항상 `local.system.replset` 컬렉션의 도큐먼트에 보관된다.
    - 이 도큐먼트는 복제 셋의 모든 멤버에서 같다.
    - 절대 `update`를 이용해 도큐먼트를 변경하지 말자. 항상 rs 보조자나 `replSetReconfig` 명령을 사용하자.

### 2-1. 복제 셋 생성하기

---

- 멤버로 만들 `mongod` 들을 시작하고 그 중 하나에 `rs.initiate()`를 사용해 구성 정보를 전달함으로써 복제셋을 생성한다.
    
    ```bash
    > var config = {
    	  "_id" : <setName>,
        "members" : [
          {"_id":0, "host":<host1>},
          {"_id":1, "host":<host2>},
          {"_id":2, "host":<host3>}
        ]
      }
    }
    > rs.initiate(config)
    ```
    
    - 항상 config 객체를 `rs.initiate()`에 전달해야 한다. 그렇지 않으면 몽고DB는 자동으로 단일 멤버 복제 셋을 위한 config를 생성한다.
        - 이 config는 사용자가 원하는 호스트명을 사용하지 않으며, 복제 셋을 올바르게 구성하지 않는다.

### 2-2. 복제 셋 멤버 교체하기

---

- 복제 셋에 새로운 멤버를 추가할 때는 데이터 디렉터리에 아무것도 존재하지 않거나(이 경우 초기 동기화를 수행한다) 다른 멤버 데이터의 복제본이 있어야 한다.

- 프라이머리에 연결하고 다음처럼 새로운 멤버를 추가하자.
    
    ```bash
    # rs.add({"host": "spock:27017", "priority": 0, "hidden": true})
    mdbDefGuide [direct: primary] test> rs.add("spock:27017")
    {
      ok: 1,
      '$clusterTime': {
        clusterTime: Timestamp({ t: 1671352378, i: 1 }),
        signature: {
          hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
          keyId: Long("0")
        }
      },
      operationTime: Timestamp({ t: 1671352378, i: 1 })
    }
    ```
    
    - `rs.status()` 결과
        
        ```bash
        mdbDefGuide [direct: primary] test> rs.status()
        {
          set: 'mdbDefGuide',
          date: ISODate("2022-12-18T08:33:50.098Z"),
          myState: 1,
          term: Long("13"),
          syncSourceHost: '',
          syncSourceId: -1,
          heartbeatIntervalMillis: Long("2000"),
          majorityVoteCount: 2,
          writeMajorityCount: 2,
          votingMembersCount: 3,
          writableVotingMembersCount: 3,
          optimes: {
            lastCommittedOpTime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
            lastCommittedWallTime: ISODate("2022-12-18T08:33:46.062Z"),
            readConcernMajorityOpTime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
            appliedOpTime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
            durableOpTime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
            lastAppliedWallTime: ISODate("2022-12-18T08:33:46.062Z"),
            lastDurableWallTime: ISODate("2022-12-18T08:33:46.062Z")
          },
          lastStableRecoveryTimestamp: Timestamp({ t: 1671352378, i: 1 }),
          electionCandidateMetrics: {
            lastElectionReason: 'electionTimeout',
            lastElectionDate: ISODate("2022-12-18T08:32:16.029Z"),
            electionTerm: Long("13"),
            lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
            lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1671351541, i: 2 }), t: Long("11") },
            numVotesNeeded: 2,
            priorityAtElection: 1,
            electionTimeoutMillis: Long("10000"),
            numCatchUpOps: Long("0"),
            newTermStartDate: ISODate("2022-12-18T08:32:16.049Z"),
            wMajorityWriteAvailabilityDate: ISODate("2022-12-18T08:32:16.612Z")
          },
          members: [
            {
              _id: 0,
              name: 'mongo1:27017',
              health: 1,
              state: 2,
              stateStr: 'SECONDARY',
              uptime: 104,
              optime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
              optimeDurable: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
              optimeDate: ISODate("2022-12-18T08:33:46.000Z"),
              optimeDurableDate: ISODate("2022-12-18T08:33:46.000Z"),
              lastAppliedWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              lastDurableWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              lastHeartbeat: ISODate("2022-12-18T08:33:48.937Z"),
              lastHeartbeatRecv: ISODate("2022-12-18T08:33:48.935Z"),
              pingMs: Long("0"),
              lastHeartbeatMessage: '',
              syncSourceHost: 'mongo3:27017',
              syncSourceId: 2,
              infoMessage: '',
              configVersion: 2,
              configTerm: 13
            },
            {
              _id: 1,
              name: 'mongo2:27017',
              health: 1,
              state: 2,
              stateStr: 'SECONDARY',
              uptime: 104,
              optime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
              optimeDurable: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
              optimeDate: ISODate("2022-12-18T08:33:46.000Z"),
              optimeDurableDate: ISODate("2022-12-18T08:33:46.000Z"),
              lastAppliedWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              lastDurableWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              lastHeartbeat: ISODate("2022-12-18T08:33:48.936Z"),
              lastHeartbeatRecv: ISODate("2022-12-18T08:33:48.936Z"),
              pingMs: Long("0"),
              lastHeartbeatMessage: '',
              syncSourceHost: 'mongo3:27017',
              syncSourceId: 2,
              infoMessage: '',
              configVersion: 2,
              configTerm: 13
            },
            {
              _id: 2,
              name: 'mongo3:27017',
              health: 1,
              state: 1,
              stateStr: 'PRIMARY',
              uptime: 122,
              optime: { ts: Timestamp({ t: 1671352426, i: 1 }), t: Long("13") },
              optimeDate: ISODate("2022-12-18T08:33:46.000Z"),
              lastAppliedWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              lastDurableWallTime: ISODate("2022-12-18T08:33:46.062Z"),
              syncSourceHost: '',
              syncSourceId: -1,
              infoMessage: '',
              electionTime: Timestamp({ t: 1671352336, i: 1 }),
              electionDate: ISODate("2022-12-18T08:32:16.000Z"),
              configVersion: 2,
              configTerm: 13,
              self: true,
              lastHeartbeatMessage: ''
            },
            {
              _id: 3,
              name: 'spock:27017',
              health: 0,
              state: 8,
              stateStr: '(not reachable/healthy)',
              uptime: 0,
              optime: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
              optimeDurable: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
              optimeDate: ISODate("1970-01-01T00:00:00.000Z"),
              optimeDurableDate: ISODate("1970-01-01T00:00:00.000Z"),
              lastAppliedWallTime: ISODate("1970-01-01T00:00:00.000Z"),
              lastDurableWallTime: ISODate("1970-01-01T00:00:00.000Z"),
              lastHeartbeat: ISODate("2022-12-18T08:33:49.913Z"),
              lastHeartbeatRecv: ISODate("1970-01-01T00:00:00.000Z"),
              pingMs: Long("0"),
              lastHeartbeatMessage: 'Error connecting to spock:27017 :: caused by :: Could not find address for spock:27017: SocketException: Host not found (authoritative)',
              syncSourceHost: '',
              syncSourceId: -1,
              infoMessage: '',
              configVersion: -1,
              configTerm: -1
            }
          ],
          ok: 1,
          '$clusterTime': {
            clusterTime: Timestamp({ t: 1671352426, i: 1 }),
            signature: {
              hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
              keyId: Long("0")
            }
          },
          operationTime: Timestamp({ t: 1671352426, i: 1 })
        }
        ```
        

- "`host`" 필드를 이용해서 멤버를 제거할 수도 있다.
    
    ```bash
    mdbDefGuide [direct: primary] test> rs.remove("spock:27017")
    {
      ok: 1,
      '$clusterTime': {
        clusterTime: Timestamp({ t: 1671352530, i: 1 }),
        signature: {
          hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
          keyId: Long("0")
        }
      },
      operationTime: Timestamp({ t: 1671352530, i: 1 })
    }
    ```
    

- 재구성을 통해 멤버의 설정을 바꿀 수 있다. 멤버의 설정을 바꾸는 데는 다음과 같은 제약 사항이 있다.
    - 멤버의 "`_id`"는 변경할 수 없다.
    - 재구성 정보를 전달하려는 멤버(일반적으로 프라이머리)의 우선순위를 0으로 할 수 없다.
    - 아비터에서 아비터가 아닌 것으로 변경할 수 없으며, 반대도 마찬가지다.
    - 멤버의 "`buildIndexes`"를 `false`에서 `true`로 변경할 수 없다.

- 멤버의 "`host`"필드는 변경할 수 있다.
    - 호스트 정보를 잘못 명시했다면 나중에 간단히 구성을 변경해 올바른 IP로 정정할 수 있다.
    
    ```bash
    # 호스트명을 변경
    > var config = rs.config()
    > config.members[0].host = "spock:27017"
    > rs.reconfig(config)
    ```
    
    - ex> 위의 spock:27017 → spocky:27017 로 변경해보기
        
        ```bash
        mdbDefGuide [direct: primary] test> var config = rs.config()
        mdbDefGuide [direct: primary] test> config.members[3].host = "spocky:27017"
        spocky:27017
        mdbDefGuide [direct: primary] test> rs.reconfig(config)
        {
          ok: 1,
          '$clusterTime': {
            clusterTime: Timestamp({ t: 1671352836, i: 2 }),
            signature: {
              hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
              keyId: Long("0")
            }
          },
          operationTime: Timestamp({ t: 1671352836, i: 2 })
        }
        mdbDefGuide [direct: primary] test>
        ```
        
        - `rs.config()`로 확인해보기
            
            ```bash
            mdbDefGuide [direct: primary] test> rs.config()
            {
              _id: 'mdbDefGuide',
              version: 5,
              term: 13,
              members: [
                {
                  _id: 0,
                  host: 'mongo1:27017',
                  arbiterOnly: false,
                  buildIndexes: true,
                  hidden: false,
                  priority: 1,
                  tags: {},
                  secondaryDelaySecs: Long("0"),
                  votes: 1
                },
                {
                  _id: 1,
                  host: 'mongo2:27017',
                  arbiterOnly: false,
                  buildIndexes: true,
                  hidden: false,
                  priority: 1,
                  tags: {},
                  secondaryDelaySecs: Long("0"),
                  votes: 1
                },
                {
                  _id: 2,
                  host: 'mongo3:27017',
                  arbiterOnly: false,
                  buildIndexes: true,
                  hidden: false,
                  priority: 1,
                  tags: {},
                  secondaryDelaySecs: Long("0"),
                  votes: 1
                },
                {
                  _id: 3,
                  host: 'spocky:27017',
                  arbiterOnly: false,
                  buildIndexes: true,
                  hidden: true,
                  priority: 0,
                  tags: {},
                  secondaryDelaySecs: Long("0"),
                  votes: 1
                }
              ],
              protocolVersion: Long("1"),
              writeConcernMajorityJournalDefault: true,
              settings: {
                chainingAllowed: true,
                heartbeatIntervalMillis: 2000,
                heartbeatTimeoutSecs: 10,
                electionTimeoutMillis: 10000,
                catchUpTimeoutMillis: -1,
                catchUpTakeoverDelayMillis: 30000,
                getLastErrorModes: {},
                getLastErrorDefaults: { w: 1, wtimeout: 0 },
                replicaSetId: ObjectId("639de752e0e9a389ba1a8bae")
              }
            }
            ```
            

<aside>
💡 `rs.config()`로 구성을 가져와서 원하는 부분을 수정한 후 `rs.reconfig()`에 새로운 구성 정보를 전달함으로써 복제 셋을 재구성한다.

</aside>

### 2-3. 큰 복제 셋 만들기

---

- 복제 셋에서 멤버는 50개, 투표 멤버는 7개로 제한된다.
    - 다른 멤버에 하트비트를 보내는 데 필요한 네트워크 트래픽량을 줄이고 선출에 걸리는 시간을 제한하기 위함

- 멤버가 7개 이상인 복제 셋을 생성한다면, 모든 추가적인 멤버는 투표권이 0개여야 한다.
    
    ```bash
    # 멤버 구성 정보에 다음처럼 명시한다.
    > rs.add({"_id": 7, "host": "server-7:27017", "votes": 0})
    ```
    
    - 해당 멤버들이 (거부권을 갖더라도) 선출 과정에서 투표권을 행사하는 것을 방지한다.****

### 2-4. 재구성 강제하기

---

- 복제 셋의 과반수를 영구적으로 잃으면, 프라이머리가 없는 상태에서 설정을 재구성할 필요가 있다.
    - 일반적으로 재구성을 프라이머리에 요청하기 때문이다.

- 셸에서 세컨더리에 연결하고 "`force`" 옵션을 이용해 재구성 명령을 전달한다.
    - `rs.reconfig(config, {"force": true})`
- 강제 재구성은 일반 재구성과 같은 규칙을 따르며, 사용자는 유효하고 잘 구성된 구성 정보를 올바른 옵션으로 보내야 한다.
    - "`force`" 옵션은 유효하지 않은 구성 정보를 허용하지 않고, 다만 세컨더리가 재구성 정보를 받아들이도록 허용한다.
- 강제 재구성은 복제 셋 "`version`" 번호를 크게 높인다.
- 세컨더리는 재구성 정보를 받으면 자신의 구성 정보를 갱신하고 새로운 구성 정보를 다른 멤버에 전달한다.
    - 모든 멤버가 새로운 호스트명을 가지면 복제 셋의 각 멤버를 다운시키고, 새로운 멤버를 독립 실행형 모드로 시작한 뒤, `local.system.replset` 도큐먼트를 수동으로 변경하고 멤버를 재시작한다.

## 3. 멤버 상태 조작

---

### 3-1. 프라이머리에서 세컨더리로 변경하기

---

- `stepDown` 함수를 이용하면 프라이머리를 세컨더리로 강등할 수 있다.
    
    ```bash
    mdbDefGuide [direct: primary] test> rs.stepDown()
    {
      ok: 1,
      '$clusterTime': {
        clusterTime: Timestamp({ t: 1671371994, i: 1 }),
        signature: {
          hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
          keyId: Long("0")
        }
      },
      operationTime: Timestamp({ t: 1671371994, i: 1 })
    }
    mdbDefGuide [direct: secondary] test>
    ```
    
    - 프라이머리를 60초 동안 `SECONDARY` 상태로 만든다.
    - 그 기간동안에는 다른 프라이머리가 선출되지 않으면 세컨더리 상태로 변했던 프라이머리는 재선출을 시도할 수 있다.****
    
    ```bash
    # 더 길거나 짧은 시간 동안 세컨더리 상태로 남겨두려면 몇 초간 SECONDARY 상태로 머물게 할지 지정
    > rs.stepDown(600) # 10분
    ```
    

### 3-2. 선출 방지하기

---

- 프라이머리상에서 유지 보수 작업을 수행해야 하고, 그 사이에 자격이 있는 다른 멤버가 프라이머리가 되지 않도록 하려면, 각각의 멤버에 `freeze` 명령을 실행함으로써 세컨더리 상태에 머물게 한다.
    
    ```bash
    > rs.freeze(10000) # 멤버는 10000초 동안 세컨더리 상태로 유지된다.
    ```
    
- 시간이 경과하기 전에 프라이머리에서 수행 중인 유지 보수를 완료해 다른 멤버의 `freeze` 상태를 해제하려면, 각 멤버에 타임아웃을 0초로 지정해 명령을 다시 실행한다.
    
    ```bash
    > rs.freeze(0) # freeze 상태를 해지할 수 있다
    ```
    

- `freeze` 상태가 아닌 멤버는 원한다면 선출을 실시할 수 있다.
- 강등했던 프라이머리도 `rs.freeze(0)`을 사용해 `freeze` 상태를 해제할 수 있다.
    
    ```bash
    mdbDefGuide [direct: secondary] test> rs.freeze(0)
    {
      info: 'unfreezing',
      ok: 1,
      '$clusterTime': {
        clusterTime: Timestamp({ t: 1671372614, i: 1 }),
        signature: {
          hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
          keyId: Long("0")
        }
      },
      operationTime: Timestamp({ t: 1671372614, i: 1 })
    }
    ```
    

## 4. 복제 모니터링

---

- 모든 멤버가 정상적으로 기동했는지, 멤버가 어떤 상태인지, 복제가 얼마나 최신 상태인지 모니터링한다.****
- 복제와 관련된 문제는 일시적일 때가 많다.
    - 서버가 다른 서버에 도달할 수 없었다가 다시 도달할 수 있게 될 때가 있다.
    
    → 로그를 확인하면 이 같은 문제를 쉽게 볼 수 있다.
    

### 4-1. 상태 정보 가져오기

---

- `replSetGetStatus`: 복제 셋의 모든 멤버의 정보를 얻는 데 가장 유용한 명령
    
    ```bash
    mdbDefGuide [direct: secondary] test> rs.status()
    {
      set: 'mdbDefGuide',
      date: ISODate("2022-12-18T14:15:00.928Z"),
      myState: 2,
      term: Long("26"),
      syncSourceHost: 'mongo1:27017',
      syncSourceId: 0,
      heartbeatIntervalMillis: Long("2000"),
      majorityVoteCount: 2,
      writeMajorityCount: 2,
      votingMembersCount: 3,
      writableVotingMembersCount: 3,
      optimes: {
        lastCommittedOpTime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
        lastCommittedWallTime: ISODate("2022-12-18T14:14:54.107Z"),
        readConcernMajorityOpTime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
        appliedOpTime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
        durableOpTime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
        lastAppliedWallTime: ISODate("2022-12-18T14:14:54.107Z"),
        lastDurableWallTime: ISODate("2022-12-18T14:14:54.107Z")
      },
      lastStableRecoveryTimestamp: Timestamp({ t: 1671372874, i: 1 }),
      electionParticipantMetrics: {
        votedForCandidate: true,
        electionTerm: Long("26"),
        lastVoteDate: ISODate("2022-12-18T14:00:03.863Z"),
        electionCandidateMemberId: 0,
        voteReason: '',
        lastAppliedOpTimeAtElection: { ts: Timestamp({ t: 1671371994, i: 1 }), t: Long("25") },
        maxAppliedOpTimeInSet: { ts: Timestamp({ t: 1671371994, i: 1 }), t: Long("25") },
        priorityAtElection: 1,
        newTermStartDate: ISODate("2022-12-18T14:00:03.910Z"),
        newTermAppliedDate: ISODate("2022-12-18T14:00:04.868Z")
      },
      members: [
        {
          _id: 0,
          name: 'mongo1:27017',
          health: 1,
          state: 1,
          stateStr: 'PRIMARY',
          uptime: 5772,
          optime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
          optimeDurable: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
          optimeDate: ISODate("2022-12-18T14:14:54.000Z"),
          optimeDurableDate: ISODate("2022-12-18T14:14:54.000Z"),
          lastAppliedWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          lastDurableWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          lastHeartbeat: ISODate("2022-12-18T14:14:59.202Z"),
          lastHeartbeatRecv: ISODate("2022-12-18T14:14:59.202Z"),
          pingMs: Long("0"),
          lastHeartbeatMessage: '',
          syncSourceHost: '',
          syncSourceId: -1,
          infoMessage: '',
          electionTime: Timestamp({ t: 1671372003, i: 1 }),
          electionDate: ISODate("2022-12-18T14:00:03.000Z"),
          configVersion: 6,
          configTerm: 26
        },
        {
          _id: 1,
          name: 'mongo2:27017',
          health: 1,
          state: 2,
          stateStr: 'SECONDARY',
          uptime: 5772,
          optime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
          optimeDurable: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
          optimeDate: ISODate("2022-12-18T14:14:54.000Z"),
          optimeDurableDate: ISODate("2022-12-18T14:14:54.000Z"),
          lastAppliedWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          lastDurableWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          lastHeartbeat: ISODate("2022-12-18T14:14:59.204Z"),
          lastHeartbeatRecv: ISODate("2022-12-18T14:14:59.202Z"),
          pingMs: Long("0"),
          lastHeartbeatMessage: '',
          syncSourceHost: 'mongo1:27017',
          syncSourceId: 0,
          infoMessage: '',
          configVersion: 6,
          configTerm: 26
        },
        {
          _id: 2,
          name: 'mongo3:27017',
          health: 1,
          state: 2,
          stateStr: 'SECONDARY',
          uptime: 20592,
          optime: { ts: Timestamp({ t: 1671372894, i: 1 }), t: Long("26") },
          optimeDate: ISODate("2022-12-18T14:14:54.000Z"),
          lastAppliedWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          lastDurableWallTime: ISODate("2022-12-18T14:14:54.107Z"),
          syncSourceHost: 'mongo1:27017',
          syncSourceId: 0,
          infoMessage: '',
          configVersion: 6,
          configTerm: 26,
          self: true,
          lastHeartbeatMessage: ''
        }
      ],
      ok: 1,
      '$clusterTime': {
        clusterTime: Timestamp({ t: 1671372894, i: 1 }),
        signature: {
          hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
          keyId: Long("0")
        }
      },
      operationTime: Timestamp({ t: 1671372894, i: 1 })
    }
    ```
    

- `rs.status()`의 가장 유용한 필드들
    - "`self`": `rs.status()`가 실행된 멤버에만 존재****
    - "`stateStr`": 서버의 상태를 나타내는 문자열
    - "`uptime`": 멤버에 도달할 수 있었던 시간(초) 혹은 이 서버가 "`self`" 멤버를 위해 시작된 이후부터의 시간(초)이다.
    - "`optimeDate`": 각 멤버의 oplog에서 (해당 멤버가 동기화된) 마지막 연산 수행 시각을 의미한다.
        - 하트비트에 의해 보고된 각 멤버의 상태이고, 보고된 연산 수행 시각은 몇 초 정도 차이가 날 수 있음
    - "`lastHearBeat`": 서버가 "`self`" 멤버로부터 마지막으로 하트비트를 받은 시간
        - 네트워크에 문제가 있거나 서버가 분주하면 이 시간은 2초 전보다 길어질 수 있다.
    - "`pingMs`": 이 서버에 대한 하트비트에 걸린 평균 실행 시간
        - 어느 멤버로부터 동기화할지 결정하는 데 사용된다.
    - "`errmsg`": 멤버가 하트비트 요청에 반환하기로 선택한 모든 상태 메세지
        - 정보 전달용이며 오류 메시지는 아니다.

### 4-2. 복제 그래프 시각화하기

---

- 세컨더리에서 `rs.status()`를 실행하면 "`syncingTo`"라는 최상위 필드를 확인할 수 있다.
    
    ```bash
    > server1.adminCommand({replSetGetStatus: 1})['syncingTo']
    server0:27017
    # server0은 server1의 복제 소스
    ```
    
    - `mongosh`의 `rs.status()`에는 없다. 대신 `syncSourceHost`, `syncSourceId`가 있다.
        
        ```bash
        # 1: primary
        mdbDefGuide [direct: primary] test> db.adminCommand({replSetGetStatus: 1})['syncSourceHost']
        
        # 2, 3: secondary
        mdbDefGuide [direct: secondary] test> db.adminCommand({replSetGetStatus: 1})['syncSourceHost']
        mongo1:27017
        ```
        

- 몽고DB는 핑 시간을 기준으로 동기화할 대상을 결정한다.
    - 멤버는 다른 멤버에 하트비트를 보낼 때, 요청이 처리되기까지 걸리는 시간을 잰다.
        - 몽고DB는 이러한 평균 실행 시간을 보관한다.
    - 동기화할 멤버를 선택할 때, 멤버는 가장 가깝고 복제에서 자기보다 앞서 있는 멤버를 찾는다.
        - 따라서 복제 순환이 발생하지 않는다. 멤버는 자신보다 앞서 있는 프라이머리나 세컨더리만 복제
        - 자동 복제 사슬에는 몇 가지 단점이 있다.
            - 복제 홉이 많을수록 모든 서버에 쓰기를 복제하는 데 시간이 좀 더 오래 걸린다.

- `replSetSyncFrom`(`rs.syncFrom()`): 복제 소스를 수정
    
    ```bash
    > secondary.adminCommand({"replSetSyncFrom": "server0:27017"})
    ```
    

### 4-3. 복제 루프

---

- 복제 루프: 모든 멤버가 다른 멤버로부터 복제를 수행하는 상태 → 모든 복제 루프 멤버는 프라이머리가 될 수 없음
    - ex> A는 B로부터 동기화하고, B는 C로부터, C는 다시 A로부터 동기화하는 상태

- 멤버가 동기화할 멤버를 자동으로 고를 때는 복제 루프가 발생하지 않는다.
    - 😵 그러나 `replSetSyncFrom` 명령을 이용해 복제 루프를 강제로 수행할 수 있다.

### 4-4. 복제 사슬 비활성화하기

---

- 복제 사슬은 세컨더리가 또 다른 세컨더리와 동기화할 때 발생한다.
    - "`chainingAllowed`" 설정을 `false`로 변경해(default: `true`) 모든 멤버가 프라이머리와 동기화하게 함으로써 복제 사슬을 사용하지 않도록 설정할 수 있다.
        
        ```bash
        mdbDefGuide [direct: secondary] test> var config = rs.config()
        
        # (존재하지 않는다면) 하위 객체 생성
        mdbDefGuide [direct: secondary] test> config.settings = config.settings||{}
        {
          chainingAllowed: true,
          heartbeatIntervalMillis: 2000,
          heartbeatTimeoutSecs: 10,
          electionTimeoutMillis: 10000,
          catchUpTimeoutMillis: -1,
          catchUpTakeoverDelayMillis: 30000,
          getLastErrorModes: {},
          getLastErrorDefaults: { w: 1, wtimeout: 0 },
          replicaSetId: ObjectId("639de752e0e9a389ba1a8bae")
        }
        
        # 모든 멤버는 프라이머리와 동기화한다.
        mdbDefGuide [direct: secondary] test> config.settings.chainingAllowed = false
        false
        
        mdbDefGuide [direct: primary] test> rs.reconfig(config)
        {
          ok: 1,
          '$clusterTime': {
            clusterTime: Timestamp({ t: 1671375264, i: 2 }),
            signature: {
              hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
              keyId: Long("0")
            }
          },
          operationTime: Timestamp({ t: 1671375264, i: 2 })
        }
        ```
        
        - 프라이머리가 이용 불가능한 상태가 되면 세컨더리와 동기화하게 된다.

### 4-5. 지연 계산하기

---

- 지연: 세컨더리가 얼마나 뒤쳐져 있는지 나타내는데, 프라이머리가 마지막으로 수행한 연산과 세컨더리가 마지막으로 적용한 연산의 타임스탬프의 차이를 의미한다.

- `rs.printReplicationInfo()`: 연산의 크기와 날짜 범위를 포함하는 프라이머리 oplog의 요약 정보를 제공한다.
    
    ```bash
    mdbDefGuide [direct: primary] test> rs.printReplicationInfo()
    actual oplog size
    '17270.468017578125 MB'
    ---
    configured oplog size
    '17270.468017578125 MB'
    ---
    log length start to end
    '82830 secs (23.01 hrs)'
    ---
    oplog first event time
    'Sat Dec 17 2022 15:59:14 GMT+0000 (Coordinated Universal Time)'
    ---
    oplog last event time
    'Sun Dec 18 2022 14:59:44 GMT+0000 (Coordinated Universal Time)'
    ---
    now
    'Sun Dec 18 2022 14:59:48 GMT+0000 (Coordinated Universal Time)'
    ```
    
    - 위에서 oplog는 약 17270메가바이트이며 약 23시간의 연산까지만 수행할 수 있다.

- `rs.printSlaveReplicationInfo()`: 각 멤버의 `syncedTo`값과 마지막 oplog 항목이 각 세컨더리에 기록된 시간을 가져올 수 있다.
    - 몽고DB 6.0 기준으로는 `rs.printSecondaryReplicationInfo()` 쓰라고 함
        
        ```bash
        mdbDefGuide [direct: primary] test> rs.printSlaveReplicationInfo()
        MongoshDeprecatedError: [COMMON-10003] printSlaveReplicationInfo has been deprecated. Use printSecondaryReplicationInfo instead
        
        mdbDefGuide [direct: primary] test> rs.printSecondaryReplicationInfo()
        source: mongo2:27017
        {
          syncedTo: 'Sun Dec 18 2022 16:00:15 GMT+0000 (Coordinated Universal Time)',
          replLag: '0 secs (0 hrs) behind the primary '
        }
        ---
        source: mongo3:27017
        {
          syncedTo: 'Sun Dec 18 2022 16:00:15 GMT+0000 (Coordinated Universal Time)',
          replLag: '0 secs (0 hrs) behind the primary '
        }
        ```
        

- 복제 셋 멤버의 지연은 절대적인 시간이 아니라 프라이머리를 기준으로 상대적으로 계산된다.****

### 4-6. Oplog 크기 변경하기

---

- 프라이머리의 oplog는 유지 보수 시간으로 여겨진다.
    - 프라이머리의 oplog 길이가 한 시간 정도라면, 잘못된 부분을 고칠 수 있는 시간이 한 시간 정도라는 의미
    
    → 일반적으로 며칠에서 1주 정도 데이터를 보유할 수 있는 oplog가 바람직하다.
    
- 와이어드타이거 스토리지 엔젠을 사용하면 서버가 실행되는 동안 oplog 크기를 조정할 수 있다.

- oplog의 크기를 늘리려면 다음 단계를 수행
    1. 복제 셋 멤버에 연결한다. 인증이 활성화됐다면 local 데이터베이스를 수정할 권한이 있는 사용자를 사용해야 한다.
    2. oplog의 현재 크기를 확인한다.
        
        ```bash
        mdbDefGuide [direct: primary] test> use local
        switched to db local
        
        # 컬렉션 크기가 메가바이트 단위로 표시된다.
        mdbDefGuide [direct: primary] local> db.oplog.rs.stats(1024*1024).maxSize
        17270
        ```
        
    3. 복제 셋 멤버의 oplog크기를 변경한다.
        
        ```bash
        # 복제 셋 멤버의 oplog 크기를 16기가바이트로 변경한다.
        mdbDefGuide [direct: primary] local> db.adminCommand({replSetResizeOplog : 1, size: 160000})
        {
          ok: 1,
          '$clusterTime': {
            clusterTime: Timestamp({ t: 1671379825, i: 1 }),
            signature: {
              hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
              keyId: Long("0")
            }
          },
          operationTime: Timestamp({ t: 1671379825, i: 1 })
        }
        ```
        
    4. 마지막으로, oplog의 크기를 줄였다면 `compact`를 실행해 할당된 디스크 공간을 회수해야 할 수도 있다. 하지만 대상 멤버가 프라이머리인 동안에는 실행해서는 안 된다.****

- 일반적으로 oplog의 크기는 줄이면 안 된다.
    - oplog 크기가 수개월이더라도 일반적으로 디스크 공간은 충분하며 램이나 CPU 같은 귀중한 리소스를 소모하지는 않는다.

### 4-7. 인덱스 구축하기

---

- 프라이머리에 인덱스 구축을 전송하면 프라이머리는 정상적으로 인덱스를 구축하며, 세컨더리는 '`build index`' 연산을 복제할 때 인덱스를 구축한다.
    - 인덱스 구축은 멤버를 이용 불가능한 상태로 만들 수도 있는 리소스 집약적인 연산이다.
- 만약 모든 세컨더리가 동시에 인덱스를 구축하기 시작한다면 복제 셋의 거의 모든 멤버는 인덱스 구축이 완료될 때까지 오프라인 상태가 된다.
    - 이 과정은 복제 셋에만 해당된다.(샤드 클러스터에서 인덱스를 구축하는 방법은 다름)

<aside>
💡 "`unique`" 인덱스를 만들 때는 컬렉션에 대한 모든 쓰기를 중지해야 한다. 중지하지 않으면 복제 셋 멤버끼리 데이터가 일치하지 않게 될 수 있다.

</aside>

- 애플리케이션에 대한 영향을 최소화하려면 인덱스는 한 번에 한 멤버씩 구축하는 것이 바람직하다.
    1. 세컨더리를 종료한다.
    2. 종료한 세컨더리를 독립 실행형 서버로 재시작한다.
    3. 재시작한 서버에 인덱스를 구축한다.
    4. 인덱스 구축이 완료되면 서버를 복제 셋 멤버로 재시작한다.
        - 멤버를 재시작할 때 명령행 옵션이나 구성 파일에 `disableLogicalSessionCacheRefresh` 매개변수가 있으면 제거해야 한다.
    5. 복제 셋의 각 세컨더리에 1단계부터 4단계까지 반복한다.

➕

- 다음 두 가지 선택지 중, 실제 시스템에 가장 적게 영향을 미치는 방법을 선택해야 한다.
    1. 프라이머리에 인덱스를 구축한다. 트래픽이 적을 때 'off' 시간을 가질 수 있다면 이때 구축하면 좋다. 또한 읽기 선호도를 수정함으로써 구축이 진행 중일 때 세컨더리로 더 많은 부하를 보내도록 일시적으로 경로를 변경할 수 있다.
    프라이머리는 인덱스 구축을 세컨더리에 복제하지만, 세컨더리는 이미 인덱스가 있으므로 어떤 연산 작업도 발생하지 않는다.
    2. 프라이머리를 강등한 후 앞에서 설명한 2~4단계를 거친다. 이 방식에는 장애 조치가 필요하지만 기존 프라이머리가 인덱스를 구축하는 동안 정상적으로 작동하는 프라이머리가 있다. 인덱스 구축이 완료된 후 기존 프라이머리를 복제 셋에 재도입할 수 있다.

- 인덱스가 다른 멤버는 절대 프라이머리가 될 수 없으며 우선순위가 항상 0임을 기억하자.****

- 고유 인덱스를 구축할 때는 프라이머리가 중복 삽입을 하지 않아야 하며, 인덱스를 프라이머리에 먼저 구축해야 한다.
    - 그렇지 않으면 프라이머리는 중복 삽입을 하게 되고 세컨더리에 복제 오류를 발생시킨다.

### 4-8. 한정된 예산에서 복제하기

---

- 고성능 서버를 두 개 이상 구하기 어렵다면, 적은 램과 CPU, 속도가 느린 디스크 입출력을 갖는 재해 복구용 세컨더리 서버를 고려해보자.
    - 좋은 서버는 항상 프라이머리로 쓰고, 상대적으로 값싼 서버는 절대 클라이언트 트래픽을 처리하지 않게 한다.(모든 읽기 요청을 프라이머리에 보내도록 클라이언트를 구성한다)

- 값싼 서버를 설정하기 위한 옵션
    - "`priority`": 0
        - 해당 서버가 절대 프라이머리가 되지 않도록 설정한다.
    - "`hidden`": true
        - 클라이언트가 해당 세컨더리에 읽기 요청을 절대 보내지 않도록 설정한다.
    - "`buildIndexes`": false
        - 이는 선택적이지만, 해당 서버가 처리해야 하는 부하를 상당히 줄일 수 있다. 이 서버로부터 복원할 경우 인덱스를 재구축해야 한다.
    - "`votes`": 0
        - 서버가 두 개뿐이라면 세컨더리의 "`votes`"를 0으로 설정해서 해당 서버가 다운되더라도 프라이머리가 프라이머리로 유지되도록 한다.
        - 세 번째 서버가 있다면(애플리케이션 서버라도) 해당 서버의 "`votes`"를 0으로 설정하는 대신 아비터를 실행한다.

- 참고
    - [https://jackjeong.tistory.com/m/185](https://jackjeong.tistory.com/m/185)
    - [https://github.com/kdh92/kdh92.github.io/blob/main/몽고DB완벽가이드/13.관리.md](https://github.com/kdh92/kdh92.github.io/blob/main/%EB%AA%BD%EA%B3%A0DB%EC%99%84%EB%B2%BD%EA%B0%80%EC%9D%B4%EB%93%9C/13.%EA%B4%80%EB%A6%AC.md)