# 트랜잭션 isolation 레벨 정리

**READ UNCOMMITTED(Level 0)**
 - 어떤 트랜잭션의 변경내용이 COMMIT이나 ROLLBACK과 상관없이 다른 트랜잭션에서 보여진다.
 - Dirty Read, Non-Repeatable Read, Phantom Read 현상이 발생한다.

**READ COMMITTED(Level 1)**
 - 다른 트랜잭션에서 커밋된 내용만 참조할 수 있다.
 - Dirty Read 가 발생하지 않지만 Non-Repeatable Read, Phantom Read 현상은 여전히 발생한다.

**REPEATABLE READ(Level 2)**
 - 트랜잭션에 진입하기 이전에 커밋된 내용만 참조할 수 있다.
 - Dirty Read와 같은 현상은 발생하지 않지만 Phantom Read 현상은 여전히 발생한다.

**SERIALIZABLE(Level 3)**
 - 트랜잭션에 진입하면 락을 걸어 다른 트랜잭션이 접근하지 못하게 한다
 - 가장 엄격한 격리 수준으로 완벽한 읽기 일관성 모드를 제공함

레벨이 높아질 수록 트랜잭션간 고립 정도가 높아지며, 성능이 떨어지는 것이 일반적이며, 일반적인 온라인 서비스에서는 READ COMMITTED나 REPEATABLE READ 중 하나를 사용한다. 

## Dirty Read
변경 후 아직 Commit 되지 않은 값 읽고, Rollback 후의 값을 다시 읽어 최종 결과 값이 상이한 현상이다
|순서|트랜잭션 1|트랜잭션 2|
|--|--|--|
|1|select * from t <br> result <br> empty||
|2||insert into t values (1)|
|3|select * from t <br> result <br> a : 1 ||
|4||rollback|
|5|select * from t <br> result <br> empty||

## Non-Repeatable Read
한 트랜잭션 내에서 같은 쿼리를 두번 수행할 때, 그 사이에 다른 트랜잭션이 값을 수정 또는 삭제함으로써 두 쿼리가 상이하게 나타나는 비 일관성이 발생하는 것을 말한다.

|순서|트랜잭션 1|트랜잭션 2|
|--|--|--|
|1|select * from t <br> result <br> a : 1||
|2|update t set a = a + 1||
|3||delete from t where a = 1 <br> 2번 작업이 commit될 때까지 대기|
|4|commit||
|5||delete complete|
|5||select * from t <br> result <br> a : 2|

## Phantom Read
한 트랜잭션 안에서 일정범위의 레코드를 두번 이상 읽을 때, 첫 번재 쿼리에서 없던 유령 레코드가 두번째 쿼리에서 나타나는 현상을 말한다.
|순서|트랜잭션 1|트랜잭션 2|
|--|--|--|
|1|insert into t values (4)||
|2||select * from t <br> result <br> a : 2|
|3||update t set a = a + 1 <br> 1번이 커밋될 때까지 대기|
|4|commit||
|5||update complete|
|5||select * from t <br> result <br> a : 3 <br> a : 4|