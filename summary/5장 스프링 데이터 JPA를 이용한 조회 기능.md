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
- 조회 시 자연스럽게 정렬에 대한 이야기가 같이 나온다. 스프링 데이터 JPA는 정렬 옵션을 두 가지 형태로 제공한다. 메서드 이름에 `OrderBy` 키워드 명시를 통한 자동 쿼리 생성과 `Sort` 타입의 파라미터를 전달하는 것이다.
    - `OrderBy` 키워드는 프로퍼티 이름과 `ASC`, `DESC` 를 더한 문자열을 메서드 시그니처에 더해주면 된다.
        - `findByOrdererId**OrderByNumberDescNumberAsc**`
    - `Sort` 타입 역시 프로퍼티를 전달하고 `ascending()`, `descending()` 을 메서드 체이닝 형태로 덧붙히면 된다. 다중 조건이라면 `and()` 메서드를 활용할 수 있다.
        - `Sort sort = Sort.by(”number”).ascending().and(Sort.by(”orderDate”).descending());`
- 페이징 이야기도 빼놓을 수 없다. 페이징은 `Pageable` 타입을 파라미터로 전달하는 방식을 사용한다.
    - 일반적으로 `PageRequest.of()` 정적 팩토리 메서드를 활용해 페이지 번호(몇 페이지)와 페이지당 노출 개수를 표기한다. 정렬이 필요한 경우 `Sort` 타입을 추가로 전달하면 된다.
        
        ```java
        public static PageRequest of(int page, int size) {
            return of(page, size, Sort.unsorted());
        }
        
        public static PageRequest of(int page, int size, Sort sort) {
            return new PageRequest(page, size, sort);
        }
        ```
        
    - 반환 타입에 `Page` 타입을 사용하는 경우 스프링 데이터 JPA는 조회 쿼리와 COUNT 쿼리를 동시에 실행한다. 이를 통해 아래와 같은 추가 정보들을 얻을 수 있다.
        - `getContent()` 를 통해 조회 결과 반환
        - `getTotalElements()` 를 통해 전체 개수를 반환
        - `getTotalPages()` 를 통해 전체 페이지 번호를 반환
        - `getNumber()` 를 통해 현재 페이지 번호 반환
        - `getNumberOfElements()` 를 통해 조회 결과의 개수를 반환
        - `getSize()` 를 통해 페이지 크기를 반환