# 27강. 에러 핸들러 3 - try/catch 1

## `try/catch` 문

---

- 솔리디티(Solidity) 0.6 버전부터 도입
- 스마트 컨트랙트에서 예외 처리를 세밀하게 관리할 수 있도록 해준다.
- 프로그램이 예상치 못한 에러로 종료되는 것을 방지하고, 에러 발생 시 적절한 대처를 할 수 있다.

### 특징

---

1. 에러 핸들링의 개선: 기존의 `assert`, `revert`, `require` 문은 에러 발생 시 프로그램을 종료시켰지만, `try/catch` 문을 사용하면 에러를 잡아내어 프로그램이 계속 실행될 수 있도록 한다.
2. 다양한 `catch` 블록: 솔리디티의 `catch` 블록은 세 가지 유형이 있다.
    - `catch Error(string memory reason)`: `revert`나 `require`에 의해 생성된 에러를 처리한다.
    - `catch Panic(uint errorCode)`: `assert`에 의해 발생하는 패닉 에러를 처리하며, 0.8.1 버전부터 지원된다.
    - `catch (bytes memory lowLevelData)`: 로우 레벨 에러를 처리한다.

### 활용 사례

---

1. 외부 스마트 컨트랙트의 함수 호출 시
    - 다른 스마트 컨트랙트의 함수를 호출할 때, 해당 함수에서 발생하는 에러를 `try/catch` 문으로 처리할 수 있다.
2. 외부 스마트 컨트랙트의 생성 시
    - 새로운 스마트 컨트랙트를 생성할 때 발생할 수 있는 에러를 `try/catch` 문으로 관리할 수 있다.
3. 내부 함수 호출 시
    - 동일한 스마트 컨트랙트 내에서 함수를 호출할 때도 `try/catch` 문을 사용하여 에러를 처리할 수 있다. 이때 `this` 키워드를 사용하여 현재 컨트랙트를 참조한다.

## 예제

---

- lec27.sol
    
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
            
    /*
    catch Panic(uint errorCode) { ... } :  assert 를 통해 생선된 에러가 날 때 이 catch에 잡혀요. 
    
    errorCode 는  솔리디티에서 정의 Panic 에러 별로 나온답니다. 
    0x00: Used for generic compiler inserted panics.
    0x01: If you call assert with an argument that evaluates to false.
    0x11: If an arithmetic operation results in underflow or overflow outside of an unchecked { ... } block.
    0x12; If you divide or modulo by zero (e.g. 5 / 0 or 23 % 0).
    0x21: If you convert a value that is too big or negative into an enum type.
    0x22: If you access a storage byte array that is incorrectly encoded.
    0x31: If you call .pop() on an empty array.
    0x32: If you access an array, bytesN or an array slice at an out-of-bounds or negative index (i.e. x[i] where i >= x.length or i < 0).
    0x41: If you allocate too much memory or create an array that is too large.
    0x51: If you call a zero-initialized variable of internal function type.
    예를들어, 나누기가 0 이 된다면 0x12(=18) errorCode 를 리턴해요.
    */
    contract math{    
        function division(uint256 _num1,uint256 _num2) public pure returns (uint256){
            require(_num1<10,"num1 shoud not be more than 10");//
            return _num1/_num2; //6/3 => 2
        }
    }
    
    contract runner{
        event catchErr(string _name,string _err);
        event catchPanic(string _name,uint256 _err);
        event catchLowLevelErr(string _name,bytes _err);
        
        math public mathInstance = new math() ;
        
        function playTryCatch(uint256 _num1, uint256 _num2) public returns(uint256,bool){
            try mathInstance.division(_num1, _num2) returns(uint256 value){
                return(value,true);
            } catch Error(string memory _err) {
                emit catchErr("revert/require",_err);
                return(0,false);
            } catch Panic(uint256 _errorCode) {
                emit catchPanic("assertError/Panic",_errorCode);
                return(0,false);
            } catch (bytes memory _errorCode) {
                emit catchLowLevelErr("LowlevelError",_errorCode);
                return(0,false);
            }
        } 
    }
    ```
    
    - 실행
        - runner 컨트랙트 배포
            
            ![image.png](27%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%203%20-%20try%20catch%201%201c7b5a48de44808fb966f886186366f5/image.png)
            
            - `require` 실패: 프로그램이 끝나지 않는다.
                
                ![image.png](27%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%203%20-%20try%20catch%201%201c7b5a48de44808fb966f886186366f5/image%201.png)
                
            - `panic` 실패: 프로그램이 끝나지 않는다.
                
                ![image.png](27%E1%84%80%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A6%E1%84%85%E1%85%A5%20%E1%84%92%E1%85%A2%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A5%203%20-%20try%20catch%201%201c7b5a48de44808fb966f886186366f5/image%202.png)
                
                - `"_err": "18"` 부분이 `0x12` 에러코드라는 의미다.