# 내부구조정의
## 1.마이크로서비스내부 패키지구조
## 사례
### 헥사고널+DDD 적용 패키지구조

![패키지](/img/package.png)  

#### 패키지 명명규칙
|구분|패키지 명|유형|명명 규칙|역할|작성기준|
|---|--|---|---|---|---|
|내부 영역|	도메인(domain)|Class|도메인개념을 명확히 표현할 수 있는 명사형	|도메인 모델 비지니스 개념 및 로직 표현.<br>어그리게잇,엔터티,VO, 표준 타입 패턴으로 구현|어그리게잇 단위|
|	|서비스(Service)|Interface|~Service|서비스의 인터페이스|어그리게잇당 1개|
|	|	|Class|~ServcieImpl|서비스의 구현체 <br>업무처리흐름 구현	|서비스 I/F당 1개|
| | |레파지토리(repository)|Interface|~Repository|저장소 처리	엔터티 당  1개|
|외부 영역|웹REST(web.rest)|Class|~Resource|REST API 발행 , 인바운드 요청 처리||
| |어댑터(adaptor)|Class|~Client|동기 아웃바운드 처리.<br>타서비스 동기 호출	|호출할 타 서비스당 1개|
| | |Class|~Consumer|비동기 메시지 인바운드 처리(수취)||	
| | |Interface|	~ Producer|	비동기 아웃바운드 메시지 전송 정의하는 인터페이스	|호출할 타 서비스당 1개|
|	|	|Class|	~ProducerImpl|비동기 아웃바운드 메시지 전송 구현체|Producer에 의존|
|	|dto|Class|~DTO|데이터전송객체(Data Transfer Object)<br>동기 호출 시 데이터 전송 객체로 사용	|API에 의존됨|




### 헥사고널+트랜젝션스크립트 패키지구조
추가예정
  
