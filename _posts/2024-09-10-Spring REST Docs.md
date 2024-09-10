---

layout: post
title: Spring REST Docs
date: 2024-09-10 00:00:00 +0800
categories: [CS Study, testcode]
tags: [Sophomore, Testcode]
pin: true

---


## Spring REST Docs

<br>

### 1. gradle 파일에 Asciidoctor 관련 설정 넣어주기

- 플러그인 넣어주기

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.7'
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'
    id "org.asciidoctor.jvm.convert" version "4.0.3" // asciidoctor 플러그인
}
``` 

<br>

- configurations 넣어주기

```gradle
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    asciidoctorExt // 확장 설정(asciidoctor 관련 작업에 필요한 의존성을 정의 및 확장)
}
``` 

<br>

- 의존성 넣어주기

```gradle
dependencies {
    // RestDocs
    asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'

    // MockMvc와 통합하여 api 테스트를 수행하고 그 결과로부터 REST문서를 생성
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}
``` 

<br>

- API 문서 자동 생성 관련 설정

```gradle
ext { // 전역 변수 -> 해당 경로의 디텍토리로 Docs 테스트가 생성하는 스니펫이 저장된다.
    snippetsDir = file('build/generated-snippets')
}

test {
    // 테스트 실행 후 그 결과를 해당 디텍토리에 저장하겠다는 뜻
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExt'

    sources { // 특정 파일만 html로 만든다.
        include("**/index.adoc")
    }
    baseDirFollowsSourceFile() // 다른 adoc 파일을 include 할 때 경로를 baseDir로 맞춘다.
    dependsOn test // test가 완료된 후 실행되도록 한다.
}

// asciidoctor로 생성된 문서를 jar 파일 내 static/docs 디렉토리에 넣음.
// application에서 /docs 경로로 문서에 접근할 수 있게 함.
bootJar {
    dependsOn asciidoctor
    from("${asciidoctor.outputDir}") {
        into 'static/docs'
    }
}
``` 

<br>


### 2. 인텔리제이 플러그인 설치

<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/asciidoc_plugin.png?raw=true">  

<br>

### 3. RestDocsSupport 클래스 만들기

```java
@ExtendWith(RestDocumentationExtension.class) // RESTful api 문서화 지원

// @SpringBootTest // 스프링 의존성 필요 없음

// REST Docs 기능을 테스트 클래스에 통합하는 공통 설정
public abstract class RestDocsSupport {

    protected MockMvc mockMvc;
    protected ObjectMapper objectMapper = new ObjectMapper();

// 이렇게 하면 application context를 띄움.
//    @BeforeEach
//    void setup(WebApplicationContext webApplicationContext, RestDocumentationContextProvider Provider) {
//        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
//                .apply(documentationConfiguration(Provider))
//                .build();
//    }

    // 문서화할 때는 application context를 띄우지 않아도 됨.
    @BeforeEach
    void setUp(RestDocumentationContextProvider provider) {
        this.mockMvc = MockMvcBuilders.standaloneSetup(initController())
            .apply(documentationConfiguration(provider))
            .build();
    }

    protected abstract Object initController(); // 테스트하고 싶은 컨트롤러에 extends 해주면 됨.

}
```

### 4. ProductController 테스트 해보기

- initController에 ProductController 넣어주기  

```java
public class ProductControllerDocsTest extends RestDocsSupport {

    // mock 만들기
    private final ProductService productService = mock(ProductService.class);

    // initController를 오버라이딩함으로써 ProductController에 대한 테스트임을 명시
    @Override
    protected Object initController() {
        return new ProductController(productService); // 컨트롤러에 의존성을 주입
    }

}
```

<br>

- 문서화 설정

ApiResponse와 Product의 형식이 아래와 같이 설정되어있다고 가정함. 
 
```java
@Getter
public class ApiResponse<T> {

    private int code;
    private HttpStatus status;
    private String message;
    private T data;

    public ApiResponse(HttpStatus status, String message, T data) {
        this.code = status.value();
        this.status = status;
        this.message = message;
        this.data = data;
    }

    public static <T> ApiResponse<T> of(HttpStatus httpStatus, String message, T data) {
        return new ApiResponse<>(httpStatus, message, data);
    }

    public static <T> ApiResponse<T> of(HttpStatus httpStatus, T data) {
        return of(httpStatus, httpStatus.name(), data);
    }

    public static <T> ApiResponse<T> ok(T data) {
        return of(HttpStatus.OK, data);
    }

}

```

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productNumber;

    @Enumerated(EnumType.STRING)
    private ProductType type;

    @Enumerated(EnumType.STRING)
    private ProductSellingStatus sellingStatus;

    private String name;

    private int price;

    @Builder
    private Product(String productNumber, ProductType type, ProductSellingStatus sellingStatus, String name, int price) {
        this.productNumber = productNumber;
        this.type = type;
        this.sellingStatus = sellingStatus;
        this.name = name;
        this.price = price;
    }

}
```

<br>

위의 response 형식과 Product 형식에 맞춰서 문서화를 해준다.
```java
    @DisplayName("신규 상품을 등록하는 API")
    @Test
    void createProduct() throws Exception {
        ProductCreateRequest request = ProductCreateRequest.builder()
            .type(ProductType.HANDMADE)
            .sellingStatus(ProductSellingStatus.SELLING)
            .name("아메리카노")
            .price(4000)
            .build();

        given(productService.createProduct(any(ProductCreateServiceRequest.class)))
            .willReturn(ProductResponse.builder()
                .id(1L)
                .productNumber("001")
                .type(ProductType.HANDMADE)
                .sellingStatus(ProductSellingStatus.SELLING)
                .name("아메리카노")
                .price(4000)
                .build()
            );

        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isOk())
            // 여기까지는 복사 & 붙여넣기
            
            // REST Docs 문서화 설정 시작
            .andDo(document("product-create", 

                // 요청과 응답을 보기 좋게 정리
                preprocessRequest(prettyPrint()), 
                preprocessResponse(prettyPrint()), 

                // 요청 필드에 대한 문서화
                requestFields( 
                    fieldWithPath("type").type(JsonFieldType.STRING)
                        .description("상품 타입"),
                    fieldWithPath("sellingStatus").type(JsonFieldType.STRING)
                        .optional()
                        .description("상품 판매상태"),
                    fieldWithPath("name").type(JsonFieldType.STRING)
                        .description("상품 이름"),
                    fieldWithPath("price").type(JsonFieldType.NUMBER)
                        .description("상품 가격")
                ),

                // 응답 필드에 대한 문서화
                responseFields( 
                    fieldWithPath("code").type(JsonFieldType.NUMBER)
                        .description("코드"),
                    fieldWithPath("status").type(JsonFieldType.STRING)
                        .description("상태"),
                    fieldWithPath("message").type(JsonFieldType.STRING)
                        .description("메시지"),
                    fieldWithPath("data").type(JsonFieldType.OBJECT)
                        .description("응답 데이터"),
                    fieldWithPath("data.id").type(JsonFieldType.NUMBER)
                        .description("상품 ID"),
                    fieldWithPath("data.productNumber").type(JsonFieldType.STRING)
                        .description("상품 번호"),
                    fieldWithPath("data.type").type(JsonFieldType.STRING)
                        .description("상품 타입"),
                    fieldWithPath("data.sellingStatus").type(JsonFieldType.STRING)
                        .description("상품 판매상태"),
                    fieldWithPath("data.name").type(JsonFieldType.STRING)
                        .description("상품 이름"),
                    fieldWithPath("data.price").type(JsonFieldType.NUMBER)
                        .description("상품 가격")
                )
            ));
    }
```

<br>

### 4. Asciidoctor 실행하기

인텔리제이 우측에 gradle에서 Task 하위의 documentation에서 asciidoctor를 실행시킨다.
<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/asciidoctor.png?raw=true">  

<br>

그러면 build 하위에 generated-snippets라는 파일이 생기고, 그 안에 문서화에 필요한 코드 조각들이 생성된다.
<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/generated-snippets.png?raw=true">  

<br>

이제 이 코드 조각들을 이용해서 문서를 작성해보자. 

- product.adoc 코드
```adoc
[[product-create]]
=== 신규 상품 등록

==== HTTP Request
include::{snippets}/product-create/http-request.adoc[]
include::{snippets}/product-create/request-fields.adoc[]

==== HTTP Response
include::{snippets}/product-create/http-response.adoc[]
include::{snippets}/product-create/response-fields.adoc[]
```

<br>

- index.adoc 코드
```adoc 
ifndef::snippets[]
:snippets: ../../build/generated-snippets
endif::[]
= CafeKiosk REST API 문서
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

[[Product-API]]
== Product API

include::api/product/product.adoc[]
```

<br>

그럼 아래와 같이 보기 좋은 api 문서가 생성된다.
<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/api.png?raw=true">  

<br>

이 상태에서 gradle에 build를 한 번 해주면, build/docs/asciidoc에 index.html 파일이 하나 생성된다.
<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/image.png?raw=true">  

<br>

이 파일을 chrome에서 열어주면, application이 실행됐을 때, 보여줄 수 있는 api 문서가 생성되었다!

<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/image-1.png?raw=true">  