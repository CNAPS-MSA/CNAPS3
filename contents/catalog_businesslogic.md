# 카탈로그(Catalog)서비스 구현
**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**

Sample에서 보여줄 기능은 아래와 같다.

## 구현기능
  - 도서목록 및 검색기능
    1. 도서입고 시 Catalog생성
  - 인기도서목록 기능   
    1. 대출기록으로 인기도서 목록 생성 

## API설계

|API명|인기도서목록 조회|
|----|------|
|리소스URI|/book-catalogs/top-10|
|Method|GET|
|Request| |
|Response| |

대여 횟수가 많은 순으로 상위 10개의 도서목록을 조회한다.

## 도메인 모델링

BookCatalog 서비스는 도서 정보와 도서 목록 조회를 위한 서비스이다. Book 서비스에서 도서가 등록/수정/삭제 됨에 따라 BookCatalog 또한 등록/수정/삭제된다.
또한, BookCatalog는 조회와 검색을 위한 서비스인만큼 NoSQL인 MongoDB로 구현하였다.

## 유스케이스 흐름

## 내부영역 - 도메인 모델 개발

### BookCatalog.java

Catalog 엔티티이다. 조회/검색 용도이기 때문에 사용자가 도서 조회/검색시 필요한 속성들만을 가지고 있다.
따라서, 도서 제목, 설명, 작가, 출판사 등의 속성을 갖고 있다.
DB가 MongoDB이기 때문에 기존의 다른 서비스들과 다르게 엔티티에 `@Entity`가 아닌 `@Document`라는 어노테이션이 쓰인다. 

```java
@Document(collection = "book_catalog")
@Data
@ToString
public class BookCatalog implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    private String id;

    @Field("title")
    private String title;

    @Field("description")
    private String description;

    @Field("author")
    private String author;

    @Field("publication_date")
    private LocalDate publicationDate;

    @Field("classification")
    private String classification;

    @Field("rented")
    private Boolean rented;

    @Field("rent_cnt")
    private Long rentCnt;

    @Field("book_id")
    private Long bookId;
  
  ...(중략)...

```

## 내부영역 - 서비스 개발

BookCatalogService에서는 도서 정보 생성/수정/삭제기능을 구현하였다.
도서 정보 생성/수정/삭제는 외부서비스인 Book서비스의 비동기호출에 의해 이뤄지므로 인바운드 어댑터 개발에서 다루도록 한다.
다음은 도서 제목으로 도서를 검색하는 기능과 인기도서 목록을 불러오는 기능이다.

### BookCatalogService.java

```java
public interface BookCatalogService {
 ...(중략)...
 Page<BookCatalog> findBookByTitle(String title, Pageable pageable);

 List<BookCatalog> loadTop10();

}
```

### BookCatalogServiceImpl.java

```java

@Service
public class BookCatalogServiceImpl implements BookCatalogService {
  ...(중략)...
    @Override
    public Page<BookCatalog> findBookByTitle(String title, Pageable pageable)
    {
        return bookCatalogRepository.findByTitleContaining(title, pageable);
    }

    @Override
    public List<BookCatalog> loadTop10() {
        return bookCatalogRepository.findTop10ByOrderByRentCntDesc();
    }

}
```
도서 검색기능과 인기도서 검색 기능 모두 BookCatalogRepository의 JPA 메소드를 실행시켜 목록을 불러온다. 

## 내부영역 - 레파지토리 개발

BookCatalogService에서 쓰인 도서 검색 메소드와 인기도서 검색 메소드를 살펴보자.

### BookCatalogRepository.java

```java
@Repository
public interface BookCatalogRepository extends MongoRepository<BookCatalog, String> {
    
    Page<BookCatalog> findByTitleContaining(String title, Pageable pageable);

    List<BookCatalog> findTop10ByOrderByRentCntDesc();

    ..(중략)...
}
```

도서 제목으로 검색하는 메소드는 `findByTitleContaining`으로, 받아온 title을 포함하고 있는 BookCatalog 리스트를 반환한다.

인기도서를 검색하는 메소드는 `findByTitle10ByOrderByRentCntDesc()`로, 메소드를 천천히 해석해보면
- 상위10개의 BookCatalog검색
- RentCnt(대여횟수) 순으로 정렬
- 내림차순
즉, 대여횟수 순으로 내림차순 정렬하여 상위10개의 BookCatalog 리스트를 반환한다. 

또한, 위에서 살펴볼 수 있듯이 NoSQL인 MongoDB를 사용한다하더라도 Spring Data JPA 쿼리메소드를 동일하게 사용할 수 있다.
단, NoSQL은 Document 형식이며 ID의 속성이 String인 점을 감안하여 약간의 차이가 있을 수 있으므로 Spring Data MongoDB의 공식 문서를 살펴보는 것을 권장한다.

## 외부영역 - REST 컨트롤러 개발

BookCatalog

## 인바운드 어댑터 개발

## 단위테스트 수행

