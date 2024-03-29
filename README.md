# JPA #
### ORM(Object-Relational Mapping) ###
- 객체(Object)와 관계형 데이터(Relational data)를 매핑하기 위한 기술이다. 
- 쉽게 말해 객체와 테이블을 매핑시켜주는 프레임워크이다.
- ORM 기술에 대한 표준 명세가 바로 JPA이다.
- JPA 표준을 구현한 대표적인 프레임워크가 Hibernate이다.

### Hibernate ###
- JPA를 사용하기 위해서 JPA를 구현한 ORM 프레임워크 중 하나이다. ORM 프레임워크 중 가장 많이 사용되며, JPA 인터페이스의 실제 구현부를 담당한다.

### SQL Mapper vs ORM ###
- SQL Mapper는 모두가 흔히 알고있는 Mybatis의 프레임워크다. SQL Mapper는 SQL문으로 직접 DB를 조작한다.
- ORM은 객체와 DB의 데이터를 자동으로 매핑해주기 때문에 직접 SQL문을 작성하지 않는다.

### JPA 프록시(Proxy)란 ###
- 프록시: 가짜 객체
- 프록시는 JPA 하이버네이트 뿐만 아니라 스프링에서도 초기화를 지연시키거나, 트랜잭션을 적용하는 부가 기능을 추가할 때도 프록시 기술을 사용한다.
- 지연 로딩을 하려면 연관된 엔티티의 실제 데이터가 필요할 때 까지 조회를 미루어야 한다.
- 하이버네이트는 지연 로딩을 사용하는 연관관계 자리에 프록시 객체를 주입하여 실제 객체가 들어있는 것처럼 동작하도록 한다. 
- 덕분에 우리는 연관관계 자리에 프록시 객체가 들어있든 실제 객체가 들어있든 신경쓰지 않고 사용할 수 있다. 
- 참고로 프록시 객체는 지연 로딩을 사용하는 것 외에도 em.getReference를 호출하여 프록시를 호출할 수도 있다.

### 영속성 컨텍스트란? ###
- 엔티티를 영구저장하는 환경
- 애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 데이터베이스 같은 역할을 한다. 
- EntityManager를 통해 엔티티를 저장하거나 조회하면 EntityManager는 **영속성 컨텍스트**에 엔티티를 보관하고 관리한다.

````java
EntityManager.persist(entity);
````

영속성 국어사전 뜻: 영원히 계속되는 성질이나 능력 from naver 사전

### EntityManagerFactory와 EntityManager ###
- EntityManagerFactory는 여러 스레드에서 동시에 접근해도 안전하지만,  생성되는 시점에 DB 커넥션 풀을 생성하기에 생성 비용이 상당히 크다.
- 따라서 EntityManagerFactory에서 요청이 올 때마다 생성 비용이 거의 없는 EntityManager를 생성한다.

<img src = "https://user-images.githubusercontent.com/24876345/236078788-10f55fc1-a120-4d1a-9609-5f6221fb44a4.png" width="700px">

### 엔티티의 생명주기 ###

비영속(new/transient)|영속성 컨텍스트와 전혀 관계가 없는 상태
---|---
영속(managed)|영속성 컨텍스트에 저장된 상태
준영속(detached)|영속성 컨텍스트에 저장되었다가 분리된 상태
삭제(removed)|삭제된 상태


### @Entity ###
- javax.persistence.Entity
````
// spring boot
	- spring-boot-starter-parent [2.1.0.RELEASE] (higher)
		+ spring-boot-starter-data-jpa [2.1.0.RELEASE]
			- hibernate-core [5.3.7.Final]
			- javax.persistence-api [2.2]
````

- org.hibernate.annotations.Entity (deprecated되었으니 주의)


## SAVE 동작방식 ##

````java
SimpleJpaRepository.java

/*
 * (non-Javadoc)
 * @see org.springframework.data.repository.CrudRepository#save(java.lang.Object)
 */
@Transactional
@Override
public <S extends T> S save(S entity) {

	if (entityInformation.isNew(entity)) {
		em.persist(entity);
		return entity;
	} else {
		return em.merge(entity);
	}
}
````

### persist ###
- persist는 insert 문을 바로 호출

### merge ###
- select 후  식별자가 있으면 Update, 없으면 insert 문 호출

### isNew 함수 실행 ###
1. 식별자가 null 또는 0일 경우 new 상태로 인식
	- Long 타입과 같이 Wrapper 일 경우 null 을 newState 인식하고, int 와 같이 Primitive 일 경우 new 상태로 인식
2. Entity 필드에 @Version 필드가 null 이면 new 로 간주

3. Persistable interface 를 구현
````java
@Entity
public class Test implements Persistable<Long> {
    @Override
    public Long getId() {
        return null;
    }

    @Override
    public boolean isNew() {
        return false;
    }
}
````
출처: https://velog.io/@rainmaker007/spring-data-jpa-save-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC


## 0. bulk update ##
- https://brunch.co.kr/@purpledev/32

## 1. Bulk Insert ##
- RDBMS에서 bulk insert란 한번의 쿼리로 여러건의 데이터를 insert 할 수 있는 기능을 제공하는 것이다.
- 한번의 쿼리로 여러건의 데이터를 한번에 insert 할 수 있기 때문에 데이터베이스와 어플리케이션 사이의 통신에 들어가는 비용을 줄여주어 성능상 이득을 얻는다.
- 하지만 bulk insert를 원하는 테이블에서 auto_increment를 사용하고 있다면 bulk insert는 JPA를 통해서는 해결할 수 없다. 
- 프레임워크를 추가하거나 바꾸지 못 하고 JPA 해결해야되는 상황이라면 어쩔 수 없이 다수의 insert 쿼리를 통해 할 수 밖에 없다.
- 그래도 어떤 방법이 그나마 빠르게 이를 수행할 수 있을지 비교한다. 출처: https://sabarada.tistory.com/195

````java
// bulk insert example
insert into user (name, age)
values ('chd', 21),
       ('cha', 26),
       ('lolo', 15);
````

### 1-1. save 밖에서 for을 통해 insert ###

````java
@Test
void service() {
    long startTime = System.currentTimeMillis();
    
    for (int i = 0; i < 1000; i++) {
        bulkService.bulkService();
    }
    
    System.out.println("elapsed = " + (System.currentTimeMillis() - startTime) + "ms"); // 7531ms
}
````
````java
public void bulkService() {
    ...
    sampleRepository.save(review);
}
````

````java
// JPA save의 내부 코드에 @Transactional이 들어가 있다.
// save를 할 때 마다 트랜잭션을 잡는 행위를 하기 때문에 위 같은 경과시간을 보여주었다는 것을 확인할 수 있다.
@Transactional
@Override
public <S extends T> S save(S entity) {

    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
````

### 1-2. Transactional 을 통한 save ###
- for 문을 도는 부분을 @Tranasactional로 묶어서 트랜잭션 전파(propagation)을 통해 save 한다면 save시 마다 트랜잭션을 새로 열지 않을것이며 성능개선을 할 수 있을 것으로 판단
````java
@Test
void service() {
    long startTime = System.currentTimeMillis();
    
    bulkService.bulkService();    
    
    System.out.println("elapsed = " + (System.currentTimeMillis() - startTime) + "ms"); // 4255ms
}
````
````java
@Transactional
public void bulkService() {
    for (int i = 0; i < 1000; i++) {
        ...
        sampleRepository.save(review);
    }
}
````

### 1-3. @Transaction 안에서 saveAll ###
````java
@Test
void service() { 
    long startTime = System.currentTimeMillis();
    
    bulkService.bulkService();    
    
    System.out.println("elapsed = " + (System.currentTimeMillis() - startTime) + "ms"); // 2850ms
}
````
````java
public void bulkService() {
    List<SampleReq> lists = new ArrayList<>();
    for (int i = 0; i < 1000; i++) {
        ...
        lists.add(SampleReq);
    }

    sampleRepository.saveAll(
            sampleReq.stream()
                    .map(req -> sampleMapper.buildReview(userId, sampleId, req))
                    .collect(Collectors.toList())
    )
}
````

### 1-4. Spring Data JDBC의 batchUpdate()를 활용 ###
- Spring Data JDBC는 Spring Data JPA와 함께 혼용해서 사용
- @Transactional을 통해 트랜잭션이 관리될 수 있으므로, 현실적으로 가장 나은 방법

````java
public interface SampleRepository extends JpaRepository<Sample, String>, SampleRepositoryJdbc {
}
````

````java
public interface SampleRepositoryJdbc {
	void batchInsert(List<User> users);

}
````

````java
@Repository
public class SampleRepositoryJdbcImpl implements SampleRepositoryJdbc {
	private JdbcTemplate jdbcTemplate;

	public SampleRepositoryJdbcImpl(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}
	
	public void batchInsert(List<User> users) {
		jdbcTemplate.batchUpdate(
				"insert into SP_SAMPLE(ID, PW) "
				+ "values(?, ?)",
				 new BatchPreparedStatementSetter() {
					@Override
					public void setValues(PreparedStatement ps, int i) throws SQLException {
						User user = users.get(i);
						ps.setString(1, user.getId());
						ps.setString(2, user.getPw());
					}

					@Override
					public int getBatchSize() {
						return users.size();
					}
				});
	}

}
````

## DTO 클래스를 이용한 Request, Response 를 사용해야 한다. ##
- Request 경우 Entity를 사용하게된다면 원치 않은 데이터를 컨트롤러를 통해 넘겨받을 수 있게되고, 그로인한 변경이 발생할 수 있다.
- Response 경우, 비밀번호 같은 민감한정보를 포함해 모든 정보가 노출 된다.

#### DTO Sample ####
````java
public class AccountDto {
    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class SignUpReq {
        private String email;
        private String address2;
        private String zip;

        @Builder
        public SignUpReq(String email, String fistName, String lastName, String password, String address1, String address2, String zip) {
            this.email = email;
            this.address2 = address2;
            this.zip = zip;
        }

        public Account toEntity() {
            return Account.builder()
                    .email(this.email)
                    .address2(this.address2)
                    .zip(this.zip)
                    .build();
        }

    }

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class MyAccountReq {
        private String address1;
        private String address2;
        private String zip;

        @Builder
        public MyAccountReq(String address1, String address2, String zip) {
            this.address1 = address1;
            this.address2 = address2;
            this.zip = zip;
        }
    }

    @Getter
    public static class Res {
        private String email;
        private String address2;
        private String zip;

        public Res(Account account) {
            this.email = account.getEmail();
            this.address2 = account.getAddress2();
            this.zip = account.getZip();
        }
    }
}
````
