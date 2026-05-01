# 📝 도메인(Domain)이란?

온라인 서점을 한번 떠올려보면 장바구니 기능, 쿠폰 적용 기능, 결제 기능, 배송 추적 기능 등 상당히 많은 기능들을 가지고 있다. 이때 **온라인 서점**이 바로 소프트웨어로 해결하고자 하는 문제 영역, 즉 **도메인(Domain)**에 해당한다.

한 도메인은 다시 하위의 여러 도메인으로 나눌 수 있다. 위의 온라인 서점만 생각하더라도 카탈로그 도메인, 주문 도메인, 혜택 도메인, 배송 도메인 등 여러 도메인으로 쪼개질 수 있다. 이 하위 도메인들이 서로 연동하여 완전한 기능을 제공하는 것이다. 여기서 하나의 도메인을 맡았다고 그 도메인이 제공해야 할 모든 기능을 직접 구현하는 것은 아니다. 예를 들어, 배송 시스템은 다른 업체의 제품을 채택하고 그 제품을 사용하기 위해 필요한 기능만 일부 연동하는 것이다.

알다시피 개발자는 고객의 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다. 따라서 요구사항을 명확하게 이해하는 것이 가장 중요한데, 이를 위해 그 해당 도메인의 전문가와 직접 대화해서 그 전문가만큼은 아니어도 해당 도메인 지식을 갖춰야 할 필요가 있다. 도메인 주도 설계는 단순히 클래스를 잘 나누는 설계 기법이 아니다. 복잡한 비즈니스 문제를 이해하고, 도메인 전문가와 개발자가 같은 언어로 소통하며, 그 이해를 모델과 코드에 반영해 나가는 개발 방식이다.

&nbsp;

# 🩻 도메인 모델(Domain Model)

도메인 모델은 특정 도메인을 개념적으로 표현한 것이다. 예시로 온라인 서점의 주문 모델을 객체 모델로 구성하면 아래 그림과 같이 만들 수 있다.

### 🍔 클래스 다이어그램

![](https://velog.velcdn.com/images/rocker_nun/post/3fbe6033-94b3-452b-a54b-b80d8214f597/image.png)


위 도메인 모델은 **객체를 이용한 도메인 모델**이다. 도메인을 이해하기 위해서는 해당 **도메인이 제공하는 기능**과 **필요한 주요 데이터**를 알아야 하는데, 이런 면에서 객체 모델이 도메인을 모델링하기에 안성맞춤이다. 주문(`Order`)은 주문 번호(`orderNumber`), 총 주문 금액(`totalAmounts`)이 있고, 배송 정보(`ShippingInfo`)를 변경(`changeShipping`)할 수 있고, 주문을 취소(`cancel`)할 수 있다는 사실을 바로 알 수 있다. 이처럼 개발자가 아니더라도 해당 도메인에 대한 이해가 필요한 모든 사람들이 지식을 이해하고 공유하는데 많은 도움이 된다.

하지만 도메인을 모델링하기 위해 꼭 객체만 사용할 수 있는 것은 아니다. 아래와 같이 상태 다이어그램을 이용해서 주문의 상태 전이를 모델링할 수도 있다.

&nbsp;

### 🔁 상태 다이어그램

![](https://velog.velcdn.com/images/rocker_nun/post/d453bf53-57a1-4b0f-a086-23f3a4aa22b9/image.png)


이외에도 다양한 방법으로 도메인을 모델링할 수 있다. 관계가 중요한 도메인이라면 그래프를 이용할 수 있고, 계산 규칙이 중요하다면 수학 공식을 활용해서 모델링 할 수도 있다. 하여간 도메인을 이해하는데 도움이 된다면 그 어떤 것이든 가능하다.

정리하자면, 도메인 모델은 도메인 자체를 이해하기 위한 개념 모델이다. 이 개념 모델만으로는 바로 기능을 구현할 수 있는 것은 아니기에 구현 기술에 맞는 별도의 구현 모델도 필요하다. ~~그나마 위 클래스 다이어그램은 객체지향 언어로 개념 모델에 가깝게 구현할 수는 있다…~~ 처음부터 완벽한 개념 모델을 만들겠다는 마음보다는 전반적인 개요를 알 수 있는 수준으로 개념 모델을 작성해야 한다. 이후에 코드로 구현하는 과정에서 개념 모델을 구현 모델로 점진적으로 발전시켜 나가야 한다.

&nbsp;

### 💥 주의 사항

따라서 여러 하위 도메인의 개념을 하나의 모델에 억지로 합치려고 하면 의미가 뒤섞일 수 있다. 같은 “상품”이라는 용어도 카탈로그 컨텍스트에서는 가격과 설명을 가진 판매 정보일 수 있고, 배송 컨텍스트에서는 실제 고객에게 전달되는 물리적인 물품을 의미할 수 있다.

그래서 도메인 모델은 특정 문맥 안에서 일관된 의미를 가져야 한다. DDD에서는 이런 모델의 의미가 유지되는 경계를 **바운디드 컨텍스트(Bounded Context)**라고 부른다. 하위 도메인은 비즈니스 문제 영역을 나눈 것이고, 바운디드 컨텍스트는 특정 모델이 일관된 의미를 갖는 경계다.

&nbsp;

# 📦 도메인 모델 패턴

일반적인 애플리케이션의 아키텍처는 아래와 같다.

![](https://velog.velcdn.com/images/rocker_nun/post/4587fae4-97a6-4b0d-945b-90847aa14edd/image.png)


- `사용자 인터페이스(UI) or 표현(Presentation)`: 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여기서 사용자는 소프트웨어를 사용하는 사람뿐만 아니라 외부 시스템일 수도 있다.

- `응용(Application)`: 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다.
- `도메인(Domain)`: 시스템이 제공할 도메인 규칙을 구현한다.
- `인프라스트럭처(Infrastructure)`: 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.


&nbsp;

위에서 살펴봤던 도메인 모델은 진짜 도메인에 대해 이해하기 위해 필요한 개념 모델을 의미했다면, 지금부터 살펴볼 도메인 모델은 일종의 패턴을 의미한다. 좀 더 자세히 말하자면, 아키텍처 상의 도메인 계층에서는 도메인의 핵심 규칙을 구현하는데, 이런 도메인의 규칙을 객체지향 기법으로 구현하는 패턴을 말한다. 직접 코드를 보면서 이해해보자.

```java
// 주문 상태와 배송 정보를 담은 Order 클래스
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippingInfo = newShippingInfo;
    }
    
    ...
}

// 주문 상태에 대한 정보를 담은 OrderState 클래스
public enum OrderState {
    PAYMENT_WAITING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    PREPARING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    SHIPPED, DELIVERING, DELIVERY_COMPLETED;

    public boolean isShippingChangeable() {
        return false;
    }
}
```

주문 도메인의 일부 기능을 도메인 모델 패턴으로 구현한 것이다. 주문 상태를 표현하는 `OrderState`는 현재 주문 상태에서 배송지 정보를 변경할 수 있는지 판단하는 `isShippingChangeable()` 메서드를 제공한다. `PAYMENT_WAITING` 또는 `PREPARING` 상태에서는 아직 출고 전이므로 배송지 변경이 가능하고, `SHIPPED`, `DELIVERING`, `DELIVERY_COMPLETED` 상태에서는 배송지 변경이 불가능하다는 도메인 규칙을 코드로 표현한 것이다.

근데 여기서 `OrderState`는 `Order`에 속한 데이터라고 판단할 수도 있지 않나? 주문 상태를 변경할 수 있는지에 대한 판단 정도는 그냥 `Order`에 넣어도 될거 같다. 아래 코드를 보자.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippingInfo = newShippingInfo;
    }

	// 주문 상태를 변경할 수 있는지 여부를 검사하는 메서드를 Order 클래스 내부로 이동
    private boolean isShippingChangeable() {
        return state == OrderState.PAYMENT_WAITING || state == OrderState.PREPARING;
    }

    ...

}

public enum OrderState {
    PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED;
}
```

큰 무리는 없어 보인다. 만약 해당 메서드가 주문 상태뿐만 아니라 다른 정보를 함께 사용하고 있다면 `OrderState`만으로는 배송지를 변경할 수 있는지 판단할 수 없으므로 `Order`에서 로직을 구현해야 할 것이다. 아무튼 여기서 중요한 점은 해당 메서드가 `Order`에 있든 `OrderState`에 있든 결국 주문과 관련된 중요 업무 규칙을 주문 도메인 모델인 `Order`나 `OrderState`에서 구현해야 한다는 것이다.

&nbsp;

# 🔍 도메인 모델 도출

> ***“그래서 도메인 모델을 어떻게 뽑아내는데?”***
>

개인적으로 그동안 가장 많이 고민하고 판단 기준이 모호하다고 느낀 부분이다. 위에서 언급되었듯이 답은 요구사항에 있다. 요구사항으로부터 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다. 주문 도메인에 아래와 같은 요구사항이 있다고 해보자.

- 최소 한 종류 이상의 상품을 주문해야 한다.
- 한 상품을 1개 이상 주문할 수 있다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
- 주문할 때 배송지 정보를 반드시 지정해야 한다.
- 배송지 정보는 받는 사람 이름, 전화번호, 주소로 구성된다.
- 출고를 하면 배송지를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다.
- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

&nbsp;

위 요구사항에서 뽑아낼 수 있는 것은 주문은 배송지를 변경하는 기능, 결제를 완료하는 기능, 출고 상태로 변경하는 기능, 주문을 취소하는 기능을 제공한다는 것이다.

```java
public class Order {
    public void changeShippingInfo(ShippingInfo shippingInfo) {...}
    public void completePayment() {...}
    public void changeShipped() {...}
    public void cancel() {...}
}
```

&nbsp;

그리고 *“한 상품을 1개 이상 주문할 수 있다”*, *“각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다”* 라는 요구사항에서는 주문 항목의 데이터를 뽑아낼 수 있다. 주문할 상품, 상품의 가격, 구매 수량, 각 구매 항목의 구매 가격을 포함해야 한다. 이를 바탕으로 `OrderLine`을 구현해보자.

```java
public class OrderLine {
    private Product product;
    private int price;
    private int quantity;
    private int amount;
    
    public OrderLine(Product product, int price, int quantity) {
        this.product = product;
        this.price = price;
        this.quantity = quantity;
        this.amount = calculateAmount();
    }
    
    public int getAmount() {
        ...
    }
    
    private int calculateAmount() {
        return price * quantity;
    }
}
```

보다시피 `OrderLine`은 한 상품을 얼마에, 몇 개 살지를 담고 있고 `calculateAmount()` 메서드로 구매 가격을 구하는 로직을 담고 있다. 그리고 나서 아래 요구사항으로 `Order`와의 관계를 알 수 있다.

- 최소 한 종류 이상의 상품을 주문해야 한다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.

&nbsp;

`Order` 클래스를 리팩토링 해보자.

```java
public class Order {
    private List<OrderLine> orderLines;
    private Money totalAmount;
    
    public Order(List<OrderLine> orderLines) {
        setOrderLines(orderLines);
    }
    
    private void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calculateTotalAmount();
    }
    
    private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
        if (orderLines == null || orderLines.isEmpty()) {
            throw new IllegalArgumentException("No OrderLines...");
        }
    }
    
    private void calculateTotalAmount() {
        int sum = orderLines.stream()
                .mapToInt(x -> x.getAmount())
                .sum();
        this.totalAmount = new Money(sum);
    }
    
    ...  // 다른 메서드
}
```

최소 한 종류 이상의 상품을 주문해야 한다는 규칙은 실제 `Order`를 생성할 때 `setOrderLines()` 메서드에 `OrderLine` 리스트를 전달해서 `verifyAtLeastOneOrMoreOrderLines()` 메서드로 검사한다. 검사를 통과하면 `calculateTotalAmount()` 메서드로 총 주문 금액을 계산하도록 처리했다.

&nbsp;

이제 받는 사람 이름, 전화번호, 주소 데이터를 가지고 있는 배송지 정보 클래스를 아래와 같이 정의해보자.

```java
public class ShippingInfo {
    private String receiverName;
    private String receiverPhoneNumber;
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipcode;
    
    ... 생성자, getter
}
```

&nbsp;

위의 요구사항 중 *“주문할 때 배송지 정보를 반드시 지정해야 한다”* 라는 규칙이 있었다. 이 규칙으로 `Order`를 새로 생성할 때 `OrderLine` 뿐만 아니라 `ShippingInfo`도 전달해야 한다는 것을 알 수 있다. 다시 `Order` 클래스를 리팩토링 해보자.

```java
public class Order {
    private List<OrderLine> orderLines;
    private ShippingInfo shippingInfo;  // 배송 정보 필드 추가
    
    ...

    public Order(List<OrderLine> orderLines, ShippingInfo shippingInfo) {
        setOrderLines(orderLines);
        setShippingInfo(shippingInfo);
    }

    private void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calculateTotalAmount();
    }
    
    // 생성 시 배송 정보를 설정하는 메서드
    private void setShippingInfo(ShippingInfo shippingInfo) {
        if (shippingInfo == null) {
            throw new IllegalArgumentException("No ShippingInfo...");
        }
        this.shippingInfo = shippingInfo;
    }
    
    ... 
}
```

&nbsp;


이외에도 여러가지 제약과 규칙이 존재한다. *“출고를 하면 배송지 정보를 변경할 수 없다”* 와 *“출고 전에 주문을 취소할 수 있다”* 와 같은 요구사항은 출고 상태가 되기 전과 후의 제약사항이다. 그럼 `Order`는 최소한 출고 상태에 대해서는 알고 있어야 한다. 이제 기존의 `changeShippingInfo()` 메서드와 `cancel()`은 출고 전에 실행되도록 리팩토링하자.

```java
public class Order {
    private OrderState state;
    
    ...

	// 출고 전일 때만 배송지를 변경할 수 있는 제약 추가
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);
    }
    
    // 출고 전일 때만 주문을 취소할 수 있는 제약 추가
    public void cancel() {
        verifyNotYetShipped();
        this.state = OrderState.CANCELED;
    }

	// 이미 출고 되었는지 검증하는 메서드
    private void verifyNotYetShipped() {
        if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
            throw new IllegalArgumentException("Already shipped...");
        }
    }
}
```

이런 식으로 도메인 모델을 점진적으로 만들어나가면 된다.

&nbsp;

# 🎏 엔티티(Entity)와 밸류(Value)

위에서 도출한 모델은 크게 **엔티티(Entity)**와 **밸류(Value)**로 구분할 수 있다. 아래 클래스 다이어그램을 보자.

![](https://velog.velcdn.com/images/rocker_nun/post/f4de6c4e-4b52-4cf8-958c-c317a0469ab5/image.png)


엔티티는 **식별자(Identifier)**를 가진다. 각 엔티티는 구분할 수 있다는 말이다. 위의 `Order` 클래스가 바로 엔티티이며, *“주문번호”* 라는 식별자를 속성으로 가져야 한다.

![](https://velog.velcdn.com/images/rocker_nun/post/7bea5f12-f26a-40e0-8935-d5d28d76ab6b/image.png)


엔티티의 식별자는 바뀌지 않고 고유하기 때문에 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 말할 수 있다. 엔티티를 구현한 클래스는 식별자를 이용해서 `equals()` 메서드와 `hashCode()` 메서드를 구현해야 한다. 아래는 리팩토링한 `Order` 클래스다.

```java
public class Order {
    private String orderNumber;

    @Override
    public boolean equals(Object object) {
        if (this == object) {
            return true;
        }
        if (object == null || object.getClass() != Order.class) {
            return false;
        }
        Order other = (Order) object;
        if (this.orderNumber == null) {
            return false;
        }
        return this.orderNumber.equals(other.orderNumber);
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((this.orderNumber == null) ? 0 : this.orderNumber.hashCode());
        return result;
    }
}
```

&nbsp;

### 🌱 식별자 생성

근데 식별자를 저렇게 항상 필드에 박아 넣어야 할까? 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용 기술에 따라 달라지게 된다. 보통 아래 방법 중 하나를 채택하게 된다.

- 특정 규칙에 따라 생성한다.
- UUID나 Nano ID와 같은 고유 식별자 생성기를 사용한다.
- 값을 직접 입력한다.
- 시퀀스나 DB의 자동 증가 컬럼과 같은 일련번호를 사용한다.

주문번호처럼 사용자가 직접 확인하는 식별자는 현재 시각, 일련번호, 난수 등을 조합해서 의미 있는 형식으로 만들기도 한다. 반면 내부 기술 식별자는 UUID, Nano ID, DB 시퀀스, 자동 증가 컬럼 등을 사용할 수 있다. 어떤 방식을 선택할지는 도메인의 요구사항과 사용하는 기술에 따라 달라진다.


추가로, 식별자를 DB가 생성하는 경우에는 객체 생성 시점에는 식별자가 없을 수 있다. 이 상태에서 `equals()`와 `hashCode()`를 식별자만으로 구현하면 주의가 필요하다. 그래서 도메인에서 중요한 식별자는 가능하면 객체 생성 시점에 함께 생성하거나, 별도의 식별자 타입으로 다루는 방법도 고려할 수 있다.

&nbsp;

이제 `ShippingInfo` 클래스를 살펴보자.

```java
public class ShippingInfo {
    // 받는 사람
    private String receiverName;
    private String receiverPhoneNumber;
    
    // 주소
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipcode;
    
    ... 
}
```

보다시피 받는 사람과 주소에 대한 데이터가 존재한다. 주석처럼 나누어 놓은 이유는 `receiverName`와 `receiverPhoneNumber`는 분명 다른 데이터지만 개념적으로는 *“받는 사람”* 을 의미하고, 밑에 `shippingAddress`와 `shippingZipcode` 필드도 개념적으로는 *“주소”* 를 의미하기 때문이다.

&nbsp;

이처럼 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다. 따라서 “받는 사람” 과 “주소” 를 각각 도메인 개념으로 표현하기 위해 `Receiver` 클래스와 `Address` 클래스로 나눠서 설계해줄 필요가 있다. 아래 코드를 보자.

```java
// 받는 사람
public class Receiver {
    private String name;
    private String phoneNumber;

    public Receiver(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

    public String getName() {
        return name;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }
}

// 주소
public class Address {
    private String address1;
    private String address2;
    private String zipcode;

    public Address(String address1, String address2, String zipcode) {
        this.address1 = address1;
        this.address2 = address2;
        this.zipcode = zipcode;
    }

    public String getAddress1() {
        return address1;
    }

    public String getAddress2() {
        return address2;
    }

    public String getZipcode() {
        return zipcode;
    }
}
```

&nbsp;

이제 기존의 `ShippingInfo` 클래스에서의 받는 사람에 관한 데이터를 `Receiver`, 주소에 관한 데이터는 `Address` 밸류 타입을 사용해서 보다 명확하게 표현할 수 있게 된 것이다. 밸류 타입을 도입함으로써  `ShippingInfo` 클래스는 아래와 같이 리팩토링 된다.

```java
public class ShippingInfo {
    private Receiver receiver;
    private Address address;
    
    ... 
}
```

&nbsp;

이처럼 밸류 타입은 의미를 명확하게 표현하기 위해 사용한다. 다음으로 `OrderLine`을 살펴보도록 하자. `price`와 `amount`는 개념적으로는 금액을 의미하기 때문에 별도로 `Money` 라는 밸류 타입을 설계해서 추가했다.

```java
public class OrderLine {
	private Product product;
	private Money price;
	private int quantity;
	private Money amount;
		
		... 
}
```

&nbsp;

밸류 타입의 또 다른 장점은 해당 밸류 타입만의 기능을 추가할 수 있다는 점이다. `Money` 타입에서 금액 계산 같은 로직을 추가할 수 있다.

```java
public class Money {
    private final int value;

    public Money(int value) {
        this.value = value;
    }
    
    ... 
    
    // 금액 추가 기능
    public Money add(Money money) {
        return new Money(this.value + money.value);
    }
    
    // 금액 곱셈 기능
    public Money multiply(int multiplier) {
        return new Money(this.value * multiplier);
    }
}
```

&nbsp;

이제 아래의 리팩토링된 `OrderLine` 클래스를 보면 금액 계산이라는 의미가 첨가되어 코드 가독성이 향상된 것을 볼 수 있다.

```java
public class OrderLine {
    private final Product product;
    private final Money price;
    private final int quantity;
    private final Money amount;
    
    public OrderLine(Product product, Money price, int quantity) {
        this.product = product;
        this.price = price;
        this.quantity = quantity;
        this.amount = calculateAmount();
    }
    
    private Money calculateAmount() {
        return price.multiply(quantity);
    }
    
    
    public Money getAmount() {
        ...
    }
    
    ...
}
```

&nbsp;

### 🤔 왜 객체를 새로 생성하지?

근데 `Money` 밸류 타입에서 연산을 수행한 후에 그 연산 결과를 바탕으로 새로운 인스턴스를 생성하고 있다. 금액이라는 개념은 바뀔 수 없기 때문에 그런가? 생각해보면 내가 들고 있는 만원이 값을 다르게 세팅한다고 5만원이 되지는 않는다. 아무튼 `Money` 처럼 데이터 변경 기능을 제공하지 않는 타입을 **불변(Immutable)**하다고 표현한다.

이렇게 불변으로 처리하는 이유 중 가장 중요한 것은 바로 안전한 코드를 작성하기 위함이다. 막말로 `setter`를 도입해서 기존 객체의 값을 변경했다고 치자. 이미 해당 금액이 반영된 `OrderLine`이 생성됐는데 이후에 값이 변경되면 기존 `OrderLine`에 있는 금액도 변경되는 참사가 발생할 것이다.

따라서 밸류 타입에는 `setter`를 두지 않는 것이 좋다. 값이 바뀌어야 한다면 기존 객체를 수정하는 대신 새로운 밸류 객체를 생성하는 방식이 안전하다. 엔티티 역시 무분별한 `setter`보다는 `cancel()`, `changeShippingInfo()`처럼 의미 있는 도메인 메서드를 통해 상태를 변경하는 것이 좋다.

&nbsp;

# 🤪 도메인 용어와 유비쿼터스 언어

코드를 작성할 때 도메인에서 사용하는 용어는 아주 중요하다. 아래 주문 상태 코드를 보자.

```java
public enum OrderState {
	STEP1, STEP2, STEP3, STEP4, STEP5, STEP6
}
```

아니, STEP이 뭔데? 몇 단계가 어떤 상태라는거지? 상태에 대해 명확하게 알지 못 하니까 아래와 같은 의도를 알기 어려운 코드를 작성할 가능성이 높다.

```java
public class Order {
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        verifyStep1OrStep2();
        setShippingInfo(newShippingInfo);
    }
    
    private void verifyStep1OrStep2() {
        if (state != OrderState.STEP1 && state != OrderState.STEP2) {
            throw new IllegalArgumentException("Already shipped...");
        }
    }
}
```

&nbsp;

일단 그냥 봐도 `verifyStep1OrStep2()` 메서드가 무슨 검증을 하는지 감이 안 온다. `STEP1`과 `STEP2`가 각각 *“결제 대기 중”*, *“상품 준비 중”* 상태를 의미하는 것을 알아야 한다. 따라서 아래 코드처럼 도메인 용어를 사용해서 주문 상태를 구현하면 위와 같은 불필요한 변환 과정이 필요없다.

```java
public enum OrderState {
    PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED;
}
```

&nbsp;

보다시피 각 상태가 어떤 의미인지 직관적으로 파악이 가능해서 코드를 분석하고 이해하는 시간을 대폭 줄여준다. 도메인 주도 설계에서 언어의 중요성은 몇번을 강조해도 모자르다. 해당 도메인에 관련된 모든 이해관계자들은 공통의 언어를 만들고 이를 대화, 문서, 도메인 모델, 코드, 테스트 등 모든 곳에서 그 언어를 사용한다. 시간이 지나면서 도메인에 대한 이해가 깊어지면 해당 내용을 더 잘 표현할 수 있는 용어를 찾아내서 다시 공통의 언어로 만들어 다 같이 사용하기도 한다.