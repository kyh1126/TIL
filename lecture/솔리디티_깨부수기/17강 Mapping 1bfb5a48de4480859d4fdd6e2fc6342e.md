# 17강. Mapping

## 정의

---

- `Mapping`: `Key`와 `Value` 값이 존재한다. `length`를 알 수는 없다.
    
    ```solidity
    mapping(uint256=>uint256) private ageList;
    ```
    

## 예제

---

- lec17.sol
    
    ```solidity
    // SPDX-License-Identifier:GPL-30
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec17{
        mapping(string=>uint256) private priceList;
        mapping(uint256=>string) private nameList;
        mapping(uint256=>uint256) private ageList;
        
        
        function setAgeList(uint256 _key,uint256 _age) public {
            ageList[_key] = _age;
        }
        
        function getAge(uint256 _key) public view returns(uint256){
            return ageList[_key];
        }
        
        function setNameList(uint256 _key,string memory _name) public {
            nameList[_key] = _name;
        }
        
        function getName(uint256 _key) public view returns(string memory){
            return nameList[_key];
        }
        
        function setPriceList(string memory _itemName,uint256 _price) public {
            priceList[_itemName] = _price;
        }
        
        function getPriceList(string memory _key) public view returns(uint256){
            return priceList[_key];
        }
        
    }
    
    ```