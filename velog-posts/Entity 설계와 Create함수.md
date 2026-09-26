<p>Entity를 설계한다는건 결국 DB와 1대1로 연동되는 클래스를 설계하는 일이다. Entity 클래스에는 해당 테이블에 넣는 행을 만드는 것이 create 함수이다. 테이블을 설계할때 필수로 작성하게 되는 함수이다. 아래는 내가 작성한 create함수인데, 함수를 작성하다가 궁금한 것들이 생겨서 회고록을 작성하게 되었다. </p>
<pre><code class="language-java">// DB에 추가할 유효한 객체 생성 -&gt; 실제 삽입은 Repository가 한다.
public static Rehearsal create(Team team, String title, LocalDateTime startsAt, LocalDateTime endsAt, String place, String memo) {
    Rehearsal rehearsal = new Rehearsal();
    rehearsal.team = team;
    rehearsal.title = title;
    rehearsal.startsAt = startsAt;
    rehearsal.endsAt = endsAt;
    rehearsal.place = place;
    rehearsal.memo = memo;
    return rehearsal;
}</code></pre>
<hr />
<h3 id="db혹은-시스템에서-설정하는-값">DB혹은 시스템에서 설정하는 값</h3>
<p>함수 생성시 매개변수로 받아오지 않는게 있는데 바로 시스템에서 설정하는 값들이다.</p>
<pre><code class="language-java">@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;</code></pre>
<p>다음과 같이 PK이면서 시스템(DB)에서 할당하는 값은 생성시 매개변수로 받아오지 않도록 해야한다. DB에서 관리하는 값이 아니더라도 시스템에서 관리하는 값도 여기에 포함된다.</p>
<pre><code class="language-java">@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
}</code></pre>
<p><strong><code>@EntityListeners</code> :</strong> JPA의 콜백을 외부 클래스에 위임하는 어노테이션이다. </p>
<p>createdAt과 같은 값은 시스템에서 설정해야 하는 값이기에 create() 호출시에 이를 매개변수로 받아올 필요가 없다. 시스템이 해당 값을 어떻게 채우는지 찾아보니 다음과 같은 순서로 이루어 지고 있었다. </p>
<ol>
<li><p>JPA가 특정 이벤트(저장)를 감지</p>
</li>
<li><p>JPA가 <strong><code>@EntityListeners</code></strong> 에 등록된 클래스의 코드를 읽어가며 JPA의 어노테이션(ex, @Prepersist)이 붙어있는 함수를 실행시킨다. 여기서 이벤트 리스너에 등록되는 클래스는 Spring Data JPA의 AuditingEntityLisener 클래스이다. </p>
<pre><code class="language-java"> // AuditingEntityLisener 클래스 내부 함수
 @PrePersist
 public void touchForCreate(Object target) {

     Assert.notNull(target, &quot;Entity must not be null&quot;);

     if (handler != null) {

         AuditingHandler object = handler.getObject();
         if (object != null) {
             object.markCreated(target);
         }
     }
 }</code></pre>
</li>
<li><p>해당 함수는 엔티티를 훑어보며 <strong><code>@CreatedDate</code></strong> 어노테이션이 붙은 필드를 찾고 그 값을 변경한다.</p>
</li>
</ol>
<p>JPA와 Spring Data JPA를 적절히 활용하여서 <strong><code>@CreatedDate</code></strong> 와 같은 어노테이션이 동작한다. 개발자가 직접 <strong><code>@PrePersist</code></strong> 어노테이션을 통해서 함수를 만들어야 하는 불편함을 해결해 주었다. </p>
<hr />
<h3 id="생성시점-고정되는-필드값">생성시점 고정되는 필드값</h3>
<p>매개변수로 받지 않고 생성시점에는 그 값이 고정되는 필드 또한 매개변수로 전달해줄 필요가 없다. 그렇기에 함수 설계 단계에서 필드값이 생성시점에 고정되는 값인지도 고려해 봐야 한다. 예시 코드는 다음과 같다. </p>
<pre><code class="language-java">public static TeamMember create(Team team, TeamRole role, Instrument instrument) {
    TeamMember member = new TeamMember();
    member.team = team;
    member.role = role;
    member.instrument = instrument;
    member.status = MemberStatus.ACTIVE;      // ← 파라미터로 안 받음. 항상 ACTIVE로 고정
    member.joinedAt = LocalDateTime.now();    // ← 파라미터로 안 받음. 항상 &quot;지금&quot;으로 고정
    return member;
}</code></pre>
<p>그 외의 필드값들은 매개변수로 받아와서 설정해야 한다.</p>
<hr />
<h3 id="회고마무리">회고마무리</h3>
<p>create 함수 만들때 왜 만들어야 하는지 매개변수는 뭘 기준으로 작성해야 하는지 모르는 부분이 많아서 이렇게 회고로 정리했다.</p>