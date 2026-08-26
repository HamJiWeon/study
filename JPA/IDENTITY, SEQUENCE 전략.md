## `@GeneratedValue(stratege = GenerationType.IDENTITY)`
`IDENTITY` 전략은 기본 키 생성을 데이터베이스에 위임하는 전략이다.

**사용 데이터베이스**
- MySQL
- PostgreSQL
- SQL Server
- DB2

```sql
CREATE TABLE BOARD(
    ID BIGINT NOT NULL PRIMARY KEY,
    DATA VARCAHR(255)
)

INSERT INTO BOARD(DATA) VALUES('A');
INTERT INTO BOARD(DATA) VALUES('B');
```

`IDENTITY`전략은 먼저 Entity에 ID값을 저장한 후, 식별자를 조회해서 엔티티의 식별자에 할당한다.

## `@GeneratedValue(strategy = GenerationType.SEQUENCE)`
> **Sequence란 무엇일까?**
> 순차적으로 Unique 한 숫자를 자동으로 생성해주는 데이터베이스 객체
> **특성**
> **start value**: 시퀀스의 시작값을 지정
> **increment by**: 시퀀스의 값이 증가하는 크기
> **max value, min value**: 시퀀스 값의 범위 제한
> **cycle**: 최대 값 또는 최소 값에 도달하면 시퀀스 값을 다시 시작

데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다.  
`SEQUENCE` 전략은 이 시퀀스를 사용해서 기본 키를 생성한다.

**사용 데이터베이스**
- Oracle
- PostgreSQL
- DB2
- H2

```sql
CREATE TABLE BOARD(
    ID BIGINT NOT NULL PRIMARY KEY,
    DATA VARCAHR2(255)
)

// 시퀀스 생성
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```

`SEQUENCE`전략은 `em.persist()`를 호출할 때 먼저 데이터베이스 시쿼스를 사용해서 식별자를 조회한다.  
그리고 조회한 식별자를 엔티티에 할당한 후에 엔티티 영속성 컨텍스트에 저장한다. 이후 트랜잭션에서 `commit` 후에 `flush`가 일어나면 엔티티를 데이터베이스에 저장한다.

`SEQUENCE`는 매핑을 따로 해줘야 하는데 예시 코드를 확인해보자.
```java
@Entity
@SequenceGenerator(
    name = "식별자 생성기 이름",
    sequenceName = "데이터베이스에 등록되어 있는 시퀀스 이름", // Default = hibernate_sequence
    initialValue = "DDL 생성시에만 사용됨. 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정", // Default = 1
    allocationSize = "시퀀스 한 번 호출에 등가하는 수(성능 최적화에 사용됨) // Default = 50
    catalog, schema = "데이터베이스 catalog, schema 이름"
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    
}
```
