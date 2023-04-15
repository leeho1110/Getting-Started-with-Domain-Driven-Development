# 5장: 스프링 데이터 JPA를 이용한 조회 기능

- 스프링 데이터 JPA 조회 기능을 사용할 때 `org.springframework.data.jpa.domain` 에서 제공하는 `Specification<T>` 인터페이스로 검색 조건을 명시할 수 있다.
    - T는 JPA 엔티티를 의미하며 Predicate를 반환하는 메서드 시그니처를 제공한다.
        - 만약 ID 일치 조건을 만들고 싶다면 아래와 같이 구현할 수 있다.
        
        ```java
        public class OrdererIdSpec implements Specification<OrderSummary> {
        
            private String ordererId;
        
            public OrdererIdSpec(String ordererId) {
                this.ordererId = ordererId;
            }
        
            @Override
            public Predicate toPredicate(Root<OrderSummary> root,
                                         CriteriaQuery<?> query,
                                         CriteriaBuilder cb) {
                return cb.equal(root.get(OrderSummary_.ordererId), ordererId);
            }
        }
        ```
        
    - 다중 조건이 필요한 경우 `Specification<T>` 인터페이스에서 디폴트 메서드로 제공하는 `and`, `or` 을 활용할 수 있다.
        
        ```java
        public interface Specification<T> extends Serializable {
        		...
        		default Specification<T> and(@Nullable Specification<T> other) {
        		    return SpecificationComposition.composed(this, other, CriteriaBuilder::and);
        		}
        
        		default Specification<T> or(@Nullable Specification<T> other) {
        		    return SpecificationComposition.composed(this, other, CriteriaBuilder::or);
        		}
        }
        ```
        
    - 만약 더 많은 스펙들을 조합해야 한다면 스펙 빌더 클래스를 사용할 수 있다.
