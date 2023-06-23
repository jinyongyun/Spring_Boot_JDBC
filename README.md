# Spring_Boot_JDBC
JDBC를 이용하여 간단한 CRUD 학습 &amp; DB 커넥션 풀 적용
<div class="page-header-icon page-header-icon-with-cover"><img class="icon" src="https://www.notion.so/icons/cafe_gray.svg"/></div><h1 class="page-title">JDBC 프로그래밍</h1><p class="page-description"></p><table class="properties"><tbody></tbody></table></header><div class="page-body"><h2 id="1f8fe9ce-3b3a-4afe-bc60-8b6fad0067ff" class="">JDBC를 이용하여 간단한 CRUD 학습</h2><p id="e5bbe0fe-c87b-4f41-9a3d-4e4f03d60261" class="">JDBC(Java Database Connectivity)</p><ul id="78ec9385-2868-48e7-a570-8f9bfa1916d4" class="bulleted-list"><li style="list-style-type:disc">자바 애플리케이션에서 DB 프로그래밍을 할 수 있도록 도와주는 표준 인터페이스</li></ul><ul id="05a315bf-e2d7-48de-962c-f963c5068f2e" class="bulleted-list"><li style="list-style-type:disc">JDBC 인터페이스들을 구현한 구현체들은 각 데이터베이스 벤더 사들이 제공하고 있고 이를 JDBC 드라이버라고 한다.</li></ul><p id="4a1e5362-7a86-428b-9e9e-321aa349b20c" class="">
</p><h2 id="417ed5d4-e5ca-4753-a860-ca3010b608b0" class="">DB 커넥션 풀 적용</h2><p id="f35a533e-d140-4cea-b07f-576a14e105fc" class="">DBCP(Database Connection Pool)</p><ul id="e62a63eb-ba05-4274-bce9-6f77e78f5bc2" class="bulleted-list"><li style="list-style-type:disc">미리 일정량의 DB 커넥션을 생성해서 풀에 저장해 두고 있다가 HTTP 요청에 따라 필요할 때 풀에서 커넥션을 가져다 사용하는 기법 </li></ul><ul id="9b27fbbb-583d-4e87-a66c-04b863b6d14c" class="bulleted-list"><li style="list-style-type:disc">참고로 Spring Boot 2.0 부터는 디폴트 커넥션 풀로 HikariCP 사용</li></ul><p id="01695fda-9d16-4e41-a908-d9edd143886e" class="">
</p><p id="7940da44-7ad9-4e51-88a7-25d1baae3854" class="">커넥션 풀 사용 시 유의사항</p><ul id="8e1a8169-85b6-4e29-8b6d-bafc8d1c6dc1" class="bulleted-list"><li style="list-style-type:disc">커넥션의 사용 주체는 WAS 스레드이므로 커넥션 개수는 WAS 스레드 수와 함께 고려해야 함</li></ul><ul id="e01d92c5-061d-4c9e-bc00-c0583f7e6828" class="bulleted-list"><li style="list-style-type:disc">커넥션 수를 크게 설정하면 메모리 소모가 큰 대신 동시 접속자 수가 많아지더라도 사용자 대기 시간이 상대적으로 줄어들게 되고, 반대로 커넥션 개수를 작게 설정하면 메모리 소모는 적은 대신 그만큼 대기시간이 길어질 수 있다. 따라서 적정량의 커넥션 객체를 생성해 두어야 한다.</li></ul><p id="73ce0074-1599-4055-b798-c921899e73c6" class="">
</p><p id="952d6d0e-dd6a-4d94-8337-defce910a57e" class="">커넥션 풀을 적용하기 위해서는 자바에서 DataSource를 사용할텐데</p><p id="7e85177d-3091-4a85-b2fb-cc2506018227" class="">DataSource는 커넥션을 획득하기 위한 표준 인터페이스 </p><p id="e04cb191-f92f-4e07-beab-6611d4d086c3" class="">우리는 HikariCP의 DataSource를 사용할 예정이다.</p><p id="7173b63f-48d7-4928-a3e7-87ee0f73a4e1" class="">
</p><p id="81fbb24b-a4b4-4b7d-8fb0-a2e721b3dcca" class="">
</p><h1 id="c4d702b6-88b1-4849-8645-210a10205f95" class="">JDBC 실습</h1><p id="0cddd48e-71d7-492b-969d-7975957cce78" class="">dependency는 다음과 같이 설정한다.</p><pre id="849fea9d-c3ca-435b-94e6-cc2ec4502a64" class="code"><code>dependencies {

    implementation(&#x27;com.zaxxer:HikariCP:5.0.1&#x27;);
    implementation(&#x27;org.springframework:spring-jdbc:5.3.22&#x27;);

    testImplementation &#x27;org.junit.jupiter:junit-jupiter-api:5.8.1&#x27;
    testRuntimeOnly &#x27;org.junit.jupiter:junit-jupiter-engine:5.8.1&#x27;

    testImplementation(&#x27;org.assertj:assertj-core:3.22.0&#x27;); //이건 테스트 코드 가독성을 위해
    testImplementation(&#x27;com.h2database:h2-mvstore:2.1.214&#x27;); //데이터베이스
}</code></pre><p id="7bf325d7-f698-4490-ba05-7e87746e8ce5" class="">test&gt;java&gt;org.example&gt;UserDaoTest 클래스 생성</p><p id="f64fa380-c425-4d0a-9874-d34df17a2f41" class="">이것도 TDD 방식으로 진행!</p><p id="c86d92f9-29f6-427e-88e3-2bc2b31765c8" class="">
</p><p id="cce675f9-fb89-4105-9f8f-5e9cbe59a8c0" class="">command + N 으로 SetUp Method를 만들자</p><p id="917d7560-cb7a-4042-bedd-a7690ded4dc7" class="">@BeforeEach라는 annotaion이 붙은 SetUp메소드는 테스트 코드를 실행하기에 앞서 수행해야하는 작업이 있다면 이곳에서 코드를 작성하면 된다. SetUp메소드 안에는 ResourceDatabasePopulator라는 클래스를 생성한다.(DB에 초기 데이터를 적재하는 스크립트 로더이다) </p><p id="834c0e80-0064-4f2d-a0f1-e7fdc61ebee1" class="">ResourceDatabasePopulator populator = new ResourceDatabasePopulator();</p><p id="39465669-87a1-4f16-b54c-ce0ca93507f2" class="">이렇게 생성하여, </p><p id="d6fc48ce-d846-4b7a-8e7b-21b24bbad3d1" class="">populator.addScript(new ClassPathResource(”db_schema.sql”);</p><p id="abc0f52b-9e84-4333-914c-e27f428cea55" class="">이런 식으로 만들어 준다면, db_schema.sql이라는 스크립트 파일을 classPath에서 읽어와서 추가해 줄 수 있다.</p><p id="edbe480b-2a16-4bde-bb46-18472df26cf0" class="">그리고 DatabasePopulatorUtils.execute(populator, ConnectionManager.getDataSource());</p><p id="14c88d7b-df66-4af5-8c69-f4b589ea8cd6" class="">를 하면, 해당 스크립트를 실행해주고 원래 DatabasePopulatorUtils.execute 메소드에서는 두 번째 인자로 DataSource를 받는데 지금은 없는 ConnectionManager라는 클래스에서 데이터 소스를 받아올 것이다.</p><pre id="6cff8c7a-a2bd-4382-a79a-61b1cdc3201d" class="code"><code>package org.example;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.core.io.ClassPathResource;
import org.springframework.jdbc.datasource.init.DatabasePopulatorUtils;
import org.springframework.jdbc.datasource.init.ResourceDatabasePopulator;

import java.sql.SQLException;

import static org.assertj.core.api.Assertions.assertThat;


public class UserDaoTest {
    @BeforeEach
    void setUp() {
        ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
        populator.addScript(new ClassPathResource(&quot;db_schema.sql&quot;));
        DatabasePopulatorUtils.execute(populator, ConnectionManager.getDataSource());
    }
}</code></pre><p id="eadcaa51-7731-47af-8e79-f773f70aa483" class="">그렇다면 ConnectionManager를 구현해보자</p><p id="cca460b7-646b-47b4-9616-4d089798df58" class="">일단 ConnectionManager에 getDataSource부터 구현해야한다. (그래야 저 보기만 해도 불안한 빨간줄을 없앨 수 있다)</p><p id="9c66e560-b1b1-420d-9702-713b713cf8be" class="">
</p><p id="7592d0da-e036-49b6-b214-b3e299ed0380" class="">우리가 Dependency에서 HikariCP를 사용했는데, 바로 이 getDataSource에서 HikariCP를 이용해서 데이터소스를 받아오도록 한다. </p><pre id="8d90466f-9a22-4965-ba2b-db4954195b1e" class="code"><code>package org.example;

import com.zaxxer.hikari.HikariDataSource;

import javax.sql.DataSource;

public class ConnectionManager {
    public static DataSource getDataSource() {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setDriverClassName(&quot;org.h2.Driver&quot;);
        hikariDataSource.setJdbcUrl(&quot;jdbc:h2:mem://localhost/~/jdbc-practice;MODE=MySQL;DB_CLOSE_DELAY=-1&quot;);
        hikariDataSource.setUsername(&quot;sa&quot;);
        hikariDataSource.setPassword(&quot;&quot;);

        return hikariDataSource;
    }
}</code></pre><p id="7ad6246c-194f-47e0-b6cd-a01fedb6db0b" class="">
</p><p id="57e98476-5e74-456d-933f-671d041fc073" class="">ConnectionManager를 다 완성했다면</p><p id="b546c6f5-23eb-4a41-9f61-d5d7ff194d3a" class="">이전 populator.addScript(new ClassPathResource(”db_schema.sql”));</p><p id="c52af10f-f5c3-460c-a1f7-9a5723c74b26" class="">에서 사용되었던 db_schema.sql를 만들어보자</p><p id="3f4cc119-f16a-4a03-9118-33c87c456cf3" class="">table은 간단하게 구성했다.</p><pre id="a7b2e10d-ef8a-4f40-b030-10c1fdd99790" class="code"><code>DROP TABLE IF EXISTS USERS;

CREATE TABLE USERS (
     userId         varchar(12)         NOT NULL,
     password       varchar(12)         NOT NULL,
     name           varchar(12)         NOT NULL,
     email          varchar(50),

     PRIMARY KEY                (userId)
);</code></pre><p id="dbb4f2fb-3e48-4e5f-83d2-a07323844674" class="">아이디, 비밀번호, 이름, 이메일만 있는 간단한 테이블</p><p id="49e15134-ad80-4859-a2ad-0dd3ba948fce" class="">
</p><p id="9cb37833-0076-4601-9525-f92506840e80" class="">즉 @BeforeEach 는 테스트 코드 실행하기 전에 수행되는 코드라고 했기 때문에 해당 sql을 읽어서 테스트 코드 실행하기 전에 수행해준다. 즉 테이블을 먼저 만들어주는 것이다.</p><p id="98559f87-5f6c-4b0a-89b0-09beef9cc04f" class="">
</p><p id="736bc4fb-8839-45e5-948c-1eaa7e99b08a" class="">sql까지 읽어왔으니, 테스트 코드를 작성해준다.</p><pre id="387dbe0c-ab6e-417c-957b-d3074b1c2078" class="code"><code>package org.example;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.core.io.ClassPathResource;
import org.springframework.jdbc.datasource.init.DatabasePopulatorUtils;
import org.springframework.jdbc.datasource.init.ResourceDatabasePopulator;

import java.sql.SQLException;

import static org.assertj.core.api.Assertions.assertThat;


public class UserDaoTest {
    @BeforeEach
    void setUp() {
        ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
        populator.addScript(new ClassPathResource(&quot;db_schema.sql&quot;));
        DatabasePopulatorUtils.execute(populator, ConnectionManager.getDataSource());
    }

    //Dao(data access object) 디비 작업할 때 UserDao에게 위임
    @Test
    void createTest() throws SQLException {
        UserDao userDao = new UserDao(); //UserDao 객체 생성

        userDao.create(new User(&quot;wizard&quot;, &quot;password&quot;, &quot;name&quot;, &quot;email&quot;)); // UserDao에게 데이터베이스에 해당 정보 저장하라고 요청

        User user = userDao.findByUserId(&quot;wizard&quot;); // 해당 아이디에 해당하는 정보를 조회
        assertThat(user).isEqualTo(new User(&quot;wizard&quot;, &quot;password&quot;, &quot;name&quot;, &quot;email&quot;)); // 이 정보는 위에꺼랑 같아?
    }
}</code></pre><p id="ecca4256-c063-4ee9-9d6c-6c87df0316f5" class="">UserDao에서 Dao라는 의미는 Data access object라는 뜻이다. 즉 디비 작업을 수행할 때 UserDao에게 위임을 할 예정이다. </p><p id="86698b5d-6bd6-4ee2-941c-8c3e36196248" class="">userDao.create() 메서드를 수행할 때 해당 값을 전달하면 userDao는 디비에 해당하는 유저 정보를 저장하는 역할을 수행할 것이다. (User 객체를 통해 넘겨주었다)</p><p id="d0857cfe-cbf3-4526-b38b-a9e0a6e29800" class="">디비에 저장은 이렇게 했고, userDao.findByUserId를 통해 해당하는 아이디와 일치하는 유저 정보를 조회해서 user 객체로 만들었다.(유저 클래스도 만들어야한다)</p><p id="941fed37-8a29-47ff-b3ec-259026c0d168" class="">최종적으로는 조회해온 유저 정보가 우리가 만들어낸 유저 정보와 같은 지 테스트 하는 코드를 완성했다.</p><p id="502278c9-c741-4a72-acdd-aa3fd61638d3" class="">
</p><p id="f1bb57b7-24ce-48e3-897e-38a2509b395e" class="">마지막으로는 테스트 코드에서 필요했던 클래스들을 만들어내면 된다.</p><p id="14ebdba9-7f28-4631-9474-26e6cd8b1a68" class="">User클래스와 UserDao 클래스를 만들어낸다. </p><p id="3099eb49-81d2-4c3b-bff2-84b6af4daa9b" class="">userDao에서 create 메서드 만들어낸다.</p><pre id="fc876751-e7d5-4ae2-8a10-0ac574adabab" class="code"><code>package org.example;

import java.util.Objects;

public class User {
    private final String userId;
    private final String name;
    private final String password;
    private final String email;

    public User(String userId, String password, String name, String email) {
        this.userId = userId;
        this.password = password;
        this.name = name;
        this.email = email;

    }

    public String getUserId() {
        return userId;
    }

    public String getName() {
        return name;
    }

    public String getPassword() {
        return password;
    }

    public String getEmail() {
        return email;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(userId, user.userId) &amp;&amp; Objects.equals(name, user.name) &amp;&amp; Objects.equals(password, user.password) &amp;&amp; Objects.equals(email, user.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(userId, name, password, email);
    }
}</code></pre><p id="91612de4-ebaf-442b-9a07-7c9235f959aa" class="">User를 통해 객체와 객체를 비교하기 때문에 equals 그리고 hashCode를 만들어줘야 함에 주의해라!</p><p id="e5669681-3580-4303-8a06-3eaa999d1eef" class="">
</p><pre id="f4166ddf-f388-4de4-be64-548705e66c5c" class="code"><code>import java.sql.*;

public class UserDao {

    private Connection getConnection() {
        String url = &quot;jdbc:h2:mem://localhost/~/jdbc-practice;MODE=MySQL;DB_CLOSE_DELAY=-1&quot;;
        String id = &quot;sa&quot;;
        String pw = &quot;&quot;;

        try {
            Class.forName(&quot;org.h2.Driver&quot;);
            return DriverManager.getConnection(url, id, pw);
        }catch(Exception ex){
            return null;
        }
    }
    public void create(User user) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;

        try{
            con = getConnection();
            String sql = &quot;INSERT INTO USERS VALUES (?, ?, ?, ?,)&quot;;
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, user.getUserId());
            pstmt.setString(2,user.getPassword());
            pstmt.setString(3,user.getName());
            pstmt.setString(4, user.getEmail());

            pstmt.executeUpdate();
;        } finally {
             if(pstmt != null){
                 pstmt.close();
             }

             if(con != null){
                 con.close();
             }
        }
    }



    public User findByUserId(String userid) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            con = getConnection();
            String sql = &quot;SELECT userid, password, name, email FROM USERS WHERE userid = ?&quot;;
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1,userid);


            rs = pstmt.executeQuery();
            User user = null;
            if(rs.next()){
                user = new User(
                        rs.getString(&quot;userId&quot;),
                        rs.getString(&quot;password&quot;),
                        rs.getString(&quot;name&quot;),
                        rs.getString(&quot;email&quot;));

            }
            return user;
        } finally {
            if(rs != null){
                rs.close();
            }
            if(pstmt != null){
                pstmt.close();
            }
            if(con != null){
                con.close();
            }
        }
    }
}</code></pre><p id="1187737a-8a7f-4d24-96a3-95c276d90ae5" class="">UserDao 클래스는 JDBC 프로그래밍을 아주 날 것의 형태로 만들어봤다.</p><p id="f16da48c-e52d-4c7c-9e00-90f8ac7a30e8" class="">코드 리팩토링은 다음 시간에 할 것 같다.</p><p id="6c9dd076-5d2c-4b8b-9280-2fc71251a2a4" class="">사실 JDBC는 최동완 교수님 DB수업 때 1차 프로젝트로 많이 해보았기에 익숙했다.</p><p id="b1fa3938-fb8a-4d5c-ba93-34c2ec9a8224" class="">create 메서드에 Connection과 PreparedStatement를 생성해주고 커넥션을 얻어오는 메소드를 별도로 만들어준다.  </p><h3 id="1ea1b147-478c-4e71-8a57-cbe215b8348d" class=""><strong>1.JDBC 드라이버 로딩</strong></h3><ul id="b79f8731-d2ac-4580-9052-25925d1eb669" class="bulleted-list"><li style="list-style-type:disc">MySQL의 JDBC Driver Class를 로딩합니다.</li></ul><ul id="b2068d9d-cb35-48cf-bcf0-0de9766991ba" class="bulleted-list"><li style="list-style-type:disc"><strong>Class.forName(“driver”)</strong>을 이용해서 Driver Class를 로딩하면 객체가 생성되고, DriverManager에 등록됩니다.</li></ul><ul id="bab189a1-4df9-42c5-8670-d95fd2e2b6ea" class="bulleted-list"><li style="list-style-type:disc">ex) Class.forName(“com.mysql.jdbc.Driver”)</li></ul><ul id="887e61ba-2224-4bea-9f2e-5ab382bc5014" class="bulleted-list"><li style="list-style-type:disc">Driver 클래스를 찾지 못할 경우, <strong>ClassNotFoundException</strong> 예외가 발생 합니다. (try _catch로 해결)</li></ul><h3 id="0b267438-9aea-4fd8-be5e-aae099c805ec" class=""><strong>2. Connection 생성</strong></h3><ul id="9d76947e-31d7-4dd4-b7f2-364172f52c04" class="bulleted-list"><li style="list-style-type:disc">Connection - 데이터베이스와 연결하는 객체입니다.</li></ul><ul id="a20145c8-e290-4a5f-8416-8432ce3883d6" class="bulleted-list"><li style="list-style-type:disc"><strong>DriverManager.getConnection(연결문자열, DB_ID, DB_PW)</strong> 으로 Connection 객체를 생성합니다.</li></ul><ul id="2d0fb93a-c425-4909-85d0-f6300dfc00c4" class="bulleted-list"><li style="list-style-type:disc">연결문자열(Connection String) - <strong>“jdbc:Driver 종류://IP:포트번호/DB명”</strong></li></ul><ul id="e7523d75-52e7-4c6a-b743-50dd698e4df6" class="bulleted-list"><li style="list-style-type:disc">ex) jdbc:mysql://localhost:3306/test_db</li></ul><ul id="eb5121b9-1902-4479-b80e-beda02044354" class="bulleted-list"><li style="list-style-type:disc">DB_ID : MySQL 아이디</li></ul><ul id="bfe4c718-0a85-48ce-9942-33ccf2edfc06" class="bulleted-list"><li style="list-style-type:disc">DB_PW : MySQL 비밀번호</li></ul><p id="535f8f34-0901-45d8-bb43-64e386cfc479" class="">
</p><p id="6565b0b3-a15d-4852-b6df-69d7dae9502d" class="">커넥션을 생성한 다음에는 만들어둔 테이블에 User 정보를 넣어준다.</p><p id="18801a37-3d1d-4139-98cd-73cba8d98d99" class="">
</p><figure id="510d5bb7-6d4b-4ded-a50c-8c44eeac3c6e" class="image"><a href="JDBC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20b60122a0dfdb472b8ba86c57f230a79a/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-06-23_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_10.37.31.png"><img style="width:3584px" src="JDBC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20b60122a0dfdb472b8ba86c57f230a79a/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-06-23_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_10.37.31.png"/></a></figure><h1 id="1ec0d9f1-5e5a-4235-82f6-d99688baaf97" class="">JDBC코드 리팩토링 및 DB 커넥션 풀 적용</h1><p id="8b76de72-e229-4b8c-8c1d-8e511678c06e" class="">이전에 작성했던 JDBC 코드를 리팩토링 하고 DB 커넥션 풀을 적용해보겠다.</p><p id="aa8e12a5-b481-47af-9912-c823c0ef22f5" class="">getConnection()메서드를 커넥션 매니저 쪽으로 이동시킨다.(static으로 만들어주는 것도 잊지 말기)</p><pre id="fb0124ea-e905-442d-8a42-0d29b8db223c" class="code"><code>package org.example;

import com.zaxxer.hikari.HikariDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.DriverManager;

public class ConnectionManager {

    private static Connection getConnection() {
        String url = &quot;jdbc:h2:mem://localhost/~/jdbc-practice;MODE=MySQL;DB_CLOSE_DELAY=-1&quot;;
        String id = &quot;sa&quot;;
        String pw = &quot;&quot;;

        try {
            Class.forName(&quot;org.h2.Driver&quot;);
            return DriverManager.getConnection(url, id, pw);
        }catch(Exception ex){
            return null;
        }
    }
    public static DataSource getDataSource() {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setDriverClassName(&quot;org.h2.Driver&quot;);
        hikariDataSource.setJdbcUrl(&quot;jdbc:h2:mem://localhost/~/jdbc-practice;MODE=MySQL;DB_CLOSE_DELAY=-1&quot;);
        hikariDataSource.setUsername(&quot;sa&quot;);
        hikariDataSource.setPassword(&quot;&quot;);

        return hikariDataSource;
    }
}</code></pre><p id="46e954c6-a28e-42f8-97dd-3926c9a538e6" class="">이제는 커넥션 풀을 적용해 볼 시간이다.</p><p id="ba09f381-e6c7-4db1-81dd-c0a6ab38f492" class="">기존의 getConnection() 내용을 삭제해주고 getDataSource()로부터 커넥션을 받아올 수 있도록 해준다.</p><p id="47a72cf3-7f08-4e16-8b28-7f027a0e1bff" class="">그리고 getDataSource() 내부에 있던 hikariDataSource관련된 함수들의 인자는 전부 상수로 주는 것이 좋다.</p><p id="2172c3b8-8e60-4e3b-95b3-9403bf7588b9" class="">그리고 hikariDataSource에는 setMaximumPoolSize()라는 메서드가 있는데(커넥션을 맥시멈 몇까지 설정할 것인가…) 이것도 설정해준다.</p><p id="6b5a033a-8585-430b-99f9-e5b5d0050b58" class=""><code>private static final DataSource </code><code><em>ds</em></code><code>;</code></p><p id="f81c5aa0-6f1d-455d-bba7-7d95df8d02ad" class="">를 두고 데이터 소스는 하나여야 하기 때문에, static 에서 초기화 해준다.</p><p id="9cf5724f-0ed9-499b-a36a-1ffdf12f8dc4" class="">→커넥션 풀을 하나만 가지도록 설정!! (기존 getDataSource 메서드 삭제)</p><pre id="3383b397-c403-462a-88f9-ade8ae78f1bc" class="code"><code>package org.example;

import com.zaxxer.hikari.HikariDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionManager {

    public static final String DB_DRIVER = &quot;org.h2.Driver&quot;;
    public static final String DB_URL = &quot;jdbc:h2:mem://localhost/~/jdbc-practice;MODE=MySQL;DB_CLOSE_DELAY=-1&quot;;
    public static final int MAX_POOL_SIZE = 40;

    private static final DataSource ds;

    static {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setDriverClassName(DB_DRIVER);
        hikariDataSource.setJdbcUrl(DB_URL);
        hikariDataSource.setUsername(&quot;sa&quot;);
        hikariDataSource.setPassword(&quot;&quot;);
        hikariDataSource.setMaximumPoolSize(MAX_POOL_SIZE);
        hikariDataSource.setMinimumIdle(MAX_POOL_SIZE);

        ds = hikariDataSource;
    }

    private static Connection getConnection() {
        try {
            return ds.getConnection();
        } catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }

}
</code></pre><p id="acaa75fb-8a24-4478-befe-98825ffea356" class="">그리고 이전에 UserDaoTest에 있던 </p><pre id="2ad7814d-6638-452c-847e-6dd227d83658" class="code"><code>@BeforeEach
    void setUp() {
        ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
        populator.addScript(new ClassPathResource(&quot;db_schema.sql&quot;));
        DatabasePopulatorUtils.execute(populator, ConnectionManager.getDataSource());
    }</code></pre><p id="78b6b55f-504b-4796-9c65-a2f4a7353449" class="">이 부분에서 getDataSource() 하는 부분을 수정해주기 위해 </p><p id="36cf985c-c601-4502-a6c3-46f71ac1f1d1" class="">ConnectionManager에 getter를 추가해준다.</p><pre id="331ee923-5896-4c3f-b15d-98ca0fe7cd55" class="code"><code>public static DataSource getDataSource(){
        return da;
    }</code></pre><p id="61527f3d-de55-45d2-9384-503850f8d448" class="">
</p><p id="376cb767-b407-4757-bd02-f14442044f18" class="">그 다음 상당히 날 것으로 작성했던 UserDao의 커넥션을 받아오는 부분을 깔끔하게 리팩토링 해보겠다.</p><pre id="01ef7e99-727b-44c9-9a25-93a6e1b2793c" class="code"><code>package org.example;

import javax.xml.transform.Result;
import java.sql.*;

public class UserDao {


    public void create(User user) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;

        try{
            con = ConnectionManager.getConnection();
            String sql = &quot;INSERT INTO USERS VALUES (?, ?, ?, ?,)&quot;;
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, user.getUserId());
            pstmt.setString(2,user.getPassword());
            pstmt.setString(3,user.getName());
            pstmt.setString(4, user.getEmail());

            pstmt.executeUpdate();
;        } finally {
             if(pstmt != null){
                 pstmt.close();
             }

             if(con != null){
                 con.close();
             }
        }
    }



    public User findByUserId(String userid) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            con = ConnectionManager.getConnection();
            String sql = &quot;SELECT userid, password, name, email FROM USERS WHERE userid = ?&quot;;
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1,userid);


            rs = pstmt.executeQuery();
            User user = null;
            if(rs.next()){
                user = new User(
                        rs.getString(&quot;userId&quot;),
                        rs.getString(&quot;password&quot;),
                        rs.getString(&quot;name&quot;),
                        rs.getString(&quot;email&quot;));

            }
            return user;
        } finally {
            if(rs != null){
                rs.close();
            }
            if(pstmt != null){
                pstmt.close();
            }
            if(con != null){
                con.close();
            }
        }
    }
}</code></pre><p id="765147cd-3336-4fe5-8e87-3db0d2ab646e" class="">이게 원래 코드였고</p><p id="50a0fe37-43ae-4fd1-a965-60a096381bb0" class="">일단 JdbcTemplate 클래스를 하나 만들어준다.</p><p id="8d432984-f820-4999-9141-0d1f4a637084" class="">이 JdbcTemplate는 라이브러리이다.</p><p id="80aa1ee2-d0be-45fa-a91f-061c9038ef75" class="">기존에 있던 create 메서드를 JdbcTemplate에 옮기도 executeUpdate로 이름을 바꿔준다.</p><p id="d090913d-9ad6-446d-99ef-4bf4c88d97df" class="">그리고 <code>String sql = &quot;INSERT INTO USERS VALUES (?, ?, ?, ?)&quot;;</code><div class="indented"><p id="442c48a5-d96e-4ec8-a8c9-b20dc77e3868" class="">이 부분을 외부에서 받아오도록 수정해보겠다.</p></div></p><p id="35d0414d-2006-4e93-b000-070a7b9384d3" class="">그리고 preparedStatement도 변경될 수 있는 부분이기에 외부로부터 받아올 수 있도록 한다.</p><p id="9672f4c4-02d7-4657-919e-dccf8fa6eff2" class="">이때 pstmt를 전부 지워주고, 인자로 PreparedStatement pstmt를 받으면 안되는 걸까…? 라고 생각할 수 있지만 그렇게 하려면 외부에 커넥션이 있어야 한다. (pstmt 자체가 커넥션으로부터 얻어진다)</p><p id="d458ccb2-270d-4231-9ca3-af3f5c48b3ef" class="">그러므로 세팅하는 부분만 인자로 받는다.   </p><p id="c2c20963-f426-4590-8153-4c09da3527cd" class="">→<code>PreparedStatementSetter</code> 라는 인터페이스 설정하고 인자로 받기!!</p><pre id="b445e08c-2f29-4cf5-a543-667bcac5b58d" class="code"><code>package org.example;

import java.sql.PreparedStatement;

public interface PreparedStatementSetter {
    void setter(PreparedStatement pstmt);
}</code></pre><p id="5dcf719a-b902-4af5-bb1c-239b97c7db47" class="">PreparedStatement에서는 PreparedStatement를 받아서 세팅을 해줄 것이다.</p><p id="3a89d2e4-2a8d-45c8-941f-3100cd881fb7" class="">그러기 위해서는 원래 create 메서드가 있던 UserDao에서 create2를 설정해서 </p><p id="74bfb23c-eff2-44f1-a013-650004b4e27f" class="">preparedStatement에서 set해주는 부분은 사용하는 쪽, 즉 전달하는 쪽마다 다르기 때문에</p><p id="c9361ee7-b9e7-4799-9c5c-a1ad47bfb1a1" class="">전달하는 쪽에서 set하게 해준다.</p><pre id="82628fe4-f401-4315-9ef7-b29776e51671" class="code"><code>public void create2(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        String sql = &quot;INSERT INTO USERS VALUES (?, ?, ?, ?)&quot;;
        jdbcTemplate.executeUpdate(user, sql, new PreparedStatementSetter() {
            @Override
            public void setter(PreparedStatement pstmt) throws SQLException {
                pstmt.setString(1, user.getUserId());
                pstmt.setString(2,user.getPassword());
                pstmt.setString(3,user.getName());
                pstmt.setString(4, user.getEmail());
            }
        });
    }</code></pre><p id="0a72baa6-daea-4f71-b174-f7242ca22775" class="">그럼 받는 쪽에서(JdbcTemplate) 더이상 몇 개를 세팅해야 하는지 알 필요가 없다.</p><pre id="835850d8-5f65-422e-97bc-785367acaabb" class="code"><code>package org.example;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class JdbcTemplate {

    public void executeUpdate(User user, String sql, PreparedStatementSetter pss) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;

        try{
            con = ConnectionManager.getConnection();
            pstmt = con.prepareStatement(sql);
            pss.setter(pstmt); //이렇게 setter에다 pstmt 전달해주면 끝

            pstmt.executeUpdate();
            } finally {
            if(pstmt != null){
                pstmt.close();
            }

            if(con != null){
                con.close();
            }
        }
    }
}</code></pre><p id="b9f2a2e6-a1e8-492f-a74c-8a126a00bc81" class="">그 다음 UserDao.java에 있던 findByUserId도 리팩토링을 해본다.</p><p id="7379773b-9fe3-4fa6-a6c1-0308bfbe580b" class="">여기서도 변경될 수 있는 부분이 쿼리기 때문에 바깥에서 받아보도록 한다.</p><p id="ae36f624-058f-4151-8723-e1395aa8fe81" class="">위에서와 비슷하게 sql 외부에서 받아올 수 있도록 해주고 setter를 사용해서 세팅해주는 것 까지 동일</p><p id="b9a0283b-74b4-4f2d-9ddf-211b1abf00bb" class="">그리고 </p><pre id="acefbaf4-3edd-486d-ba94-6a35d751a4e8" class="code"><code>if(rs.next()){
                user = new User(
                        rs.getString(&quot;userId&quot;),
                        rs.getString(&quot;password&quot;),
                        rs.getString(&quot;name&quot;),
                        rs.getString(&quot;email&quot;));

            }</code></pre><p id="aa41c8e6-2ca3-4e8f-8459-86aefa325f6a" class="">이 부분에서 new User(~ 부터 변경될 여지가 있기 때문에</p><p id="c21e0228-b75a-4a26-b1fc-c4bc0ae67a8c" class="">RowMapper라는 인터페이스를 만들어 해결해보고자 한다.</p><pre id="c1b38493-2f31-4c53-a211-2f6057cde6b9" class="code"><code>package org.example;

public interface RowMapper {
    Object map();
}</code></pre><p id="9fbe3225-e1e8-4fcf-87e1-136487cb0f05" class="">그리고 userDao에 이런 메소드 하나 만들어주면 된다.</p><pre id="a8febf5a-e742-4bec-a4b4-e2f76a9c0e9a" class="code"><code>public User findByUserId(String userid) throws SQLException {
     JdbcTemplate jdbcTemplate = new JdbcTemplate();

     String sql = &quot;SELECT userid, password, name, email FROM USERS WHERE userid = ?&quot;;
     return jdbcTemplate.executeQuery(userid, sql, pstmt -&gt; pstmt.setString(1,userid), new RowMapper() {
         @Override
         public Object map(ResultSet resultSet) throws SQLException{
             return new User(
                     resultSet.getString(&quot;userId&quot;),
                     resultSet.getString(&quot;password&quot;),
                     resultSet.getString(&quot;name&quot;),
                     resultSet.getString(&quot;email&quot;));

         }

     });


    }</code></pre><p id="b0ffb645-2982-44b1-9bb2-0378c2d7987a" class="">변경되는 부분은 API를 호출하는 호출자 입장에서 전달하면 된다. </p><pre id="96aa9af4-3300-4e84-9791-cecf125a9e8c" class="code"><code>public Object executeQuery(String userid, String sql, PreparedStatementSetter pss, RowMapper rowMapper) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = ConnectionManager.getConnection();
            pstmt = con.prepareStatement(sql);
            pss.setter(pstmt);


            rs = pstmt.executeQuery();
            Object obj = null; //원래 User였는데 종속걸릴까봐 Object로 수정
            if (rs.next()) {

                return rowMapper.map(rs); // 이거 추가
            }
            return obj;
        }finally {
            if(rs != null){
                rs.close();
            }
            if(pstmt != null){
                pstmt.close();
            }
            if(con != null){
                con.close();
            }
        }
    }
}</code></pre><p id="b408281a-aa8c-47ff-98a9-86091f798837" class="">JdbcTemplate에 있던 executeQuery 메서드도 이런 식으로 수정해주고</p><p id="3796670d-9417-482c-9df6-7b19b7dd2420" class="">기존의 UserDao의 findByUserId의 리턴 값도 (User)로 타입캐스팅을 해준다.</p><p id="8c9a7b64-7f4f-4e13-b3dd-8f7728c37192" class="">
</p>
</p></div></article></body></html>
