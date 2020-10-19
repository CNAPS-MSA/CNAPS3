도메인 모델과 JPA를 매핑하기 위해 자주 사용되는 연관 관계는 다음과 같다.

- 다대일 단방향
- 다대일 양방향
- 일대다 단방향

![manytoone](https://user-images.githubusercontent.com/15258916/96412189-afa6d100-1224-11eb-8da1-988c0cb3465e.png)

메시지와 게스트의 관계는 다대일 단방향 관계이다.
message.guest 와 같이 참조 가능하지만,   반대로 guest에서는 메시지를 참조하는 필드가 없다. 따라서 메시지와 손님은 다대일 단방향 관계이다. 

```java
DataAllArgsConstructor
@Table(name = "T_MSG")
//@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})
public class Messages extends AbstractEntity implements AggregateRoot{          
@Columnprivate 
String title;      

@Column(nullable = false)
@NotEmptyprivate 
String message;         

@ManyToOne//(fetch = FetchType.LAZY)
@JoinColumn(name = "GUEST_ID")       
private Guest guest;  
}

/*** <pre>* 다대일 단방향[N:1] 샘플 객체 양방향 관계에서 연관관계의 주인은 항상 다쪽이다.
</pre>*/DataAllArgsConstructor@Table(name = "T_GUEST")
@NamedQuery(name = "findByAge", query = "from Guest where age > :min and age < :max") 
@AttributeOverrides({@AttributeOverride(name = "id", column = @Column(name = "GUEST_ID"))})
public class Guest extends AbstractEntity{           
@Column(length = 50, nullable = false)
@NotEmpty      
private String name;

@Column(length = 200, nullable = true)
@Email      
private String mail;      

@Column(nullable = true)
@Min(value = 0)     
@Max(value = 200)      
private Integer age;
} 
```
![manyyoone2](https://user-images.githubusercontent.com/15258916/96412194-b2092b00-1224-11eb-9bda-b03df60d364a.png)

- 객체 양방향 관계에서 주인은 항상 다쪽이다.
- 양방향 연관관계는 항상 서로를 참조해야한다. 항상 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는것디 좋은데 setTeam(), addMember() 메소드가 편의 메소드들이다.
- 편의 메소드는 한 곳에서만 작성하거나 양쪽에 다 작성할 수 있는데, 양쪽에 다 작성하면 무한루프에 빠지므로 주의해야한다.

```java
*** <pre>다대일 양방향[N:1, 1:N] 예제 , 양방향이라 함은 양쪽에서 모두 참조를 호출할 수 있는 비지니스를 의미
EntityNoArgsConstructor
@Table(name = "T_TEAM") 
@AttributeOverrides({@AttributeOverride(name = "id", column = @Column(name = "TEAM_ID"))})
public class Team extends AbstractEntity implements AggregateRoot 
{       
@Column(name = "TEAM_NAME")
private String teamName;

@Column(name = "TEAM_DESC")
private String teamDesc;          

@OneToMany(cascade = CascadeType.ALL, mappedBy = "teams") // @OneToMany의 fetch 기본전략은 LAZY이다.
private List<Member> member = new ArrayList<>();               

public void addMember(Member m){this.member.add(m);//멤버의 팀이 자기자신이 아니면 멤버에 팀 추가if(m.getTeams() != this){//양쪽에 작성되어 있기에  무한루프 방지
m.setTeam(this);
}}}   /** <pre>* 다대일 양방향[N:1, 1:N] 예제 , 양방향이라 함은 양쪽에서 모두 참조를 호출할 수 있는 비지니스를 의미

EntityNoArgsConstructor
@Table(name = "T_MEMBER")
public class Member extends AbstractEntity {           
@Column(name = "MEMBER_NAME")
private String name;

@Column(name = "AGE")
private String age;      

@Setter(AccessLevel.NONE)
@ManyToOne(optional = true,fetch = FetchType.LAZY)// @ManyToOne의 fetch 기본전략은 EAGER이다.
@JoinColumn(name = "TEAM_ID")       
private Team teams;              

public void setTeam(Team team){this.teams = team;//팀의 멤버중에 자기 자신이 없으면 추가if(!teams.getMember().contains(this)){ //양쪽에 작성되어 있기에 무한루프 방지teams.getMember().add(this);
}}}
```
![onetoone](https://user-images.githubusercontent.com/15258916/96412202-b5041b80-1224-11eb-8e35-7a726d677eca.png)

일대다 관계는 엔티티를 하나이상 참조할 수 있다.(Collection, List, Map, Set)
본인 테이블에 외래 키가 있으면 엔티티의 저장과 연관관계 처리를 INSERT SQL 한 번으로 끝낼 수 있지만, 다른 테이블에 외래 키가 있기때문에,
연관관계 처리를 위한 UPDATE SQL을 추가로 실행해야 한다.
따라서 일대다 단방향 매핑보다는 다대일 양방향 매핍을 사용하는 것이 좋은 것 같다. 일대다 단뱡향 은 다른 테이블에서 외래키를 관리해야 하기 때문이다.

```java
</pre>/Entity
NoArgsConstructor
@Table(name = "T_STUDENT")
public class Student extends AbstractEntity{ 
@Column(name = "STUDENT_NAME")
private String username;
} /** <pre>* 일대다 단방향[1:N] 샘플*

@Entity
@NoArgsConstructor
@Table(name = "T_SCHOOL")
@AttributeOverrides({@AttributeOverride(name = "id", column = @Column(name = "SCHOOL_ID"))})
public class School extends AbstractEntity implements AggregateRoot 
{ 
@Column(name = "SCHOOL_NAME")
private String name; //Student 마다 각각의 외래키를 가지고 있다. 본인이 아닌 다른테이블에 관리
@JoinColumn(name = "SCHOOL_ID")
private List<Student> student = new ArrayList<>();
}
```




