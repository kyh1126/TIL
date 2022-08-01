# 읽기 좋은 코드가 좋은 코드다

## 1. 코드는 이해하기 쉬워야 한다

- 무엇이 코드를 '더 좋게' 만드는가? → 가독성
- 가독성의 기본 정리: 가독성과 관련한 가장 중요한 통계적 사실
    - 이해를 위한 시간: 다른 사람이 코드를 이해하는 데 들이는 시간
- 분량이 적으면 항상 더 좋은가? → 이해를 위한 시간을 최소화하는게 더 좋은 목표다.


# Part 1. 표면적 수준에서의 개선

## 2. 이름에 정보 담기

- 특정한 단어 고르기: get 대신 fetch 나 download 를 사용하는 것이 더 나을 수 있다.
    - 매우 구체적인 단어를 선택하여 무의미한 단어를 피하는 것
        - stop() 대신 kill(), pause()
    - 더 화려한 단어 고르기

        ![Untitled.png](theartofreadablecode/Untitled.png)

    - 재치 있는 이름보다 명확하고 간결한 이름이 더 좋다.

- 보편적인 이름 피하기 (혹은 언제 그런 이름을 사용해야 하는지 깨닫기)
    - tmp: 대상이 짧게 임시적으로만 존재하고, 임시적 존재 자체가 변수의 가장 중요한 용도일 때에 한해서 사용해야 한다.
    - retval: 정보를 제대로 담고 있지 않다. 대신 변수값을 설명하는 이름을 사용하라.
    - 루프반복자의 i, it 대신 더 명확한 의미를 드러내는 이름을 사용하자.

    → 보편적인 이름을 사용하려면, 꼭 그렇게 해야하는 이유가 있어야 한다.

- 추상적인 이름 대신 구체적인 이름 사용하기
    - serverCanStart() 보다 canListenOnPort()

- 접두사 혹은 접미사로 이름에 추가적인 정보 덧붙이기
    - 단위를 포함하는 값들: start 대신 start_ms

        ![Untitled1.png](theartofreadablecode/Untitled%201.png)

    - 다른 중요한 속성 포함하기: 위험한 요소 혹은 나중에 놀랄만한 내용이 있다면 변수명에 이를 포함해야 한다. 이스케이핑을 수행하는 변수 앞에 raw_ 를 붙인다.

        ![Untitled2.png](theartofreadablecode/Untitled%202.png)

- 이름이 얼마나 길어져도 좋은지 결정하기
    - 좁은 범위에서는 짧은 이름이 괜찮다.
    - 긴 이름 입력하기 - 더 이상 문제가 되지 않는다.
    - 약어와 축약형 기준
        - 축약이 나중에 일어날지 모르는 혼란을 무릅쓸만한 가치가 있을까?
        - 팀에 새로 합류한 사람이 의미하는 바를 이해할 수 있으면 그 이름은 괜찮은 것이다.
    - 불필요한 단어 제거하기: convertToString() 대신 toString()

- 추가적인 정보를 담을 수 있게 이를 구성하기
    - 밑줄과 대시, 대문자를 잘 이용하면 이름에 더 많은 정보를 담을 수 있다.
    - 클래스 멤버를 로컬 변수와 구분하기 위해 _ 를 붙일 수 있다.
    - 언어별 포맷팅 관습을 지키자.


## 3. 오해할 수 없는 이름들

- 본인이 지은 이름을 '다른 사람들이 다른 의미로 해석할 수 있을까?' 질문을 던져보며 철저하게 확인해야 한다.
    - filter(): 대상을 고르는 것인지 제거하는 것인지 불분명하다. select(), exclude() 가 더 낫다.

- 경계를 포함하는 한계값을 다룰 때는 min 과 max 를 사용하라: MIN_ITEMS_IN_CART
- 경계를 포함하는 범위에는 first, last 를 사용하라

    ![Untitled3.png](theartofreadablecode/Untitled%203.png)

- 경계를 포함하고/배제하는 범위에는 begin, end 를 사용하라

    ![Untitled4.png](theartofreadablecode/Untitled%204.png)

- 불리언 변수에 이름 붙이기: true, false 가 각각 무엇을 의미하는지 명확해야 한다.
    - 일반적으로 is, has, can, should 같은 단어를 더하면 의미가 더 명확해진다.
    - 의미를 부정하는 용어(disable_ssl)를 피하는 것이 좋다.
- 사용자의 기대에 부응하기: 사용자가 어떤 이름의 의미를 특정한 방식으로 이해해서 실제로 다른 의미가 있음에도 오해를 초래할 때가 있는데, 그것이 일반적인 의미를 갖도록 하는게 좋다.


## 4. 미학

- 좋은 소스코드는 '눈을 편하게' 해야 한다. 미학적으로 보기 좋은 코드가 사용하기 더 편리하다.
- 읽기 편한 소스코드를 작성하는 방법
    - 코드를 읽는 사람이 이미 친숙한, 일관성 있는 레이아웃을 사용하라.
        - 의미있는 순서를 선택하고 일관성 있게 사용하라: 코드의 순서가 코드의 정확성에 아무런 영향을 미치지 않는 경우, 한 곳에서 언급된 순서를 지키자.

            ![Untitled5.png](theartofreadablecode/Untitled%205.png)

            - 변수의 순서를 HTML 폼에 있는 `<input>` 필드의 순서대로 나열하라.
            - '가장 중요한 것'에서 시작해서 '가장 덜 중요한 것'까지 순서대로 나열하라.
            - 알파벳 순서대로 나열하라.
        - 개인적인 스타일 대 일관성: 스타일이 뒤섞이면 가독성에 영향을 준다. 일관성 있는 스타일은 '올바른' 스타일보다 더 중요하다.
    - 비슷한 코드는 서로 비슷해 보이게 만들어라.
        - 일관성과 간결성을 위해서 줄 바꿈을 재정렬하기

            ![Untitled6.png](theartofreadablecode/Untitled%206.png)

            - 주석 반복 제거하고자 좀 더 간결하게 개선해 볼 수 있다.

                ![Untitled7.png](theartofreadablecode/Untitled%207.png)

        - 메소드를 활용하여 불규칙성을 정리하라. 헬퍼 메소드 적용 효과.
            - 중복된 코드를 없애서 코드를 더 간결하게 한다.
            - 이름이나 에러 문자열 같은 테스트의 중요 부분들이 한 눈에 보이게 모아졌다. 수정 전에는 database_connection 이나 error 같은 토큰들과 섞인 채 흩어져 있었기 때문에 코드를 한 눈에 파악하기 어려웠다.
            - 새로운 테스트 추가가 훨씬 쉬워졌다.
        - 도움이 된다면 코드의 열을 맞춰라

            ![Untitled8.png](theartofreadablecode/Untitled%208.png)

            ![Untitled9.png](theartofreadablecode/Untitled%209.png)

            ![Untitled10.png](theartofreadablecode/Untitled%2010.png)

    - 서로 연관된 코드는 하나의 블록으로 묶어라.
        - 선언문을 블록으로 구성하라
            - before

                ![Untitled11.png](theartofreadablecode/Untitled%2011.png)

            ![Untitled12.png](theartofreadablecode/Untitled%2012.png)

        - 코드를 '문단'으로 쪼개라
            - before

                ![Untitled13.png](theartofreadablecode/Untitled%2013.png)

            ![Untitled14.png](theartofreadablecode/Untitled%2014.png)

            - 각 문단의 주석 처리는 사용자가 코드를 훑어보는 데 도움을 준다.


## 5. 주석에 담아야 하는 대상

- 주석의 목적: 코드를 읽는 사람이 코드를 작성한 사람만큼 코드를 잘 이해하게 돕는다.
- 설명하지 말아야 하는 것
    - 코드에서 빠르게 유추할 수 있는 내용은 주석으로 달지 말라.
        - 설명 자체를 위한 설명을 달지 말라.
        - before

            ![Untitled15.png](theartofreadablecode/Untitled%2015.png)

        ![Untitled16.png](theartofreadablecode/Untitled%2016.png)

        - 함수의 선언과 주석 내용이 실질적으로 일치하는 경우, 주석을 삭제하거나 더 중요한 세부 사항을 적는 것이 낫다.
    - 나쁜 이름에 주석을 달지 마라 - 대신 이름을 고쳐라.
        - before

            ![Untitled17.png](theartofreadablecode/Untitled%2017.png)

        ![Untitled18.png](theartofreadablecode/Untitled%2018.png)

        - 좋은 이름은 함수가 사용되는 모든 곳에서 드러나므로 좋은 주석보다 더 낫다.
- 코딩을 수행하면서 머릿속에 있는 정보를 기록하기.
    - '감독의 설명'을 포함하라: 코드가 특정한 방식으로 작성된 이유를 설명해주는 내용
        - 중요한 통찰을 기록한 주석을 코드에 포함시켜야 한다.
        - 코드를 최적화 하느라 시간을 허비하지 않게 도와준다.

            ![Untitled19.png](theartofreadablecode/Untitled%2019.png)

        - 오해의 소지로 인한 시간 허비 방지

            ![Untitled20.png](theartofreadablecode/Untitled%2020.png)

        - 코드가 왜 훌륭하지 않은지도 설명할 수 있다.

            ![Untitled21.png](theartofreadablecode/Untitled%2021.png)

            - 다음 사람에게 어떻게 수정해야 하는지 알려준다. 만약 이 주석이 없으면 많은 사람이 이 코드에 겁을 먹어 건드리지 않으려고 할 것이다.
    - 코드에 있는 결함을 설명하라
        - 코드는 지속적으로 진화하며, 그 과정 중에 버그를 갖게될 수 밖에 없다. 결함을 설명하는 것을 부끄러워할 필요는 없다.
        - 개선 아이디어 설명하기
            - 다음과 같이 개선이 필요할 때

                ![Untitled22.png](theartofreadablecode/Untitled%2022.png)
              
            - 코드가 불완전할 때

                ![Untitled23.png](theartofreadablecode/Untitled%2023.png)

        ![Untitled24.png](theartofreadablecode/Untitled%2024.png)

    - 상수에 대한 설명
        - 상수에 별도의 설명이 필요한 경우

            ![Untitled25.png](theartofreadablecode/Untitled%2025.png)

        - 상수의 특정한 값이 아무런 의미를 갖지 않음을 알려주는 주석

            ![Untitled26.png](theartofreadablecode/Untitled%2026.png)

        - 신중하게 설정되었으니 변경하지 마라 표시

            ![Untitled27.png](theartofreadablecode/Untitled%2027.png)

- 코드를 읽는 사람 입장에서 필요한 정보가 무엇인지 유추하기.
    - 나올 것 같은 질문 예측하기

        ![Untitled28.png](theartofreadablecode/Untitled%2028.png)

    - 사람들이 쉽게 빠질 것 같은 함정을 경고하라
        - 함수의 '세부 사항'을 설명하는 주석

            ![Untitled29.png](theartofreadablecode/Untitled%2029.png)

        - 함수의 단점을 주석으로 미리 알려주자

            ![Untitled30.png](theartofreadablecode/Untitled%2030.png)

    - '큰 그림'에 대한 주석
        - 상위 레벨 주석

            ![Untitled31.png](theartofreadablecode/Untitled%2031.png)

        - 파일 레벨 주석

            ![Untitled32.png](theartofreadablecode/Untitled%2032.png)

    - 요약 주석
        - 함수 내부에서 '큰 그림'을 설명하는 방식.
        - 더 아래의, 하위 레벨 주석: 코드의 내용을 간결하게 요약한다.

            ![Untitled33.png](theartofreadablecode/Untitled%2033.png)


## 6. 명확하고 간결한 주석 달기

- 주석은 높은 '정보 대 공간' 비율을 갖춰야 한다.
    - 주석을 간결하게 하라
    - 모호한 대명사는 피하라. ex> it, this
        - 혼동의 여지가 조금이라도 있으면 대명사를 원래 명사로 대체하자.
    - 엉터리 문장을 다듬어라
        - before: `// 이 URL을 전에 이미 방문했는지에 따라서 다른 우선순위를 부여한다.`
        - `// 전에 방문하지 않은 URL에 높은 우선순위를 부여하라.`
    - 함수의 동작을 명확하게 설명하라
        - before: `// 이 파일에 담긴 줄 수를 반환한다.`
        - `// 파일 안에 새 줄을 나타내는 바이트('\n')가 몇 개 있는지 샌다.`
    - 코너케이스를 설명해주는 입/출력 예를 사용하라
        - 주석을 작성하는 데 신중하게 선택된 입/출력 예는 천 마디 말보다 위력적이다.
        - 지나치게 간단한 입출력 예는 별로 유용하지 않다.
        - before

            ![Untitled34.png](theartofreadablecode/Untitled%2034.png)

        ![Untitled35.png](theartofreadablecode/Untitled%2035.png)

    - 코드의 의도를 명시하라
        - before

            ![Untitled36.png](theartofreadablecode/Untitled%2036.png)

        ![Untitled37.png](theartofreadablecode/Untitled%2037.png)

    - 이름을 가진 함수 파라미터(Named Function Parameter) 주석
        - 의미가 불분명한 함수의 인수를 설명하라.

        ![Untitled38.png](theartofreadablecode/Untitled%2038.png)

    - 정보 축약형 단어를 사용하라
        - before

            ![Untitled39.png](theartofreadablecode/Untitled%2039.png)

        ![Untitled40.png](theartofreadablecode/Untitled%2040.png)


# Part 2. 루프와 논리를 단순화하기

## 7. 읽기 쉽게 흐름제어 만들기

- 흐름을 제어하는 조건과 루프 그리고 여타 요소를 최대한 ‘자연스럽게’ 만들도록 노력하라.
    - 코드를 읽다가 다시 되돌아가서 코드를 읽지 않아도 되게끔 만들어야 한다.

- 조건문에서 인수의 순서
    - 왼쪽: 값이 더 유동적인 ‘질문을 받는’ 표현
    - 오른쪽: 더 고정적인 값으로, 비교대상으로 사용되는 표현
    - ex> `if (length > 10)`
- `if/else` 블록의 순서
    - 부정이 아닌 긍정을 다루어라.
        - 즉 `if(!debug)`가 아니라 `if(debug)`를 선호하자.
    - 간단한 것을 먼저 처리하라.
        - 이렇게 하면 동시에 같은 화면에 `if`와 `else`구문을 나타낼 수도 있다.
        - 두 개의 주문을 동시에 보는 게 더 좋다.
    - 더 흥미롭고, 확실한 것을 먼저 다루어라.
        - before

          ![Untitled41.png](theartofreadablecode/Untitled%2041.png)

      ![Untitled42.png](theartofreadablecode/Untitled%2042.png)

    - 그러나, 부정부터 다루어야 더 단순하고 흥미로우면서 동시에 위험해지는 경우도 있다.

      ![Untitled43.png](theartofreadablecode/Untitled%2043.png)

        - 이 경우에는 상세 내용을 따지고 난 뒤 판단을 내려야 한다.
- 삼항 연산자(`?:`)를 이용하는 조건문 표현
    - 삼항 연산자가 읽기 편하고 간결한 경우: 매우 간단할 때만 사용해야 한다.
        - before

            ```java
            if (hour >= 12) {
                time_str += "pm";
            } else {
                time_str += "am";
            }
            ```

        ```java
        time_str += (hour >= 12) ? "pm" : "am";
        ```

    - 복잡해지는 경우: 간단한 두 값에서 선택하는 문제가 아닌 경우.

        ```java
        return exp >= 0 ? m + (1 << exp) : m / (1 << -exp);
        ```

        - 코드를 이렇게 작성하는 것은 그저 '모든 것을 한 줄에 쓰기'다. → `if/else`문이 낫다.
- `do/while` 루프를 피하라: 조건이 '눈에 뜨이는 곳에 미리' 나타나는 것이 좋다.
    - `if`, `while`, `for`문의 동작 원리는 모두 코드를 위에서 아래로 읽는다.
    - 그러나 `do/while`은 역순이라 코드를 두 번 읽어야 해서 부자연스럽다.
- 함수 중간에서 반환하기
    - 함수 중간에서 반환하는 것이 바람직할 때도 있어, 완전히 허용되어야 한다.

      ![Untitled44.png](theartofreadablecode/Untitled%2044.png)

        - 위의 함수를 중간에서 반환하는 보호장치 없이 구현하면 매우 부자연스러워질 것이다.
    - 반환 포인트를 하나만 두려는 건 함수의 끝부분에서 실행되는 클린업(cleanup) 코드의 호출을 보장하려는 의도다.
        - 언어 차원에서 제공하는 더 정교한 클린업 코드를 실행시키는 방법을 사용하자.

      ![Untitled45.png](theartofreadablecode/Untitled%2045.png)

- 악명 높은 `goto`는 피하는 게 낫다.
- 중첩을 최소화하기
    - 코드의 중첩이 일어날 때마다 코드를 읽는 사람의 '정신적 스택'에 추가적인 조건이 입력된다.
    - 함수 중간에서 반환하여 중첩을 제거하라
        - before

          ![Untitled46.png](theartofreadablecode/Untitled%2046.png)

      ![Untitled47.png](theartofreadablecode/Untitled%2047.png)

    - 루프 내부에 있는 중첩 제거하기 - 루프 중간에서 반환하기
        - before

          ![Untitled48.png](theartofreadablecode/Untitled%2048.png)

      ![Untitled49.png](theartofreadablecode/Untitled%2049.png)

        - `if/return`을 함수의 보호 장치로 사용했듯이, `if/continue`도 루프의 보호 장치로 사용할 수 있다.

- 실행 흐름을 따라올 수 있는가?
    - 눈에 보이는 코드의 '뒤에서' 실행되는 몇 가지

      ![Untitled50.png](theartofreadablecode/Untitled%2050.png)

  → 이러한 구조가 차지하는 비율이 너무 높지 않아야 한다.


## 8. 거대한 표현을 잘게 쪼개기

- 설명 변수(추가 변수)
    - 커다란 표현을 쪼개는 가장 쉬운 방법은 작은 하위표현을 담을 추가 변수를 만드는 것이다.
    - before

        ```python
        if line.split(':')[0].strip() == "root":
            ...
        ```

    ```python
    username = line.split(':')[0].strip()
    if username == "root":
        ...
    ```

- 요약 변수
    - 커다란 코드 덩어리를 짧은 이름으로 대체하여 더 쉽게 관리하고 파악하는 목적을 가진 변수
    - before

        ```java
        if (request.user.id == document.owner_id) {
            // 사용자가 이 문서를 수정할 수 있다...
        }
        ...
        if (request.user.id != document.owner_id) {
            // 무넛는 읽기전용이다...
        }
        ```

    - `request.user.id == document.owner_id`는 변수 5개를 담고 있어, 이 표현을 읽으려면 추가적인 시간이 필요하다.

    ```java
    final boolean user_owns_document = (request.user.id == document.owner_id);

    if (user_owns_document) {
        // 사용자가 이 문서를 수정할 수 있다...
    }
    ...
    if (!user_owns_document) {
        // 무넛는 읽기전용이다...
    }
    ```

    - user_owns_document 라는 표현을 맨 위에 두어 코드를 읽는 사람에게 “이것이 바로 이 함수에서 생각해야 하는 주된 개념이로군” 하는 생각이 들게 한다.
- 드모르간의 법칙 사용하기
    - 동일한 불리언 표현은 다음과 같이 두 방법으로 작성할 수 있다.
    1. `not (a or b or c)` == `(not a) and (not b) and (not c)`
    2. `not (a and b and c)` == `(not a) or (not b) or (not c)`
    - before

        ```java
        if (!(file_exists && !is_protected)) Error("sorry, can't read file");
        ```

    ```java
    if (!file_exists || is_protected) Error("sorry, can't read file");
    ```

- 쇼트 서킷 논리 오용하지 말기
    - 대부분의 프로그래밍 언어에서 불리언 연산은 쇼트 서킷 평가를 수행한다.
    - before

        ```java
        assert((!(bucket = FindBucket(key))) || !bucket -> IsOccupied());
        ```

    ```java
    bucket = FindBucket(key);
    if(bucket != NULL) assert(!bucket -> IsOccupied());
    ```

    - before 의 경우, 프로그래머는 의미를 이해하기 위해 손을 멈추고 생각해야 한다. 동일한 일을 수행하고 코드가 두 줄로 늘어났지만, 훨씬 이해하기 쉬워졌다.
- 예: 복잡한 논리와 씨름하기
    - before

        ```java
        return (begin >= other.begin && begin < other.end) ||
               (end > other.begin && end <= other.end) ||
               (begin <= other.begin && end >= other.end);
        ```

    ```java
    bool Range::OverlapsWith(Range other) {
        if (other.end <= begin) return false; // 우리가 시작하기 전에 끝난다.
        if (other.begin >= end) return false; // 우리가 끝난 후에 시작한다.

        return true;
    }
    ```

    - 똑같은 문제를 ‘반대되는’ 방법으로 해결할 수 있는지 확인해보자.
- 거대한 구문 나누기
    - 조건부 내의 동일한 부분을 요약 변수로 추출해보자.


## 9. 변수와 가독성

- (가독성에 도움되지 않는) 변수 제거하기
    - 불필요한 임시 변수들

        ```java
        **now** = datetime.datetime.now()
        root_message.last_view_time = now
        ```

        - 복잡한 표현을 잘게 나누지 않는다.
        - 명확성에 도움이 되지 않는다. datetime.datetime.now()는 그 자체로 명확하다.
        - 한 번만 사용되어 중복된 코드를 압축하지 않는다.
    - 중간 결과 삭제하기
        - before

            ```java
            var remove_one = function (array, value_to_remove) {
                var index_to_remove = null;
                for (var i = 0; i < array.length; i += 1) {
                    if (array[i] === value_to_remove) {
                        index_to_remove = i;
                        break;
                    }
                }
                if (index_to_remove !== null) {
                    array.splice(index_to_remove, 1);
                }
            }
            ```

        ```java
        var remove_one = function (array, value_to_remove) {
            for (var i = 0; i < array.length; i += 1) {
                if (array[i] === index_to_remove) {
                    array.splice(i, 1);
                    return;
                }
            }
        }
        ```

        - 할 수만 있따면 이처럼 함수를 최대한 빨리 반환하는 게 좋다.
    - 흐름 제어 변수 제거하기

        ```java
        boolean done = false;

        while (/* 조건 */ && !done) {
            ...

            if (...) {
                done = true;
                continue;
            }
        }
        ```

        - 흐름 제어 변수: 순수하게 프로그램의 실행 방향을 설정한다.
            - 데이터를 저장하지 않는다.
            - done 같은 변수

        ```java
        while (/* 조건 */) {
            ...
            if (...) {
                break;
            }
        }
        ```

        - 위와 같이 수정할 수 있다. 복잡하면 루프 안에서 반복되는 코드를 새로운 함수로 만들면 된다.
- 변수의 범위를 좁혀라
    - 변수가 적용되는 범위를 최대한 좁게 만들어라.
    - 많은 메소드를 정적 static 으로 만들어서 클래스 멤버 접근을 제한해라.
        - 정적 메소드는 코드를 읽는 사람에게 ‘이 코드는 저 변수들로부터 독립적’이라는 사실을 알려주는 매우 좋은 방법이다.
    - 커다란 클래스를 여러 작은 클래스로 나누는 방법
        - 작은 클래스들이 서로 독립적일 때 유용하다.
    - 정의를 아래로 옮기기
        - 많은 변수를 가지고 있는 긴 함수일 때, 지금 당장 사용되지 않는 변수조차 일단 염두에 두게 강제하므로 좋지 않다.
        - 각각의 정의를 실제로 사용하기 바로 직전 위치로 옮기는 게 좋다.
- 값을 한 번만 할당하는 변수를 선호하라
    - 상수는 코드를 읽는 사람에게 별다른 추가적인 생각을 요구하지 않는다. 자바에서 `final` 사용을 권장한다.
    - 변수에 값을 한 번만 할당하게 할 수 없더라도, 최대한 적은 횟수로 변하게 하는 일은 도움이 된다.


# Part 3. 코드 재작성하기

## 10. 상관없는 하위문제 추출하기

- 주어진 함수나 코드 블록을 보고, ‘상위수준에서 본 이 코드의 목적은 무엇인가?’ 질문하라.
- 코드의 모든 줄에 ‘이 코드는 직접적으로 목적을 위해서 존재하는가? 혹은 목적을 위해서 필요하긴 하지만 목적 자체와 직접적으로 상관없는 하위문제를 해결하는가?’ 질문하라.
- 만약 상당히 원래의 목적과 직접적으로 관련되지 않은 하위문제를 해결하는 코드 분량이 많으면, 이를 추출해서 별도의 함수로 만든다.
- 예: findClosestLocation()
    - before

        ```java
        // 'array'의 어느 요소가 주어진 위/경도에 가장 가까운지 찾아서 반환한다.
        // 지구를 완전한 구로 모델링한다.
        var findClosestLocation = function (lat, lng, array) {
            var closest;
            var closest_dist = Number.MAX_VALUE;

            for (var i = 0; i < array.length; i += 1) {
                // 두 점 모두를 라디안으로 변환한다.
                var lat_rad = radians(lat);
                var lng_rad = radians(lng);
                var lat2_rad = radians(array[i].latitude);
                var lng2_rad = radians(array[i].longitude);

                // '코사인의 특별법칙' 공식을 사용한다.
                var dist = Math.acos(Math.sin(lat_rad) * Math.sin(lat2_rad) + Math.cos(lat_rad) * Math.cos(lat2_rad) * Math.cos(lng2_rad - lng_rad));

                if (dist < closest_dist) {
                    closest = array[i];
                    closest_dist = dist;
                }
            }
            return closest;
        }
        ```

    - 루프 내부의 상관없는 하위문제(구 위의 두 개의 위/경도 점 사이의 거리 계산)를 별도의 함수로 추출하자.

        ```java
        var spherical_distance = function (lat1, lng1, lat2, lng2) {
                // 두 점 모두를 라디안으로 변환한다.
                var lat1_rad = radians(lat1);
                var lng1_rad = radians(lng1);
                var lat2_rad = radians(lat2);
                var lng2_rad = radians(lng2);

                // '코사인의 특별법칙' 공식을 사용한다.
                return Math.acos(Math.sin(lat1_rad) * Math.sin(lat2_rad) + Math.cos(lat1_rad) * Math.cos(lat2_rad) * Math.cos(lng2_rad - lng1_rad));
        }
        ```

        ```java
        // 'array'의 어느 요소가 주어진 위/경도에 가장 가까운지 찾아서 반환한다.
        // 지구를 완전한 구로 모델링한다.
        var findClosestLocation = function (lat, lng, array) {
            var closest;
            var closest_dist = Number.MAX_VALUE;

            for (var i = 0; i < array.length; i += 1) {
                var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longitude);
                if (dist < closest_dist) {
                    closest = array[i];
                    closest_dist = dist;
                }
            }
            return closest;
        }
        ```

        - 상위수준의 목적에 집중할 수 있어 전반적으로 코드의 가독성이 높아졌다.
        - spherical_distance()는 독립적인 테스트도 더 용이하다.
- 순수한 유틸리티 코드
    - 문자열 변경, 해시테이블 사용, 파일 읽기/쓰기 등 ‘기본적인 유틸리티’는 해당 프로그래밍 언어에 내장된 라이브러리에 있다.
    - 유틸리티 코드에 존재하지 않는 직접 상관없는 하위 문제를 다루는 함수라면, 다른 프로젝트에서도 사용할 수 있는 별도의 유틸리티 코드 모음을 만들자.
- 일반적인 목적의 코드
    - before

        ```java
        ajax_post({
            url: "http://example.com/submit",
            data: data,
            on_success: function (response_data) {
                var str = "{\n";
                for (var key in response_data) {
                    str += " " + key + " = " + response_data[key] + "\n";
                }
                alert(str + "}");
            }
        });
        ```

    - 이 코드의 상위수준 목적은 서버에 Ajax 호출을 하고 응답을 처리하는 것이다.
    - 이러한 목적과 직접 상관없는 하위문제(딕셔너리에 담긴 내용을 예쁘게 출력하는 일)를 추출해서 format_pretty(obj) 함수로 만들자.

        ```java
        var format_pretty = function (obj) {
            var str = "{\n";
            for (var key in obj) {
                str += " " + key + " = " + obj[key] + "\n";
            }
            return str + "}";
        }
        ```

- 일반적인 목적을 가진 코드를 많이 만들어라
    - 상관없는 하위문제를 다루는 함수들은 매우 기본적이고 폭넓게 적용할 수 있는 일을 수행하므로 다른 프로젝트에서도 사용할 수 있다.
    - 코드베이스는 이 같은 코드를 담아두는 (ex> util/ 같은) 디렉터리를 따로 두고 있으므로 코드를 쉽게 공유할 수 있다.
    - 일반적인 목적을 가진 코드는 프로젝트의 나머지 부분에서 완전히 분리되므로 좋다.
    - 이러한 코드는 개발하고, 테스트하고, 이해하기도 쉽다.
- 특정한 프로젝트를 위한 기능
    - util/ 디렉터리에 놓아도 되고, 함수가 사용되는 장소에 그대로 두어도 좋다. 필요하면 옮기면 된다.
- 기존의 인터페이스를 단순화하기
    - 사용하는 인터페이스가 깔끔하지 않다면, 이를 둘러싸는 함수를 작성하여 지저분한 내부를 감출 수 있다.
- 자신의 필요에 맞춰서 인터페이스의 형태를 바꾸기
- 지나치게 추출하기: 가독성 비용을 생각하자.
    - 자잘한 함수를 사용하면 오히려 가독성을 해친다.
    - 사용자가 신경 써야 하는 내용이 늘어나고, 실행 경로를 추적하려면 코드의 곳곳을 돌아다녀야 하기 때문이다.
    - 다른 프로젝트에서도 사용된다면 추출하는 것이 그리 나쁘진 않지만, 그런 순간이 오기 전에는 그럴 필요가 없다.


## 11. 한번에 하나씩

- 함수는 오직 한 가지 작업만 수행해야 한다.
    1. 코드가 수행하는 모든 ‘작업’을 나열한다. 이는 “이 객체가 정상적으로 존재하는지 확인하라”처럼 작은 일일 수도 있고, “트리 안에 있는 모든 노드를 방문하라”처럼 모호한 일일 수도 있다.
    2. 이러한 작업을 분리하여 서로 다른 함수로 혹은 적어도 논리적으로 구분되는 영역에 놓을 수 있는 코드로 만들어라.

- 작업은 작을 수 있다.
- 객체에서 값 추출하기
    - ‘한 번에 한 가지 일’ 원리 적용하기
    - before

        ```java
        var place = location_info["LocalityName"]; // ex> Santa Monica
        if (!place) {
            place = location_info["SubAdministrativeAreaName"]; // ex> Los Angeles
        }
        if (!place) {
            place = location_info["AdministrativeAreaName"]; // ex> California
        }
        if (!place) {
            place = "Middle-of-Nowhere";
        }
        if (location_info["CountryName"]) {
            place += ", " + location_info["CountryName"]; // ex> USA
        } else {
            place += ", Planet Earth";
        }

        return place;
        ```

    - location_info에서 값을 읽는 첫 번째 작업

        ```java
        var town = location_info["LocalityName"]; // ex> Santa Monica
        var city = location_info["SubAdministrativeAreaName"]; // ex> Los Angeles
        var state = location_info["AdministrativeAreaName"]; // ex> California
        var country = location_info["CountryName"]; // ex> USA
        ```

    - 반환되는 값의 ‘두 번째 절반’이 무엇인지 알아야 한다.

        ```java
        var second_half = "Planet Earth";
        if (country) {
            second_half = country;
        }
        if (state && country === "USA") {
            second_half = state;
        }
        ```

    - 이 같이 ‘첫 번째 절반’도 알아내자.

        ```java
        var first_half = "Middle-of-Nowhere";
        if (state && country != "USA") {
            first_half = state;
        }
        if (city) {
            first_half = city;
        }
        if (town) {
            first_half = town;
        }
        ```

    - 위의 두 절반을 묶는다. `return first_half + “, ” + second_half;`
- 더 큰 예제
    - 코드 전반에 걸쳐서 뒤섞여 있는 별도의 작업을 보자.
        1. 각 키를 위한 기본값으로 “unknown” 사용하기
        2. HttpDownload의 멤버 중 존재하지 않는 값이 있는지 확인하기
        3. 값을 읽어서 문자열로 변환하기
        4. counts[] 갱신하기
    - 별도의 영역으로 분리하면 한 개 영역을 읽는 동안에는 다른 영역을 생각할 필요가 전혀 없다.
    - 나열된 작업 대상 중 일부만 분리되어도 큰 도움이 된다.


## 12. 생각을 코드로 만들기

- 코드를 더 명확하게 만드는 간단한 과정
    1. 코드가 할 일을 옆의 동료에게 말하듯이 평범한 영어로 묘사하라
    2. 이 설명에 들어가는 핵심적인 단어와 문구를 포착하라
    3. 설명과 부합하는 코드를 작성하라

- 논리를 명확하게 설명하기
    - before

        ```java
        $is_admin = is_admin_request();
        if ($document) {
            if  (!$is_admin && ($document['username'] != $_SESSION['username'])) {
                return not_authorized();        
            }
        } else {
            if (!$is_admin) {
                return not_authorized();
            }
        }
        ```

    - 위의 논리를 쉬운 말로 묘사하는 것부터 시작해보자.
        1. 관리자다.
        2. 만약 문서가 있다면 현재 문서를 소유하고 있다.

      그렇지 않으면 허가되지 않는다.

    ```java
    if (is_admin_request()) {
        // 허가
    } else if ($document && ($document['username'] == $_SESSION['username'])) {
        // 허가
    } else {
        return not_authorized();
    }
    ```

    - 이 버전은 코드의 분량이 더 적고, 부정문이 없어 논리도 더 간단해 졌다.
- 라이브러리가 제공하는 기능을 잘 활용하자.


## 13. 코드 분량 줄이기

- 그 기능을 구현하려고 애쓰지 마라 - 그럴 필요가 없다.
    - 요구사항을 제거하기: 정말로 필요한 기능인지 정확히 생각해보자.
- 요구사항에 질문을 던지고 질문을 잘게 나누어 분석하라
    - 더 간단한 문제를 해결하기: 기능을 정확하게 구현하는 데에 필요한 상황을 생각해보자.
- 코드베이스를 작게 유지하기
    - 일반적인 ‘유틸리티’를 많이 생성하여 중복된 코드를 제거하라
    - 사용하지 않는 코드 혹은 필요 없는 기능을 제거하라
    - 프로젝트가 서로 분절된 하위프로젝트로 구성되게 하라
    - 코드베이스의 ‘무게’를 항상 의식하여 가볍고 날렵하게 유지시켜라
- 자기 주변에 있는 라이브러리에 친숙해져라
    - 매일 15분씩 자신의 표준 라이브러리에 있는 모든 함수/모듈/타입들의 이름을 읽어라.
    - 라이브러리 전체를 암기하는게 아니고, 그냥 그 안에 무엇이 있는지 감을 잡아놓고, 나중에 새로운 코드를 작성할 때 “잠깐만, 이건 전에 API 에서 봤던 것과 뭔가 비슷한데..”하고 생각하길 바라는 것이다.
    - 평균 수준의 소프트웨어 엔지니어는 출시할 수 있는 수준의 코드를 하루 평균 10줄 정도 작성한다고 한다.
        - ‘출시할 수 있는’ 코드다.
        - 완숙한 라이브러리 안에 있는 코드는 한 줄 한 줄 모두 상당한 분량의 설계, 디버깅, 재작성, 문서화, 최적화, 테스트를 거쳤다.
        - 이런 엄격한 적자생존의 과정을 겪으며 살아남은 코드는 모두 소중하다. 라이브러리를 이용하면 좋은 이유가 바로 여기 있다.
        - 라이브러리를 사용하면 시간도 절약하고, 코드양이 줄어들기도 한다.
- 예: 코딩 대신 유닉스 도구를 활용하기


# Part 4. 선택된 주제들

## 14. 테스트와 가독성

- 읽거나 유지보수하기 쉽게 테스트를 만들어라
    - 다른 프로그래머가 수정하거나 새로운 테스트를 더하는 걸 쉽게 느낄 수 있게 테스트 코드는 읽기 쉬워야 한다.
    - 테스트 코드가 크고 두렵게 느껴지면..,
        - 코드를 수정하는 일이 두려워진다.
            - 아니, 우리는 이 코드에 손대고 싶지 않아. 테스트 케이스를 모두 변경하는 일은 너무나 끔찍하다고!
        - 새로운 코드를 작성하면 그에 따르는 새로운 테스트를 작성하지 않는다.
            - 시간이 흐르면서 더 낮은 비율의 코드가 테스트되며, 코드에 대한 확신이 줄어들 수밖에 없다.

- before

    ```java
    void Test1() {
        vector<ScoredDocument> docs;
        docs.resize(5);

        docs[0].url = "http://example.com";
        docs[0].score = -5.0;
        docs[1].url = "http://example.com";
        docs[1].score = 1;
        docs[2].url = "http://example.com";
        docs[2].score = 4;
        docs[3].url = "http://example.com";
        docs[3].score = -99998.7;
        docs[4].url = "http://example.com";
        docs[4].score = 3.0;

        SortAndFilterDocs(&docs);

        assert(docs.size() == 3);
        assert(docs[0].score == 4);
        assert(docs[1].score == 3.0);
        assert(docs[2].score == 1);
    }
    ```

    - 잘못된 곳
        1. 테스트가 너무 길고 중요하지 않은 자세한 내용으로 가득 찼다. 이 테스트가 하는 일을 한 문장으로 표현할 수 있으므로, 실제 테스트 구문이 지나치게 길 이유가 없다.
        2. 새로운 테스트를 추가하기 쉽지 않다. 복사/붙이기/수정 방법을 쓰고 싶은 유혹을 느끼겠지만, 그렇게하면 코드를 더 길고 중복되게 한다.
        3. 테스트 실패 메시지가 별로 도움이 되지 않는다. 테스트에 실패하면 단지 Assertion failed: docs.size() == 3 같은 내용을 출력할 것이다. 디버깅에 도움되는 정보를 포함하지 않는다.
        4. 모든 것을 한꺼번에 테스트하려고 애쓰고 있다. 음수를 필터링하는 기능, 수를 정렬하는 기능을 동시에 테스트하는 것이다. 여러 개의 테스트로 나누면 더 읽기 쉬울 것이다.
        5. 테스트 입력이 간단치 않다. 특히 -99998.7 같은 입력은 그 값이 특별히 중요한 의미를 갖지 않음에도 불필요하게 시선을 끈다. 더 간단한 음수값을 사용해도 충분하다.
        6. 테스트 입력값들이 코드를 꼼꼼하게 실행시키지 않는다. 예를 들어 점수가 0인 경우는 다루지 않는다.
        7. 비어있는 입력 벡터, 매우 큰 벡터 혹은 중복된 점수 같이 비정상적인 값을 가지는 입력을 테스트하지 않는다.
        8. Test1() 라는 이름은 아무 의미가 없다. 이름은 반드시 테스트되는 함수나 상황을 설명해야 한다.
- 이 테스트를 더 읽기 쉽게 만들기: 덜 중요한 세부 사항은 숨겨서 더 중요한 내용이 눈에 잘 띄게 해야한다.
    - 각 테스트의 최상위 수준은 최대한 간결해야 한다. 이상적으로는 각 테스트의 입출력이 한 줄의 코드로 설명될 수 있어야 한다.
- 읽기 편한 메시지 만들기: 테스트에 실패하면, 버그를 추적해서 수정하는 데 도움될 만한 에러 메시지를 출력해야 한다.
    - 위에서 `assert(output == expected_output)`가 실패하면 에러 메시지가 출력될 것이다. output 과 expected_output 값이 무엇인지 궁금해 질 것이다.

        ```java
        Assertion failed: (output == expected_output),
          function CheckScoresBeforeAfter, file test.cc, line 37.
        ```

    - 향상된 버전의 assert() 사용하기
        - assert() 대신 assertEqual() 같은 메소드를 사용하면 더 도움되는 에러 메시지를 출력한다.
    - 손수 작성한 에러 메시지: 에러 메시지가 최대한 유용하도록 목적에 맞는 assert 를 스스로 구현해도 된다.
- 좋은 테스트 입력값의 선택: 가능하면 가장 간단한 입력으로 코드를 완전히 검사할 수 있어야 한다.
    - 입력값을 단순화하기
        - `CheckScoresBeforeAfter(“-5, 1, 4, -99998.7, 3”, “4, 3, 1”)` 에서 -99998.7 는 ‘시끄러운’ 값이다.
        - 테스트의 효율성을 전혀 손상시키지 않으면서 입력되는 값들을 단순하게 만들자.
    - 결과: `CheckScoresBeforeAfter(“1, 2, -1, -3”, “3, 2, 1”)`
    - 다양한 기능의 테스트: 코드를 구석구석 철저하게 수행하려고 하나의 ‘완벽한’ 입력을 사용하는 것보다는 작은 테스트를 여러 개 사용하는 방식이 더 쉽고 효과적이다.
    - SortAndFilterDocs()에 대한 4가지 테스트

        ```java
        CheckScoresBeforeAfter(“2, 1, -3”, “3, 2, 1”); // 기본적인 정렬
        CheckScoresBeforeAfter(“0, -0.1, -10”, “0”); // 0보다 작은 값을 모두 제거
        CheckScoresBeforeAfter(“1, -2, 1, 2”, “1, 1”); // 중복은 문제되지 않는다.
        CheckScoresBeforeAfter(“”, “”); // 빈 입력도 OK
        ```

- 테스트 함수에 이름 붙이기: 테스트를 상세하게 묘사할 수 있는 이름을 사용하는게 좋다.
    - 테스트되는 클래스
    - 테스트되는 함수
    - 테스트되는 상황이나 버그
    - 'Test_' 같은 접두사를 이용하여 필요한 정보를 모두 하나로 붙이는 것
        - `Test_함수이름_상황()`형태를 이용하면 된다.
        - ex> `void Test_SortAndFilterDocs_BasicSorting()`
    - 테스트 함수명은 실제 코드베이스에서 호출되는 함수가 아니어서 길어도 된다.
        - 만약 테스트가 실패하면 함수명을 출력하기 때문에 상황을 잘 묘사하는 이름을 사용하면 도움이 된다.
- 테스트의 수정이나 추가가 쉬워야 한다.

- 테스트에 친숙한 개발
    - 테스트하기 어려운 코드의 특징과 이것이 설계와 관련된 문제에 미치는 영향

      ![Untitled51.png](theartofreadablecode/Untitled%2051.png)

    - 테스트와 설계가 가지는 좋은 특징

      ![Untitled52.png](theartofreadablecode/Untitled%2052.png)

- 지나친 테스트
    - 테스트를 가능하게 하려고 실제 코드의 가독성을 희생시킨다.
        - 테스트를 가능하게 하려고 실제 코드에 지저분한 코드를 넣어야 한다면, 뭔가 잘못된 것이다.
    - 100% 코드 테스트에 집착하는 일
        - 버그로 인한 비용이 별로 높지 않기 때문에 굳이 테스트할 필요가 없는 사용자 인터페이스나 이상한 에러 케이스를 포함하고 있을지도 모른다.
    - 사실, 코드를 100% 테스트하는 일은 일어나지 않는다.
        - 테스트되지 않은 버그가 있을 수도 있고, 테스트되지 않은 기능이 있을 수도 있으며, 요구사항이 달라졌다는 사실을 모르고 있을 수도 있기 때문이다.
    - 버그가 야기하는 비용이 어느 정도인지에 따라서, 테스트 코드를 작성하는 시간이 의미를 갖는 부분이 있고 그렇지 않은 부분도 있기 마련이다.
        - 만약 웹사이트의 프로토타입을 만든다면, 테스트 코드 작성은 전혀 의미가 없다.
        - 우주선이나 의료 장비를 통제하는 프로그램을 작성한다면 아마 테스트 코드에 주된 관심을 쏟아야 할 것이다.
    - 테스트 코드로 실제 제품 개발이 차질을 빚게 되는 일


## 15. '분/시간 카운터' 를 설계하고 구현하기

- 웹 서버가 1분 동안, 그리고 지난 1시간 동안 얼마나 많은 바이트를 전송했는지 추적해보자.

### 클래스 인터페이스 정의하기

---

- before

    ```cpp
    class MinuteHourCounter {
    	public:
    		// 카운트를 더한다.
    		void Count(int num_bytes);

    		// 1분 동안의 카운트를 반환한다.
    		int MinuteCount();

    		// 1시간 동안의 카운트를 반환한다.
    		int HourCount();
    };
    ```

- 이름을 개선하기
    - 'get' 단어는 보편적으로 '가벼운 접근자'라는 의미를 내포한다. 그러므로 지금이 괜찮은 메소드 명이다.
    - Count(): 전체 시간에 대한 카운트의 총합을 반환할 것만 같다. 직관에 위배된다.
        - 메소드 이름을 '수치적으로 더하라', '데이터 리스트에 더하라' 두 의미의 `void Add()`로 고치자.
        - num_bytes 이름이 지나치게 특징적이다. 간단하고, 일반적이고, '음수가 아니라는' 사실을 함축하고 있는 `count`로 변경하자.
- 주석을 개선하기
    - 첫 번째 주석은 함수명과 완전히 중복이므로, 지우거나 개선되어야 한다.
        - // 새로운 데이터 포인트를 더한다 (count ≥ 0)
        - // 다음 1분 동안 MinuteCount()는 +count에 의해서 값이 커진다.
        - // 다음 1시간 동안 HourCount()는 +count에 의해서 값이 커진다.

        ```cpp
        // 새로운 데이터 포인트를 더한다 (count ≥ 0)
        // 다음 1분 동안 MinuteCount()는 +count에 의해서 값이 커진다.
        // 다음 1시간 동안 HourCount()는 +count에 의해서 값이 커진다.
        void Count(int num_bytes);
        ```

    - 2, 3 번째 주석은 현재의 분 반환/시계와 상관없이 60초 동안에 해당하는 카운트 반환 2가지 해석을 나타내는데, 실제 동작하는 방식은 후자이다.

        ```cpp
        // 지난 60초 동안 누적된 카운트를 반환한다.
        int MinuteCount();

        // 지난 3600초 동안 누적된 카운트를 반환한다.
        int HourCount();
        ```

- 인터페이스

    ```cpp
    // 지난 1분과 지난 1시간 동안 누적된 카운트를 추적한다.
    // 예를 들어 최근의 대역폭 사용량을 확인할 수 있다.
    class MinuteHourCounter {
    	public:
    		// 새로운 데이터 포인트를 더한다 (count ≥ 0)
    		// 다음 1분 동안 MinuteCount()는 +count에 의해서 값이 커진다.
    		// 다음 1시간 동안 HourCount()는 +count에 의해서 값이 커진다.
    		void Add(int count);

    		// 지난 60초 동안 누적된 카운트를 반환한다.
    		int MinuteCount();

    		// 지난 3600초 동안 누적된 카운트를 반환한다.
    		int HourCount();
    };
    ```

### 시도1: 순진한 해결책

---

- 단순한 해결책부터 시작하자. 시간을 담고 있는 '이벤트'의 리스트를 이용하자.

    ```cpp
    class MinuteHourCounter {
    	struct Event {
    		Event(int count, time_t time) : count(count), time(time) {}
    		int count;
    		time_t time;
    	};
    	list<Event> events;

    	public:
    		void Add(int count) {
    			events.push_back(Event(count, time()));
    		}
    		..
    };
    ```

- 그 다음, 필요에 따라서 가장 최근 이벤트들을 순차적으로 반복한다.

    ```cpp
    class MinuteHourCounter {
    		..
    		int MinuteCount() {
    			int count = 0;
    			count time_t now_secs = time();
    			for (list<Event>::reverse_iterator i = events.rbegin();
               i != events.rend() && i->time > now_secs - 60; ++i) {
    				count += i->count;
    			}
    			return count;
    		}

    		int HourCount() {
    			int count = 0;
    			count time_t now_secs = time();
    			for (list<Event>::reverse_iterator i = events.rbegin();
               i != events.rend() && i->time > now_secs - 3600; ++i) {
    				count += i->count;
    			}
    			return count;
    		}
    };
    ```

    - 이 코드는 이해하기 쉬운가?
        - 이 솔루션은 '정확'하지만, 가독성 관련 2가지 문제가 있다.
        1. for 루프가 다소 산만하다. 루프를 읽을 때 대부분의 프로그래머는 속도가 늦춰진다.
        2. MinuteCount() 와 HourCount() 내용이 거의 똑같다. 중복된 코드가 상대적으로 복잡하기 때문에 코드 공유는 특히 중요한 의미를 갖는다.

- 더 읽기 쉬운 버전

    ```cpp
    class MinuteHourCounter {
    	struct Event {
    		Event(int count, time_t time) : count(count), time(time) {}
    		int count;
    		time_t time;
    	};
    	list<Event> events;

    	int CountSince(time_t cutoff) {
    		int count = 0;
    		for (list<Event>::reverse_iterator rit = events.rbegin();
             rit != events.read(); ++rit) {
    			if (rit->time <= cutoff) {
    				break;
    			}
    			count += rit->count;
    		}
    		return count;
    	}

    	public:
    		void Add(int count) {
    			events.push_back(Event(count, time()));
    		}

    		int MinuteCount() {
    			return CountSince(time() - 60);
    		}

    		int HourCount() {
    			return CountSince(time() - 3600);
    		}
    };
    ```

    - CountSince()가 절대값 cutoff 를 파라미터로 받고 있다.
        - 상대값(60, 3600)도 지장은 없지만, 고정된 값을 받아들이는게 더 쉽다.
    - 반복자를 i 에서 rit 로 바꾸었다.
        - 역방향 반복자라는 사실을 강조한다.
    - `rit->time <= cutoff` 조건을 for 루프에서 꺼내어 별도의 `if`문으로 만들었다.
        - 전통적인 `for`루프가 더 읽기 쉽기 때문이다.
- 성능 문제
    - 이 설계는 성능과 관련된 2가지 심각한 문제가 있다.
    - 계속해서 늘어나기만 한다.
        - 이 클래스는 한 번이라도 발생한 이벤트를 모두 보관하기 때문에 메모리 사용량이 끝없이 증가한다. MinuteHourCounter 에서 필요 없는 1시간이 지난 데이터를 자동으로 삭제하는 편이 바람직하다.
    - MinuteCount(), HourCount() 가 너무 느리다.
        - CountSince()는 시간 범위에 있는 데이터 포인트 개수가 n일 때 O(n) 만큼 시간을 소모한다.
        - 초당 100번 Add() 를 호출하는 고성능 서버라면, HourCount()가 호출될 때마다 100만 개가 넘는 데이터 포인트를 처리해야 한다.
        - MinuteHourCounter 에 별도의 minute_count, hour_count 변수를 두어 Add()가 호출될 때마다 값을 반영하는 방법이 바람직하다.

### 시도2: 컨베이어 벨트 설계

---

- 앞의 두 문제를 모두 해결해주는 설계가 필요하다.
    1. 필요 없는 데이터를 삭제하라.
    2. 미리 계산된 minute_count, hour_count 의 총합이 가장 최근에 계산된 값을 담게 하라.
- 리스트를 마치 컨베이어 벨트처럼, 한쪽 끝에 도착한 새로운 데이터를 총합에 더하고, 데이터가 너무 오래되면(유효시간을 넘기면) 해당 데이터가 한쪽 끝에서 '떨어져 나가고', 우리는 해당 값을 총합에서 빼자.
- 접근: '2단계' 컨베이어 벨트 설계 방식 선택
    - 독립적인 리스트 2개를 사용하는 방법
        - 하나는 지난 1분 동안에 대한 리스트, 다른 하나는 지난 1시간에 대한 리스트
        - 새로운 이벤트가 도착하면 값을 복사하여 두 리스트에 넣는다.

  → 매우 간단하지만, 모든 이벤트의 복사본을 항상 2번 만들어서 비효율적이다.

    - 이벤트가 일단 하나의(지난 1분 동안에 대한) 리스트에 들어가고, 2번째(지난 1시간에 대한, 하지만 지난 1분 동안에 대한 리스트는 아닌) 리스트에 들어가는 방법이 있다.

- 2단계 컨베이어 벨트를 설계하고 구현하기

    ```cpp
    class MinuteHourCounter {
    	struct Event {
    		Event(int count, time_t time) : count(count), time(time) {}
    		int count;
    		time_t time;
    	};
    	list<Event> minute_events;
    	list<Event> hour_events; // minute_events에 들어있지 않은 것들만 담는다.
    	list minute_count;
    	list hour_count; // 지난 1분을 포함해서 지난 1시간 동안 발생한 이벤트를 모두 센다.

    	public:
    		void Add(int count) {
    			const time_t now_secs = time();
    			ShiftOldEvents(now_secs);

    			// 분을 위한 리스트에 넣는다(시간을 위한 리스트에 들어가는 것은 나중이다).
    			minute_events.push_back(Event(count, now_secs));		

    			minute_count += count;
    			hour_count += count;
    		}

    		int MinuteCount() {
    			ShiftOldEvents(time());
    			return minute_count;
    		}

    		int HourCount() {
    			ShiftOldEvents(time());
    			return hour_count;
    		}
    };
    ```

- ShiftOldEvents() 헬퍼 메소드: 시간이 지나면 이벤트를 '옮겨서' minute_events 에서 hour_events 로 움직이고, 이에 따라 minute_count, hour_count 를 적절히 수정한다.

    ```cpp
    // 오래된 이벤트를 찾아서 삭제하고, hour_count, minute_count 값을 감소시킨다.
    void ShiftOldEvents(time_t now_secs) {
    	const int minute_ago = now_secs - 60;
    	const int hour_ago = now_secs - 3600;

    	// 1분 이상 지난 이벤트는 minute_events에서 hour_events로 이동시킨다.
    	// (1시간 이상 지난 이벤트는 두 번째 루프에서 삭제될 것이다.)
    	while (!minute_events.empty() && minute_events.front().time <= minute_ago) {
    		hour_events.push_back(minute_events.front());

    		minute_count -= minute_events.front().count;
    		minute_events.pop_front();
    	}

    	// 1시간 이상 지난 이벤트는 hour_events로부터 삭제한다.
    	while (!hour_events.empty() && hour_events.front().time <= hour_ago) {
    		hour_count -= hour_events.front().count;
    		hour_events.pop_front();
    	}
    }
    ```

- 문제
    - 설계가 유연하지 않다.
        - 지난 24시간에 대한 카운트를 알고 싶다고 하면, 대대적인 수정이 불가피하다.
    - 이 클래스는 상당히 많은 메모리를 소비한다.
        - 트래픽이 많은 서버가 Add()를 초당 100번 호출한다고 하면, 지난 1시간 분량의 데이터를 보관하므로 이 코드는 대략 5MB 정도의 메모리를 사용한다.
        - Add()가 더 자주 호출되면 더 많은 메모리가 소모된다. 호출 횟수에 상관없이 일정량의 메모리를 사용하는 것이 바람직하다.

### 시도3: 시간-바구니 설계

---

- 0.99 초에 이벤트가 발생하면 0으로 내림되는 등, 총합에 포함되지 못하는 경우의 수 버그가 있다.
- 작은 시간 범위에 들어오는 이벤트를 1초 너비의 각 바구니에 담고 총합을 구하자.
    - 60초 혹은 60분 마다 바구니 하나를 사용하는데, 더 높은 정확성이 필요하다면 더 큰 메모리 용량을 사용하는 대신 더 많은 바구니를 사용할 수 있다.
    - 이러한 설계는 고정되고 예측 가능한 메모리 용량을 사용한다.

- 시간-바구니를 설계하고 구현하기
    - TrailingBucketCounter: 어떤 시간 간격(ex> 지난 1시간 동안)에 대한 카운트를 저장하는 클래스
    - TrailingBucketCounter 인터페이스

        ```cpp
        // 지나간 N개의 바구니에 대한 카운트를 보관하는 클래스
        class TrailingBucketCounter {
        	public:
        		// 예: TrailingBucketCounter(30, 60)은 지난 30분에 해당하는 바구니를 처리한다.
        		TrailingBucketCounter(int num_buckets, int secs_per_bucket**);**

        		void Add(int count, time_t now);

        		// 마지막 num_buckets의 시간에 대한 총 카운트를 반환합니다.
        		int TrailingBucketCount(time_t now);
        }
        ```

        - 현재 시간을 인수로 전달하는 방법의 장점
            - TrailingBucketCounter 는 '시계가 없는' 클래스일 수 있어 테스트하기 더 쉽고, 버그도 적다.
            - time() 호출을 모두 MinuteHourCounter 내부로 제한할 수 있다.
                - 시간에 민감한 시스템에서는 시간을 얻는 함수를 한 곳에 모아두고 호출하는 편이 좋다.

    ```cpp
    class MinuteHourCounter {
    	TrailingBucketCounter minute_counts;
    	TrailingBucketCounter hour_counts;

    	public:
    		MinuteHourCounter() :
    			minute_counts(/* num_buckets = */ 60, /* secs_per_bucket = */ 1),
    			hour_counts(/* num_buckets = */ 60, /* secs_per_bucket = */ 60) {
    		}

    		void Add(int count) {
    			time_t now = time();
    			minute_counts.Add(count, now);
    			hour_counts.Add(count, now);
    		}

    		int MinuteCount() {
    			time_t now = time();
    			return minute_counts.TrailingBucketCount(now);
    		}

    		int HourCount() {
    			ShiftOldEvents(time());
    			return hour_counts.TrailingBucketCount(now);
    		}
    };
    ```

    - 전보다 훨씬 읽기 쉽고, (정확성을 높이는 대신 더 많은 메모리를 사용하도록 변경하기) 유연하다.
- ConveyorQueue: 기저에 깔린 카운트와 그들의 총합을 다루는 일을 수행하는 데이터 구조
    - 인터페이스

        ```cpp
        // 오래된 데이터가 큐의 한쪽 끝에서 '떨어져 나가는' 최대로 많은 수의 슬롯을 가지고 있는 큐
        class ConveyorQueue {
        	ConveyorQueue(int max_items);

        	// 큐의 뒤에 있는 값을 증가시킨다.
        	void AddToBack(int count);

        	// 큐에 있는 각 값이 'num_shifted'만큼 앞으로 움직인다.
        	// 이제 항목들은 다시 0으로 초기화된다.
        	// 가장 오래된 항목들이 제거되므로 항목의 수는 max_items보다 작거나 같다.
        	void Shift(int num_shifted);

        	// 현재 큐에 있는 모든 항목의 총합을 반환한다.
        	int TotalSum();
        }
        ```

    ```cpp
    // 지나간 N개의 바구니에 대한 카운트를 보관하는 클래스
    class TrailingBucketCounter {
    	ConveyorQueue buckets;
    	const int secs_per_bucket;
    	time_t last_update_time; // Update()가 마지막으로 호출된 시간

    	// 얼마나 많은 '바구니 시간'이 지났는지 계산하고 그에 따라서 Shift()를 호출한다.
    	void Update(time_t now) {
    		int current_bucket = now / secs_per_bucket;
    		int last_update_bucket = last_update_time / secs_per_bucket;

    		buckets.Shift(current_bucket - last_update_bucket);
    		last_update_time = now;
    	}

    	public:
    		// 예: TrailingBucketCounter(30, 60)은 지난 30분에 해당하는 바구니를 처리한다.
    		TrailingBucketCounter(int num_buckets, int secs_per_bucket) :
    			buckets(num_buckets),
    			secs_per_bucket(secs_per_bucket) {
    		}

    		void Add(int count, time_t now) {
    			Update(now);
    			buckets.AddToBack(count);
    		}

    		// 마지막 num_buckets의 시간에 대한 총 카운트를 반환합니다.
    		int TrailingBucketCount(time_t now) {
    			Update(now);
    			return buckets.TotalSum();
    		}
    }
    ```

- ConveyorQueue 구현하기

    ```cpp
    // 오래된 데이터가 큐의 한쪽 끝에서 '떨어져 나가는' 최대로 많은 수의 슬롯을 가지고 있는 큐
    class ConveyorQueue {
    	queue<int> q;
    	int max_items;
    	int total_sum; // q에 있는 모든 항목의 총합

    	public:
    		ConveyorQueue(int max_items) : max_items(max_items), total_sum(0) {
    		}

    		// 현재 큐에 있는 모든 항목의 총합을 반환한다.
    		int TotalSum() {
    			return total_sum;
    		}
    	
    		// 큐에 있는 각 값이 'num_shifted'만큼 앞으로 움직인다.
    		// 이제 항목들은 다시 0으로 초기화된다.
    		// 가장 오래된 항목들이 제거되므로 항목의 수는 max_items보다 작거나 같다.
    		void Shift(int num_shifted) {
    			// 너무 많은 항목이 움직이면, 그냥 큐가 비도록 만든다.
    			if (num_shifted >= max_items) {
    				q = queue<int>(); // 큐 clear
    				total_sum = 0;
    				return;
    			}
    			// 모든 필요한 0을 큐에 넣는다.
    			while (num_shifted > 0) {
    				q.push(0);
    				num_shifted--;
    			}

    			// 경계를 벗어난 항목들이 모두 떨어져 나가도록 만든다.
    			while (q.size() > max_items) {
    				total_sum -= q.front();
    				q.pop();
    			}
    		}

    		// 큐의 뒤에 있는 값을 증가시킨다.
    		void AddToBack(int count) {
    			if (q.empty()) Shift(1); // q가 최소한 1개의 항목을 갖도록 하라.
    			q.back() += count;
    			total_sum += count;
    		}
    }
    ```

### 3가지 해결책 비교하기

---

- 초당 100번의 Add()호출 시, 성능 통계수치

  ![Untitled53.png](theartofreadablecode/Untitled%2053.png)

- 사용자는 클래스1(MinuteHourCounter) 만 알면 충분하다.
    - ConveyorQueue: 시스템이 허락하는 한도 내에서 무제한의 크기를 갖는 큐로서 옆으로 한 칸 움직일 수 있으며, 큐 안에 담긴 모든 항목의 총합을 관리한다.
    - TrailingBucketCounter: 시간의 흐름에 따라서 ConveyorQueue를 옆으로 한 칸 이동시키고, (현재 시간에 가장 가까운 최근) 시간구간의 카운트를 저장한다.
    - MinuteHourCounter: TrailingBucketCounter 2개를 보관한다. 하나는 1분, 다른 하나는 1시간에 대한 카운트용이다.


- [Notion link](https://www.notion.so/f4cac16fc0de4bfd924448f2a7360376)
