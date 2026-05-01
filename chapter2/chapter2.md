# 4️⃣ 네 개의 영역

![](https://velog.velcdn.com/images/rocker_nun/post/33c3f699-cac6-4e61-99be-65e76c1de4fe/image.png)


위 4개의 영역 중 **표현(Presentation)** 영역은 사용자의 요청을 받아 **응용(Application)** 영역에 전달하고 응용 영역의 결과를 다시 사용자에게 보여주는 역할을 한다. 표현 영역을 통해 사용자의 요청을 전달받는 응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현하는데 *“주문 등록”*, *“주문 취소”*, *“상품 상세 조회”* 와 같은 기능 구현을 예로 들 수 있다. 응용 영역 역시 기능을 구현하기 위해 도메인 영역의 도메인 모델을 사용한다. 아래 코드를 보자.

```java
public class CancelOrderService {
    
    @Transactional
    public void cancelOrder(String orderId) {
        Order order = findOrderById(orderId);
        if (order == null) {
            throw new OrderNotFoundException(orderId);          
        }
        order.cancel();
    }
    
    ...
}
```

주문 취소 기능을 제공하는 응용 서비스 코드다. 이처럼 응용 서비스는 로직을 직접 수행하기보다는 `Order`라는 도메인 모델에 책임을 위임해서 `Order` 클래스가 `cancel()` 메서드가 실행하도록 처리했다. 이처럼 응용 영역은 도메인 모델을 이용해서 사용자에게 제공할 기능을 구현한다. 실제 도메인 로직 구현은 도메인 모델에 위임한다.

**인프라스트럭처(Infrastructure)** 영역은 DB 연동, 메시징 큐에 메시지를 전송하거나 수신하는 기능을 구현하는 등 논리적인 개념을 표현하기보다는 실제 구현을 다루는 영역이다. 표현 영역은 HTTP, JSON, 웹 프레임워크 같은 기술을 사용해 외부 요청을 처리한다. 응용 영역도 트랜잭션 처리와 같은 프레임워크 기능을 일부 사용할 수 있다. 다만 핵심 도메인 규칙은 특정 구현 기술에 의존하지 않도록 도메인 영역에 위치시키는 것이 중요하다. 인프라스트럭처 영역은 DB, 메시징, 외부 API, 룰 엔진 같은 구체 기술을 담당한다.

&nbsp;

# ↕️ 계층 구조 아키텍처

위의 4개의 영역으로 나눠놓은 계층 구조에서는 상위 계층에서 하위 계층으로의 의존만 존재할 뿐, 그 반대 방향의 의존은 존재하지 않는다. 계층 구조를 엄격하게 지킨다고 한다면, 바로 아래의 하위 계층에만 의존해야 하지만, 상황에 따라 더 아래 계층에도 의존할 수도 있다.

<img src="https://velog.velcdn.com/images/rocker_nun/post/114bd3dd-71a0-4bb9-9a51-c946db8589b6/image.png" width=500 />


결국 응용 계층과 도메인 계층은 인프라스트럭처 계층의 기능을 사용하기 때문에 위와 같은 계층 구조가 직관적으로 이해하기 쉬울 것이다. 다만, 각 계층이 구체적인 구현 기술이 담겨 있는 인프라스트럭처 계층에 종속된다는 점은 명백한 사실이다.

&nbsp;

예시로 도메인의 가격 계산 규칙 코드를 보자.

```java
public class DroolsRuleEngine {
    private KieContainer kContainer;
    
    public DroolsRuleEngine() {
        KieServices ks = KieServices.Factory.get();
        kContainer = ks.getKieClasspathContainer();
    }
    
    public void evaluate(String sessionName, List<?> facts) {
        KieSession kSession = kContainer.newKieSession(sessionName);
        try {
            facts.forEach(x -> kSession.insert(x));
            kSession.fireAllRules();
        } finally {
            kSession.dispose();
        }
    }
}
```

&nbsp;

`evaluate()` 메서드에 파라미터를 넘기면 별도 파일로 작성한 규칙을 이용해서 연산을 수행하는 간단한 코드다. 이제 응용 영역은 가격 계산을 위해 인프라스트럭처 영역의 `DroolsRuleEngine`을 사용할 것이다.

```java
// 응용 계층의 서비스 코드
public class CalculateDiscountService {
    private DroolsRuleEngine ruleEngine;
    
    public CalculateDiscountService() {
        ruleEngine = new DroolsRuleEngine();
    }
    
    public Money calculateDiscount(List<OrderLine> orderLines, String customerId) {
        Customer customer = findCustomer(customerId);
        
        MutableMoney money = new MutableMoney(0);
        
        List<?> facts = new ArrayList<>();
        facts.add(customer);
				facts.add(money);
        facts.addAll(orderLines);
        
        ruleEngine.evaluate("discountCalculation", facts);
        return money.toImmutableMoney();
    }
    
    ...
}
```

별다른 문제가 없어 보이지만, 2가지 치명적인 문제가 있다. 먼저 `CalculateDiscountService` 클래스만 **테스트하기 어렵다**는 것이다. 왜냐하면 해당 클래스를 테스트하기 위해서는 그 전에 `ruleEngine`이 완벽하게 동작해야 한다. 두 번째 **문제는 구현 방식을 변경하기 어렵다**는 것이다. `evaluate()` 메서드로 연산을 수행하기 위해 *“discountCalculation”* 라는 이름의 세션을 넘겨야 하는데 이는 `Drools`의 세션 이름이다. 따라서 그 세션 이름이 바뀐다? `CalculateDiscountService` 코드까지 찾아와서 추가로 수정해줘야 한다. `Drools`가 아닌 다른 구현 기술로 변경하기로 결정했다면 문제는 더욱 커진다.

&nbsp;

# 👍 DIP로 문제 해결

![](https://velog.velcdn.com/images/rocker_nun/post/24f7f334-3781-4bf5-93f7-58543cd685ef/image.png)


가격 할인 계산 기능을 구현하는 `CalculateDiscountService` 고수준 모듈은 고객 정보도 구해야 하고, 룰을 실행하는 등 여러 하위 기능이 필요한 상황이다. 앞선 문제점들을 해결하기 위해서는 저수준 모듈이 고수준 모듈에 의존하도록 설계를 변경해야 할 필요가 있다. 바로 이때 **인터페이스(Interface)**를 사용하면 된다.

```java
// 고객 정보와 룰을 적용해서 할인 금액을 구하는 역할을 하는 인터페이스
public interface RuleDiscounter {
    Money applyRules(Customer customer, List<OrderLine> orderLines);
}

// 인터페이스에 의존하도록 리팩토링 한 서비스 코드
public class CalculateDiscountService {
    private RuleDiscounter ruleDiscounter;

    public CalculateDiscountService(RuleDiscounter ruleDiscounter) {
        this.ruleDiscounter = ruleDiscounter;
    }

    public Money calculateDiscount(List<OrderLine> orderLines, String customerId) {
        Customer customer = findCustomer(customerId);
        return ruleDiscounter.applyRules(customer, orderLines);
    }

    ...
}
```

이제 더 이상 `Drools`에 의존하는 코드는 전혀 찾아볼 수 없다. 그냥 `RuleDiscounter`가 룰을 적용한다는 사실만 알 뿐이다. 이제 상황에 맞게 해당 인터페이스를 구현한 구현체가 생성자를 통해 필드에 꽂힐 것이다.

![](https://velog.velcdn.com/images/rocker_nun/post/e1d60000-f1a5-445a-86b7-8095d9eabc70/image.png)


위 그림이 바로 최종 DIP를 적용한 다이어그램이다. 고수준 모듈이 저수준 모듈을 사용하려면 고수준 모듈이 저수준 모듈에 의존해야 하는데, 반대로 저수준 모듈이 고수준 모듈에 의존한다고 해서 **DIP(Dependency Inversion Principle) 의존 역전 원칙**이라고 한다.

&nbsp;

이제 테스트를 위해 고객을 찾는 기능도 고수준 인터페이스로 추가로 설계해서 서비스 코드를 리팩토링 해보자.

```java
public class CalculateDiscountService {
    private CustomerRepository customerRepository;  // 고객 조회 고수준 인터페이스에 의존
    private RuleDiscounter ruleDiscounter;  // 규칙 인터페이스에 의존

    public CalculateDiscountService(CustomerRepository customerRepository, RuleDiscounter ruleDiscounter) {
        this.customerRepository = customerRepository;
        this.ruleDiscounter = ruleDiscounter;
    }

    public Money calculateDiscount(List<OrderLine> orderLines, String customerId) {
        Customer customer = findCustomer(customerId);
        return ruleDiscounter.applyRules(customer, orderLines);
    }
    
    private Customer findCustomer(String customerId) {
        Customer customer = customerRepository.findById(customerId);
        if (customer == null) {
            throw new NoCustomerException();
        }
        return customer;
    }
    
    ...
}
```

이제 룰에 대한 로직과 고객을 조회하는 구체 클래스가 구현되어 있지 않더라도 대역 객체를 사용해서 테스트를 충분히 진행할 수 있게 됐다.

```java
public class CalculateDiscountServiceTest {
    
    @Test
    public void noCustomer_thenExceptionShouldBeThrown() {
        // 대역 객체 생성
        CustomerRepository stubRepo = mock(CustomerRepository.class);
        when(stubRepo.findById("noCusId")).thenReturn(null);
        
        RuleDiscounter stubRule = (cust, lines) -> null;
        
        // 대역 객체들을 주입 받아서 테스트 진행 가능
        CalculateDiscountService service = new CalculateDiscountService(stubRepo, stubRule);
        assertThrows(NoCustomerException.class, () -> service.calculateDiscount(someLines, "noCusId"));
    }
}
```

&nbsp;

### 💥 DIP 주의사항

DIP는 단순히 인터페이스와 구체 클래스로 쪼개는 것이 아니라 고수준 모듈이 저수준 모듈에 의존하지 않도록 하는 것이 핵심이다. 따라서 아래와 같이 저수준 모듈에서 인터페이스를 추출하면 안 된다.

![](https://velog.velcdn.com/images/rocker_nun/post/b6658408-6867-4a24-834a-a772aef38b5d/image.png)


이건 그냥 도메인 영역인 `CalculateDiscountService`가 구현 기술을 다루는 인프라스트럭처 계층에 직접적으로 의존하고 있는 상황이다. 고수준 모듈이 아닌 그냥 `RuleEngine`이라는 저수준 모듈에서 인터페이스를 뽑아낸 것이다. DIP를 적용할 때는 아래와 같이 **하위 기능을 추상화한 인터페이스는 고수준 모듈 관점에서 도출해야 한다.**

![](https://velog.velcdn.com/images/rocker_nun/post/4a256018-c293-4e0e-b559-909b304c8aed/image.png)


이처럼 인프라스트럭처 영역은 저수준, 응용 영역과 도메인 영역은 고수준 모듈이다. 근데 4개의 영역 구조를 보면 인프라스트럭처 영역이 가장 하단에 위치했는데 DIP를 적용하면 인프라스트럭처 영역이 응용 영역과 도메인 영역에 의존하는 구조가 된다. 하지만 고수준 모듈에서 정의한 인터페이스를 상속 받아서 구현하기 때문에 상위 계층에 영향을 주지 않는다. 아래 다이어그램을 보자.

![](https://velog.velcdn.com/images/rocker_nun/post/a23ec057-2361-40ee-8818-a4d769f1ace2/image.png)


다이어그램을 보면 인프라스트럭처 영역의 `CompositeNotifier` 클래스는 응용 영역의 `Notifier` 인터페이스를 구현하고, `JpaRepository`는 도메인 영역의 `OrderRepository` 인터페이스, `DroolsRuleDiscounter` 는 `RuleDiscounter` 인터페이스를 구현하고 있기 때문에 향후 새로운 요구사항이 들어왔을 때도 응용 영역의 `OrderService` 코드 변경 없이 기능을 교체하거나 확장할 수 있다.

&nbsp;

# 🦾 도메인 영역의 주요 구성요소

도메인 영역의 모델은 도메인의 주요 개념을 표현하며 핵심 로직을 구현한다. 도메인 영역의 주요 구성요소는 엔티티와 밸류 타입이라고 했는데 사실 다른 요소들이 더 존재한다. 전체적으로 정리해보자.

- **엔티티(Entity)**: 고유의 식별자를 갖는 객체로 자신의 생명주기를 갖는다. 주문, 회원, 상품과 같이 도메인의 고유한 개념을 표현한다. 도메인 모델의 데이터를 포함하며 해당 데이터와 관련된 기능을 함께 제공한다.

- **밸류(Value)**: 고유의 식별자를 갖지 않는 객체로 주로 개념적으로 하나인 값을 표현할 때 사용된다. 배송지 주소를 표현하기 위한 주소나 구매 금액을 위한 금액과 같은 타입이 밸류 타입이다. 엔티티의 속성으로 사용할 뿐만 아니라 다른 밸류 타입의 속성으로도 사용할 수 있다.
- **애그리거트(Aggregate)**: 애그리거트는 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다. 예를 들어, 주문과 관련된 `Order` 엔티티, `OrderLine` 밸류, `Orderer` 밸류 객체를 주문 애그리거트로 묶을 수 있다.
- **리포지토리(Repository)**: 도메인 모델의 영속성을 처리한다. 예를 들어, DBMS 테이블에서 엔티티 객체를 로딩하거나 저장하는 기능을 제공한다.
- **도메인 서비스(Domain Service)**: 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 할인 금액 계산은 상품, 회원 등급, 구매 금액 등 다양한 조건을 이용해서 구현하게 되는데, 이렇게 도메인 로직이 여러 엔티티와 밸류를 필요로 하면 도메인 서비스에서 로직을 구현한다.

&nbsp;

근데 엔티티를 볼 때마다 든 생각인데, *“DB 테이블을 설계할 때의 그 엔티티랑 같은 건가?”* 라는 의문이 들었다. 결론부터 말하면 도메인 모델의 엔티티와 DB 관계형 모델의 엔티티는 같은 것이 아니다. 차이점은 도메인 모델의 엔티티는 데이터와 함께 도메인 기능을 함께 제공한다는 점이다. 뭔 소리지? 아래 코드를 보자.

```java
public class Order {
    private OrderNo number;
    private Orderer orderer;
    private ShippingInfo shippingInfo;
    
    ...
    
    // 도메인 모델 엔티티는 도메인 기능도 함께 제공
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        ...
    }
}
```

이처럼 그냥 데이터만 담고 있는 객체가 아니라 구체적인 기능을 제공할 수 있다는 말이다. DB 테이블은 그냥 필드만 작성돼 있지, 메서드(기능)가 적혀 있지 않으니까…

그리고 또 다른 차이점은 도메인 모델의 엔티티는 2개 이상의 데이터가 개념적으로 하나인 경우에 밸류 타입으로 뽑아낼 수 있다는 점이다. 위의 `Orderer`만 보더라도 아래와 같이 주문자 이름과 이메일 데이터를 포함할 수 있다.

```java
public class Orderer {
    private String name;
    private String email;
    
    ...
}
```

DB 테이블에서 밸류 타입을 표현하기 위해서는 해당 테이블에 데이터를 넣거나 별도 테이블로 분리해야 한다.

&nbsp;

## 📦 애그리거트(Aggregate)

도메인의 규모가 커질수록 도메인 모델의 구성요소 또한 많아지고 복잡해진다. 이때 도메인 모델 개별 객체뿐만 아니라 상위 수준에서 모델을 볼 수 있어야 전체 모델의 관계와 개별 모델을 이해할 수 있다. 도메인 모델에서 전체 구조를 이해하는 데 도움이 되는 것이 바로 **애그리거트(Aggregate)**다.

애그리거트는 단순히 관련 있는 객체를 모아둔 묶음이 아니라, 함께 변경되어야 하고 같은 규칙 안에서 일관성을 지켜야 하는 객체들의 경계다. 따라서 애그리거트를 나눌 때의 기준은 ***“관련이 있는가?”*** 보다 ***“같은 트랜잭션 안에서 반드시 일관성을 유지해야 하는가?”*** 에 가깝다. 예를 들어 주문도 애그리거트인데, 주문은 *“주문”, “배송지 정보”, “주문자”, “주문 목록”, “총 결제 금액”* 의 하위 모델로 이루어져 있다. 애그리거트를 사용하면 각각의 객체가 아닌 관련 객체를 묶어서 객체 군집 단위로 도메인을 바라볼 수 있다. 따라서 애그리거트 간의 관계로 확장해서 도메인 모델을 이해할 수 있다.

애그리거트는 군집에 속한 객체를 관리하는 **루트 엔티티(Root Entity)**를 갖는다. 이 엔티티는 애그리거트에 속해 있는 엔티티와 밸류 객체를 이용해서 애그리거트가 구현해야 할 기능을 제공한다. 애그리거트를 사용하는 코드는 루트 엔티티가 제공하는 기능을 실행하고, 루트 엔티티를 통해 애그리거트 내의 다른 엔티티나 밸류 객체에 접근하여 내부 구현을 숨겨서 애그리거트 단위로 구현을 캡슐화할 수 있도록 돕는다. 이해를 위해 아래 다이어그램을 살펴보자.

![](https://velog.velcdn.com/images/rocker_nun/post/c536d602-3d3c-4458-bcbd-f73ca27c2496/image.png)


`Order` 애그리거트 루트는 주문 도메인 로직에 맞게 애그리거트의 상태를 관리한다. `Order`의 배송지 정보를 변경하기 위해서는 일단 배송지를 변경할 수 있는지 확인한 뒤에 배송지를 변경하는 것처럼 말이다. 아래 코드를 보자.

```java
public class Order {
    ...
    
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        checkShippingInfoChangeable();  // 배송지 변경 가능 여부 확인
        this.shippingInfo = newShippingInfo;
    }
    
    private void checkShippingInfoChangeable() {
        // 배송지 변경이 가능한지 여부를 검사하는 도메인 규칙
    }
}
```

쉽게 말해, 주문 애그리거트는 루트 엔티티인 `Order`를 통하지 않고서는 `ShippingInfo`를 변경할 수 있는 방법을 제공하지 않는다는 것이다. 애그리거트를 어떻게 구현해야 하는지는 이후에 살펴보도록 하자.

&nbsp;

### 🤔 도메인 모델 vs 애그리거트

**도메인 모델(Domain Model)**은 특정 도메인을 이해하고 표현하기 위한 개념 모델이다. 예를 들어 주문 도메인을 모델링한다면 `Order`, `OrderLine`, `ShippingInfo`, `Orderer`, `Money`, `OrderState` 같은 개념들이 등장할 수 있다. 이처럼 도메인 모델은 해당 도메인을 설명하기 위해 필요한 개념, 규칙, 상태, 관계를 포함하는 넓은 범위의 모델이다.

반면, **애그리거트(Aggregate)**는 도메인 모델 안에서 관련 객체들을 하나의 일관성 단위로 묶은 군집이다. 단순히 객체들이 서로 관련 있다고 해서 모두 하나의 애그리거트가 되는 것은 아니다. 애그리거트는 함께 변경되어야 하고, 반드시 같은 규칙 안에서 일관성을 유지해야 하는 객체들을 하나의 경계로 묶은 것이다.

예를 들어 주문은 `Order`, `OrderLine`, `ShippingInfo`, `Orderer`, `Money` 같은 여러 객체로 구성될 수 있다. 이때 `OrderLine`은 주문 없이 독립적으로 존재하기 어렵고, 총 주문 금액은 주문 항목들의 금액 합과 일치해야 하며, 배송지 변경이나 주문 취소 같은 규칙도 주문 상태에 따라 통제되어야 한다. 따라서 이 객체들은 `Order`를 중심으로 하나의 애그리거트로 묶을 수 있다.

```
Order 애그리거트
 ├── Order	// 애그리거트 루트
 ├── OrderLine
 ├── ShippingInfo
 ├── Orderer
 ├── Money
 └── OrderState
```

여기서 `Order`는 애그리거트 루트다. 애그리거트 루트는 애그리거트 내부 객체에 접근하고 변경하는 진입점 역할을 한다. 외부에서는 `OrderLine`이나 `ShippingInfo`를 직접 수정하는 것이 아니라, `Order.changeShippingInfo()`, `Order.cancel()` 같은 메서드를 통해 변경해야 한다. 그래야 애그리거트 내부의 규칙과 일관성을 안전하게 지킬 수 있다.

따라서 애그리거트는 도메인 모델이라는 큰 범주 안에 포함된다. 모든 도메인 모델이 애그리거트인 것은 아니지만, 애그리거트는 도메인 모델을 구성하는 중요한 설계 단위다. 한 문장으로 정리하면, **도메인 모델(Domain Model)은 도메인을 이해하기 위한 전체 지도이고, 애그리거트(Aggregate)는 그 지도 안에서 함께 일관성을 지켜야 하는 객체들의 경계**라고 볼 수 있다.

&nbsp;

## 💾 리포지토리(Repository)

도메인 객체를 지속적으로 사용하기 위해서는 DB에 안전하게 보관해야 한다. 이를 위해 사용하는 도메인 영역의 구성요소가 바로 **리포지토리(Repository)**다.

리포지토리는 애그리거트 단위로 도메인 객체를 저장하고 조회하는 기능을 정의한다. 예시로 주문 애그리거트를 위한 리포지토리는 아래와 같이 구현할 수 있다.

```java
public interface OrderRepository {
    Order findByNumber(OrderNumber number);
    void save(Order order);
    void delete(Order order);
}
```

보다시피 찾고 저장하는 대상이 `Order` 루트 애그리거트다. 도메인 모델을 사용해야 하는 코드는 일단 리포지토리를 통해 도메인 객체를 꺼내와서 기능을 수행하도록 해야 한다.

```java
public class CancelOrderService {
    private OrderRepository orderRepository;
    
    public void cancel(OrderNumber number) {
        Order order = orderRepository.findByNumber(number);  // 리포지토리에서 도메인 객체를 꺼냄
        if (order == null) {
            throw new NoOrderException(number);
        }
        order.cancel();  // 그 다음 기능 수행
    }
}
```

위 코드를 보면 알겠지만 리포지토리 도메인 모델은 도메인 객체를 영속화하는 데 필요한 기능들을 인터페이스로 추상화했기 때문에 고수준 모듈에 속한다. `OrderRepository`를 구현한 클래스들은 저수준 모듈로 인프라스트럭처 영역에 속하는 것이다.

![](https://velog.velcdn.com/images/rocker_nun/post/9bb4b0c1-76b0-40ff-9cd9-50405689a96b/image.png)


위의 코드와 다이어그램을 보면 응용 서비스는 의존 관계 주입을 통해 실제 리포지토리 구현 객체(`JpaOrderRepository`)에 접근하게 되는 것이다. 이처럼 응용 서비스는 리포지토리를 통해 애그리거트를 조회하고, 도메인 기능을 실행한 뒤, 변경된 상태가 저장소에 일관되게 반영되도록 트랜잭션 경계를 관리한다.

&nbsp;

# 🌊 요청 처리 흐름

알다시피 사용자의 요청을 처음 받는 영역은 **표현(Presentation)** 영역이다. 표현 영역은 사용자가 전송한 데이터 형식이 올바른지 검사하고 문제가 없다면 데이터를 이용해서 응용 서비스에 기능 실행을 위임한다. 이때 표현 영역은 데이터를 응용 서비스가 요구하는 형식으로 변환해서 전달하게 된다. 아래는 대략적인 요청 처리 흐름이다.

![](https://velog.velcdn.com/images/rocker_nun/post/26f233e9-181b-4e4f-87e7-b195f2340a1e/image.png)


보다시피 컨트롤러(표현 영역)가 사용자의 요청을 받아서 응용 서비스가 요구하는 형식으로 데이터를 변환해서 응용 서비스에 전달한다. 응용 서비스는 도메인의 기능을 사용하기 위해 리포지토리로부터 도메인 객체를 꺼내서 실행하거나 신규 도메인 객체를 생성해서 리포지토리에 저장한다. 추가로 도메인의 상태가 변경되는 기능은 리포지토리에 일관되게 반영되도록 트랜잭션을 관리해야 한다.

```java
public class CancelOrderService {
    private OrderRepository orderRepository;

	// 응용 서비스는 트랜잭션을 관리해야 한다.
    @Transactional
    public void cancel(OrderNumber number) {
        Order order = orderRepository.findByNumber(number);
        if (order == null) {
            throw new NoOrderException(number);
        }
        order.cancel();
    }
    ...
}
```

&nbsp;

# 🏗️ 인프라스트럭처 개요

**인프라스트럭처(Infrastructure)**는 다른 영역에서 필요로 하는 프레임워크, 구현 기술, 보조 기능을 지원한다. 여기서 명심해야 할 점은 위에서도 말했지만 고수준 모듈에서 정의한 인터페이스를 저수준 모듈인 인프라스트럭처 영역에서 구현하는 것이 시스템을 더 유연하게 만들고 테스트하기 쉽게 만들어준다. 하지만 상황에 따라 인프라스트럭처에 대한 의존을 일부 도메인에 넣을 수도 있다.


&nbsp;

# 🗂️ 모듈 구성

위에서 봤던 표현, 응용, 도메인, 인프라스트럭처 계층이 있었듯이 각 영역별로 모듈이 위치할 패키지를 구성해주는 것이 기본이다. 여기서 도메인에 따라 알맞은 패키지로 대체할 수 있다. 도메인 하나가 너무 크다면 여러 개의 하위 도메인으로 나누고 그 도메인마다 패키지를 구성하면 된다. 여기서 도메인 모듈은 도메인에 속한 애그리거트를 기준으로 다시 패키지를 구성한다.

```
com.myshop.order
 ├── ui
 │   └── OrderController
 ├── application
 │   ├── CancelOrderService
 │   └── PlaceOrderService
 ├── domain
 │   ├── Order
 │   ├── OrderLine
 │   ├── OrderNo
 │   ├── OrderRepository
 │   └── ShippingInfo
 └── infrastructure
     └── JpaOrderRepository
```

애그리거트, 모델, 리포지토리는 같은 패키지에 위치시킨다. 도메인이 너무 복잡하면 도메인 모델과 도메인 서비스를 별도 패키지에 위치시킬 수도 있다. 이처럼 모듈 구조를 어디까지 나눠야 하는지에 대한 정답은 없다. 하나의 패키지에 몇 개 정도의 타입을 구성해야 하는지에 대한 본인만의 기준을 세우는 것이 좋다.