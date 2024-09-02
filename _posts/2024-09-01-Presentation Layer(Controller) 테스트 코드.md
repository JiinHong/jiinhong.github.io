---

layout: post
title: Presentation Layer(Controller) 테스트
date: 2024-09-01 00:00:00 +0800
categories: [CS Study, testcode]
tags: [Sophomore, Testcode]
pin: true

---


## Presentation Layer(Controller) 테스트

<br>

### 1. WebMvcTest와 MockBean

```java
// 테스트 하고자하는 컨트롤러 명시
@WebMvcTest(controllers = ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    // 2번에서 설명
    @Autowired
    private ObjectMapper objectMapper;

    // 서비스 layer에 MockBean 명시
    @MockBean
    private ProductService productService;

    // ProductRepository까지 명시해줄 필요는 없음.
    // 여기서의 ProductService는 mock(가짜)이니까...
}
```

@WebMvcTest는 컨트롤러만을 테스트하기 위한 어노테이션이다.  
Postman으로 직접 HTTP 요청을 만들지 않아도 컨트롤러의 동작을 테스트할 수 있다.  

- @WebMvcTest : 컨트롤러 레이어만 신속하게 테스트 (단위 테스트)
- Postman : 실제 배포된 api 동작을 테스트 (통합 테스트)

<br>

여기서, 해당 컨트롤러의 테스트만을 진행하는 @WebMvcTest이기 때문에, 그 컨트롤러가 의존하고 있는 Service 객체를 @MockBean으로 컨테이너에 등록해줘야한다. 그러면, 실제로 서비스 로직을 실행하지 않고도, mock된 서비스 객체를 사용해서 컨트롤러를 독립적으로 테스트할 수 있다.

<br>

### 2. mockMvc를 이용한 api 요청 테스트

```java
@DisplayName("신규 상품을 등록한다.")
@Test
void createProduct() throws Exception {
    // given
    ProductCreateRequest request = ProductCreateRequest.builder()
        .type(ProductType.HANDMADE)
        .sellingStatus(ProductSellingStatus.SELLING)
        .name("아메리카노")
        .price(4000)
        .build();

    // when // then
    // perform : api를 쏘는 수행
    // objectMapper : json과 object 간의 직렬화, 역직렬화를 도와줌.
    mockMvc.perform(
            post("/api/v1/products/new")
                .content(objectMapper.writeValueAsString(request))
                .contentType(MediaType.APPLICATION_JSON)
        )
        .andDo(print())
        .andExpect(status().isOk());
}
```

perform는 api 요청을 수행해주는 메서드이다. 그런데 여기서, objectMapper를 통해 json 형태의 문자열로 직렬화하는 과정이 필요하다. 

<br>

**여기서, json의 직렬화와 역직렬화란?**  

1. 직렬화 : 객체를 외부 시스템에서 사용할 수 있도록 바이트 형태로 데이터를 변환하여 json 파일로 만드는 과정이다.
2. 역직렬화 : 거꾸로 json 파일(외부 시스템의 바이트 형태의 데이터)을 객체로 변환하는 과정  

Jackson 라이브러리는 직렬화와 역직렬화를 쉽게 처리하게 해주는데, Spring Boot에서는 이 Jackson 라이브러리를 기본적으로 통합하여 사용한다. **여기서 ObjectMapper가 직렬화와 역직렬화를 지원한다.**  
우리가 자주 사용하는 @RequestBody 또한 json 요청 본문이 자동으로 자바 객체로 역직렬화하게 해주는 기능인 것이다. (반대로, @ResponseBody는 json으로 직렬화)  

<br>

print()로 요청과 응답을 콘솔에 출력하고, status().isOk()로 api 응답 상태 코드가 200 OK인지 확인한다.

<br>

### <참고> 직렬화와 역직렬화 확인해보기
```java
package student;

import java.io.Serializable;

public class Student implements Serializable {

    private long studentId;
    private String name;
    private transient int age;	// transient 변수는 직렬화에서 제외

    public Student(long studentId, String name, int age) {
        this.studentId = studentId;
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return String.format("%d - %s - %d", studentId, name, age);
    }

}
```

```java
package student;

import java.io.*;
import java.util.Base64;

public class StudentMain {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Student student = new Student(123L, "박진홍", 24);
        byte[] serializedMember; // 직렬화된 student를 담을 byte배열

        // 바이트 배열에 데이터를 쓸 수 있는 스트림 생성
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        // ObjectOutputStream으로 객체를 바이트로 변환
        ObjectOutputStream oos = new ObjectOutputStream(baos);

        // 여기서 student 객체를 직렬화하여 바이트로 변환
        oos.writeObject(student);

        // 직렬화된 바이트 데이터를 byte 배열로 변환
        serializedMember = baos.toByteArray();
        
        // 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
        System.out.println(student.toString());
        System.out.println(Base64.getEncoder().encodeToString(serializedMember));
        String base64Student = Base64.getEncoder().encodeToString(serializedMember);

        // 그럼 이제, 직렬화된 데이터를 다시 역직렬화 해보자.
        byte[] serializedMember1 = Base64.getDecoder().decode(base64Student);

        // ByteArrayInputStream에 raw한 데이터를 저장
        ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember1);
        
        // ObjectOutputStream으로 바이트를 객체로 변환
        ObjectInputStream ois = new ObjectInputStream(bais);
        
        // 역직렬화된 Student 객체를 읽어온다.
        Object objectMember = ois.readObject();
        Student student1 = (Student) objectMember;
        System.out.println(student1);
    }
}
```

하지만 이러한 방식은 치명적인 보안 이슈가 있고, 여러가지 제약 상황(객체 구조 변경 불가, 엄격한 타입 체크 등)이 많아서 사용되지 않는다. 조슈아 블로크도 JSON 등의 포맷을 사용하는 것을 추천한다.

<br>

**<강의 도중 트러블 슈팅(?)>**  
(1) @EnableJpaAuditing과 @WebMvcTest의 문제
```java
@EnableJpaAuditing
@SpringBootApplication
public class  CafekioskApplication {

    public static void main(String[] args) {
        SpringApplication.run(CafekioskApplication.class, args);
    }

}
```  
@EnableJpaAuditing을 써놓으면, jpa와 관련된 빈들이 필요하다. 하지만, @WebMvcTest는 컨트롤러만을 테스트하기 위한 용도이기 때문에, JPA 관련 빈들을 로드하지 않는다. 그래서 @EnableJpaAuditing가 필요로 하는 빈을 찾을 수 없다고 오류가 발생한 것이다.  
따라서, 아래와 같이 관련 내용을 config로 분리하여 해결한다.

```java
@EnableJpaAuditing
@Configuration
public class JpaAuditingConfig {
}
```

<br>

(2) 역직렬화 과정에서 기본생성자가 필요
```java
@Getter
@NoArgsConstructor // 기본 생성자 필요
public class ProductCreateRequest {

    @NotNull(message = "상품 타입은 필수입니다.")
    private ProductType type;

    @NotNull(message = "상품 판매상태는 필수입니다.")
    private ProductSellingStatus sellingStatus;

    @NotBlank(message = "상품 이름은 필수입니다.")
    private String name;

    @Positive(message = "상품 가격은 양수여야 합니다.")
    private int price;

    @Builder
    private ProductCreateRequest(ProductType type, ProductSellingStatus sellingStatus, String name, int price) {
        this.type = type;
        this.sellingStatus = sellingStatus;
        this.name = name;
        this.price = price;
    }
}
```

jackson 라이브러리에서는 기본 생성자를 이용하여 객체를 먼저 생성한 다음, json 데이터 값을 해당 객체의 필드에 설정한다. 근데 Builder를 써놓고, 기본 생성자를 만들지 않으면, 객체를 생성할 수 없기 때문에 역직렬화(json -> 객체)가 실패하게 된다.

<br>


### 3. 데이터 유효성 검증

```java
@PostMapping("/api/v1/products/new")
public ApiResponse<ProductResponse> createProduct(@Valid @RequestBody ProductCreateRequest request) {
    return ApiResponse.ok(productService.createProduct(request.toServiceRequest()));
}
```

파라미터에 @Valid을 붙여주면, 아래와 같이 해당 request에서 다양한 유효성 검사를 할 수 있다.

```java
public class ProductCreateRequest {

    // null이 아니어야 함.
    @NotNull(message = "상품 타입은 필수입니다.")
    private ProductType type;

    @NotNull(message = "상품 판매상태는 필수입니다.")
    private ProductSellingStatus sellingStatus;

    // null이 아니어야 하며, 빈 문자열이 아니어야함.
    @NotBlank(message = "상품 이름은 필수입니다.")
    private String name;

    // 값이 양수여야 함.
    @Positive(message = "상품 가격은 양수여야 합니다.")
    private int price;
}
```

```java
@DisplayName("신규 상품을 등록할 때 상품 판매상태는 필수값이다.")
@Test
void createProductWithoutSellingStatus() throws Exception {
    // given
    ProductCreateRequest request = ProductCreateRequest.builder()
        .type(ProductType.HANDMADE)
        .name("아메리카노")
        .price(4000)
        .build();

    // when // then
    mockMvc.perform(
            post("/api/v1/products/new")
                .content(objectMapper.writeValueAsString(request)) // 여기서 직렬화
                .contentType(MediaType.APPLICATION_JSON)
        )
        .andDo(print())
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.code").value("400"))
        .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
        .andExpect(jsonPath("$.message").value("상품 판매상태는 필수입니다."))
        .andExpect(jsonPath("$.data").isEmpty())
    ;
}
```

<br>
