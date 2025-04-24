# 3장. Node.js와 모듈

- Node.js의 2가지 모듈 분할 방식
    - CommonJS 모듈
    - ECMAScript 모듈
    
    ```jsx
    // CommonJS modules
    const fs = require('fs');
    
    // ECMAScript modules
    import fs from 'fs';
    ```
    

## 3.1 CommonJS 모듈

---

### 3.1.1 exports와 require

---

- `exports`: 파일 단위로 자동 생성되는 변수에 함수나 변수를 할당해서 외부로 공개할 수 있다.
    
    ```jsx
    exports.num = 1;
    exports.add = (a, b) => {
      return a + b;
    };
    exports.sub = (a, b) => {
      return a - b;
    };
    ```
    
- `require`: 모듈을 다른 파일에서 불러온다.
    
    ```jsx
    // path/to/folder/
    // I----- index.js
    // 1----- calc.js
    const calc = require('./calc');
    
    console.log(calc.num); // 1
    
    let res = calc.add(3, 1);
    console.log(res); // 4
    
    res = calc.sub(3, 1);
    console.log(res); // 2
    ```
    
    - calc라는 변수 이름으로 calc.js로 분할한 모듈을 불러온다.

- `require`와 생략
    - `require('./path')`는 `path.js` 또는 `path/index.js`를 자동으로 찾음
    - `.js`를 생략 가능하지만 명시하는 것이 더 명확하다.
    
    ```jsx
    const hoge = require('calc') // 문법적으로는 문제없음
    ```
    

- `module.exports`
    - `exports.xxx` 이외에도 `module.exports`라는 변수를 이용할 수 있다.
        
        ```jsx
        // module.exports 방식
        module.exports = {
          add: (a, b) => {
            return a + b;
          },
          sub: (a, b) => {
            return a - b;
          }
        };
        ```
        
    - `exports.xxx`와 `module.exports`는 함께 사용 불가. 모두 사용할 경우 `module.exports` 우선
        - 코드를 작성할 때는 둘 중 하나로 통일하도록 한다.

### 3.1.2 모듈 읽기와 싱글턴

---

- 모듈은 싱글턴으로 읽힌다.
    
    ```jsx
    // path/to/folder/
    // I----- index.js
    // I----- a.js
    // I----- b.js
    // 1----- calc.js
    
    // a.js
    const calc = require('./calc');
    calc.num = 5;
    
    // b.js
    const calc = require('./calc');
    setTimeout(() => { calc.num = 10 }, 1000);
    
    // index.js
    require('./a');
    require('./b');
    setTimeout(() => console.log(calc.num), 1500); // 10 출력
    ```
    
    → Node.js는 싱글 스레드로 동작하기 때문에 로직 도중에 요청이 뒤섞일 가능성이 있다.
    

- `require`와 분할 대입
    - `require`로 받은 대상은 단순한 변수이므로 분할 대입을 이용해 필요한 것만 불러올 수 있다.
    
    ```jsx
    const { add } = require('./calc');
    
    const res = add(3, 1);
    console.log(res); // 4
    ```
    

## 3.2  ECMAScript 모듈

---

- ECMAScript 모듈(ESM): 자바스크립트의 표준 모듈 방식

### 3.2.1 모듈 분할 방식의 차이와 주의점

---

- CommonJS 모듈: `.js`파일이 많다.
- ECMAScript 모듈: `.mjs` 확장자 사용 (or `package.json`에 `"type": "module"` 설정)

### 3.2.2 export와 import

---

- `import`: 모듈을 불러올 때 사용한다.
    
    ```jsx
    // calc.mjs
    export const num = 1;
    export const add = (a, b) => a + b;
    export const sub = (a, b) => a - b;
    
    // index.mjs
    import { num, add, sub } from './calc.mjs';
    ```
    
    - `import`는 확장자까지 지정해야 한다.
- `as`: 여러 모듈을 불러온다. 모듈에 이름을 붙여 새로운 객체로 불러올 수 있다.
    
    ```jsx
    import * as calc from './calc.mjs';
    ```
    

- default export
    - CommonJS 모듈의 `module.exports`와 비슷하다.
        - 하나의 기본값을 내보낼 수 있다.
        
        ```jsx
        export const num = 1;
        
        export const add = (a, b) => {
          return a + b;
        };
        
        export const sub = (a, b) => {
          return a - b;
        };
        
        export default function () {
          console.log('calc');
        }
        ```
        
        - CommonJS 모듈과 달리 `export`와 `export default`는 함께 사용할 수 있다.
            - 그러나 일반적인 `export`로 통일하는 편이 작성, 읽기에 쉽다.
    - default export된 함수는 호출하는 측에서 이름을 자유롭게 붙일 수 있다.
        
        ```jsx
        import defaultCalc, { add, sub } from './calc.mjs';
        
        defaultCalc(); // calc
        ```
        

### 3.2.3 동적으로 모듈 불러오기

---

- 동적으로 모듈 불러오기
    - ECMAScript 모듈의 특징
    - `import`에 모듈의 경로를 지정하면 `import`를 호출한 시점에 처음으로 모듈을 불러올 수 있다.
        
        ```jsx
        import('./calc.mjs')
          .then((module) => {
            console.log(module.add(l, 2))
          })
        ```
        
    - 주로 프론트엔드에서 클릭 등 이벤트 발생 시점에 모듈을 최초 다운로드하여 초기 렌더링 비용을 줄인다.

❗Node.js 코드에서는 동적으로 불러오기를 사용하는 빈도가 그렇게 높지 않다.

## 3.3 모듈 사용 구분

---

- Node.js 모듈은 CommonJS와 ECMAScript 모듈 두 가지 선택 가능
    - `.js`, `.cjs`는 CommonJS, `.mjs`는 ECMAScript 모듈
- `package.json`의 `type` 속성으로 모듈 형식을 고정할 수도 있다.
    
    ```jsx
    {
      "type": "module" // package.json이 있는 디렉터리 아래의 .js 파일을 ECMAScript 모듈로 해석하게 한다.
    }
    ```
    
- CommonJS 모듈과 ECMAScript 모듈의 형식은 함께 사용할 수 있지만, 한 애플리케이션 내부에서는 하나의 형식만 이용하는 것이 좋다.
    - ECMAScript 모듈만으로 코드 작성은 간단한 애플리케이션을 제외하고는 아직 비용이 다소 높다. (CommonJS 모듈을 사용한 프로젝트의 수를 넘어서려면 긴 시간이 걸릴 것)

### 3.3.1 애플리케이션 개발

---

- 현재로서는 애플리케이션 개발 시 CommonJS가 개발 중 막히는 일이 적어 안정적이다.

### 3.3.2 라이브러리 개발

---

- 라이브러리 개발 시에는 CommonJS, ECMAScript 모듈 모두에 대응하는 방법을 제공하면 좋다.
    - `npm`에서 관리하는 패키지나 표준 모듈은 패키지 이름만으로도 불러올 수 있다.
        
        ```jsx
        const foo = require('foo'); // CommonJS 모듈 파일에서 foo 패키지를 불러온다.
        
        import foo from 'foo'; // ECMAScript 모듈 파일에서 foo 패키지를 불러온다.
        ```
        
- 듀얼 패키지: CommonJS와 ECMAScript 모듈이 사용할 수 있는 라이브러리(모듈)를 설정한다.
    
    ```json
    {
      "exports": {
        "import": "./index.mjs",
        "require": "./index.cjs"
      }
    }
    ```
    
    - 이 기능을 이용해서 ECMAScript 모듈을 래퍼로 사용해 양쪽을 대응하는 접근 방식도 있다.
        
        ```json
        {
          "exports": {
            "import": "./wrapper.mjs",
            "require": "./index.cjs"
          }
        }
        ```
        

- 듀얼 패키지에서의 과제
    - 이상적인 대응이라고 생각할 수 있지만 다른 파일이 호출되는 문제점도 발생한다.
    - 2개 파일로 나누게 되면 각각을 호출했을 때 동작이 다를 수도 있다.
    - 이용자의 실행 환경을 한정할 수 있는 모듈이라면 `package.json`의 `type`을 지정해 CommonJS 또는 ECMAScript 모듈만으로 한정하는 방법을 검토하면 좋다.

❗ Node.js v12 이하 버전이 LTS에서 제외되어 ECMAScript 모듈로만 제공되는 모듈도 늘어나고 있다.

## 3.4 표준 모듈

---

- Node.js: V8 엔진과 여러 자체 구현을 조합한 실행 환경
- 표준 모듈(Core API): 표준 자바스크립트에는 없는 여러 구현
    - `fs`, `path`, `http/https`, `os`, `crypto`, `assert` 등
    - 상대 경로 지정 없이 불러올 수 있다.
        
        ```jsx
        require('fs');
        ```
        
- 최신 Node.js는 명시적 `node:` 접두사를 붙이는 방식도 지원한다.
    - Node.js의 표준 모듈인 것을 나타내는 접두사
        
        ```jsx
        require('node:fs')
        ```
        
    - 외부 모듈과 표준 모듈에서 이름 공간이 충돌하는 것을 방지하기 위해 도입됐다.

- (CommonJS) 모듈 안의 특수한 함수
    - `__filename`: 현재 파일 이름
    - `__dirname`: 현재 디렉터리 이름

## 3.5 npm과 외부 모듈 불러오기

---

- Node.js에서는 `npm`을 통해 외부 공개 모듈을 쉽게 활용할 수 있음 (https://npmjs.com)

### 3.5.1 package.json/packaage-lock.json

---

- npm 모듈을 이용, 관리하기 위해서는 `package.json` 파일이 필요하다.
    
    ```powershell
    $ mkdir test_npm
    $ cd test_npm
    # package.json 작성
    $ echo '( "private": true }' >> package.json
    $ cat package.json
    { "private": true }
    # undici 설치
    $ npm install undici —save
    ```
    
    - `private: true`: 이것은 프라이빗한 애플리케이션이다
        - 실수로 로컬의 애플리케이션을 `npm` 모듈로 공개하는 것을 방지
        - 원래 필요한 속성을 생략할 수 있다.
- 설치하면 `package.json`이 자동으로 업데이트되고, `dependencies`에 의존성 버전이 추가된다.
    
    ```json
    {
      "private": true,
      "dependencies": {
        "undici": "*.14.0"
      }
    )
    ```
    

- `package.json`: 프로젝트 의존성 및 정보 관리 파일
    - 애플리케이션의 의존 관계를 관리할 수 있다.
    - 스크립트의 바로가기 등록, `npm`에 애플리케이션 공개 시 정보를 기입 등 다양한 기능을 제공한다.

- 설치한 `npm` 모듈의 실체는 `node_modules` 디렉터리에 저장된다.
    
    ```powershell
    directory/
    I----- node_modules/
    I----- package.json
    1----- package-lock.json
    ```
    
    - `package-lock.json`: 설치한 의존성의 버전을 고정하고, `node_modules` 재구축을 도와준다.
    - 디렉터리는 설치 환경에 따라 달라진다 → `node_modules` 디렉터리는 깃에 커밋하면 안 된다.

### 3.5.2 npm scripts

---

- `package.json`은 `npm scripts`라는 간단한 태스크 러너 기능을 제공한다.
- `package.json`의 `script` 속성 안에 정의된 태스크는 `npm run xxx` 명령어로 실행 가능하다.
    
    ```json
    {
      "private": true,
      "scripts": {
        "prebuild": "echo 'pre build'",
        "build": "echo 'build'",
        "postbuild": "echo 'post build'"
      }
    }
    ```
    
    - `pre`를 붙인 스크립트는 태스크 직전, `post`를 붙인 스크립트는 태스크 직후에 자동으로 실행된다.
        
        ```powershell
        $ npm run build
        
        > prebuild
        > echo 'pre build'
        
        pre build
        
        > build
        > echo 'build'
        
        build
        
        > postbuild
        > echo 'post build'
        
        post build
        ```
        

### 3.5.3 시맨틱 버저닝

---

- `npm` 패키지는 시맨틱 버저닝(semantic versioning, semver)에 따라 버전을 관리한다.
    - `Major.Minor.Patch` 업데이트 규칙
        
        
        | 이름 | 규칙 |
        | --- | --- |
        | Major | 버그/기능 추가에 상관없이 하위 호환성을 보장하지 않는 릴리스를 할 때 증가되며 Minor와 Patch를 0으로 설정한다. |
        | Minor | 하위 호환성이 있는 기능을 추가해 릴리스를 할 때 증가되며 Patch를 0으로 설정한다. |
        | Patch | 하위 호환성이 있는 버그를 수정해 릴리스를 할 때 증가된다. |
- `npm` 패키지 업데이트의 동작도 시맨틱 버전을 기준으로 동작한다.

### 3.5.4 모듈 이용

---

- 표준 모듈과 마찬가지로 `npm`에서 설치한 모듈도 상대 경로를 지정하지 않고 불러올 수 있다.
    
    ```jsx
    const { request } = require('undici');
    
    request('https://www.google.com')
      .then((res) => res.body.text())
      .then((body) => {
        console.log(body);
      });
    ```
    
    - 설치된 모듈의 실체는 `node_modules` 디렉터리에 존재한다.
    - 만약 `node_modules` 디렉터리를 삭제하면 모듈을 찾지 못하는 에러가 발생하므로, 다시 사용하려면 `npm install`을 실행해야 한다.
        
        ```powershell
        $ rm -rf node_modules
        $ node index.js
        node:internal/modules/cjs/loader:936
          throw err;
          ^
        
        Error: Cannot find module 'undici'
        ```
        

- yarn, pnpm 그리고 Node.js
    - `npm` 외에도 `yarn`, `pnpm` 등 다른 패키지 관리자의 이용도 늘어나고 있다.
        - Node.js 자체적으로 `yarn`, `pnpm` 지원 실험적 동작도 있었다.
    - 어떤 것을 이용해도 큰 차이는 없으나, 한 프로젝트에서는 하나의 패키지 관리자를 일관되게 사용하는 것이 좋다.