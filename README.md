# Lombok #
- Lombok 사용

## @NoArgsConstructor 접근 권한 최소화 ##
- JPA는 프록시를 생성을 위해 기본 생성자를 반드시 하나를 생성해야한다. 

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

