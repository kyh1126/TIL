# 41강. import

## `import`

---

- `import`: 다른 Solidity 파일에 정의된 코드(예: 스마트 컨트랙트, 라이브러리 등)를 현재 파일로 가져와 재사용할 수 있다. 이를 통해 코드의 모듈화와 유지보수성을 향상시킬 수 있다.

## 예제

---

- lec41.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.8.0;
    
    import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/docs-v3.x/contracts/math/SafeMath.sol";
    
    contract lec41 {
        using SafeMath for uint256;
        uint public a;
        uint public maximum = ~uint256(0); // ==2**256-1; // 2**256 == 2^256
        function becomeOverflow(uint _num1, uint _num2) public {
            a = _num1.add(_num2);
        }
    }
    ```
    
    - 0.7.6으로 컴파일 후 배포한다.