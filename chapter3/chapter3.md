# 🏰 애그리거트

온라인 쇼핑몰 시스템을 상위 수준 개념을 이용해서 바라보면 아래와 같이 전체 모델들 간의 관계를 이해할 수 있다.

![](https://velog.velcdn.com/images/rocker_nun/post/07a7f2b9-3e3f-4d8a-a76d-589cd2aaa7f8/image.png)


위와 같은 상위 수준 모델 간의 관계를 제대로 이해하지 않고 개별 객체들 간의 관계만을 보고 전체 모델의 관계를 파악하기는 매우 어렵고, 코드를 변경하고 확장하는 것이 매우 힘들어진다. 이를 위해 상위 수준에서 모델을 관찰할 수 있게 해주는 **애그리거트(Aggregate)**가 등장한다. 도메인 규칙에 따라 함께 일관성을 유지해야 하는 객체들을 하나의 애그리거트 경계로 묶어서 상위 수준에서 도메인 모델 간의 관계를 파악하는 것이다.

![](https://velog.velcdn.com/images/rocker_nun/post/7c08746d-bca9-495e-8ed3-44747c5d2e64/image.png)


위의 그림처럼 애그리거트는 전체 모델 간의 관계를 이해할 수 있도록 도와줄 뿐만 아니라, 일관성을 관리하는 기준도 될 수 있다. 그리고 관련된 모델을 하나로 모았기 때문에 한 애그리거트에 속하는 객체는 유사하거나 동일한 라이프 사이클을 갖는다. 그리고 그림을 보면 알 수 있듯이 한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다. 경계를 명확히 해서 복잡한 도메인을 단순한 구조로 만드는 것이 애그리거트의 임무이기 때문에 어찌 보면 당연하다.

이 경계를 어떻게 결정하는지에 대한 답은 도메인 규칙과 요구사항으로부터 찾아야 한다. 일단 도메인 규칙에 따라 함께 생성되고 변경되는 구성요소는 한 애그리거트에 속할 가능성이 높다. 그리고 주의할 점은 *“A가 B를 갖는다”* 와 같은 요구사항을 보고 A와 B는 무조건 하나의 애그리거트에 속할 것이라고 속단하는 것이다. 아래 상품과 리뷰 예시를 보자.

상품 상세 페이지에 들어가면 보통 리뷰가 달려 있으니 상품과 리뷰는 하나의 애그리거트에 포함시키는 것이 이상하다고 느껴지지 않는다. 하지만 분명 상품과 리뷰는 함께 생성되지도, 변경되지도 않는다. 게다가 변경 주체도 상품은 관리자가 변경하고, 리뷰는 고객이 변경 주체이기 때문에 하나의 애그리거트에 포함시키는 것은 무리가 있다.

![](https://velog.velcdn.com/images/rocker_nun/post/78460e31-38da-4795-9ba6-a770c6445d0f/image.png)

&nbsp;

# 👨🏻‍🔧 애그리거트 루트

도메인 규칙을 지키기 위해서는 하나의 애그리거트에 속한 여러 객체들이 모두 정상 상태를 유지해야 한다. 따라서 애그리거트 전체를 컨트롤할 수 있는 관리자가 필요한데, 이 책임을 지는 것이 바로 애그리거트의 **루트 엔티티(Root Entity)**다.

루트 엔티티의 임무는 애그리거트의 일관성이 깨지지 않도록 하는 것이다. 이를 위해 루트 엔티티는 애그리거트가 제공해야 할 도메인 기능을 구현한다. 불필요한 중복을 피하고 루트 엔티티를 통해서만 도메인 로직을 구현하게 만들기 위해서는 `setter` 메서드를 외부에서 접근할 수 없도록 만들고, 밸류 타입은 불변으로 구현해야 한다.

그리고 루트 엔티티는 애그리거트 내부의 다른 객체를 조합해서 기능을 완성한다. 아래 `Order` 루트 엔티티 코드와 `Member` 루트 엔티티 코드를 살펴보자.

```java
public class Order {
    private Money totalAmount;
    private List<OrderLine> orderLines;
    
    private void calculateTotalAmount() {
        int sum = orderLines.stream()
                .mapToInt(ol -> ol.getPrice() * ol.getQuantity())
                .sum();
        this.totalAmount = new Money(sum);
    }
}
```

```java
public class Member {
    private Password password;
    
    public void changePassword(Password currentPassword, Password newPassword) {
        if (!password.match(currentPassword)) {
            throw new PasswordNotMatchException();
        }
        this.password = new Password(newPassword);
    }
}
```

`Order`는 총 주문 금액을 구하기 위해 `OrderLine` 목록을 사용하고, `Member`는 비밀번호를 변경하기 위해 `Password` 밸류 타입에 비밀번호가 일치하는지 확인하고 있다. 추가로 루트 엔티티는 구성요소의 필드만 참조하는 것이 아니라 기능 실행을 위임하기도 한다.

&nbsp;

## 🚧 트랜잭션 범위

트랜잭션의 범위는 작을수록 좋다. 예를 들어 1개의 테이블을 수정하면 락의 대상이 그 테이블의 1개 행일 뿐이지만, 3개의 테이블을 수정한다면 락의 대상이 많아질 수밖에 없다. 이는 동시에 처리할 수 있는 트랜잭션 개수가 줄어든다는 것을 의미하고 전체적인 성능을 떨어뜨린다.

이와 마찬가지로 한 번에 수정하는 애그리거트 개수가 많으면 많아질수록 전체 처리량이 떨어지기 때문에 한 트랜잭션에서는 한 개의 애그리거트만 수정해야 한다. 다시 말하자면, 애그리거트 내부에서 다른 애그리거트의 상태를 변경하는 기능을 되도록 두지 말자는 것이다. 아래 코드를 보자.

```java
public class Order {
    private Orderer orderer;
    
    public void shipTo(ShippingInfo newShippingInfo, boolean useNewShippingAddressAsMemberAddress) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);  // 주문 애그리거트의 배송지 정보를 변경
        if (useNewShippingAddressAsMemberAddress) {
		        // 회원의 주소를 변경된 배송지로 설정...
            orderer.getMember().changeAddress(newShippingInfo.getAddress());
        }
    }
    
    ...
}
```

위의 코드는 주문 애그리거트에서 회원 애그리거트 내부 상태까지 변경하고 있는 상태다. `Order` 루트 엔티티나 `Member` 루트 엔티티는 각각 주문, 회원 애그리거트 내부의 일관성을 책임져야 하는데 `Order` 루트 엔티티가 회원 애그리거트 내부 일관성을 해치고 있는 것이다. 하지만 상황에 따라 한 트랜잭션으로 2개 이상의 애그리거트를 수정해야 한다면 위의 코드처럼 직접 수정하는 것보다 응용 서비스에서 두 애그리거트를 수정하도록 구현해야 한다.

```java
public class ChangeOrderService {
    private OrderRepository orderRepository;
    private MemberRepository memberRepository;

    @Transactional
    public void changeShippingInfo(
            OrderId orderId,
            ShippingInfo newShippingInfo,
            boolean useNewShippingAddressAsMemberAddress
    ) {
        Order order = orderRepository.findById(orderId);
        if (order == null) {
            throw new OrderNotFoundException();
        }
        order.shipTo(newShippingInfo);
        
        if (useNewShippingAddressAsMemberAddress) {
            Member member = findMember(order.getOrderer());
            member.changeAddress(newShippingInfo.getAddress());
        }
    }
    
    ...
}
```

정리하면, 한 트랜잭션에서 한 개의 애그리거트를 변경하는 것이 가장 좋지만, 아래의 경우에는 한 트랜잭션에서 2개 이상의 애그리거트를 변경하는 것을 고려할 수도 있다.

- **팀 표준**: 팀이나 조직의 표준에 따라 사용자 유스케이스와 관련된 응용 서비스의 기능을 한 트랜잭션으로 실행해야 하는 경우가 있다.
- **기술 제약**: 기술적으로 이벤트 방식을 도입할 수 없는 경우 한 트랜잭션에서 다수의 애그리거트를 수정해서 일관성을 처리해야 한다.
- **UI 구현의 편리**: 운영자의 편리함을 위해 주문 목록 화면에서 여러 주문의 상태를 한 번에 변경하고 싶을 것이다. 이 경우 한 트랜잭션에서 여러 주문 애그리거트의 상태를 변경해야 한다.

&nbsp;

# 💾 리포지토리와 애그리거트

객체의 영속성을 처리하는 리포지토리도 애그리거트 단위로 존재한다. 새로운 애그리거트를 만들면 저장소에 애그리거트를 영속화하고 애그리거트를 사용하려면 저장소에서 애그리거트를 읽어 와야 하기 때문에 리포지토리는 기본적으로 아래 2개의 메서드를 제공한다.

- `save()`: 애그리거트를 저장
- `findById()`: ID로 애그리거트를 조회

알다시피 리포지토리를 구현하는 기술들은 정말 다양하다. 어떤 기술을 채택하느냐에 따라 리포지토리 구현 방식도 달라진다. 그리고 리포지토리 구현체는 애그리거트 루트뿐만 아니라 애그리거트에 속한 구성요소까지 함께 저장하고 조회해야 한다. 만약 `Order` 애그리거트 루트와 그와 관련된 테이블이 여러 개가 있다면 루트 엔티티뿐만 아니라 나머지 애그리거트에 속한 모든 구성요소에 매핑된 테이블에 데이터를 저장해야 한다.

```java
// 애그리거트 전체를 영속화
orderRepository.save(order);

// 완전한 주문 애그리거트를 조회
Order order = orderRepository.findById(orderId);
```

&nbsp;

# 🆔 ID를 이용한 애그리거트 참조

아까 봤다시피 하나의 애그리거트도 다른 애그리거트를 참조할 수 있다. 더 정확히 말하면, 다른 애그리거트의 루트 엔티티를 참조하는 것이다. 앞에서 본 예시 코드처럼 애그리거트 간의 참조는 필드를 통해 쉽게 구현할 수 있다.

![](https://velog.velcdn.com/images/rocker_nun/post/9266b8b1-1081-4cb8-b6c0-98b3771d206d/image.png)


이처럼 애그리거트 간의 참조를 구현하는 것은 쉽지만 몇 가지 문제들이 발생할 수 있다는 것을 유념해야 한다.

- 편한 탐색 오용
- 성능에 대한 고민
- 확장 어려움

&nbsp;

일단 첫 번째로, 한 애그리거트가 관리하는 범위는 자기 자신으로 한정하는 것이 가장 좋다고 했다. 하지만 구현이 워낙 편리하기 때문에 다른 애그리거트를 수정하고자 하는 유혹에 빠지기 쉽다. 또한 애그리거트 간의 결합도가 높아져 애그리거트의 변경을 어렵게 만든다.

두 번째 문제는 성능이다. 애그리거트를 객체로 직접 참조하면 JPA 같은 ORM을 사용할 때 지연 로딩과 즉시 로딩 중 무엇을 선택할지 고민해야 한다. 잘못 선택하면 불필요한 쿼리가 많이 실행되거나, 반대로 필요하지 않은 객체까지 한 번에 조회하는 문제가 생길 수 있다.

마지막으로, 확장에 대한 문제다. 서비스가 성장하고 사용자 수가 늘면 자연스럽게 부하를 분산하기 위해 하위 도메인별로 시스템을 분리해야 할 것이다. 이 과정에서 하위 도메인마다 각기 다른 DBMS를 사용하거나 아예 다른 데이터 저장소를 사용할 수도 있다. 이렇게 되면 다른 애그리거트 루트를 참조하기 위해 JPA와 같은 단일 기술은 사용할 수 없게 된다.

&nbsp;

위와 같은 문제점들을 한 번에 해결할 수 있는 방법이 바로 ID를 이용해서 다른 애그리거트를 참조하는 방법이다. 아래 다이어그램을 보자.

![](https://velog.velcdn.com/images/rocker_nun/post/7f7987b6-0296-4be0-96a3-164604474bed/image.png)

보다시피 ID 참조를 사용하면 모든 객체가 참조로 연결되지 않고 한 애그리거트에 속한 객체들만 참조로 연결되기 때문에 모델의 복잡도를 낮추고 각 애그리거트의 응집도도 높여줄 수 있다. 추가로 구현 복잡도도 낮아진다. 이제 직접적으로 참조하지 않기 때문에 JPA를 예로 들면 애그리거트 간의 참조를 지연 로딩으로 할지 즉시 로딩으로 할지 더 이상 고민하지 않아도 된다. 참조하는 애그리거트가 필요하다면 그냥 응용 서비스에서 ID로 로딩하면 된다.

```java
public class ChangeOrderService {
    
    ...

    @Transactional
    public void changeShippingInfo(
            OrderId orderId,
            ShippingInfo newShippingInfo,
            boolean useNewShippingAddressAsMemberAddress
    ) {
        Order order = orderRepository.findById(orderId);
        if (order == null) {
            throw new OrderNotFoundException();
        }
        order.changeShippingInfo(newShippingInfo);
        
        if (useNewShippingAddressAsMemberAddress) {
            // ID를 이용해서 참조하는 애그리거트를 구한다.
            Member member = memberRepository.findById(
                    order.getOrderer().getMemberId()
            );
            member.changeAddress(newShippingInfo.getAddress());
        }
    }
    
    ...
}
```

&nbsp;

## ⚙️ 조회 성능

하지만 다른 애그리거트를 ID로 참조하게 되면 여러 애그리거트를 읽을 때 조회 속도가 저하될 수도 있다. 예를 들어 주문 목록을 보여주려면 상품 애그리거트와 회원 애그리거트를 같이 읽어 와야 할 것이다. 아래 코드를 보자.

```java
...

Member member = memberRepository.findById(ordererId);

List<Order> orders = orderRepository.findByOrderer(ordererId);
List<OrderView> dtos = orders.stream()
        .map(order -> {
            ProductId productId = order.getOrderLines().get(0).getProductId();

            // 각 주문마다 첫 번째 주문 상품 정보 로딩을 위한 쿼리 실행
            Product product = productRepository.findById(productId);

            return new OrderView(order, member, product);
        })
        .collect(Collectors.toList());
        
 ...
```

만약 주문이 10개면 주문을 읽어오기 위한 1번의 쿼리와 주문별로 각 상품을 읽어오기 위한 10번의 쿼리를 실행해야 한다. 이게 그 유명한 **N + 1 조회 문제**다. 이는 더 많은 쿼리를 날리기 때문에 당연히 성능을 저하시킬 수밖에 없다.

이 문제를 해결하려면 조회 목적에 맞는 전용 쿼리를 사용하는 것이 좋다. 객체 참조와 즉시 로딩으로 해결할 수도 있지만, 이는 애그리거트 간 결합도를 높이고 필요하지 않은 데이터까지 함께 조회할 위험이 있다.

따라서 목록 화면처럼 여러 애그리거트의 데이터를 한 번에 보여줘야 하는 경우에는 조회 전용 DAO나 조회 전용 모델을 두고 조인 쿼리로 필요한 데이터만 읽어오는 방식이 더 적합하다. 예를 들어 데이터 조회를 위한 별도 DAO를 만들고 DAO의 조회 메서드에서 조인을 이용해 1번의 쿼리로 필요한 데이터를 로딩하면 된다. 아래 특정 사용자의 주문 내역을 보여주는 코드를 보자.

```java
@Repository
public class JpaOrderViewDao implements OrderViewDao {
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public List<OrderView> selectByOrderer(String ordererId) {
        String selectQuery =
                "select new com.myshop.order.application.dto.OrderView(o, m, p) " + 
                        "from Order o join o.orderLines ol, Member m, Product p " +
                        "where o.orderer.memberId.id = :ordererId " +
                        "and o.orderer.memberId = m.id " +
                        "and index(ol) = 0 " +
                        "and ol.productId = p.id " +
                        "order by o.number.number desc";
        TypedQuery<OrderView> query =
                em.createQuery(selectQuery, OrderView.class);
        query.setParameter("ordererId", ordererId);
        return query.getResultList();
    }
}
```

이 JPQL은 `Order` 애그리거트와 `Member` 애그리거트, `Product` 애그리거트를 조인으로 조회해서 1번의 쿼리로 로딩한다. 따라서 즉시 로딩이든 지연 로딩이든 상관없이 조회 화면에서 필요한 애그리거트 데이터를 1번의 쿼리로 로딩할 수 있는 것이다.

&nbsp;

# ⛓️ 애그리거트 간 집합 연관

이제 애그리거트 간의 일대다 관계, 다대다 관계에 대해 알아보자. 이 두 연관은 **컬렉션(Collection)**을 이용한 연관이다. 일단 애그리거트 간의 일대다 관계로, 하나의 카테고리와 그에 연관된 상품을 값으로 갖는 컬렉션을 필드로 아래와 같이 정의할 수 있다.

```java
public class Category {
    
    private Set<Product> products;
    
    ...
}
```

&nbsp;

근데 개념적으로 존재하는 애그리거트 간의 일대다 관계를 실제 구현에 반영하는 것이 요구사항을 충족하는 것과는 상관없을 때가 있다. 특정 카테고리에 속한 상품 목록을 보여주는 요구사항을 생각해보자. 보통 목록과 관련된 요구사항은 한 번에 모든 정보를 보여주기보다는 페이징 기법을 이용해 요소들을 나눠서 보여준다. 이 기능을 카테고리 입장에서 일대다 관계를 이용해서 구현하면 아래와 같이 코드를 작성할 수 있다.

```java
public class Category {
    
    private Set<Product> products;
    
    public List<Product> getProducts(int page, int size) {
        List<Product> sortedProducts = sortById(products);
        return sortedProducts.subList((page - 1) * size, page * size);
    }
    
    ...
}
```

하지만 이 코드를 실제 DB와 연동한다면 카테고리에 있는 모든 상품들이 다 딸려 나온다. 그러면 성능에 아주 심각한 문제가 생긴다. 따라서 개념적으로는 애그리거트 간에 일대다 연관이 있더라도 실제 구현에 반영하지는 않는다. 카테고리에 속한 상품 목록이 필요하다면, 카테고리 애그리거트가 상품 컬렉션을 직접 들고 있기보다 상품 조회용 리포지토리나 조회 전용 DAO를 통해 `categoryId` 조건으로 페이징 조회하는 편이 더 적절하다.

다대다 연관도 일대다 연관과 마찬가지로, 실제 요구사항을 고려해서 구현에 포함할지를 결정해야 한다. 카테고리와 상품을 예로 들면, 개념적으로는 상품과 카테고리 사이에 양방향 다대다 관계가 존재할 수 있다. 하지만 실제 구현에서는 요구사항에 필요한 방향만 반영하면 된다.

예를 들어 상품 상세 화면에서 상품이 속한 카테고리 정보만 필요하다면, 상품에서 카테고리 ID 목록을 참조하거나 별도 조회 쿼리로 상품과 카테고리 관계를 읽어오는 방식으로 충분할 수 있다. 반대로 특정 카테고리에 속한 상품 목록이 필요하다면, 카테고리가 상품 컬렉션을 직접 들고 있기보다 상품 조회용 DAO나 조회 전용 쿼리를 통해 페이징해서 조회하는 편이 더 적절하다.

&nbsp;

# 🏭 애그리거트를 팩토리로 사용하기

고객이 특정 상점을 신고해서 해당 상점이 더 이상 물건을 등록하지 못하는 상황을 생각해보자. 상품 등록 기능을 구현한 응용 서비스는 상점 계정이 차단 상태가 아닌 경우에만 상품을 등록할 수 있도록 로직을 구현해야 할 것이다. 아래 코드를 보자.

```java
public class RegisterProductService {
    
    ...
    
    public ProductId registerNewProduct(NewProductRequest request) {
        Store store = storeRepository.findById(request.getStoreId());
        checkNull(store);
        if (store.isBlocked()) {
            throw new StoreBlockedException();
        }
        ProductId productId = productRepository.nextId();
        Product product = new Product(productId, store.getId(), ...);
        productRepository.save(product);
        return productId;
    }
    
    ...
}
```

겉으로 보기에는 문제가 없어 보이지만, `Store`가 상품을 생성할 수 있는지 여부를 검사하고 상품을 생성하는 로직은 분명 논리적으로 하나의 도메인 기능인데 현재 응용 서비스에서 구현하고 있는 것이다. 해당 도메인 기능을 별도의 도메인 서비스나 팩토리 클래스를 만들 수도 있지만 아래와 같이 `Store` 애그리거트에 구현하는 방법도 있다.

```java
public class Store {
    ...
    
    public Product createProduct(ProductId newProductId, ...) {
        if (isBlocked()) {
            throw new StoreBlockedException();
        }
        return new Product(newProductId, getId(), ...);
    }
}
```

`Store` 애그리거트의 `createProduct()` 메서드는 `Product` 애그리거트를 생성하는 팩토리 역할을 하면서도 중요한 도메인 로직을 구현하고 있다. 이제 응용 서비스에서 해당 팩토리 기능을 사용하기만 하면 된다.

```java
public class RegisterProductService {
    ...
    
    public ProductId registerNewProduct(NewProductRequest request) {
        Store store = storeRepository.findById(request.getStoreId());
        checkNull(store);
        ProductId productId = productRepository.nextId();
        Product product = store.createProduct(productId, ...);
        productRepository.save(product);
        return productId;
    }
    
    ...
}
```

이제 상품을 생성할 수 있는지 여부를 검사하는 도메인 로직에 변경이 일어나더라도 응용 서비스는 전혀 영향을 받지 않기 때문에 도메인의 응집도가 높아진 것이다. 이게 바로 애그리거트를 팩토리로 사용할 때의 장점이다. 따라서 앞으로 애그리거트가 갖고 있는 데이터를 이용해서 다른 애그리거트를 생성해야 한다면 이런 식으로 애그리거트에 팩토리 메서드를 구현하는 것을 고려하도록 하자.