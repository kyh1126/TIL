# Go 1.18. Generic 프로그래밍

## 제네릭 함수

---

- 예제: [https://go.dev/play/](https://go.dev/play/) 실행
    
    ```go
    package main
    
    import "fmt"
    
    func print[T any](a T) {
    	fmt.Println(a)
    }
    
    func main() {
    	var a int = 10
    	print(a)
    	var b float32 = 3.14
    	print(b)
    	var c string = "Hello"
    	print(c)
    }
    ```
    
    ```powershell
    10
    3.14
    Hello
    ```
    

- 그러나 크다, 작다 연산 비교같은 경우는 특정 타입만 가능하다.
    
    ```go
    package main
    
    import "fmt"
    
    func min[T any](a, b T) T {
    	if a < b {
    		return a
    	}
    	return b
    }
    
    func main() {
    	var a int = 10
    	var b int = 20
    	fmt.Println(min(a, b))
    
    	var c int16 = 10
    	var d int16 = 20
    	fmt.Println(min(c, d))
    
    	var e float32 = 3.14
    	var f float32 = 3.98
    	fmt.Println(min(e, f))
    }
    ```
    
    ```powershell
    ./prog.go:8:5: invalid operation: a < b (type parameter T is not comparable with <)
    ```
    

### 타입 지정하기

---

- 예제: [https://go.dev/play/](https://go.dev/play/) 실행
    
    ```go
    package main
    
    import "fmt"
    
    func min[T int | int8 | int16 | float32 | float64 | int64](a, b T) T {
    	if a < b {
    		return a
    	}
    	return b
    }
    
    func main() {
    	var a int = 10
    	var b int = 20
    	fmt.Println(min(a, b))
    
    	var c int16 = 10
    	var d int16 = 20
    	fmt.Println(min(c, d))
    
    	var e float32 = 3.14
    	var f float32 = 3.98
    	fmt.Println(min(e, f))
    }
    ```
    
    ```powershell
    10
    10
    3.14
    ```
    
- 매번 타입을 다 넣어주기가 힘들다.

### 타입 제한자

---

- 예제: [https://go.dev/play/](https://go.dev/play/) 실행
    
    ```go
    package main
    
    import "fmt"
    
    type Integer interface {
    	int | int8 | int16 | int32 | int64
    }
    
    type Float interface {
    	float32 | float64
    }
    
    func min[T Integer | Float](a, b T) T {
    	if a < b {
    		return a
    	}
    	return b
    }
    
    func main() {
    	var a int = 10
    	var b int = 20
    	fmt.Println(min(a, b))
    
    	var c int16 = 10
    	var d int16 = 20
    	fmt.Println(min(c, d))
    
    	var e float32 = 3.14
    	var f float32 = 3.98
    	fmt.Println(min(e, f))
    }
    ```
    
    ```powershell
    10
    10
    3.14
    ```
    

- 타입 제한자는 인터페이스로는 쓸 수 없다. 제네릭 함수에서만 가능하다.
    - 기존 인터페이스 형식이 아니다.
- 타입 제한자에 메소드 제한까지 추가할 수 있다.

### `constraints` 패키지 타입 제한자

---

- 예제: [https://go.dev/play/](https://go.dev/play/) 실행
    
    ```go
    package main
    
    import (
    	"fmt"
    
    	"golang.org/x/exp/constraints"
    )
    
    func min[T constraints.Ordered](a, b T) T {
    	if a < b {
    		return a
    	}
    	return b
    }
    
    func main() {
    	var a int = 10
    	var b int = 20
    	fmt.Println(min(a, b))
    
    	var c int16 = 10
    	var d int16 = 20
    	fmt.Println(min(c, d))
    
    	var e float32 = 3.14
    	var f float32 = 3.98
    	fmt.Println(min(e, f))
    
    	var g string = "Hello"
    	var h string = "World"
    	fmt.Println(min(g, h))
    }
    ```
    
    ```powershell
    10
    10
    3.14
    Hello
    ```
    

### `comparable` 타입

---

- `!=`, `==`로 비교할 수 있는 타입을 지정한다.

## 제네릭 타입

---

```go
type Node[T any] struct {
	val T
	next *Node[T]
}
```

- 기존 `struct` 선언엔 `[T any]` 부분이 없었다.
- 런타임에 동작하진 않고, 빌드할 때 컴파일러가 타입을 만들어준다.

## 틸트(`~`)

---

- 이 타입을 베이스로 하는 별칭 타입까지 포함한다는 의미

- 예제: [https://go.dev/play/](https://go.dev/play/) 실행
    
    ```go
    package main
    
    import (
    	"fmt"
    )
    
    type Integer interface {
    	int | int8 | int16 | int32 | int64
    }
    
    type MyInt int
    
    func min[T Integer](a, b T) T {
    	if a < b {
    		return a
    	}
    	return b
    }
    
    func main() {
    	var a int = 10
    	var b int = 20
    	fmt.Println(min(a, b))
    
    	var c int16 = 10
    	var d int16 = 20
    	fmt.Println(min(c, d))
    
    	var g MyInt = 10
    	var h MyInt = 20
    	fmt.Println(min(g, h))
    }
    ```
    
    ```powershell
    ./prog.go:33:17: MyInt does not satisfy Integer (possibly missing ~ for int in Integer)
    ```
    

👉

```go
package main

import (
	"fmt"
)

type Integer interface {
	~int | int8 | int16 | int32 | int64
}

type MyInt int

func min[T Integer](a, b T) T {
	if a < b {
		return a
	}
	return b
}

func main() {
	var a int = 10
	var b int = 20
	fmt.Println(min(a, b))

	var c int16 = 10
	var d int16 = 20
	fmt.Println(min(c, d))

	var g MyInt = 10
	var h MyInt = 20
	fmt.Println(min(g, h))
}
```

```powershell
10
10
10
```