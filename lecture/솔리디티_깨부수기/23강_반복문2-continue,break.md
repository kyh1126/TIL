# 23강. 반복문 2 - continue, break

- Solidity의 반복문에서 `break`와 `continue` 문은 코드의 흐름을 제어하는 데 사용된다.

## continue

---

- `continue` 문: 현재 반복의 나머지 코드를 건너뛰고, 다음 반복을 시작한다.
    - 특정 조건에서만 코드를 실행하고 싶을 때 유용하다.
    
    ```solidity
    pragma solidity ^0.8.0;
    
    contract ContinueExample {
        function filterEvenNumbers(uint[] memory numbers) public pure returns (uint[] memory) {
            uint[] memory evenNumbers = new uint[](numbers.length);
            uint count = 0;
            for (uint i = 0; i < numbers.length; i++) {
                if (numbers[i] % 2 != 0) {
                    continue; // 홀수이면 다음 반복으로 건너뜀
                }
                evenNumbers[count] = numbers[i];
                count++;
            }
            return evenNumbers;
        }
    }
    ```
    

## break

---

- `break` 문: 반복문을 즉시 종료하고, 반복문 다음에 오는 코드로 제어를 이동시킨다.
    - 주로 특정 조건이 충족되었을 때 반복을 중단하기 위해 사용된다.
    
    ```solidity
    pragma solidity ^0.8.0;
    
    contract BreakExample {
        function findNumber(uint[] memory numbers, uint target) public pure returns (bool) {
            for (uint i = 0; i < numbers.length; i++) {
                if (numbers[i] == target) {
                    return true; // 목표 숫자를 찾으면 반복문 종료
                }
            }
            return false; // 목표 숫자가 배열에 없음
        }
    }
    ```
    

## 예제

---

- lec23.sol
    
    ```solidity
    // SPDX-License-Identifier:GPL-30
    pragma solidity >= 0.7.0 < 0.9.0;
    
    contract lec23 {
    
        event CountryIndexName(uint256 indexed _index, string _name);
        string[] private countryList = ["South Korea","North Korea","USA","China","Japan"];
    
        function useContinue() public {
            for(uint256 i =0; i<countryList.length; i++){
                if(i%2==1){ // odd
                    continue;
                }
                emit CountryIndexName(i, countryList[i]);
            }
        }
    
        function useBreak() public {
            for(uint256 i =0; i<countryList.length; i++){
                if(i==2){
                    break;
                }
                emit CountryIndexName(i, countryList[i]);
            }
        }
    }
    ```