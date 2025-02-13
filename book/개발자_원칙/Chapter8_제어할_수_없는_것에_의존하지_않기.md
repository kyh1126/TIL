# 8장. 제어할 수 없는 것에 의존하지 않기

- 프로그래머에게 요구되는 것은 100점이 아닌 80~90점짜리 프로그램을 기한 내에 완성하는 일이다.
    
    → 어떻게 하면 아무리 급해도 항상 80~90점짜리 소프트웨어를 개발할 수 있는지
    

- 빠듯한 일정으로 일이 주어져도 항상 80~90점의 소프트웨어를 출시하는 분들이 있다.
    - 일반적으로 A 코드가 더 나을지, B 코드가 더 나을지 고민하면서 적지 않은 시간을 보내게 되는데,
        
        → 그분들은 A 코드와 B 코드 중 현재 상황에 더 적합한 코드를 판별하는 기준과 원칙이 있어 고민 없이 곧바로 선택한다.
        

😃 기준과 원칙에 따라 빠르게 결정을 내리면, 정말 중요한 설계와 선택이 필요할 때 더 깊게 사고할 수 있는 시간을 확보하고 사용하게 된다.

- 나는 좋은 원칙을 알고 있는가? 그 원칙들이 무의식적으로도 발현되게 내재화되어 있는가?

- 소프트웨어 개발자 사이에 가장 널리 알려진 원칙
    - DRY(Do not Repeat Yourself): 똑같은 기능, 코드를 반복하지 마라
    - YAGNI(You Ain’t Gonna Need It): 그 기능이 필요하기 전까지는 미리 만들지 말라
    - KISS(Keep It Simple Stupid): 최대한 단순함을 유지하라
- 당연하게 적용한 원칙: 언어와 프레임워크가 다르더라도 동일하게 적용될 거다.
- 가장 애정하는 원칙: 제어할 수 없는 것에 의존하지 않기
    
    > 현실 세계의 변화와 설계 사이의 결합도를 줄여야 한다. 전화번호를 식별자로 사용하는가? 자신의 힘으로 제어할 수 없는 속성에 의존하지 말라
    - 실용주의 프로그래머
    > 
    
    🥲 제어할 수 없는 것에 의존하면 변화에 민감한, 흔들리기 쉬운 소프트웨어가 된다.
    

## 코드 설계에 적용하기

---

- 테스트 코드 작성이 어려울 때: 코드 자체가 테스트하기에 적당하지 않은 코드일 때이다.
    
    ```jsx
    // 주문일이 일요일이면 주문 금액의 10%를 할인하는 메서드
    export default class Order {
      ...
      discount() {
        const now = LocalDateTime.now(); // 현재 시간을 반환하는 메서드
        
        if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
          this._amount = this._amount * 0.9
        }
      }
    }
    ```
    
    🥲 이 코드는 결과를 예측하기 어려운, 즉 테스트하기 어려운 코드이다.
    
    - 실행할 때마다 달라지는 값, 현재 시간 생성 함수 `LocalDateTime.now()`를 사용하고 있기 때문
    - 테스트 코드
        
        ```jsx
        it('일요일에는 주문 금액이 10% 할인된다', () => {
          const sut = Order.of(10_000, OrderStatus.APPROVAL);
          
          sut.discount();
          
          expect(sut.amount).toBe(9_000);
        });
        ```
        
        - 매주 일요일에 수행할 때만 통과할 수 있다.

→ 제어하기 어려운 코드인 현재 시간 메서드를 외부에서 주입받도록 수정하면 된다.

```jsx
export default class Order {
  ...
  // 현재 시간 메서드(now)를 밖에서 주입받음
  discountWith(now: LocalDateTime) {    
    if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
      this._amount = this._amount * 0.9
    }
  }
}
```

😃 제어할 수 없는 값인 now()를 메서드 인수로 빼도록 변경하는 순간 Order.discountWith() 메서드는 항상 같은 결과를 뱉어내고, 테스트 역시 항상 일관된 결과를 출력하게 된다.

- 제어할 수 없는 값에 의존하는 코드들을 최대한 멀리한다.
- 주요 비즈니스 로직은 모두 제어할 수 있는 값만 의존하게 해 테스트 코드 작성이 쉬운 형태로 구성한다.

- 메서드/함수가 제어할 수 없는 값에 의존하지 않을수록 항상 같은 결과를 반환하는 부수효과가 적은 메서드/함수가 된다.

## 이직에 적용하기

---

- 시리즈 A 이하의 스타트업이라 하면 정말 많은 부분이 부족하다.
    - 소수의 개발팀
    - 적은 제품 성공 경험
    - 부족한 복지, 보상

- 제어할 수 없는 요소인 외부의 투자가 없어도 생전 가능한 조직은 강력한 선택 기준이되었다.

## 조직과 매니징에 적용하기

---

- 인프랩 합류 후, 처음 한 달 동안 어떤 특별한 액션을 하기보다는 기존 팀원들이 어떻게 일을 하는지, 사내의 프로세스는 어떻게 되는지 지켜보기만 했다.
- 가서 바로 무언가를 하려고 하기보다는 1~2달은 가만히 지켜만 봐라
    
    → '조직 내에서 '제어할 수 있는 것'과 '제어할 수 없는 것'을 분류하는 기간
    

- CTO가 제어할 수 없는 것
    - 개발팀 전체의 연봉 테이블
        - 회사의 매출, 투자 규모, 다른 직군과의 연봉 격차, 회사의 연간 예산 계획 등 (제어할 수 없는)모든 것을 고려해야 하기 때문이다.
    - 개발팀 규모의 확장
    - 기존에 같이 일해본 혹은 전 직장 분들의 합류
    - 기존 서비스의 기술 스택 교체
        - 특히나 기존 서비스의 기술 스택을 교체하는 일을 신뢰 관계가 전혀 없는 새로 온 CTO가 진행하면 기존 구성원들의 반발심만 키울 뿐이다.

- 제어할 수 있는 영역 위주로 신뢰를 쌓기 시작했다.
    - 1 on 1 미팅: 인간적인 신뢰
        - 3개월 동안 1인당 최소 2회 이상씩 티타임을 가지며 이야기를 나누었다.
        - 상호 유대감을 얼마나 쌓아놓느냐가 일할 때 정말 중요하다.
        - 기존 팀원들에게 제가 어떤 사람인지, 팀원들과 어떤 목표를 달성했으면 하는지, 각 팀원들의 꿈과 비전은 무엇인지 등을 티타임을 가지며 이야기했다.
    - 모니터링과 로깅 시스템을 도입하기로 결정: 엔지니어링 영역에서의 신뢰
        - (제어할 수 없는 영역인) 기존 시스템에 큰 변화를 주지 않으면서도 큰 효과를 볼 수 있는 영역
        - 없으면, 과거의 실수로부터 배우지 못한다.
            - 어느 시점부터 문제가 시작된 것일까?
            - 이상 징후가 시작된 지점은 언제부터였을까?
            - 우리 시스템의 트래픽 임계점은 어디까지일까?
        - 당장 도입하기보다 현재 상황을 먼저 정리하고 확인한 뒤에 당위성을 얻어서 진행했다.
            - 이렇게 해야 조직의 신뢰가 더 쌓일 거라 믿었기 때문
            - 과거의 장애 내역을 정리하고 공유하면서 현재 시스템이 얼마나 불안한 상태인지, 우리가 얼마나 놓치고 있는 게 많은지 등에 공감대를 형성했다.
- 공감대가 형성된 이후의 일
    - 데이터베이스 개선(Aurora PostgreSQL로 마이그레이션)
    - 데이터베이스 쿼리 로깅 환경 구축 및 슬로우 쿼리 알람
        - 3초 이상 쿼리들에 대한 알람 설정과 발견된 슬로우 쿼리 개선
    - 애플리케이션 모니터링 시스템 도입
    - 필요한 알람과 불필요한 알람 정리

→ 서비스의 안정성과 구성원들의 신뢰를 회복할 수 있다.

- 더 큰 규모의 팀이 되려면 결국 동시대를 살아가는 개발자들의 흐름을 놓치지 말아야 한다.
    
    → 개발 팀원 전체가 동시대성을 갖추기 위한, 성장을 위한 일을 진행했다.
    
    - 제어할 수 없는 영역: 구성원 본인의 성장하고자 하는 열망
        
        🥲 혹자는 성장에 대한 열정도 주변에서 심어줄 수 있다고 생각하겠지만, 저는 그걸 성공해본 적이 없다.
        
    - 제어할 수 있는 영역
        - 성장에 대한 열망을 가지고 있는 분을 과제와 면접을 통해 채용하기
            - 신규 채용에서는 열망이 있는 분만 뽑았다.
        - 성장에 필요한 과제와 환경을 제공하기
            - 채용 과정에서 실제 웹 서비스를 만드는 과제 테스트 추가하기. 제출된 과제들을 팀원들과 함께 리뷰하면서 외부의 개발자들은 어떤 구조로, 어떤 형태로 코드를 작성하는지 자연스럽게 공유하기
            - 앞으로 우리가 갈 길을 기존 개발자 분들에게 명시하는 방향으로 채용 공고 작성하기
            - 신규 프로젝트를 새로운 기술 스택만으로 구성하기
            - 개발팀 전체가 함께 하는 스터디 진행하기
            - 주변의 좋은 시니어들과 팀원들과의 티타임, 미팅 등 주선하기
        - 옆에서 지켜보면서 적절한 피드백을 전달하기

- 작은 개발팀을 꾸리고 있는 스타트업에서 빅테크로 이직하는 동료가 많으면 위기를 겪게 된다.
    - 조금만 삐끗하면 입사 후 1~2년 차에 빅테크로 이직하는 조직이 되고, 1~2년 뒤에 빅테크로 이직하는 것을 목표로 하는 사람들만 입사한다.
- '어떻게 하면 이 상황을 긍정적으로 해석할 수 있을까'를 고민하고 행동으로 옮겨야 한다.
    - 물론 회사가 부족할 때 함께 해준 팀원들에 대해 정말 감사한 마음을 갖고, 점진적으로 충분한 보상을 해줘야 하는 것은 당연하다.

- 제어할 수 있는 것에 의존하고 집중해야만 어떤 일과 상황을 만나더라도 앞으로 전진할 수 있다.
    - 외부의 요소, 이미 발생한 사건, 결정권이 없는 일 등은 제어할 수 없다.
    - 소프트웨어를 설계한다면 제어할 수 있는 속성에 항상 의존하게 설계하야 하며, 현실 세계의 문제라면 현재의 사건과 환경을 어떻게 하면 더 유리하게 바라볼 수 있는지 고민하고 행동으로 옮겨야 한다.