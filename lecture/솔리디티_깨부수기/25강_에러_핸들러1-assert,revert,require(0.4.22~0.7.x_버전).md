# 25강. 에러 핸들러 1 - assert, revert, require (0.4.22 ~ 0.7.x 버전)

- 에러 처리를 위해 제공되는 세 가지 핸들러: `assert`, `require`, `revert`
    - 정의된 조건에 부합하지 않으면(`false`), 에러를 발생시킨다.
    - 솔리디티 0.4.22 버전에 소개되었다.

## assert

---

- `assert`: 내부 오류를 테스트하거나 불변성을 확인하는 데 사용된다. 조건이 `false`일 경우, 즉시 실행을 중단하고 모든 가스를 소모한다.
    - 이미 함수가 다 돌고나서(가스 다 소비하고 나서), `assert`로 체크된다.
    - 코드가 정상적으로 작동하는지 테스트 용도로 사용한다.
    
    ```solidity
    function assertExample(uint256 value) public pure {
        assert(value > 0);
        // 추가 로직
    }
    ```
    

### 예제

---

- lec25.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec25{
    
        /*
        0.4.22 ~ 0.7.x
        assert: gas를 다 소비한후, 특정한 조건에 부합하지 않으면 (false일 때) 에러를 발생시킨다.
        revert: 조건없이 에러를 발생시키고, gas를 환불 시켜준다.
        require: 특정한 조건에 부합하지 않으면(false일 때) 에러를 발생시키고, gas를 환불 시켜준다.
        */
        function assertNow() public pure {
            assert(false);
        }
    }
    ```
    
    - 실행
        
        ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image.png)
        

## revert

---

- `revert`: 특정 조건 없이도 실행을 중단하고 에러를 발생시킬 수 있으며, 사용되지 않은 가스를 환불 시켜준다. 주로 복잡한 조건에서 에러를 발생시키고자 할 때 사용된다.
    
    ```solidity
    function revertExample(uint256 value) public pure {
        if (value == 0) {
            revert("Value cannot be zero");
        }
        // 추가 로직
    }
    ```
    

### 예제

---

- lec25.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec25{
    
        /*
        0.4.22 ~ 0.7.x
        assert: gas를 다 소비한후, 특정한 조건에 부합하지 않으면 (false일 때) 에러를 발생시킨다.
        revert: 조건없이 에러를 발생시키고, gas를 환불 시켜준다.
        require: 특정한 조건에 부합하지 않으면(false일 때) 에러를 발생시키고, gas를 환불 시켜준다.
        */
        function assertNow() public pure {
            assert(false);
        }
        function revertNow() public pure {
            revert("error");
        }
    }
    ```
    
    - 실행
        
        ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image%201.png)
        

## require

---

- `require`: 주로 입력 값의 유효성 검사나 실행 전 조건을 확인하는 데 사용된다. 조건이 `false`일 경우, 에러를 발생시키고, 사용되지 않은 가스를 환불 시켜준다.
    
    ```solidity
    function requireExample(uint256 value) public pure {
        require(value > 0, "Value must be greater than zero");
        // 추가 로직
    }
    ```
    

### 예제

---

- lec25.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec25{
    
        /*
        0.4.22 ~ 0.7.x
        assert: gas를 다 소비한후, 특정한 조건에 부합하지 않으면 (false일 때) 에러를 발생시킨다.
        revert: 조건없이 에러를 발생시키고, gas를 환불 시켜준다.
        require: 특정한 조건에 부합하지 않으면(false일 때) 에러를 발생시키고, gas를 환불 시켜준다.
        */
        function assertNow() public pure {
            assert(false);
        }
        function revertNow() public pure {
            revert("error");
        }
        function requireNow() public pure {
            require(false, "occurred");
        }
    }
    ```
    
    - 실행
        - revertNow
            
            ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image%202.png)
            
        - requireNow
            
            ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image%203.png)
            

## 예제

---

- lec25.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec25{
    
        /*
        0.4.22 ~ 0.7.x
        assert: gas를 다 소비한후, 특정한 조건에 부합하지 않으면 (false일 때) 에러를 발생시킨다.
        revert: 조건없이 에러를 발생시키고, gas를 환불 시켜준다.
        require: 특정한 조건에 부합하지 않으면(false일 때) 에러를 발생시키고, gas를 환불 시켜준다.
        */
        function assertNow() public pure {
            assert(false);
        }
        function revertNow() public pure {
            revert("error");
        }
        function requireNow() public pure {
            require(false, "occurred");
        }
    
        function onlyAdults(uint256 _age) public pure returns(string memory) {
            if(_age < 19) {
                revert("You are not allowed to pay for the cigarette");
            }
            return "Your payment is succeeded";
        }
        
        function onlyAdults2(uint256 _age) public pure returns(string memory) {
            require(_age >= 19,"You are not allowed to pay for the cigarette");
            return "Your payment is succeeded";
        }
    }
    ```
    
    - 컴파일 시 0.7.4 버전으로 컴파일
    - 실행
        - onlyAdults
            
            ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image%204.png)
            
        - onlyAdults2
            
            ![image.png](25%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%201%20-%20assert,%20revert,%20require%20(0%201c0b5a48de448059a27fe7fdd7ecaf15/image%205.png)