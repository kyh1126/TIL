# Jackson @JsonUnwrapped

- Jackson 모듈 with Kotlin 레퍼런스: [https://github.com/FasterXML/jackson-module-kotlin/](https://github.com/FasterXML/jackson-module-kotlin/)

## 첫 번째 문제

```kotlin
data class InvoiceDetailCreateModel(
    @NotNull
    @ApiModelProperty(value = "명세서 스냅샷")
    var snapshot: InvoiceSnapshotDto?
) {
    @NotNull
    @field:Valid
    @JsonUnwrapped
    lateinit var invoiceModel: InvoiceCreateModel
    //...
}
```

- 위의 `@JsonUnwrapped`대상인 클래스 InvoiceCreateModel
    
    ```kotlin
    class InvoiceCreateModel(
        @NotNull
        @CustomerId
        @ApiModelProperty(value = "고객사 ID")
        val customerId: Long,
    
        @NotNull
        @ApiModelProperty(value = "계약 ID")
        val contractId: Long,
        //...
    )
    // TODO: 다른 프로젝트 fn-product 에선 잘 작동하였으니, 추후 비교 필요
    ```
    

의 **Validation 이 작동하지 않았다.**


## 해결

- 클래스 생성자로 필드 선언이 아닌, 본문으로 써주니 잘 작동하였다.
    
    ```kotlin
    class InvoiceCreateModel {
        @NotNull
        @CustomerId
        @ApiModelProperty(value = "고객사 ID")
        val customerId: Long = 0
    
        @NotNull
        @ApiModelProperty(value = "계약 ID")
        val contractId: Long = 0
        //...
    }
    ```
    
- 참고
    - [https://velog.io/@lsb156/SpringBoot-Kotlin에서-Valid가-동작하지-않는-원인JSR-303-JSR-380](https://velog.io/@lsb156/SpringBoot-Kotlin%EC%97%90%EC%84%9C-Valid%EA%B0%80-%EB%8F%99%EC%9E%91%ED%95%98%EC%A7%80-%EC%95%8A%EB%8A%94-%EC%9B%90%EC%9D%B8JSR-303-JSR-380)
    - [https://velog.io/@freddiey/Spring-validationwith-kotlin](https://velog.io/@freddiey/Spring-validationwith-kotlin)
    - [https://blog.gangnamunni.com/post/kotlin-annotation/](https://blog.gangnamunni.com/post/kotlin-annotation/)

- cf> 코틀린은 [Annotation use-site target](https://kotlinlang.org/docs/reference/annotations.html#annotation-use-site-targets)을 지정하지 않은 경우 다음 순서로 `@Annotation`의 `@Target`의 가능여부를 조사하여 가능한 첫번째 `@Target`을 선택한다.
    1. param - (a constructor parameter)
    2. property - (the Kotlin's property, it is not accessible from Java code)
    3. field - (field)


## 두 번째 문제

```kotlin
data class InvoiceDetailCreateModel(
    @NotNull
    @ApiModelProperty(value = "명세서 스냅샷")
    var snapshot: InvoiceSnapshotDto?,

    @NotNull
    @field:Valid
    @field:JsonUnwrapped
    val invoiceModel: InvoiceCreateModel
)
```

- 이런식으로 생성자 파라미터로 `@JsonUnwrapped`대상인 클래스 InvoiceCreateModel 를 써주면 컴파일은 되지만, **런타임 시** `InvalidDefinitionException` 발생한다.
    
    ```kotlin
    com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot define Creator property "invoiceModel" as `@JsonUnwrapped`: combination not yet supported
    ```
    

## 해결

- Databind 는 생성자나 static creator 메서드에서 사용되는 `@JsonUnwrapped`를 지원하지 않는다.
    - 들어오는 매개 변수와 사용 가능한 creator 매개 변수 간에 1대 1 관계가 있을 것으로 예상한다.
- 하나의 속성을 `lateinit` 속성으로 클래스 본문으로 이동하여 해결하였다.
    - `첫 번째 문제`에서 제시되는 클래스의 구조는 이 문제로 인하여 나오게 되었다.
    
    ```kotlin
    data class InvoiceDetailCreateModel(
        @NotNull
        @ApiModelProperty(value = "명세서 스냅샷")
        var snapshot: InvoiceSnapshotDto?
    ) {
        @NotNull
        @field:Valid
        @JsonUnwrapped
        lateinit var invoiceModel: InvoiceCreateModel
        //...
    }
    ```
    

- 참고
    - [https://github.com/FasterXML/jackson-module-kotlin/issues/56](https://github.com/FasterXML/jackson-module-kotlin/issues/56)
