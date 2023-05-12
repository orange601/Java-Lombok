# Lombok #
- Lombok 잘 사용하기

## @Data 지양 ##
- 너무 많은 어노테이션(기능)을 포함한다.
- @ToString, @EqualsAndHashCode, @Setter, @Getter, @RequiredArgsConstructor
- 그에 따른 부작용도 많다.

### 1. @Tostring 무한순환참조 ###
- 양방향 연관관계시 무한 순환 참조가 발생

````java
@Getter @Setter @ToString
public class Parent {
	private String address;
	private Child child;
}
````
````java
@Getter @Setter @ToString
public class Child {
	private String name;
	private Parent parent;
}
````
````java
@Test
void test() {
	Parent parent = new Parent();
	parent.setAddress("address");
	
	Child child = new Child();
	child.setName("name");
	
	parent.setChild(child);
	child.setParent(parent);
	
	System.out.println(parent);
	System.out.println(child);
}
````

- 메모리 에러 발생
````java
java.lang.StackOverflowError
````

- 해결방법
````java
// ToString에 exclude추가해서 연관관계를 제외한다.
@Getter @Setter @ToString(exclude = "child")
public class Parent {
	private String address;
	private Child child;
}
````

### 2. @EqualsAndHashCode ###
- 자바 bean에서 동등성 비교를 위해 equals와 hashcode 메소드를 오버라이딩해서 사용하는데, 
- @EqualsAndHashCode어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있다.   
: 두 객체의 내부의 값이 같은지 숫자로 확인하는 값은 hashcode()이다.   
: 같은 객체인지 확인하는 메소드는 equals()이다.   
[출처](https://velog.io/@gloom/Lombok-Data%EC%9D%98-EqualsAndHashCode%EC%9D%B4-%EB%AD%90%ED%95%98%EB%8A%94-%EC%95%A0%EC%9D%BC%EA%B9%8C)

- 무분별한 @EqualsAndHashCode 사용 자제
[권남-참고](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)

### 3. @Setter 무분별한 Setter 남용 ###
- Setter는 의도가 분명하지 않고, 객체를 언제든지 변경할 수 있는 상태가 되어서 객체의 안전성을 보장받기 힘들다.
- JPA에서 Setter는 Insert(Update) 쿼리를 의미한다. Setter는 어디에서든지 호출될 수 있으므로 특정 컬럼이 어디에서 수정되었는지 파악하기 힘들고, 컬럼을 수정하기 전 검증이 필요한 경우 이를 무시할 수 있기에 무결성이 훼손되는 등의 부작용이 발생할 수 있다.
- **@Builder** 사용으로 대체한다.

## @Builder ##


### 1. 필드 기본값 설정 ###
    - @Builder.Default 사용   
    
````java
@Data
@Builder
public class User {
  private String name;
  @Builder.Default private int age = 19;
  private int num = 19; // @Builder.Default 사용하지 않았기 때문에 기본값은 0이다.
}
````

### 2. 생성자에 @Builder 사용 권장 ###
    - 필요한 데이터만 설정
    - 객체를 생성할 때 인자 값의 순서가 상관없다.
    - 하나의 생성자로 대체가 가능 ( 여러 생성자를 두지 않고 하나의 생성자를 통해서 객체 생성이 가능 )
    - 물론 생성자 대신 @Builder를 사용하면 안좋은 단점이 존재한다.

#### 생성자 대신에 빌더 사용시 안좋은 점 ####
- 새로운 필드를 추가해야될때 실수로 빌더를 호출하는 코드에 새로운 필드를 추가하는 것을 깜빡하면 컴파일러는 이에 대해 경고하지 않는다.
- 그러면 런타임에 유효성 검증 로직이 동작해서 예외가 발생하게 된다.
- 하지만 빌더 대신에 생성자를 직접 사용하면 새로운 필드를 추가하거나 삭제할 때마다 컴파일 에러를 따라서 바로 에러를 잡을 수 있다.

### 3. @Builder 안전하게 생성하기 ###

````java
@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Account {

  @NotEmpty @Column(name = "bank_name", nullable = false)
  private String bankName;

  @NotEmpty @Column(name = "account_number", nullable = false)
  private String accountNumber;

  @NotEmpty @Column(name = "account_holder", nullable = false)
  private String accountHolder;

  // 불안전한 객채 생성 패턴
  @Builder
  public Account(String bankName, String accountNumber, String accountHolder) {
    this.bankName = bankName;
    this.accountNumber = accountNumber;
    this.accountHolder = accountHolder;
  }

  // 안전한 객채 생성 패턴
  @Builder
  public Account(String bankName, String accountNumber, String accountHolder) {
    Assert.hasText(bankName, "bankName must not be empty");
    Assert.hasText(accountNumber, "accountNumber must not be empty");
    Assert.hasText(accountHolder, "accountHolder must not be empty");

    this.bankName = bankName;
    this.accountNumber = accountNumber;
    this.accountHolder = accountHolder;
  }
}
````
[출처](https://cheese10yun.github.io/spring-builder-pattern/)

### 4. Builder 이름으로 책임을 부여 하자 ###
주문에 대해 신용카드취소, 계좌 기반 환불이 있다고 가정하자.   
= 신용카드 취소 / 계좌 기반 환불에 대해 각각 정의하여 @Builder설계

````java
public class Refund {
   private Long id;
   private Account account;
   private CreditCard creditCard;
   private Order order;

   @Builder(builderMethodName = "of", builderClassName = "of")
   private Refund(Account account, CreditCard creditCard, Order order) {
       this.account = account;
       this.creditCard = creditCard;
       this.order = order;
   }

   // 계좌 기반 환불
   @Builder(builderClassName = "byAccountBuilder", builderMethodName = "byAccountBuilder")
   public static Refund byAccount(Account account, Order order) {
       Assert.notNull(account, "account must not be null");
       Assert.notNull(order, "order must not be null");

       return Refund.of().account(account).order(order).build();
   }

   // 신용카드 환불
   @Builder(builderClassName = "byCreditBuilder", builderMethodName = "byCreditBuilder")
   public static Refund byCredit(CreditCard creditCard, Order order) {
       Assert.notNull(creditCard, "creditCard must not be null");
       Assert.notNull(order, "order must not be null");

       return Refund.of().creditCard(creditCard).order(order).build();
   }
}
````

출처: https://velog.io/@dahye4321/%EC%8A%A4%ED%94%84%EB%A7%81-%EA%B0%80%EC%9D%B4%EB%93%9C-2

### 5. 클래스 레벨에서 @Builder 사용 금지 ###
    - @NoArgsConstructor를 함께 쓰면 오류가 발생
    - 이를 해결하기 위해 모든 필드를 가지는 생성자를 만드는 @AllArgsConstructor 사용해야 한다.
    - 하지만 @AllArgsConstructor의 문제점이 아래와 같이 존재한다.

## @AllArgsConstructor 사용금지 ##
- 해당 객체 내에 있는 모든 변수들을 인수로 받는 생성자를 만들어내는 어노테이션

1. 변수(인스턴스 멤버)의 선언 순서에 영향을 받는다.
2. 변수의 순서를 바꾸면 생성자의 입력 값 순서도 바뀌게 되어 검출되지 않는 치명적인 오류를 발생시킬 수 있다.

````java
@AllArgsConstructor
public static class Order {
    private long cancelPrice;
    private long orderPrice;
}
 
// 취소금액 5,000원, 주문금액 10,000원
Order order = new Order(5000L, 10000L); 
````

````java
// 위의 필드를 cancelPrice와 orderPrice 순서를 아래와 같이 변경한다.
@AllArgsConstructor
public static class Order {
    private long orderPrice;
    private long cancelPrice;
}
````
> IDE가 제공해주는 리팩토링은 전혀 작동하지 않고, lombok이 개발자도 인식하지 못하는 사이에 생성자의 파라미터 순서를 필드 선언 순서에 맞춰 orderPrice,cancelPrice로 바꿔버린다.
> 게다가 이 두 필드는 동일한 Type 이라서 기존 생성자호출 코드에서는 인자 순서를 변경하지 않았음에도 어떠한 오류도 발생하지 않는다.

> 이에 의해, 위의 생성자를 호출하는 코드는 아무런 에러없이 잘 작동하는 듯 보이지만 실제로 입력된 값은 바뀌어 들어가게 된다.
````java
// 주문금액 5,000원, 취소금액 10,000원. 취소금액이 주문금액보다 많아짐!
Order order = new Order(5000L, 10000L); // 인자값의 순서 변경 없음
````

**따라서, 직접 만든 생성자에 @Builder 애노테이션을 붙이는 것을 권장한다.**
[출처-권남](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)


## @NoArgsConstructor 접근 권한 최소화 ##
- 파라미터가 없는 기본 생성자를 생성

1. JPA는 **프록시** 생성을 위해 기본 생성자를 반드시 하나를 생성해야 한다.
2. 이때 접근 권한이 protected 이면 된다. 굳이 외부에서 생성을 열어 둘 필요가 없다.
3. 아무런 값도 갖지 않는 의미 없는 객체의 생성을 막게 된다.
4. 무분별한 객체 생성에 대해 한번 더 체크할 수 있다.

````java
import lombok.AccessLevel;

@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class SampleDTO {
	...
}
````

#### 잠깐! 프록시(가짜엔티티)란? ####
> 만약 회원 엔티티만 출력하는 사용하는 경우 em.find()로 회원 엔티티를 조회할 때 회원과 연관된 팀 엔티티까지 데이터베이스에서 함께 조회해 두는 것은 효율적이지 않다.
> JPA는 이런 문제를 해결하려고 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공하는데 이것을 지연 로딩이라 한다.
> 지연 로딩 기능을 사용하려면 실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체가 필요한데 이것을 프록시 객체라 한다.


## lombok.config 설정 ##
- @Data 등 사용을 했을 경우 위험 부담이 있는 어노테이션들은 해당 설정에서 제한 할 수 있다.
- lombok.config 파일을 작성한 뒤 Proejct root path에 위치 시킨다.
````properties
lombok.data.flagUsage= error
````

## @Embeddable ##
- Entity 내부의 값을 더 응집시켜 **객체로 데이터를 표현**한다.
- Lombok기능은 아니지만, Lombok과 같이 사용하면 좋은 Annotation이다.

````java
// @Embeddable 사용하기전
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="user_id")
    private Long id;
    private String name;
    private String phoneNum;
    
    // @Embedded 통해 아래 3개필드는 Address 라는 객체로 묶어서 관리하는게 주소라는 의미를 확실하게 표현 할 수 있다.
    private String zipCode;
    private String address;
    private String addressDetail;
}
````

````java
// @Embeddable 적용 후
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="user_id")
    private Long id;
    private String name;
    private String phoneNum;
    
    @Embedded
    private Address address;
}

@Embeddable
public class Address {
    private String zipCode;
    private String address;
    private String addressDetail;
}
````



## Lombok-Issue ##
- is로 시작하는 변수명 사라짐
````java
// json으로 변경시 "isSuccess"가 아닌 "success" 로 표기됨
@Data
public class TestDTO {
    private boolean isSuccess;
}
````

````java
// 해결방법 // getIsSuccess함수 추가
@Data
public class TestDTO {
    private boolean isSuccess;
    public boolean getIsSuccess() {
        return isSuccess;
    }
}
````
 
