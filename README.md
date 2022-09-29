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

