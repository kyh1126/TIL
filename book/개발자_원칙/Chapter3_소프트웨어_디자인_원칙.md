# 3장. 소프트웨어 디자인 원칙

## 디자인이란 무엇인가?

---

- 디자인의 한국말은 '설계'
    1. 계획을 세움
    2. 건축물 설립이나 토지 공사, 기계의 제작 따위에서 그 목적에 따라 실제적인 계획을 세우고 구체적으로 도면을 그려 명시하는 일

## 설계와 요구사항

---

- 설계: 제품이 요구사항을 만족시키는 것을 증명하는 조건을 정의하는 행위

## 소프트웨어 설계 원칙: 통합적으로 설계하라

---

- 소프트웨어를 설계할 때 최대한 많은 정보를 기반으로 요구사항을 정리해야 한다.
- 해당 도메인에 10년 이상 경험이 있지 않는 한 기획자가 개발에 필요한 정보를 구체적으로 소프트웨어 요구사항 명세(Software Requirement Specification, SRS)에 기술하기에는 한계가 있다.

## 명시적 소프트웨어 설계

---

- 기본적으로 명시적으로 요구사항과 연결되는 설계

### 기능 설계

---

- SRS나 사용자 요구사항을 해결하는 1차적이고 가장 기본적인 그래서 가장 중요한 설계

### 성능 설계

---

- 설계의 결과물로 반드시 성능과 관련된 부분들도 표시 또는 기록해야 한다.

### 유지보수 설계

---

- 요구사항에 부응하는 유지보수 관련 기술이나 제품을 선정해야 한다.

### 미적 설계

---

- 사용자에게 물리적으로 심리적으로 즐거움과 편리함을 줄 수 있도록 제품을 미적으로 설계해야 한다.

### 사용성 검증

---

- 요구사항을 만족시키는 조건을 결정할 때 반드시 검증을 고려해야 한다.

## 암묵적 소프트웨어 설계

---

- 암묵적 설계: SRS에 직접적으로 명시되지는 않지만, 비즈니스나 기획 심지어 개발에서도 실제로는 그 일을 수행한다.
    - 암묵적 설계 항목은 소프트웨어 개발의 60% 이상을 차지한다.
    
    > 사실41. 유지보수는 소프트웨어 전체 비용 중에 40~80%를 소비한다. 그리고 아마도 소프트웨어 사이클에서 가장 중요한 영역이다.
    > 
    > 
    > 사실42. 개선하는 데는 유지보수 비용의 60% 정도가 필요하다.
    > 

- 암묵적 설계는 유지보수만을 위한 설계는 아니지만 많은 부분이 소프트웨어 제품을 지속적으로 운영하는 데 필요한 요소를 고려하기 때문에 유지보수 개념도 포함한다.
- 현대의 소프트웨어는 명시적 설계와 암묵적 설계를 나누는 방식을 선호하지 않는다.

### 서비스 지속성 설계

---

- 출시된 서비스가 지속적으로 실행되는 데 문제가 없도록 하는 설계 요소들

- 가용성 설계
    - 가용성을 고려하려면 단순히 서버를 두 대 마련하는 수준에서는 안 된다.
    - '평균 무고장 시간(Mean Time Between Failure, MTBF), 평균 수리 시간(Mean Time To Repair, MTTR)은 얼마인가요?' 라는 질문에 대한 답할 수 있어야 한다.
    
    → 제품 초기라면 평균 무고장 시간은 100일, 평균 수리 시간은 12시간 정도로 설계해야 한다.
    
- 용량 설계
    - 처리 용량
        - 처리해야 하는 트래픽양은 얼마인지
        - 네트워크라면 몇 bPSbit Per Second이고
        - 서버라면 예상되는 동시 세션은 얼마이고 그래서 TPS(Transaction Per Second)는 얼마여야 요구사항을 만족할 수 있는지
        - 평균 응답 시간(Mean Response Time, MRT)은 얼마가 나와야 하는지
    - 저장 용량
        - 생성되는 로그양과 데이터 저장량은 기존의 모노리틱 구조에서 마이크로서비스 아키텍처로 분화된 개수만큼 늘기 때문에 데이터 보관하고 처리할 용량도 미리 설계해야 한다.
- 연속성 설계
    - 가용성과 비슷하지만 가용성을 바탕으로 소프트웨어가 끊임없이 서비스되게 설계하는 점이 다르다.
- 보안 설계
    - 데이터의 기밀성(conformity)과 정합성(integrity)에 대한 수준을 수치적으로 제시하는 조건을 설정해야 한다.
    - 절차적 보안사항 외에도 보안 수준을 수치적으로 표시하는 지표를 목표로 설계해야 한다.
        - 대표적으로 CVSS(Common Vulnerability Scoring System) 점수 목표를 설정하고 그에 따른 컴포넌트들을 설계에 포함해야 한다.

### 서비스 전환 설계

---

- 소프트웨어는 늘 새로운 기능을 제공하기 때문에 반드시 서비스의 전환에 대한 설계를 해두어야 한다.

- 변환 설계
    - 소프트웨어 제품이 추가되거나 또는 기능이 새롭게 개발되거나 폐기되었을 때 기존 제품에 미치는 영향을 최소화하는 설계 요소
    - 새로운 기능 출시나 개발을 막는 가장 큰 어려움: 기존에 시장에서 실행되고 있는 제품 → 레거시
    - 장애 등급을 나눈다.
        - 3급: 30분 내외
        - 2급: 1시간 내외
        - 1급: 수 시간 장애
- 릴리즈 설계
    - 새로 들어간 기능의 버전을 어떻게 할지 메이저 버전 업그레이드인지 아닌지, 릴리즈를 할 정도로 소프트웨어가 완성되었는지에 대한 설계
        - 주로 테스트 계획으로 많이 통용된다.
    - 릴리즈를 어떻게 배포할지도 설계해야 한다.
        - 블루/그린으로 할지 아니면 카나리아 형태로 할지
- 설정 관리 설계
    - 리소스 관련 설정을 저장하고 변경이나 접근을 관리하는 CMDB(configuration management DB)를 사용할 것이냐 말 것이냐, 여기에 어떤 정보를 넣고 접근 권한은 어떻게 줄 것인가와 관련된 부분도 설계해두어야 한다.

### 서비스 운영 설계

---

- 릴리즈된 소프트웨어가 설계된 조건(MTR, SLA 등)을 만족하도록 하는 설계

- 장애 대비 설계
    - 온라인으로 제공되는 서비스가 외부의 상황이나 내부의 서비스 하부 컴포넌트의 중단에 영향을 받지 않도록 설계하고, 실제로 장애가 발생했을 때 수치에 따라 정해진 등급대로 행동하도록 지정해야 한다.
- 요구 수행 설계
    - 사용자 문의에 대한 대답이나, 소프트웨어에 대한 컨설팅을 실행하는 것
    - ISMS-P 규격에 의하면 개발자와 운영자를 반드시 분리해야 하기 때문에 미리 담당 영역을 정해두어야 한다.
- 문제 대응 설계
    - 서비스 지표들을 기반으로 특정 값들이 자주 나타나거나 임곗값을 넘어가면 다시 설계를 하자고 대응책을 마련할 수 있다.
    - 주로 기술적 부채로 부르는 문제를 어떻게 다룰지 미리 설계해두어야 한다.

### 서비스 개선 설계

---

- 소프트웨어가 실제 환경에서 실행되는 순간부터는 정말 문제없이 정해진 일들을 수행하는지에 대한 지표를 확인하고 개선할 준비를 해야 한다.

- 서비스 리포팅 설계
    - 지표: 서비스 지속성 설계, 서비스 전환 설계
    - 리포팅은 앞선 설계 예측치들을 적절하게 보여줄 수 있게 설계해야 한다.
- 서비스 측정 설계
    - 정상 작동 상태를 파악할 수 있는 지표들을 설정하고, 표시하도록 만들어야 한다.
- 서비스 레벨 설계
    - 개인적으로 소프트웨어 설계에서 가장 강조하는 영역
    - 반드시 한계점을 잡아두라
        - ex> 지표 자료를 15분 안에는 화면에 표시해야 한다는 하한치를 수립하고 일단 설계 목표로 사용할 수 있다.

- 제품 개발에서 가장 중요하다고 생각했던 기능 설계 분량이 전체 소프트웨어 설계에서 1/17만 차지한다.
- 소프트웨어 설계 대부분의 영역에 측정 가능한 조건을 설정할 수 있다.
- 전체 요소가 연관되게 설계해야 한다.
    - 각 요소를 그다음 단계로 강화시키거나 새로운 기능을 집어넣을 때마다 모든 요소에 영향을 미치기 때문에 그때마다의 최적화를 해야 한다.
    - ex> 데이터가 화면에 표시되기까지 15분 걸리던 하한값을 1분으로 줄인다면 분명히 암묵적 설계로 되어 있지만 성능 설계부터 또는 기능 설계부터 다시 해야 한다.

## 통합 설계의 미래

---

- 2021년 8월 NTIA은 SBOM에 들어갈 기초적이지만 필수적인 항목을 지정했다.
    
    > - 데이터 항목: 소프트웨어를 구성하는 데 필요한 컴포넌트의 공급망과 다른 소프트웨어의 연결 상태 그리고 취약성 등을 파악하는 데 필요한 정보의 데이터 항목을 반드시 포함해야 한다.
    (제조사 이름, 컴포넌트 이름, 컴포넌트 버전, 구분키, 의존성 정보, 시간 정보 등)
    - 자동화 지원: 소프트웨어를 자동으로 설치하고 관리하는 데 필요한 아키텍처 정보, 스케일아웃 정도를 표준화한 자동화 데이터로 제공해야 한다.
    - 실행과 프로세스: 소프트웨어의 새로운 빌드나 릴리즈가 생길 때 해당 내역을 SBOM에 반영할 수 있어야 한다. 주기, 의존성 깊이 정보, 의존성 한계 표시, 배포와 전달 방식, 접근 제어, 장애 포용 등의 내용이 적용되어야 한다.
    > 

### 소프트웨어 디자인 도구의 변화

---

- 2022년 IT 최고의 화두: 노코드(no-code) 또는 로우코드(low-code)용 소프트웨어 제품
    - 소프트웨어 개발자 수요가 폭발했다.
    - 그러나 프로그래머 수가 이를 따라가지 못했다.

- 방향성은 명확해지고 있다.
    - 앞으로 10년 후면 소프트웨어 설계란 코드 편집이 아니라 특정 제품을 실행하고, 17개 항목에 맞도록 제품의 기능, 성능, 미학 설계를 하고, 암묵적 설계를 완료한 다음에
    - 특정 제품의 설계 결과물인 SBOM을 기반으로 테스트와 배치까지 실행하는 것 을 의미할 것인지도 모른다.

→ 이것이 개발자가 코드와 객체지향 수준의 원칙을 넘어 설계 원칙을 익혀야 하는 이유이다.