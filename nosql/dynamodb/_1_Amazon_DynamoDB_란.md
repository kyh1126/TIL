# Amazon DynamoDB 란?

- 레퍼런스: [https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Introduction.html](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Introduction.html)

## 특징

- 완벽 관리되는 NoSQL DB 서비스
- 원활한 확장성 + 빠르고 예측 가능한 성능 제공
- 분산 DB 운영, 조정에 따른 관리 부담을 줄일 수 있어, 하드웨어 프로비저닝, 설정 및 구성, 복제, 소프트웨어 패치, 클러스터 조정에 대해 걱정할 필요가 없다.
- 유휴 시 암호화를 제공하여 중요한 데이터 보호 관련 운영 부담, 복잡성 제거

- 데이터 규모 관계없이 데이터 저장/검색하고, 어떤 수준의 요청 트래픽도 처리할 수 있는 DB 테이블을 생성할 수 있다.
- 다운타임 또는 성능 저하 없이 테이블의 처리 능력을 확장 또는 축소할 수 있다.
- AWS Management Console을 사용하여 리소스 사용률 및 성능 지표를 모니터링할 수 있다.

- 온디맨드 백업 기능을 제공
    - 따라서 테이블의 전체 백업을 생성하여 규제 준수 요건에 맞도록 장기간 유지하고 보관할 수 있다.
- 테이블에 대해 특정 시점으로 복구를 활성화하는 것, 특정 시점으로 복구를 생성할 수도 있다.
    - 특정 시점으로 복구 기능을 사용하면 우발적인 쓰기 또는 삭제 작업으로부터 테이블을 보호할 수 있다.
    - 특정 시점으로 복구가 설정되어 있으면 최근 35일 중 원하는 시점으로 테이블을 복원할 수 있다.
- 테이블에서 만료된 항목을 자동으로 삭제할 수 있으므로 스토리지 사용량을 줄일 수 있으며, 더 이상 관련 없는 데이터를 저장하는 데 비용을 낭비하지 않아도 된다.
    - 자세한 내용: [DynamoDB TL(Time to Live) 을 사용하여 항목 만료](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/TTL.html) 단원


## 높은 가용성 및 내구성

- DynamoDB 는 테이블의 데이터와 트래픽을 충분한 수의 서버로 자동 분산하여 처리량 및 스토리지 요구 사항을 처리하면서도 일관되고 빠른 성능을 유지한다.
- 모든 데이터가 SSD(Solid State Disk)에 저장되고, AWS 리전의 여러 Availability Zones 에 걸쳐, 내장된 고가용성 및 데이터 내구성을 제공한다.
- 전역 테이블을 사용하여 DynamoDB 테이블을 AWS 리전 간 동기화를 유지할 수 있다.


## DynamoDB 시작하기

다음 단원을 읽고 시작하면 도움이 된다.

- **[Amazon DynamoDB: 작동 방식](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/HowItWorks.html) →** 기본 DynamoDB 개념 학습하기
- **[DynamoDB 설정](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/SettingUp.html) →** DynamoDB (다운로드 가능 버전 또는 웹 서비스) 설정 방법 학습하기
- **[DynamoDB 액세스](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/AccessingDynamoDB.html) →** 콘솔을 사용하여 DynamoDB 에 액세스 하려면 AWS CLI, API를 사용할 수 있다.

- DynamoDB 신속하게 사용 → [DynamoDB 및 시작하기 AWS SDK](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/GettingStarted.html).
- 애플리케이션 개발에 대한 자세한 내용
    - [DynamoDB 및AWS SDK](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Programming.html)
    - [테이블, 항목, 쿼리, 스캔 및 인덱스 작업](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/WorkingWithDynamo.html)
- 성능 극대화 + 처리량 비용 최소화 방안을 신속히 → [DynamoDB 를 사용한 설계 및 아키텍처 설계 모범 사례](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/best-practices.html)
- DynamoDB 리소스에 태그 지정하는 방법 → [리소스에 태그 및 레이블 추가](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Tagging.html)
- 모범 사례, 방법 안내서 및 도구 → [Amazon DynamoDB 리소스](http://aws.amazon.com/dynamodb/resources/)
- AWS DB Migration Service(AWS DMS)로 RDB 또는 MongoDB 에서 DynamoDB 테이블로 데이터 마이그레이션 → [AWS Database Migration Service 사용 설명서](https://docs.aws.amazon.com/dms/latest/userguide/)
- 마이그레이션 소스로 MongoDB 를 사용하는 방법 → [AWS DMS 에서](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.DynamoDB.html) [MongoDB 를 원본으로 사용](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.MongoDB.html)
- 마이그레이션 대상으로 DynamoDB 를 사용하는 방법 → [AWS DMS 에서 Amazon DynamoDB 데이터베이스를 대상으로 사용](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.DynamoDB.html)


- [Notion link](https://jennyuni.notion.site/Amazon-DynamoDB-dcbab5d023f34952856c2ea74cc067e6)
