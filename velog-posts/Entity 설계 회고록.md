<p>본 글은 AI가 아닌 직접 작성한 글로 D&amp;X Confrenece 개발과정에서 Entity 설계에 대한 회고를 담고 있습니다. 먼저 Entity란 무엇인지에 대해서 설술하면 다음과 같습니다. </p>
<p><strong>JPA Entity</strong> : 데이터베이스의 테이블을 자바 클래스로 매핑한 것을 의미한다. 엔티티 클래스는 데이터베이스 테이블의 각행을 표현하며, 클래스의 인스터스가 해당 테이블의 한 행을 의미하게 된다. </p>
<p>데이터 베이스와 쿼리로 직접 소통하는게 아닌 이와 같이 Entity를 설계하여 활용하는 이유는 자바로 작성된 객체지향 코드와 관계형 데이터베이스 사이의 구조차이 때문이다. 자바는 객체(필드 + 메서드), 상속, 참조 같은 개념으로 데이터를 다루며. 관계형 데이터베이스는 테이블, 행, 외래키, 같은 개념으로 데이터를 다룬다. </p>
<p>이러한 패러다임의 불일치를 해결하는데 도움이 되는게 바로 JPA Entity이다. </p>
<p>데이터베이스에 대해서 잘 이해하고 있다면 JPA를 통해서 테이블을 어떻게 클래스로 정의하는지 그와 관련된 어노테이션에 대해서 알아보면 도움이 된다. </p>
<hr />
<h3 id="team클래스와-entity-어노테이션">Team클래스와 Entity 어노테이션</h3>
<pre><code class="language-java">package com.moyeorock.domain.team.entity;

import com.moyeorock.global.common.entity.BaseEntity;
import com.moyeorock.global.common.enums.Region;
import com.moyeorock.domain.team.enums.TeamStatus;
// import com.moyeorock.domain.performance.entity.Performance;
// → 4팀의 Performance 엔티티가 아직 없어서 주석 처리. @ManyToOne 필드 자체는 미리 적어 두고,
//   Performance 클래스가 생기면 이 import와 아래 필드의 주석만 풀면 되게 해뒀다.
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@Table(name = &quot;teams&quot;)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Team extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // --- performance 연관관계는 Performance 엔티티가 없어서 임시로 주석 처리 ---
    // @ManyToOne(fetch = LAZY): architecture.md §5 &quot;연관관계는 전부 LAZY&quot; 규칙.
    //   EAGER로 두면 Team을 조회할 때마다 필요 없어도 performances를 매번 조인해서 가져온다.
    // @JoinColumn(name = &quot;performance_id&quot;): FK 컬럼명을 ERD와 맞춤. nullable 기본값 true라
    //   &quot;NULL = 독립 팀&quot;이라는 ERD 규칙을 그대로 만족한다(따로 nullable=false 안 붙임).
    // @ManyToOne(fetch = FetchType.LAZY)
    // @JoinColumn(name = &quot;performance_id&quot;)
    // private Performance performance;

    @Column(nullable = false, length = 50)
    private String name;

    @Column(columnDefinition = &quot;TEXT&quot;)
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(length = 50)
    private Region region;

    @Enumerated(EnumType.STRING)
    @Column(length = 10, nullable = false)
    private TeamStatus status;

    public static Team create(String name, Region region) {
        Team team = new Team();
        team.name = name;
        team.region = region;
        team.status = TeamStatus.ACTIVE;
        return team;
    }

    public void disband() {
        this.status = TeamStatus.DISBANDED;
    }
}</code></pre>
<p>아직은 ERD 명세서를 보고 AI의 도움없이 코드를 혼자서 작성하기에는 무리가 있는거 같았기에 해당 코드는 클로드의 힘을 빌려서 작성했다. </p>
<p>먼저 코드 상단 불러온 라이브러리를 보면 크게 두가지 라이브러리인 lombok.<em>과 jakarta.persistence.</em>이 있다. 하나 하나씩 자세하게 봐보자. </p>
<hr />
<h3 id="lombok---보일러플레이트-코드를-자동-생성해주는-라이브러리">Lombok - 보일러플레이트 코드를 자동 생성해주는 라이브러리</h3>
<p><strong>보일러플레이트 코드(boilerplate code)</strong> : 매번 거의 똑같이 반복해서 써야 하는데, 그 자체로는 프로그램의 핵심 로직과 상관없는 정형화된 코드로서 대표적으로 Getter, Setter가 있다. </p>
<p>Lombok의 핵심은 생성자, Getter, Setter자와 같은걸 전부 손으로 써야하는 불편함을 덜어주는 기능을 담고 있는 라이브러리로 어노테이션을 붙이면 컴파일 시점에 해당 코드를 자동으로 생성해주는 라이브러리이다. </p>
<p><strong><code>@Getter</code></strong> : 클래스에 붙이면 그 클래스의 모든 필드에 대해서 getter 메서드를 자동 생성한다. 위 테이블을 기준으로는 실제 코드에는 없지만 컴파일 이후에 다음과 같은 코드가 붙게된다. </p>
<pre><code class="language-java">public Long getId() { return this.id; }
public String getName() { return this.name; }
public String getDescription() { return this.description; }
public Region getRegion() { return this.region; }
public TeamStatus getStatus() { return this.status; }</code></pre>
<p><strong><code>@NoArgsConstructor(access = AccessLevel.PROTECTED)</code></strong> : 해당 어노테이션은 파라미터가 없는 기본 생성자를 자동으로 만들어준다. 해당 어노테이션이 필요한 이유한 이유는 JPA 자체가 인자 없는 생성자를 반드시 가져야 한다고 강제하기 때문이다. </p>
<p>Hibernate 프레임 워크는 엔티티들이 만들어지기 전에 컴파일 되는데 Hibernate는 이런 클래스(엔티티)들을 다뤄야 한다. 라이브러리가 만들어지는 단계에서 후에 생성될 도메인을 알 수 없으니 리플렉션 방법을 사용한다. </p>
<p>리플렉션으로 인자 있는 생성자도 호출할 수는 있지만, 그러려면 클래스마다 다른 인자 정보를 Hibernate가 알아야 하므로 모든 엔티티에 통일되게 적용할 방법이 없다. 이를 해결하기 위해 인자 0개라는, 모든 클래스에 대해 동일하게 적용 가능한 유일한 생성자 형태를 강제한다.</p>
<p>보일러플레이트 코드가 뭐냐는 질문에 머리가 하얘졌었는데, getter와 setter와 같이 정형화된 코드를 의미한다는걸 이번 회고록작성 과정에서 알게 되었다. 이름이 왜 보일러플레이트인지 궁금해서 찾아보니 20세기 증기 보일러를 만들때, 규격화 되어서 매번 새로 설계할 필요 없는 부품이 바로 보일러플레이트 였다고 한다. 이러한 특징이 현대로 넘어와서 형식적인 문구나 정형화된 코드를 의미하게 된거라고 한다. </p>
<hr />
<h3 id="jakartapersistence-라이브러리">Jakarta.persistence 라이브러리</h3>
<p><strong>Jakarta.persistence</strong> : JPA(Jakarta Persistence API) 라는 표준 스펙을 정의 해놓은 패키지이다. 라이브러리라고 부르지만 실제는 특정 어노테이션이 특정 의미로 동작하게하는 규격 즉, 인터페이스의 모음을 담고있다. 그리고 이를 실제로 동작하게 만든게 Hibernate가 맡는다. 가끔 가다가 JPA와 Hibernate가 Spring Boot에 속한 기능이 아니라 Spring Boot와 독립적으로 존재하는 자바 생태계 기술이다. Spring Boot는 단순히 JPA를 가져다 쓰기 편하게 통합 해주는 역할이다.</p>
<p>어려운 용어가 많이 등장하니, 용어 들을 간단하게 정리해보자면 다음과 같다. 우선 ORM에 대해서 얘기를 하고 시작해보자. </p>
<p><strong><code>ORM(Object-Relational Mapping)</code></strong> → 객체 지향 프로그래밍 언어에서 사용하는 객체와 관계형 데이터베이스의 테이블 간의 데이터를 자동으로 변환해 주는 기술 또는 그 기술을 구현한 라이브러리를 뜬한다. </p>
<p>전통적으로 SQL문을 직접 작성하여 DBMS와 통신하던 방식을 대신하여 개발자가 사용하는 프로그래밍 언어의 객체를 통해 데이터베이스를 조작할 수 있도록 도와주는 라이브러리 이다. </p>
<p>ORM을 사용할때의 장단점이 명확한데 우선 장점으로는 SQL을 직접 작성하지 않아도 되기 때문에 개발 속도 향상 코드 가독성과 유지보수성의 향상, 객체 지향적인 코드 구조 유지 가능이라는 장점이 있다.</p>
<p>ORM이 만은은 아니기 때문에 복잡한 쿼리를 ORM으로만 처리하려다 보면 성능 저하나 예측 불가능한 SQL이 생성될 수도 있다. 단점으로는 복잡한 집계, 다중 조인 등은 결국 SQL로 작성하는 것이 더 효율적인 경우 존재, 잘못된 설정으로 N + 1 쿼리 문제 발생 가능, 런타임 성능 오버헤드가 존재한다. </p>
<p>장단점이 명확한 만큼 대규모 트래픽이나 고성능이 필요한 시스템에서는 ORM + Native SQL 병행 전략을 채탱하는 경우가 더 많다.</p>
<p><strong><code>jakarta.persistence</code></strong> →ORM 표준 명세(어노테이션, 인터페이스)</p>
<p><strong><code>Hibernate</code></strong> → JPA 구현체(규칙을 실제 SQL, 리플렉션, 영속성 컨텍스트로 구현한 제품)</p>
<p>내가 자주 혼동했던게 있는데 JPA와 Spring Data JPA를 같다고 생각하는거다. JPA는 표준 명세이고 Spring Data JPA는 Repository 인터페이스만으로 CRUD를 자동으로 구현해주는 라이브러리이다. 즉 다음과 같은 구조인거다. Spring Data JPA에 대해서는 나중에 Repository를 설계하면서 공부해 봐야 겠다.</p>
<pre><code class="language-json">Spring Boot (자동 설정 + 편의 기능)
  └─ Spring Data JPA (Repository 인터페이스만으로 CRUD 자동 구현)
       └─ JPA (ORM 표준 명세: @Entity, EntityManager 등)
            └─ Hibernate (JPA 명세의 실제 구현체: 리플렉션, SQL 생성, 캐싱 등)
                 └─ JDBC (실제 DB와 통신하는 저수준 표준)</code></pre>
<p><strong><code>@Entity</code></strong> : 클래스는 DB 테이블과 매핑되는 개체임을 JPA에게 알려주는 어노테이션이다. </p>
<p><strong><code>@Table(name = teams)</code></strong> : 매핑될 테이블 이름을 설정하고 이를 설정하지 않으면 클래스명을 기본값으로 한다. </p>
<p><strong><code>@Id</code></strong> : 해당 필드값이 기본키(PK)임을 알려준다. </p>
<p><strong><code>@GenratedValue(strategy = GerataionType.IDENTITY)</code></strong> : PK 값을 프로그래머가 직접 생성하여 넣는게 아닌 DB의 auto_increment 기능에 맡긴다는 의미이다. </p>
<p><strong><code>@Column(nullable = false, length = 50)</code></strong> : 클래스의 필드가 매핑될 컬럼의 세부 조건을 지정한다. 소괄호 안에 함수에 인자를 넘겨주듯 설정을 할 수 있는데, NOT NULL제약을 의미하며 컬럼 타입을 TEXT로 지정한다. </p>
<p><strong><code>@Column(columnDefinition = &quot;TEXT&quot;)</code></strong> : 컬럼 타입을 직접 TEXT로 지정하는 거다. </p>
<p><strong><code>@Enumerated(EnumType.String)</code></strong> : enum 타입 필드를 DB에 어떻게 저장할지 지정. EnumType.STRING(enum 이름을 문자열 그대로 저장.)</p>
<hr />
<h3 id="회고를-마치며">회고를 마치며</h3>
<p>생각보다 간단한 코드임에도 공부할게 많다는걸 세삼 느꼈다. 특히 JPA와 Spring Data JPA 라이브러리의 차이점 그리고 Lombok 라이브러리의 보일러 플레이트 코드들 말이다. 각각의 어노테이션이 무슨 기능을 하게 되는지는 차차 코드 설계를 하면서 자연스럽게 알아가게 될거 같다. </p>
<p>내 첫 회고이니 부족하더라도 재미있게 읽어주면 좋겠다. 이상!!</p>