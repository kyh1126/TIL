# Chapter 15. 아키텍처란?

- 소프트웨어 아키텍트는 최고의 프로그래머이며, 앞으로도 계속 프로그래밍 작업을 맡을 뿐만 아니라 동시에 나머지 팀원들이 생산성을 극대화할 수 있는 설계를 하도록 방향을 이끌어 준다.
    - 소프트웨어 아키텍트는 코드와 동떨어져서는 안 된다.
    - 프로그래밍 작업을 계속하는 이유는, 발생하는 문제를 경험해보지 않는다면 다른 프로그래머를 지원하는 작업을 제대로 수행할 수 없기 때문이다.

- 좋은 아키텍처는 시스템을 쉽게 이해하고, 쉽게 개발하며, 쉽게 유지보수하고, 또 쉽게 배포하게 해준다.
    - 아키텍처의 주된 목적은 시스템의 생명주기를 지원하는 것이다.

## 개발

---

- 시스템 아키텍처는 개발팀(들)이 시스템을 쉽게 개발할 수 있도록 뒷받침해야만 한다.

## 배포

---

- 소프트웨어 아키텍처는 시스템을 단 한 번에 쉽게 배포할 수 있도록 만드는 데 그 목표를 두어야 한다.

## 운영

---

- 운영에서 겪는 대다수의 어려움은 소프트웨어 아키텍처에는 극적인 영향을 주지 않고도 단순히 하드웨어를 더 투입해서 해결할 수 있다.
- 좋은 소프트웨어 아키텍처는 시스템을 운영하는 데 필요한 요구도 알려준다.

## 유지보수

---

- 유지보수는 모든 측면에서 봤을 때 소프트웨어 시스템에서 비용이 가장 많이 든다.
    - 새로운 기능은 끝도 없이 행진하듯 발생하고, 뒤따라서 발생하는 결함은 피할 수 없으며, 결함을 수정하는 데도 엄청난 인적 자원이 소모된다.
    - 주의를 기울여 신중하게 아키텍처를 만들면 이 비용을 크게 줄일 수 있다.
    - 시스템을 컴포넌트로 분리하고, 안정된 인터페이스를 두어 서로 격리한다.
        - 이를 통해 미래에 추가될 기능에 대한 길을 밝혀 둘 수 있을 뿐만 아니라 의도치 않은 장애가 발생할 위험을 크게 줄일 수 있다.

## 선택사항 열어 두기

---

- 소프트웨어를 부드럽게 유지하는 방법: 선택사항을 가능한 한 많이, 그리고 가능한 한 오랫동안 열어 두는 것
    - 열어 둬야 할 선택사항: 중요치 않은 세부 사항

- 모든 소프트웨어 시스템은 주요한 두 가지 구성요소로 분해할 수 있다.
    - 정책: 모든 업무 규칙과 업무 절차를 구체화한다.
    - 세부사항: 사람, 외부 시스템, 프로그래머가 정책과 소통할 때 필요한 요소지만, 정책이 가진 행위에는 조금도 영향을 미치지 않는다.
        - 입출력 장치, 데이터베이스, 웹 시스템, 서버, 프레임워크, 통신 프로토콜 등

- 아키텍트의 목표: 시스템에서 정책을 가장 핵심적인 요소로 식별하고, 동시에 세부사항은 정책에 무관하게 만들 수 있는 형태의 시스템을 구축
    - 좋은 아키텍트는 결정되지 않은 사항의 수를 최대화한다.

## 장치 독립성

---

- 오늘날의 운영체제는 입출력 장치를 소프트웨어 함수로 추상화했고, 해당 함수는 천공카드와 같은 단위 레코드를 처리한다.
    - 개방 폐쇄 원칙

## 광고 우편

---

- 어떤 장치를 사용할지 전혀 모른 채, 그리고 고려하지 않고도 프로그램을 작성할 수 있었다.
    - 장치 독립성이 지닌 가치

## 물리적 주소 할당

---

- 시스템에서 고수준의 정책이 디스크의 물리적 구조로부터 독립되도록 수정했다.

## 결론

---

- 좋은 아키텍트는 세부사항을 정책으로부터 신중하게 가려내고, 정책이 세부사항과 결합되지 않도록 엄격하게 분리한다.
    - 이를 통해 정책은 세부사항에 관한 어떠한 지식도 갖지 못하게 되며, 어떤 경우에도 세부사항에 의존하지 않게 된다.
- 좋은 아키텍트는 세부사항에 대한 결정을 가능한 한 오랫동안 미룰 수 있는 방향으로 정책을 설계한다.