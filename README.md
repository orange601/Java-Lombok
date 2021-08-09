# Lombok-Issue
Lombok - is로 시작하는 변수명 사라짐


## IS로 시작하는 변수명 사라짐 ##
````java
@Data
public class TestDTO {
    private boolean isSuccess;
}
````
"success": true 로 표기됨


## 해결방법 ##
````java
@Data
public class TestDTO {
    private boolean isSuccess;
    public boolean getIsSuccess() {
        return isSuccess;
    }
}
````

isSuccess함수 
