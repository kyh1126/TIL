# 2장. 자바스크립트/Node.js의 문법

## 2.1 개발 환경 도입

---

- Node.js를 설치
    - 공식 사이트에서 설치 파일을 다운로드: 최신 LTS 버전을 이용
    - 25년 4월 기준, v22.14.0 (`nvm install —lts`, `npm install -g npm@latest`)

- 확인
    
    ```powershell
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 ~ % nvm install lts/*
    Downloading and installing node v22.14.0...
    Downloading https://nodejs.org/dist/v22.14.0/node-v22.14.0-darwin-x64.tar.xz...
    ############################################################################################################# 100.0%
    Computing checksum with shasum -a 256
    Checksums matched!
    Now using node v22.14.0 (npm v10.9.2)
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 ~ % node -v
    v22.14.0
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 ~ % npm -v
    11.3.0
    ```
    
- 실습
    - 작업용 디렉터리(nodescript)를 만들고 index.js 파일을 만든다.
        
        ```jsx
        console.log('hello Node.js');
        ```
        
    - 실행
        
        ```powershell
        kim-yoonhee@gim-yunhuiui-MacBookPro-2 nodescript % node index.js
        hello Node.js
        ```
        

- Node.js에서는 대화형으로 코드를 실행할 수 있는 REPL(Read-Eval-Print Loop)을 제공한다.
    - 입력한 계산식이나 함수 등의 결과가 다음 행에 출력되므로 간단한 동작이나 정규 표현을 확인하기에 편리하다.
    - `node` 명령어만 실행하면 REPL이 실행된다.
        
        ```powershell
        kim-yoonhee@gim-yunhuiui-MacBookPro-2 nodescript % node
        Welcome to Node.js v22.14.0.
        Type ".help" for more information.
        > 1+2
        3
        > console.log('xyz') //인수 표시 후, console.log 자신의 반환값인 undefined가 표시됨
        xyz
        undefined
        ```
        

### 2.1.1 Node.js의 버전

---

- Node.js는 Current(개발 버전)와 v18처럼 짝수 번호의 버전을 LTS(안정 버전)로 릴리스하는 형식을 갖는다.
    - 프로덕션 환경에서 운용할 때는, 기본적으로는 지원 기간이 가장 긴 최신 LTS 버전을 사용한다.
        - 지원 기간이 종료된 버전은 버그 및 취약성을 대응하지 않음
- 자바스크립트는 하위 호환성이 높다.
    - 다만, 메이저 버전을 올릴 때는 표준 모듈(Core API)을 이용하는 위치나 외부 모듈의 의존 관계 등에 주의하는 것이 좋다.

- Node.js 도입 방법
    - 공식 사이트에서 제공하는 빌드 완료된 바이너리를 이용하는 것을 권장한다.
        - 빌드 완료된 바이너리를 다운로드하고 PATH를 등록하면, node 본체와 패키지 관리자인 `npm`(`npx` 및 corepack)을 이용할 수 있다.
        - Node.js의 빌드 완료된 바이너리 목록: https://nodejs.org/dist/
    - Node.js의 버전을 관리할 수 있는 `nvm`이라는 소프트웨어도 있다.
        - 하지만, 설치 과정에서 오류가 발생하는 경우 공식으로 제공되는 바이너리를 사용하는 편이 간단하고 문제를 특정하기 쉽다.

## 2.2 자바스크립트 기초

---

### 2.2.1 변수

---

- 변수 선언 (이용 우선순위: `const` > `let` > `var`)
    - 범위: 변수의 값이나 식이 참조할 수 있는 범위
    - `var`: 변수를 선언, 초기화할 수 있다.선언한 범위를 넘어 참조할 수 있다.
        
        ```jsx
        if (true) {
          var foo = 5;
        }
        console.log(foo); // 5
        ```
        
    - `const`: 범위 안에서 유효하고 재할당 불가능한 변수를 선언할 수 있다.
        
        ```jsx
        if (true) {
          const foo = 5;
        }
        console.log(foo); // ReferenceError: bar is not defined
        ```
        
    - `let`: 범위 안에서 유효한 변수를 선언, 초기화할 수 있다.

- 함수 이름(식별자)은 문자, 언더스코어(`_`), 달러 기호(`$`)로만 시작할 수 있다.
    
    ```jsx
    const abc = 'abc'; // OK
    const _abc = '_abc'; // OK
    const abc123 = 'abc123'; // OK
    const 123 = '123'; // NG, 숫자는 맨 앞에 올 수 없다.
    ```
    

### 2.2.2 연산자

---

- 할당, 비교, 산술, 문자열 등의 연산자를 사용할 수 있다.
- 동등 비교
    - `===`(일치 연산자)는 `==`(동등 연산자)보다 엄격한 비교를 수행한다.
        
        ```jsx
        const a = 1;
        const b = '1';
        
        const equal = a == b; // true
        const equal2 = a === b; // false
        ```
        
    - 모호한 부등 연산자(`!=`), 엄격한 부등 연산자(`!==`)도 있다.

### 2.2.3 데이터 타입

---

- 자바스크립트는 동적 타입 언어이므로 명시적으로 타입을 선언하지 않는다.
    - 하지만 각각의 값을 연산하거나 비교하는 경우에는 종종 변수 내부에 들어있는 타입을 의식해야 할 때가 있다.

- 자바스크립트(ECMAScript): 7가지의 원시(primitive) 타입과, 이들을 복합적으로 다루는 `object` 타입
    - `String`: 문자열을 나타내는 타입. 큰따옴표(`"`), 작은따옴표(`'`), 백틱(```)으로 감싸 나타낸다.
        - 큰따옴표(`"`), 작은따옴표(`'`)는 모두 같은 결과를 반환한다.
            - 코드 안에서는 둘 중 하나로 통일하는 것이 좋다.
            - `\n`(줄 바꿈)과 같은 문자도 지원한다.
            
            ```powershell
            > console.log('첫 번째 행\n두 번째 행')
            첫 번째 행
            두 번째 행
            undefined
            ```
            
        - 백틱(```): ES6에서 추가된 템플릿 리터럴 표기법. 변수 전개나 여러 행에 걸친 문자열도 정의 가능.
            
            ```powershell
            > const one = '첫 번째';
            undefined
            > const two = '두 번째';
            undefined
            > console.log(`'${one}' '${two}'`)
            '첫 번째' '두 번째'
            undefined
            ```
            
    - `Number`: 정수나 부동 소수점과 같은 숫자를 다루는 타입
    - `BigInt`: 큰 자리를 다루는 정숫값
    - `Boolean`: `true` 또는 `false`를 가지는 불리언값
    - `Symbol`: 유일한 값이 되는 심벌 타입
    - `undefined`: 정의되지 않았음을 나타내는 타입
    - `null`: 데이터가 없는 것을 나타내는 타입
        - 타입스크립트를 도입하면 `null`이나 `undefined`를 컴파일 시점에 체크할 수 있기 때문에 런타임 에러를 미리 감지하기 쉽다.
    - `Object`: 객체, 배열, 정규 표현, 함수 등 원시 이외의 타입

- `typeof` 연산자: 데이터 타입을 확인. 반환하는 데이터 타입의 일부는 위와 다르다.
    
    ```powershell
    kim-yoonhee@gim-yunhuiui-MacBookPro-2 nodescript % node
    Welcome to Node.js v22.14.0.
    Type ".help" for more information.
    > typeof 'string'
    'string'
    > typeof []
    'object'
    > typeof console.log
    'function'
    > typeof null
    'object'
    ```
    

### 2.2.4 Object

---

- 원시타입 이외의 타입이 `Object`를 상속해 구현된다.
    - ex> 배열, `Date`, 정규 표현, 함수 등
- 초기화: 중괄호로 감싼 뒤 `속성_이름: 값`으로 표현한다.
    
    ```jsx
    const obj = {
        foo: {
            bar: 'baz',
        },
        now: new Date(),
        func: function () {
            console.log('function');
        }
    }
    ```
    
    - ES6부터는 더 유연하게 작성할 수 있다.
        - 속성 이름을 생략하고 작성하는 표기법을 자주 이용한다.
        - 속성 이름을 객체 바깥에서 정의한 값을 이용해 초기화할 수도 있다.
        
        ```jsx
        const key = 'value';
        const key2 = 'value2';
        
        const obj2 = {
            [key]: 'valuee',
            key,
            key2
        }
        
        console.log(obj2); // { value: 'valuee', key: 'value', key2: 'value2' }
        ```
        
- 접근: 마침표(`.`) 또는 대괄호로 접근할 수 있다.
    - 마침표는 속성 이름이 변수의 식별자와 같은 패턴에서만 사용할 수 있다.
    
    ```jsx
    const obj3 = {
        foo: 'hello',
        bar: {
            baz: 'world',
        }
    };
    
    console.log(obj3.foo); // hello
    console.log(obj3['foo']); // hello
    console.log(obj3.bar.baz); // world
    console.log(obj3['bar']['baz']); // world
    
    const obj4 = {
        123: '숫자',
        '': '빈 문자열'
    };
    
    console.log(obj4.123); // Uncaught SyntaxError: missing ) after argument list
    console.log(obj4[123]); // 숫자
    console.log(obj4.''); // Uncaught SyntaxError: Unexpected string
    console.log(obj4['']); // 빈 문자열
    ```
    
- `Object`의 속성은 선언 후에 바꿔쓸 수도 있다. `Object.freeze()`로 객체를 고정해 속성을 덮어쓰는 것을 막을 수 있다.
    
    ```jsx
    Object.freeze(obj2);
    obj2.key = 'sample2';
    > console.log(obj2)
    { value: 'valuee', key: 'sample', key2: 'value2' }
    undefined
    ```
    

- Object와 JSON
    - JSON: 문자열, 숫자, 배열, `boolean`, `null`, JSON Object만 가질 수 있다.
        
        ```jsx
        const obj = {
          foo: function() {console.log('foo')},
          bar: 'bar'
        };
        
        // foo 속성은 함수이므로 JSON으로 변환할 수 없기 때문에 사라진다.
        const str = JSON.stringify(obj); // '{"bar":"bar"}'
        ```
        
    - `Object`: 함수를 저장할 수 있다.

### 2.2.5 배열

---

- `[ ]`로 감싼 것. 구분자는 콤마(`,`). 인덱스는 0부터 시작한다.
    
    ```jsx
    const students = [
      { name: "Alice", age: 20 },
      { name: "Bob", age: 20 },
      { name: "Charlie", age: 30 },
    ];
    ```
    
- `.length`: 배열의 길이
- `Array.prototype.map`: 배열의 요소를 반복하면서 새로운 배열로 변환
    
    ```jsx
    const nameArray1 = students.map((student) => student.name);
    const nameArray2 = students.map(function (person) {
      return person.name;
    });
    
    console.log(nameArray1); // ['Alice', 'Bob', 'Charlie']
    console.log(nameArray2); // ['Alice', 'Bob', 'Charlie']
    ```
    
- `Array.prototype.filter`: 조건에 일치하는 요소만 추출
    
    ```jsx
    const under20_1 = students.filter((student) => student.age < 20);
    const under20_2 = students.filter(function (person) {
      return person.age < 20;
    });
    
    console.log(under20_1); // [{name: 'Alice', age: 20}]
    console.log(under20_2); // [{name: 'Alice', age: 20}]
    ```
    

### 2.2.6 함수

---

- `function`이라는 키워드로 정의한다.
- 함수의 인수에 객체를 전달할 때는 참조로 전달한다.
    
    ```jsx
    function setName(obj) {
      obj.name = "John";
    }
    
    const person = { name: "Alice" };
    setName(person);
    console.log(person); // {name: 'John'}
    ```
    
- 함수는 함수 표현식으로 별도의 변수에 할당할 수 있다. 이때 함수 이름을 생략할 수 있다.
    - 익명 함수는 콜백이나 즉시 실행 함수에도 사용할 수 있다.
    
    ```jsx
    const add = function (a, b) {
      return a + b;
    };
    
    // 콜백에 익명 함수
    setTimeout(function () {
      console.log("1s");
    }, 1000);
    // Timeout {
    //   _idleTimeout: 1000,
    //   _idlePrev: [TimersList],
    //   _idleNext: [TimersList],
    //   _idleStart: 1995069,
    //   _onTimeout: [Function (anonymous)],
    //   _timerArgs: undefined,
    //   _repeat: null,
    //   _destroyed: false,
    //   [Symbol(refed)]: true,
    //   [Symbol(kHasPrimitive)]: false,
    //   [Symbol(asyncId)]: 270,
    //   [Symbol(triggerId)]: 6,
    //   [Symbol(kAsyncContextFrame)]: undefined
    // }
    // 1s
    
    // 즉시 실행 함수, 곧바로 실행된다.
    (function () {
      console.log("executed");
    })();
    // executed
    ```
    
    - 함수 이름을 붙이면 에러의 원인이 되는 위치를 바로 특정할 수 있어 쉽게 디버깅할 수 있다.

- ES6 이후의 함수
    - 기본 인수: 함수의 인수를 생략했을 때 기본값을 전달할 수 있다.
        
        ```jsx
        function add(a, b = 2) {
          return a + b;
        }
        
        const total = add(1);
        console.log(total); // 3
        
        const total2 = add(1, 3);
        console.log(total2); // 4
        ```
        
    - 화살표 함수: 인수가 하나뿐인 `()`, 혹은 함수 내부 처리가 1행인 경우 `{}`와 `return`을 생략할 수 있어 간단하게 작성할 수 있다.
        
        ```jsx
        const add = (a, b) => {
          return a + b;
        };
        
        const double = a => a * 2;
        console.log(double(3)); // 6
        ```
        

❗함수 전체가 상당히 짧은 경우가 아니라면 되도록 `()`, `{}`, `return`을 생략하지 않고 작성한다.

- 개인 생각을 우선하는 것보다 ESLint가 제공하는 Linter, prettier와 같은 포매터의 기본 설정에 맞추는 경우가 많다.

## 2.3 자바스크립트와 상속

---

- 자바스크립트는 자바 등에서 볼 수 있는 `class`를 베이스로 한 언어는 아니다.
    - 대신 `prototype`이라는 구조로 상속을 구현한다.
        
        ```jsx
        const a = [1, 2, 3];
        
        Array.prototype.showLength = function () {
          console.log(this.length);
        };
        
        a.showLength(); // 3
        ```
        
    - 자바스크립트에서는 `Object`를 상속해서 `Array`와 같은 여러 파생 타입을 정의한다.

### 2.3.1 자바스크립트와 class

---

- ES6 이후의 자바스크립트에는 `class` 구문이 도입됐다.
    - 코드로 상속을 표현하고 싶은 경우에는 `prototype`보다 `class`로 표현하는 것을 권장한다.
    - 실제 운용하는 코드에서는 `prototype`을 사용하지 않는 편이 안전하다.
- 자바스크립트의 `class`는 `prototype`의 문법적 설탕(syntactic sugar)이다.
    - 자바스크립트는 원래 `prototype`으로 `class`와 같은 역할을 구현하고 있다.
        
        ```jsx
        function People(name) {
          this.name = name;
        }
        
        People.prototype.printName = function () {
          console.log(this.name);
        };
        
        const foo = new People("foo-name");
        foo.printName(); // foo-name
        ```
        

- 클래스를 어떻게 다루어야 하는가?
    - `class`와 같이 상태를 내포하는 설계는 피하는 편이 좋다.
        - 함수를 조합해서 기능을 구현하는 것을 선호한다.
    - 싱글 스레드/싱글 프로세스라는 Node.js 특성상 '객체의 상태에 따라 동작이 변한다 = 함수가 인수 이외의 사이드 이펙트를 갖는다'라는 설계는 치명적인 장애를 일으킬 수 있다.

## 2.4 자바스크립트와 this

---

- `this` 키워드: 인스턴스 자체에 대한 참조에 접근할 수 있다.
- 실행된 위치(콘텍스트)에 따라 값이 달라지는 속성이다.
    - 전역 콘텍스트에서 실행된 `this`는 Node.js에서는 `global` 객체를 가리킨다.
        
        ```powershell
        $ node
        > this === global
        true
        ```
        
    - 객체에서 실행된 `this`는 객체 자신이 된다.
        
        ```jsx
        function printName() {
          console.log(this.name);
        }
        
        const obj = { name: "obj-name", printName: printName };
        obj.printName(); // obj-name
        printName.call(obj); // obj-name
        obj.printName; // [Function: printName]
        ```
        
- 코드의 `this`를 원래의 대상으로 되돌리려면 `bind`를 사용해 `this`를 고정하거나 `this`를 별명으로 저장해두는 등의 기법을 사용할 수 있다.
    
    ```jsx
    function printName() {
      setTimeout(
        function () {
          console.log(this.name);
        }.bind(this), // 이 함수의 콘텍스트를 printName의 this로 고정한다.
        1000
      );
    }
    
    const obj = { name: "obj-name", printName: printName };
    obj.printName(); // obj-name
    ```
    
- 화살표 함수를 이용하면 `bind`와 같은 효과를 얻을 수 있다.
    - 실행 시의 콘텍스트를 고정하는 역할을 한다.
    
    ```jsx
    function printName() {
      setTimeout(() => {
        console.log(this.name);
      }, 1000);
    }
    
    const obj = { name: "obj-name", printName: printName };
    obj.printName(); // obj-name
    ```
    

❗`this`가 나타나는 위치의 함수는 되도록 화살표 함수를 사용하는 편이 좋다.

## 2.5 ES6 이후의 중요한 문법

---

### 2.5.1 전개 구문

---

- 전개(spread) 구문: 0개 이상의 인수와 배열, 객체 등을 전개하는 구문이다.
    - 여러 개의 배열이나 객체에서 새로운 참조를 가진 배열이나 객체를 생성할 때 편리하다.
- 변수 앞에 `...`를 붙여 객체를 전개할 수 있다.
    
    ```jsx
    const a = [1, 2, 3];
    const b = [4, 5, 6];
    const c = [...a, ...b]; // [1, 2, 3, 4, 5, 6]
    
    const obj1 = {
      a: "aaa",
      b: "bbb",
    };
    
    const obj2 = {
      c: "ccc"
    };
    
    const obj3 = { ...obj1, ...obj2 };
    console.log(obj3); // {a: 'aaa', b: 'bbb', c: 'ccc'}
    
    ```
    
    ❗배열, 객체 합성이라면 `Array.push`를 이용하거나 속성 추가 등의 방법을 사용할 수도 있다.
    
- 함수의 인수에도 이용할 수 있다.
    
    ```jsx
    const args = [1, 2, 3];
    function add(x, y, z) {
      return x + y + z;
    }
    
    const result = add(...args);
    console.log(result); // 6
    ```
    

- 장점: 새로운 참조를 가진 배열이나 객체를 생성할 수 있다.
    - 객체가 포함돼 있을 때는 주의해야 한다. 객체 참조가 들어있으므로 해당 객체의 값을 바꿔 쓰면 변경된다.
        - 전개 구문으로 만들어진 객체의 객체도 변경된다.

### 2.5.2 분할 대입

---

- 분할 대입: 배열, 객체 등에서 값을 모아서 꺼내는 구문이다.
    
    ```jsx
    const [first, second, ...foo] = [1, 2, 3, 4, 5];
    console.log(first); // 1
    console.log(second); // 2
    console.log(foo); // [3, 4, 5]
    
    const { a, b, ...bar } = { a: 1, b: 2, c: 3, d: 4 };
    console.log(a); // 1
    console.log(b); // 2
    console.log(bar); // {c: 3, d: 4}
    ```
    
- 함수의 반환값을 받을 때 편리하다.
    - 리액트 훅에서 자주 볼 수 있는 구문이다.
    
    ```jsx
    function returnArray() {
      return [1, 2, 3];
    }
    
    // 필요없는 값에는 변수명을 붙이지 않으면 건너뛸 수 있다.
    const [one, , three] = returnArray();
    console.log(one); // 1
    console.log(three); // 3
    ```
    
- 객체도 콜론(`:`)을 사용해 별칭을 붙여서 꺼낼 수 있다.
    
    ```jsx
    const obj = {
      a: 1,
      b: 2,
      c: 3,
    };
    
    const { a: foo, c: bar } = obj;
    console.log(foo); // 1
    console.log(bar); // 3
    ```
    

### 2.5.3 루프

---

- C 언어와 같은 문법의 `for`문이다.
    
    ```jsx
    const arr = ["foo", "bar", "baz"];
    
    for (let i = 0; i < arr.length; i++) {
      console.log(arr[i]);
    }
    ```
    
    - `Array` 객체에는 `forEach` 함수가 있다.
        
        ```jsx
        arr.forEach((item) => {
          console.log(item);
        });
        ```
        
- ES6 이후로는 `for ... of`를 이용하는 것이 좋다.
    
    ```jsx
    for (let i of arr) {
      console.log(i);
    }
    ```
    
    - `forEach`와 비슷하지만, 배열 이외의 객체(`iterator`)에 루프를 할 수 있고, 도중에 비동기 처리를 삽입할 수 있다는 강점이 있다.
    
    ❗객체의 키를 루프 처리하는 `for ... in`도 있다.
    
    - 임의의 순서로 배열을 반복하므로, 순서가 중요한 배열을 반복할 때는 사용해서는 안 된다.
    - 또한 프로토타입 속성까지 반복하므로 명확한 의도가 없는 한 사용하지 않는 것이 좋다.

- 엄격 모드: 일부 호환성을 해제하고, 더욱 엄격한 자바스크립트를 작성/동작하도록 하는 선언
    - `use strict`: 엄격 모드를 활성화한다.
    - ex> `var` 선언을 잊어버린 변수 정의는 전역 변수로 자동 정의된다.
        - 엄격 모드를 활성화하면 에러가 검출된다.
    - ECMAScript 모듈이나 `class` 내부에서는 자동으로 엄격 모드가 된다.