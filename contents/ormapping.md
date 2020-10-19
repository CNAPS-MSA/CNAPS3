도메인 모델과 JPA를 매핑하기 위해 자주 사용되는 연관 관계는 다음과 같다.

- 다대일 단방향
- 다대일 양방향
- 일대다 단방향


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
