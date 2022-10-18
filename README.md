# Lombok #
- 실무에서 Lombok 잘 사용하기

## @Data 지양 ##
- 너무 많은 어노테이션(기능)을 포함한다.
- @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor
- 그에 따른 부작용도 많다.

1. 무분별한 Setter 남용
    -  Setter는 의도가 분명하지 않고, 객체를 언제든지 변경할 수 있는 상태가 되어서 객체의 안전성을 보장받기 힘들다.

2. ToString으로 인한 양방향 연관관계시 순환 참조 문제
    - 무한 순환 참조가 발생. Json으로 직렬화 하는 가정에서 발생하는 문제와 동일한 이유

## @NoArgsConstructor 접근 권한 최소화 ##
- 파라미터가 없는 기본 생성자를 생성

1. JPA는 **프록시** 생성을 위해 기본 생성자를 반드시 하나를 생성해야 한다.
2. 이때 접근 권한이 protected 이면 된다. 굳이 외부에서 생성을 열어 둘 필요가 없다.
3. 아무런 값도 갖지 않는 의미 없는 객체의 생성을 막게 된다.
4. 무분별한 객체 생성에 대해 한번 더 체크할 수 있다.

#### 잠깐! 프록시(가짜엔티티)란? ####
> 만약 회원 엔티티만 출력하는 사용하는 경우 em.find()로 회원 엔티티를 조회할 때 회원과 연관된 팀 엔티티까지 데이터베이스에서 함께 조회해 두는 것은 효율적이지 않다.
> JPA는 이런 문제를 해결하려고 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공하는데 이것을 지연 로딩이라 한다.
> 지연 로딩 기능을 사용하려면 실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체가 필요한데 이것을 프록시 객체라 한다.

## @Builder ##
- 의미있는 객체 생성

1.기본값 설정
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

2. 생성자에 @Builder 사용 권장
    - 필요한 데이터만 설정할 수 있다. 

3. @Builder 안전하게 생성하기
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

4. 클래스 레벨에서 @Builder 사용 금지
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


## @Setter 사용금지 ##
- Setter는 의도가 분명하지 않고, 객체를 언제든지 변경할 수 있는 상태가 되어서 객체의 안전성을 보장받기 힘들다.

## @EqualsAndHashCode ##
- 자바 bean에서 동등성 비교를 위해 equals와 hashcode 메소드를 오버라이딩해서 사용하는데, 
- @EqualsAndHashCode어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있다.   
: 두 객체의 내부의 값이 같은지 숫자로 확인하는 값은 hashcode()이다.   
: 같은 객체인지 확인하는 메소드는 equals()이다.   
[출처](https://velog.io/@gloom/Lombok-Data%EC%9D%98-EqualsAndHashCode%EC%9D%B4-%EB%AD%90%ED%95%98%EB%8A%94-%EC%95%A0%EC%9D%BC%EA%B9%8C)

- 무분별한 @EqualsAndHashCode 사용 자제
[권남-참고](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)

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

