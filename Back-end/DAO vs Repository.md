# DAO와 Repository의 차이는 무엇일까? (실무에선 어떻게 사용?/차이를 구분할때의 장점)

### DAO(Data Access Object)

**DAO는 데이터 영속성을 추상화한 개념으로, 데이터베이스 테이블 중심의 기본 storage에 가까운 개념이다.**

대부분의 경우 DAO는 데이터베이스 테이블과 일치하므로 까다로운 쿼리를 추상화하여 스토리지에서 데이터를 보다 간단하게 전송하고 검색하는 기능을 수행한다.

> **User 도메인 클래스**
> 

```java
public class User {
	private String name;
	private String email;
	
	//getters and setters
}
```

> **User 도메인에 대하여 CRUD 연산을 수행할 수 있는 UserDao 인터페이스**
> 

```java
public interface UserDao {
    void create(User user);
    User read(Long id);
    void update(User user);
    void delete(String userName);
}
```

> **UserDao 인터페이스를 구현한 UserDaoImpl 클래스**
> 

```java
public class UserDaoImpl implements UserDao {
    private final EntityManager entityManager;
    
    @Override
    public void create(User user) {
        entityManager.persist(user);
    }

    @Override
    public User read(long id) {
        return entityManager.find(User.class, id);
    }

    // ...
}
```

### Repository

Eric Evans의 책 Domain-Driven Desgin에 따르면 리포지토리는 저장, 검색 및 검색 동작을 캡슐화하는 메커니즘이며 이 메커니즘은 **객체의 Collection을 모방**한다.

여기서 Collection이란 자바의 Collection이 아닌 객체의 모음집 정도로 이해하면 좋을것 같다. 

리포지토리도 데이터를 처리하고 DAO와 유사하게 쿼리를 추상화한다. 하지만 앱의 비즈니스 논리에 더 까운 고수준에 위치한다. 

결과적으로 리포지토리는 DAO를 사용하여 데이터베이스에서 데이터를 가져오고 도메인 객체를 채울 수 있다. 또는 도메인 객체에서 데이터를 준비하고 영속화를 위해 DAO를 사용하여 스토리지 시스템으로 보낼 수 있다. 

> **UserRepository interface**
> 

```java
public interface UserRepository {
    User get(Long id);
    void add(User user);
    void update(User user);
    void remove(User user);
}
```

> **UserRepositoryImpl**
> 

UserRepository 인터페이스를 구현한 UserRepositoryImple 클래스이다.

UserDaoImpl을 사용하여 데이터베이스에 데이터를 저장하거나 가져오고 있다. 

```java
public class UserRepositoryImpl implements UserRepository {
    private UserDaoImpl userDaoImpl;
    
    @Override
    public User get(Long id) {
        User user = userDaoImpl.read(id);
        return user;
    }

    @Override
    public void add(User user) {
        userDaoImpl.create(user);
    }

    // ...
}
```

사실 이렇게만 봤을땐 `User` 도메인 클래스가 단순하여 DAO와 Repository는 같은 개념으로 보일 수 있다. 그러나 원론적인 개념으로 들어가면 다르다.

<aside>
💡 DAO는 데이터에 액세스, Repository는 비즈니스 유스케이스를 구현하는 개념이다.

</aside>

해당 문장을 이해하기 위한 예시를 아래에서 살펴보자.

### 여러개의 DAO를 사용한 리포지터리 패턴

> ***DAO는 데이터에 액세스, Repository는 비즈니스 유스케이스를 구현하는 개념이다.***
> 

> **Tweet 클래스**
> 

tweet 정보에 대한 몇몇 속성을 포함하는 Tweet 클래스를 생성해보자.

```java
public class Tweet {
    private String email;
    private String tweetText;    
    private Date dateCreated;

    // getters and setters
}
```

> **TweetDao 인터페이스**
> 

그런 다음 user의 email을 통해 tweet을 fetch 할 수 있는 TweetDao 인터페이스를 만들고 해당 메소드를 TweetDaoImpl에 구현한다.

```java
public interface TweetDao {
    List<Tweet> fetchTweets(String email);    
}

public class TweetDaoImpl implements TweetDao {
    @Override
    public List<Tweet> fetchTweets(String email) {
        List<Tweet> tweets = new ArrayList<Tweet>();
        
        //call Twitter API and prepare Tweet object
        
        return tweets;
    }
}
```

> **UserSocialMedia 클래스**
> 

User 도메인을 확장하여 List<Tweet> 속성을 가질 수 있는 UserSocialMedia 클래스를 만들었다.

```java
public class UserSocialMedia extends User{
	private List<Tweet> tweets;
	
	//getters and setters
}
```

> **UserRepositoryImpl**
> 

이제 tweet 리스트를 가질 수 있는 User 도메인을 제공하기 위해 UserRepositoryImpl을 업그레이드해보겠다.

```java
public class UserRepositoryImpl implements UserRepository {
	private UserDaoImpl userDaoImpl;
	private TweetDaoImpl tweetDaoImpl;
	
	@Override
	public User get(Long id) {
		UserSocialMedia user = (UserSocialMedia) userDaoImpl.read(id);
		
		List<Tweet> tweeets = tweetDaomImpl.fetchTweets(user.getEmail());
		user.setTweets(tweets);
		
		return user;
	}
}
```

해당 구현체에서는 UserDaomImpl을 사용하여 User 데이터를 가져오고, TweetDaoImpl을 사용하여 tweet 리스트를 가져오고 있다. 

그런 다음 두 데이터를 합쳐 UserSocialMedia 도메인 클래스를 제공하고 있다. 

그러므로 리포지토리는 데이터에 접근하기 위해 DAO에 의존하여 다양한 data source에 접근할 수 있는것이다.

---

### 두 개념을 구분하는 것은 오버 엔지니어링? 🤔

Repository와 DAO가 같은 개념이면 왜 이렇게 다른 이름이 붙여진걸까? 하는 의문에서 시작하여 두 개념을 정리하게 되었다. 

결론적으로 Repository와 DAO는 다른 개념이다. 그렇다면 실무에서도 명확히 구분하는건지 궁금해서 질문을 해보았다. 

**🧑‍💻 *:* “실무는 실용성을 많이 따지거든요. *굳이 다르게 할 필요가 있어? 우리한테 이런 엔지니어링이 필요해*? 하면 *아니다*라고 답하는 조직이 더 많을겁니다.”**

### DAO, Repository를 구분 지으면 빛을 발하는 순간

그런데! 이게 진정 빛을 발하는 순간이 있다고 한다. 

Repository를 여러 DAO의 조합으로 구성하고 각 DAO들은 다양한 데이터 소스에 접근해 도메인 모델을 구성하도록 동작할 수 있다. 

가령 RDBMS + NoSQL 조합으로 구성돼있다면 Repository 입장에서는 data source가 어떻게 구성돼있든 도메인 객체를 조회하고 영속화하는 역할만 하면 된다. 

구현체 안에서 어떻게 동작하든 도메인에 영향을 끼치지 않고 도메인을 영속화 하고 Data source + Cache 조합으로 조회 성능을 끌어올리는 등의 구현도 가능하다고 한다.

해당 내용과 관련해선 아래 링크를 살펴보면 좋다.

https://github.com/sirloin-dev/meatplatform-sandbox/blob/main/server/api-core-infra-impl/src/main/kotlin/com/sirloin/sandbox/server/core/domain/user/repository/UserReadonlyRepositoryImpl.kt

---

### 비교 요약

- `DAO`는 데이터 영속화를 추상화한것. `Repository`는 객체의 Collection을 추상화한것.
- `DAO`는 저수준의 개념으로 스토리지 시스템과 가깝다. `Repository`는 고수준의 개념으로 도메인 객체와 가깝다.
- `DAO`는 데이터에 접근하고 매핑하는 역할을 한다. `Repository`는 도메인과 데이터 액세스 계층 사이에 존재하여 도메인 객체를 준비하는 복잡성을 추상화한다.
- `Repository` 는 DAO를 사용하여 스토리지에 접근할 수 있다.
