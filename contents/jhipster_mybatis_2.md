# 게시판 기능 구현

구현해야할 Board 기능은 다음과 같다.

1. 게시글 작성(공지사항, 일반 게시글 구분)
2. 해당 게시글에 댓글 작성
3. 조회수별 게시글 정렬
4. 게시글 검색기능

위 기능 중 조회수별 게시글 정렬 기능은 View에서 처리할 수 있다. 따라서, 가이드 내용에서 제외하도록 한다.

## 게시글 작성

게시글 작성은 Board 추가의 개념이다. 따라서 create/insert Board로 백엔드를 구현한다.

먼저 최종 게시판 모델은 다음과 같다.

### Board.java

```java
package com.skcc.board.domain;

import com.skcc.board.domain.enumeration.Category;
import lombok.Data;



@Data
public class Board {
    private Long id;
    private String title;
    private String content;
    private String writerName;
    private Long writerId;
    private String createdDate;
    private Category category;
    private int hit;

}
```

속성값으로는 id, 제목, 내용, 작성자이름, 작성자Id, 생성날짜, 게시판 카테고리, 조회수가 있다.
Getter, Setter, 생성자 등은 @Data 어노테이션을 사용하였다. 

### BoardMapper.java

BoardMapper와 SQL부터 살펴보자. BoardMapper에서 게시글 추가 메서드는 다음과 같다.

```java
@Mapper
@Repository
public interface BoardMapper {
    ...생략...

    void insertBoard(Board board);
    
}
```

JPA와의 차이점은 JPA와 달리 Mybatis를 적용했을 때는 객체 생성 메서드는 void로 선언해야한다. 

다음은 BoardMapper.xml이다.

### BoardMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.skcc.board.repository.BoardMapper">
    
    <insert id="insertBoard" useGeneratedKeys="true" keyProperty="id" parameterType="Board">
        INSERT INTO board ( title, content, writer_name, writer_id, created_date, category, hit)
        VALUES (#{title}, #{content}, #{writerName},#{writerId}, #{createdDate}, #{category}, #{hit})
    </insert>
   

</mapper>
```

BoardMapper.xml을 살펴보면 insert 쿼리에 옵션이 몇 가지 추가되어있음을 알 수 있다.
먼저, useGeneratedKeys를 살펴보자. useGeneratedKeys를 true로 설정하면 객체를 insert 할 때 키(index값)를 자동으로 생성 후 파라메터로 넘어온 객체에 셋팅할 수 있다.
이때 index칼럼명을 id로 설정하였기 때문에 keyProperty는 id로 설정해주었다.

SQL문을 살펴보면 파라미터로 넘어온 Board 속성들을 칼럼에 입력하는 Insert문임을 알 수 있다. 이전에 SQL 문을 카멜케이스로 변경해주는 설정을 해두었기 때문에 데이터를 삽입하는 칼럼은 SQL 작성법으로, 데이터 값은 java의 카멜케이스 그대로 사용할 수 있다. 

다음은 게시판 서비스와 게시판 서비스의 구현체를 작성해보자.

먼저 게시판 서비스는 다음과 같다.

### BoardService.java

```java

public interface BoardService {

   ...생략...
    
    Board registerNewBoard(Board board);

}
```

게시판 서비스의 구현체는 다음과 같다.

### BoardServiceImpl.java

```java
package com.skcc.board.service.impl;

@Service
@Transactional
public class BoardServiceImpl implements BoardService {

    private final BoardMapper boardMapper;

    public BoardServiceImpl(BoardMapper boardMapper) {
        this.boardMapper = boardMapper;
    }

    //게시글 등록
    @Override
    public Board registerNewBoard(Board board) {
        board.setCreatedDate(LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"))); //생성 날짜 현재 날짜로 세팅
        board.setHit(0); //최초 등록시 조회수 0으로 설정
        boardMapper.insertBoard(board);
        return board;
    }


}

```

구현체에서는 먼저 BoardMapper를 wired하여 사용한다. @Autowired 어노테이션을 사용하기도 하지만, 지양하는 방식이기 때문에 위와 같이 final로 선언한 후 생성자에 포함시켜 사용한다.

게시글 등록 메서드를 보자. 먼저, Board를 파라미터로 받아 생성 날짜와 조회수를 설정한다.
이 외의 속성값들은 클라이언트에서 요청시에 함께 전달될 것이다.

값설정을 마친후에는 boardMapper의 insertBoard를 호출하여 데이터를 저장한다. 

다음으로 클라이언트의 요청이 들어오는 컨트롤러를 살펴보자.

### BoardResource.java

```java
package com.skcc.board.web.rest;

@RestController
@RequestMapping("/api")
public class BoardResource {
    private final Logger log = LoggerFactory.getLogger(BoardResource.class);
    private final BoardService boardService;
    private static final String ENTITY_NAME = "boardBoard";

    @Value("${jhipster.clientApp.name}")
    private String applicationName;

    public BoardResource(BoardService boardService) {
        this.boardService = boardService;
    }

    ...생략...

    @PostMapping("/board")
    public ResponseEntity<Board> addNewBoard(@RequestBody Board board) throws URISyntaxException {
        log.debug("REST request to save board : {}", board);
        if (board.getId() != null) {
            throw new BadRequestAlertException("A new board cannot already have an ID", ENTITY_NAME, "idexists");
        }
        Board result = boardService.registerNewBoard(board);

        return ResponseEntity.created(new URI("/api/board/" + result.getId()))
            .headers(HeaderUtil.createEntityCreationAlert(applicationName, true, ENTITY_NAME, result.getId().toString()))
            .body(result);
    }

}

```

게시판 추가 메서드만 살펴보도록 하자.
먼저, 데이터에 추가하는 것이기 때문에 Post 매핑으로 선언하였고, Board를 리퀘스트 바디 형식으로 받아온다.
받아온 객체가 Null인지 검증한 후, 게시판 서비스를 통해 게시판을 등록한다.


