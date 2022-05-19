# Git Internals

## 1. 문제

---

- `main` 브랜치와 `dev` 브랜치의 히스토리가 꽤 차이가 나는 상황
- `main`에서 많은 파일들이 수정됐어야 했고, 커밋 후 놓친 부분이 있어 핫픽스로 커밋한 다음, `dev`에 `merge`함
- 이 때, 많은 충돌이 발생하여 이를 해결

![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZTzLNNxaqttG2MzzSR3xEgsCQLFQ8PcqfUxoaoxRuF8mkspdFWZbEqEm68SSuZTGL7E3SHCnyhH9aKjDx3jWesLfAmgTfygvwne8Pg3RwiLNCMVDJv8QoTT4USbNNasQL4e4854cV8WgE84P4LcoUTbbTB/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZTzLNNxaqttG2MzzSR3xEgsCQLFQ8PcqfUxoaoxRuF8mkspdFWZbEqEm68SSuZTGL7E3SHCnyhH9aKjDx3jWesLfAmgTfygvwne8Pg3RwiLNCMVDJv8QoTT4USbNNasQL4e4854cV8WgE84P4LcoUTbbTB/Image.jpg)

- 그런데, 충돌을 해결하고 나서 찾아보니 또 놓친 부분이 있어서(😅) 핫픽스로 커밋하고 다시 `dev`에 `merge`함
- 결국 Merge를 두번하게 된 예쁘지 않은 히스토리가 나오게됨
    
    ![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZd4RJ6E4Fy2cVNakn7BdhYKVuxz5pGqhb1q74J74BGFG5VYTPhskvkY5Rydj9cMXiSMFX5fzXX1nkajRCRVDqFSKEzaKXEtjFkpWnNxa1B1dNqaSg3NiTk7nAxpUzNm51Rs7uFprZBbUqsKRCynfuBtWkB/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZd4RJ6E4Fy2cVNakn7BdhYKVuxz5pGqhb1q74J74BGFG5VYTPhskvkY5Rydj9cMXiSMFX5fzXX1nkajRCRVDqFSKEzaKXEtjFkpWnNxa1B1dNqaSg3NiTk7nAxpUzNm51Rs7uFprZBbUqsKRCynfuBtWkB/Image.jpg)
    

## 2. 하고 싶은 것

---

- `[HOTFIX] head.php 회사소개 sfresh 링크 복구` 다음 merge한 커밋을 없애고 싶다.

```bash
8656838 (HEAD -> dev) Merge branch 'main' into dev
|\
| * 03e293f (main) [HOTFIX] sfreshmall.com/shop -> /shop으로 수정
* | ceaf5f9 Merge branch 'main' into dev 👈👈👈이 커밋을 제거하고 싶다!
|\|
| * 7f46035 (origin/main) [HOTFIX] head.php 회사소개 sfresh 링크 복구
| * 6c1eaa0 [SMR-47] 하드코딩된 http 제거
| * 7d9ff8c [HOTFIX] GTM 아이디값 수정
| * f81119f [SMR-42] Google Tag Manager 추가
* | 78f177f (origin/dev) [SMR-45] 상품 일괄변경 검색 및 엑셀 다운로드 구현
* | b1d49fc Merge branch 'main' into dev
```

```bash
??????? (HEAD -> dev) Merge branch 'main' into dev
|\
| * 03e293f (main) [HOTFIX] sfreshmall.com/shop -> /shop으로 수정
| * 7f46035 (origin/main) [HOTFIX] head.php 회사소개 sfresh 링크 복구
| * 6c1eaa0 [SMR-47] 하드코딩된 http 제거
| * 7d9ff8c [HOTFIX] GTM 아이디값 수정
| * f81119f [SMR-42] Google Tag Manager 추가
* | 78f177f (origin/dev) [SMR-45] 상품 일괄변경 검색 및 엑셀 다운로드 구현
* | b1d49fc Merge branch 'main' into dev
```

- `rebase`는 merge commit을 수정할 수 없다.
- `--rebase-merges`와 함께라면 가능하다고 한다.
- 하지만 `squash`는 불가능하다.

![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZArqWm49pMANAdK8VFWcHgYuXPJF2HVstGsxq8aR3tyMZ4mt3WtUBHXNQv8pkKMaC6RZuxv4dtTRuqxAvYWoTNBzR9hNkXFGD9vSVP43HHkRyiV1RZrc2DF33zXXfB9M5rttRDZRbbWuywcDJnJv4G93Dt/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZArqWm49pMANAdK8VFWcHgYuXPJF2HVstGsxq8aR3tyMZ4mt3WtUBHXNQv8pkKMaC6RZuxv4dtTRuqxAvYWoTNBzR9hNkXFGD9vSVP43HHkRyiV1RZrc2DF33zXXfB9M5rttRDZRbbWuywcDJnJv4G93Dt/Image.jpg)

![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZVTU83bCAbycBumeBB7L9QKYw9MDGjBT5gsmRbreFr2TkbaTBc9FnrVXcScWa44xeHDJ1G77udYFQyQxa4Kd24jC6HF7CwV4SYt6d2as2hc4CkMzwkWqQ9s4f483qq9wCiQsVvSSBhFo5ctSVu7xmkAQyc/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZVTU83bCAbycBumeBB7L9QKYw9MDGjBT5gsmRbreFr2TkbaTBc9FnrVXcScWa44xeHDJ1G77udYFQyQxa4Kd24jC6HF7CwV4SYt6d2as2hc4CkMzwkWqQ9s4f483qq9wCiQsVvSSBhFo5ctSVu7xmkAQyc/Image.jpg)

- 그렇다고 이전 커밋으로 `reset`해서 다시 `merge`하려고 하니, **충돌을 다시 해결해야 된다** → 이건 너무 귀찮다!

## 3. 해결

---

- 구글링을 하니, 역시 [스택오버플로우](https://stackoverflow.com/questions/58802891/how-to-squash-subsequent-merge-commits)에 누군가 비슷한 질문을 했었다.

![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZgGh5iPMiyHVSYdKSC77GgRJKkWXgsotxzPLJgqmnG8wduQgvPuUKUhjMCie2evqNdAZ8PQvkSVe3QKNhNY9aRajBrL977tyjEdSHBXZX1VBSBGyoraGnwRRUTwRvX4anBD8zWUJ6wtVoYrEEQ1mSAjhb3/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZgGh5iPMiyHVSYdKSC77GgRJKkWXgsotxzPLJgqmnG8wduQgvPuUKUhjMCie2evqNdAZ8PQvkSVe3QKNhNY9aRajBrL977tyjEdSHBXZX1VBSBGyoraGnwRRUTwRvX4anBD8zWUJ6wtVoYrEEQ1mSAjhb3/Image.jpg)

- `git commit-tree`의 사용법은 아래와 같다.
    
    ```bash
    # https://git-scm.com/docs/git-commit-tree
    git commit-tree <tree> [(-p <parent>)…] [-S[<keyid>]] [(-m <message>)…]
    		  [(-F <file>)…] <tree>
    ```
    

- 현재 나의 상황의 경우
    - 현재의 HEAD 커밋인 `8656838 (HEAD -> dev) Merge branch 'main' into dev` 커밋을
    - 첫번째 부모커밋(`dev` 커밋)과
    - 두번째 부모커밋(`master` 커밋)을 부모로 하는 커밋으로 변경한다.

```bash
8656838 (HEAD -> dev) Merge branch 'main' into dev
|\
| * 03e293f (main) [HOTFIX] sfreshmall.com/shop -> /shop으로 수정 👈👈👈 그대로 두번째 부모커밋
* | ceaf5f9 Merge branch 'main' into dev 👈👈👈 부모(였던 것)
|\|
| * 7f46035 (origin/main) [HOTFIX] head.php 회사소개 sfresh 링크 복구
| * 6c1eaa0 [SMR-47] 하드코딩된 http 제거
| * 7d9ff8c [HOTFIX] GTM 아이디값 수정
| * f81119f [SMR-42] Google Tag Manager 추가
* | 78f177f (origin/dev) [SMR-45] 상품 일괄변경 검색 및 엑셀 다운로드 구현 👈👈👈 새로운 첫번째 부모커밋
* | b1d49fc Merge branch 'main' into dev
===
d m
```

- 따라서 다음과 같이 한다.
    
    ```bash
    $ git checkout \
    > $( git commit-tree HEAD^{tree} -p 78f177f -p 03e293f -m "Merge branch 'main' into dev")
    ```
    
- `commit-tree`의 결과는 해당 커밋 오브젝트이다.
- 그 커밋 오브젝트로 `checkout` 한다.

![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZmMYVjA4KLS2Mv6gCUyHxXe3odBRRwcqvwrVxnUJKek7BqZupnJ5XD2ku2xEofS5ULQ7fLmNuWDLUWBFALmbEi55ATjtG4P2XytFaX5QxFjNbef74CmLB9GqgDWJoALdB2L6rMPjk8h4tbmiJmtEukgXZ8/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZmMYVjA4KLS2Mv6gCUyHxXe3odBRRwcqvwrVxnUJKek7BqZupnJ5XD2ku2xEofS5ULQ7fLmNuWDLUWBFALmbEi55ATjtG4P2XytFaX5QxFjNbef74CmLB9GqgDWJoALdB2L6rMPjk8h4tbmiJmtEukgXZ8/Image.jpg)

- 이제, 현재 커밋을 `dev` 브랜치가 강제로 가리키게 한다.
    
    ```bash
    # dev 브런치를 강제로 현재 커밋으로 가리키게 한다.
    $ git branch -f dev
    
    # dev로 체크아웃
    $ git checkout dev
    ```
    
- 이제 커밋 로그를 살펴보면
    
    ```bash
    # before
    *   8656838 (HEAD -> dev) Merge branch 'main' into dev
    |\
    | * 03e293f (main) [HOTFIX] sfreshmall.com/shop -> /shop으로 수정
    * | ceaf5f9 Merge branch 'main' into dev
    |\|
    | * 7f46035 (origin/main) [HOTFIX] head.php 회사소개 sfresh 링크 복구
    | * 6c1eaa0 [SMR-47] 하드코딩된 http 제거
    | * 7d9ff8c [HOTFIX] GTM 아이디값 수정
    | * f81119f [SMR-42] Google Tag Manager 추가
    * | 78f177f (origin/dev) [SMR-45] 상품 일괄변경 검색 및 엑셀 다운로드 구현
    
    # after
    *   e93d109 (HEAD -> dev) Merge branch 'main' into dev
    |\
    | * 03e293f (main) [HOTFIX] sfreshmall.com/shop -> /shop으로 수정
    | * 7f46035 (origin/main) [HOTFIX] head.php 회사소개 sfresh 링크 복구
    | * 6c1eaa0 [SMR-47] 하드코딩된 http 제거
    | * 7d9ff8c [HOTFIX] GTM 아이디값 수정
    | * f81119f [SMR-42] Google Tag Manager 추가
    * | 78f177f (origin/dev) [SMR-45] 상품 일괄변경 검색 및 엑셀 다운로드 구현
    ```
    

### 문제는 해결됐지만, 어떻게 가능했을까?

---

- `commit-tree`가 무슨 역할을 하길래?
- `tree` 오브젝트는 무엇인가?
- 충돌이 해결된 변경사항을 가진 `ceaf5f9` 가 어떻게 `8656838`과 어우러져 새로운 `e93d109` 커밋이 만들어졌는가?
- 이 때, 깃 내부가 어떻게 돌고 있는지 전혀 모르고 있었다는 사실을 깨닫게 됐다.

## 4. Git Objects

---

- 깃은 key-value 로 이루어진 일종의 DB라고 볼 수 있다.
- key는 SHA-1 해시값이다.
- value는 컨텐츠다.
- key-value로 저장되는 데이터를 오브젝트라고 보면 된다.
- 이들은 `.git/objects`에 저장된다.
- 중요한 3가지의 오브젝트는 다음과 같다.
    - blob
    - tree
    - commit

### blob

---

- 파일이라고 생각하면 된다.
- 파일의 내용을 가지고 있다.

### tree

---

- 폴더라고 생각하면 된다.
- `blob`과 또 다른 `tree`를 가리킨다.

### commit

---

- 커밋
- 하나의 `tree`를 가리킨다.
- 0개 이상의 부모를 가리킨다.
    - 부모가 없는 경우: 최초의 커밋인 루트 커밋
    - 부모가 1개인 경우: 일반적인 커밋
    - 부모가 2개 이상인 경우: merge 커밋
- 커밋 오브젝트가 생성된 날짜 정보
- 커밋 오브젝트를 만든 유저의 정보 (git config의 user.name과 user.email 등)
- 커밋 메세지 정보

### 파일 상태 라이프사이클

---

- 기본적인 내용이지만, 용어의 정리를 위해 한 번 다시 살펴보도록 하자.
    
    ![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZAhqbsWjFUroV84nA3QwdBcK6kd5ZR1ry1VtCtT6DUUSTW3qVbkLrRC3ogfEnbc7kZq3Lij2bahW6yNy17VBaYMsDQYyaj2JnmvSLRTDuJK17ZzLWWzXavzF5NFqtipMACPPBi2fKnKU3FtL2DaeHujjFG/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZAhqbsWjFUroV84nA3QwdBcK6kd5ZR1ry1VtCtT6DUUSTW3qVbkLrRC3ogfEnbc7kZq3Lij2bahW6yNy17VBaYMsDQYyaj2JnmvSLRTDuJK17ZzLWWzXavzF5NFqtipMACPPBi2fKnKU3FtL2DaeHujjFG/Image.jpg)
    
- `Untracked`는 Git에 의해 추적되지 않는 파일
- `Staging`을 하면 (`git add`) 인덱싱 되며 추적되기 시작한다.
    - 인덱싱된 파일은 Staging area에 해당 파일이 복사된다.
    - 해당 복사본과 현재 워킹 디렉터리의 파일과 비교한다.
        - 같으면 `Unmodified`
        - 다르면 `Modified`
- `git commit`을 하면 Staging area에 있던 파일들이 오브젝트로 영속된다.
    
    ![https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZP9evJB2WmKdeyEqAW61jG8KvKqQxxvejR9C7X31nhsdUheUTm7YwccpW1x1ZJfdzsKLpNRxZhc7iHz8hwkaRpukoyXbftuPFxQYtPmryJV1hL3J3Lbp68bGGQLDMRoyBspr9ozLszbnQGBEWjU1cBmYj8/Image.jpg](https://secure-res.craft.do/v1/2A7273S5LqVuwFo2M4VKb1RYsFGcLHnaN2zVDMyfDmjvK6yC6azvrz5dyNbRsmHvV3TZ5zupzN4XftXkd5NKwArRnjnZxqbb3j9NEH4HrcEPnYtA1k7ttYKPf9uXB5dJubkZP9evJB2WmKdeyEqAW61jG8KvKqQxxvejR9C7X31nhsdUheUTm7YwccpW1x1ZJfdzsKLpNRxZhc7iHz8hwkaRpukoyXbftuPFxQYtPmryJV1hL3J3Lbp68bGGQLDMRoyBspr9ozLszbnQGBEWjU1cBmYj8/Image.jpg)
    
- `staging area`를 다른말로는 `index` 라고도 부르기도 한다.
- `git add`는 `index`에 파일을 추가하는 것이고, 이는 스테이징이라는 표현을 보통 사용한다.

## 5. low-level command 들을 활용하여 히스토리를 쌓아보자

---

### `git hash-object`

---

- 파일의 내용을 바탕으로 `blob` 오브젝트를 만든다.
    
    ```bash
    $ echo 'Hello, SCN' | git hash-object -w --stdin
    34e5563c9ee0da4f494200508ca60eab841b0f36
    
    $ git cat-file -p 34e5563
    Hello, SCN
    
    $ git cat-file -t 34e5563
    blob
    ```
    

### `git update-index`

---

- 파일 또는 object 를 스테이징한다.
    
    ```bash
    # object를 바로 스테이징 했기 때문에
    # git status를 치면 deleted 상태로 나타난다.
    $ git status
    On branch main
    
    No commits yet
    
    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
    	new file:   scn.txt
    	new file:   sfn.txt
    
    Changes not staged for commit:
      (use "git add/rm <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
    	deleted:    scn.txt
    ```
    
    ```bash
    # hello.txt 파일 생성
    $ echo 'Hello, SFN' > sfn.txt
    
    # working directory에 있는 파일을 스테이징
    $ git update-index --add sfn.txt
    
    # object를 바로 스테이징
    $ git update-index --add --cacheinfo \
    > 100644 34e5563c9ee0da4f494200508ca60eab841b0f36 scn.txt
    ```
    

### `git write-tree`

---

- 현재 인덱싱(스테이징)된 요소들을 `tree` 오브젝트로 만든다.
    
    ```bash
    $ git write-tree
    9cdf63e446fa97aa87ab12f40cf2450f6766a03d
    
    $ find .git/objects -type f
    .git/objects/34/e5563c9ee0da4f494200508ca60eab841b0f36 # scn.txt
    .git/objects/9c/df63e446fa97aa87ab12f40cf2450f6766a03d # tree
    .git/objects/17/67eb661906752863f2b3c2b178fb7d056c3b2b # sfn.txt
    ```
    
    ```bash
    $ git cat-file -p 34e5563
    Hello, SCN
    
    $ git cat-file -p 9cdf63e
    100644 blob 34e5563c9ee0da4f494200508ca60eab841b0f36 scn.txt
    100644 blob 1767eb661906752863f2b3c2b178fb7d056c3b2b sfn.txt
    
    $ git cat-file -p 1767eb6
    Hello, SFN
    ```
    

### `git commit-tree`

---

- `tree` 오브젝트를 가리키는 `commit` 오브젝트를 만든다.
- 최초 커밋이므로 `-p`로 부모를 지정하지 않는다.
- `commit` 오브젝트는 author, committer, date, message 등의 정보를 이용해 해시를 만들기 때문에 실행환경마다 다를 수 밖에 없다.
    
    ```bash
    $ git commit-tree 9cdf63e446fa97aa87ab12f40cf2450f6766a03d -m "init"
    897db30ab07cb26c4e49194b3ba0395052be6e5b
    
    $ git cat-file -p 897db30
    tree 9cdf63e446fa97aa87ab12f40cf2450f6766a03d
    author Taehoon (Nio) Kim <taehoon.kim@smartfoodnet.com> 1652854220 +0900
    committer Taehoon (Nio) Kim <taehoon.kim@smartfoodnet.com> 1652854220 +0900
    
    init
    ```
    
    ```bash
    $ find .git/objects -type f
    .git/objects/34/e5563c9ee0da4f494200508ca60eab841b0f36 # scn.txt
    .git/objects/9c/df63e446fa97aa87ab12f40cf2450f6766a03d # tree
    .git/objects/89/7db30ab07cb26c4e49194b3ba0395052be6e5b # commit
    .git/objects/17/67eb661906752863f2b3c2b178fb7d056c3b2b # sfn.txt
    ```
    

### `git read-tree`

---

- 특정 `tree`를 읽어서 스테이징한다.
    
    ```bash
    $ git read-tree --prefix=bak 9cdf63e
    $ git status
    On branch main
    
    No commits yet
    
    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
    	new file:   bak/scn.txt
    	new file:   bak/sfn.txt
    	new file:   scn.txt
    	new file:   sfn.txt
    
    Changes not staged for commit:
      (use "git add/rm <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
    	deleted:    bak/scn.txt
    	deleted:    bak/sfn.txt
    	deleted:    scn.txt
    ```
    
    ```bash
    $ git write-tree
    6b90661eb520ef2062ba18bd5da8fb24146678ae
    
    $ git commit-tree 6b90661 -p 897db30 -m "bak"
    75738d1e78288a343a7e0d1918bca98d9ae2e90c
    ```
    

### `git checkout && git branch -f`

---

- 이제, 우리가 만든 커밋으로 체크아웃 하고 `main` 브런치가 가리키게 해보자.
    
    ```bash
    $ git checkout 75738d1
    D	bak/scn.txt
    D	bak/sfn.txt
    D	scn.txt
    Note: switching to '75738d1'.
    
    You are in 'detached HEAD' state. You can look around, make experimental
    changes and commit them, and you can discard any commits you make in this
    state without impacting any branches by switching back to a branch.
    
    If you want to create a new branch to retain commits you create, you may
    do so (now or later) by using -c with the switch command. Example:
    
      git switch -c <new-branch-name>
    
    Or undo this operation with:
    
      git switch -
    
    Turn off this advice by setting config variable advice.detachedHead to false
    
    HEAD is now at 75738d1 bak
    ```
    
    ```bash
    $ git branch -f main
    $ git checkout main
    D	bak/scn.txt
    D	bak/sfn.txt
    D	scn.txt
    Switched to branch 'main'
    ```
    
    ```bash
    $ git log --oneline
    75738d1 (HEAD -> main) bak
    897db30 init
    ```
    

❗이렇게 커밋 히스토리가 만들어졌다.

## 6. 파일을 수정하고 어떻게 되는지 살펴보자

---

```bash
$ git status
On branch main
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    bak/scn.txt
	deleted:    bak/sfn.txt
	deleted:    scn.txt

$ echo 'Bye, SFN!' >> sfn.txt
$ git status
On branch main
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    bak/scn.txt
	deleted:    bak/sfn.txt
	deleted:    scn.txt
	modified:   sfn.txt
```

```bash
$ git update-index sfn.txt
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   sfn.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    bak/scn.txt
	deleted:    bak/sfn.txt
	deleted:    scn.txt
```

```bash
$ git commit -m "update"
[main f8f3523] update
 1 file changed, 1 insertion(+)

$ git log -3 --oneline
f8f3523 (HEAD -> main) update
75738d1 bak
897db30 init

$ find .git/objects -type f
.git/objects/34/e5563c9ee0da4f494200508ca60eab841b0f36 # scn.txt
.git/objects/9c/df63e446fa97aa87ab12f40cf2450f6766a03d # tree (처음 write-tree)
.git/objects/89/7db30ab07cb26c4e49194b3ba0395052be6e5b # commit (init)
.git/objects/17/67eb661906752863f2b3c2b178fb7d056c3b2b # sfn.txt
.git/objects/75/738d1e78288a343a7e0d1918bca98d9ae2e90c # commit (bak)
.git/objects/86/50f5bc38892e76d33091378dc6d90dfa2bc314 # ???
.git/objects/44/4a20bab66fca19d8e43cd041d0bf16e0aacc97 # ???
.git/objects/6b/90661eb520ef2062ba18bd5da8fb24146678ae # tree (다음 write-tree)
.git/objects/f8/f3523473df20d30198ec5ce836af891ffebc34 # commit (update)
```

### 새로 생긴 오브젝트들이 무엇인지 살펴보자

---

```bash
$ git cat-file -p 8650f5bc
Hello, SFN
Bye, SFN!

$ git cat-file -p 444a20ba
040000 tree 9cdf63e446fa97aa87ab12f40cf2450f6766a03d bak
100644 blob 34e5563c9ee0da4f494200508ca60eab841b0f36 scn.txt
100644 blob 8650f5bc38892e76d33091378dc6d90dfa2bc314 sfn.txt
```

```bash
$ find .git/objects -type f
.git/objects/34/e5563c9ee0da4f494200508ca60eab841b0f36 # scn.txt
.git/objects/9c/df63e446fa97aa87ab12f40cf2450f6766a03d # tree (처음 write-tree)
.git/objects/89/7db30ab07cb26c4e49194b3ba0395052be6e5b # commit (init)
.git/objects/17/67eb661906752863f2b3c2b178fb7d056c3b2b # sfn.txt
.git/objects/75/738d1e78288a343a7e0d1918bca98d9ae2e90c # commit (bak)
.git/objects/86/50f5bc38892e76d33091378dc6d90dfa2bc314 # tree (update 커밋의 스냅샷 트리)
.git/objects/44/4a20bab66fca19d8e43cd041d0bf16e0aacc97 # sfn.txt (두번째 버전)
.git/objects/6b/90661eb520ef2062ba18bd5da8fb24146678ae # tree (다음 write-tree)
.git/objects/f8/f3523473df20d30198ec5ce836af891ffebc34 # commit (update)
```

### 무엇을 알 수 있는가?

---

- `commit`오브젝트는 하나의 `tree`오브젝트를 가리킨다.
- `tree`오브젝트는 `blob`오브젝트와 다른 `tree`오브젝트를 가리킨다.
- 파일의 변경이 발생하면, 변경된 파일이 통째로 `blob`으로 저장된다.
- 변경된 파일은 모두 새로운 `blob`으로 저장된다.
- 만약 동일한 내용을 가진 파일이 있다면, `blob`의 해시도 같을 것이므로 하나만 존재하게 된다.

### 결국

---

- 커밋은 현재 인덱싱에 대한 스냅샷이다.
    - 정확히 말하면, 현재 인덱싱 `tree` 오브젝트가 스냅샷이다.
- 따라서, `commit-tree`를 이용해 커밋을 이리저리 옮겨도, 커밋 됐을 때의 상태는 항상 유지된다.
    - 어차피 커밋은 현재 인덱싱에 대한 스냅샷이기 때문이다.

- 참고
    - [https://www.craft.do/s/xX95kPcZRcv0Om](https://www.craft.do/s/xX95kPcZRcv0Om)
    - Git Internals: [https://git-scm.com/book/en/v2/Git-Internals-Git-Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
