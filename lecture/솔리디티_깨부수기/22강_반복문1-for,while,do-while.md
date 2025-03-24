# 22강. 반복문 1 - for,while,do-while

- 반복문: 주로 배열이나 매핑과 같은 데이터 구조를 순회하거나 특정 조건이 충족될 때까지 코드를 반복 실행하는 데 사용된다.

## for

---

- `for` 문: 반복 횟수가 정해져 있을 때 주로 사용된다.
    
    ```solidity
    for (초기화; 조건; 증감) {
        // 실행할 코드
    }
    ```
    
    - 초기화: 반복을 시작하기 전에 한 번 실행되며, 반복 제어 변수를 초기화합니다.
    - 조건: 각 반복 전에 평가되며, 참(`true`)인 동안 루프가 계속 실행됩니다.
    - 증감: 각 반복의 끝에서 실행되며, 제어 변수를 업데이트합니다.

## while

---

- `while` 문: 반복 횟수가 정해져 있지 않고, 특정 조건이 충족될 때까지 반복해야 할 때 사용된다.
    
    ```solidity
    while (조건) {
        // 실행할 코드
    }
    ```
    

## do-while

---

- `do-while` 문: 최소 한 번은 코드를 실행한 후, 조건을 검사하여 반복 여부를 결정한다.
    
    ```solidity
    do {
        // 실행할 코드
    } while (조건);
    ```
    

## 예제

---

- lec22.sol
    
    ```solidity
    // SPDX-License-Identifier:GPL-30
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec22 {
        // for
        event CountryIndexName(uint256 indexed _index, string _name);
        string[] private countryList = ["South Korea","North Korea","USA","China","Japan"];
    
        function forLoopEvents() public {
            for(uint256 i =0; i<countryList.length; i++){
            emit CountryIndexName(i,countryList[i]);
            }
        }
    
        // while
        function whileLoopEvents() public {
            uint256 i = 0;
            while(i<countryList.length){
                 emit CountryIndexName(i,countryList[i]);
                 i++;
            }
        }
    
        // do-while
            function doWhileLoopEvents() public {
            uint256 i = 0;
            do{
                emit CountryIndexName(i,countryList[i]);
                i++;
            }
            while(i<countryList.length);
        }
    }
    ```
