Repository 단위 테스트를 진행하기 전에,

우선 **Repository Layer라는게 정확히 무엇인지** 알아보고 고로 **Repository Layer에서는 무엇을 단위 테스트 해야하는지** 살펴봤다.

### Repository Layer는 무엇을 하는 계층일까?

**Repository 계층은 도메인을 영속화 하는 계층이다.**

**리포지토리 패턴**은 소프트웨어 개발에 사용되는 설계 패턴으로 도메인 모델 계층과 구현 기술을 분리시키는 것을 의미한다.


데이터 액세스 코드를 추상화하여 구체적인 접근 방식을 가림으로써 깨끗한 아키텍처를 유지할 수 있다.

리포지토리 계층은 도메인과 data source layer 간에 중개자 역할을 수행하게 되는데,

영속성 장치에서 쿼리의 결과로 받아온 데이터를 repository에서는 도메인에서 사용하기 적합하도록 매핑하는 역할을 수행한다.

### Repository에서는 무엇을 테스트 해야 할까?

리포지토리 계층은 도메인 객체를 저장, 검색, 삭제 할 수 있도록 하는 계층이며 영속화를 위해 주로 데이터베이스를 사용한다.

그렇기에 리포지토리 단위 테스트의 목적은 도메인 객체에 대해 저장, 삭제, 조회 기능이 올바르게 작동되는지 확인하는 것이다.

---

### JdbcTemplate 테스트하기

**@Jdbctest**

- JDBC 기반 컴포넌트만 테스트하는 JDBC 테스트를 위한 어노테이션
- auto-configuration이 비활성화 되고, JDBC 테스트와 관련된 configuration만 적용된다.
- 기본적으로 트랜잭션 처리가 됨
- 각 테스트가 종료된 후에 롤백 처리
- 인메모리 DB 사용

테스트를 위해 자동으로 내장된 인메모리 데이터베이스를 구성해주는 테스트용 구성 어노테이션이다.

기본적으로 클래스패스 상에 존재하는 내장 데이터베이스 중 하나를 사용한다. 만약 클래스패스 상에 여러 내장 데이터베이스가 존재한다면 `spring.datasource.embedded-database-connection` 설정에 따라 결정되며, 설정하지 않았다면 스프링부트가 자동으로 판단해 하나를 선택한다.

이는 application.properties에 설정된 H2 데이터베이스 설정을 무시하고 별도의 내장 데이터베이스로 교체될 수 있다는 의미이다. 그렇기에 `@JdbcTest` 어노테이션이 적용된 테스트에서는 예상치 않게 다른 인메모리 데이터베이스를 사용하게 될 가능성이 있다. 이를 특정한 디비로 고정하고 싶다면 `@AutoConfigureTestDatabase` 어노테이션을 사용하여 테스트에 사용할 디비를 명시적으로 설정할 수 있다.

---

### 테스트 코드 작성

StudyRoom 도메인을 저장하는 `StudyRoomRepository` 인터페이스가 존재한다.

```java
public interface StudyRoomRepository {
    StudyRoom save(String roomName, int total, int memberSeqId);
    Optional<StudyRoom> findByRoomId(int roomId);
    void update(int roomId, String roomName, int maxParticipants, int currentParticipants, int managerSequenceId);
    List<StudyRoom> findByActivatedTrue();
}
```

그리고 해당 인터페이스를 구현하는 `StduyRoomRepositoryImpl` 클래스가 존재한다.

```java

@Repository
public class StudyRoomRepositoryImpl implements StudyRoomRepository {
    private final JdbcTemplate jdbcTemplate;

    public StudyRoomRepositoryImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public StudyRoom save(String roomName, int maxParticipants, int memberSeqId) {
        LocalDate createDate = LocalDate.now();

        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("STUDY_ROOM").usingGeneratedKeyColumns("ROOM_ID");

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("ROOM_NAME", roomName);
        parameters.put("MAX_PARTICIPANTS", maxParticipants);
        parameters.put("CURRENT_PARTICIPANTS", 1);
        parameters.put("CREATE_DATE", createDate);
        parameters.put("ACTIVATED", StudyRoom.ActivateStatus.ACTIVATED.getStatusValue());
        parameters.put("MANAGER_SEQ_ID", memberSeqId);

        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));

        return StudyRoom.builder()
                .roomId(key.intValue())
                .roomName(roomName)
                .maxParticipants(maxParticipants)
                .createDate(createDate)
                .activateStatus(StudyRoom.ActivateStatus.ACTIVATED)
                .managerSequenceId(memberSeqId)
                .build();

    }
    
    // 그외 메서드들
    
 }

```

> 테스트 클래스
>

테스트 클래스를 생성해주고 `@JdbcTest` 어노테이션과 데이터베이스 스키마 파일과 테스트 데이터 파일을 위해 `@Sql` 어노테이션을 달아주었다.

```java
@JdbcTest
@Sql({"classpath:schema.sql", "classpath:test-data.sql"})
class StudyRoomRepositoryTest {
 //
 }
```

> 테스트 코드
>

```java
@JdbcTest
@Sql({"classpath:schema.sql", "classpath:test-data.sql"})
class StudyRoomRepositoryTest {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    private StudyRoomRepository studyRoomRepository;

    @BeforeEach
    void setup() {
        this.studyRoomRepository = new StudyRoomRepositoryImpl(jdbcTemplate);
    }

    @Test
    @DisplayName("StudyRoom을 save 하면 StudyRoom 객체가 반환된다.")
    void whenStudyRoomIsSaved_thenReturnsSavedStudyRoom() {
        //given
        String roomName = "test room name";
        int maxParticipants = 10;
        int memberSequenceId = 1;

        //when
        StudyRoom studyRoom = studyRoomRepository.save(roomName, maxParticipants, memberSequenceId);

        //then
        assertNotNull(studyRoom);
        assertEquals(roomName, studyRoom.getRoomName());
        assertEquals(maxParticipants, studyRoom.getMaxParticipants());
        assertEquals(memberSequenceId, studyRoom.getManagerSequenceId());
    }
}
```

### 테스트 코드를 작성하면서 느낀점

이후로 다른 메서드 및 Repository 계층에도 이와 유사한 방식으로 테스트를 진행했다.

테스트 코드를 작성하면서 의문이 생겼던 지점이 있다. 바로 메서드의 파라미터에 관해서다.

지금 `StudyRoomRepository`의 `save()` 메서드를 살펴보면

```java
StudyRoom save(String roomName, int total, int memberSeqId);
```

이렇게 데이터베이스에 저장해야 할 속성을 직접 지정해 parameter로 넣어주고 있다.

리포지토리 패턴이란 앞서 말했듯이 **도메인 모델 계층 - 영속화 기술**을 분리하기 위해 중개자 역할로 리포지토리를 만든것이다.

그리고 이 패턴의 목적은 **“데이터 액세스 방식을 추상화하여 구체적인 접근 방식을 가림으로써 깨끗한 아키텍처를 유지하고 도메인 및 비즈니스 로직에만 집중할 수 있는 것”**이라 였다.

그런데 내가 속성을 지정해서 넘겨준다는 것은 결국 데이터베이스에 해당 속성들이 컬럼으로 필요하다는 것을 알고 넘겨주는 것이 아닌가? 그리고 이러한 생각은 결국 DB 중심적 설계가 아닌가? 😶

사실 절대적인 정답은 없다.. 직접 속성을 지정해서 파라미터로 전달해주는 것이 틀린것은 아니다.

다만, DB 중심적인 설계를 하면 도메인이 변경되는 상황이나 영속화 기술이 변경되는 시점에 유지보수가 굉장히 어려워진다는 답변을 들었다.

또한, 이와 관련하여 Repository에 대한 정의를 더 확실히 하기 위해 조금 더 깊이 있게 들어가서 Repository와 DAO의 차이에 대해서도 알아봤다. 해당 내용은 따로 포스팅 하였다.

[DAO와 Repository의 차이는 무엇일까? (실무에선 어떻게 사용?/차이를 구분할때의 장점)](https://dev-wooni.tistory.com/6)

현재는 리포지토리의 구현체가 데이터베이스에 직접 액세스 하도록 만들고 있지만 엄밀히 말하면 데이터베이스에 직접 액세스 하는 역할을 수행하는 것은 리포지토리가 아니라 DAO 의 역할이다.

그리고 리포지토리는 해당 DAO들을 사용하여 도메인 클래스로 매핑한다.

이는 DDD에서 나오는 빌딩블록과도 연관이 있는데 DDD에서는 리포지토리를 도메인 모듈의 빌딩 블록이라 정의한다. 즉 도메인 모듈을 쌓아 올려서 온전히 한 모듈인 집합으로 만들어주는 역할인데, 그 관점에서는 도메인 객체를 데이터베이스에 저장할때 그 도메인 객체들이 이루는 집합 자체(객체 자체)를 넘긴다.

그래야 리포지토리가 빌딩블록으로서 온전한 책임을 수행할 수 있다.

DDD는 아직 잘 모르는 분야라 미숙하기에 지금은 이정도로 이해하고 넘어가보려 한다.

결론적으로 디비에 어떤 속성들이 필요한지 난 모르겠고 난 그냥 도메인 그대로 넘길게 ! 리포지토리 너네가 알아서 영속화 해! ← 이 방향이 리포지토리 패턴을 도입하는 이유라고 생각한다.

그리고 무엇보다도 속성을 직접 지정해서 넘겨주는것은 내가 개발자로써 해야할 일이 추가적으로 생긴다고 느꼈다. (*이건 개발 효율성과 연결 지을 수 있겠다 ^^..*)

그래서 저장 및 업데이트 같은 경우에는 필요한 속성을 지정해서 parameter에 넘기는것보단 도메인 객체 자체를 넘기는 방향으로 리팩토링 하기로 결정했다.