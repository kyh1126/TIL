# 39강. Interface 인터페이스

## interface

---

- `interface`: 다른 스마트 컨트랙트와의 상호작용을 위해 사용하는, 함수의 시그니처만 선언하고 실제 구현은 포함하지 않는다.

### 특징

---

- 함수의 구현 없이 함수 시그니처만 정의
- 상태 변수 선언 불가능
- 생성자(constructor) 정의 불가능
- 함수는 모두 `external`이어야 함
- 다른 인터페이스를 상속 가능

❗인터페이스를 구현하는 경우, 해당 인터페이스에 선언된 모든 함수를 구현해야 한다.

- 만약 일부 함수만 구현하고 나머지를 구현하지 않으면, 해당 컨트랙트는 추상 컨트랙트로 간주되어 직접 배포할 수 없다.
    - 이러한 경우, 추상 컨트랙트를 상속하는 다른 컨트랙트에서 나머지 함수를 구현해야 한다.
- Solidity 0.6.0 버전부터는 인터페이스의 함수를 구현할 때 `override` 키워드를 사용하여 해당 함수가 부모 인터페이스의 함수를 재정의함을 명시해야 한다.

## 예제

---

- lec39.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 < 0.9.0;
    
    /*
    interface : 스마트컨트랙 내에서 정의되어야할 필요한 것 
    1, 함수는 external로 표시
    2, enum, structs 가능 
    3, 변수, 생성자 불가(constructor X)
    */
    interface ItemInfo {
        struct item{
            string name;
            uint256 price;
        }
        function addItemInfo(string memory _name,uint256 _price) external;
        function getItemInfo(uint256 _index) external view returns(item memory);
    }
    
    contract lec39 is ItemInfo {
        item[] public itemList;
        function addItemInfo(string memory _name,uint256 _price) override public {
            itemList.push(item(_name,_price));
        }
        function getItemInfo(uint256 _index) override public view returns(item memory) {
            return itemList[_index];
        }
    }
    ```