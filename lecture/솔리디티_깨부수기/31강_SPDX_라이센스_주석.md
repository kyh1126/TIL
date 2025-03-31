# 31강. SPDX 라이센스/주석

## SPDX 라이센스

---

- 솔리디티(Solidity) 0.6.8 버전 이후부터는 소스 코드 상단에 SPDX 라이선스 식별자를 명시하는 것이 권장된다.
    - 스마트 컨트랙트의 저작권 정보를 명확히 하여 코드의 신뢰성과 투명성을 높이는 데 기여한다.

### SPDX 라이선스 식별자

---

- SPDX(Software Package Data Exchange) 라이선스 식별자: 소프트웨어의 라이선스를 표준화된 방식으로 표현하는 문자열. 이를 통해 소프트웨어의 사용 조건과 저작권 정보를 명확하게 전달할 수 있다.
- 솔리디티에서의 사용 예시
    
    ```solidity
    // SPDX-License-Identifier: MIT
    ```
    
    - 위의 주석은 해당 소스 코드가 MIT 라이선스를 따른다는 것을 나타낸다.
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    ```
    

- 사용 가능한 SPDX 라이선스 식별자의 전체 목록: [SPDX 공식 웹사이트](https://spdx.org/licenses/)에서 확인할 수 있다.
- 컴파일러는 지정된 라이선스가 해당 목록에 있는지 확인하지 않으므로, 임의의 문자열을 사용할 수 있지만, 표준화된 식별자를 사용하는 것이 좋다.
- 라이선스를 명시하지 않으려면, 다음과 같이 `UNLICENSED`를 사용할 수 있다.
    
    ```solidity
    // SPDX-License-Identifier: UNLICENSED
    ```
    

## 주석

---

- 솔리디티에서 주석은 코드에 대한 설명이나 메모를 추가하는 데 사용된다.
- 주석은 컴파일 시 무시되며, 코드의 가독성과 유지보수성을 향상시키는 데 도움이 된다.

### 블록 주석(Block Comment)

---

- 여러 줄의 주석을 작성할 때 사용한다.
    
    ```solidity
    /*
    이 함수는 두 숫자를 더합니다.
    입력값: uint256 a, uint256 b
    반환값: uint256
    */
    ```
    

### 한 줄 주석(Line Comment)

---

- 한 줄짜리 주석을 작성할 때 사용한다.
    
    ```solidity
    uint256 sum = a + b; // 두 숫자의 합을 계산합니다.
    ```