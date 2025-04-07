# 2장. 이더리움 IDE - Remix 입문

## 이더리움 IDE - Remix 입문

---

- Remix: 이더리움 뿐 아니라 탈중앙화 토큰, 탈중앙화, DApp을 전반적으로 이해하고 DApp을 작성하는 기술을 배우는 데 도움을 준다.
- 콘솔: 맨 아래 있는 부분. 코드 작성 시 도움말 박스.
    - 코드에 문제가 생겼거나 작성할 때 실수가 있었는지 알 수 있다.
    - 디버거 역할을 하면서 블록체인의 거래를 기록하는 데 사용할 것이다.

## 문서 편집기 개요

---

- Visual Studio Code: 문서 편집기
    - 코드를 작성한다.
- ex>
    
    ```powershell
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 ~ % cd defi-staking-app/
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 defi-staking-app % code .
    ```
    
- Remix IDE가 이와 비슷한 환경을 가지고 있다.

## IDE Solidity 컴파일러 & 배포 도구

---

- `.sol` 파일: Solidity를 의미한다.
    - 파일을 실행하면 파일이 정보를 저장하는 데이터 타입을 알고 있다.
- SOLIDITY COMPILER: 컨트랙트를 컴파일하는 방식을 정할 수 있다.
- DEPLOY & RUN TRANSACTIONS: 블록체인에서 거래가 체결되는 것이다.
- SOLIDITY STATIC ANALYSIS: 컨트랙트 및 정보를 분석해준다.
    - Gax Economy나 Security, Miscellaneous등을 이해하는 데 도움을 준다.
- SOLIDITY UNIT TESTING: 테스트 환경에 영구적으로 배포될 실제 스마트 컨트랙트를 작성할 때 필요하다.

## 스마트 계약에 대한 개요

---

- 스마트 계약: 두 계약 당사자 간에 이뤄지는 합의
    - 스마트: 제3자가 컴퓨터 프로그램으로 대체되기 때문이다.
    - 스마트 계약에 들어가는 코드는 양 당사자를 위한 것
    - 장점: 더 많은 시스템이 블록체인 네트워크를 활용할 수 있다.
- 스마트 계약을 사용하면 자체 토큰도 만들 수 있다. 그래서 페그(peg)할 수 있다.
    - 블록체인의 네이티브 화폐로 묶는 것