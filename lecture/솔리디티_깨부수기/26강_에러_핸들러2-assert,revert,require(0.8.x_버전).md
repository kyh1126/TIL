# 26강. 에러 핸들러 2 - assert, revert, require (0.8.x 버전)

## assert

---

- 솔리디티(Solidity) 0.8 버전에서는 `assert` 문과 관련하여 중요한 변화가 있었다.
    - 이전 버전(0.4.22 ~ 0.7.x)에서는 `assert` 문이 실패할 경우, 사용된 가스를 환불하지 않고 모두 소모하였다.
- 주요 변경 사항
    1. 가스 환불: 0.8 버전부터 `assert` 문 실패하면, 사용되지 않은 가스가 호출자에게 환불된다.
        - `assert` 문이 `INVALID` opcode 대신 `REVERT` opcode를 사용하기 때문이다.
    2. 에러 처리: `assert` 문이 실패하면 `Panic(uint256)` 에러 타입이 발생힌다.
        - 이 에러는 내부 오류나 불변성 체크 실패 시 발생하며, 스마트 컨트랙트 내 버그를 나타낼 수 있다.
    3. 사용 목적: `assert` 문은 내부적인 에러 테스트나 불변성 체크 용도로만 사용해야 한다.
        - 만약 `assert` 문이 실패한다면, 이는 코드에 심각한 문제가 있음을 의미하므로, 해당 부분을 수정하는 것이 중요하다.

## **Panic 에러 코드**

---

- 0.8.1~ 버전에서는 `Panic` 에러를 발생시키며, 각 상황에 따라 고유한 에러 코드를 제공한다.
    - 0x01: `assert` 문이 실패한 경우
    - 0x11: 산술 연산이 언더플로우 또는 오버플로우를 발생시킨 경우
    - 0x12: 0으로 나누기를 시도한 경우
    
    → 발생한 문제의 원인을 보다 정확하게 파악할 수 있다.
    

## **추가 사항**

---

- 반면, `require`와 `revert` 문은 여전히 `Error(string)` 타입의 에러를 발생시키며, 주로 입력 값 검증이나 조건 체크에 사용된다.

## 예제

---

- lec26.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec26{
    
        /*
        panic: 0.8.1~
        assert: 오직 내부적 에러 테스트 용도, 불변성 체크 용도.
                assert가 에러를 발생시키면, Panic(uint256) 에러 타입의 에러 발생
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
    
    - 컴파일 시 0.8.7 버전으로 컴파일
    - 실행
        - assertNow
            
            ![image.png](26%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%202%20-%20assert,%20revert,%20require%20(0%201c7b5a48de4480288bdddbbf21f1977c/image.png)
            
            - output의 마지막 `00000001` 부분이 Panic의 `0x01` 에러다.