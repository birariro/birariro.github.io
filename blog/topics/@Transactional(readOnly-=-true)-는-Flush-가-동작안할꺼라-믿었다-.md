# @Transactional(readOnly = true) 는 Flush 가 동작안할꺼라 믿었다 


![](beedf595-d4c9-47db-a772-a66ee4094757.png)

    public class MarineService {
        WeaponService weaponService;
    
        @Transactional
        public Marine getMarine(){
            Factory factory = factoryRepository.findById(1L).get();
            Marine marine = factory.createMarine("marine");
    
            Weapon weapon = weaponService.getWeapon();
    
            marine.setWeaponName(weapon.getName());
            return marine
        }
    }
    
    public class WeaponService {
        @Transactional(readOnly = true)
        public Weapon getWeapon(){
             return weaponRepository.findById(1L).get();
        }
    }

위의 코드는 단순한 조회 로직을 가진 코드이며

getMarine() 메소드를 호출 후 로그를 보면 다음과 같은 로그를 볼 수 있다.

    //트랜젝션 생성
    Creating new transaction with name [com.MarineService.getMarine]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
    Opened new EntityManager for JPA transaction
    
    // getMarine() 기존 트랜젝션 참여
    Getting transaction for [com.MarineService.getMarine]
    
    // findById() 기존 트랜젝션 참여
    Getting transaction for [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findById]
    
    select f1_0.id from tb_factory f1_0 where f1_0.id=1
    
    // getWeapon() 기존 트랜젝션 참여
    Getting transaction for [com.WeaponService.getWeapon]
    
    // findById() 기존 트랜젝션 참여
    Getting transaction for [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findById]
    Skipping reading Query result cache data: cache-enabled = false, cache-mode = IGNORE
    
    select w1_0.id, w1_0.name from tb_weapon w1_0 where w1_0.id=1 
    
    // 트랜젝션 종료 과정 진행
    Completing transaction for [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findById]
    Completing transaction for [com.WeaponService.getWeapon]
    Completing transaction for [com.MarineService.getMarine]
    Initiating transaction commit
    Committing JPA transaction on EntityManager 
    committing
    Processing flush-time cascades
    Executing identity-insert immediately
    
    Closing JPA EntityManager after transaction
    

호출되는 메소드의 순서대로 트랜젝션에 참여하고 로직을 수행하고 종료되는 과정을 볼 수 있다.

    public class WeaponService {
        @Transactional(readOnly = true)
        public Weapon getWeapon(){
            //return weaponRepository.findById(1L).get();
             return weaponRepository.findByName("gun").get();
        }
    }

위의 과정에서 Weapon 을 가져오는 방식을 IfindById()를 findByName()으로 변경하게 되면 다른 동작을 볼 수 있다.

    //트랜젝션 생성
    Creating new transaction with name [com.MarineService.getMarine]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
    Opened new EntityManager for JPA transaction
    
    // getMarine() 기존 트랜젝션 참여
    Getting transaction for [com.MarineService.getMarine]
    
    // findById() 기존 트랜젝션 참여
    Getting transaction for [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findById]
    
    select f1_0.id from tb_factory f1_0 where f1_0.id=1
    
    // getWeapon() 기존 트랜젝션 참여
    Getting transaction for [com.WeaponService.getWeapon]
    
    // findByName() This method is not transactional.
    No need to create transaction for [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findByName]: This method is not transactional.
    
    //flush
    Processing flush-time cascades
    Executing identity-insert immediately
    
    //Dirty checking
    // ...
    

findByName()을 처리하는 과정에서 갑자기 Flush 가 동작하게 된다.

Flush는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 과정으로 다음과 같은 기준으로 동작하게 된다.

*   트랜젝션 커밋 전
*   JPQL 실행 전
*   직접 Flush를 실행 시

위의 코드에서는 FindByName() 호출 시 JPA에서는 메소드 이름을 분석해서 EntityManger의 CreateQuery를 사용하여 JPQL을 생성한다.

그렇기에 Flush 가 동작.

**org.springframework.data.jpa.repository.query.PartTreeJpaQuery.class**

    private TypedQuery<?> createQuery(CriteriaQuery<?> criteriaQuery) {
    
        if (this.cachedCriteriaQuery != null) {
            synchronized (this.cachedCriteriaQuery) {
                return getEntityManager().createQuery(criteriaQuery);
            }
        }
    
        return getEntityManager().createQuery(criteriaQuery);
    }

비슷한 과정으로 JPARepository의 findAll()의 경우도 CreateQuery를 사용하여 JPQL를 생성하게 된다.

**org.springframework.data.jpa.repository.support.SimpleJpaRepository.class**

    protected <S extends T> TypedQuery<S> getQuery(@Nullable Specification<S> spec, Class<S> domainClass, Sort sort) {
    
        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<S> query = builder.createQuery(domainClass);
    
        Root<S> root = applySpecificationToCriteria(spec, domainClass, query);
        query.select(root);
    
        if (sort.isSorted()) {
            query.orderBy(toOrders(sort, aroot, builder));
        }
    
        return applyRepositoryMethodMetadata(entityManager.createQuery(query));
    }

처음 findById()에서 Flush 가 동작하지 않았던 이유는 JPQL이 아닌 Entity Manager의 find라는 내장 기능을 활용하기에 Flush 가 동작하지 않았다.

**org.springframework.data.jpa.repository.support.SimpleJpaRepository.class**

    @Override
    public Optional<T> findById(ID id) {
    
        Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    
        Class<T> domainType = getDomainClass();
    
        if (metadata == null) {
            return Optional.ofNullable(entityManager.find(domainType, id));
        }
    
        LockModeType type = metadata.getLockModeType();
        Map<String, Object> hints = getHints();
    
        return Optional.ofNullable(type == null ? entityManager.find(domainType, id, hints) : entityManager.find(domainType, id, type, hints));
    }

읽기 전용 트랜젝션인 @Transactional(readOnly = true)를 사용하면 Flush Mode 가 MANUAL로 변경되어 Flush를 수동으로 실행하지 않는 한 실행되지 않게 된다.

또한 @Transactional(readOnly = true)는 Select 시 스냅샷을 저장하지 않게 되는데

Flush Mode 가 MANUAL로 전환되기에 Dirty checking 도 하지 않는다.

그렇다면 왜 getWeapon() 위에 @Transactional(readOnly = true) 이 붙어있는데 Flush 가 동작하는 걸까

**org.springframework.orm.jpa.vendor.HibernateJpaDialect.class**

    @Nullable
    protected FlushMode prepareFlushMode(Session session, boolean readOnly) throws PersistenceException {
    	FlushMode flushMode = session.getHibernateFlushMode();
    	if (readOnly) {
    		// We should suppress flushing for a read-only transaction.
    		if (!flushMode.equals(FlushMode.MANUAL)) {
    			session.setHibernateFlushMode(FlushMode.MANUAL);
    			return flushMode;
    		}
    	}
    	else {
    		// We need AUTO or COMMIT for a non-read-only transaction.
    		if (flushMode.lessThan(FlushMode.COMMIT)) {
    			session.setHibernateFlushMode(FlushMode.AUTO);
    			return flushMode;
    		}
    	}
    	// No FlushMode change needed...
    	return null;
    }

readOnly가 true 면 Flush Mode를 MANUAL 로 false 면 Flush Mode 를 AUTO로 설정하는 모습을 볼 수 있다.

문제는 트랜젝션을 시작하는 과정에 있다.

**org.springframework.transaction.interceptor.TransactionAspectSupport.class**

    	protected TransactionInfo createTransactionIfNecessary(
        		@Nullable PlatformTransactionManager tm,
    			@Nullable TransactionAttribute txAttr, 
                final String joinpointIdentification) {
    
            // If no name specified, apply method identification as transaction name.
            if (txAttr != null && txAttr.getName() == null) {
                txAttr = new DelegatingTransactionAttribute(txAttr) {
                    @Override
                    public String getName() {
    					return joinpointIdentification;
                    }
                };
            }
    
            TransactionStatus status = null;
            if (txAttr != null) {
                if (tm != null) {
                    status = tm.getTransaction(txAttr);
                }
                else {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
                                "] because no transaction manager has been configured");
                    }
                }
            }
            return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
        }

트랜젝션을 시작할 때 보면 기존 트랜젝션이 있다면 합류하고 그렇지 않다면 생성하게 된다.

현재 트랜젝션에는 ReadOnly를 활성해도 상위 트랜젝션 메소드가 ReadOnly = faluse 이기에 적용되지 않는다.

    @Transactional(readOnly = true ,propagation = Propagation.REQUIRES_NEW)
    public Weapon getWeapon(){
       return weaponRepository.findByName("gun").get();
    }

위처럼 트랜젝션을 합류하지 않고 신규 생성하게 한다면 Flush Mode 가 MANUAL로 동작하게 되고

findByName() 호출 시에 Flush 가 동작하지 않는다.