# 섹션 1. 테스트는 왜 필요할까?
### 테트코드를 작성하지 않는다면
- 변화가 생기는 매순간마다 발생할 수 있는 모든 Case를 고려해야 한다.
- 변화가 생기는 매순간마다 모든 팀원이 동일한 고민을 해야 한다.
- 빠르게 변화하는 소프트웨어의 안정성을 보장할 수 없다.

### 올바른 테스트 코드는
- 자동화 테스트로 비교적 빠른 시간 안에 버그를 발견할 수 있고, 수동 테스트에 드는 비용을 크게 절약할 수 있다.
- 소프트웨어의 빠른 변화를 지원한다.
- 팀원들의 집단 지성을 팀 차원의 이익으로 승격시킨다.
- **가까이 보면 느리지만, 멀리 보면 가장 빠르다.**

### 결론
> **테스트는 `귀찮다.` 귀찮지만 `해야 한다.`**

<br><br>

# 섹션 2. 테스트 케이스 세분화하기
![세분화](https://github.com/wbluke/practical-testing/assets/68748397/62e56b66-286e-46e0-a97d-465b9a36a5ae)
- ex) 5 이상의 조건이라면 해피 케이스로 5, 예외 케이스를 4로 두 가지에 대하여 경계값을 테스트한다.

### 예외 케이스 예시
![image](https://github.com/wbluke/practical-testing/assets/68748397/8361fc56-e892-40f9-8c5f-be0a4588e896)

## 테스트하기 어려운 영역을 분리하기
![image](https://github.com/wbluke/practical-testing/assets/68748397/cde2489a-7ac9-4a5a-86b5-26be90cf363e)

> **아래와 같이 값이 계속해서 변경되는 코드는 테스트가 어렵기 때문에 외부로 분리하자**
### [Before]
![image](https://github.com/wbluke/practical-testing/assets/68748397/5c333818-6e09-49e7-a4ee-7c0ccb66bac6)

### [After]
![image](https://github.com/wbluke/practical-testing/assets/68748397/dad0b137-61ce-4515-9e1a-1bb08e7cfaaa)

<br><br>

# 섹션 3. TDD: Test Driven Development
> **TDD의 기본 프로세스는 레드 - 그린 - 리팩토링**

<br><br>

# 섹션 4. 테스트는 [문서]다.
![image](https://github.com/wbluke/practical-testing/assets/68748397/b96555f3-913d-4c4f-97e5-d57b0d985204)

## DisplayName을 섬세하게
> - ~~음료 1개 추가 테스트~~ -> `"~테스트" 지양하기`
> - ~~음료를 1개 추가할 수 있다.~~ -> `명사의 나열보다 문장으로`
    - A이면 B이다.
    - A이면 B가 아니고 C다.
> - 음료를 1개 추가하면 주문 목록에 담긴다. -> `테스트 행위에 대한 결과까지 기술하기`

<br>

> - ~~특정 시간 이전에 주문을 생성하면 실패한다.~~
> - 영업 시작 시간 이전에는 주문을 생성할 수 없다. -> `도메인 용어를 사용하여 한층 추상화된 내용을 담기`**(메서드 자체의 관점보다 도메인 정책 관점으로)**
> - 테스트의 현상을 중점으로 기술하지 말 것!

<br><br>

# 섹션 5. Spring & JPA 기반 테스트
### ORM(Object-Relational Mapping)
- 객체 지향 패러다임과 관계형 DB 패러다임의 불일치
- 이전에는 개발자가 객체의 데이터를 한땀한땀 매핑하여 DB에 저장 및 조회 (CRUD)
- ORM을 사용함으로써 개발자는 단순작업을 줄이고, 비즈니스 로직에 집중할 수 있다.

### JPA(Java Persistence API)
- Java 진영의 ORM 기술 표준
- 인터페이스이고, 여러 구현체가 있지만 보통 Hibernate를 많이 사용한다.
- **Spring 진영에서는 JPA를 한번 더 추상화한 Spring Data JPA 제공**
- QueryDSL과 조합하여 많이 사용한다. (타입체크, 동적쿼리)

## Persistence Layer
- Data Access의 역할
- 비즈니스 가공로직이 포함되어서는 안된다. (Data에 대한 CRUD에만 집중한 레이어)

```java
 // List 테스트 예제

 @ActiveProfiles("test")
//@SpringBootTest
@DataJpaTest
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @DisplayName("원하는 판매상태를 가진 상품들을 조회한다.")
    @Test
    void findAllBySellingStatusIn() {
        // given
        Product product1 = Product.builder()
                .productNumber("001")
                .type(HANDMADE)
                .sellingStatus(SELLING)
                .name("아메리카노")
                .price(4000)
                .build();
        Product product2 = Product.builder()
                .productNumber("002")
                .type(HANDMADE)
                .sellingStatus(HOLD)
                .name("카페라떼")
                .price(4500)
                .build();
        Product product3 = Product.builder()
                .productNumber("003")
                .type(HANDMADE)
                .sellingStatus(STOP_SELLING)
                .name("팥빙수")
                .price(7000)
                .build();
        productRepository.saveAll(List.of(product1, product2, product3));

        // when
        List<Product> products = productRepository.findAllBySellingStatusIn(List.of(SELLING, HOLD));

 // then
        assertThat(products).hasSize(2)
                .extracting("productNumber", "name", "sellingStatus") // 검증하고자 하는 필드만 추출
                .containsExactlyInAnyOrder( // 순서 상관없이 포함 여부 (.containsExactly: 순서 포함)
                        tuple("001", "아메리카노", SELLING),
                        tuple("002", "카페라떼", HOLD)
                );
```
- **✅ `extracting`, `containsExactlyInAnyOrder`, `tuple`, `containsExactly`**
## Business Layer 테스트 (1)
- Persistence Layer와의 상호작용(Data를 읽고 쓰는 행위)을 통해 비지니스 로직을 전개시킨다.
- `트랜잭션`을 보장해야 한다.

<br>

```java
assertThat(orderResponse.getId()).isNotNull(); // 아이디는 중요하지 않을 때 사용
assertThat(order.getOrderStatus()).isEqualByComparingTo(OrderStatus.INIT); // ENUM 비교
```
- **✅ `isNotNull`, `isEqualByComparingTo`**

## Business Layer 테스트 (2)
> ### **ex. `findAllByProductNumberIn` 쿼리 시, 아메리카노라는 상품의 상품번호로 주문이 2개 이상 중복일 경우 문제**
```java
public class OrderService {

        private List<Product> findProductsBy(List<String> productNumbers) {

        // in절에서 상품 중복 제거된 product만 리턴됨
        List<Product> products = productRepository.findAllByProductNumberIn(productNumbers);

        // map으로 가공, productNumber를 key값으로 하고 product를 value로 만듬
        Map<String, Product> productMap = products.stream()
                .collect(Collectors.toMap(Product::getProductNumber, p -> p));

        // productNumber를 순회하면서 productMap에서 product를 추출하여 중복된 상품도 리스트로 만듬
        return productNumbers.stream()
                .map(productMap::get)
                .collect(Collectors.toList());
        }
}
```

## Business Layer 테스트 (3)
> ### 아래와 같이 간단해 보이는 메소드도 테스트한다.   
> ### 이유: 언제 새로운 타입이 추가되거나 삭제될지 모르기 때문에
```java
@Getter
@RequiredArgsConstructor
public enum ProductType {

    HANDMADE("제조 음료"),
    BOTTLE("병 음료"),
    BAKERY("베이커리");

    private final String text;

    public static boolean containsStockType(ProductType type) {
        return List.of(BOTTLE, BAKERY).contains(type);
    }
}

class ProductTypeTest {

    @DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
    @Test
    void containsStockType() {
        // given
        ProductType givenType = ProductType.HANDMADE;

        // when
        boolean result = ProductType.containsStockType(givenType);

        // then
        assertThat(result).isFalse();
    }

    @DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
    @Test
    void containsStockType2() {
        // given
        ProductType givenType = ProductType.BAKERY;

        // when
        boolean result = ProductType.containsStockType(givenType);

        // then
        assertThat(result).isTrue();
    }

}
```
- **✅ `isZero == isEqualTo(0)`**

