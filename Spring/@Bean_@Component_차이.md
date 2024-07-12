`@Bean`, `@Component`는 둘다 Bean을 생성하는 점에서 같은 목적을 갖지만 차이가 있다.

### 차이1. 선언 위치

Component는 클래스 레벨에 선언되고, Bean은 메소드 레벨에 선언된다.

> **@Bean 어노테이션**
>
![Untitled (2).png](..%2F..%2F..%2FDownloads%2FUntitled%20%282%29.png)

Target이 메소드로 지정돼 있다.

> **@Component 어노테이션**
>
![Untitled (3).png](..%2F..%2F..%2FDownloads%2FUntitled%20%283%29.png)

Target이 타입으로 지정되어 있으므로 클래스 레벨에 선언된다.

### 차이2. 사용 용도

> **@Bean 어노테이션**
>

@Bean 어노테이션은 복잡한 구성 로직이 필요하거나 외부 라이브러리 객체를 스프링 컨텍스트에 통합해야 하는 경우에 주로 사용된다.

특정 설정이 필요한 라이브러리, API 클라이언트, 데이터베이스 연결 등 복잡한 객체의 생성과 관리에 사용된다. 개발자가 직접 빈 생성 로직을 제어할 수 있게 해주어 스프링 애플리케이션에 쉽게 통합할 수 있게 해준다.

```java
@Configuration
public Class AppConfig {

	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
    dataSource.setUsername("username");
    dataSource.setPassword("password");
    return dataSource;
	}
}
```

이 메소드의 반환 값이 Spring 컨테이너에 의해 관리되는 객체임을 나타낸다.

> **@Component 어노테이션**
>

클래스 레벨에서 사용되며, 스프링은 이 어노테이션이 적용된 클래스의 인스턴스를 자동으로 빈으로 등록한다. 흔히 사용하는 `@Controller`, `@Service`, `@Repository` 어노테이션들에도 @Component 어노테이션이 포함되어 있다.