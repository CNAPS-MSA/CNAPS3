# SimpleCRUD 게시판 만들기

OR mapper가 아닌 Mybatis + H2 + MySQL 기반의 SimpleCRUD 게시판을 만들어보자.

먼저 board 디렉토리를 생성 후 Jhipster Microservice Application을 생성한다.

```
mkdir board
cd board
jhipser
```
Jhipster 옵션은 아래와 같이 선택한다. 

```
? Which *type* of application would you like to create? Microservice application


? [Beta] Do you want to make it reactive with Spring WebFlux? No
? What is the base name of your application? board
? As you are running in a microservice architecture, on which port would like yo
ur server to run? It should be unique to avoid port conflicts. 8084
? What is your default Java package name? com.skcc.board
? Which service discovery server do you want to use? JHipster Registry (uses Eur
eka, provides Spring Cloud Config support and monitoring dashboards)
? Which *type* of authentication would you like to use? JWT authentication (stat
eless, with a token)
? Which *type* of database would you like to use? SQL (H2, MySQL, MariaDB, Postg
reSQL, Oracle, MSSQL)
? Which *production* database would you like to use? MySQL
? Which *development* database would you like to use? H2 with in-memory persistence
? Do you want to use the Spring cache abstraction? Yes, with the Hazelcast imple
mentation (distributed cache, for multiple nodes, supports rate-limiting for gat
eway applications)
? Do you want to use Hibernate 2nd level cache? No
? Would you like to use Maven or Gradle for building the backend? Maven
? Which other technologies would you like to use? Asynchronous messages using Apache Kafka
? Would you like to enable internationalization support? Yes
? Please choose the native language of the application Korean
? Please choose additional languages to install English
? Besides JUnit and Jest, which testing frameworks would you like to use? Gatlin
g
? Would you like to install other generators from the JHipster Marketplace? No

Installing languages: ko, en for server
Git repository initialized.
```

옵션 선택 완료 후 Jhipser 애플리케이션이 생성된다.

Jhister는 Mybatis를 지원하지 않기 때문에, 이제부턴 세부적으로 설정할 것들이 남아있다.

## Lombok 적용하기

[Lombok 적용하기](/contents/jhipster_lombok.md)

## Mybatis 적용하기

Lombok을 적용하였으면 pom.xml을 수정하여 Mybatis를 적용시킨다.

```xml
<dependencies>
...

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

</dependencies>
```

## feign과 h2 설정하기

이제 board 애플리케이션의 database 연결, feign 연결 등을 설정해준다.

먼저 application.yml부터 수정해보자.

### application.yml

application.yml에서 feign 설정 중 timeout 관련 코드는 주석처리 되어있을 것이다. 

아래와 같이 주석을 해제시킨다.

```yaml
feign:
  hystrix:
    enabled: true
    client:
      config:
        default:
          connectTimeout: 5000
          readTimeout: 5000
```

### application-dev.yml

application-dev.yml에서는 h2 및 liquibase를 설정한다.

simpleCRUD 게시판에서는 h2의 in-memory DB를 통해 데이터를 확인하기 위해서 h2 console을 사용할 것이다. 따라서 아래와 같이 console 사용을 true로 설정한다.
또한, liquibase의 fakeData 를 사용하지 않을 것이기 때문에 liquibase의 contexts에서 faker를 삭제한다. 

```yaml
h2:
    console:
      enabled: true
  ...

  liquibase:
    # Remove 'faker' if you do not want the sample data to be loaded automatically
    contexts: dev

```

## Mybatis 설정하기

이제 Mybatis를 사용할 수 있도록 설정해보자.

`com.skcc.board.config`폴더로 이동하여 DatabaseConfiguration.java를 아래와 같이 수정한다.

### DatabaseConfiguration.java

```java
package com.skcc.board.config;

...(중략)...

@Configuration
@EnableJpaRepositories("com.skcc.board.repository")
@EnableJpaAuditing(auditorAwareRef = "springSecurityAuditorAware")
@EnableTransactionManagement
@MapperScan(value = "com.skcc.board.repository", annotationClass = Mapper.class, sqlSessionFactoryRef = "sqlSessionFactory")
public class DatabaseConfiguration {

    private final Logger log = LoggerFactory.getLogger(DatabaseConfiguration.class);

    private final Environment env;

    private final ApplicationContext applicationContext;

    public DatabaseConfiguration(Environment env, ApplicationContext applicationContext) {
        this.env = env;
        this.applicationContext = applicationContext;
    }



    /**
     * Open the TCP port for the H2 database, so it is available remotely.
     *
     * @return the H2 database TCP server.
     * @throws SQLException if the server failed to start.
     */
    @Bean(initMethod = "start", destroyMethod = "stop")
    @Profile(JHipsterConstants.SPRING_PROFILE_DEVELOPMENT)
    public Object h2TCPServer() throws SQLException {
        String port = getValidPortForH2();
        log.debug("H2 database is available on port {}", port);
        return H2ConfigurationHelper.createServer(port);
    }

    private String getValidPortForH2() {
        int port = Integer.parseInt(env.getProperty("server.port"));
        if (port < 10000) {
            port = 10000 + port;
        } else {
            if (port < 63536) {
                port = port + 2000;
            } else {
                port = port - 2000;
            }
        }
        return String.valueOf(port);
    }


    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setTypeAliasesPackage("com.skcc.board.domain"); //모델이 위치하는 패키지
        sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis-mapper/**/*.xml")); //mapper.xml의 위치 설정
        return sqlSessionFactoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

추가 및 수정한 부분은 다음과 같다.

1. MapperScan 어노테이션 추가

    @MapperScan(value = "com.skcc.board.repository", annotationClass = Mapper.class, sqlSessionFactoryRef = "sqlSessionFactory")

    : MapperScan 어노테이션은 mapper.xml파일들이 바라볼 기본 패키지 위치를 지정하는 어노테이션이다. BoardMapper파일은 `com.skcc.board.repository`에 생성할 예정이기 때문에 위와 같이 설정한다.

1. SqlSessionFactory 추가
    
    모든 Mybatis 애플리케이션은 SqlSessionFactory인스턴스를 사용한다. 따라서 아래 코드를 추가하였다.

    ```java

    private final ApplicationContext applicationContext;

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setTypeAliasesPackage("com.skcc.board.domain"); //모델이 위치하는 패키지
        sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:mybatis-mapper/**/*.xml")); //mapper.xml의 위치 설정
        return sqlSessionFactoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
    ```

    여기서 setMapperLocation의 기본 위치는 src/main/resource/이다. 샘플에서는 resource 폴더에 mybatis-mapper라는 폴더를 생성하여 SQL쿼리문을 담고 있는 xml 파일을 관리할 예정이다. 따라서 path를 위 코드와 같이 작성하였다.

## 도메인 모델 개발

Board는 제목, 내용, 작성자, 카테고리, 생성날짜, 조회수를 속성으로 갖는다.
추후 댓글 기능을 추가하면 board에 comments 속성을 추가할 예정이다.

### Board.java

```java
@Data
public class Board {
    private Long id;
    private String title;
    private String content;
    private String writer;
    private LocalDateTime createdDate;
    private Category category;
    private int hit;
}

```

## DB 세팅을 위한 스크립트 작성

H2 in-memory DB를 사용할 때, board라는 테이블이 없으면 에러가 발생할 것이다. 따라서, src/main/resource 폴더에 schema.xml파일을 생성한 뒤 스크립트를 작성하여 기본 board 테이블을 생성하자.

작성한 스크립트 파일은 애플리케이션을 동작시킬 때 실행되어 테이블을 생성한다.
이때 스크립트 파일은 H2 SQL 문법에 맞추어 작성하였다.

```sql
drop table if exists board;

create table board
(
    id bigint auto_increment primary key not null,
    title varchar(255),
    content varchar(1000),
    writer varchar(50) not null,
    createdDate varchar(50) not null,
    category varchar(255) not null,
    hit integer
);


```

## Mapper 인터페이스 개발

Mapper 인터페이스를 개발해보자. Mapper 인터페이스는 서비스에서 데이터베이스로 접근할 때 호출된다.

### BoardMapper.java

```java
@Mapper
@Repository
public interface BoardMapper {
    Board selectBoardById(Long id);
    List<Board> selectAllBoard();
    void insertBoard(Board board);
}
```

우선 기본적인 CRU 메소드만 구성하였다.

다음으로는 Mapper인터페이스에서 메소드가 호출되었을 때 실행될 SQL문이 작성될 xml 파일을 개발한다.

## Mapper XML 개발

먼저 src/main/resource 폴더에 mybatis-mapper 폴더를 생성한다.
그 다음, boardMapper.xml 파일을 생성하여 아래와 같이 코드를 작성한다.

### boardMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.skcc.board.repository.BoardMapper">
    <select id="selectBoardById" resultType="Board">
        SELECT * FROM board WHERE id = #{id}
    </select>
    <select id="selectAllBoard" resultType="Board">
        SELECT * FROM board
    </select>
    <insert id="insertBoard" parameterType="Board">
        INSERT INTO board ( title, content, writer, createdDate, category, hit)
        VALUES ( #{title}, #{content}, #{writer}, #{createdDate}, #{category}, #{hit})
    </insert>
</mapper>
```

namespace는 XML파일과 연동될 mapperInterface의 위치를 작성한다.

그 다음, 해당 인터페이스에서 제공할 메소드에 따라 쿼리를 작성한다. 
id는 인터페이스의 메소드 명이며 parameterType이나 resultType 또한 메소드의 파라미터나 리턴타입에 맞게 작성한다. 

다음으로 서비스 레이어를 작성해보자.

## BoardService 구현

기본 CRU 메소드만 먼저 구현해보자.


### BoardService.java

```java

import com.skcc.board.domain.Board;

import java.util.List;

public interface BoardService {

    Board findBoardById(Long id);

    List<Board> findAllBoard();

    void addNewBoard(Board board);

}

```

위 서비스 인터페이스의 구현체는 다음과 같다.

### BoardServiceImpl.java

```java

package com.skcc.board.service.impl;

import com.skcc.board.domain.Board;
import com.skcc.board.repository.BoardMapper;
import com.skcc.board.service.BoardService;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional
public class BoardServiceImpl implements BoardService {

    private final BoardMapper boardMapper;

    public BoardServiceImpl(BoardMapper boardMapper) {
        this.boardMapper = boardMapper;
    }

    @Override
    public Board findBoardById(Long id) {
        return boardMapper.selectBoardById(id);
    }

    @Override
    public List<Board> findAllBoard() {
        return boardMapper.selectAllBoard();
    }

    @Override
    public void addNewBoard(Board board) {
        boardMapper.insertBoard(board);
    }
}

```

그럼 이제 ApplicationTest를 통해 Mapper와 DB가 잘 동작하는지 테스트해보자.

## DemoApplicationTest 작성

ApplicationTest를 작성할 위치는 com.skcc.board에 작성하도록 한다.(기본 BoardApp.java와 동일한 위치)

테스트 코드는 아래와 같다.

### DemoApplicationTest.java

```java
package com.skcc.board;

import com.skcc.board.domain.Board;
import com.skcc.board.domain.enumeration.Category;
import com.skcc.board.service.BoardService;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.time.LocalDateTime;


import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.junit.Assert.assertThat;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class DemoApplicationTest {
    @Autowired
    private BoardService boardService;

    private Board board;

    @Before
    public void setUp(){
        board = new Board();
        board.setCategory(Category.NORMAL);
        board.setContent("test content");
        board.setCreatedDate(LocalDateTime.now());
        board.setHit(123);
        board.setTitle("test title");
        board.setWriter("test writer");
        boardService.addNewBoard(board);
    }
    @Test
    public void boardTest() {

        Board findBoard = boardService.findBoardById(1L);
        assertThat(board.getContent(), is(equalTo(findBoard.getContent())));
        System.out.println(findBoard);
    }

}

```

테스트 결과는 다음과 같다.

```
..(생략)..
2020-11-03 18:16:56.795 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Creating a new SqlSession
2020-11-03 18:16:56.795 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@514c1b69]
2020-11-03 18:16:56.797 DEBUG 3275 --- [           main] o.m.s.t.SpringManagedTransaction         : JDBC Connection [HikariProxyConnection@726270291 wrapping conn0: url=jdbc:h2:mem:board user=SA] will be managed by Spring
2020-11-03 18:16:56.801 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@514c1b69]
2020-11-03 18:16:56.801 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Transaction synchronization committing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@514c1b69]
2020-11-03 18:16:56.801 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Transaction synchronization deregistering SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@514c1b69]
2020-11-03 18:16:56.801 DEBUG 3275 --- [           main] org.mybatis.spring.SqlSessionUtils       : Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@514c1b69]
Board(id=1, title=test title, content=test content, writer=test writer, createdDate=2020-11-03T18:16:56.757098, category=NORMAL, hit=123)
2020-11-03 18:16:56.808 DEBUG 3275 --- [           main] reactor.core.publisher.Hooks             : Reset onLastOperator: org.springframework.security.core.context.SecurityContext
2020-11-03 18:16:56.811  INFO 3275 --- [.ShutdownThread] com.hazelcast.instance.Node              : [192.168.123.6]:5701 [dev] [3.12.7] Running shutdown hook... Current state: ACTIVE
2020-11-03 18:16:56.836  INFO 3275 --- [extShutdownHook] c.skcc.board.config.CacheConfiguration   : Closing Cache Manager

Process finished with exit code 0
```

위 결과와 같이 테스트가 성공적으로 통과하였음을 확인할 수 있다. 이제 본격적으로 Board 기능을 구현해보자.

구현해야할 Board 기능은 다음과 같다.

1. 게시글 작성(공지사항, 일반 게시글 구분)
2. 해당 게시글에 댓글 작성
3. 조회수별 게시글 정렬
4. 게시글 검색기능


