# 12장. 아키텍처 스타일 결정하기

- 언제 실제로 육각형 아키텍처 스타일을 사용해야 할까?

## 도메인이 왕이다

---

- 육각형 아키텍처 스타일의 주요 특징: 영속성 관심사나 외부 시스템에 대한 의존성 등의 변화로부터 자유롭게 도메인 코드를 개발할 수 있는 것
    
    > 외부의 영향을 받지 않고 도메인 코드를 자유롭게 발전시킬 수 있다는 것은 육각형 아키텍처 스타일이 내세우는 가장 중요한 가치다.
    > 
    - 육각형 스타일과 같은 도메인 중심의 아키텍처 스타일은 DDD의 조력자
    - 도메인을 중심에 두는 아키텍처 없이는, 설계가 항상 다른 요소들에 의해 주도되고 말 것이다.

😃 만약 도메인 코드가 애플리케이션에서 가장 중요한 것이 아니라면 이 아키텍처 스타일은 필요하지 않을 것이다.

## 경험이 여왕이다

---

- 아키텍처 스타일에 대해서 괜찮은 결정을 내리는 유일한 방법: 다른 아키텍처 스타일을 경험해 보는 것

😃 육각형 아키텍처에 대한 확신이 없다면 지금 만들고 있는 애플리케이션의 작은 모듈에 먼저 시도해 보라.

→ 이 경험이 다음 아키텍처 결정을 이끌어 줄 것이다.

## 그때그때 다르다

---

- 어떤 소프트웨어를 만드느냐에 따라서도 다르고, 도메인 코드의 역할에 따라서도 다르다.
- 팀의 경험에 따라서도 다르다.
- 최종적으로는 내린 결정이 마음에 드느냐에 따라서도 다르다.