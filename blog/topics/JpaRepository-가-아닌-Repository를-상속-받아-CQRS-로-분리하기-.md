# JpaRepository 가 아닌 Repository를 상속 받아 CQRS 로 분리하기 


    @Repository
    public interface MemberRepository extends JpaRepository<Member,Long>, CustomMemberRepository {
    }

위같이 흔히 볼 수 있는 JpaRepository + CustomRepository 조합에서

데이터베이스는 하나지만 왠지 모르게 저 인터페이스를 CQRS로 분류하고 싶은 순간이 다들 한 번쯤 올 것이다.(아님 말고)

![](35a3fbf3-e494-46af-a5c8-7613e6a365b0.png)

하고싶으면 할꺼야

CQRS의 Command를 담당할 Repository의 이름을 MemberCommandRepository로

Query를 담당할 Repository의 이름을 MemberQueryRepository라고 한다면

어떻게 구성할수있을까?

    @Repository
    public interface MemberQueryRepository extends JpaRepository<Member,Long>, CustomMemberRepository {
    
    }
    @Repository
    public interface MemberCommandRepository extends JpaRepository<Member,Long> {
    
    }

가장 쉽게 생각해 보면 위처럼 할 수도 있을 것이다.

하지만 위처럼 구성하면 둘 다 JPARepository를 상속받았기에

QueryRepository에서 save나 update와 같은 쓰기 기능이 가능해지고

CommandRepository에서 findAll이나 findById와 같은 읽기 기능이 가능해지기에 CQRS로 분류가 된 것으로는 보이지 않는다.

[인프런에도 위와 같은 내용에 질문이 있었는데](https://www.inflearn.com/questions/201480/cqrs-리포지토리-질문)

![](39859c4e-6996-4d25-b290-1ff05eaba8c4.png)

CQRS로 Repository를 완전히 분리하지 않고

JPARepository와 QueryRepository로 분류하는 것도 좋은 방법이라고 답을 주신 것 같다.

    @Repository
    public interface MemberQueryRepository {}
    
    @Repository
    public interface MemberRepository extends JpaRepository<Member,Long> {}

이렇게 간단한 분리로도 기본적인 단순 쿼리와 복잡한 비즈니스 쿼리가 분리되는 느낌은 받았다.

하지만 내가 원하는 것은 

CommandRepository 에는 변경 기능만.

QueryRepository에는 조회 기능만 제공하도록 분리하고 싶다.

JPARepository를 상속받은 Repository 하나로는 분류가 안되기에

포트와 어댑터 패턴으로 Command 와 Query의 분류가 가능하다

    public interface MemberCommandRepository {
    
      Member save(Member member);
      void delete(Member member);
    }
    
    public interface MemberQueryRepository {
    
      Optional<Member> findById(Long id);
      List<Member> findAll();
    }

위처럼 Command, Query 포트 인터페이스를 작성하고

    @RequiredArgsConstructor
    public class MemberRepositoryImpl implements MemberQueryRepository, MemberCommandRepository{
    
      private final MemberRepository memberRepository;
      @Override
      public Member save(Member member) {
        return memberRepository.save(member);
      }
    
      @Override
      public void delete(Member member) {
        memberRepository.delete(member);
      }
    
      @Override
      public Optional<Member> findById(Long id) {
        return memberRepository.findById(id);
      }
    
      @Override
      public List<Member> findAll() {
        return memberRepository.findAll();
      }
    }

JPARepository로 어댑터를 구현한다.

분류 전의 Repository를 사용할 때는 JpaRepository에서 제공되는 모든 기능에 대해 열려있었으나

![](8a44e039-e5b5-4f43-a97a-8f84cfade2a6.png)

분류를 한 후에는 포트에서 제공되는 필요한 기능만 열려있다.

![](55422bab-689b-4f79-9474-0e21bc205de5.png)

이 방법 말고도 JpaRepository가 아닌 Repository를 상속받는 것으로도 가능하다.

    public interface MemberQueryRepository extends Repository<Member, Long> {
    
      Optional<Member> findById(Long id);
      List<Member> findAll();
    }
    
    public interface MemberCommandRepository extends Repository<Member, Long> {
    
      Member save(Member member);
      void delete(Member member);
    }