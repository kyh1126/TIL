# Test-Driven Development

# 1부. 화폐 예제

- 테스트 주도 개발(TDD) 의 리듬
    1. 재빨리 테스트를 하나 추가한다.
    2. 모든 테스트를 실행하고 새로 추가한 것이 실패하는지 확인한다.
    3. 코드를 조금 바꾼다.
    4. 모든 테스트를 실행하고 전부 성공하는지 확인한다.
    5. 리팩토링을 통해 중복을 제거한다
- 효과
    - 각 테스트가 기능의 작은 증가분을 어떻게 커버하는지
    - 새 테스트를 돌아가게 하기 위해 얼마나 작고 못생긴 변화가 가능한지
    - 얼마나 자주 테스트를 실행하는지
    - 얼마나 수 없이 작은 단계를 통해 리팩토링이 되어가는지


## 1장. 다중 통화를 지원하는 Money 객체

- 앞으로 어떤 일을 해야 하는지 알려주고, 지금 하는 일에 집중할 수 있도록 도와주며, 언제 일이 다 끝나는지 알려줄 수 있게끔 할일 목록을 작성해보자.
    1. 통화가 다른 두 금액을 더해서 주어진 환율에 맞게 변한 금액을 결과로 얻을 수 있어야 한다.
    2. 어떤 금액(주가)을 어떤 수(주식의 수)에 곱한 금액을 결과로 얻을 수 있어야 한다.
- 한 항목에 대한 작업을 시작하면 그 항목을 굵은 글씨체로 나타내자.
- 작업을 끝낸 항목에는 ~~이런 식으로~~ 줄을 긋자.

- 2. 곱한 금액을 결과로 얻을 수 있어야 한다. 부터 다뤄보자.
    - 어떤 객체가 있어야 할까? (X)

    → 객체를 만들면서 시작하는게 아니라 테스트를 먼저 만들어야 한다.

- 테스트 주기
    1. 작은 테스트를 하나 추가한다.
    2. 모든 테스트를 실행해서 테스트가 실패하는 것을 확인한다.
    3. 조금 수정한다.
    4. 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다.
    5. 중복을 제거하기 위해 리팩토링을 한다.

- 테스트를 작성할 때는 오퍼레이션(메소드)의 완벽한 인터페이스에 대해 상상해보는 것이 좋다. → 가짜로 구현하기
    - 간단한 곱셈의 예

        ```java
        public void testMultiplication() {
        	Dollar five = new Dollar(5);
        	five.times(2);
        	assertEquals(10, five.amount);
        }
        ```

- 위의 여러 문제들은 적어놓고 건너뛰고, 작은 단계로 시작하자.

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - **$5 * 2 = $10**
    - amount 를 private 로 만들기
    - Dollar 부작용(side effect)?
    - Money 반올림?
    ```

- 실행은 안되더라도 컴파일은 되게 하자.(스텁 구현) 현재 컴파일 에러 4개

    ```java
    - Dollar 클래스가 없음
    - 생성자가 없음
    - times(int) 메서드가 없음
    - amount 필드가 없음
    ```

    - 한 번에 하나씩 정복해보자: 항상 작업의 진척도를 알려줄 수 있는 수치적인 척도를 찾기 위해 노력하자.
        - Dollar 클래스를 정의하자. 생성자를 만들자.

            ```java
            public class Dollar {
                public Dollar(int amount) {
                }
            }
            ```

        - 컴파일용 times() 최소한의 구현을 하자.

            ```java
            public void times(int multiplier) {
            }
            ```

        - amount 필드를 추가하자. `public int amount;`
- 코드를 조금 수정해서 테스트가 성공되게 하자.

    ```java
    public class Dollar {
        public int amount = 10;

        public Dollar(int amount) {
        }

        public void times(int multiplier) {
        }
    }
    ```

    - 10을 `5 * 2`로 변경해보자. 이제 5와 2가 테스트 코드와 객체 두 곳에 존재한다. 중복을 제거해야 한다.
    - 한 번에 제거할 수 있는 방법은 없다. 하지만 객체의 초기화 단계에 있는 설정 코드를 times() 안으로 옮겨보면 어떨까?

        ```java
        public class Dollar {
            public int amount;

            public Dollar(int amount) {
            }

            public void times(int multiplier) {
                amount = 5 * 2;
            }
        }
        ```

    - 테스트 코드와 작업 코드 사이의 중복을 없애보자.

        ```java
        public class Dollar {
            public int amount;

            public Dollar(int amount) {
                this.amount = amount;
            }

            public void times(int multiplier) {
                amount *= multiplier;
            }
        }
        ```

- 첫 번째 테스트 완료

    ```java
    class DollarTest {

        @Test
        void testMultiplication() {
            Dollar five = new Dollar(5);
            five.times(2);
            assertEquals(10, five.amount);
        }
    }
    ```

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    - Dollar 부작용(side effect)?
    - Money 반올림?
    ```

→ TDD 의 핵심은 이런 작은 단계를 밟아야 한다는 것이 아니라, 이런 작은 단계를 밟을 능력을 갖추어야 한다는 것이다.

### 요약

---

- 지금까지 한 작업 검토
    - 우리가 알고있는 작업해야 할 테스트 목록을 만들었다.
    - 오퍼레이션이 외부에서 어떻게 보이길 원하는지 말해주는 이야기를 코드로 표현했다.
    - JUnit 에 대한 상세한 사항들은 잠시 무시하기로 했다.
    - 스텁 구현을 통해 테스트를 컴파일했다.
    - 끔찍한 죄악을 범하여 테스트를 통과시켰다.
    - 돌아가는 코드에서 상수를 변수로 변경하여 점진적으로 일반화했다.
    - 새로운 할일들을 한번에 처리하는 대신 할일 목록에 추가하고 넘어갔다.


## 2장. 타락한 객체

- 일반적인 TDD 주기
    - 1. 테스트를 작성한다.
        - 마음속에 있는 오퍼레이션이 코드에 어떤 식으로 나타나길 원하는지 생각해보라. 이야기를 써내려가는 것이다.
        - 원하는 인터페이스를 개발하라. 올바른 답을 얻기 위해 필요한 이야기의 모든 요소를 포함시켜라.
    - 2. 실행 가능하게 만든다.
        - 다른 무엇보다도 중요한 것은 빨리 초록 막대를 보는 것이다. 깔끔하고 단순한 해법이 명백히 보인다면 그것을 입력하라.
        - 만약 깔끔하고 단순한 해법이 있지만 구현하는 데 몇 분 정도 걸릴 것 같으면 일단 적어놓은 뒤에 원래 문제(초록 막대를 보는 것)로 돌아오자.
    - 3. 올바르게 만든다.
        - 이제 시스템이 작동하므로 직전에 저질렀던 죄악을 수습하자.
        - 좁고 올곧은 소프트웨어 정의의 길로 되돌아와서 중복을 제거하고 초록 막대로 되돌리자.
- 우리의 목적은 작동하는 깔끔한 코드를 얻는 것이다.
    - 전체 문제 중에서 '작동하는'에 해당하는 부분을 먼저 해결하자.
    - 그러고 나서 '깔끔한 코드' 부분을 해결하자.
    - 아키텍처 주도 개발과 정반대다.
- 위에서 연산을 수행한 후 해당 Dollar 의 값이 바뀌는 부작용을 수정해보자.

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    - **Dollar 부작용?**
    - Money 반올림?
    ```

- 작은 테스트를 하나 추가한다.

    ```java
    @Test
    void testMultiplication_2() {
        Dollar five = new Dollar(5);
        five.times(2);
        assertEquals(10, five.amount);

        five.times(3);
        assertEquals(15, five.amount);
    }
    ```

    - 부작용을 인지하고, times()에서 새로운 객체를 반환하게 만들면 원래 5달러의 값은 변하지 않을 것이므로 인터페이스, 테스트를 수정하자.

        ```java
        @Test
        void testMultiplication_2() {
            Dollar five = new Dollar(5);
            Dollar product = five.times(2);
            assertEquals(10, product.amount);

            product = five.times(3);
            assertEquals(15, product.amount);
        }
        ```

- 스텁 구현: 컴파일 되게 하자. → 명백한 구현 사용하기

    ```java
    public class Dollar {
        public int amount;

        public Dollar(int amount) {
            this.amount = amount;
        }

        public Dollar times(int multiplier) {
            amount *= multiplier;
            return null;
        }
    }
    ```

- 코드를 조금 수정해서 테스트를 성공시키자.
    - 새 Dollar 를 반환시키자.

        ```java
        public class Dollar {
            public int amount;

            public Dollar(int amount) {
                this.amount = amount;
            }

            public Dollar times(int multiplier) {
                return new Dollar(amount * multiplier);
            }
        }
        ```

- 두 번째 테스트 완료

    ```java
    class DollarTest {

        @Test
        @Disabled
        void testMultiplication_1() {
            Dollar five = new Dollar(5);
            five.times(2);
            assertEquals(10, five.amount);
        }

        @Test
        void testMultiplication_2() {
            Dollar five = new Dollar(5);
            Dollar product = five.times(2);
            assertEquals(10, product.amount);

            product = five.times(3);
            assertEquals(15, product.amount);
        }
    }
    ```

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    ~~-~~ ~~Dollar 부작용?~~
    - Money 반올림?
    ```

- 최대한 빨리 초록색을 보기 위한 전략
    1. 가짜로 구현하기: 상수를 반환하게 만들고 진짜 코드를 얻을 때까지 단계적으로 상수를 변수로 바꾸어간다.
    2. 명백한 구현 사용하기: 실제 구현을 입력한다.
    3. 삼각측량법

### 요약

---

- 지금까지 배운 것 검토
    - 설계상의 결함(Dollar 부작용)을 그 결함으로 인해 실패하는 테스트로 변환했다.
    - 스텁 구현으로 빠르게 컴파일을 통과하도록 만들었다.
    - 올바르다고 생각하는 코드를 입력하여 테스트를 통과했다.


## 3장. 모두를 위한 평등

- 지금의 Dollar 객체: 값 객체 패턴
    - 객체의 인스턴스 변수가 생성자를 통해서 일단 설정된 후에는 결코 변하지 않는다.
    - 모든 연산은 새 객체를 반환해야 한다.
    - 값 객체는 `equals()`를 구현해야 한다.
        - 다른 $5 와 이 $5 객체는 같은 값으로 여겨져야 하기 때문이다.
    - 객체를 해시 테이블의 키로 쓸 생각이면, `equals()`를 구현할 때 `hashCode()`도 같이 구현해야 한다.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - **equals()**
    - hashCode()
    ```

- 작은 테스트를 하나 추가한다.

    ```java
    @Test
    void testEquality_3() {
        assertTrue(new Dollar(5).equals(new Dollar(5)));
    }
    ```

- 코드를 조금 수정해서 테스트를 성공시키자.

    ```java
    public class Dollar {
        public int amount;

        public Dollar(int amount) {
            this.amount = amount;
        }

        public Dollar times(int multiplier) {
            return new Dollar(amount * multiplier);
        }

        @Override
        public boolean equals(Object o) {
            return true;
        }
    }
    ```

- 곧장 리팩토링하는 대신, 삼각측량을 하기 위해 두 번째 테스트를 추가한다.
    - 두 번째 예가 좀더 일반적인 해를 필요로 할 때, 오로지 그때만 비로소 일반화한다.

    ```java
    @Test
    void testEquality_3() {
        assertTrue(new Dollar(5).equals(new Dollar(5)));
        assertFalse(new Dollar(5).equals(new Dollar(6)));
    }
    ```

- 코드를 조금 수정해서 테스트를 성공시키자.
    - 동치성(equality)을 일반화 해보자.

    ```java
    public class Dollar {
        public int amount;

        public Dollar(int amount) {
            this.amount = amount;
        }

        public Dollar times(int multiplier) {
            return new Dollar(amount * multiplier);
        }

        @Override
        public boolean equals(Object o) {
            Dollar dollar = (Dollar) o;
            return amount == dollar.amount;
        }
    }
    ```

- 세 번째 테스트 완료

    ```java
    class DollarTest {

        @Test
        @Disabled
        void testMultiplication_1() {
            Dollar five = new Dollar(5);
            five.times(2);
            assertEquals(10, five.amount);
        }

        @Test
        void testMultiplication_2() {
            Dollar five = new Dollar(5);
            Dollar product = five.times(2);
            assertEquals(10, product.amount);

            product = five.times(3);
            assertEquals(15, product.amount);
        }

        @Test
        void testEquality_3() {
            assertTrue(new Dollar(5).equals(new Dollar(5)));
            assertFalse(new Dollar(5).equals(new Dollar(6)));
        }
    }
    ```

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    ```

- 켄트 백 생각으론, 삼각측량은 조금 이상한 면이 있다.
    - 코드와 테스트 사이의 중복을 제거하고 일반적인 해법을 구할 방법이 보이면 그냥 그 방법대로 구현한다.
    - 한번에 끝낼 수 있는 일을 두고 또다른 테스트를 만드는 피곤함.
    - 설계를 어떻게 할지 떠오르지 않을 때면, 삼각측량은 문제를 조금 다른 방향에서 생각해볼 기회를 제공한다.
        - 지금 설계하는 프로그램이 어떤 변화 가능성을 지원해야 하는가?

### 요약

---

- 위의 내용들을 검토
    - 우리의 디자인 패턴(값 객체)이 하나의 또 다른 오퍼레이션을 암시한다는 걸 알아챘다.
    - 해당 오퍼레이션을 테스트했다.
    - 해당 오퍼레이션을 간단히 구현했다.
    - 곧장 리팩토링하는 대신 테스트를 조금 더 했다.
    - 두 경우를 모두 수용할 수 있도록 리팩토링했다.


## 4장. 프라이버시

- 일반적인 동일성 비교 문제가 있다. 널 값이나 다른 객체들과 비교할 경우.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - amount 를 private 로 만들기
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    ```

- `equals()`가 완성되었으니, 보다 정확한 테스트를 추가하자.

    ```java
    @Test
    void testMultiplication_4() {
        Dollar five = new Dollar(5);
        assertEquals(new Dollar(10), five.times(2));
        assertEquals(new Dollar(15), five.times(3));
    }
    ```

    - 위험한 상황을 만들었다. → TDD 를 하면서 적극적으로 관리해야 할 위험 요소
        - 동치성 테스트가 검증 실패하면, 곱하기 테스트 역시 검증을 실패하게 된다.
        - 우리는 모든 것을 2번 말함으로써(코드, 테스트) 자신감을 가지고 전진할 수 있을 만큼만 결함의 정도를 낮추기를 희망할 뿐이다.
        - 때때로 추론이 맞지 않아서 결함이 손가락 사이로 빠져나가는 수가 있다.
        - 그럴때면 테스트를 어떻게 작성해야 했는지에 대한 교훈을 얻고 다시 앞으로 나아간다.
- 네 번째 테스트 완료: 이로써 amount 를 `private`로 만들 수 있게 되었다.

    ```java
    public class Dollar {
        private int amount;

        public Dollar(int amount) {
            this.amount = amount;
        }

        public Dollar times(int multiplier) {
            return new Dollar(amount * multiplier);
        }

        @Override
        public boolean equals(Object o) {
            Dollar dollar = (Dollar) o;
            return amount == dollar.amount;
        }
    }
    ```

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    ```

### 요약

---

- 지금까지 배운 것 검토
    - 오직 테스트를 향상시키기 위해서만 개발된 기능을 사용했다.
    - 두 테스트가 동시에 실패하면 망한다는 점을 인식했다.
    - 위험 요소가 있음에도 계속 진행했다.
    - 테스트와 코드 사이의 결합도를 낮추기 위해, 테스트하는 객체의 새 기능(`assertEquals`)을 사용했다.


## 5장. 솔직히 말하자면

- 첫 번째 테스트에 접근하자니 너무 큰 발걸음인 것 같다.
    - 우선은 Dollar 객체와 비슷하지만 달러 대신 프랑(Franc)을 표현할 수 있는 객체가 필요할 것 같다.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - **5CHF * 2 = 10CHF**
    ```

- Dollar 테스트를 복사한 후 Franc 테스트로 추가해보자.

    ```java
    @Test
    void testMultiplication_5() {
        Franc five = new Franc(5);
        assertEquals(new Franc(10), five.times(2));
        assertEquals(new Franc(15), five.times(3));
    }
    ```

- 코드를 조금 수정(Dollar 객체를 복사하여 Franc 객체를 추가)해서 테스트를 성공시키자.

    ```java
    public class Franc {
        private int amount;

        public Franc(int amount) {
            this.amount = amount;
        }

        public Franc times(int multiplier) {
            return new Franc(amount * multiplier);
        }

        @Override
        public boolean equals(Object o) {
            Franc franc = (Franc) o;
            return amount == franc.amount;
        }
    }
    ```

    - 코드를 실행시키기까지의 단계가 짧았기 때문에 '컴파일되게 하기' 단계도 넘어갈 수 있었다.
    - 중복이 엄청나게 많기 때문에 다음 테스트를 작성하기 전에 이것들을 제거해야 한다.
    - equals()를 일반화하는 것부터 시작하자.
- 다섯 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - 공용 equals
    - 공용 times
    ```

- 다시 보는 테스트 주기
    1. 테스트 작성
    2. 컴파일되게 하기
    3. 실패하는지 확인하기 위해 실행
    4. 실행하게 만듦
    5. 중복 제거

    → 처음 네 단계는 빨리 진행해야 한다.

### 요약

---

- 검토해보면
    - 큰 테스트를 공략할 수 없다. 그래서 진전을 나타낼 수 있는 자그마한 테스트를 만들었다.
    - 뻔뻔스럽게도 중복을 만들고 조금 고쳐서 테스트를 작성했다.
    - 설상가상으로 모델 코드까지 복사하고 수정해서 테스트를 통과했다.
    - 중복이 사라지기 전에는 집에 가지 않겠다고 약속했다.


## 6장. 돌아온 '모두를 위한 평등'

- 테스트를 빨리 통과하기 위해 코드를 복붙하는 엄청난 죄를 저질렀다.
    - 청소할 방법 한 가지는 우리가 만든 클래스 중 하나가 다른 클래스를 상속받게 하는 것이다.
    - Dollar → Franc 는 아닌 것 같고, Money 라는 공통 상위 클래스를 만들어보자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - **공용 equals**
    - 공용 times
    ```

- 공통 상위 클래스 Money 가 공통의 equals 코드를 갖게 해보자.
    - 간단한 것부터 시작해보자. Money 클래스 생성!

        ```java
        public class Money {
        }
        ```

    - Dollar 가 Money 를 상속하게 해보자. 테스트는 깨지지 않는다.

        ```java
        public class Dollar extends Money {
            private int amount;

            public Dollar(int amount) {
                this.amount = amount;
            }

            public Dollar times(int multiplier) {
                return new Dollar(amount * multiplier);
            }

            @Override
            public boolean equals(Object o) {
                Dollar dollar = (Dollar) o;
                return amount == dollar.amount;
            }
        }
        ```

    - Dollar 의 amount 인스턴스 변수를 Money 로 옮기자.
        - 하위 클래스에서도 변수를 볼 수 있도록 가시성을 private 에서 protected 로 변경.

            ```java
            public class Money {
                protected int amount;
            }
            ```

        - `equals()`의 임시 변수 선언 부분을 변경하자.

            ```java
            public class Dollar extends Money {
                public Dollar(int amount) {
                    this.amount = amount;
                }

                public Dollar times(int multiplier) {
                    return new Dollar(amount * multiplier);
                }

                @Override
                public boolean equals(Object o) {
                    Money money = (Money) o;
                    return amount == money.amount;
                }
            }
            ```

    - `equals()`를 Dollar 에서 Money 로 옮기자.
        - 그 후 Franc 에서도 지우자. → 리팩토링 전에 테스트 작성을 하자(Dollar 테스트 복붙 + 중복 탄생)

            ```java
            public class Money {
                protected int amount;

                @Override
                public boolean equals(Object o) {
                    Money money = (Money) o;
                    return amount == money.amount;
                }
            }
            ```

- 여섯 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - Franc 와 Dollar 비교하기
    ```

- 적절한 테스트를 갖지 못한 코드에서 TDD 를 해야하는 경우가 종종 있다.

### 요약

---

- 지금까지 한 작업 검토
    - 공통된 코드를 첫 번째 클래스(Dollar)에서 상위 클래스(Money)로 단계적으로 옮겼다.
    - 두 번째 클래스(Franc)도 Money 의 하위 클래스로 만들었다.
    - 불필요한 구현을 제거하기 전에 두 equals() 구현을 일치시켰다.


## 7. 사과와 오렌지

- Franc 와 Dollar 를 비교하면 어떻게 될까?
    - 테스트를 작성해보자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - **Franc 와 Dollar 비교하기**
    ```

- 작은 테스트를 하나 추가한다.

    ```java
    @Test
    void testEquality_7() {
        assertFalse(new Franc(5).equals(new Dollar(5)));
    }
    ```

- 코드를 조금 수정해서(Dollar 와 Franc 클래스 비교) 테스트를 성공시키자.

    ```java
    public class Money {
        protected int amount;

        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && getClass().equals(money.getClass());
        }
    }
    ```

    - 모델 코드에서 클래스를 이런식으로 사용하는 것은 좀 지저분해 보인다.
    - 자바 객체의 용어를 사용하는 것 보다 재정 분야에 맞는 용어를 사용하자.
        - 하지만 현재는 통화(currency) 개념 같은 게 없고, 통화 개념을 도입할 충분한 이유가 없으므로 잠시 이대로 둔다.
- 일곱 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - ~~Franc 와 Dollar 비교하기~~
    - 통화?
    ```

### 요약

---

- 이번 장 성과 검토
    - 우릴 괴롭히던 결함을 끄집어내서 테스트에 담아놨다.
    - 완벽하진 않지만 그럭저럭 봐줄만한 방법(`getClass()`)으로 테스트를 통과하게 만들었다.
    - 더 많은 동기가 있기 전에는 더 많은 설계를 도입하지 않기로 했다.


## 8장. 객체 만들기

- Money 의 두 하위 클래스는 그다지 많은 일을 하는 것 같지 않으므로 아예 제거해버리고 싶다.
    - 그러나 한번에 그렇게 큰 단계를 밟는 것은 TDD 를 효과적으로 보여주기에 적절하지 않다.
    - 하위 클래스에 대한 직접적인 참조를 줄여 하위 클래스 제거에 한 발짝 다가가자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - **Dollar/Franc 중복**
    - ~~공용 equals~~
    - 공용 times
    - ~~Franc 와 Dollar 비교하기~~
    - 통화?
    ```

- 작은 단계 시작
    - times() 메소드가 Money 를 반환하게 만들어 Dollar, Franc 를 더 비슷하게 만들어보자.

        ```java
        public Money times(int multiplier) {
            return new Franc(amount * multiplier);
        }
        ```

    - Dollar 에 대한 참조를 없애길 바라며 테스트의 선언부를 수정한다.

        ```java
        @Test
        void testMultiplication_8() {
            Dollar five = Money.dollar(5);
            assertEquals(Money.dollar(10), five.times(2));
            assertEquals(Money.dollar(15), five.times(3));
        }

        @Test
        void testEquality_8() {
            assertTrue(Money.dollar(5).equals(Money.dollar(5)));
            assertFalse(Money.dollar(5).equals(Money.dollar(6)));
            assertTrue(new Franc(5).equals(new Franc(5)));
            assertFalse(new Franc(5).equals(new Franc(6)));
            assertFalse(new Franc(5).equals(Money.dollar(5)));
        }
        ```

        - 하위 클래스의 존재를 테스트에서 분리(decoupling)함으로써 어떤 모델 코드에도 영향을 주지 않고 상속 구조를 마음대로 변경할 수 있게 되었다.
- 스텁 구현: 컴파일 되게 하자.

    ```java
    public abstract class Money {
        protected int amount;

        public static Money dollar(int amount) {
            return new Dollar(amount);
        }

        public abstract Money times(int multiplier);

        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && getClass().equals(money.getClass());
        }
    }
    ```

- 작은 테스트(Franc 도 작업)를 하나 추가한다.

    ```java
    @Test
    void testFrancMultiplication_8() {
        Money five = Money.franc(5);
        assertEquals(Money.franc(10), five.times(2));
        assertEquals(Money.franc(15), five.times(3));
    }
    ```

- 여덟 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - ~~Franc 와 Dollar 비교하기~~
    - 통화?
    - testFrancMultiplication 을 지워야 할까?
    ```

### 요약

---

- 검토
    - 동일한 메서드(times)의 두 변이형 메서드 서명부를 통일시킴으로써 중복 제거를 향해 한 단계 더 전진했다.
    - 최소한 메서드 선언부만이라도 공통 상위 클래스(superclass)로 옮겼다.
    - 팩토리 메서드를 도입하여 테스트 코드에서 구체 하위 클래스의 존재 사실을 분리해냈다.
    - 하위 클래스가 사라지면 몇몇 테스트는 불필요한 여분의 것이 된다는 것을 인식했다. 하지만 일단 그냥 뒀다.


## 9장. 우리가 사는 시간

- 어떻게 해야 불필요한 하위 클래스를 제거하는 데 도움이 될까?
    - 통화 개념을 도입해보자. → 통화개념을 어떻게 구현하길 원하는가? (X)
    - 통화 개념을 어떻게 테스트하길 원하는가?
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - ~~Franc 와 Dollar 비교하기~~
    - **통화?**
    - testFrancMultiplication 을 지워야 할까?
    ```

- 작은 테스트를 하나 추가한다.

    ```java
    @Test
    void testCurrency_9() {
        assertEquals("USD", Money.dollar(1).currency());
        assertEquals("CHF", Money.franc(1).currency());
    }
    ```

- 스텁 구현: 컴파일 되게 하자.

    ```java
    public abstract class Money {
        protected int amount;

        public static Money dollar(int amount) {
            return new Dollar(amount);
        }

        public static Money franc(int amount) {
            return new Franc(amount);
        }

        public abstract Money times(int multiplier);

        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && getClass().equals(money.getClass());
        }

        public abstract String currency();
    }
    ```

    ```java
    public class Dollar extends Money {
        public Dollar(int amount) {
            this.amount = amount;
        }

        public Money times(int multiplier) {
            return new Dollar(amount * multiplier);
        }

        @Override
        public String currency() {
            return "USD";
        }
    }
    ```

- 리팩토링 단계

    ```java
    public class Dollar extends Money {
        private String currency;

        public Dollar(int amount) {
            this.amount = amount;
            currency = "USD";
        }

        public Money times(int multiplier) {
            return new Dollar(amount * multiplier);
        }

        @Override
        public String currency() {
            return currency;
        }
    }
    ```

    - 두 currency() 메소드를 비슷하게 만들었다. 또 위로 올릴 수 있게 되었다.

        ```java
        public abstract class Money {
            protected int amount;
            protected String currency;

            public static Money dollar(int amount) {
                return new Dollar(amount);
            }

            public static Money franc(int amount) {
                return new Franc(amount);
            }

            public abstract Money times(int multiplier);

            @Override
            public boolean equals(Object o) {
                Money money = (Money) o;
                return amount == money.amount
                        && getClass().equals(money.getClass());
            }

            public String currency() {
                return currency;
            }
        }
        ```

    - 문자열 'USD', 'CHF' 를 정적 팩토리 메서드로 옮기면 두 생성자가 동일해져서, 또 공통 구현을 만들 수 있을 것이다.
        - 리팩토링 진행 전에 times()를 정리하자.

            ```java
            public Money times(int multiplier) {
                return Money.franc(amount * multiplier);
            }
            ```

        - 생성자를 비슷하게 만들자.

            ```java
            public Dollar(int amount, String currency) {
                this.amount = amount;
                currency = currency;
            }
            ```

        - 비슷해진 생성자를 위로 올리자.

            ```java
            public abstract class Money {
                protected int amount;
                protected String currency;

                public static Money dollar(int amount) {
                    return new Dollar(amount, "USD");
                }

                public static Money franc(int amount) {
                    return new Franc(amount, "CHF");
                }

                public Money(int amount, String currency) {
                    this.amount = amount;
                    this.currency = currency;
                }

                public abstract Money times(int multiplier);

                @Override
                public boolean equals(Object o) {
                    Money money = (Money) o;
                    return amount == money.amount
                            && getClass().equals(money.getClass());
                }

                public String currency() {
                    return currency;
                }
            }
            ```

            ```java
            public class Franc extends Money {
                public Franc(int amount, String currency) {
                    super(amount, currency);
                }

                public Money times(int multiplier) {
                    return Money.franc(amount * multiplier);
                }
            }
            ```

            ```java
            public class Dollar extends Money {
                public Dollar(int amount, String currency) {
                    super(amount, currency);
                }

                public Money times(int multiplier) {
                    return Money.dollar(amount * multiplier);
                }
            }
            ```

- 아홉 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - 공용 times
    - ~~Franc 와 Dollar 비교하기~~
    - ~~통화?~~
    - testFrancMultiplication 을 지워야 할까?
    ```

### 요약

---

- 지금까지 한 것 검토
    - 큰 설계 아이디어를 다루다가 조금 곤경에 빠졌다. 그래서 좀 전에 주목했던 더 작은 작업을 수행했다.
    - 다른 부분들을 호출자(팩토리 메서드)로 옮김으로써 두 생성자를 일치시켰다.
    - times()가 팩토리 메서드를 사용하도록 만들기 위해 리팩토링을 잠시 중단했다.
    - 비슷한 리팩토링(Dollar 에 했던 일을 Franc 에도 적용)을 한번의 큰 단계로 처리했다.
    - 동일한 생성자들을 상위 클래스로 올렸다.


## 10장. 흥미로운 시간

- Money 를 나타내기 위한 단 하나의 클래스만을 갖도록 times()를 동일하게 만들어보자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - **공용 times**
    - ~~Franc 와 Dollar 비교하기~~
    - ~~통화?~~
    - testFrancMultiplication 제거
    ```

- times()를 완전히 동일하게 만들도록 전진하기 위해 물러서자
    - 팩토리 메서드를 인라인 시키자.

        ```java
        public class Franc extends Money {
            public Franc(int amount, String currency) {
                super(amount, currency);
            }

            public Money times(int multiplier) {
                return new Franc(amount * multiplier, "CHF");
            }
        }
        ```

        ```java
        public class Dollar extends Money {
            public Dollar(int amount, String currency) {
                super(amount, currency);
            }

            public Money times(int multiplier) {
                return new Dollar(amount * multiplier, "USD");
            }
        }
        ```

    - Franc 에서는 인스턴스 변수 currency가 항상 'CHF'이므로, 또 변경하자.

        ```java
        public class Franc extends Money {
            public Franc(int amount, String currency) {
                super(amount, currency);
            }

            public Money times(int multiplier) {
                return new Franc(amount * multiplier, currency);
            }
        }
        ```

    - times()를 완전히 똑같이 만들어보자. Money 를 구체 클래스로 바꾼다.

        ```java
        public Money times(int multiplier) {
            return new Money(amount * multiplier, currency);
        }
        ```

        - 테스트코드를 돌려보면 에러가 난다. 다음과 같은 메시지가 찍힌다.

            ```
            expected:<Money.Franc@31aebf> but was:<Money.Money@478a43>
            ```

    - 도움이 되는 메시지는 아니므로, Money 에 `toString()`을 정의하자.

        ```java
        @Override
        public String toString() {
            return amount + " " + currency;
        }
        ```

        - 테스트 없이 코드를 작성했다. 이제 나타나는 메시지는 다음과 같다.

        ```
        expected: money.domain.Franc@a45a63b<10 CHF> but was: money.domain.Money@7a72da47<10 CHF>
        ```

        - 문제는 `equals()` 구현에 있다.
- 작은 테스트를 하나 추가한다.
    - 테스트를 통과하지 못하므로, 변경했던 부분을 되돌리고 `equals()`용 테스트 코드 추가

    ```java
    @Test
    void testDifferentClassEquality_10() {
        assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
    }
    ```

    - 변경했던 times()를 되돌리고 다시 초록 막대 상태로 돌아간다.
- 조금 수정해서 테스트가 성공하는 것을 확인한다.
    - Money 의 `equals()`수정

        ```java
        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && currency().equals(money.currency());
        }
        ```

    - 이후 times()가 다시 동일하도록 변경한다.
- 열 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - Dollar/Franc 중복
    - ~~공용 equals~~
    - ~~공용 times~~
    - ~~Franc 와 Dollar 비교하기~~
    - ~~통화?~~
    - testFrancMultiplication 제거
    - testDifferentClassEquality 제거
    ```

- 테스트코드 없이 `toString()` 작성한 이유
    - 화면에 나타나는 결과를 보려던 참이다.
    - `toString()`은 디버그 출력에만 쓰이기 때문에 이게 잘못 구현됨으로 인해 얻게 될 리스크가 적다.
    - 이미 빨간 막대 상태인데, 이 상태에서는 새로운 테스트를 작성하지 않는게 좋을 것 같다.

### 요약

---

- 지금까지 한 일 검토
    - 두 times()를 일치시키기 위해 그 메서드들이 호출하는 다른 메서드들을 인라인시킨 후 상수를 변수로 바꿔주었다.
    - 단지 디버깅을 위해 테스트 없이 `toString()`을 작성했다.
    - Franc 대신 Money 를 반환하는 변경을 시도한 뒤 그것이 잘 작동할지를 테스트가 말하도록 했다.
    - 실험해본 걸 뒤로 물리고 또 다른 테스트를 작성했다. 테스트를 작동했더니 실험도 제대로 작동했다.


## 11장. 모든 악의 근원

- 이제 생성자만 있는 하위 클래스를 제거하자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - **Dollar/Franc 중복**
    - ~~공용 equals~~
    - ~~공용 times~~
    - ~~Franc 와 Dollar 비교하기~~
    - ~~통화?~~
    - testFrancMultiplication 제거
    - testDifferentClassEquality 제거
    ```

- times()를 공통 상위 클래스로 올리고, 불필요한 테스트코드를 정리하자.
    - times()를 공통 상위 클래스로 올리고, 정적 팩토리 메서드가 Money 를 리턴하게 비슷하게 바꾸자.

        ```java
        public class Money {
            protected int amount;
            protected String currency;

            public static Money dollar(int amount) {
                return new Money(amount, "USD");
            }

            public static Money franc(int amount) {
                return new Money(amount, "CHF");
            }

            public Money(int amount, String currency) {
                this.amount = amount;
                this.currency = currency;
            }

            public Money times(int multiplier) {
                return new Money(amount * multiplier, currency);
            }

            @Override
            public boolean equals(Object o) {
                Money money = (Money) o;
                return amount == money.amount
                        && currency().equals(money.currency());
            }

            public String currency() {
                return currency;
            }

            @Override
            public String toString() {
                return amount + " " + currency;
            }
        }
        ```

    - 이제 Dollar, Franc 에 대한 참조는 없어졌다. 과한 테스트코드를 좀 정리하자.

        ```java
        @Test
        void testEquality_8() {
            assertTrue(Money.dollar(5).equals(Money.dollar(5)));
            assertFalse(Money.dollar(5).equals(Money.dollar(6)));
            assertFalse(Money.franc(5).equals(Money.dollar(5)));
        }
        ```

- (Dollar, Franc 제거 시) 컴파일이 깨지는, testDifferentClassEquality 및 하위 클래스 제거
- 여러 클래스가 존재할 때만 의미있는 testFrancMultiplication 테스트도 삭제하자.
- 열한 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 * 2 = $10~~
    - ~~amount 를 private 로 만들기~~
    - ~~Dollar 부작용?~~
    - Money 반올림?
    - ~~equals()~~
    - hashCode()
    - Equal null
    - Equal object
    - ~~5CHF * 2 = 10CHF~~
    - ~~Dollar/Franc 중복~~
    - ~~공용 equals~~
    - ~~공용 times~~
    - ~~Franc 와 Dollar 비교하기~~
    - ~~통화?~~
    - ~~testFrancMultiplication 제거~~
    - ~~testDifferentClassEquality 제거~~
    ```

### 요약

---

- 지금까지 한 작업 검토
    - 하위 클래스의 속을 들어내는 걸 완료하고, 하위 클래스를 삭제했다.
    - 기존 소스 구조에서는 필요했지만 새로운 구조에서는 필요없게 된 테스트를 제거했다.


## 12장. 드디어, 더하기

- 남은 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - Money 반올림?
    - hashCode()
    - Equal null
    - Equal object
    ```

- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - **$5 + $5 = $10**
    ```

- 할일 목록이 지저분해졌을 때,
    - 해결되지 않은 항목들을 새 목록에 옮긴다.
    - 저자는 손으로 옮겨 적는걸 즐긴다.
    - 자잘한 항목이 많으면 옮기기보다 그냥 처리해버린다.

- 작은 테스트를 하나 추가한다. (주기 1)

    ```java
    @Test
    void testSimpleAddition_11() {
        Money sum = Money.dollar(5).plus(Money.dollar(5));
        assertEquals(Money.dollar(10), sum);
    }
    ```

- 모든 테스트를 실행해서 테스트가 실패하는 것을 확인한다. (주기 2)
- 조금 수정한다. 컴파일 에러 수정 (주기 3)
    - Money.dollar(10) 를 반환하는 식으로 가짜 구현을 할 수도 있지만, 어떻게 구현해야할지 명확하므로 다음과 같이 plus 를 추가한다.

    ```java
    public class Money {
        protected int amount;
        protected String currency;

        public static Money dollar(int amount) {
            return new Money(amount, "USD");
        }

        public static Money franc(int amount) {
            return new Money(amount, "CHF");
        }

        public Money(int amount, String currency) {
            this.amount = amount;
            this.currency = currency;
        }

        public Money times(int multiplier) {
            return new Money(amount * multiplier, currency);
        }

        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && currency().equals(money.currency());
        }

        public String currency() {
            return currency;
        }

        @Override
        public String toString() {
            return amount + " " + currency;
        }

        public Money plus(Money addend) {
            return new Money(amount + addend.amount, currency);
        }
    }
    ```

- 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다. (주기 4)

- 가지고 있는 객체가 우리가 원하는 동작을 하지 않을 경우, 그 객체와 외부 프로토콜이 같으면서 내부 구현은 다른 새로운 객체(imposter)를 만들 수 있다.
    - Money 와 비슷하게 동작하지만 두 Money 의 합을 나타내는 객체를 만드는 것이다.
    - 1. 한 가지는 Money 의 합을 마치 지갑처럼 취급하는 것이다.
        - 금액과 통화가 다른 여러 화폐들이 들어갈 수 있다.
    - 2. 또 다른 메타포는 '(2 + 3) * 5' 같은 수식이다.
        - 이렇게 하면 Money 가 수식의 가장 단위가 된다.
        - 연산의 결과로 Expression 들이 생기는데, 그 중 하나는 Sum 이 될 것이다.
        - 연산이 완료되면, 환율을 이용해서 결과 Expression 을 단일 통화로 축약(reduced)할 수 있다.

- 2번 메타포를 테스트에 적용해본다.
    - reduced: Expression 에 환율을 적용한 것

        ```java
        @Test
        void testSimpleAddition_11() {
            ...
            assertEquals(Money.dollar(10), reduced);
        }
        ```

    - 환율이 적용되는 곳은 은행이다.

        ```java
        @Test
        void testSimpleAddition_11() {
            Money sum = Money.dollar(5).plus(Money.dollar(5));
            ...
            Money reduced = bank.reduce(sum, "USD");
            assertEquals(Money.dollar(10), reduced);
        }
        ```

- Bank 객체를 생성한다.

    ```java
    public class Bank {
        public Bank() {
        }

        public Money reduce(Money sum, String usd) {
            return null;
        }
    }
    ```

- 작은 테스트를 추가한다. 두 Money 의 합은 Expression 이어야 한다. (주기 1)

    ```java
    @Test
    void testSimpleAddition_11() {
        Money five = Money.dollar(5);
        Expression sum = five.plus(five);

        Bank bank = new Bank();
        Money reduced = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(10), reduced);
    }
    ```

- 조금 수정한다. 컴파일 에러 수정 (주기 3)

    ```java
    public interface Expression {
    }
    ```

    ```java
    public class Bank {
        public Bank() {
        }

        public Money reduce(Expression sum, String usd) {
            return null;
        }
    }
    ```

    ```java
    public class Money implements Expression {
        protected int amount;
        protected String currency;

        public static Money dollar(int amount) {
            return new Money(amount, "USD");
        }

        public static Money franc(int amount) {
            return new Money(amount, "CHF");
        }

        public Money(int amount, String currency) {
            this.amount = amount;
            this.currency = currency;
        }

        public Money times(int multiplier) {
            return new Money(amount * multiplier, currency);
        }

        @Override
        public boolean equals(Object o) {
            Money money = (Money) o;
            return amount == money.amount
                    && currency().equals(money.currency());
        }

        public String currency() {
            return currency;
        }

        @Override
        public String toString() {
            return amount + " " + currency;
        }

        public Money plus(Money addend) {
            return new Money(amount + addend.amount, currency);
        }
    }
    ```

- 모든 테스트를 실행해서 테스트가 실패하는 것을 확인한다. (주기 2)
- 조금 수정한다. 가짜 구현 추가 (주기 3)

    ```java
    public class Bank {
        public Bank() {
        }

        public Money reduce(Expression sum, String usd) {
            return Money.dollar(10);
        }
    }
    ```

- 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다. (주기 4)
    - 리팩토링할 준비가 됐다.

- 왜 reduce 메소드가 Expression 이 아닌 bank 의 책임이어야 할까?
    - Expression 은 우리가 하려고 하는 일의 핵심에 해당한다.
        - 저자는 핵심이 되는 객체가 다른 부분에 대해서 될 수 있는 한 모르도록 노력한다.
        - 그렇게 하면 핵심 객체가 가능한 오랫동안 유연할 수 있다.
        - 게다가 테스트하기 쉬울 뿐 아니라, 재활용하거나 이해하기에 모두 쉬운 상태로 남아있을 수 있다.
    - Expression 과 관련이 있는 오퍼레이션이 많을거라고 상상할 수 있다.
        - 만약에 모든 오퍼레이션을 Expression 에만 추가한다면 Expression 은 무한히 커질 것이다.

### 요약

---

- 지금까지 한 것 검토
    - 큰 테스트를 작은 테스트($5 + 10CHF 에서 $5 + $5)로 줄여서 발전을 나타낼 수 있도록 했다.
    - 우리에게 필요한 계산에 대한 가능한 메타포들을 신중히 생각해봤다.
    - 새 메타포에 기반하여 기존의 테스트를 재작성했다.
    - 테스트를 빠르게 컴파일했다.
    - 그리고 테스트를 실행했다.
    - 진짜 구현을 만들기 위해 필요한 리팩토링을 약간의 전율과 함께 기대했다.


## 13장. 진짜로 만들기

- 모든 중복을 제거하기 전까지는 $5 + $5 = $10 테스트에 완료 표시를 할 수 없다.
    - 코드 중복은 없지만 데이터 중복이 있다.
    - 가짜 구현에 있는 $10 는 테스트 코드에 있는 $5 + $5 와 같다.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - **$5 + $5 = $10**
    - $5 + $5 에서 Money 반환하기
    ```

- Money.plus() 는 그냥 Money 가 아닌, Expression(Sum) 을 반환해야 한다. 작은 테스트를 추가한다. (주기 1)

    ```java
    @Test
    void testPlusReturnsSum_13() {
        Money five = Money.dollar(5);
        Expression result = five.plus(five);
        Sum sum = (Sum) result;
        assertEquals(five, sum.augend);
        assertEquals(five, sum.addend);
    }
    ```

    - 수행하고자 하는 연산의 외부 행위가 아닌 내부 구현에 대해 너무 깊게 관여하고 있다.
- 조금 수정한다. 컴파일 에러 수정 (주기 3)
    - Sum 클래스 생성

        ```java
        public class Sum {
            public Money augend;
            public Money addend;
        }
        ```

    - ClassCastException 부분 수정
        - Money.plus()는 Money 가 아닌, Expression 을 반환한다.

            ```java
            public Expression plus(Money addend) {
            //    public Money plus(Money addend) {
            //        return new Money(amount + addend.amount, currency);
                return new Sum(this, currency);
            }
            ```

        - Expression 타입인 Sum 에 생성자를 추가한다.

            ```java
            public class Sum implements Expression {
                public Money augend;
                public Money addend;

                public Sum(Money augend, Money addend) {
                }
            }
            ```

    - Sum 의 생성자 내에서 인자에 바로 할당한다.

        ```java
        public Sum(Money augend, Money addend) {
            this.augend = augend;
            this.addend = addend;
        }
        ```

- 작은 테스트를 추가한다. (주기 1)
    - Sum 이 갖고 있는 Money 통화가 모두 동일하고, reduce 를 통해 얻어내고자 하는 통화도 같다면, 결과는 Sum 내의 Money 들 amount 를 합친 값을 갖는 Money 객체여야 한다.

    ```java
    @Test
    void testReduceSum_13() {
        Expression sum = new Sum(Money.dollar(3), Money.dollar(4));
        Bank bank = new Bank();
        Money result = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(7), result);
    }
    ```

- 모든 테스트를 실행해서 테스트가 실패하는 것을 확인한다. (주기 2)
- 조금 수정한다. (주기 3)
    - Bank.reduce() 를 구현해본다. 지저분한 코드 탄생.

        ```java
        public Money reduce(Expression source, String currency) {
            Sum sum = (Sum) source;
            int amount = sum.augend.amount + sum.addend.amount;
            return new Money(amount, currency);
        }
        ```

    - reduce 의 상세 역할은 Sum 내부로 옮긴다.
        - Bank.reduce()

            ```java
            public class Bank {
                public Bank() {
                }

                public Money reduce(Expression source, String currency) {
                    Sum sum = (Sum) source;
                    return sum.reduce(currency);
                }
            }
            ```

        - Sum.reduce()

            ```java
            public class Sum implements Expression {
                public Money augend;
                public Money addend;

                public Sum(Money augend, Money addend) {
                    this.augend = augend;
                    this.addend = addend;
                }

                public Money reduce(String currency) {
                    int amount = augend.amount + addend.amount;
                    return new Money(amount, currency);
                }
            }
            ```

- 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다. (주기 4)
- 할 일 목록에 할 일 추가: Bank.reduce()의 인자로 Money 를 넘겼을 경우 어떻게 테스트 할 것인지?

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - **$5 + $5 = $10**
    - $5 + $5 에서 Money 반환하기
    - Bank.reduce(Money)
    ```

- 작은 테스트(Bank.reduce(Money))를 추가한다. (주기 1)

    ```java
    @Test
    void testReduceMoney_13() {
        Bank bank = new Bank();
        Money result = bank.reduce(Money.dollar(1), "USD");
        assertEquals(Money.dollar(1), result);
    }
    ```

- 모든 테스트를 실행해서 테스트가 실패하는 것을 확인한다. (주기 2)
- 조금 수정한다. Bank.reduce(Money) 일 경우를 고려한다. (주기 3)

    ```java
    public Money reduce(Expression source, String currency) {
        if (source instanceof Money) return (Money) source;
        Sum sum = (Sum) source;
        return sum.reduce(currency);
    }
    ```

- 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다. (주기 4)
    - 초록 막대 상태이므로 리팩토링을 할 수 있다.

- 클래스를 명시적으로 검사하는 코드가 있을 때에는 항상 다형성을 사용하도록 바꾸는 것이 좋다.
    - 현재 Sum 은 reduce(String) 을 구현하고 있다.

        ```java
        public Money reduce(String currency) {
            int amount = augend.amount + addend.amount;
            return new Money(amount, currency);
        }
        ```

    - Money 도 구현하도록 만든다면, reduce()를 Expression 인터페이스에도 추가할 수 있게 된다.
    - 지저분한 캐스팅과 클래스 검사 코드를 제거할 수 있다.

- 조금 수정한다. Bank.reduce()의 명시적 클래스 검사 부분 변경 (주기 3)
    - Bank.reduce()

        ```java
        public Money reduce(Expression source, String currency) {
            return source.reduce(currency);
        }
        ```

    - Expression 에 reduce() 추가

        ```java
        public interface Expression {
            Money reduce(String currency);
        }
        ```

    - Money 에 reduce() 추가

        ```java
        public class Money implements Expression {
            protected int amount;
            protected String currency;

            public static Money dollar(int amount) {
                return new Money(amount, "USD");
            }

            public static Money franc(int amount) {
                return new Money(amount, "CHF");
            }

            public Money(int amount, String currency) {
                this.amount = amount;
                this.currency = currency;
            }

            public Money times(int multiplier) {
                return new Money(amount * multiplier, currency);
            }

            @Override
            public boolean equals(Object o) {
                Money money = (Money) o;
                return amount == money.amount
                        && currency().equals(money.currency());
            }

            public String currency() {
                return currency;
            }

            @Override
            public String toString() {
                return amount + " " + currency;
            }

            public Expression plus(Money addend) {
                return new Sum(this, addend);
            }

            public Money reduce(String currency) {
                return this;
            }
        }
        ```

- 할 일 목록에 할 일 추가: Bank.reduce(Expression, String)과 Expression.reduce(String) 차이점에 대한 의도를 명백히 표현할 수 있도록 하자.
    - 위치 매개 변수만으로는 두 메서드가 어떻게 다른지에 대해 코드에 명확히 담아내는 것이 쉽지 않다.

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - **$5 + $5 = $10**
    - $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - Money 에 대한 통화 변환을 수행하는 Reduce
    - Reduce(Bank, String)
    ```

- 이전에는 가짜 구현이 있을 때 진짜 구현으로 거꾸로 작업해가는 것이 명확했다.
    - 단순히 상수를 변수로 치환하는 일이었다.
    - 이번엔 순방향으로 작업해본다.
- [Bank.reduce()]() 코드가 지저분한 이유
    - 캐스팅(형변환): 이 코드는 모든 Expression 에 대해 작동해야 한다.
    - 공용(public) 필드와 그 필드들에 대한 두 단계에 걸친 레퍼런스.

### 요약

---

- 지금까지 한 작업 검토
    - 모든 중복이 제거되기 전까지는 테스트를 통과한 것으로 치지 않았다.
    - 구현하기 위해 역방향이 아닌 순방향으로 작업했다.
    - 앞으로 필요할 것으로 예상되는 객체(Sum)의 생성을 강요하기 위한 테스트를 작성했다.
    - 빠른 속도로 구현하기 시작했다.(Sum 의 생성자)
    - 일단 한 곳에 캐스팅을 이용해서 코드를 구현했다가, 테스트가 돌아가자 그 코드를 적당한 자리로 옮겼다.
    - 명시적인 클래스 검사를 제거하기 위해 다형성을 사용했다.


## 14장. 바꾸기

- 프랑을 달러로 바꿔보자.
- 할 일 목록

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - $5 + $5 = $10
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - **Money 에 대한 통화 변환을 수행하는 Reduce**
    - Reduce(Bank, String)
    ```

- 작은 테스트를 추가한다. (주기 1)

    ```java
    @Test
    void testReduceMoneyDifferentCurrency_14() {
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Money result = bank.reduce(Money.franc(2), "USD");
        assertEquals(Money.dollar(1), result);
    }
    ```

- 조금 수정한다. 컴파일 에러 수정 (주기 3)
    - 깨지는 테스트를 돌아가게 Money.reduce() 수정

        ```java
        public Money reduce(String to) {
            int rate = (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
            return new Money(amount / rate, to);
        }
        ```

- 이 코드로 인해 Money 가 환율에 대해 알게 되었다. → 환율 관련은 모두 Bank 가 처리해야 한다.
    - Expression.reduce() 인자로 Bank 를 넘겨야 할 것이다.
    - 호출하는 부분 Bank.reduce() 에서 Bank 넘기게 변경.

        ```java
        public Money reduce(Expression source, String currency) {
            return source.reduce(this, currency);
        }
        ```

- 중복을 제거하기 위해 리팩토링을 한다. (주기 5)
    - 2 라는 숫자가 테스트와 코드 두 부분에 나오는데, Bank 에서 환율표를 갖게 하자.

        ```java
        public class Bank {
            private Hashtable<Pair, Integer> rates = new Hashtable<>();

            public Bank() {
            }

            public Money reduce(Expression source, String currency) {
                return source.reduce(this, currency);
            }

            public void addRate(String from, String to, int rate) {
                rates.put(new Pair(from, to), rate);
            }

            public int rate(String from, String to) {
                if (from.equals(to)) return 1;
                return rates.get(new Pair(from, to));
            }
        }
        ```

    - 환율표의 key 가 될 Pair 객체 추가

        ```java
        public class Pair {
            private String from;
            private String to;

            Pair(String from, String to) {
                this.from = from;
                this.to = to;
            }

            @Override
            public boolean equals(Object o) {
                Pair pair = (Pair) o;
                return Objects.equals(from, pair.from) && Objects.equals(to, pair.to);
            }

            @Override
            public int hashCode() {
                return 0;
            }
        }
        ```

        - Pair 를 키로 쓸 거니까 equals()와 hashCode()를 구현해야 한다.
        - 지금은 리팩토링하는 중에 코드를 작성하는 것이기 때문에 테스트를 작성하지는 않는다.
        - 0은 최악의 해시 코드다.
            - 하지만 구현하기 쉽고 현재 빨리 진행할 수 있는 장점이 있다.
            - 해시 코드를 이대로 둔다면 해시 테이블에서 검색이 선형 검색과 비슷하게 수행될 것이다.
    - 같은 통화일 때 1이 리턴되어야 하는 발견한 내용을 테스트로 만들어두자.

        ```java
        @Test
        void testIdentityRate_14() {
            assertEquals(1, new Bank().rate("USD", "USD"));
        }
        ```

- 열네 번째 테스트 완료

    ```
    - $5 + 10CHF = $10 (환율이 2:1일 경우)
    - ~~$5 + $5 = $10~~
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    ```

- 환율표의 키로 배열을 쓸 수 있을까? → 테스트를 추가해보자
    - Array.equals()가 각각의 원소에 대한 동치성 검사를 수행하는지 테스트 추가 → 실패!

    ```java
    @Test
    void testArrayEquals_14() {
        assertEquals(new Object[]{"abc"}, new Object[]{"abc"});
    }
    ```

### 요약

---

- 지금까지 한 것 검토
    - 필요할 거라고 생각한 인자를 빠르게 추가했다.
    - 코드와 테스트 사이에 있는 데이터 중복을 끄집어냈다.
    - 자바의 오퍼레이션에 대한 가정을 검사해보기 위한 테스트(testArrayEquals)를 작성했다.
    - 별도의 테스트 없이 전용(private) 도우미(helper) 클래스를 만들었다.
    - 리팩토링하다가 실수를 했고, 그 문제를 분리하기 위해 또 하나의 테스트를 작성하면서 계속 전진해 가기로 선택했다.


## 15장. 서로 다른 통화 더하기

- 할 일 목록

    ```
    - **$5 + 10CHF = $10 (환율이 2:1일 경우)**
    - ~~$5 + $5 = $10~~
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    ```

- 작은 테스트를 추가한다. (주기 1)

    ```java
    @Test
    void testMixedAddition_15() {
        Expression fiveBucks = Money.dollar(5);
        Expression tenFrancs = Money.dollar(10);
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);

        Money result = bank.reduce(fiveBucks.plus(tenFrancs), "USD");
        assertEquals(Money.dollar(10), result);
    }
    ```

- 조금 수정한다. 컴파일 에러 수정 (주기 3)
    - Money.plus(Money)를 제대로 사용할 수 있게 수정한다.

        ```java
        @Test
        void testMixedAddition_15() {
            Money fiveBucks = Money.dollar(5);
            Money tenFrancs = Money.franc(10);
            Bank bank = new Bank();
            bank.addRate("CHF", "USD", 2);

            Money result = bank.reduce(fiveBucks.plus(tenFrancs), "USD");
            assertEquals(Money.dollar(10), result);
        }
        ```

- 조금 수정한다. (주기 3)
    - Sum.reduce()가 인자를 축약하지 않는 부분을 수정한다.

        ```java
        public class Sum implements Expression {
            public Money augend;
            public Money addend;

            public Sum(Money augend, Money addend) {
                this.augend = augend;
                this.addend = addend;
            }

            @Override
            public Money reduce(Bank bank, String currency) {
                int amount = augend.reduce(bank, currency).amount + addend.reduce(bank, currency).amount;
                return new Money(amount, currency);
            }
        }
        ```

    - Expression 이어야 하는 Money 들을 조금씩 쪼아서 없앨 수 있다.
        - Money 를 Expression으로 일반화한다.
        - Sum 의 인자들을 Expression 으로 변경하고 plus, times 를 Expression 으로 올린다.
            - Sum 의 구현을 스텁 구현으로 바꾸고, 할일 목록에 적어둔다.

            ```java
            public class Sum implements Expression {
                public Expression augend;
                public Expression addend;

                public Sum(Expression augend, Expression addend) {
                    this.augend = augend;
                    this.addend = addend;
                }

                @Override
                public Money reduce(Bank bank, String currency) {
                    int amount = augend.reduce(bank, currency).amount + addend.reduce(bank, currency).amount;
                    return new Money(amount, currency);
                }

                @Override
                public Expression times(int multiplier) {
                    return null;
                }

                @Override
                public Expression plus(Expression addend) {
                    return null;
                }
            }
            ```

        - Money 의 리턴을 Expression 으로 변경한다.

            ```java
            public class Money implements Expression {
                protected int amount;
                protected String currency;

                public static Money dollar(int amount) {
                    return new Money(amount, "USD");
                }

                public static Money franc(int amount) {
                    return new Money(amount, "CHF");
                }

                public Money(int amount, String currency) {
                    this.amount = amount;
                    this.currency = currency;
                }

                @Override
                public Expression times(int multiplier) {
                    return new Money(amount * multiplier, currency);
                }

                @Override
                public boolean equals(Object o) {
                    Money money = (Money) o;
                    return amount == money.amount
                            && currency().equals(money.currency());
                }

                public String currency() {
                    return currency;
                }

                @Override
                public String toString() {
                    return amount + " " + currency;
                }

                @Override
                public Expression plus(Expression addend) {
                    return new Sum(this, addend);
                }

                @Override
                public Money reduce(Bank bank, String to) {
                    int rate = bank.rate(currency, to);
                    return new Money(amount / rate, to);
                }
            }
            ```

        - 테스트 코드의 Money 를 Expression 으로 변경한다.

            ```java
            @Test
            void testMixedAddition_15() {
                Expression fiveBucks = Money.dollar(5);
                Expression tenFrancs = Money.franc(10);
                Bank bank = new Bank();
                bank.addRate("CHF", "USD", 2);

                Money result = bank.reduce(fiveBucks.plus(tenFrancs), "USD");
                assertEquals(Money.dollar(10), result);
            }
            ```

- 열다섯 번째 테스트 완료

    ```
    - ~~$5 + 10CHF = $10 (환율이 2:1일 경우)~~
    - ~~$5 + $5 = $10~~
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    - Sum.plus
    - Expression.times
    ```

- 처음 코드 수정이 다음으로 계속해서 퍼져나갈 수 있도록 하는 방법 2가지
    - 좁은 범위의 한정적인 테스트를 빠르게 작성한 후에 일반화하는 방법
    - 우리의 모든 실수를 컴파일러가 잡아줄거라 믿고 진행하는 방법

    → 퍼져나가는 변화를 한 번에 하나씩 제대로 고치기만 할 것이다.

### 요약

---

- 지금까지 한 작업 검토
    - 원하는 테스트를 작성하고, 한 단계에 달성할 수 있도록 뒤로 물렀다.
    - 좀더 추상적인 선언을 통해 가지에서 뿌리(애초의 테스트 케이스)로 일반화했다.
    - 변경 후(Expression fiveBucks), 그 영향을 받은 다른 부분들을 변경하기 위해 컴파일러의 지시를 따랐다.(Expression 에 plus()를 추가하기 등등)


## 16장. 드디어, 추상화

- Expression.plus 를 끝마치려면 Sum.plus()를 구현해야 한다. 그 후 Expression.times()를 구현하자.
- 할 일 목록

    ```
    - ~~$5 + 10CHF = $10 (환율이 2:1일 경우)~~
    - ~~$5 + $5 = $10~~
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    - Sum.plus
    - Expression.times
    ```

- 작은(Sum.plus()에 대한) 테스트를 추가한다. (주기 1)

    ```java
    @Test
    void testSumPlusMoney_16() {
        Expression fiveBucks = Money.dollar(5);
        Expression tenFrancs = Money.franc(10);
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);

        Expression sum = new Sum(fiveBucks, tenFrancs).plus(fiveBucks);
        Money result = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(15), result);
    }
    ```

- 조금 수정한다. (주기 3)
    - Sum 의 plus()를 Money 와 동일하게 복붙 구현한다.

        ```java
        @Override
        public Expression plus(Expression addend) {
            return new Sum(this, addend);
        }
        ```

    - 추상 클래스의 외침이 들려오는 것만 같다.
- 할 일 목록 하나 끝남

    ```
    - ~~$5 + 10CHF = $10 (환율이 2:1일 경우)~~
    - ~~$5 + $5 = $10~~
    ****- $5 + $5 에서 Money 반환하기
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    - ~~Sum.plus~~
    - **Expression.times**
    ```

- 작은(Expression.times()에 대한) 테스트를 추가한다. (주기 1)
- 조금 수정한다. (주기 3)
    - 컴파일되게 만들기 위해 선언한 Expression.times() 작업 때문에 Money.times()와 Sum.times()의 가시성을 높여줘야 한다.
        - Sum.times() 구현

            ```java
            @Override
            public Expression times(int multiplier) {
                return new Sum(augend.times(multiplier), addend.times(multiplier));
            }
            ```

- 할 일 목록 하나 끝남

    ```
    - ~~$5 + 10CHF = $10 (환율이 2:1일 경우)~~
    - ~~$5 + $5 = $10~~
    ****- **$5 + $5 에서 Money 반환하기**
    - ~~Bank.reduce(Money)~~
    - ~~Money 에 대한 통화 변환을 수행하는 Reduce~~
    - ~~Reduce(Bank, String)~~
    - ~~Sum.plus~~
    - ~~Expression.times~~
    ```

- 작은($5 + $5 에서 Money 반환에 대한) 테스트를 추가한다. (주기 1)

    ```java
    @Test
    void testPlusSameCurrencyReturnsMoney_16() {
        Expression sum = Money.dollar(1).plus(Money.dollar(1));
        assertTrue(sum instanceof Money);
    }
    ```

- 조금 수정한다. (주기 3)
    - 인자가 Money 일 경우에만, 인자의 통화를 확인하는 분명하고도 깔끔한 방법이 없다.
    - 실험은 실패했고, 테스트를 삭제하고 떠난다.

- TDD 로 구현할 때 특징
    - 테스트 코드의 줄 수와 모델 코드의 줄 수가 거의 비슷한 상태로 끝난다.
    - TDD 가 경제적이기 위해서는 매일 만들어내는 코드의 줄 수가 2배가 되거나, 동일한 기능을 구현하되 절반의 줄 수로 해내야 한다.

### 요약

---

- 한 작업들 검토
    - 미래에 코드를 읽을 다른 사람들을 염두에 둔 테스트를 작성했다.
    - TDD 와 여러분의 현재 개발 스타일을 비교해볼 수 있는 실험 방법을 제시했다.
    - 또 한 번 선언부에 대한 수정이 시스템 나머지 부분으로 번져갔고, 문제를 고치기 위해 역시 컴파일러의 조언을 따랐다.
    - 잠시 실험을 시도했는데, 제대로 되지 않아서 버렸다.


## 17. Money 회고

### 다음에 할 일은 무엇인가

---

- 지저분한 중복 정리
    - Expression 을 인터페이스 대신 클래스로 바꾼다면 공통되는 코드(Sum.plus(), Money.plus())를 담아낼 적절한 곳이 될 것이다.
    - 보통 클래스가 인터페이스로 바뀌는 일이 잦은데, 그 반대는 일반적이진 않다.
- TDD 는 완벽을 위한 노력의 일환이다.
    - 만약 시스템이 크다면, 개발자가 늘 건드리는 부분들은 절대적으로 견고해야 한다.
    - 그래야 나날이 수정할 때 안심할 수 있다.
- 작업을 끝낸 후 코드 감정(code critic) 프로그램을 실행하길 추천한다.
    - 스몰토크의 스몰린트(SmallLint), 인텔리제이의 SonarLint 같은 라이브러리.
    - 놓친 부분은 감정 프로그램이 지적할 것이다.
- 어떤 테스트들이 추가로 더 필요할까?
    - 실패해야 하는 테스트가 성공하는 경우, 이유를 찾아내야 한다.
    - 실패해야 하는 테스트가 실패하는 경우, 이미 알려진 제한사항 or TO-DO 의미로 기록해둘 수 있다.
- 이때까지 설계한 것을 검토하자.
    - 말과 개념이 서로 잘 통하는가?
    - 현재 설계로는 제거하기 힘든 중복이 있는가?

### 메타포

---

- 메타포라는건 단지 이름들을 얻어내는 데 필요한 것일 뿐? → 절대 그렇지 않다.

### JUnit 사용도

---

- 보통 1분에 한 번 정도 테스트를 실행했다.

### 프로세스

---

- TDD 의 주기
    1. 작은 테스트를 추가한다.
    2. 모든 테스트를 실행하고, 실패하는 것을 확인한다.
    3. 코드에 변화를 준다.
    4. 모든 테스트를 실행하고, 성공하는 것을 확인한다.
    5. 중복을 제거하기 위해 리팩토링 한다.

### 테스트의 질

---

- TDD 로 생기는 테스트는 다른 종류의 테스트들을 대체할 순 없다.
    - 성능 테스트
    - 스트레스 테스트
    - 사용성 테스트

- 작성한 테스트에 대해 널리 사용되는 지표 몇 가지
    - 명령문 커버리지(statement coverage)
        - 테스트의 질에 대한 충분한 평가 기준이 될 수 없음이 확실하지만, 테스트의 시작점이다.
        - TDD 는 100% 명령문 커버리지를 종교적으로 따른다.
        - JProbe 는 오직 한 메서드(Money.`toString()`)만이 테스트 케이스에 의해 검증되지 않는다고 하는데, 이는 실제 모델 코드를 위해 작성한 것이 아니라 디버깅을 위해 작성한 것이다.
    - 결함 삽입(defect insertion)
        - 테스트의 질을 평가하는 또 다른 방법
        - 코드의 의미를 바꾼 후, 테스트가 실패하는지 보는 것이다.
        - Jester 같은 툴을 사용할 수 있다.
            - 내용을 변경해도 테스트가 실패하지 않는 단 한 줄(Pair.`hashCode()`)의 코드가 있다고 보고하는데, 무조건 0을 반환하도록 한 곳이다. 결함이라고 볼 수는 없다.
    - 테스트 커버리지를 향상시키는 또다른 방법
        - 테스트 수는 그대로 두면서 프로그램 로직을 단순화하는 것.
        - 리팩토링 단계가 종종 이 효과를 가져온다.

### 최종 검토

---

- 테스트를 확실히 돌아가게 만드는 3가지 접근법: 가짜로 구현하기, 삼각측량법, 명백하게 구현하기
- 설계를 주도하기 위한 방법: 테스트와 실제 코드 사이의 중복을 제거하기
- 길이 미끄러우면 속도를 줄이고, 상황이 좋으면 속도를 높이는 식으로 테스트 사이의 간격을 조절할 수 있는 능력.


# 2부. xUnit 예시


## 18장. xUnit으로 가는 첫걸음

- 테스트 프레임워크에 대한 할 일 목록

    ```
    - **테스트 메서드 호출하기**
    - 먼저 setUp 호출하기
    - 나중에 tearDown 호출하기
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    ```

- 첫 번째 원시테스트 추가 - 작은 테스트를 추가한다. (주기 1)
    - 테스트 메서드가 호출되면 true, 그렇지 않으면 false 반환
    - 메서드가 실행되었는지를 알려주는 테스트 케이스 클래스 WasRun 추가

    ```python
    class WasRun:
        def __init__(self, name):
            self.wasRun = None
    ```

- 실행 코드 추가 - 모든 테스트를 실행하고, 실패하는 것을 확인한다. (주기 2)
    - Execute.py

        ```python
        from xUnit.WasRun import WasRun

        test = WasRun("testMethod")
        print(test.wasRun)
        test.testMethod()
        print(test.wasRun)
        ```

    - 실행 결과: testMethod 를 정의해야 한다고 알려준다.

        ```python
        None
        Traceback (most recent call last):
          File "/Users/kim-yoonhee/PycharmProjects/test-driven-development-python/xUnit/execute.py", line 5, in <module>
            test.testMethod()
        AttributeError: 'WasRun' object has no attribute 'testMethod'
        ```

- 런타임 에러 수정 testMethod 추가 - 코드에 변화를 준다. (주기 3)

    ```python
    class WasRun:
        def __init__(self, name):
            self.wasRun = None

        def testMethod(self):
            self.wasRun = 1
    ```

- 모든 테스트를 실행하고, 성공하는 것을 확인한다. (주기 4)

- 코드에 변화를 준다. (주기 3)
    - 테스트 메서드를 직접 호출하는 대신 진짜 인터페이스인 run() 사용

        ```python
        class WasRun:
            def __init__(self, name):
                self.wasRun = None

            def testMethod(self):
                self.wasRun = 1

            def run(self):
                self.testMethod()
        ```

        ```python
        from xUnit.WasRun import WasRun

        test = WasRun("testMethod")
        print(test.wasRun)
        test.run()
        print(test.wasRun)
        ```

    - 파이썬은 클래스명이나 메서드명을 함수처럼 다룰 수 있다. - 테스트 메서드를 직접 호출 걷어냄

        ```python
        class WasRun:
            def __init__(self, name):
                self.wasRun = None
                self.name = name

            def testMethod(self):
                self.wasRun = 1

            def run(self):
                # self.testMethod()
                method = getattr(self, self.name)
                method()
        ```

        - 이제 WasRun 은 독립된 두 가지 일을 수행한다.
            1. 메서드가 호출되었는지 그렇지 않은지 기억하는 일
            2. 메서드를 동적으로 호출하는 일
- 모든 테스트를 실행하고, 성공하는 것을 확인한다. (주기 4)
- 리팩토링 한다. (주기 5)
    - 유사분열을 일으키자: 비어있는 TestCase 상위 클래스를 만들고 WasRun 이 이를 상속받게 만들자.
        - 그 후 name 속성을 상위 클래스로 끌어올리자.

        ```python
        from xUnit.TestCase import TestCase

        class WasRun(TestCase):
            def __init__(self, name):
                self.wasRun = None
                TestCase.__init__(self, name)

            def testMethod(self):
                self.wasRun = 1

            def run(self):
                # self.testMethod()
                method = getattr(self, self.name)
                method()
        ```

        ```python
        class TestCase:
            def __init__(self, name):
                self.name = name
        ```

    - run()을 상위 클래스(TestCase)로 올리고, Execute.py 로직을 담은 클래스 TestCaseTest 생성

        ```python
        class TestCase:
            def __init__(self, name):
                self.name = name

            def run(self):
                method = getattr(self, self.name)
                method()
        ```

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            def testRunning(self):
                test = WasRun("testMethod")
                assert not test.wasRun
                test.run()
                assert test.wasRun

        TestCaseTest("testRunning").run()
        ```

        ```python
        from xUnit.TestCase import TestCase

        class WasRun(TestCase):
            def __init__(self, name):
                self.wasRun = None
                TestCase.__init__(self, name)

            def testMethod(self):
                self.wasRun = 1
        ```

        - 메서드 추출의 한 복잡한 형태: 프린트문이 단언(assertion)으로 바뀌었다.
- 열여덟 번째 테스트 완료

    ```
    - ~~테스트 메서드 호출하기~~
    - 먼저 setUp 호출하기
    - 나중에 tearDown 호출하기
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    ```

- 리팩토링의 일반적인 패턴
    - 하나의 특별한 사례에 대해서만 작동하는 코드를 가져다가 다른 여러 사례에 대해서도 작동할 수 있도록 상수를 변수로 변화시켜 일반화하는 것이다.
    - 머릿속 순수한 추론에 의해 일반화하게 하지 않고, 잘 돌아가는 구체적인 사례에서 시작하여 일반화 할 수 있게 해준다는 점에서 TDD 는 이를 잘 지원한다.

### 요약

---

- 지금까지 한 것 검토
    - 자기 과신에 차서 몇 번의 잘못된 출발을 한 후, 아주 자그마한 단계로 시작하는 법을 알아냈다.
    - 일단 하드코딩을 한 다음에 상수를 변수로 대체하여 일반성을 이끌어내는 방식으로 기능을 구현했다.
    - 플러거블 셀렉터(Pluggable Selector)를 사용했다. 플러거블 셀렉터는 정적 코드 분석을 어렵게 만들기 때문에 앞으로 최소 4개월 안에는 사용하지 않기로 하자.
    - 테스트 프레임워크를 작은 단계로만 부트스트랩했다.


## 19장. 테이블 차리기

- 테스트 작성 시 공통 패턴(3A)
    1. 준비(arrange) - 객체를 생성한다.
    2. 행동(act) - 어떤 자극을 준다.
    3. 확인(assert) - 결과를 검사한다.
    - 준비 단계는 여러 테스트에 걸쳐 동일한 경우가 종종 있다.
        - 서로 다른 스케일에서 객체 초기화가 반복되는 문제에 직면하게 된다.
        - 이 때 다음 2가지 제약이 상충한다.
            - 성능
                - 우린 테스트가 될 수 있는 한 빨리 실행되길 원한다.
                - 여러 테스트에서 같은 객체를 사용한다면, 하나만 생성해서 모든 테스트가 이 객체를 쓰게 할 수 있다.
            - 격리
                - 우린 한 테스트에서의 성공이나 실패가 다른 테스트에 영향주지 않기를 원한다.
                - 만약 테스트들이 객체를 공유하는 상태에서, 한 테스트가 공유 객체의 상태를 변경한다면 다음 테스트 결과에 영향을 미칠 가능성이 있다.
- 할 일 목록

    ```
    - ~~테스트 메서드 호출하기~~
    - **먼저 setUp 호출하기**
    - 나중에 tearDown 호출하기
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    ```

- 작은 테스트를 추가한다. (주기 1)- 테스트가 돌 때 마다 객체를 생성한다.
    - 객체 생성이 충분히 빠를 경우로 가정한다.

    ```python
    def testSetUp(self):
        test = WasRun("testMethod")
        test.run()
        assert test.wasSetUp
    ```

- 모든 테스트를 실행하고, 실패하는 것을 확인한다. (주기 2)
- 코드에 변화를 준다. (주기 3)
    - 런타임 에러 수정 - WasRun 에 wasSetUp 속성 추가

        ```python
        class TestCase:
            def __init__(self, name):
                self.name = name

            def run(self):
                self.setUp()
                method = getattr(self, self.name)
                method()

            def setUp(self):
                pass
        ```

        ```python
        from xUnit.TestCase import TestCase

        class WasRun(TestCase):
            def __init__(self, name):
                self.wasRun = None
                TestCase.__init__(self, name)

            def testMethod(self):
                self.wasRun = 1

            def setUp(self):
                self.wasSetUp = 1
        ```

    - 테스트 케이스 하나를 돌아가게 하기 위해 2단계가 필요한데, 까다로운 상황에서는 너무 많은 단계다.
- 리팩토링 한다. (주기 5) - 먼저 setUp()을 호출하게 해, 테스트를 단순화 한다.

    ```python
    from xUnit.TestCase import TestCase

    class WasRun(TestCase):
        def __init__(self, name):
            TestCase.__init__(self, name)

        def testMethod(self):
            self.wasRun = 1

        def setUp(self):
            self.wasRun = None
            self.wasSetUp = 1
    ```

    ```python
    from xUnit.TestCase import TestCase
    from xUnit.WasRun import WasRun

    class TestCaseTest(TestCase):
        def setUp(self):
            self.test = WasRun("testMethod")

        def testRunning(self):
            # assert not test.wasRun
            self.test.run()
            assert self.test.wasRun

        def testSetUp(self):
            self.test.run()
            assert self.test.wasSetUp

    TestCaseTest("testRunning").run()
    TestCaseTest("testSetUp").run()
    ```

- 열아홉 번째 테스트 완료

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - 나중에 tearDown 호출하기
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    ```

- 테스트 간 커플링: 한 테스트가 깨지면 다음 테스트 코드가 올바르더라도 같이 깨진다. 만들지 말아야 한다.
- 한 번에 메서드를 하나 이상 수정하지 않으면서 테스트가 통과하게 만들 수 있는 방법을 찾아내려고 노력해라.

### 요약

---

- 한 일 검토
    - 일단은 테스트를 작성하는 데 있어 간결함이 성능 향상보다 더 중요하다고 생각하기로 했다.
    - setUp()을 테스트하고 구현했다.
    - 예제 테스트 케이스를 단순화하기 위해 setUp()을 사용했다.
    - 테스트 케이스를 단순화하기 위해 setUp()을 사용했다.


## 20장. 뒷정리하기

- 외부 자원을 할당하는 경우, 작업을 마치기 전에 tearDown() 같은 곳에서 자원을 반환할 필요가 있다.
    - 플래그를 더 추가하기엔 이제 귀찮다.
    - setUp()은 테스트 메서드가 실행되기 전에 호출되어야 하고 tearDown()은 테스트 메서드가 실행된 후에 호출되어야 한다.
    - 로그 끝부분에 기록을 추가해 메서드 호출 순서를 파악하자.
- 할 일 목록

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - **나중에 tearDown 호출하기**
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    - **WasRun 에 로그 문자열 남기기**
    ```

- 작은 단계 시작 (주기 1)
    - testSetUp()이 플래그 대신 로그를 검사하도록 변경하자.

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            def setUp(self):
                self.test = WasRun("testMethod")

            def testRunning(self):
                # assert not test.wasRun
                self.test.run()
                assert self.test.wasRun

            def testSetUp(self):
                self.test.run()
                # assert self.test.wasSetUp
                assert "setUp " == self.test.log

        TestCaseTest("testRunning").run()
        TestCaseTest("testSetUp").run()
        ```

        ```python
        from xUnit.TestCase import TestCase

        class WasRun(TestCase):
            def __init__(self, name):
                TestCase.__init__(self, name)

            def testMethod(self):
                self.wasRun = 1

            def setUp(self):
                self.wasRun = None
                self.wasSetUp = 1
                self.log = "setUp "
        ```

    - wasSetup 플래그를 지우고 테스트 메서드의 실행을 기록하자.

        ```python
        from xUnit.TestCase import TestCase

        class WasRun(TestCase):
            def __init__(self, name):
                TestCase.__init__(self, name)

            def testMethod(self):
                self.wasRun = 1
                self.log = self.log + "testMethod "

            def setUp(self):
                self.wasRun = None
                self.wasSetUp = 1
                self.log = "setUp "
        ```

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            def setUp(self):
                self.test = WasRun("testMethod")

            def testRunning(self):
                # assert not test.wasRun
                self.test.run()
                assert self.test.wasRun

            def testSetUp(self):
                self.test.run()
                assert "setUp testMethod " == self.test.log

        TestCaseTest("testRunning").run()
        TestCaseTest("testSetUp").run()
        ```

    - testSetUp() 은 이제 testRunning() 과 testSetUp() 두 역할을 다 한다.
        - WasRun 인스턴스를 한 곳에서만 사용하게 setUp()으로 분리했던 것을 되돌려 놓는다.
        - testRunning() 삭제, testSetUp()은 testTemplateMethod 로 메소드명 변경한다.

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            # def setUp(self):
            #     self.test = WasRun("testMethod")

            # def testRunning(self):
            #     self.test.run()
            #     assert self.test.wasRun

            def testTemplateMethod(self):
                test = WasRun("testMethod")
                test.run()
                assert "setUp testMethod " == test.log

        # TestCaseTest("testRunning").run()
        # TestCaseTest("testSetUp").run()
        TestCaseTest("testTemplateMethod").run()
        ```

- 할 일 목록 하나 끝남

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - **나중에 tearDown 호출하기**
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    - ~~WasRun 에 로그 문자열 남기기~~
    ```

- 작은 테스트를 추가한다. (주기 1) - tearDown()을 테스트할 준비가 됐다.

    ```python
    class TestCaseTest(TestCase):
        def testTemplateMethod(self):
            test = WasRun("testMethod")
            test.run()
            assert "setUp testMethod tearDown " == test.log

    TestCaseTest("testTemplateMethod").run()
    ```

- 모든 테스트를 실행하고, 실패하는 것을 확인한다. (주기 2)
- 코드에 변화를 준다. (주기 3) - tearDown() 구현 추가

    ```python
    from xUnit.TestCase import TestCase

    class WasRun(TestCase):
        def __init__(self, name):
            TestCase.__init__(self, name)

        def testMethod(self):
            self.wasRun = 1
            self.log = self.log + "testMethod "

        def setUp(self):
            self.wasRun = None
            self.wasSetUp = 1
            self.log = "setUp "

        def tearDown(self):
            self.log = self.log + "tearDown "
    ```

    ```python
    class TestCase:
        def __init__(self, name):
            self.name = name

        def run(self):
            self.setUp()
            method = getattr(self, self.name)
            method()
            self.tearDown()

        def setUp(self):
            pass

        def tearDown(self):
            pass
    ```

- 할 일 목록 하나 끝남

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - 수집된 결과를 출력하기
    - ~~WasRun 에 로그 문자열 남기기~~
    ```

### 요약

---

- 지금까지 한 일 검토
    - 플래그에서 로그로 테스트 전략을 구조 조정했다.
    - 새로운 로그 기능을 이용하여 tearDown()을 테스트하고 구현했다.
    - 문제를 발견했는데 뒤로 되돌아가는 대신 과감히 수정했다.(잘한 일일까?)


## 21장. 셈하기

- 테스트 메서드에서 예외 발생에 상관없이 tearDown()이 호출되도록 보장해주자.
    - 테스트가 작동하도록 하려면 예외를 잡아야 한다.
    - 만약 이 기능을 구현하다가 실수하면 예외가 보고되지 않기 때문에, 실수를 알아챌 방법이 없다.
- 할 일 목록

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - **수집된 결과를 출력하기**
    - ~~WasRun 에 로그 문자열 남기기~~
    ```

- 작은 테스트를 추가한다. (주기 1) - TestCase.run()이 한 테스트 실행 결과를 기록하는 TestResult 객체를 반환하게 하자.

    ```python
    from xUnit.TestCase import TestCase
    from xUnit.WasRun import WasRun

    class TestCaseTest(TestCase):
        def testTemplateMethod(self):
            test = WasRun("testMethod")
            test.run()
            assert "setUp testMethod tearDown " == test.log

        def testResult(self):
            test = WasRun("testMethod")
            result = test.run()
            assert "1 run, 0 failed" == result.summary()

        def testFailedResult(self):
            test = WasRun("testMethod")
            result = test.run()
            assert "1 run, 1 failed" == result.summary()

    TestCaseTest("testTemplateMethod").run()
    ```

- 모든 테스트를 실행하고, 실패하는 것을 확인한다. (주기 2)
- 코드에 변화를 준다. (주기 3)
    - 가짜 구현으로 시작: TestResult 생성

        ```python
        class TestResult:
            def summary(self):
                return "%d run, 0 failed" % self.runCount
        ```

    - TestCase.run() 이 TestResult 를 반환하게 변경

        ```python
        from xUnit.TestResult import TestResult

        class TestCase:
            def __init__(self, name):
                self.name = name

            def run(self):
                self.setUp()
                method = getattr(self, self.name)
                method()
                self.tearDown()
                return TestResult()

            def setUp(self):
                pass

            def tearDown(self):
                pass
        ```

- 모든 테스트를 실행하고, 성공하는 것을 확인한다. (주기 4)

- 코드에 변화를 준다. (주기 3) - summary() 구현 실체화

    ```python
    class TestResult:
        def __init__(self):
            self.runCount = 0

        def testStarted(self):
            self.runCount += 1

        def summary(self):
            return "%d run, 0 failed" % self.runCount
    ```

    ```python
    from xUnit.TestResult import TestResult

    class TestCase:
        def __init__(self, name):
            self.name = name

        def run(self):
            result = TestResult()
            result.testStarted()

            self.setUp()
            method = getattr(self, self.name)
            method()
            self.tearDown()
            return result

        def setUp(self):
            pass

        def tearDown(self):
            pass
    ```

- 모든 테스트를 실행하고, 성공하는 것을 확인한다. (주기 4)

- 작은 테스트를 추가한다. (주기 1) - 실패하는 테스트 수를 나타내는 테스트 작성

    ```python
    from xUnit.TestCase import TestCase
    from xUnit.WasRun import WasRun

    class TestCaseTest(TestCase):
        def testTemplateMethod(self):
            test = WasRun("testMethod")
            test.run()
            assert "setUp testMethod tearDown " == test.log

        def testResult(self):
            test = WasRun("testMethod")
            result = test.run()
            assert "1 run, 0 failed" == result.summary()

        def testFailedResult(self):
            test = WasRun("testMethod")
            result = test.run()
            assert "1 run, 1 failed" == result.summary()

    TestCaseTest("testTemplateMethod").run()
    ```

    ```python
    from xUnit.TestCase import TestCase

    class WasRun(TestCase):
        def __init__(self, name):
            TestCase.__init__(self, name)

        def testMethod(self):
            self.wasRun = 1
            self.log = self.log + "testMethod "

        def setUp(self):
            self.wasRun = None
            self.wasSetUp = 1
            self.log = "setUp "

        def tearDown(self):
            self.log = self.log + "tearDown "

        def testBrokenMethod(self):
            raise Exception
    ```

- 스물한 번째 테스트 완료
    - 할 일 목록에 추가: 실패한 테스트 보고하기

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - 실패한 테스트 보고하기
    ```

### 요약

---

- 한 일 검토
    - 가짜 구현을 한 뒤에 단계적으로 상수를 변수로 바꾸어 실제 구현으로 만들었다.
    - 또 다른 테스트를 작성했다.
    - 테스트가 실패했을 때 좀더 작은 스케일로 또 다른 테스트를 만들어서 실패한 테스트가 성공하게 만드는 것을 보조할 수 있었다.


## 22장. 실패 처리하기

- 실패한 테스트를 발견하면 좀더 세밀한 단위의 테스트를 작성해서 올바른 결과를 출력하는걸 확인해보자.
- 할 일 목록

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - **실패한 테스트 보고하기**
    ```

- 실패한 테스트를 TestCase.run()에서 예외 잡아서 TestResult 객체로 정상적으로 반환하게 테스트 작성 및 구현하였다.
    - testStarted(), testFailed(): 테스트가 시작될 때, 실패할 때 보낼 메시지. 바로 테스트 작성부터 쭉 쓰겠다.

    ```python
    from xUnit.TestCase import TestCase
    from xUnit.TestResult import TestResult
    from xUnit.WasRun import WasRun

    class TestCaseTest(TestCase):
        def testTemplateMethod(self):
            test = WasRun("testMethod")
            test.run()
            assert "setUp testMethod tearDown " == test.log

        def testResult(self):
            test = WasRun("testMethod")
            result = test.run()
            assert "1 run, 0 failed" == result.summary()

        def testFailedResult(self):
            test = WasRun("testBrokenMethod")
            result = test.run()
            assert "1 run, 1 failed" == result.summary()

        def testFailedResultFormatting(self):
            result = TestResult()
            result.testStarted()
            result.testFailed()
            assert "1 run, 1 failed" == result.summary()

    print("testTemplateMethod:", TestCaseTest("testTemplateMethod").run().summary())
    print("testResult:", TestCaseTest("testResult").run().summary())
    print("testFailedResult:", TestCaseTest("testFailedResult").run().summary())
    print("testFailedResultFormatting:", TestCaseTest("testFailedResultFormatting").run().summary())
    ```

    ```python
    from xUnit.TestResult import TestResult

    class TestCase:
        def __init__(self, name):
            self.name = name

        def run(self):
            result = TestResult()
            result.testStarted()
            self.setUp()
            try:
                method = getattr(self, self.name)
                method()
            except:
                result.testFailed()
            self.tearDown()
            return result

        def setUp(self):
            pass

        def tearDown(self):
            pass
    ```

    ```python
    class TestResult:
        def __init__(self):
            self.runCount = 0
            self.failureCount = 0

        def testStarted(self):
            self.runCount += 1

        def testFailed(self):
            self.failureCount += 1

        def summary(self):
            return "%d run, %d failed" % (self.runCount, self.failureCount)
    ```

    - setUp() 예외는 잡히지 않는 상황이다.
- 스물두 번째 테스트 완료
    - 할 일 목록에 추가: setUp 에러를 잡아서 보고하기

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - 여러 개의 테스트 실행하기
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - ~~실패한 테스트 보고하기~~
    - setUp 에러를 잡아서 보고하기
    ```

### 요약

---

- 검토
    - 작은 스케일의 테스트가 통과하게 만들었다.
    - 큰 스케일의 테스트를 다시 도입했다.
    - 작은 스케일의 테스트에서 보았던 메커니즘을 이용하여 큰 스케일의 테스트를 빠르게 통과시켰다.
    - 중요한 문제를 발견했는데 이를 바로 처리하기보다는 할일 목록에 적어두었다.


## 23장. 얼마나 달콤한지

- 테스트들을 모아서 한 번에 실행할 수 있는 기능을 추가해보자.
    - 파일의 끝부분에는 모든 테스트들을 호출하는 코드가 있는데, 좀 비참해 보인다.
    - 중복은 언제나 나쁘다.
    - TestSuite 구현을 통해 컴포지트 패턴의 순수한 예제를 제공할 수 있다.
- 할 일 목록

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - **여러 개의 테스트 실행하기**
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - ~~실패한 테스트 보고하기~~
    - setUp 에러를 잡아서 보고하기
    ```

- 작은(TestSuite) 테스트 추가 (주기 1)

    ```python
    def testSuite(self):
        suite = TestSuite()
        suite.add(WasRun("testMethod"))
        suite.add(WasRun("testBrokenMethod"))
        result = suite.run()
        assert "2 run, 1 failed" == result.summary()
    ```

- 코드에 변화를 준다. (주기 3)
    - TestSuite 추가

        ```python
        class TestSuite:
            def __init__(self):
                self.tests = []

            def add(self, test):
                self.tests.append(test)

            def run(self, result):
                for test in self.tests:
                    test.run(result)
        ```

    - TestCase.run(TestResult)로 인자 받게 수정

        ```python
        class TestCase:
            def __init__(self, name):
                self.name = name

            def run(self, result):
                result.testStarted()
                self.setUp()
                try:
                    method = getattr(self, self.name)
                    method()
                except:
                    result.testFailed()
                self.tearDown()

            def setUp(self):
                pass

            def tearDown(self):
                pass
        ```

    - 테스트 호출부 변경

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.TestResult import TestResult
        from xUnit.TestSuite import TestSuite
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            def testTemplateMethod(self):
                test = WasRun("testMethod")
                test.run()
                assert "setUp testMethod tearDown " == test.log

            def testResult(self):
                test = WasRun("testMethod")
                result = test.run()
                assert "1 run, 0 failed" == result.summary()

            def testFailedResult(self):
                test = WasRun("testBrokenMethod")
                result = test.run()
                assert "1 run, 1 failed" == result.summary()

            def testFailedResultFormatting(self):
                result = TestResult()
                result.testStarted()
                result.testFailed()
                assert "1 run, 1 failed" == result.summary()

            def testSuite(self):
                suite = TestSuite()
                suite.add(WasRun("testMethod"))
                suite.add(WasRun("testBrokenMethod"))
                result = TestResult()
                suite.run(result)
                assert "2 run, 1 failed" == result.summary()

        # print("testTemplateMethod:", TestCaseTest("testTemplateMethod").run().summary())
        # print("testResult:", TestCaseTest("testResult").run().summary())
        # print("testFailedResult:", TestCaseTest("testFailedResult").run().summary())
        # print("testFailedResultFormatting:", TestCaseTest("testFailedResultFormatting").run().summary())

        suite = TestSuite()
        suite.add(TestCaseTest("testTemplateMethod"))
        suite.add(TestCaseTest("testResult"))
        suite.add(TestCaseTest("testFailedResult"))
        suite.add(TestCaseTest("testFailedResultFormatting"))
        suite.add(TestCaseTest("testSuite"))
        result = TestResult()
        suite.run(result)
        print(result.summary())
        ```

- 모든 테스트를 실행하고, 성공하는 것을 확인한다. (주기 4)
- 할 일 목록 하나 추가: TestCase 클래스에서 TestSuite 생성하기

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - **여러 개의 테스트 실행하기**
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - ~~실패한 테스트 보고하기~~
    - setUp 에러를 잡아서 보고하기
    - TestCase 클래스에서 TestSuite 생성하기
    ```

- 깨지는 테스트 3개를 고쳐보자.
    - 기존의 인자 없는 run 인터페이스를 사용하고 있어서 테스트가 깨지고 있다.

    ```python
    class TestCaseTest(TestCase):
        def testTemplateMethod(self):
            test = WasRun("testMethod")
            result = TestResult()
            test.run(result)
            assert "setUp testMethod tearDown " == test.log

        def testResult(self):
            test = WasRun("testMethod")
            result = TestResult()
            test.run(result)
            assert "1 run, 0 failed" == result.summary()

        def testFailedResult(self):
            test = WasRun("testBrokenMethod")
            result = TestResult()
            test.run(result)
            assert "1 run, 1 failed" == result.summary()
        ...
    ```

- 중복을 제거하기 위해 리팩토링 한다. (주기 5)
    - 테스트 호출 시 중복되는 TestResult 를 setUp()에서 생성하게 하여 테스트를 단순화하자.

        ```python
        from xUnit.TestCase import TestCase
        from xUnit.TestResult import TestResult
        from xUnit.TestSuite import TestSuite
        from xUnit.WasRun import WasRun

        class TestCaseTest(TestCase):
            def setUp(self):
                self.result = TestResult()

            def testTemplateMethod(self):
                test = WasRun("testMethod")
                test.run(self.result)
                assert "setUp testMethod tearDown " == test.log

            def testResult(self):
                test = WasRun("testMethod")
                test.run(self.result)
                assert "1 run, 0 failed" == self.result.summary()

            def testFailedResult(self):
                test = WasRun("testBrokenMethod")
                test.run(self.result)
                assert "1 run, 1 failed" == self.result.summary()

            def testFailedResultFormatting(self):
                self.result.testStarted()
                self.result.testFailed()
                assert "1 run, 1 failed" == self.result.summary()

            def testSuite(self):
                suite = TestSuite()
                suite.add(WasRun("testMethod"))
                suite.add(WasRun("testBrokenMethod"))
                suite.run(self.result)
                assert "2 run, 1 failed" == self.result.summary()

        # print("testTemplateMethod:", TestCaseTest("testTemplateMethod").run().summary())
        # print("testResult:", TestCaseTest("testResult").run().summary())
        # print("testFailedResult:", TestCaseTest("testFailedResult").run().summary())
        # print("testFailedResultFormatting:", TestCaseTest("testFailedResultFormatting").run().summary())

        suite = TestSuite()
        suite.add(TestCaseTest("testTemplateMethod"))
        suite.add(TestCaseTest("testResult"))
        suite.add(TestCaseTest("testFailedResult"))
        suite.add(TestCaseTest("testFailedResultFormatting"))
        suite.add(TestCaseTest("testSuite"))
        result = TestResult()
        suite.run(result)
        print(result.summary())
        ```

- 스물세 번째 테스트 완료

    ```
    - ~~테스트 메서드 호출하기~~
    - ~~먼저 setUp 호출하기~~
    - ~~나중에 tearDown 호출하기~~
    - 테스트 메서드가 실패하더라도 tearDown 호출하기
    - ~~여러 개의 테스트 실행하기~~
    - ~~수집된 결과를 출력하기~~
    - ~~WasRun 에 로그 문자열 남기기~~
    - ~~실패한 테스트 보고하기~~
    - setUp 에러를 잡아서 보고하기
    - TestCase 클래스에서 TestSuite 생성하기
    ```

### 요약

---

- 이번 장 검토
    - TestSuite 를 위한 테스트를 작성했다.
    - 테스트를 통과시키지 못한 채 일부분만 구현하였다. 이것은 '규칙' 위반이다. 만약 그때 이걸 직접 발견했다면, 돈주머니에서 테스트 케이스 두 개를 공짜로 가져가도 좋다. 테스트를 통과시키고 초록 막대 상태에서 리팩토링 할 수 있게 할 간단한 가짜 구현이 있을 것 같긴 한데..,
    - 아이템과 아이템의 모음(컴포지트)이 동일하게 작동할 수 있도록 run 메서드의 인터페이스를 변경하였고 마침네 테스트를 통과시켰다.
    - 공통된 셋업 코드를 분리했다.


## 24장. xUnit 회고

- xUnit 을 사용하다 보면 단언(assertion)의 실패와 나머지 종류의 에러 사이에 큰 차이점이 있음을 알게 될 것이다.
    - 일반적으로 assertion failure 가 디버깅 시간을 더 많이 잡아먹어서, 대부분의 xUnit 구현에서는 단언 실패와 에러를 구별한다.
    - JUnit 은 간단한 Test 인터페이스를 선언하는데 TestCase 와 TestSuite 모두 이를 상속받는다.
        - 만약 Junit 도구가 우리의 테스트를 실행하게 만들고 싶다면 Test 인터페이스를 구현하면 된다.

        ```java
        public interface Test {
            public abstract int countTestCases();
            public abstract void run(TestResult result);
        }
        ```


# 3부. 테스트 주도 개발의 패턴


## 25장. 테스트 주도 개발 패턴

- 기본적인 테스트 전략 질문
    - 테스트한다는 것은 무엇을 뜻하는가?
    - 테스트를 언제 해야 하는가?
    - 테스트할 로직을 어떻게 고를 것인가?
    - 테스트할 데이터를 어떻게 고를 것인가?

### 테스트(명사)

---

- 작성한 소프트웨어를 어떻게 테스트할 것인가? → 자동화된 테스트를 만들어라.

- 양성 피드백 고리(positive feedback loop)
    - '테스트할 시간이 없다' 의 죽음의 나선
    - 스트레스를 많이 받으면 테스트를 점점 더 뜸하게 한다.
    - 테스트를 뜸하게 하면 에러는 점점 많아질 것이다.
    - 에러가 많아지면 더 많은 스트레스를 받게 된다.

→ '테스트' 를 '자동화된 테스트' 로 치환하자!

### 격리된 테스트

---

- 테스트를 실행하는 것은 서로 아무 영향이 없어야 한다.
- 테스트의 교훈 → 각 테스트는 다른 테스트와 완전히 독립적이어야 한다.
    - 테스트가 충분히 빨라서 내가 직접, 자주 실행할 수 있게끔 만들자.
    - 어마어마한 양의 테스트 실패가 반드시 어마어마한 양의 문제를 의미하는 것은 아니다.
        - 앞의 테스트가 실패한 영향으로 다음 테스트부터 예측 불가 상태도 있다.
    - 테스트는 전체 애플리케이션을 대상으로 하는 것보다 좀더 작은 스케일로 하는 게 좋다.

- 격리된 테스트가 암묵적으로 내포하는 특징
    - 테스트가 실행 순서에 독립적이게 된다.
    - 주어진 문제를 작은 단위로 분리하기 위해 노력(때론 많은 노력이 필요)하여 각 테스트를 실행하기 위한 환경을 쉽고 빠르게 셋팅할 수 있게 해야 한다.
    - 응집도는 높고 결합도는 낮은 객체의 모음으로 구성되도록 만든다.

### 테스트 목록

---

- 뭘 테스트해야 하나? → 시작하기 전에 작성해야 할 테스트 목록을 모두 적어두자.

- 켄트 백의 할 일 목록 습관
    - 향후 몇 시간 내로 해치워야 하는 모든 할일 목록을 컴퓨터 옆에 있는 종이 조각에 적어놓는 습관이 있다.
    - 주/월 단위 목록도 만들어서 벽에 붙여둔다.
    - 새로운 항목이 나타나면 빠르고 의식적으로 이 항목이 '지금' 할일에 속하는지 '나중에' 할일에 속하는지, 할 필요가 없는 일인지를 결정한다.
- 테스트 주도 개발
    - 구현해야 할 것들에 대한 테스트를 목록에 적는다.
    - 우선 구현할 필요가 있는 모든 오퍼레이션의 사용 예들을 적는다.
    - 그 다음, 이미 존재하지 않는 오퍼레이션에 대해서는 해당 오퍼레이션의 널 버전(아무 일도 하지 않는)을 리스트에 적는다.
    - 마지막으로 깔끔한 코드를 얻기 위해 이번 작업을 끝내기 전에 반드시 해야 할 리팩토링 목록을 적는다.

- 테스트를 통과하게 만드는 과정에서 우리가 작성한 코드들은 새로운 테스트가 필요함을 암시적으로 알려줄 것이다.
    - 이 새 테스트를 리팩토링과 마찬가지로 할일 목록에 적어 놓아라.
- 현재 작업 범위를 넘어서는 큰 리팩토링 거리를 발견한다면, '다음' 할일 목록으로 옮겨라.
    - 테스트 케이스는 하나라도 '다음' 목록으로 넘기지 말자.
    - 제대로 작동하지 않는 테스트가 있다면, 제대로 되게 하는 것이 코드를 릴리즈하는 것보다 더 중요하다.

### 테스트 우선

---

- 테스트를 언제 작성하는 것이 좋을까? → 테스트 대상이 되는 코드를 작성하기 직전에 작성하는 것이 좋다.
    - 코드를 작성한 후에는 테스트를 만들지 않을 것이다.

### 단언 우선

---

- 테스트를 작성할 때 단언(assert)은 언제 쓸까? → 단언을 제일 먼저 쓰고 시작하라.
    - 시스템을 개발할 때 무슨 일부터 하는가?

        → 완료된 시스템이 어떨 거라고 알려주는 user story 부터 작성한다.

    - 특정 기능을 개발할 때 무슨 일부터 하는가?

        → 기능이 완료되면 통과할 수 있는 테스트부터 작성한다.

    - 테스트를 개발할 때 무슨 일부터 하는가?

        → 완료될 때 통과해야 할 단언부터 작성한다.

- 단언을 먼저 작성하면 작업을 단순하게 만드는 강력한 효과를 볼 수 있다.

- 테스트만 작성할 때도 몇 가지 문제들을 한번에 해결한다.
    - 테스트하고자 하는 기능이 어디에 속하나? 기존 메서드를 수정해야 하나, 기존 클래스에 새 메서드를 추가해야 하나, 아니면 이름이 같은 메서드를 새 장소에? 또는 새 클래스에?
    - 메서드명은 뭐라고 해야하나?
    - 올바른 결과를 어떤 식으로 검사할 것인가?
    - 이 테스트가 제안하는 또 다른 테스트에는 뭐가 있을까?
- 예시: 테스트의 아웃라인 만들기
    - 소켓을 통해 다른 시스템과 통신하려 한다고 가정하자. 통신을 마친 후 소켓은 닫혀있고, 소켓에서 문자열 'abc'를 읽어와야 한다고 치자.

        ```java
        testCompleteTransaction() {
            ...
            assertTrue(reader.isClosed());
            assertEquals("abc", reply.contents());
        }
        ```

    - reply 는 어디서 얻어오나? socket 이다.

        ```java
        testCompleteTransaction() {
            ...
            Buffer reply = socket.contents();
            assertTrue(reader.isClosed());
            assertEquals("abc", reply.contents());
        }
        ```

    - socket 은 어디에서 나오나? 서버에 접속할 때 생성된다.

        ```java
        testCompleteTransaction() {
            ...
            Socket reader = Socket("localhost", defaultPort());
            Buffer reply = socket.contents();
            assertTrue(reader.isClosed());
            assertEquals("abc", reply.contents());
        }
        ```

    - 작업에 앞서, 서버를 먼저 열어야 한다.

        ```java
        testCompleteTransaction() {
            Server writer = Server(defaultPort(), "abc");
            Socket reader = Socket("localhost", defaultPort());
            Buffer reply = socket.contents();
            assertTrue(reader.isClosed());
            assertEquals("abc", reply.contents());
        }
        ```

### 테스트 데이터

---

- 테스트할 때 어떤 데이터를 사용해야 하는가?
    - 테스트를 읽을 때 쉽고 따라가기 좋을만한 데이터를 사용하라.
    - 여러 의미를 담는 동일한 상수를 쓰지 마라.

- 테스트 데이터에 대한 대안: 실제 세상의 데이터를 사용하는 것이다.
- 실제 데이터가 유용한 경우
    - 실제 실행을 통해 수집한 외부 이벤트의 결과를 이용하여 실시간 시스템을 테스트하고자 하는 경우
    - 예전 시스템의 출력과 현재 시스템의 출력을 비교하고자 하는 경우(병렬 테스팅)
    - 시뮬레이션 시스템을 리팩토링 한 후, 기존과 정확히 동일한 결과가 나오는지 확인하고자 하는 경우. 특히 부동소수점 값의 정확성이 문제가 될 수 있다.

### 명백한 데이터

---

- 데이터의 의도를 어떻게 표현할 것인가? → 테스트 자체에 예상되는 값과 실제 값을 포함하고 이 둘 사이의 관계를 드러내기 위해 노력하라.
- 명백한 데이터가 주는 이점
    - 후에 코드를 읽을 다른 사람들에게 코드에 많은 실마리를 남길 수 있다.
    - 프로그래밍이 더 쉬워진다.
- 단일 메서드 범위라면, 어떤 매직넘버(5라는 숫자) 사이의 관계는 명백하다. 하지만 이미 정의된 기호 상수가 있다면 그것을 사용하자.


## 26장. 빨간 막대 패턴

- 이 패턴들은 테스트를 언제 어디에 작성할 것인지, 테스트 작성을 언제 멈출지에 대한 것이다.

### 한 단계 테스트

---

- 목록에서 다음 테스트를 고를 때 무엇을 기준으로 할 것인가? → 우리에게 새로운 무언가를 가르쳐 줄 수 있으며, 구현할 수 있다는 확신이 드는 테스트를 고를 것.
    - 각 테스트는 우리를 최종 목표로 한 단계 진전시켜 줄 수 있어야 한다.
- 만약 메타포가 어떤 방향성을 가질 필요가 있다면 ‘아는 것에서 모르는 것으로’ 방향이 유용할 것이다.

### 시작 테스트

---

- 어떤 테스트부터 시작하는게 좋을까? → 오퍼레이션이 아무 일도 하지 않는 경우를 먼저 테스트할 것.

- 새 오퍼레이션에 대한 첫 질문: “이 오퍼레이션을 어디에 넣어야 하지?”
- 첫 걸음으로 현실적인 테스트를 하나 작성한다면 상당히 많은 문제를 한번에 해결해야 하는 상황이 될 것이다.
- 현실적인 테스트 하나로 시작하면 너무 오랫동안 피드백이 없을 것이다. → 빨강/초록/리팩토링 고리가 몇 분 이내로 반복되어야 한다.
    - 정말 발견하기 쉬운 입/출력을 사용하면 이 시간을 짧게 줄일 수 있다.

- 켄트 백이 자주 테스트 주도로 개발하는 예는 간단한 소켓 기반 서버다. 첫 번째 테스트의 모습.

    ```java
    StartServer
    Socket = new Socket
    Message = "hello"
    Socket.write(message)
    AssertEquals(message, socket.read)
    ```

### 설명 테스트

---

- 자동화된 테스트가 더 널리 쓰이게 하려면 어떻게 해야 할까? → 테스트를 통해 설명을 요청하고 테스트를 통해 설명하라.

### 학습 테스트

---

- 외부에서 만든 소프트웨어에 대한 테스트를 작성해야 할 때도 있을까? → 패키지의 새로운 기능을 처음 사용해보기 전에 작성할 수 있다.
    - 만약 테스트가 통과되지 않는다면 애플리케이션 역시 실행되지 않을 것이 뻔하기 때문에 애플리케이션을 실행해 볼 필요도 없다.

### 또 다른 테스트

---

- 어떻게 하면 주제에서 벗어나지 않고 기술적 논의를 계속할 수 있을까? → 주제와 무관한 아이디어가 떠오르면 이에 대한 테스트를 할 일 목록에 적어놓고 다시 주제로 돌아올 것.

### 회귀 테스트

---

- 시스템 장애가 보고될 때 무슨 일을 제일 먼저 하는가? → 그 장애로 인하여 실패하는 테스트, 그리고 통과할 경우엔 장애가 수정되었다고 볼 수 있는 테스트를 가장 간단하게 작성하라.

- 회귀 테스트: 우리에게 완벽한 선견지명이 있다면, 처음 코딩할 때 작성했어야 하는 테스트다.
    - 회귀 테스트를 작성할 때는 이 테스트를 작성해야 한다는 사실을 어떻게 하면 애초에 알 수 있었을지 항상 생각해보라.
    - 전체 애플리케이션 차원의 회귀 테스트는 시스템의 사용자들이 여러분에게 정확히 무엇을 기대했으며 무엇이 잘못되었는지 말할 기회를 준다.
    - 좀더 작은 차원에서 회귀 테스트는 당신의 테스트를 개선하는 방법이 된다.

### 휴식

---

- 지치고 고난에 빠졌을 땐 뭘 해야 하나? → 그럴 땐 좀 쉬는 게 좋다.
    - 음료수도 마시고, 좀 걷고, 낮잠도 자보자.
    - 내가 결정한 것에 대한 감정적 책임과 당신이 타이핑해 넣은 문자들을 손에서 깨끗이 씻어버리도록 하라.
    - 거리 두기를 통해 부족했던 아이디어가 튀어나올 수 있다.
- 만약 ‘아이디어’를 얻지 못한다면, 현재 세션의 목적을 다시 검토해보라.
    - 여전히 현실적인가, 아니면 새로운 목적을 골라야 하는가?
    - 당신이 이루려고 노력했던 것이 불가능한 건 아닌가? 만약 그렇다면 팀에는 어떤 의미가 있나?

- 휴식의 역학: 피로해지면 (판단력이 떨어지므로) 내가 피로해졌다는 것을 올바로 인식하기가 힘들어진다. 그래서 계속 일하면 더 피곤해진다. 이 고리에서 빠져나가는 방법은 추가로 외부 요소를 도입하는 것이다.
    - 시간 단위로는, 물병을 키보드 옆에 두어서 생리 현상으로 규칙적인 휴식을 하도록 유도하라.
    - 하루 단위로는, 더 진행하기 전에 잠이 필요한 경우 정규 근무 시간 후의 약속이 진행을 일단 멈추는 데에 도움이 될 수 있다.
    - 주 단위로는, 의식적이고 에너지 소모적인 업무 관련 생각을 떨쳐버리는 데에 주말 활동이 도움이 된다.
    - 년 단위로는, 강제 휴가 정책이 재충전을 완벽히 도와줄 것이다.

- 프로그래밍 문화는 마초 정신에 심하게 길들여져 있다. “내 건강을 희생해서, 가족도 멀리하고, 필요하다면 목숨도 버릴 각오가 되었소” → 권하고 싶지 않은 문화다.

### 다시 하기

---

- 길을 잃은 느낌이 들 땐 어떻게 할까? → 코드를 다 지워버리고 처음부터 다시 해보자.

### 싸구려 책상, 좋은 의자

---

- TDD 를 할 때 어떤 물리적 환경이 적절한가? → 나머지는 싸구려를 쓸지라도 정말 좋은 의자를 구해라.


## 27장. 테스팅 패턴

- 이 패턴들은 더 상세한 테스트 작성법에 대한 것이다.

### 자식 테스트

---

- 지나치게 큰 테스트 케이스를 어떻게 돌아가도록 할 수 있을까? → 원래 테스트 케이스의 깨지는 부분에 해당하는 작은 테스트 케이스를 작성하고, 그 작은 테스트 케이스가 실행되도록 하라. 그 후 다시 원래의 큰 테스트 케이스를 추가하라.

- 빨강/초록/리팩토링 리듬은 성공이 지속되는 데 너무나도 중요하다.

### 모의 객체

---

- 비용이 많이 들거나 복잡한 리소스에 의존하는 객체를 테스트하려면 어떻게 해야할까? → 상수를 반환하게끔 만든 속임수 버전의 리소스를 만들면 된다.
- ex> 데이터베이스
    - 해법은 대부분의 경우에 진짜 데이터베이스를 사용하지 않는 것이다.
    - 대다수의 테스트는, 마치 DB 인 것처럼 행동하지만 실제로는 메모리에만 존재하는 객체를 통해 작성될 수 있다.

    ```java
    public void testOrderLookUp() {
        Database db = new Database();
        db.expectQuery("select order_no from Order where cut_no is 123");
        db.returnResult(new String[] {"Order 2", "Order 3"});
        ...
    }
    ```

- 모의 객체의 가치
    - 성능과 견고함
    - 가독성
    - 모든 객체의 가시성에 대해 고미하도록 격려해서, 설계에서 커플링이 감소하도록 한다.

- 만약 모의 객체를 사용하길 원한다면, 값비싼 자원을 전역 변수에 손쉽게 저장해 버릴 수는 없다.(비록 그것들이 싱글톤이더라도)
    - 만약 그렇게 한다면, 전역 변수를 모의 객체로 설정하고, 테스트를 실행한 후 다시 전역 변수를 복구시켜 놓아야 한다.
    - 켄트 백은 전역 변수(서로 다른 환율)를 가지고 시도하다가, 그 환율 객체 Exchange 를 모든 필요한 곳에 전달하기로 결정했다. 10개 메서드에 인자를 추가했고, 다른 설계 문제도 깨끗이 정리했다.
- 모의 객체가 진짜 객체와 동일하게 동작하지 않으면, 모의 객체용 테스트 집합을 진짜 객체가 사용 가능해질 때 그대로 적용해서 이러한 위험을 줄일 수 있다.

### 셀프 션트

---

- 한 객체가 다른 객체와 올바르게 대화하는지 테스트하려면 어떻게 할까? → 테스트 대상이 되는 객체가 원래의 대화 상대가 아니라 테스트 케이스와 대화하도록 만들면 된다.

- 셀프 션트 패턴
    - 셀프 션트 패턴을 이용해 작성한 테스트가 그렇지 않은 테스트보다 읽기에 더 수월하다.
    - 셀프 션트 패턴은 테스트 케이스가 구현할 인터페이스를 얻기 위해 인터페이스 추출을 해야 한다.
        - 인터페이스 추출이 더 쉬운지, 존재하는 클래스를 블랙 박스로 테스트하는 것이 더 쉬운지는 당신이 결정해야 할 것이다.
        - 켄트 백은 셀프 션트를 위해 추출해 낸 인터페이스는 여러 곳에서 쓰이는 경우가 많다고 생각한다.
- 자바의 경우, 셀프 션트를 사용한 결과로 인터페이스 안의 온갖 기괴한 메서드들을 다 구현한 테스트들을 보게 될 것이다.
    - 자바에서는 빈 메서드로라도 인터페이스의 모든 오퍼레이션들을 구현해야 한다.
- 인터페이스에 대한 구현은 또한 적절한 값을 되돌리거나 부적절한 오퍼레이션이 호출된 경우 예외를 던지게끔 만들어야 할 것이다.

### 로그 문자열

---

- 메시지의 호출 순서가 올바른지를 검사하려면 어떻게 해야할까? → 로그 문자열을 가지고 있다가 메시지가 호출될 때마다 그 문자열에 추가하도록 한다.

- 로그 문자열은 특히 옵저버를 구현하고, 이벤트 통보가 원하는 순서대로 발생하는지를 확인하고자 할 때 유용하다.
    - 만약 어떤 이벤트 통보들이 일어나는지를 검사하지만, 그 순서는 상관이 없을 경우엔 문자열 집합을 저장하고 있다가 단언에서 Set 비교를 수행하면 된다.
- 로그 문자열은 셀프 션트와도 잘 작동한다.
    - 해당 테스트 케이스는 로그를 추가하고 적절한 값을 반환하는 식으로 셀프 션트한 인터페이스의 메서드를 구현한다.

### 크래시 테스트 더미

---

- 호출되지 않을 것 같은 에러 코드(발생하기 힘든 에러 상황)를 어떻게 테스트 할 것인가? → 실제 작업을 수행하는 대신 그냥 예외를 발생시키기만 하는 특수한 객체를 만들어서 이를 호출한다.

- 테스트되지 않은 코드는 작동하는 것이 아니다. 수많은 에러 상황에 대해서는 어떻게 테스트할 것인가?
    - 작동하길 원하는 부분에 대해서만 테스트하면 된다.
    - 파일 시스템에 여유 공간이 없을 경우 발생할 문제에 대해 테스트하기를 원한다고 생각해보자.
        - 실제로 큰 파일을 많이 만들어서 파일 시스템을 꽉 채울 수도 있고, 가짜 구현(시뮬레이션)을 사용할 수도 있다.
- 객체 전체를 흉내낼 필요가 없다는 점을 제외하면 크래시 테스트 더미는 모의 객체와 유사하다.
- 자바의 익명 내부 클래스는 우리가 테스트하기 원하는 적절한 메서드만이 오류를 발생시키게끔 하기 위해 유용하게 쓰인다.
    - 테스트 케이스 안에서 원하는 메서드 하나만 재정의할 수 있다.

### 깨진 테스트

---

- 혼자서 프로그래밍할 때 프로그래밍 세션을 어떤 상태로 끝마치는 게 좋을까? → 마지막 테스트가 깨진 상태로 끝마치는 것이 좋다.

- 프로그래밍 세션을 끝낼 때 테스트 케이스를 작성하고, 이것이 실패하는 것을 확실히 확인한다.
    - 나중에 다시 코딩하기 위해 돌아왔을 때, 어느 작업부터 시작할 것인지 명백히 알 수 있다.
    - 전에 하고 있던 생각에 대한 명확하고 구체적인 책갈피를 가지게 되는 것이다.

### 깨끗한 체크인

---

- 팀 프로그래밍을 할 때 프로그래밍 세션을 어떤 상태로 끝마치는 게 좋을까? → 모든 테스트가 성공한 상태로 끝마치는 것이 좋다.
    - 안심이 되고 확신이 있는 상태에서 시작할 필요가 있다.

- 통합 테스트 슈트에서 테스트가 실패하는 경우,
    - 체크인하기 전에 실행하는 통합 테스트 슈트는 작업 중에 분 단위로 실행하는 테스트 슈트보다 더 클 것이다.
    - 가장 단순한 규칙은 그동안 내가 작업한 코드를 날려버리고 다시 하는 것이다.
    - 이 방법보다 약간 방탕한 접근은, 문제를 수정하고 테스트를 다시 실행해보는 것이다.(포기해버릴수도..)
    - 테스트 슈트를 통과시키기 위해 주석 처리 하는 것은 엄격히 금지다.


## 28장. 초록 막대 패턴

### 가짜로 구현하기(진짜로 만들기 전까지만)

---

- 실패하는 테스트를 만든 후 첫 번째 구현은 어떻게 하는 게 좋을까? → 상수를 반환하게 하라. 일단 테스트가 통과하면 단계적으로 상수를 변수를 사용하는 수식으로 변형한다.

- 가짜로 구현하기의 강력한 효과
    - 심리학적: 초록 막대 상태에 있는 것은 빨간 막대 상태에 있는 것과 천지 차이다.
        - 막대가 초록색일 때, 자신이 어디에 서 있는지 안다.
        - 우리는확신을 갖고 거기부터 리팩토링해 갈 수 있다.
    - 범위 조절: 하나의 구체적인 예에서 시작해서 일반화하게 되면, 쓰잘데기 없는 고민으로 때 이르게 혼동하는 일을 예방할 수 있다.
        - 프로그래머들은 모든 종류의 미래 문제를 상상하는 데에 탁월하다.
        - 다음 테스트 케이스를 구현할 때, 이전 테스트의 작동이 보장된다는 것을 알기 때문에 그 다음 테스트 케이스에도 집중할 수 있다.

### 삼각측량

---

- 추상화 과정을 테스트로 주도할 때 어떻게 최대한 보수적으로 할 수 있겠는가? → 오로지 예가 2개 이상일 때만 추상화를 하라.

- 삼각측량이 매력적인 이유는 그 규칙이 매우 명확하기 때문이다.
- 어떤 계산을 어떻게 해야 올바르게 추상화할 것인지에 대해 감잡기 어려울 때만 삼각측량을 사용한다.
    - 그 외의 경우, 명백한 구현이나 가짜로 구현하기에 의존한다.

### 명백한 구현

---

- 단순한 연산들을 어떻게 구현하는가? → 그냥 구현해 버려라.

- 때로 어떤 연산을 어떻게 구현해야 할 지 확신이 들기도 한다. 빨간 막대를 본다면 그제서야 좀더 작은 발걸음으로 옮겨가자.
- ‘제대로 동작하는’ 을 푸는 동시에 ‘깨끗한 코드’ 를 해결하려는 것은 한번에 하기에는 너무 많은 일일 수 있다.
    - 그렇게 되면 우선 ‘제대로 동작하는’ 으로 되돌아 가서 그걸 해결하고, 그 후에 ‘깨끗한 코드’ 를 느긋하게 해결하도록 하자.

### 하나에서 여럿으로

---

- 객체 컬렉션을 다루는 연산은 어떻게 구현하나? → 일단은 컬렉션 없이 구현하고 그 다음에 컬렉션을 사용하게 하자.


## 29장. xUnit 패턴

- xUnit 계열 테스팅 프레임워크를 위한 패턴이다.

### 단언(assertion)

---

- 테스트가 잘 작동하는지 어떻게 검사할 것인가? → 불리언 수식을 작성해서 프로그램이 대신 자동으로 코드가 동작하는지에 대한 판단을 수행하도록 하라.

- 테스트를 완전히 자동화 하기: 결과를 평가하는 데 개입되는 인간의 판단을 모조리 끄집어내야 한다.
    - 판단 결과가 불리언 값이어야 한다. 일반적으로 참 값은 모든 테스트가 통과했음을 의미하고, 거짓 값은 뭔가 예상치 못했던 일이 발생했음을 의미한다.
    - 이 불리언 값은 컴퓨터에 의해 검증되어야 한다. 보통 다양한 형태의 assert()를 호출하여 이 값을 얻어낸다.
    - 단언은 구체적이어야 한다.

- 객체를 블랙박스처럼 취급하기란 쉽지 않다. 하지만 화이트박스 테스트를 바라는 것은 테스트 문제가 아니라 설계 문제다.
    - 그치만 정말 설게 아이디어가 떠오르지 않으면, 그냥 변수를 검사하게 만들고 TODO 로 적어놓고서 다음 작업을 진행하자.

### 픽스처

---

- 여러 테스트에서 공통으로 사용하는 객체들을 생성할 때 어떻게 하면 좋을까? → 각 테스트 코드에 있는 지역 변수를 인스턴스 변수로 바꾸고 setUp()을 재정의하여 이 메서드에서 인스턴스 변수들을 초기화하도록 한다.

- 객체들을 세팅하는 코드는 여러 테스트에 걸쳐 동일한 경우가 있다.(이러한 객체들이 테스트 픽스처)
- 테스트 픽스처 중복이 좋지 않은 이유
    - 복붙을 한다고 하더라도 이런 코드를 반복 작성하는 것엔 시간이 소요되는데, 우리는 테스트를 빨리 작성하길 원한다.
    - 인터페이스를 수동으로 변경할 필요가 있을 경우, 여러 테스트를 고쳐줘야 한다.(이는 중복에 대해 정확히 예상되는 결과다)

### 외부 픽스처

---

- 픽스처 중 외부 자원이 있을 경우 이를 어떻게 해제할 것인가? → tearDown()을 재정의하여 이곳에서 자원을 해제하자.

- 각 테스트의 목적 중 하나는 테스트가 실행되기 전/후 의 외부 세계가 동일하게 유지되도록 만드는 것이다.
- xUnit 은 각 테스트가 끝난 후에 tearDown() 호출을 보장해준다.

### 테스트 메서드

---

- 테스트 케이스 하나를 어떻게 표현할 것인가? → ‘test’ 로 시작하는 이름의 메서드로 나타내면 된다.
- 테스트 메서드는 의미가 그대로 드러나는 코드로, 읽기 쉬워야 한다.

- 테스트를 작성하기 전에 일단 원하는 테스트에 대한 짧은 아웃라인을 적는다.

    ```java
    // 튜플 공간에 추가하기
    // 튜플 공간에서 갖고오기
    // 튜플 공간에서 읽어들이기
    ```

- 켄트 백은 아웃라인에 보통 2, 3 레벨만 넣는다. 아웃라인을 그보다 더 많이 단 경우는 없다.

### 예외 테스트

---

- 예외가 발생하는 것이 정상인 경우에 대한 테스트는 어떻게 작성할 것인가? → 예상되는 예외를 잡아서 무시하고, 예외가 발생하지 않은 경우에 한해서 테스트가 실패하게 만들면 된다.

- 우리가 원하는 예외인 경우에만 잡고, 그 외엔 xUnit 의 fail()로 테스트가 실패했음을 알려주자.

### 전체 테스트

---

- 모든 테스트를 한번에 실행하려면 어떻게 해야할까? → 모든 테스트 슈트에 대한 모음을 작성하면 된다.
    - 각 패키지에 대해 하나씩, 그리고 전체 애플리케이션의 패키지 테스트를 모아주는 테스트 슈트


## 30장. 디자인 패턴

- 우리가 다룰 디자인 패턴
    - 커맨드
        - 계산 작업에 대한 호출을 메시지가 아닌 객체로 표현한다.
    - 값 객체
        - 객체가 생성된 이후 그 값이 절대로 변하지 않게 하여 별칭 문제가 발생하지 않게 한다.
    - 널 객체
        - 계산 작업의 기본 사례를 객체로 표현한다.
    - 템플릿 메서드
        - 계산 작업의 변하지 않는 순서를 여러 추상 메서드로 표현한다.
        - 이 추상 메서드들은 상속을 통해 특별한 작업을 수행하게끔 구체화된다.
    - 플러거블 객체
        - 둘 이상의 구현을 객체를 호출함으로써 다양성을 표현한다.
    - 플러거블 셀렉터
        - 객체별로 서로 다른 메서드가 동적으로 호출되게 함으로써 필요없는 하위 클래스의 생성을 피한다.
    - 팩토리 메서드
        - 생성자 대신 메서드를 호출함으로써 객체를 생성한다.
    - 임포스터
        - 현존하는 프로토콜을 갖는 다른 구현을 추가하여 시스템에 변이를 도입한다.
    - 컴포지트
        - 하나의 객체로 여러 객체의 행위 조합을 표현한다.
    - 수집 매개 변수
        - 여러 다른 객체에서 계산한 결과를 모으기 위해 매개 변수를 여러 곳으로 전달한다.

### 커맨드

---

- 간단한 메서드 호출보다 복잡한 형태의 계산 작업에 대한 호출이 필요하다면 어떻게 해야할까? → 계산 작업에 대한 객체를 생성하여 이를 호출하면 된다.

- ex> 자바의 Runnable 인터페이스
    - 객체를 생성할 때 계산에 필요한 모든 매개 변수들을 초기화한다.
    - 호출할 준비가 되면 run()과 같은 프로토콜을 이용해서 계산을 호출한다.

    ```java
    interface Runnable
        public abstract void run();
    ```

### 값 객체

---

- 널리 공유해야 하지만 동일성은 중요하지 않을 때 객체를 어떤식으로 설계할 수 있을까? → 객체가 생성될 때 객체의 상태를 설정한 후 이 상태가 절대 변할 수 없도록 한다. 그리고 이 객체에 대해 수행되는 연산은 언제나 새로운 객체를 반환하게 만든다.

- 별칭 문제를 해결하기 위한 몇 가지 방법
    - 현재 의존하는 객체에 대한 참조를 결코 외부로 알리지 않는 방법. 대신 객체에 대한 복사본을 제공한다.
        - 수행 시간이나 메모리 공간 측면에서 비싼 해결책일 수도 있고, 공유 객체의 상태 변화를 공유하고 싶은 경우에는 사용할 수 없다는 단점이 있다.
    - 옵저버 패턴을 사용하는 것. 의존하는 객체에 자기를 등록해놓고, 객체의 상태가 변하면 통지를 받는 방법이다.
        - 옵저버 패턴은 제어 흐름을 이해하기 어렵게 만들 수 있고, 의존성을 설정하고 제거하기 위한 로직이 지저분해질 수 있다.
    - 객체를 덜 객체답게 취급하는 법. 객체는 시간의 흐름에 따라 변할 수 있는 상태를 갖고 있다.
        - ‘시간의 흐름에 따라 변하는 상태’ 를 제거해버리자.
        - 이 객체가 변하지 않을 것임을 안다면 이 객체에 대한 참조를 원하는 곳 어디로든 넘겨줄 수 있다.

- 값 객체
    - 모든 오퍼레이션은 기존 객체는 변하지 않은 채로 놔두고, 새로운 객체를 반환해야 한다.
    - 켄트 백은 교차하고 합해지는 기하학적 모양들, 숫자와 함께 그 단위가 따라다니는 단위값, 기호대수학 등 대수학 같아 보이는 상황이라면 언제나 값 객체를 사용하려는 경향이 있다.
    - 값 객체가 코드 읽기, 디버깅하기를 훨씬 쉽게 해주기 때문이다.
    - 모든 값 객체는 동등성을 구현해야 한다.
    - 그리고 많은 언어에서 값 객체는 암묵적으로 해싱을 구현해야 한다.

### 널 객체

---

- 객체의 특별한 상황을 표현하고자 할 때 어떻게 해야할까? → 그 특별한 상황을 표현하는 새로운 객체를 만들면 된다. 그리고 이 객체에 다른 정상적인 상황을 나타내는 객체와 동일한 프로토콜을 제공한다.

- null 체크를 하는 대신, 절대로 예외를 던지지 않는 새로운 클래스를 만드는 방법이다.

### 템플릿 메서드

---

- 작업 순서는 변하지 않지만 각 작업 단위에 대한 미래의 개선 가능성을 열어두고 싶은 경우, 이를 어떻게 표현할 것인가? → 다른 메서드들을 호출하는 내용으로만 이루어진 메서드를 만든다.

- 템플릿 메서드를 만들 때 한 가지 문제는 하위 클래스를 위한 기본 구현을 제공할지 말지 이다.
- 하위 단계가 정의되지 않은 상태에서 연산을 구현하는 것이 의미가 없는 경우라면, 어떤 방법을 이용하건 간에 이를 나타내 주어야 한다.
    - 자바에서는 하위 메서드를 추상 메서드로 선언한다.
    - 스몰토크에서는 메서드가 SubclassResponsibility 에러를 던지게 만든다.

- 템플릿 메서드는 초기의 설계에 의해서 얻어지는 것보다, 경험에 의해 발견되는 것이 좋다.
    - “아, 이 부분은 순서에 관한 내용이고, 이 부분이 상세 구현에 대한 내용이로군” 하는 생각이 들 때면, 늘 그 상세 구현을 나중에 인라인시키고 진짜로 변하는 부분만 다시 추출한다.
- 나머지 메서드들과는 다른 부분을 추출해 내면 남는 것은 템플릿 메서드다.
- 그 다음 템플릿 메서드를 상위 클래스로 보내고 중복을 제거할 수 있다.

### 플러거블 객체

---

- 변이를 어떻게 표현할 것인가? → 가장 간단한 방법은 명시적인 조건문을 사용하는 것이다.
    - 하지만 이러한 명시적인 의사 결정 코드(조건문)는 소스의 여러 곳으로 퍼져나간다.

    ```java
    if (circle) then {
        ... circley stuff ...
    } else {
        ... non circley stuff
    }
    ```

- 조건문을 2번째로 볼 때 → 객체 설계시의 가장 기초인 플러거블 객체를 끄집어낼 때이다.
    - 명시적인 조건문이 전염되는 싹을 애초에 잘라버려야 한다.
    - 중복 조건문들 대신, 플러거블 객체(SelectionMode)를 만들어서 두 가지 구현(SingleSelection, MultipleSelection)을 갖게 한다.
    - 명시적인 인터페이스를 사용하는 언어에서는 2개 이상의 플러거블 객체가 동일한 인터페이스를 구현하게 해야 한다.

### 플러거블 셀렉터

---

- 인스턴스 별 서로 다른 메서드가 동적으로 호출되게 하려면 어떻게 해야할까? → 메서드의 이름을 저장하고 있다가 그 이름에 해당하는 메서드를 동적으로 호출한다.

- 한 가지 대안은 switch 문을 갖는 하나의 클래스를 만드는 것
    - 필드값에 따라 서로 다른 메서드를 호출하면 된다.
    - 하지만 메서드의 이름이 세 곳에 나뉘어 존재하게 된다.
        - 인스턴스 생성하는 곳
        - switch문
        - 메서드 자체
- 다른 플러거블 셀렉터 해법: 리플렉션을 이용하여 동적으로 메서드를 호출한다.

    ```java
    void print() {
        Method runMethod = getClass().getMethod(printMessage, null);
        runMethod.invoke(this, new Class[0]);
    }
    ```

    - 여전히 리포트를 생성하는 곳과 출력 메서드의 이름 사이에 지저분한 의존 관계가 남아있긴 하지만, 최소한 switch 문은 없다.

- 플러거블 셀렉터의 가장 큰 문제: 메서드가 호출 되었는지 보기 위해 코드를 추적하는 것.
- 메서드를 1개만 가지는 하위 클래스들이 한 뭉치 존재하는, 확실히 직관적인 상황에서 코드 정리를 위한 용도로만 플러거블 셀렉터를 사용해야 한다.

### 팩토리 메서드

---

- 새 객체를 만들 때 유연성을 원하는 경우 객체를 어떻게 생성하는가? → 생성자를 쓰는 대신 일반 메서드에서 객체를 생성한다.

- 생성자 사용
    - 생성자는 자신을 잘 표현한다. 분명히 객체 하나를 생성하고 있다는 것을 알 수 있다.
    - 그러나 생성자는, (특히 자바에서) 표현력과 유연함이 떨어진다.

- 유연함의 한 측면: 생성자에서 다른 클래스의 객체를 생성할 수 있게 하는 것이다.
    - 메서드라는 한 단계의 인디렉션(indirection)을 추가함으로써 테스트를 변경하지 않고 다른 클래스의 인스턴스를 반환할 수 있는 유연함을 얻을 수 잇다.
- 팩토리(factory) 메서드: 객체를 생성하기 때문.
    - 단점: 인디렉션. 메서드가 생성자처럼 생기진 않았지만, 그 안에서 객체를 만든다는 사실을 기억해야만 한다.
    - 유연함이 필요할 때만 팩토리 메서드를 사용해야 한다. 그렇지 않다면 객체를 생성하는 데에는 생성자를 쓰는 것으로 충분하다.

### 사칭 사기꾼

---

- 기존의 코드에 새로운 변이를 도입하려면 어떻게 해야할까? → 기존의 객체와 같은 프로토콜을 갖지만 구현은 다른 새로운 객체를 추가한다.

- 리팩토링 중 나타나는 2가지 사칭 사기꾼
    - 널 객체: 데이터가 없는 상태를 데이터가 있는 상태와 동일하게 취급할 수 있다.
    - 컴포지트: 객체의 집합을 단일 객체처럼 취급할 수 있다.
- 사칭 사기꾼을 찾아내는 것: 리팩토링 중 중복을 제거하는 작업을 통해 유도된다.

### 컴포지트

---

- 하나의 객체가 다른 객체 목록의 행위를 조합한 것처럼 행동하게 만들려면 어떻게 해야할까? → 객체 집합을 나타내는 객체를 단일 객체에 대한 임포스터로 구현한다.

### 수집 매개 변수

---

- 여러 객체에 걸쳐 존재하는 오퍼레이션의 결과를 수집하려면 어떻게 해야할까? → 결과가 수집될 객체를 각 오퍼레이션의 매개 변수로 추가한다.

- 수집 매개 변수를 추가하는 것은 컴포지트의 일반적인 귀결이다.
    - 기대하는 결과가 더 복잡해지면, 수집 매개 변수를 도입할 필요를 느낄수도 있다.

### 싱글톤

---

- 전역 변수를 제공하지 않는 언어에서 전역 변수를 사용하려면 어떻게 해야할까? → 사용하지 마라.
    - 설계에 대해 고민하는 시간을 가져라.


## 31장. 리팩토링

- 이 패턴들은 시스템의 설계를 작은 단계를 통해 (아무리 큰 변화라도) 변화시키는 방법에 대해 설명한다.

- 리팩토링: 어떤 상황에서도 프로그램의 의미론을 변경해서는 안된다.
    - TDD 에서는 상수를 변수로 바꾸고 리팩토링이라고 부른다. 왜냐하면 통과하는 테스트의 집합에 아무 변화도 주지 않기 때문이다.

### 차이점 일치시키기

---

- 비슷해보이는 두 코드 조각을 합치려면 어떻게 해야할까? → 두 코드가 단계적으로 닮아가게끔 수정한다. 이 둘이 완전히 동일해지면 둘을 합친다.

- 피해가고자 하는 일: 불확실한 믿음에 의지하여 단계를 크게 건너뛰는 리팩토링
    - 작은 단계와 명확한 피드백을 이용하자.

- 리팩토링
    - 두 반복문의 구조가 비슷하다. 이 둘을 동일하게 만들고 나서 하나로 합친다.
    - 조건문에 의해 나눠지는 두 분기의 코드가 비슷하다. 이 둘을 동일하게 만들고 나서 조건문을 제거한다.
    - 두 클래스가 비슷하다. 이 둘을 동일하게 만들고 나서 하나를 제거한다.

### 변화 격리하기

---

- 객체나 메서드의 일부만 바꾸려면 어떻게 해야할까? → 일단, 바꿔야 할 부분을 격리한다.
    - 하지만 생각없이 자동으로 변경하지는 마라.
    - 균형을 잡자: 코드에 메서드가 하나 더 명시되는 비용 vs 또 하나의 개념이 명시되는 가치

- 변화를 격리하기 위해 사용할 수 있는 몇 가지 방법
    - 메서드 추출하기(가장 일반적)
    - 객체 추출하기
    - 메서드 객체(Method Object)

### 데이터 이주시키기

---

- 표현 양식을 변경하려면 어떻게 해야할까? → 일시적으로 데이터를 중복시킨다.

- 방법
    - 내부에서 외부로의 변화란, 내부의 표현 양식을 변경한 후 외부 인터페이스를 변화시키는 방법이다.
        - 새 포맷의 인스턴스 변수를 추가한다.
        - 기존 포맷의 인스턴스 변수를 세팅하는 모든 부분에서 새 인스턴스 변수도 세팅하게 만든다.
        - 기존 변수를 사용하는 모든 곳에서 새 변수를 사용하게 만든다.
        - 기존 포맷을 제거한다.
        - 새 포맷에 맞게 외부 인터페이스를 변경한다.
    - API 를 먼저 변화시키기를 원할 때 방법.
        - 새 포맷으로 인자를 하나 추가한다.
        - 새 포맷 인자에서 이전 포맷의 내부적 표현양식으로 번역한다.
        - 이전 포맷 인자를 삭제한다.
        - 이전 포맷을 사용하는 것들을 새 포맷으로 바꾼다.
        - 이전 포맷을 지운다.

- 이유: '하나에서 여럿으로' 는 항상 데이터 이주시키기 문제를 만들어낸다.
    - 자바의 Vector/Enumerator 에서 Collection/Iterator 로 옮기는 작업같이, 서로 다른 프로토콜을 갖는 동등한 포맷의 데이터에 대해서도 위와 같은 단계적인 데이터 이주시키기를 적용할 수 있다.

### 메서드 추출하기

---

- 길고 복잡한 메서드를 읽기 쉽게 만들려면 어떻게 할까? → 긴 메서드의 일부분을 별도의 메서드로 분리해내고 이를 호출하게 한다.

- 방법: 메서드 추출하기는 사실 좀더 복잡한 원자적(atomic) 리팩토링의 한 가지다.
    1. 기존의 메서드에서 별도의 메서드로 분리할 수 있을만한 부분을 찾아낸다. 반복문 내부의 코드나 반복문 전체, 혹은 조건문의 가지들이 일반적인 후보다.
    2. 추출할 영역의 외부에서 선언된 임시 변수에 대해 할당하는 문장이 없는지 확인한다.
    3. 추출할 코드를 복사해서 새 코드에 붙인다.
    4. 원래 메서드에 있던 각각의 임시 변수와 매개 변수 중 새 메서드에서도 쓰이는게 있으면, 이들을 새 메서드의 매개 변수로 추가한다.
    5. 기존의 메서드에서 새 메서드를 호출한다.

- 이유
    - 복잡한 코드를 이해하고자 할 때 메서드 추출하기를 사용한다.
    - 일부 서로 비슷한 내용이 있는 두 메서드에서 중복을 제거하기 위해 메서드 추출하기를 사용하기도 한다.

- 메서드를 작은 조각으로 나누는 것은 때때로 그 정도가 지나칠수도 있다.
- 더 나아갈 방법이 보이지 않을 땐, 새로운 방식으로 메서드 추출하기를 하기 위해 일단 모든 코드를 한 자리에 모아놓고 메서드 인라인 리팩토링을 자주 사용한다.

### 메서드 인라인

---

- 너무 꼬여있거나 산재한 제어 흐름을 단순화하려면 어떻게 할까? → 메서드를 호출하는 부분을 호출될 메서드의 본문으로 교체한다.

- 방법
    1. 메서드를 복사한다.
    2. 메서드 호출하는 부분을 지우고 복사한 코드를 붙인다.
    3. 모든 형식(formal) 매개 변수를 실제(actual) 매개 변수로 변경한다. 예를 들어 reader.getNext() 같은 매개 변수를 전달했다면, 이를 지역 변수에 할당해줘야 할 것이다.(reader 내부 상태를 바꾸기 때문에)

### 인터페이스 추출하기

---

- 자바 오퍼레이션에 대한 2번째 구현을 추가하려면 어떻게 해야할까? → 공통되는 오퍼레이션을 담고 있는 인터페이스를 만들면 된다.

- 방법
    1. 인터페이스를 선언한다. 때론 새로 추가될 인터페이스의 이름으로 기존 클래스의 이름을 사용해야 하는 경우가 있는데, 그런 경우라면 인터페이스를 추가하기 전에 기존 클래스의 이름을 변경해줘야 한다.
    2. 기존 클래스가 인터페이스를 구현하도록 만든다.
    3. 필요한 메서드를 인터페이스에 추가한다. 필요하다면 클래스에 존재하는 메서드들의 가시성을 높여준다.
    4. 가능한 모든 곳의 타입 선언부에서 클래스 이름 대신 인터페이스 이름을 사용하게 바꾼다.

- 이유: 때때로, 인터페이스를 추출할 필요가 있을 경우.

### 메서드 옮기기

---

- 메서드를 원래 있어야 할 장소로 옮기려면 어떻게 해야할까? → 어울리는 클래스에 메서드를 추가해주고, 그것을 호출하게 하라.

- 방법
    1. 메서드를 복사한다.
    2. 원하는 클래스에 붙이고 이름을 적절히 지어준 다음 컴파일한다.
    3. 원래 객체가 메서드 내부에서 참조된다면, 원래 객체를 새 메서드의 매개 변수로 추가한다. 원래 객체의 필드들이 참조되고 있다면 그것들도 매개 변수로 추가한다. 만약 원래 객체의 필드들이 갱신된다면 포기해야 한다.
    4. 원래 메서드의 본체를 지우고, 그곳에 새 메서드를 호출하는 코드를 넣는다.

- 이유: 보증 안되는 예상을 발견하는 데에 탁월한 방법이기 때문이다.
    - before

        ```java
        public class Shape {
            ...
            int width = bounds.right() - bounds.left();
            int height = bounds.top() - bounds.bottom();
            int area = width * height;
            ...
        }
        ```

    - after

        ```java
        public class Shape {
            ...
            int area = bounds.area();
            ...
        }
        ```

        ```java
        public class Rectangle {
            public int area() {
                int width = this.right() - this.left();
                int height = this.top() - this.bottom();
                return width * height;
        		}
            ...
        }
        ```

- 메서드 옮기기의 훌륭한 3가지 속성
    - 코드에 대한 깊은 이해가 없더라도 언제 이 리팩토링이 필요한지 쉽게 알아낼 수 있다. 다른 객체에 대한 2개 이상의 메시지를 보내는 코드를 볼 때 마다 메서드 옮기기를 해주면 된다.
    - 리팩토링 절차가 빠르고 안전하다.
    - 리팩토링 결과가 종종 새로운 사실을 알려준다. "이렇게 되면 Rectangle 이 아무 계산도 하지 않게 되잖아? 아하 이게 더 좋군."

- 메서드의 일부분만 옮기고 싶을 때
    - 일단 메서드 추출하기를 한 후, 메서드를 옮기고 원래 클래스에 있던 추출된 부분을 다시 합치면 된다.
    - 옮겨진 메서드를 호출하는 코드만 들어갈 것이므로 합칠 때엔 한 줄이 될 것이다.

### 메서드 객체

---

- 여러 개의 매개 변수와 지역 변수를 갖는 복잡한 메서드를 어떻게 표현할까? → 메서드를 꺼내서 객체로 만든다.

- 방법
    - 메서드와 같은 매개 변수를 갖는 객체를 만든다.
    - 메서드의 지역 변수를 객체의 인스턴스 변수로 만든다.
    - 원래 메서드와 동일한 내용을 갖는 run()을 만든다.
    - 원래 메서드에서는 새로 만들어진 클래스의 인스턴스를 생성하고 run()을 호출한다.

- 이유
    - 메서드 객체는 시스템에 완전히 새로운 로직을 추가하고자 할 때 유용하다.
    - 메서드 객체는 메서드 추출하기를 적용할 수 없는 코드를 간결하게 만들기 위한 용도로도 적합하다.

### 매개 변수 추가

---

- 메서드에 매개 변수를 추가하려면?

- 방법
    1. 메서드가 인터페이스에 선언되어 있다면 일단 인터페이스에 매개 변수를 추가한다.
    2. 매개 변수를 추가한다.
    3. 컴파일 에러가 어딜 고쳐야 하는지 알려줄 것이다.

- 이유
    - 매개 변수를 추가하는 것은 종종 확장 단계다.
    - 하나의 데이터 표현을 다른 표현으로 변경하는 작업의 일부로 쓰이기도 한다.

### 메서드 매개 변수를 생성자 매개 변수로 바꾸기

---

- 하나 이상의 메서드 매개 변수를 생성자로 옮기려면 어떻게 할까?

- 방법
    1. 생성자에 매개 변수를 추가한다.
    2. 매개 변수와 같은 이름을 갖는 인스턴스 변수를 추가한다.
    3. 생성자에서 인스턴스 변수의 값을 설정한다.
    4. 'parameter' 를 'this.parameter' 로 하나씩 찾아 바꾼다.
    5. 매개 변수에 대한 참조가 더이상 존재하지 않으면 해당 매개 변수를 메서드와 모든 호출자에서 제거한다.
    6. 필요 없어진 'this.' 를 제거한다.
    7. 변수명을 적절히 변경한다.

- 이유
    - 동일 매개 변수를 같은 객체의 서로 다른 몇몇 메서드로 전달하는 경우, 매개 변수를 1번만 전달하게끔 API 를 단순화 할 수 있다.(중복 제거)
    - 만약 인스턴스 변수가 오직 하나의 메서드에서만 쓰이는 경우라면 이 리팩토링을 반대로 수행할 수도 있다.


## 32장. TDD 마스터하기

- TDD 를 각자의 습관에 통합시켜 나가는 과정에서 숙고해볼 수 있는 몇 가지 질문

- 단계가 얼마나 커야 하나?
    - 각  테스트가 다뤄야 할 범위는 얼마나 넓은가?
    - 리팩토링을 하면서 얼마나 많은 중간 단계를 거쳐야 하는가?
    - 테스트 주도 개발자의 경향: 단계가 점점 작아지는 것이다.
    - 리팩토링 초기에는 아주 작은 단계로 작업할 준비가 되어있어야 한다.

- 테스트할 필요가 없는 것은 무엇인가?
    - 두려움이 지루함으로 변할때까지 테스트를 만들어라.
    - 테스트 시도 목록
        - 조건문
        - 반복문
        - 연산자
        - 다형성
    - 당신이 작성하는 것들에 대해서만 테스트해라.
        - 불신할 이유가 없다면 다른 사람이 만든 코드를 테스트하지 마라.

- 좋은 테스트를 갖췄는지의 여부를 어떻게 알 수 있는가?
    - 테스트란 탄광 속에서 자신의 고통을 통해 고약한 설계 가스의 존재를 드러내는 카나리아다.
    - 설계 문제가 있음을 알려주는 테스트의 속성
        - 긴 셋업 코드
            - 하나의 단순한 단언을 수행하기 위해 수백줄의 객체 생성 코드가 필요하다면 객체가 너무 크다는 뜻이므로 나뉠 필요가 있다.
        - 셋업 중복
            - 공통의 셋업 코드를 넣어 둘 공통의 장소를 찾기 힘들다면, 서로 밀접하게 엉킨 객체들이 너무 많다는 뜻이다.
        - 실행 시간이 오래 걸리는 테스트
            - 테스트의 실행 시간이 길다는 것은 애플리케이션의 작은 부분만 테스트할 수 없다는 것이다.
            - 설계 문제를 의미한다.
        - 깨지기 쉬운 테스트
            - 애플리케이션의 특정 부분이 다른 부분에 이상한 방법으로 영향을 끼친다는 뜻이다.
            - 연결을 끊거나 두 부분을 합하는 것을 통해 멀리 떨어진 것의 영향력이 없어지도록 설계해야 한다.

- TDD 로 프레임워크를 만들려면 어떻게 해야 하나?
    - 코드의 미래에 대해 고려하지 않음으로 인해, 코드가 더 뛰어난 적응성을 가질 수 있게 한다.
    - TDD 는 내일을 위해 코딩하고, 오늘을 위해 설계한다.
        - 첫 번째 기능을 구현한다. 이 첫 번째 기능은 단순하고 직관적으로 구현되고, 따라서 짧은 시간 안에 결함도 적은 상태로 완성된다.
        - 첫 번째 기능에 대한 변주가 되는 두 번째 기능을 구현한다. 두 기능 사이의 중복이 한 곳으로 모이고, 서로 다른 부분은 다른 곳(메서드, 클래스)으로 옮겨진다.
        - 앞의 두 기능에 대한 변주로 세 번째 기능을 구현한다. 공통의 로직은 약간의 수정만을 통해 재활용 가능한 상태로 만들어질 수 있을 것이다. 그리고 공통적이지 않은 로직들은 다른 메서드, 클래스 등 명확하게 로직이 있어야 할 곳에 있게 되는 경향이 있다.
    - 개방-폐쇄 원칙: 객체는 사용에 대해서는 열려있어야 하고, 향후 수정에 대해서는 닫혀있어야 한다.
        - 실제로 발생하는 변주들에 대해서 서서히 지켜져 간다.
        - 테스트 주도 개발은 비록 발생하지 않은(혹은 아직 발생하지 않은) 변주 종류는 잘 표현하지 못할지라도.
        - 발생하는 변주 종류들을 잘 표현하는 프레임워크를 만들게 해준다.
    - 뭔가를 잘못하지 않았다는 확신을 줄 수 있는 수 많은 테스트들이 존재하기 때문이다.

- 피드백이 얼마나 필요한가?
    - 켄트 백은 테스트를 얼마나 작성할지 고려할 때, 실패간 평균시간을 생각한다.
    - 정말 일어날 것 같지 않은 조건과 그런 조건의 결합을 테스트하는 것은, 나름대로 의미가 있다.
    - TDD 의 테스트에 대한 관점은 실용적이다.
        - TDD 에서 테스트는 우리가 깊이 신뢰할 수 있는 코드를 위한 하나의 수단이다.
        - 만약 어떤 구현에 대한 지식이 신뢰할만 하다면 그에 대한 테스트는 작성하지 않을 것이다.
    - 의도적으로 구현을 무시하는 블랙 박스 테스팅의 이점: 코드를 무시함으로써 하나의 다른 가치 체계를 드러낸다.

- 테스트를 지워야 할 때는 언제인가?
    - 서로 겹치는 2개의 테스트가 있을 때 삭제 결정 기준
        - 첫째 기준은 자신감. 테스트를 삭제할 경우 자신감이 줄어들 것 같으면 절대 테스트를 지우지 말아야 한다.
        - 둘째 기준은 커뮤니케이션. 2개의 테스트가 코드의 동일한 부분을 실행하더라도, 이 둘이 서로 다른 시나리오를 말한다면 그대로 남겨두어야 한다.

- 프로그래밍 언어나 환경이 TDD 에 어떤 영향을 주는가?
    - TDD 주기(테스트/컴파일/실행/리팩토링)를 수행하기가 힘든 언어나 환경에서 작업하게 되면 단계가 커지는 경향이 있다.
        - 각 테스트가 더 많은 부분을 포함하게 만든다.
        - 중간 단계를 덜 거치고 리팩토링을 한다.

- 거대한 시스템을 개발할 때에도 TDD 를 할 수 있는가?
    - 시스템에 있는 기능의 양은 TDD 효율에 영향을 미치지 않는다.
    - 중복을 제거함에 따라 더 작은 객체들이 만들어지게 되고, 이 작은 객체들은 애플리케이션 크기와 무관하게 독립적으로 테스트될 수 있다.

- 애플리케이션 수준의 테스트로도 개발을 주도할 수 있는가?
    - ATDD..
    - 켄트 백은 앞으로도 프로그래머 수준의 TDD 를 원할 것 같다고 한다.
        - 즉시 초록 막대를 볼 수 있고
        - 내부 설계를 단순화할 수 있도록

- 프로젝트 중반에 TDD 를 도입하려면 어떻게 해야 할까?
    - 테스트를 염두에 두지 않고 만든 코드는 테스트하기가 그리 쉽지 않다.
    - 확실히 하지 말아야 할 일: 코드 전체를 위한 테스트를 한꺼번에 다 만들고, 한번에 리팩토링 하는 일이다.
        - 이런 작업은 몇 달이나 걸릴텐데, 그동안 아무런 새 기능도 추가로 구현하지 못할 것이다.
        - 일반적으로 지속 가능한 프로세스가 아니다.
    - 해야 할 일
        - 우선, 변경의 범위를 제한한다.
            - 시스템에서 극적으로 단순화될 수 있지만 지금 당장 변할 필요가 없는 부분을 봤다면, 그냥 그대로 놔두자.
        - 다음으로 테스트와 리팩토링 사이에 존재하는 교착상태(deadlock)를 풀어주는 것이다.
            - 테스트가 아닌 다른 방법으로도 피드백을 얻을 수 있다.
                - 아주 조심스럽게 작업하는 방법이나 파트너와 함께 작업하는 방법 등
            - 이 피드백을 이용해, 우리가 바꾸어야 하는 부분이 변화에 좀더 수용적이 되도록 할 수 있다.

- TDD 는 초기 조건에 민감한가?
    - 빨강/초록/리팩토링: 테스트를 취할 때 이 순서로 하면 매우 매끄럽게 잘 넘어가는 것 같다.

- TDD 와 패턴의 관계는?
    - 켄트 백은 비슷하게 흉내낼 전문가를 1명 찾고는 무슨 일이 벌어지는지 점차 알아낸다.
        - 반복적 행동을 규칙으로 환원함으로써 규칙을 적용하는 것은 기계적이며 단순 암기가 된다.

- 어째서 TDD 가 잘 작동하는가?
    - 결함 감소: 결함을 빨리 발견해 고칠수록 비용은 낮아진다.
    - 설계 결정에 대한 피드백 고리를 단축시킨다.


- [Notion link](https://www.notion.so/Test-Driven-Development-d1bcd929915b4093a809bdac39766a48)
