# 18강. Array 배열

## 정의

---

- 배열: 값을 넣거나 뺄 수 있다.
    - 아직 배열이 생성되지도 않았는데 값을 넣으면 오류가 난다.
    - Solidity에서는 배열보다 `mapping`을 더 선호한다.
        - 배열 순환을 악의적으로 디도스 공격해서 과부화시킬 수 있기 때문이다.
        - 배열 길이를 50으로 제한시켜서 사용하는 것이 좋다.
    
    ```solidity
    uint256[] public ageArray;
    ```
    
- `mapping`과 다르게 `length`값을 구할 수 있다.
    
    ```solidity
    function AgeLength()public view returns(uint256) {
        return ageArray.length;
    }
    ```
    
- `push`: 값을 넣어 준다.
    
    ```solidity
    function AgePush(uint256 _age)public{
        ageArray.push(_age);
    }
    ```
    
- `pop`: 가장 최신의 값을 지운다. `length`도 줄어든다.
    
    ```solidity
    function AgePop()public {
        ageArray.pop();
    }
    ```
    
- `delete`: 원하는 인덱스의 값을 지운다. 그러나 `length`는 줄어들지 않는다.
    
    ```solidity
    function AgePop(uint256 _index)public {
        delete ageArray[_index];
    }
    ```
    
- 애초에 배열값을 미리 넣어주거나, 사이즈를 미리 넣어줘 제한을 걸 수도 있다.
    
    ```solidity
    uint256[10] public ageFixedSizeArray;
    string[] public nameArray= ["Kal","Jhon","Kerri"];
    ```
    

## 예제

---

- lec18.sol
    
    ```solidity
    // SPDX-License-Identifier:GPL-30
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec18{
        uint256[] public ageArray;
        uint256[10] public ageFixedSizeArray;
        string[] public nameArray= ["Kal","Jhon","Kerri"];
      
        function AgeLength()public view returns(uint256) {
            return ageArray.length;
        }
        
        function AgePush(uint256 _age)public{
            ageArray.push(_age);
        }
        function AgeChange(uint256 _index, uint256 _age)public{
            ageArray[_index] = _age;
        }
        function AgeGet(uint256 _index)public view returns(uint256){
            return ageArray[_index];
        }
        function AgePop()public {
            ageArray.pop();
        }
        
        function AgePop(uint256 _index)public {
            delete ageArray[_index];
        }
    }
    ```