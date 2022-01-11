# Spring Data Redis + Spring Boot

## 1. MacOS 로컬에 Redis 설치

```bash
jenny@gim-yunhuiui-MacBookPro % brew install redis
```

**설치된 경로**

- redis 실행파일: `/usr/local/bin` 하위에 있다. redis-server,  redis-cli ...
    - redis-benchmark
    - redis-check-aof
    - redis-check-rdb
    - redis-cli
    - redis-sentinel
    - redis-server
- redis 설정파일: `/usr/local/etc/redis.conf`
    - redis 의 기본 포트는 6379
    
    ```bash
    # Set the number of databases. The default database is DB 0, you can select
    # a different one on a per-connection basis using SELECT <dbid> where
    # dbid is a number between 0 and 'databases'-1
    databases 16
    
    # Accept connections on the specified port, default is 6379 (IANA #815344).
    # If port 0 is specified Redis will not listen on a TCP socket.
    port 6379
    ```
    

### Redis 사용해보기

---

- **redis 서버 실행:** `redis-server`
- **redis 클라이언트로 접속:** `redis-cli`
- **redis 접속 후 키 입력/삭제:** `set key value`, `get key`
    
    ```bash
    127.0.0.1:6379> set jenny value
    OK
    
    127.0.0.1:6379> get jenny
    "value"
    ```
    
- **모든키 조회:** `keys *`
    - Redis 전체가 중지되므로, 절대 사용하면 안된다.
- **데이터베이스:** redis는 서버 인스턴스당 16개 데이터베이스를 가질 수 있음
    - **데이터베이스 전환:** `select` 명령어를 통해서 데이터베이스 전환
    
    ```bash
    127.0.0.1:6379> keys *
    1) "AUTH_"
    2) "k"
    3) "jenny"
    4) "AUTH_:fs_admin@smartfoodnet.com"
    
    127.0.0.1:6379> select 1
    OK
    127.0.0.1:6379[1]> keys *
    (empty array)
    ```
    

## 2. Redis-cli

```bash
jenny@gim-yunhuiui-MacBookPro % redis-cli
127.0.0.1:6379> keys *
(empty array)

127.0.0.1:6379> keys *   # 로컬 예제 보여주기 용! 절대 사용하면 안된다.
1) "AUTH_"
2) "AUTH_:fs_admin@smartfoodnet.com"

127.0.0.1:6379> smembers AUTH_
1) "fs_admin@smartfoodnet.com"

127.0.0.1:6379> scard AUTH_
(integer) 1

127.0.0.1:6379> hget AUTH_:fs_admin@smartfoodnet.com email
"fs_admin@smartfoodnet.com"
127.0.0.1:6379> hget AUTH_:fs_admin@smartfoodnet.com accessTokenExp
"1628302357"
127.0.0.1:6379> hget AUTH_:fs_admin@smartfoodnet.com accessToken
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NTUsImVtYWlsIjoiZnNfYWRtaW5Ac21hcnRmb29kbmV0LmNvbSIsIm5hbWUiOiJGcmVzaCBTcGFjZSBBZG1pbiIsInVzZXJUeXBlIjoiYWRtaW4iLCJpYXQiOjE2MjgyMTU5NTcsImV4cCI6MTYyODMwMjM1N30.r5mQgacWSyHUBU3An4Iwd7pMWw8YCoqDtN0MkCkg9ug"
```

### Medis

---

![./image/Spring_Data_Redis+Spring_Boot/Untitled.png](./image/Spring_Data_Redis+Spring_Boot/Untitled.png)

[https://dolsup.work/posts/how-to-build-medis-on-macos/](https://dolsup.work/posts/how-to-build-medis-on-macos/)

- 올해 초(21년 1월)엔 잘 설치가 되었었는데 이제는(21년 8월) 설치가 안 된다.. 그냥 `redis-cli` 를 쓰자!

### 여러가지 고급 설정들

---

- 레디스 설정
    
    Redis의 환경 설정을 할 수 있는 방법은 설정파일(redis.conf)을 변경하거나, 실시간 명령을 이용하여 환경 설정을 변경 할 수 있다.
    
    설정 파일을 변경 하는 방법은 redis 설치 디렉터리 안의 redis.conf 파일을 수정 후 재 기동 하는 방법이고,
    
    실시간으로 변경을 하려면 config set 명령을 이용하면 된다. (Redis가 운영되는 중에만 적용이 되기 때문에 영구 적용을 하기 위해서는 redis.conf 파일을 변경 해 주어야 한다.)
    
    버전에 따라서 config rewrite 명령을 지원하여 현재 운영중인 레디스 설정 상황을 redis.conf 파일에 저장을 한다.
    
    현재 운영중인 설정은 **config get *** 명령을 이용하면 된다.
    
    redis.conf 파일은 크게 다음과 같은 영역으로 나눠져 있다.
    
    - **1. 기본 설정**
        
        레디스의 가장 기본적인 설정들인 네트워크, 로그 및 프로세스 관련 설정을 할 수 있다.
        
        - **port** : 인스턴스가 사용할 서비스 포트. 기본 값은 6379
        - **bind** : 인스턴스가 사용할 네트워크 설정. 사용할 아이피 설정. bind 192.168.10.100
        - **timeout** : 연결된 클라이언트의 idle 대기 시간 설정을 **초** 단위로 한다. 해당 시간동안 송 수신이 발생하지 않으면 클라이언트의 연결을 끊는다. 0으로 설정하면 사용하지 않음.
        - **loglevel** : 인스턴스 동작 중에 출력하는 로그의 레벨을 지정 함. (debug/verbose,notice,warning 중에 선택 할 수 있음)
        - **logfile** : 로그가 저장되는 경로와 파일명을 지정 함.  logfile ./log/redis-master.log
        - **database** : 레디스에서 제공하는 논리적으로 분리된 데이터 저장공간이다. RDBMS의 스키마 개념이라고 보면 된다. 각각의 데이터 베이스는 숫자로 구분이 된다.
            
            이 값을 저장하지 않으면, 기본값으로 16이 사용되며, 16개로 분리된 데이터 저장 공간을 사용한다.
            
            레디스에 연결된 클라이언트는 기본적으로 database 0에 접속하게 된다.
            
            0번 이외의 데이터베이스에 접속 시 명령창에 [DBindex] 가 출력되어 접속 중인 데이터 베이스를 확인 할 수 있다.
            
            ```kotlin
            127.0.0.1:6379[15]> config get databases
            	1) "databases"
            	2) "16"
            
            127.0.0.1:6379>      <--  0번 디비에 접속 중
            127.0.0.1:6379[1]>   <--  1번 디비에 접속 중
            127.0.0.1:6379[2]>   <--  2번 디비에 접속 중
            127.0.0.1:6379[15]>  <-- 15번 디비에 접속 중
            ```
            
    - **2. 스냅 샷 설정**
        
        레디스는 메모리의 데이터를 영구 저장하기 위하여 dump.rdb 파일에 메모리의 내용을 기록한다.
        
        스냅샷 이벤트가 발생 하면 메모리의 내용을 dump.rdb에 저장한다.
        
        스냅샷 이벤트에 의해서 dump.rdb 파일에 저장 중에 오류(물리적 디스크 오류, 여유공간 부족등) 가 발생하면 레디스는 모든 쓰기 요청을 거부 한다.
        
        set, lpush, hset과 같은 모든 쓰기 요청에 대하여 실패 응답을 반환한다.
        
        - **save** : 스냅샷 이벤트가 발생 하는 주기를 설정 한다. svae 단위시간(초) 키변경회수 로 저장한다.  **save 300 10** (300초 또는 키 변경이 10개 발생하면 스냅샷 이벤트 발생)
            
            save "" 이런 식으로 빈 문자열을 설정하면 스냅샷 이벤트를 사용하지 않는다. save 300 10   save 900 1 이런 식으로 여러개 설정 시에는 그 중 하나라도 만족하면 스냅샷 이벤트가 동작 한다.
        
        스냅샷 이벤트에 의해서 dump.rdb에 파일을
        
        - **stop-writes-on-bgsave-error** : 스냅샷 요청에 의해서 dump.rdb에 내용을 저장하는 중에 쓰기 오류가 발생 할 경우에도, 쓰기 요청을 에러 없이 처리 하고 싶다면 이 파라미터를 NO로 설정하면 된다.
            
            No로 설정 할 경우 쓰기 오류가 발생 하는것을 감지 할 수 없으므로 yes로 사용하는 것이 좋다.
            
        - **rdbcompression** : 스냅샷 파일을 저장할때 파일의 압축 여부를 설정. 압축을 하게 되면 dump.rdb 파일의 크기는 주어 들지만, 작업시에 CPU 사용률은 높아 진다.
        - **rdbchecksum** : 스냅 샷 파일을 저장할때 파일의 정합성을 검증하기 위한 체크섬의 사용 여부를 설정한다. 이 값을 yes로 할 경우 스냅샷 파일을 만들거나, 읽을 때 10% 정도의 성능 하락이 있다.
        - **dir** : 레디스의 작업 디렉토리를 지정 한다. 레디스가 생성하는 파일의 기본 위치 사용 됨. dump.rdb, aof 파일, 로그파일 등이 저장된다.
        - **dbfilename** : 스냅샷 파일의 파일명을 지정하는 설정. 기본은 dump.rdb로 되어 있으나, <서비스포트 또는 노드이름>.rdb 같은 형식으로 사용을 하고자 할 경우 지정해 주면 된다.
    - **3. AOF (Append Only File)설정**
        
        AOF파일은 레디스에서 발생 한 모든 데이터의 변경 이력을 저장한다.
        
        쓰기 명령어가 실행 될때 마다 AOF 파일에 명령과 데이터를 기록하여 장애에서 데이터를 복원하는데 사용한다.
        
        사용자가 잘못해서 flushall 명령을 수행했을 경우, 레디스를 종료 후에 AOF파일을 열어서 flushall 명령을 삭제하고 레디스를 재 기동 시키기만 하면 된다.
        
        - **appendonly** : AOF 기능의 사용 여부를 설정 한다. 기본값은 no 이며, 필요한 경우 yes로 설정 한다.
        - **appendfilename** : AOF파일명을 지정 함. 필요 시 경로까지 설정해 줄 수 있다. 해당 위치에 파일이 없거나 생성 실패 시에 AOF 파일이 없다는 오류 메세지를 출력하고 인스턴스 시작이 실패 한다.
        - **appendfsync** : 데이터를 dump.rdb 또는 AOF 파일에 쓸 때, fsync() 함수를 이용해서 버퍼의 내용을 디스크에 즉시 기록하도록 할지 여부를 결정 한다.
            
            쓰기가 필요할 때마다 fsync()함수가 호출되면 데이터를 유실 할 위험은 줄어 들지만, 상대적으로 파일 기록 성능이 저하 될 수 있다.
            
        - **no** : fsync() 함수를 호출 하지 않는다. OS로 쓰기 명령을 전달하고, O/S에서 알아서 파일에 저장하도록 한다. 가장 빠른 성능을 제공하지만 , O/S에 쓰기 명령을 전달하고 다 쓰기 전에 서버에
            
            문제가 생기면 쓰여지지 않은 부분에 대한 손실이 발생 할 수 있다.
            
        - **always** : 각 명령어를 AOF 파일에 기록하고 나서 항상 fsync()함수를 호출한다. 가장 느린 성능을 제공하지만 데이터 안정성 측면에서는 가장 좋다.
        - **everysec** : 매 초마다 fsync() 함수를 호출한다. 성능과 안정성의 타협점으로 불수 있으며, **레디스의 기본 설정이다**.
        - **no-appendfsync-on-rewrite** : appendfsync 값이 always나 everysec으로 되어 있을 경우, 대량의 AOF 또는 스냅샷을 쓸 때, 디스크의 성능에 따라서 몇 초에서 몇 분이 걸려서 성능 이슈가 발생 할 수 있다.
            
            이럴 경우 fsync() 함수 호출을 하지 않게 한다.  **NO**로 설정하면 기존대로 fsync()함수를 호출하고, **YES**로 설정해야 fsync()를 사용하지 않는다.
            
    - **4. 복제 설정**
        
        마스터 노드에서는 requirepass부분만 설정 하면되고, 아래의 부분은 slave에서 설정 한다.
        
        - **slaveof** : 마스터 노드의 네트워크 위치를  지정한다. **slaveof <마스터노드의 IP> <마스터노드의 포트>**
        - **masterauth** : 마스터 노드가 **requirepass** 설정에 의해서 패스워드로 보호되고 있을 때, 슬레이브 노드에서 설정 하며, 마스터 노드의 requirepass 설정에 지정된 패스워드를 설정 한다.
        - **slave-serve-stale-data** : 슬레이브 노드에서 마스터 노드에 대한 네트워크 연결이 끊어지거나, 마스터 노드의 전체 데이터를 복제 중일 때 슬레이브 노드로 들어오는 요청을 어떻게 처리 할지 결정
        - **yes** : 슬레이브 노드에서 읽거나 쓰기 요청을 모두 처리 함.
        - **no** : 에러 메세지를 전달 함.
        - **slave-read-only** : 슬레이브를 read-only로 운영 함. 쓰기 명령만 에러가 발생하고, slaveof나 config 같은 관리자 명령은 여전히 사용할 수 있다.
        - **repl-ping-slave-period** : 슬레이브 노드에서 설정 된 간격으로 마스터 노드에 ping 명령을 수행한다. 기본은 10초 간격이다.
    - **5. 보안 설정**
        
        Redis가 초기부터 기업용 솔루션으로 제작된 것이 아니기 때문에, 보안 설정이 그리 많지는 않다.
        
        클라이언트 인증을 위한 패스워드 설정과 악의적인 사용자를 막기 위한 명령어 변경이 있다.
        
        - **requirepass** : 레디스 서버에 접속하기 위한 패스워드를 설정 한다. 클라이언트는 requirement에 설정 한 값을 이용하여 인증 한 후에 레디스 명령을 사용 할 수 있다. 슬레이브가 복제 요청을 할때도 사용 됨.
        - **rename-command** : 레디스 명령을 다른 명령으로 변경 한다.
            
            rename-command incr incr_temp 로 하면 incr 명령은 더 이상 사용할 수 없고, incr_temp만이 사용된다.
            
            rename-command incr ""  이런식으로 명령어 다음에 빈 문자열로 지정하게 되면 해당 명령어 자체가 사용되지 않는다.
            
            master에서 rename-commnd에 의해서 변경 한 명령은 slave에서 제대로 반영이 되지 않아서 데이터 복제가 실패 할 수 있다. 그러므로 Master에서 명령어를 변경하게 되면, Slave도 반영을 해 주어야 한다.
            
    - **6. 제한 설정**
        
        레디스 서버가 사용할 수 있는 최대값들 설정. 주로 메모리에 관한 설정들이며 복제나 스냅샷 사용 시 숙지 해야 한다.
        
        - **maxclient** : 레디스 인스턴스에 접속 할 수 있는 클라이언트 수를 지정 한다. , -1은 최대 설정 값 (4294967295) 이 설정 되며, 설정 하지 않으면 기본 값이 1024로 된다.
            
            O/S의 ulimit 값과도 연관이 있기 때문에 ulimit 제한도 같이 확인해서 변경해 주어야 한다. (/etc/limits.conf , /etc/security/limits.conf )
            
        - **maxmemory** : 레디스 인스턴스가 데이터를 저장하기 위하여 사용할 메모리 크기를 지정한다. 이 값보다 많은 데이터를 저장하면 maxmemory-policy 설정에 지정된 값에 따라서 레디스의 동작이 달라 진다.
        - **maxmemory-policy** : 레디스에서 저장한 데이터가 maxmemory를 넘을 경우, 메모리 정리를 어떻게 할지에 대한 정책을 지정해 준다.
        - **volatile-lru** : (기본값) 만기시각이 설정된 key 들 중에서 LRU algorithm 에 의해 key 를 골라 삭제
        - **allkeys-lru** : LRU algorithm 에 의해 key 를 골라 삭제
        - **volatile-random** : 만기시각이 설정된 key 들 중에서 랜덤하게 key 를 골라 삭제
        - **allkeys-random** : 랜덤하게 key 를 골라 삭제
        - **volatile-ttl** : 만기시각이 설정된 key 들 중에서 만기시각이 가장 가까운 key 를 골라 삭제
        - **noeviction** : 어떤 key 도 삭제하지 않고 error on write operations 를 돌려준다.
        - **maxmemory-samples** : maxmemory-policy를 적용하기 위해서 레디스가 조회 할 키의 개수를 지정 함.
            
            전체 키를 읽어서 삭제 policy에 해당하는 키를 찾는게 아니라, 지정한 값 만큼의 임의의 키를 읽어서 그 중에서 삭제 대상인 키가 있는지 확인 한다. 이때 임의로 읽어 들일 키의 개수를 지정 함.
            
    - **7. 슬로우 쿼리 설정**
        
        레디스의 데이터 처리는 모두 레디스 명령어에 기반을 한다. 즉, 레디스에서의 쿼리는 데이터 처리 명령어와 같다.
        
        슬로우 쿼리는 느리게 수행되는 명령어를 뜻한다.
        
        쿼리 수행 시간이란 레디스 명령어가 인스턴스까지 도착 한 후 부터 명령을 처리한 결과를 돌려 주기까지의 시간을 말한다. 순수하게 데이터를 처리한 시간만을 말함.
        
        보통 SORT를 하는 zunionstore, zinterstore, zrangebyscore 같은 명령어들이 주로 슬로우 쿼리에 나타난다.
        
        슬로우 쿼리는 해당 명령어의 사용을 다른 명령어로 바꾸거나, 사용 빈도를 줄이도록 권고 한다.
        
        - **slow-log-slower-than** : 로깅을 할 슬로우 쿼리 수행 시간을 지정한다. 마이크로 초(1/1000000) , 0은 사용하지 않음. 기본은 10000
            
            지정한 시간보다 오래 걸리는 명령들은 slowlog-max-len 설정에 지정된 개수만큼 메모리에 저장된다.  레디스가 재 기동되면 슬로우 로그는 삭제 된다.
            
            slowlog-max-len값에 지정된 개수보다 많은 슬로우 로그가 생성되면 가장 오래된 슬로우 로그를 제거하고 새로운 슬로우 로그를 추가 한다.
            
        - **slowlog-max-len** : 최대 몇개의 슬로우 로그를 저장할 지 지정 한다. -1이면 4294967295갑이 지정되고, 기본은 128 이다.
            
            슬로우 로그를 조회하는 명령어는 slowlog 명령을 사용한다.
            
            ```kotlin
            127.0.0.1:6379[3]> **slowlog len    <-- 슬로우 로그 개수 카운팅**
            (integer) 30
            
            127.0.0.1:6379[3]> **slowlog get    <-- 슬로우 로그 확인**
            1)  1) (integer) 29
            		2) (integer) 1513679235
            		3) (integer) 109885
            		4) 1) "flushall"
            		5) "127.0.0.1:50060"
            		6) ""
            
            127.0.0.1:6379[3]> **slowlog reset  <-- 슬로우 로그 초기화**
            OK
            
            127.0.0.1:6379[3]> slowlog len
            (integer) 0
            ```
            
    - **8. 루아 설정**
        
        레디스 서버에서 루아 스크립트를 수행 할 수 있는데, 레디스 서버는 단일 스레드로 동작 되기 때문에,
        
        루아 스크립트가 돌아가는 동안 다른 명령을 처리 할 수 없게 되어(원자적), 다른 클라이언트의 응답시간이 증가 하게 된다.
        
        그래서 레디스에서 루아 스크립트에 대해서 설정 할 수 있는 부분은 실행 시간을 제어 하는 부분 뿐이다.
        
        루아 스크립트 수행은 eval 명령을 이용한다.
        
        - **lua-time-limit** : 루아 스크립트가 수행 될 최대 시간을 밀리초 단위로 설정해 준다. 해당 시간이 지낭야지만 SCRIPT KILL 명령을 이용해서 루아 스크립트를 중지 할 수 있게 된다. 기본값은 5000(0.005초)
            
            0 이나 음수를 입력하면 스크립트 제한 시간에 제한이 없게 된다.
            
            루아 스크립트가 수행되고 lua-time-limit 설정 값이 되기 전에 SCRIPT KILL 명령을 수행해도 루아 스크립트는 중지 되지 않음.
            
            무한 루프의 루아 스크립트가 수행되고 있을 때 다른 세션에서는 다음과 같은 메세지가 출력 됨.
            
            BUSY Redis is busy running a script. You can only call SCRIPT KILL or SHUTDOWN NOSAVE.
            
    - **9. 고급 설정**
        - **9.1 데이터 구조에 대한 설정**
            - hash 데이터
                - **hash-max-ziplist-entries** : 해시 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터 개수를 지정한다. 이 개수보다 많아지게 되면 hashtable 인코딩이 사용 된다.
                - **hash-max-ziplist-value** : 해시 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터의 크기를 지정한다. 이 크기보다 커지게 되면 hashtable 인코딩이 사용 된다.
            - list 데이터
                - **list-max-ziplist-entries** : 리스트 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터 개수를 지정한다. 이 개수보다 많아지게 되면 linkedlist인코딩이 사용 된다.
                - **list-max-ziplist-value** : 리스트 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터의 크기를 지정한다. 이 크기보다 커지게 되면 linkedlist인코딩이 사용 된다.
            - set 데이터
                - **set-max-intset-entries** : 셋 데이터에서 intset 인코딩을 사용하여 저장할 데이터 개수를 지정한다. 이 개수보다 많아지게 되면 hashtable 인코딩이 사용 된다.
            - sorted set 데이터
                - **zset-max-ziplist-entries** : 정렬된 셋 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터의 개수를 지정한다. 이 개수보다 커지게 되면 skiplist 인코딩이 사용 된다.
                - **zset-max-ziplist-value** : 정렬된 셋 데이터에서 ziplist 인코딩을 사용하여 저장할 데이터의 크기를 지정한다. 이 크기보다 커지게 되면 skiplist 인코딩이 사용 된다.
            - **client-output-buffer-limit** : 레디스 인스턴스는 클라이언트로 데이터를 전송하기 위하여 버퍼를 생성한다.
                
                레디스 인스턴스에서는 일반 클라이언트와 슬레이브 노드 두개의 종류가 있다고 본다.
                
                일반적인 클라이언트는 큰 문제가 되지 않지만, 슬레이브의 경우는 마스터 노드의 전체 데이터 셋을 네트워크를 통해서 전송하기 위하여 큰 용량의 버퍼가 필요하게 된다.
                
                슬레이브가 많을 경우 슬레이브 당 전송 버퍼를 할당해야 하므로 데이터를 저장할 메모리 공간이 줄어들게 된다. 그래서 이 파라미터를 제공하며 hard limt과 soft limt 두 가지를 제공한다.
                
                hard limit은 지정한 값 버퍼가 늘어나게 되면 클라이언트 연결을 해제 한다.
                
                soft limit은 시간을 보조 인자로 가지게 된다. 크기와 시간을 설정 후 해당 크기 까지 늘어난 후 설정 한 시간을 초과 할 경우 클라이언트 연결을 해제 한다.
                
                **normal** : 레디스 명령을 송수신하기 위한 클라이언트 연결
                
                **slave** : 복제를 위한 슬레이브 연결 또는 monitor 명령을 수행 중인 클라이언트 연결
                
                **pubsub** : 레디스 명령을 송수신 하기 위한 클라이언트 중에서  pub/sub 명령을 수행 중인 연결
                
                **client-output-buffer-limit normal 0 0 0 slave 256MB 64MB 60 pubsub 32MB 8MB 60**
                
                normal 0 0 0  : 버퍼의 크기에 상관없이 클라이언트 연결을 해제 하지 않는다.
                
                slave 256MB 64MB 60 : 슬레이브 노드를 위한 버퍼 크기가 256M까지 늘어나거나, 64M까지 늘어난 후 60초간 해당 상태가 지속되면 연결을 해제 함.
                
                pubsub 32MB 8MB 60 : pub/sub 명령을 위한 버퍼 크기가 32B 까지 늘어나거나, 8M까지 늘어난 후 60초간 상태가 지속되면 연결을 해제 함.
                

## 3. Spring Data Redis + Spring Boot

### Configuration

---

- **의존성 설정**
- **Config**
    
    ```kotlin
    package com.smartfoodnet.config
    
    import com.smartfoodnet.common.utils.Log
    import org.springframework.beans.factory.annotation.Value
    import org.springframework.context.annotation.Bean
    import org.springframework.context.annotation.Configuration
    import org.springframework.data.redis.connection.RedisConnectionFactory
    import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory
    import org.springframework.data.redis.core.RedisTemplate
    import org.springframework.data.redis.repository.configuration.EnableRedisRepositories
    import redis.embedded.RedisServer
    import javax.annotation.PostConstruct
    import javax.annotation.PreDestroy
    
    @Configuration
    @EnableRedisRepositories(basePackages = ["com.smartfoodnet.invoiceservice.redis"])
    class RedisConfig(
        @Value("\${spring.redis.host}") private val host: String,
        @Value("\${spring.redis.port:6379}") private val port: Int,
    ) {
    
        lateinit var redisServer: RedisServer
    
        @PostConstruct
        fun startRedisServer() {
            log.info("starting redis ...")
    
            redisServer = RedisServer(port)
            redisServer.start()
        }
    
        @PreDestroy
        fun stopRedisServer() {
            log.info("shutting down redis ...")
    
            redisServer.stop()
        }
    
        @Bean
        fun redisConnectionFactory(): RedisConnectionFactory = LettuceConnectionFactory(host, port)
    
        /**
         * stringRedisTemplate 로는 만들 수 없다. key 만 String 인 RedisTemplate bean.
         * @see org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration#stringRedisTemplate(RedisConnectionFactory)
         */
        @Bean
        fun redisTemplate(redisConnectionFactory: RedisConnectionFactory): RedisTemplate<String, Any> {
            return RedisTemplate<String, Any>().apply {
                setConnectionFactory(redisConnectionFactory)
            }
        }
    
        companion object : Log
    }
    ```
    
    - `@EnableRedisRepositories`: Redis 레파지토리 뒤질 basePackages 경로를 지정해준다.
    - 따로 지정하지 않으면, Redis 자동구성에 의해 뒤지긴 뒤지는데 이 Repository 가 Redis 전용인지? 다른 Spring Data Module 인지? 판단할 수 없어서 다음과 같은 로그가 막 찍힌다.
        
        ```kotlin
        .RepositoryConfigurationExtensionSupport : Spring Data Redis - Could not safely identify store assignment for repository candidate interface testapp.repositories.entries.UserEntryRepository.
        .RepositoryConfigurationExtensionSupport : Spring Data Redis - Could not safely identify store assignment for repository candidate interface testapp.repositories.ShareRepository.
        ...
        ```
        
        - 레퍼런스: [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.multiple-modules](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.multiple-modules)
        - 같은 문제 겪은 글: [https://stackoverflow.com/questions/39432764/info-warnings-about-multiple-modules-in-spring-boot-what-do-they-mean](https://stackoverflow.com/questions/39432764/info-warnings-about-multiple-modules-in-spring-boot-what-do-they-mean)

![./image/Spring_Data_Redis+Spring_Boot/Untitled%201.png](./image/Spring_Data_Redis+Spring_Boot/Untitled%201.png)

### redis repository 방식

---

![./image/Spring_Data_Redis+Spring_Boot/Untitled%202.png](./image/Spring_Data_Redis+Spring_Boot/Untitled%202.png)

- 레퍼런스: [https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis.repositories](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis.repositories)
- Redis Server version `2.8.0` 이상 → 위는 임베디드 레디스에서 지원하는 Redis Server version
- 트랜잭션은 지원하지 않는다.
    - `RedisTemplate` 생성 시 setEnableTransactionSupport 값을 `true`로 주면 안된다.
    - [https://github.com/spring-projects/spring-data-redis/issues/1437](https://github.com/spring-projects/spring-data-redis/issues/1437)
- 왜 사용함??
    - 도메인 객체를 쉽게 변환하고 저장할 수 있도록 해준다.
    - `RedisTemplate`를 사용시 직접 `json`으로 만들어 작업해야 되지만, 위는 쉽게 저장, 조회 가능하다.
    - 도메인 주도적으로 개발 시 매우 유용하다.

### 사용법

---

- **Entity**
    
    ```kotlin
    package com.smartfoodnet.invoiceservice.redis.entity
    
    import org.springframework.data.annotation.Id
    import org.springframework.data.redis.core.RedisHash
    
    @RedisHash(
        value = "AUTH_",            // key prefix = @RedisHash Annotation value + Redis Entity @Id value
        timeToLive = 60 * 60 * 12L  // 12h
    )
    data class AccessAuth(
        @Id
        var email: String,
        var accessTokenExp: Long,
        var accessToken: String,
    )
    ```
    
    - `@RedisHash`: 해당 엔티티가 Redis 엔티티임을 명시한다.
        - 레퍼런스: [https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisHash.html](https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisHash.html)
- **Repository**
    
    ```kotlin
    package com.smartfoodnet.invoiceservice.redis
    
    import com.smartfoodnet.invoiceservice.redis.entity.AccessAuth
    import org.springframework.data.repository.CrudRepository
    
    interface AccessAuthCacheRepository : CrudRepository<AccessAuth, String>
    ```
    
- **Service**
    
    ```kotlin
    @Service
    class AuthService(
        private val restTemplate: RestTemplate,
        private val accessAuthCacheRepository: AccessAuthCacheRepository,
    ) {
        @Value("\${sfn.base-api-paths.admin-service}")
        private val adminServiceUrl: String? = null
    
        private val headers = HttpHeaders()
        private val httpEntity: HttpEntity<Any>
    
        private val adminEmail = ""
        private val password = ""
    
        init {
            headers.contentType = MediaType.APPLICATION_JSON
            // 24시간 토큰 발행용 헤더 (https://thesfn.slack.com/archives/C0202BRTLMU/p1622005885010700)
            headers.set("wonder", "")
    
            val body = JSONObject()
                .put("email", adminEmail)
                .put("password", password)
    
            httpEntity = HttpEntity(body.toString(), headers)
        }
    
        fun getAccessToken(): String? {
            var accessAuth = accessAuthCacheRepository.findByIdOrNull(adminEmail)
    
            if (accessAuth == null || !checkValidToken(accessAuth)) {
                accessAuth = getAccessAuth()
                    ?.let { accessAuthCacheRepository.save(it) }
            }
    
            if (accessAuth == null) {
                throw Exception("access 토큰 발행이 실패하였습니다.")
            }
    
            return accessAuth.accessToken
        }
    
        fun getAuthorizedHeader(): HttpHeaders {
            val httpHeaders = HttpHeaders()
            httpHeaders.set(HttpHeaders.AUTHORIZATION, "bearer ${getAccessToken()}")
    
            return httpHeaders
        }
    
        private fun getAccessAuth(): AccessAuth? {
            val response = restTemplate.exchange(
                "$adminServiceUrl/signin",
                HttpMethod.POST,
                httpEntity,
                AccessAuthResponse::class.java
            ).body
    
            return response?.payload?.toEntity()
        }
    
        private fun checkValidToken(accessAuth: AccessAuth): Boolean {
            val now = LocalDateTime.now()
            val accessTokenExp = Timestamp(accessAuth.accessTokenExp * 1000) // to ms
            return (accessTokenExp.after(Timestamp.valueOf(now)))
        }
    }
    ```
    

- 참고
    - [https://dolsup.work/posts/how-to-build-medis-on-macos/](https://dolsup.work/posts/how-to-build-medis-on-macos/)
    - [https://ka0oll.tistory.com/25](https://ka0oll.tistory.com/25)
    - [https://bstar36.tistory.com/349](https://bstar36.tistory.com/349)
