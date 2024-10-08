## 08/01
### [ spring boot & jpa 강의 수강 3일차 ]
#### **section2 - 3 ( 엔티티 클래스 개발 )**
![Alt text](<화면 캡처 2024-08-03 132254.png>)
Album, Book, Movie는 Item 클래스의 자식클래스이다.

이 관계에서 Item 클래스에는 @Inheritance라는 어노테이션을 지정해줘야하는데,

strategy = InheritanceType에는 3가지 종류의 전략이 있다.

<br>

> @Inheritance(strategy = InheritanceType.JOINED)

가장 정규화된 스타일로 하는 것.

부모 클래스의 테이블이 공통속성을 가지고 자식 테이블들은 각자의 고유한 컬럼들을 따로 갖는다.

<br>

> @Inheritance(strategy = InheritanceType.SINGLE_TABLE)

한 테이블에 모든 자식의 정보를 다 때려박는 것. 디폴트로 설정됨.

<br>

> @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)

부모 클래스에 대한 테이블은 존재하지 않는다.
각 자식 클래스 별로 자신만의 테이블을 갖는데,
이때 부모가 갖고 있어야 할 고유한 속성들을 모든 자식 클래스가 자신의 테이블에 갖고 있는 방식

여기서는 @Inheritance(strategy = InheritanceType.SINGLE_TABLE) 방식을 채택했다.

<추상클래스(부모)>

 싱글테이블에서 구분할 수 있는 타입을 명시하는 것이 dtype이다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name="item_id")
    private Long id;

    // 상속관계 매핑
     private String name;
     private int price;
     private int stockQuantitty;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

}
```
<br>

<자식클래스(Albums)>

싱글테이블 전략을 채택했기 때문에 저장할때 구분할 수 있는 것이 있어야 한다.
그러기 위해서 @DiscriminatorValue("A") 어노테이션을 붙이고 구분할 값을 넣어주면 된다.

```java
@Entity
@DiscriminatorValue("A")
@Getter
@Setter
public class Album extends Item {
}
```
<br>

<자식클래스(Book)>

```java
@Entity
@DiscriminatorValue("B")
@Getter
@Setter
public class Book extends Item {

    private String author;
    private String isbn;
}
```

<br>

<자식클래스(Movie)>
```java
@Entity
@DiscriminatorValue("M")
@Getter
@Setter
public class Movie extends Item {

    private String director;
    private String actor;
}
```

<br>
<br>

> Enum

Enum클래스를 필드로 가지고 올때 @Enumerated을 지정해줘야하는데, 타입이 두 가지가 있다.

- @Enumerated(EnumType.ORDINAL)
    - default 타입. 절대로쓰면 안된다.
    - 처음에 man, woman을 enum값으로 넣으면 1,2 순서대로 값을 갖는데, 중간에 man, human, woman 처럼 값이 들어와 버리면 순서대로 1,2,3을 갖게 되므로 값이 꼬여버린다. 그래서 절대로 쓰면 안된다.

- @Enumerated(EnumType.STRING)
    - String은 말 그대로 문자값을 그대로 넣기 때문에 안정적이다.


```java
@Entity
@Getter
@Setter
public class Delivery {

    @Id
    @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery", fetch = LAZY)
    private Order order;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus status;  // READY, COMP
}
```

```java
public enum DeliveryStatus {
    READY, COMP
}
```

<br>
<br>

## 08/02
### [ spring boot & jpa 강의 수강 4일차 ]
#### **section2 - 5 ( 엔티티 설계시 주의점)**

- 엔티티를 변경할 때는 Setter 대신에 변경 지점이 명확하도록 변경을 위한 비즈니스 메서드를 별도로 제공해야 한다. (Setter 남발하지 않기)
    - 변경포인트가 많아서 에러가 나서 유지보수시, 특정엔티티가 어디서 어떻게 변경됐는지 알 수가 없기 때문.
- 모든 연관관계는 지연로딩으로 설정
    - 즉시로딩(EAGER)은 연관된 테이블들을 전부 다 끌어모으기 때문에 어떤 SQL이 실행될지 추적하기 어렵고, 특히 N+1 문제가 자주 발생한다.
    - 실무에서 모든 연관관계는 지연로딩(LAZY)로 설정해야하 한다.
    - @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다
    - XTOMany는 기본이 지연로딩이기 때문에 그대로 두어도 된다.
- 컬렉션은 필드에서 초기화하자.
```java
Member member = new Member();
System.out.println(member.getOrders().getClass());
em.persist(member);         // db에 영속화 하겠다는 뜻
System.out.println(member.getOrders().getClass());
```
이런 코드가 있다고 가정하자.

출력결과는 다음과 같다.
```
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag
```
같은 코드인데, 출력값이 달라졌다. 그 이유는

영속화 되면 하이버네이트가 컬렉션을 감싸기 때문에 결과가 달라지게된다. 이제 이 컬렉션을 바꾸게 되면 하이버네이트가 원하는대로 동작을 안하게 되는 문제가 발생한다.
따라서 컬렉션은 필드에서 바로 초기화 하는 것이 안전하다.


## 08/03
### [ spring boot & jpa 강의 수강 5일차 ]
#### **section2 - 5 ( 엔티티 설계시 주의점)**
> 연관관계 (편의)메서드

- 양방향 관계일때, 양쪽에 값을 다 세팅해주는것이 좋다.
- 연관관계 메서드의 위치는 행동적으로 컨트롤 하는 쪽에 위치하는것이 좋다.

<br>

편의메서드를 쓰지 않는다면
```java
Member member = new Member();
Order order = new Order();

member.getOrders().add(order);
order.setMember(member);
```

 처럼 넣어줘야한다. 하지만 이 중에서 뭔가 하나를 빼먹는 실수가 일어날 수도 있는데, 이를 방지하기 위해 이 두개를 원자적으로 묶는 메서드를 만든 것이다.
 
 그럼 코드가
 ```java
Member member = new Member();
Order order = new Order();

order.setMember(member);
 ```
이렇게 줄어들게 된다.


<br>

> SpringPhysicalNamingStrategy

하이버네이트에서 엔티티의 필드명을 자동으로 바꿔주는 기능이다.

- 카멜 케이스 -> 언더스코어 (memberPoint -> member_point)
- 점 -> 언더스코어 (. -> _)
- 대문자 -> 소문자

이 클래스에서 제공하는 방법을 그대로 따르는 방법도 있지만 내가 이걸참고해서 내 맘대로 수정할 수 있음.

## 08/04
### [ JAVA 문제풀기 - HackerRank ]

### [ spring boot & jpa 강의 수강 6일차 ]
#### **section4 - 1 ( 회원 리포지토리 개발 )**
> @Repository

 - spring bean에 의해 자동으로 관리가 됨

 <br>
 <br>

> 단축키 ctrl+alt+N
```java
List<Member> result = em.createQuery("select m from Member m", Member.class).getResultList();
        return result;
```
를
```java
return em.createQuery("select m from Member m", Member.class).getResultList();
```
이렇게 만들어줌

- 여기서 m은 Member 객체를 의미한다.
- select m은 select * 와 같은 의미이다.

 <br>
 <br>

 > @SpringBootApplication

 이 어노테이션이 있으면 이 패키지와 이 패키지 하위에 있는 패키지의 컴포넌트들을 전부 스캔, 자동등록한다.

<br>
 <br>

> @PersistenceContext

jpa가 제공하는 표준 어노테이션, spring이 entitymanager를 만들어서 주입(injection)을 해줌



<br>
 <br>

 >  @PersistenceUnit

 ```
 private EntityManagerFactory emf;
 ```
 를 하면 EntityManagerFactory를 직접 주입받을 수도 있다.

<br>
 <br>
 
 ## 08/05
 ### [ spring boot & jpa 강의 수강 7일차 ]
 #### **section4 - 2 ( 회원 서비스 개발 )**
 > 의존성 주입하는 여러가지 방법

 ##### 1번
```java
public class MemberService {
    @Autowired
    private MemberRepository memberRepository;
}
```

<br>

 ##### 2번 ( setter 주입 )
```java
public class MemberService {
    private MemberRepository memberRepository;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
}
```
장점 : MemberRepository memberRepository 에 Mock() 같은걸 직접 주입 해 줄 수 있다, 유지보수가 용이하다.

단점 : 애플리케이션 로딩 시점에 이것을 수정할 일이 없다.

<br>

 ##### 3번 ( 생성자 injection을 사용 )
```java
public class MemberService {
    private MemberRepository memberRepository;
    // final로 하는 걸 권장. final을 하면 컴파일 시점에 부족한 부분 체크 가능
    @Autowired
    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
}
```
생성자에서 injection을 해준다.

장점 : 테스트케이스를 할때,
```java
public static void main(String args){
     MemberService memberService = new MemberService(); 
     // ()에 밑줄이 생긴다. Mock()을 주입을 하던 테스트 케이스를 주입을 해줘야한다.
     // "얘는 이게 필요해! 이거에 의존하고 있어" 라는걸 명확하게 알 수 있다.
}
```
최신버전 spring에서는  @Autowired가 없어도 자동으로 등록해줌.

<br>

##### 4번 ( lombok을 사용 )
```java
@AllArgsConstructor
public class MemberService {
    private final MemberRepository memberRepository;
}
```
@AllArgsConstructor 는 모든 필드를 가지고 생성자를 만들어 주는 것이다.

<br>

##### 5번 ( @RequiredArgsConstructor 사용 )
```java
@RequiredArgsConstructor
public class MemberService {
    private final MemberRepository memberRepository;
}
```
@RequiredArgsConstructor 는 final 을 가지고 있는 필드들만을 이용해서 생성자를 만들어준다. 그렇기 때문에 @AllArgsConstructor 와 좀 더 명확하게 구분해서 쓸 수 있다.

<br>

> @PersistenceContext

-  jpa가 제공하는 표준 어노테이션
- spring + jpa를 사용하면 @PersistenceContext 를 @Autowired로 사용할 수 있다.
- 즉, @RequiredArgsConstructor를 사용가능

<br>

 #### **section4 - 3 ( 회원 기능 테스트 )**
> test 프로젝트

```
src 밑에는 main과 test 프로젝트가 나눠져 있다. 둘 다 밑에 java 프로젝트가 있다.
test프로젝트 하위에 resources 프로젝트를 만들고 하위에 yaml 파일을 붙여넣으면 프로젝트 실행시 test프로젝트의 yaml 파일이 권한을 가지기 때문에 main의 yaml은 실행하지 않는다. test의 yaml에는 db를 메모리 모드로 동작하도록 하기 때문에 아무런 내용이 들어가 있지 않아도 자동으로 db연결없이 메모리 모드로 동작하게 해 준다.
```

<br>

> @Test(expected = IllegalStateException.class)

```java
   @Test
   public void 중복_회원_예외() throws Exception{      

       //given
       Member member1 = new Member();
       member1.setName("kim");

       Member member2 = new Member();
       member2.setName("kim");

       //when
       memberService.join(member1);
       try{
           memberService.join(member2);  
       } catch (IllegalStateException e){
           return;                     
       }

        //then
        fail("예외가 발생해야 한다.");
   }
```
@Test(expected = IllegalStateException.class)를 이용해 위 코드를 깔끔하게 할 수 있다.

<br>

```java
@Test(expected = IllegalStateException.class)
    public void 중복_회원_예외() throws Exception{        
        //given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");

        //when
        memberService.join(member1);
        memberService.join(member2);  

        //then                          
        fail("예외가 발생해야 한다.");
    }
```
catch를 test 어노테이션에서 expected 로 받아준 것이다.


<br>
<br>
 
 ## 08/06
 ### [ C#/.Net 7.0 WPF 강의 수강 1일차 ]
#### Wpf와 Windform의 차이
> WPF란?

- Windows Presentation Foundation
- Ui 프레임 워크이다.
- 벡터 기반의 렌더링 엔진을 사용하는 Ui 프레임워크
- [Cdoe-behind]
    - UI 디자인은 Xaml로 하고, 돌아가는 비지니스 로직은 코드딴에서 C# 베이스로 구동을 한다.

- 코드딴과 UI 딴이 구분되어 있기 때문에 유지보수가 용이하다.
- 벡터 기반의 그래픽 시스템 사용 -> 하이 퀄리티나 스케일러블한 그래픽들, 애니메이션 같은 것들을
	쉽게 적용할 수 있다.

> Winform란?

- 컨트롤 기반으로 운영
- 유지보수 힘듦
- 그래픽 시스템이나 애니메이션이 제한적이다.

```
즉,
대규모 프로젝트는 WPF를 사용하고,
소규모 프로젝트는 대체로 Winform을 사용한다.
``` 

<br>
<br>
 
 ## 08/07
 ### [ C#/.Net 7.0 WPF 강의 수강 2일차 ]
 #### [ObservableProperty]
- DI 요소 중 하나.
- 속성 코드(property)가 자동으로 제너레이트됨.
- 변수를 할당하고 그 변수를 트래킹하고 싶은 경우, 데이터 바인딩을 걸고 싶은 경우 혹은 뷰쪽에 전달하고 싶은 경우에 달아준다.
- 그렇게 해서 생성된 변수는 속성을 통해 접근할 수 있고, 데이터 바인딩 용도로 사용된다.

<br>

```
<.....Text="{Binding ViewModel.Text, Mode=TwoWay}" />
```
Ui에서 쌍방향 통신이면 Mode가 TwoWay.
단방향 통신이면 Mode가 OneWay.

<br>

#### [RelayCommand]
- 메서드를 view와 command로서 활용하지 않고, view 로직 클래스 내부적으로만 사용하겠다면 attribute를 적용하지 않아도 되지만
	내가 이 메서드를 command로서 활용하겠다면 (사용자로부터 어떤 이벤트가 발생하면 이 함수를 호출해서 특정 로직을 수행하고 싶다)면
	적용한다.

<br>

#### command

- 사용자가 발생시킨 동작과 view모델에 정의된 함수 두개를 연결시켜줘서 "어떤 사용자 인풋이 들어왔을 때 어떤 로직을 수행해라" 라고 알려주는 것이다.

- 이러한 연결을 해주는 것이 command 이다.

<br>

#### 단축키

> F12

원하는 부분 클릭 후 F12를 누르면 해당 부분의 정의가 있는 클래스로 이동한다.

<br>
<br>
 
 ## 08/08
 ### [ C#/.Net 7.0 WPF 강의 수강 3일차 ]

 #### IOC ( 제어의 역전 )
 - 컴포넌트간의 결합도를 줄인다.
- 각 컴포넌트간의 의존성을 클래스에 직접 참조하는것이 아니라 외부에서 가져오는 것
- 외부에서 주입받아서 인터페이스에서 제공하는 함수를 호출해서 갖다 쓰면 된다.

<br>

#### 단축키

> Alt + Enter

인터페이스 생성 단축키

<br>

> ctor

생성자 코드 조각 단축키

#### 서비스를 만들고 반드시 App.xaml의 비하인드 코드인 App.xaml.cs에 services로 등록을 해줘야 동작한다.
```
services.AddSingleton<만든인터페이스, 만든서비스 클래스>();
```

#### DB Set에 접근하는 방법

- 터미널을 연다.

```
PM> Scaffold-DbContext "Host=localhost;Database=데이터베이스이름;Username=myuser;Password=mypassword" Npgsql.EntityFrameworkCore.PostgreSQL -o Models
```
을 입력하면 접근 할 수 있다.

<br>

#### List\<T>

- Generic 타입 매개변수이다.

인터페이스에서 데이터를  조회하려는 로직을 만들려고 할때, 만약, 테이블의 모든 정보를 조회한다고 하면
```c#
List<region> regions { get; }
```
이라고 작성을 한다.

하지만 이렇게 하면 인터페이스를 여러 클래스에서 구현해야 하는데, 특정 테이블만을 사용하게 된다.

구현하는 클래스마다 다른 테이블을 사용할 수 있도록 하기 위해서
```c#
List<T>? Get();
```
라고 정의를 해 주어야 한다.

 <br>

 전체코드를 살펴보면

 ```c#
 public interface IDatabase<T>
{
    //  테이블에 대한 모든 데이터 조회
    List<T>? Get();

    //  테이블에 대해 특정 ID에 해당하는 데이터 조회
    T? GetDetail(int id);

    //  테이블에 특정 DATA Insert
    void Create(T entity);

    //  테이블에 특정 DATA Update
    void Update(T entity);

    //  테이블에 특정 DATA Delete
    void Delete(int id);
}
 ```
 이런 Interface가 나오게 되는데,
 
 만약 다른 클래스에서 region이라는 테이블을 사용하려고 이 Interface를 구현하면

 ```c#
 public class regionDatabase : IDatabase<region>
{
    public List<region>? Get()
    {
        // region 테이블에서 모든 데이터를 가져옴
    }

    public region? GetDetail(int id)
    {
        // 특정 id에 해당하는 region 가져옴
    }

    public void Create(region entity)
    {
        // 새로운 region 생성
    }

    public void Update(region entity)
    {
        // 기존 region 업데이트
    }

    public void Delete(int id)
    {
        // 특정 id에 해당하는 region 삭제
    }
}
 ```
 
 이렇게 \<T> 자리에 사용하려는 region 테이블을 넣어줘서 코드의 재사용성을 높일 수 있다.

<br>

#### ? ( Nullable )

기본적으로 c#에서는 Null 값을 가질 수 없다.

하지만 Null을 받을 수 있는 상황에서 ?를 넣어주면 Null 값을 가질 수 있게 된다.

- 참조 형식(Reference Type)은 기본적으로 null을 허용하기 때문에 굳이 ?를 사용하지 않아도 됨.
- 데이터베이스나 외부 소스로부터 값을 가져올 때 값이 없을 수 있는 상황을 안전하게 처리하는 데 유용

예)
```c#
private int? _counter = 0;
```
변수 _counter를 int 형식으로 받는다. 다만 값이 null이어도 허용한다는 뜻이다.


<br>
<br>

 ## 08/09
### [ C#/.Net 7.0 WPF 강의 수강 3일차 ]

#### 느슨한 결합

![Alt text](<화면 캡처 2024-08-10 014511.png>)

#### UI 공부

```c#
<Grid>
    <Grid.ColumnDefinitions>                // 컬럼 정의하기
        <ColumnDefinition Width="300" />    // 특정값으로 설정
        <ColumnDefinition Width="*" />      // 컬럼을 균등하게 나눔

    </Grid.ColumnDefinitions>               // 2개 컬럼이 정의됨

    <Grid Grid.Column="0">                  // 첫번째 컬럼 내용
        <Grid.RowDefinitions>               // 행 정의하기
            <RowDefinition Height="auto" /> // 사용할 행 갯수만큼
            <RowDefinition Height="auto" /> // 입력하기
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
        </Grid.RowDefinitions>
```


<br>

행 정의 하지 않고 
```c#
<StackPanel Orientation="Vertical">
    
</StackPanel>
```
여기 안에 내용을 넣어도 java의 Block 개념으로 수직으로 정렬된다.
inline으로 하려면 Horizontal 속성을 넣으면 된다.


<br>

```c#
        <ui:TextBlock Grid.Row="0" x:Name="tbkAdminAgency" Margin="0,0,0,10"> // Grid.Row는 위에서 정의한 행 순서
            <Run Text="Administrative Agency" />    // Margin의 순서는 좌,상,우,하
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ComboBox Grid.Row="1" x:Name="cbxAdminAgency" ItemsSource="{Binding ViewModel.Colors}" Margin="0,0,0,10"/>
        // ItemsSource는 바인딩할때 값
        <ui:TextBlock Grid.Row="2" x:Name="tbkTotalPopulation" Margin="0,0,0,10">
            <Run Text="Total Population" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="3" x:Name="tbxTotPopulation" Margin="0,0,0,10"/>

        <ui:TextBlock Grid.Row="4" x:Name="tbkMalePopulation" Margin="0,0,0,10">
            <Run Text="Male Population" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="5" x:Name="tbxMalePopulation" Margin="0,0,0,10"/>

        <ui:TextBlock Grid.Row="6" x:Name="tbkFemalePopulation" Margin="0,0,0,10">
            <Run Text="Female Population" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="7" x:Name="tbxFemalePopulation" Margin="0,0,0,10"/>

        <ui:TextBlock Grid.Row="8" x:Name="tbkSexRatio" Margin="0,0,0,10">
            <Run Text="Sex Ratio" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="9" x:Name="tbxSexRatio" Margin="0,0,0,10"/>

        <ui:TextBlock Grid.Row="10" x:Name="tbkNumberOfHouseholds" Margin="0,0,0,10">
            <Run Text="Number Of Households" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="11" x:Name="tbxNumberOfHouseholds" Margin="0,0,0,10"/>

        <ui:TextBlock Grid.Row="12" x:Name="tbkNumberOfPeoplePerHouseholds" Margin="0,0,0,10">
            <Run Text="Number Of People Per Households" />
            <Run Text="*" Foreground="OrangeRed"/>
        </ui:TextBlock>
        <ui:TextBox Grid.Row="13" x:Name="tbxNumberOfPeoplePerHouseholds" Margin="0,0,0,10"/>

        <ui:Button Grid.Row="14" Height="50" Width="300" Background="SteelBlue" Content="Create" Foreground="White" Margin="0,0,0,10"/>

        <Grid Grid.Row="15">        // 행 안에서 또다른 행을 생성할 수 있다.
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>

            <ui:Button Grid.Column="0" Height="50" Width="145" Content="Read All" Background="LightPink" Foreground="White" HorizontalAlignment="Center" Margin="0,0,0,10"/>        // HorizontalAlignment는 정렬.
            <ui:Button Grid.Column="1" Height="50" Width="145" Content="Read All" Background="LightPink" Foreground="White" HorizontalAlignment="Center" Margin="0,0,0,10"/>

        </Grid>

        <ui:Button Grid.Row="16" Height="50" Width="300" Background="ForestGreen" Content="Update" Foreground="White" Margin="0,0,0,10"/>

        <ui:Button Grid.Row="17" Height="50" Width="300" Background="Red" Content="Delete" Foreground="White" Margin="0,0,0,10"/>


    </Grid>

    <Grid Grid.Column="1">
        
    </Grid>
</Grid>
```

<br>

> 추가 정보

- x:Name 중복 안됨
- Alt로 다중 행에서 입력 가능