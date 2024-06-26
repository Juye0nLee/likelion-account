✔️기능
==========
- ✨ 회원가입 기능
  - 이메일
    - 이메일의 비워져있음 여부 ✔️
    - 이메일 형식 검증 ✔️
  - 전화번호
    - 비워져있음 여부 ✔️
    - 전화번호 형식 검증 (ex. 01012345678)✔️
  - 아이디
    - 비워져있음 여부✔️
  - 비밀번호
    - 비워져있음 여부✔️
  - 이상 없으면 회원가입 ✔️
- ✨ 로그인 기능
  - 존재하는 회원인지✔️
  - 비밀번호가 일치하는지 ✔️
  - 이상 없으면 로그인 ✔️
- ✨ 회원탈퇴 기능
  - 쿼리 파라미터로 userId를 보내면 탈퇴가 되도록 ✔️
  - 해당 userId가 존재하지 않는 경우 응답 구현 ✔️
 
    
회원가입 기능 구현
=====================
- ## 📚 공부
  - 전화번호 형식 검증
    - @Pattern : 형식을 검증하는 어노테이션
    - ```@Pattern(regexp = "\\d{3}\\d{4}\\d{4}",message = "전화번호 형식을 맞춰주세요")```
  - @EnableJpaAuditing
    - 데이터를 생성하거나 수정할 때, 누가, 언제했는지를 알기 위해서 보통은
      ``` createAt(생성일자), modifiedAt(수정일자), createdBy(생성자), modifiedBy(수정자)```
      이런 필드를 각 테이블에 같이 포함시켜서 테이블을 생성한다.
    - 하지만 매번 이렇게 코드를 작성하는 것은 귀찮고 번거로움!
    -  **<u>@EnableJpaAuditing 어노테이션을 활용하면 엔티티 객체가 생성이 되거나 변경이 되었을 때 자동으로 값을 등록할 수 있다. </u>**
    - 참고 : <https://velog.io/@kdohyeon/Spring-EnableJpaAuditing>
- ## 🐛 이슈
  - ### 프로젝트를 중지 했다가 다시 실행하면 ID가 50씩 증가되었다
    - Done (2024.05.11)
      - 기본키를 생성해주는 **@GeneratedValue**에는 여러개의 전략이 존재함
      - 대표적으로는 IDENTITY와 SEQUENCE
      - IDENTITY : 기본 키 생성을 DB에 위임하는 전략 -> **<u> 근데 이걸로 하면 500 error가 발생했음 왜지?</u>**
      - SEQUENCE :Sequence를 사용하여 기본키를 할당하는 방식
      ```
      @Entity
      @SequenceGenerator(
      name = "ID", //식별자 생성기 이름
      sequenceName = "ID_SEQ", //DB에 등록되어 있는 Sequence 이름
      initialValue = 1, //처음 시작 value 설정
      allocationSize = 1 //Sequence 한번 호출 시 증가하는 수
      //allocationSize가 기본값이 50이므로 1로 설정하지 않을 시, sequence 호출 시 마다 50씩 증가 -> 내가 마주한 이슈
      )
      @Table(name="MEMBERS")
      @Builder
      public class Account extends BaseEntity
      {
      @Id
      //기본키 생성
      @GeneratedValue(
      strategy = GenerationType.SEQUENCE,
      generator = "ID"
      )
      @Column(name="ID")
      private Long id;
      ```
        Entity를 생성할 때 @SequenceGenerator를 사용
        @SequenceGenerator에는 다양한 속성이 있는데
        그 중에서 **allocationSize**가 기본값이 50이기 때문에 해당 속성을 1로 설정하지 않을 시, sequence 호출 시마다 50씩 증가된 것 이었다.
    - 참조 : <https://velog.io/@gillog/JPA-기본-키-생성-전략IDENTITY-SEQUENCE-TABLE>
  - ### createAt이 null로 나옴
    - Done(2024.05.16)
      - AccountApplication 클래스에 **<u>@EnableJpaAuditing</u>** 어노테이션을 달았더니 해결

✨ 로그인 기능 구현
==================
- ## 📚 공부
  - @EnableJpaAuditing
    - 데이터를 생성하거나 수정할 때, 누가, 언제했는지를 알기 위해서 보통은
      ``` createAt(생성일자), modifiedAt(수정일자), createdBy(생성자), modifiedBy(수정자)```
      이런 필드를 각 테이블에 같이 포함시켜서 테이블을 생성한다.
    - 하지만 매번 이렇게 코드를 작성하는 것은 귀찮고 번거로움!
    -  **<u>@EnableJpaAuditing 어노테이션을 활용하면 엔티티 객체가 생성이 되거나 변경이 되었을 때 자동으로 값을 등록할 수 있다. </u>**
    - 참고 : <https://velog.io/@kdohyeon/Spring-EnableJpaAuditing>
 - ## 🐛 이슈
   - ### createAt이 null로 나옴
     - Done (2024.05.16)
       - AccountApplication 클래스에 **<u>@EnableJpaAuditing</u>** 어노테이션을 달았더니 해결

✨ 회원탈퇴 기능 구현 
=====================
- ## 📚 공부
  - ### JpaRepository
    - 내일 정리할래..졸려..
- ## 🐛 이슈
  - ### 400 BAD_REQUEST 오류 발생
    - Done (2024.05.12)
      - ***요청 보낼 데이터를 @RequestBody로 Body로 넣어서 보내려고 했기 때문에*** 발생한 오류
      - HTTP DELETE 메소드는 Request Body를 지원하지 않음
      - 주로 **Path나 쿼리 파라미터**를 통해 요청 데이터 전달
      - <u>@RequestParam</u>로 바꿔서 요청 URL를 ```http://localhost:8080/api/member/withdraw?userId=1``` 이렇게 했더니 해결👏🏻👏🏻
      - <u>@PathVariable</u>로도 바꿔서 요청 URL를 ```http://localhost:8080/api/member/withdraw/1``` 이렇게 했을 때도 성공 👏🏻👏🏻
