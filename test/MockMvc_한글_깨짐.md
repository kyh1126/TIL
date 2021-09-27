# MockMvc 한글 깨짐

## 문제

```kotlin
@Test
@DisplayName("기본상품 카테고리 조회 api 정상적으로 조회된다")
@Transactional
fun givenBasicProductCategoryId_WhenGetBasicProductCategories_ThenReturn200_Success() {
    // given
    val level1CategoryId = BasicProductCategories.keys.first()

    // 기본 상품 카테고리 생성
    codeRepository.saveAll(BasicProductCategoryCodes)
    basicProductCategoryRepository.saveAll(buildBasicProductCategory())

    // when & then
    mockMvc.get("$basicProductControllerPath/categories") {
        param("level1CategoryId", level1CategoryId.toString())
        contentType = MediaType.APPLICATION_JSON
        accept = MediaType.APPLICATION_JSON
    }.andExpect {
        status { isOk() }
        jsonPath("$.payload[0].value") { value(level1CategoryId) }
        jsonPath("$.payload[0].label") { value("농산") }
    }
}
```

- 테스트 코드에서 깨진 한글이 Controller 에 유입될 수 있으며, 결국 원하는 대로 동작하지 않게 된다.


## 원인

- `MediaType.APPLICATION_JSON_UTF8`가 Deprecated 되어 `MediaType.APPLICATION_JSON`를 사용하도록 변경됨

→ Response header 의 content-type 에 `charset=UTF-8`가 제거되어 한글이 깨진다.


## 해결

```kotlin
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Import
import org.springframework.web.filter.CharacterEncodingFilter

/**
 * UTF-8 CharacterEncodingFilter bean 별도 추가
 * <p>
 * @see @AutoConfigureMockMvc 는 SpringBootMockMvcBuilderCustomizer 에서 bean 등록된 filter 들을 mockMvc filter 에 설정해준다.
 */
@Target(AnnotationTarget.ANNOTATION_CLASS, AnnotationTarget.CLASS)
@Retention
@MustBeDocumented
@AutoConfigureMockMvc
@Import(UTF8AutoConfigureMockMvc.Config::class)
annotation class UTF8AutoConfigureMockMvc {
    class Config {
        @Bean
        fun characterEncodingFilter(): CharacterEncodingFilter {
            return CharacterEncodingFilter("UTF-8", true)
        }
    }
}
```

- 문제를 해결하기 위해 MockMvc build 시 `CharacterEncodingFilter`를 추가해주면 된다.
- MockMvc 설정을 위해 붙이는 `@AutoConfigureMockMvc`는 `SpringBootMockMvcBuilderCustomizer`를 사용하는데, 여기서 `addFilters`메소드는 bean 으로 등록된 filter 들을 가져와 mockMvc filter에 설정해준다.

- 참고
    - [https://pompitzz.github.io/blog/Spring/MockMvc_Encoding.html](https://pompitzz.github.io/blog/Spring/MockMvc_Encoding.html)
    - [https://github.com/HomoEfficio/dev-tips/blob/master/Spring Test MockMvc의 한글 깨짐 처리.md](https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Test%20MockMvc%EC%9D%98%20%ED%95%9C%EA%B8%80%20%EA%B9%A8%EC%A7%90%20%EC%B2%98%EB%A6%AC.md)


- [Notion link](https://jennyuni.notion.site/MockMvc-c2f00233b68c4b13bf09dd4c0c1e3bf0)
