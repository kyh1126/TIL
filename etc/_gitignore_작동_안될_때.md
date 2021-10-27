# .gitignore 작동 안될 때

- `.gitignore` 파일에 추가했는데 커밋 대상에서 제외가 안되는 경우 ex> `.idea/` 제외하려고 할 때
    
    ```bash
    git rm -r --cached -r .idea
    ```
    
    - git 캐시가 문제이기 때문에 캐시를 삭제해주면 된다.
