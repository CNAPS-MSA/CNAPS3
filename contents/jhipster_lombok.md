## Jhister Project에 Lombok 적용시키기

Jhipser에서는 기본적으로 Lombok을 지원하지 않으나, Lombok API를 직접 추가하여 Lombok을 사용할 수 있다.

1. pom.xml 수정

각 마이크로서비스 프로젝트의 pom.xml을 아래와 같이 수정한다.

```xml
     <junit.utReportFolder>${project.testresult.directory}/test</junit.utReportFolder>
        <junit.itReportFolder>${project.testresult.directory}/integrationTest</junit.itReportFolder>	        
        <!-- jhipster-needle-maven-property -->	       
        <lombok.version>1.18.12</lombok.version>
    </properties>

	    <dependencyManagement>

  <artifactId>metrics-core</artifactId>
     </dependency>
        <!-- jhipster-needle-maven-add-dependency -->	       
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
    </dependencies>


    <build>	   
        <version>${jaxb-runtime.version}</version>
        </path>	                           
         <!-- jhipster-needle-maven-add-annotation-processor -->	                            
                <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                    <version>${lombok.version}</version>
        </path>
    </annotationProcessorPaths>	                        
</configuration>	                    
</plugin>	                  

```

위와 같이 수정한 후, 프로젝트를 다시 빌드하면 Lombok사용이 가능하다.

2. Lombok 적용하여 소스코드 수정

기존의 Entity와 DTO에는 getter와 setter 코드가 작성되어있을 것이다.
이를 삭제하고 아래와 같이 수정하여 사용한다. (Getter, Setter 삭제)

이때 주의할 점은 equals와 hashcode 메소드는 삭제하지 않아야한다.(Jhipster Test code와 충돌 에러)
간혹 ToString 또한 충돌하는 경우도 있으니 직접 테스트해보며 적용하길 권장한다.

```java
@Entity
@Table(name = "book")
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
@Data
@ToString
public class Book implements Serializable {

```

또한 DTO의 경우 아래와 같이 적용한다. DTO의 경우 또한 equals와 hashcode, toString 메소드가 충돌하는 경우엔 삭제하지 않았다.
따라서, 직접 테스트해보며 적용하길 권장한다. 

```java
@Getter
@Setter
@EqualsAndHashCode
@ToString
public class RentalDTO implements Serializable {

```

3. Lombok적용시 주의 사항

Lombok은 소스코드를 매우 간결하게 해주며 Powerful한 기능을 갖고 있다. 하지만 그만큼 위험성도 크기 때문에 Jhipster에서는 기본 개발환경 설정에 Lombok을 적용하지 않기도 했다.
따라서, 필요한 Annotation을 잘 선택하여 적용하는 것이 중요하다.

샘플 코드에서 주로 사용한 어노테이션은 다음과 같다.

- @Data : @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor 메소드 자동 생성
- @Getter : Getter 메소드 생성
- @Setter : Setter 메소드 생성
- @ToString : ToString 메소드 생성 (Class 내의 필드를 문자열로 변환)
- @EqualsAndHashCode : 객체 비교 등의 용도로 사용되는 equals()와 hashCode() 생성
- @AllArgsConstructure : 모든 필드를 포함한 생성자 메소드 생성
- @NoArgsConstructure : 모든 필드가 포함되지 않은 빈 생성자 메소드 생성

이외의 자세한 사항은 아래 Lombok 공식 홈페이지 자료를 통해 확인해보자.
> https://projectlombok.org/features/all
