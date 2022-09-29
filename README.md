# Lombok #
- Lombok 사용

## @NoArgsConstructor 접근 권한 최소화 ##
- JPA는 프록시를 생성을 위해 기본 생성자를 반드시 하나를 생성해야한다.
- 이때 접근 권한이 protected 이면 된다. 굳이 외부에서 생성을 열어 둘 필요가 없다.

프록시(가짜엔티티):
> 만약 회원 엔티티만 출력하는 사용하는 경우 em.find()로 회원 엔티티를 조회할 때 회원과 연관된 팀 엔티티까지 데이터베이스에서 함께 조회해 두는 것은 효율적이지 않다.
> JPA는 이런 문제를 해결하려고 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공하는데 이것을 지연 로딩이라 한다.
> 지연 로딩 기능을 사용하려면 실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체가 필요한데 이것을 프록시 객체라 한다.

## @Data 지양 ##
1. 무분별한 Setter 남용

2. ToString으로 인한 양방향 연관관계시 순환 참조 문제

## lombok.config 설정 ##
- @Data 등 사용을 했을 경우 위험 부담이 있는 어노테이션들은 해당 설정에서 제한 할 수 있다.
- lombok.config 파일을 작성한 뒤 Proejct root path에 위치 시킨다.
````properties
lombok.data.flagUsage= error
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
// 해결방법 // isSuccess함수 추가
@Data
public class TestDTO {
    private boolean isSuccess;
    public boolean getIsSuccess() {
        return isSuccess;
    }
}
````

