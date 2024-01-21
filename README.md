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
@DataJpaTest // jpa와 관련된 빈들만 주입해주기 때문에 Service는 주입이 안될 수 있습니다.
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

### OrderService.java
```java
private void deductStockQuantities(List<Product> products) {
        List<String> stockProductNumbers = extractStockProductNumbers(products);

        Map<String, Stock> stockMap = createStockMapBy(stockProductNumbers);
        Map<String, Long> productCountingMap = createCountingMapBy(stockProductNumbers);

        for (String stockProductNumber : new HashSet<>(stockProductNumbers)) {
                Stock stock = stockMap.get(stockProductNumber);
                int quantity = productCountingMap.get(stockProductNumber).intValue();

                if (stock.isQuantityLessThan(quantity)) {
                throw new IllegalArgumentException("재고가 부족한 상품이 있습니다.");
                }
                stock.deductQuantity(quantity);
        }
}
```

### Stock.java
```java
public boolean isQuantityLessThan(int quantity) {
        return this.quantity < quantity;
}

public void deductQuantity(int quantity) {
        if (isQuantityLessThan(quantity)) {
                throw new IllegalArgumentException("차감할 재고 수량이 없습니다.");
        }
        this.quantity -= quantity;
}
```

> **예외 처리하는 부분이 중복되는것 같아 보이지만 OrderService.java에서는 주문 생성 로직을 수행과정에서 재고 감소를 하는 것이고 Stock.java는 밖에 서비스가 어떻게 되어있는지 전혀 모릅니다. 수량을 차감한다고 했을 때 올바른 수량 차감 로직을 보장해줘야 합니다. 또한 deductQuantity 메소드를 다른곳에서 또 쓰일 수 있기 때문에 전혀 다른 상황입니다.**

> ✅ 
> - **스프링은 기본적으로 JpaRepository의 구현제로 사용하는 SimpleJpaRepository에 읽기전용 트랜잭션이 걸려있습니다. 고로 트랜잭션이 보장됩니다.**
> - **SimpleJpaRepository에 save, delete 메서드에는 @Transactional 이 붙어있습니다.**
> - **`그러나 변경감지에 대해서는 직접 설정한 트랜잭션이 있어야 합니다.`**

### 리팩토링
1. 가공 로직은 메소드로 한단계 더 추상화
2. 호출되는 private 메소드들의 순서 맞춰주기
![image](https://github.com/wbluke/practical-testing/assets/154389971/c9e22597-c0a9-479b-850f-d9615de7ed7a)

```java
@Transactional
@RequiredArgsConstructor
@Service
public class OrderService {

    private final ProductRepository productRepository;
    private final OrderRepository orderRepository;
    private final StockRepository stockRepository;

    public OrderResponse createOrder(OrderCreateRequest request, LocalDateTime registeredDateTime) {
        List<String> productNumbers = request.getProductNumbers();
        List<Product> products = findProductsBy(productNumbers);

        deductStockQuantities(products);

        Order order = Order.create(products, registeredDateTime);
        Order savedOrder = orderRepository.save(order);
        return OrderResponse.of(savedOrder);
    }

    private void deductStockQuantities(List<Product> products) {
        List<String> stockProductNumbers = extractStockProductNumbers(products);

        Map<String, Stock> stockMap = createStockMapBy(stockProductNumbers);
        Map<String, Long> productCountingMap = createCountingMapBy(stockProductNumbers);

        for (String stockProductNumber : new HashSet<>(stockProductNumbers)) {
            Stock stock = stockMap.get(stockProductNumber);
            int quantity = productCountingMap.get(stockProductNumber).intValue();

            if (stock.isQuantityLessThan(quantity)) {
                throw new IllegalArgumentException("재고가 부족한 상품이 있습니다.");
            }
            stock.deductQuantity(quantity);
        }
    }

    private List<Product> findProductsBy(List<String> productNumbers) {
        List<Product> products = productRepository.findAllByProductNumberIn(productNumbers);
        Map<String, Product> productMap = products.stream()
                .collect(Collectors.toMap(Product::getProductNumber, p -> p));

        return productNumbers.stream()
                .map(productMap::get)
                .collect(Collectors.toList());
    }

    private static List<String> extractStockProductNumbers(List<Product> products) {
        return products.stream()
                .filter(product -> ProductType.containsStockType(product.getType()))
                .map(Product::getProductNumber)
                .collect(Collectors.toList());
    }

    private Map<String, Stock> createStockMapBy(List<String> stockProductNumbers) {
        List<Stock> stocks = stockRepository.findAllByProductNumberIn(stockProductNumbers);
        return stocks.stream()
                .collect(Collectors.toMap(Stock::getProductNumber, s -> s));
    }

    private static Map<String, Long> createCountingMapBy(List<String> stockProductNumbers) {
        return stockProductNumbers.stream()
                .collect(Collectors.groupingBy(p -> p, Collectors.counting()));
    }

}
```

## Presentation Layer 테스트 (1)
- 외부 세계의 요청을 가장 먼저 받는 계층
- 파라미터에 대한 최소한의 검증을 수행한다.

### @Transactional(readOnly = true)
- readOnly = true: 읽기전용
- CURD에서 CUD 동작 X / only Read
- JPA에서의 이점: 1차 캐시에 스냅샷을 찍고 트랜잭션 커밋 플러시 시점에 더티체킹을 하여 달라진 부분에 대하여 Update 쿼리를 날리지만 CUD 동작을 하지 않기 때문에 `스냅샷 저장`, `변경 감지`를 안해도 되는 이점이 생깁니다. => **`성능 향상`**
- CQRS(Command and Query Responsibility Segregation): Command와 Query 분리
- Master DB는 Write용으로 사용하고 Slave DB는 Read용으로 사용할 때 각 DB에 대하여 접근하는 엔드포인트(URL)을 분리할 때 @Transactional의 속성 값을 보고 나누어 설정할 수 있습니다.


## Presentation Layer 테스트 (2)
```java
@WebMvcTest(controllers = ProductController.class) // 컨트롤러 레이어만 떼서 테스트 하기위해서 컨트롤러 관련 빈들만 생성하기 위한 애노테이션
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean // 스프링 테스트 컨텍스트에 Mock 객체를 주입 (@WebMvcTest에서 주로 사용)
    private ProductService productService;

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
        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print()) // 로그
            .andExpect(status().isOk());
    }

    @DisplayName("신규 상품을 등록할 때 상품 타입은 필수값이다.")
    @Test
    void createProductWithoutType() throws Exception {
        // given
        ProductCreateRequest request = ProductCreateRequest.builder()
            .sellingStatus(ProductSellingStatus.SELLING)
            .name("아메리카노")
            .price(4000)
            .build();

        // when // then
        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("400"))
            .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
            .andExpect(jsonPath("$.message").value("상품 타입은 필수입니다."))
            .andExpect(jsonPath("$.data").isEmpty())
        ;
    }

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
                    .content(objectMapper.writeValueAsString(request))
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

    @DisplayName("신규 상품을 등록할 때 상품 이름은 필수값이다.")
    @Test
    void createProductWithoutName() throws Exception {
        // given
        ProductCreateRequest request = ProductCreateRequest.builder()
            .type(ProductType.HANDMADE)
            .sellingStatus(ProductSellingStatus.SELLING)
            .price(4000)
            .build();

        // when // then
        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("400"))
            .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
            .andExpect(jsonPath("$.message").value("상품 이름은 필수입니다."))
            .andExpect(jsonPath("$.data").isEmpty())
        ;
    }

    @DisplayName("신규 상품을 등록할 때 상품 가격은 양수이다.")
    @Test
    void createProductWithZeroPrice() throws Exception {
        // given
        ProductCreateRequest request = ProductCreateRequest.builder()
            .type(ProductType.HANDMADE)
            .sellingStatus(ProductSellingStatus.SELLING)
            .name("아메리카노")
            .price(0)
            .build();

        // when // then
        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("400"))
            .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
            .andExpect(jsonPath("$.message").value("상품 가격은 양수여야 합니다."))
            .andExpect(jsonPath("$.data").isEmpty())
        ;
    }

    @DisplayName("판매 상품을 조회한다.")
    @Test
    void getSellingProducts() throws Exception {
        // given
        List<ProductResponse> result = List.of();

        when(productService.getSellingProducts()).thenReturn(result); // 모킹해서 굳이 검사 안해도 됩니다. 이유: 데이터에 대한 구체적인 검증은 Service 레이어에서 함

        // when // then
        mockMvc.perform(
                get("/api/v1/products/selling")
                // 파라미터가 있을 경우 .queryParam("name", "이름")
            )
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value("200"))
            .andExpect(jsonPath("$.status").value("OK"))
            .andExpect(jsonPath("$.message").value("OK"))
            .andExpect(jsonPath("$.data").isArray()); // 데이터에 대한 구체적인 검증은 Service 레이어에서 했을 것
    }
}

```java
@WebMvcTest(controllers = OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private OrderService orderService;

    @DisplayName("신규 주문을 등록한다.")
    @Test
    void createOrder() throws Exception {
        // given
        OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of("001"))
            .build();

        // when // then
        mockMvc.perform(
                post("/api/v1/orders/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value("200"))
            .andExpect(jsonPath("$.status").value("OK"))
            .andExpect(jsonPath("$.message").value("OK"));
        ;
    }

    @DisplayName("신규 주문을 등록할 때 상품번호는 1개 이상이어야 한다.")
    @Test
    void createOrderWithEmptyProductNumbers() throws Exception {
        // given
        OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of())
            .build();

        // when // then
        mockMvc.perform(
                post("/api/v1/orders/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("400"))
            .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
            .andExpect(jsonPath("$.message").value("상품 번호 리스트는 필수입니다."))
            .andExpect(jsonPath("$.data").isEmpty())
        ;
    }
}
```

# 섹션 6. Mock을 마주하는 자세
## Mockito로 Stubbing하기
```java
/** OrderStatisticsService.java **/

public boolean sendOrderStatisticsMail(LocalDate orderDate, String email) {
        // 해당 일자에 결제완료된 주문들을 가져와서
        List<Order> orders = orderRepository.findOrdersBy(
            orderDate.atStartOfDay(), // 오늘이 17일이라면 17일의 0시
            orderDate.plusDays(1).atStartOfDay(),
            OrderStatus.PAYMENT_COMPLETED
        );

        // 총 매출 합계를 계산하고
        int totalAmount = orders.stream()
            .mapToInt(Order::getTotalPrice)
            .sum();

        // 메일 전송
        boolean result = mailService.sendMail(
            "no-reply@cafekiosk.com",
            email,
            String.format("[매출통계] %s", orderDate),
            String.format("총 매출 합계는 %s원입니다.", totalAmount)
        );

        if (!result) {
            throw new IllegalArgumentException("매출 통계 메일 전송에 실패했습니다.");
        }

        return true;
    }
```
- 메일 전송 로직을 사용하는 곳에서는 `@Transactional`을 붙이지 않는것이 좋습니다. 메일 전송 같은 긴 작업은 실제로 트랜잭션에 참여하지 않는게 좋습니다. 
- 어차피 orderRepository.findOrdersBy()와 같이 조회같은 것은 레파지토리 단에서 트랜잭션이 걸리기 때문입니다. 
- 대신 OrderStatisticsServiceTest.java를 작성할 때 서비스단에 `@Transactional`이 없는 경우에는 롤백을 보장할 수 없어서 테스트 코드에 따라 같은 클래스에 있는 다른 메소드의 영향으로 특정 메소드가 실패할 수도 있기 때문에 아래와 같은 정리 코드를 사용하여 지워줘야 합니다.
- **참고:** https://velog.io/@dkajffkem/Transactional-%EB%AA%A8%EB%A5%B4%EB%A9%B4-%EC%A0%88%EB%8C%80-%EC%95%88%EB%90%98%EB%8A%94%EA%B2%83%EB%93%A4-%EC%A7%84%ED%9D%99%EB%B0%AD%EC%97%90%EC%84%9C-%EC%95%8C%EA%B2%8C%EB%90%9C-Spring-JPA

```java
@AfterEach
void tearDown() {
        orderProductRepository.deleteAllInBatch();
        orderRepository.deleteAllInBatch();
        productRepository.deleteAllInBatch();
        mailSendHistoryRepository.deleteAllInBatch();
}
```
- **✅ `atStartOfDay()`**
  
> ### N(Order) : M(Product)을 1(Order) : N(OrderProduct), N(OrderProduct) : 1(Product)로 풀어냈을 때 `new OrderProduct(this, product)` 부분 처리를 잘 해주자
> **\* 추가로 Order와 Product는 단방향 관계로 매핑해주었습니다.**
```java
/** Order.java */

private LocalDateTime registeredDateTime;

@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
private List<OrderProduct> orderProducts = new ArrayList<>();

@Builder
private Order(List<Product> products, OrderStatus orderStatus, LocalDateTime registeredDateTime) {
        this.orderStatus = orderStatus;
        this.totalPrice = calculateTotalPrice(products);
        this.registeredDateTime = registeredDateTime;
        this.orderProducts = products.stream()
            .map(product -> new OrderProduct(this, product))
            .collect(Collectors.toList());
    }

public static Order create(List<Product> products, LocalDateTime registeredDateTime) {
        return Order.builder()
            .orderStatus(OrderStatus.INIT)
            .products(products)
            .registeredDateTime(registeredDateTime)
            .build();
    }
```

## @Mock, @Spy, @InjectMock
<img width="872" alt="image" src="https://github.com/f-lab-edu/hotel-java/assets/68748397/ac52e2e1-2d04-4791-a44b-92e2a8b27d75">

<br>

### Mockito.mock(), @Mcok, @MockBean
- `@MockBean`: 스프링 애플리케이션 컨텍스트에 Mock 객체를 추가하기 위해 사용
  - 예를들어 **`@MockBean private MailSendClient mailSendClient;`** MailSendClient 빈을 우리가 원하는 **Mockito의 Mock 객체로 갈아 끼우겠다**의 의미입니다.
- 항상 스프링 띄울때만 목 객체를 사용하는것은 아닙니다.
- 통합테스트가 아닌 단위테스트에서 목 객체를 사용할 일이 생겼을 때는 스프링 애플리케이션 컨텍스트를 로딩하지 않고도 사용할 수 있는 순수한 모키토인 Mockito.mock() 또는 @Mock 애노테이션을 사용합니다.
- 단위 테스트 시에 무분별하게 모킹을 통해 테스트를 작성하기 보다는 외부 라이브러리와 같이 직접 제어할 수 없는 경우에 모킹을 사용하는것이 좋습니다.

### verify()
```java
/** 애노테이션을 사용한 코드 **/

@Spy // 한 객체에서 일부는 실제객체, 일부는 Mock객체를 사용하고 싶을 때사용
private MailSendClient mailSendClient;

@Mock
private MailSendHistoryRepository mailSendHistoryRepository;

@InjectMocks
private MailService mailService;
```

```java
/** 애노테이션을 사용하지 않은 코드 **/

MailSendClient mailSendClient = Mockito.mock(MailSendClient.claas);
MailSendHistoryRepository mailSendHistoryRepository = Mockito.mock(MailSendHistoryRepository.class);

MailService = mailService = new MailService(mailSendClient, mailSendHistoryRepository);
```

`verify(T mock, VerificationMode mode)`
- mock: 행위를 검증하고자 하는 mock 객체
- mode: 검증할 값을 정의하는 메소드. 옵션이다.

```java
// given

// @Mock으로 사용할 경우
when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString())).thenReturn(true);

// @Spy로 사용할 경우
doReturn(true)
    .when(mailSendClient)
    .sendEmail(anyString(), anyString(), anyString(), anyString());

// when
boolean result = mailService.sendMail("", "", "", "");

// then
// mock 객체인 mailSendHistoryRepository의 save 메서드가 1번 호출되었는지
verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));
```

### @InjectMock
- `@InjectMocks private MailService mailService;`
- MailService의 생성자를 보고 Mockito의 mock 객체로 선언된 객체를 주입해줍니다. (DI와 같은 역할))

## BDDMockito
```java
// given

// Mockito.when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))
// .thenReturn(true);

given(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))
    .willReturn(true);
```
- **given인데 문법이 Mockito.when()으로 되어있기 때문에 Mockito를 상속받아서 BDDMockito.given() 으로 BDD 스타일로 변경**

## Classicist VS Mockist
실제 프로덕션 코드에서 런타임 시점에 일어날 일을 정확하게 Stubbing 했다고 단언할 수 있는가?

# 섹션 7. 더 나은 테스트를 작성하기 위한 구체적 조언
## 한 문단에 한 주제!

## 완벽하게 제어하기
```java
/** BAD Example **/
public Order createOrder() {
    LocalDateTime currentDateTime = LocalDateTime.now(); // 현재 시간이 내부에 있는 코드

    LocalTime currentTime = currentDateTime.toLocalTime();
    if (currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}

@Test
void createOrder() {
    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano = new Americano();
    cafeKiosk.add(americano);

    Order order = cafeKiosk.createOrder();

    assertThat(order.getBeverages()).hasSize(1);
    assertThat(order.getBeverages().get(0).getName()).isEqualTo("아메리카노");
}
```

```java
/** GOOD Example **/

public Order createOrder(LocalDateTime currentDateTime) { // 현재 시간을 외부로 분리
    LocalTime currentTime = currentDateTime.toLocalTime();
    if (currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(SHOP_CLOSE_TIME)) {
        throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
    }

    return new Order(currentDateTime, beverages);
}

@Test
void createOrderWithCurrentTime() {
    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano = new Americano();
    cafeKiosk.add(americano);

    // 현재시간을 외부로 분리하여 파라미터로 고정값을 줌으로써 원하는 상황을 완벽하게 연출할 수 있게 되었습니다.
    Order order = cafeKiosk.createOrder(LocalDateTime.of(2023, 1, 17, 10, 0));

    assertThat(order.getBeverages()).hasSize(1);
    assertThat(order.getBeverages().get(0).getName()).isEqualTo("아메리카노");
}
```

## 테스트 환경의 독립성을 보장하자
```java
/** OrderServiceTest.java **/

@DisplayName("재고가 부족한 상품으로 주문을 생성하려는 경우 예외가 발생한다.")
    @Test
    void createOrderWithNoStock() {
        // given
        LocalDateTime registeredDateTime = LocalDateTime.now();

        Product product1 = createProduct(BOTTLE, "001", 1000);
        Product product2 = createProduct(BAKERY, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);
        productRepository.saveAll(List.of(product1, product2, product3));

        Stock stock1 = Stock.create("001", 2);
        Stock stock2 = Stock.create("002", 2);
        stock1.deductQuantity(1); // todo
        stockRepository.saveAll(List.of(stock1, stock2));

        OrderCreateServiceRequest request = OrderCreateServiceRequest.builder()
                .productNumbers(List.of("001", "001", "002", "003"))
                .build();

        // when // then
        assertThatThrownBy(() -> orderService.createOrder(request, registeredDateTime))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("재고가 부족한 상품이 있습니다.");
    }
```
- **이 테스트는 주문을 생성하는 createOrder라는 행위를 위한 테스트인데 `stock1.deductQuantity(1)` 재고를 차감하는 행위가 포함됨으로써 두 가지 케이스가 혼합이 되는 문제가 발생하였습니다.**
- 이
- given 에서 맥락을 이해하는데 한번 더 생각을 해야하는 상황이 됨. (분기문과 반복문과 같은 논리구조가 들어감)
- 이 테스트가 실패를 할지 언정 // when & then 쪽에서 실패를 해야하는데 `stock1.deductQuantity(3)`으로 할 경우 given에서 실패가 떨어집니다.

## 테스트 간 독립성을 보장하자
```java
/** StockTest.java **/

private static final Stock stock = Stock.create("001", 1);
```
- **이러한 전역변수를 통하여 테스트 메소드들 끼리 공유를 하지 않게 작성해야합니다.**

## 한눈에 들어오는 Test Fixture 구성하기

## Test Fixture 클렌징

## ParameterizedTest

## DynamicTest

## 테스트 수행도 비용이다. 환경 통합하기

## Q. private 메서드의 테스트는 어떻게 하나요?

## Q. 테스트에서만 필요한 메서드가 생겼는데 프로덕션 코드에서는 필요 없다면?

# 섹션 8. Appendix
## 학습 테스트
```java
@DisplayName("주어진 개수만큼 List를 파티셔닝한다.")
@Test
void partitionLearningTest1() {
    // given
    List<Integer> integers = List.of(1, 2, 3, 4, 5, 6);

    // when
    List<List<Integer>> partition = Lists.partition(integers, 3);

    // then
    assertThat(partition).hasSize(2)
        .isEqualTo(List.of(
            List.of(1, 2, 3), List.of(4, 5, 6)
        ));
}
```
- **✅ `Lists.partition()`**

```java
@DisplayName("멀티맵 기능 확인")
@TestFactory
Collection<DynamicTest> multiMapLearningTest2() {
    // given
    Multimap<String, String> multimap = ArrayListMultimap.create();
    multimap.put("커피", "아메리카노");
    multimap.put("커피", "카페라떼");
    multimap.put("커피", "카푸치노");
    multimap.put("베이커리", "크루아상");
    multimap.put("베이커리", "식빵");

    return List.of(
        DynamicTest.dynamicTest("1개 value 삭제", () -> {
            // when
            multimap.remove("커피", "카푸치노");

            // then
            Collection<String> results = multimap.get("커피");
            assertThat(results).hasSize(2)
                .isEqualTo(List.of("아메리카노", "카페라떼"));
        }),
        DynamicTest.dynamicTest("1개 key 삭제", () -> {
            // when
            multimap.removeAll("커피");

            // then
            Collection<String> results = multimap.get("커피");
            assertThat(results).isEmpty();
        })
    );
}
```
- **✅ `Multimap, ArrayListMultimap.create()`**

## Spring REST Docs
