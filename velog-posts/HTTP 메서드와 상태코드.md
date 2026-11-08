<h3 id="http-메서드-상태코드-헤더">HTTP 메서드, 상태코드, 헤더</h3>
<p>HTTP는 <strong>HyperText Transfer Protocol</strong>의 약자로, 웹에서 클라이언트와 서버가 데이터를 주고받을 때 쓰는 통신 규칙이다. URL은 “어디에 무엇을” 가리키는 주소이고, HTTP는 “어떻게 주고받을지”를 정한 규칙이다. 무상태(stateless) 성질을 가지는데 이는 서버가 이전 요청을 기억하지 못함을 의미한다.</p>
<p><strong>HyperText가 붙은 이유</strong>: 처음엔 하이퍼텍스트(링크로 연결된 문서, 즉 HTML)를 주고받으려고 만들어졌다. 지금은 HTML뿐 아니라 이미지, 영상, JSON 데이터까지 거의 모든 걸 실어 나르기 때문에 백엔드 개발자가 만드는 API도 대부분 HTTP 위에서 JSON을 주고받는 방식이다.</p>
<p><strong>Protocol이 붙은 이유</strong>: 두 사람이 대화를 하려면 같은 언어를 사용해야 한다. 컴퓨터 끼리도 요청은 이래야 하고 응답은 이래야 한다는 형식을 미리 정해둬야 서로 알아들을 수 있다. 그 약속이 프로토콜이고, 웹에서 쓰는 약속이 HTTP이다.</p>
<p><strong>요청(Request) 양식</strong></p>
<pre><code>[메서드] [경로]?[쿼리키]=[쿼리값]&amp;[쿼리키]=[쿼리값] [HTTP버전]
[헤더이름]: [헤더값]
[헤더이름]: [헤더값]
[헤더이름]: [헤더값]
(빈 줄)
[바디]

GET /posts?keyword=spring&amp;page=2 HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.abc.def</code></pre><p><strong>응답(Response) 양식</strong></p>
<pre><code>[메서드] [경로]?[쿼리키]=[쿼리값]&amp;[쿼리키]=[쿼리값] [HTTP버전]
[헤더이름]: [헤더값]
[헤더이름]: [헤더값]
[헤더이름]: [헤더값]
(빈 줄)
[바디]

HTTP/1.1 200 OK
Content-Type: application/json

{&quot;content&quot;: [{&quot;id&quot;: 11, &quot;title&quot;: &quot;스프링 입문&quot;}], &quot;page&quot;: 2}</code></pre><p><strong>HTTP 메서드</strong> </p>
<p>HTTP 메서드는 맨 앞에 오는 단어로 해당 자원에 어떤 행위를 하고 싶은지 서버에 알려주는 역할을 한다. </p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>의미</th>
<th>바디</th>
<th>성공 시 응답</th>
</tr>
</thead>
<tbody><tr>
<td>GET</td>
<td>조회</td>
<td>없음</td>
<td>200</td>
</tr>
<tr>
<td>POST</td>
<td>생성 (또는 처리 요청)</td>
<td>있음</td>
<td>201</td>
</tr>
<tr>
<td>PUT</td>
<td>전체 교체</td>
<td>있음</td>
<td>200, 204</td>
</tr>
<tr>
<td>PATCH</td>
<td>부분 수정</td>
<td>있음</td>
<td>200, 204</td>
</tr>
<tr>
<td>DELETE</td>
<td>삭제</td>
<td>없음</td>
<td>204</td>
</tr>
<tr>
<td>- <strong>GET</strong>: 서버 데이터를 읽기만 해요. 조건은 경로나 쿼리로 보내요. (<code>GET /posts/1</code>)</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- <strong>POST</strong>: 새 자원을 만들어요. 새 자원의 ID는 서버가 정하기 때문에 경로에 ID가 없어요. (<code>POST /posts</code>) 로그인처럼 CRUD에 안 맞는 &quot;처리 요청&quot;에도 써요.</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- <strong>PUT</strong>: 자원을 <strong>통째로 교체</strong>해요. 바디에 모든 필드를 담아야 하고, 빠진 필드는 비워져요.</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- <strong>PATCH</strong>: 자원의 <strong>일부 필드만</strong> 수정해요. 바디에 바꿀 필드만 담아요.</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- <strong>DELETE</strong>: 자원을 삭제해요.</td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody></table>
<p><strong>Spring 매핑</strong></p>
<pre><code class="language-java">@GetMapping(&quot;/posts/{id}&quot;)      // GET
@PostMapping(&quot;/posts&quot;)          // POST
@PutMapping(&quot;/posts/{id}&quot;)      // PUT
@PatchMapping(&quot;/posts/{id}&quot;)    // PATCH
@DeleteMapping(&quot;/posts/{id}&quot;)   // DELETE</code></pre>
<p><strong>HTTP 요청에서 서버로 데이터를 보내는 4가지 통로</strong></p>
<ol>
<li><p>예시 요청</p>
<pre><code> PATCH /posts/42?notify=false HTTP/1.1
 Host: example.com
 Authorization: Bearer eyJhbGci...
 Content-Type: application/json

 {&quot;title&quot;: &quot;제목만 수정&quot;}</code></pre></li>
<li><p>어노테이션</p>
</li>
</ol>
<pre><code>| 위치 | 이 요청에서의 값 | 꺼내는 어노테이션 |
| --- | --- | --- |
| 경로 | `42` | `@PathVariable` |
| 쿼리 | `notify=false` | `@RequestParam` |
| 헤더 | `Authorization: Bearer ...` | `@RequestHeader` |
| 바디 | `{&quot;title&quot;: &quot;제목만 수정&quot;}` | `@RequestBody` |

**경로 (`@PathVariable`)**: “어떤 자원인가”를 특정할 때. `/posts/42`의 `42`처럼 자원의 ID를 넣어요. 값이 없으면 URL 자체가 성립하지 않아서 항상 필수예요.

**쿼리 (`@RequestParam`)**: “어떻게 걸러서/어떤 옵션으로 처리할까”를 정할 때. 검색어, 정렬, 페이징, 부가 옵션이 대표적이고, 선택 값이 많아서 `required = false`나 `defaultValue`를 자주 써요.

**헤더 (`@RequestHeader`)**: 요청 자체에 대한 부가 정보를 담을 때. 인증 토큰, 바디의 형식(`Content-Type`)처럼 데이터 본문은 아니지만 서버가 알아야 하는 정보예요. 다만 `Authorization`이나 `Cookie`는 Spring Security나 필터가 대신 처리하는 경우가 많아서, 컨트롤러에서 직접 꺼내는 일은 생각보다 드물어요.

**바디 (`@RequestBody`):** 저장하거나 변경할 실제 내용을 담을 때. JSON을 자바 객체로 변환해서 받아요(Jackson이 담당). 비밀번호처럼 URL에 남으면 안 되는 민감한 값도 여기에 넣어요.</code></pre><ol start="3">
<li><p>컨트롤러</p>
<pre><code> @PatchMapping(&quot;/posts/{id}&quot;)
 public PostResponse update(
     @PathVariable Long id,                            // 경로의 {id} → 42
     @RequestParam(defaultValue = &quot;true&quot;) boolean notify,  // 쿼리 → false
     @RequestHeader(&quot;Authorization&quot;) String token,     // 헤더 → &quot;Bearer eyJ...&quot;
     @RequestBody PostPatchRequest req                 // 바디 → 객체로 변환
 ) { ... }</code></pre></li>
</ol>
<hr />
<h3 id="쿠키세션">쿠키/세션</h3>
<p>HTTP는 무상태(stateless)라서 서버는 이전 요청을 기억하지 못해요. </p>
<pre><code>1. POST /login   → 서버: &quot;로그인 성공!&quot;
2. GET  /mypage  → 서버: &quot;...누구세요?&quot;</code></pre><p>쿠키/세션은 <strong>“HTTP는 기억력이 없다”는 문제를 푸는 도구 한 쌍</strong>이다.</p>
<p><strong>쿠키:</strong> 서버가 브라우저에 주는 데이터로 “이거 갖고 있다가 다음에 보여줘”라고 맡기는 작은 데이터이다. 쿠키를 통해서 HTTP의 무상태 성질로 인해서 발생하는 문제를 다음과 같이 해결할 수 있다. </p>
<pre><code>서버 응답:      Set-Cookie: JSESSIONID=abc123     ← 저장해둬
이후 모든 요청: Cookie: JSESSIONID=abc123          ← 브라우저가 자동 첨부</code></pre><p>쿠키는 저장과 전송을 브라우저에서 담당하며, 사용자가 값을 마음대로 수정할 수 있다. 중요한 정보를 직접 넣으면 안된다. 중간에 조작 탈취가 가능하기 때문이다. </p>
<p><strong>세션</strong>: 서버에 저장되는 사용자 정보. 중요한 사용자 정보는 서버가 보관하고, 쿠키로는 해당 정보를 찾는 열쇠 (세션 ID)만 준다.</p>
<hr />