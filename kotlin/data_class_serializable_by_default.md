# data class serializable by default?

- 디폴로 serializable interface 를 구현하고 있진 않다.

```kotlin
import java.io.Serializable

data class Foo(val bar: String)

fun acceptsSerializable(s: Serializable) { }

fun main(args: Array<String>) {
    val f: Foo = Foo("baz")
    acceptsSerializable(f)  // Will not compile
}
```

- 참고
    - [https://stackoverflow.com/questions/61241155/is-kotlin-data-class-serializable-by-default](https://stackoverflow.com/questions/61241155/is-kotlin-data-class-serializable-by-default)


- [Notion link](https://jennyuni.notion.site/data-class-serializable-by-default-164113969cd546808c33844be1e31ea2)
