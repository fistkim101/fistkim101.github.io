---
layout: article
title: mysql isolation level 알아보기
tags: 동시성 lock isolation
---

<style>
red { color: Red }
orange { color: Orange }
green { color: Green }
blue { color: Blue }
</style>

<br>
<br>

**<red>mysql isolation level 은 트랜젝션간의 순차성을 보장하기 위한 lock의 적용 여부를 선택적으로 조정하여 데이터 동시성을 제어하는 개념이다.</red>**
여기서 isolation은 흔히 트랜젝션의 속성으로 언급되는 ACID 를 이루는 I 로 트랜젝션 수행시 다른 트랜젝션이 끼어들어서 연산에 영향을 주지 못하게 하는(=고립을 시켜서)
것을 의미한다.

그래서 mysql isolation level를 이해하기 위해서는 먼저 isolation 상황을 조성해주는 lock에 대해서 학습을 해야한다.

<br>

### [lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_lock) 알아보기

동시성 이슈가 발생할 수 있는 원인은 애초에 특정한 트랜젝션이 어떤 데이터에 대해 특정한 처리를 한다고 할 때,
그 처리가 찰나로 이뤄지지 않고 아무리 짧더라도 결국 시각이 아닌 시간의 개념으로 일정 duration 이 소요되기 때문이다.

이 duration 동안 다른 트랜젝션이 끼어들어서 동일한 데이터에 다른 처리를 하게 되면 먼저 수행중인 트랜젝션이 의도와 다르게
처리할 가능성이 커진다. 이게 근본적으로 동시성이 발생하는 이유다.

이런 문제점을 방지하기 위해서는 직관적으로 보아도 첫번째 트랜젝션이 처리를 할 때 '다른 트랜젝션은 이거 보지도 말고 건들지도마' 라고 하면 쉽게
해결 될 것으로 보인다. 혹은 '건들지마'까지만하고 보는 것은 허용해줘도 첫번째 트랜젝션의 처리에 영향을 주진 않을 것이다.
이런 제약의 개념이 바로 lock이며 mysql에서는 공유락(shared lock)과 베타락(exclusive lock) 이렇게 두 가지 종류의 lock이 존재한다.

개별 lock 에 대해서 알아보기 전에 mysql 공식문서에 나와있는 lock 의 정의를 보자.

> **lock** <br>
> The high-level notion of an object that controls access to a resource, such as a table, row, or internal data structure, as part of a locking strategy. For intensive performance tuning, you might delve into the actual structures that implement locks, such as mutexes and latches.

<br>

#### [shared lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_shared_lock)
> **shared lock** <br>
> A kind of lock that allows other transactions to read the locked object,
> and to also acquire other shared locks on it, but not to write to it. The opposite of exclusive lock.

요약하자면 아래와 같다.<br>
1) shared lock이 걸려있는 object 는 다른 트랜젝션이 read 는 가능하다.<br>
2) shared lock이 걸려있는 object 는 다른 트랜젝션이 write 할 수 없다.<br>
3) 이미 shared lock이 걸려있는 object 에 따른 트랜젝션이 중첩해서 shared lock 을 걸 수 있다.<br>

헷갈릴 수 있는 부분이 있는데(나는 헷갈렸다) 실습을 하다보면 shared lock이 걸려 있는데 다른 트랜젝션이 데이터를 바꿀 수 있었다.
문서에는 분명 shared lock이 걸려있으면 다른 트렌젝션의 write를 금지한다고 했는데? 왜 바뀌는걸까.

여기서 잘 알아둬야할 것이 write를 금지하는 대상은 shared lock 을 설정한 The object 인 것이다.
즉, shared lock을 설정할 때 select로 읽어들인 그 데이터에 대해서 불변을 보장(다른 트랜젝션의 write를 불허)한다는 것이다.

**<blue>그래서 shared lock이 걸린 데이터라 해도 실제로 다른 트랜젝션이 update를 해서 디스크에 쓰는것 까지 모두 가능하다.
하지만 shared lock이 걸려있는 데이터는 디스크에서 바뀌었다고 해도 shared lock이 풀리기 전이라면 동일 트랜젝션 내에서 다시 select를 해도
여전히 변경 전의 데이터를 보게 된다.</blue>**

나의 경우 이 차이에 대한 이해는 나중에 나올 mysql isolation level에서 READ COMMITED 와 REPEATABLE READ 의 차이를 이해할 때
중요하게 작용했다.

<br>

#### [exclusive lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_exclusive_lock)
> **exclusive lock** <br>
> A kind of lock that prevents any other transaction from locking the same row.
> Depending on the transaction isolation level, this kind of lock might block other transactions from writing to the same row,
> or might also block other transactions from reading the same row.
> The default InnoDB isolation level, REPEATABLE READ, enables higher concurrency by allowing transactions to read rows that have exclusive locks, a technique known as consistent read.

요약하자면 아래와 같다.<br>
1) exclusive lock 이 걸려있는 경우 같은 row에 대해서 다른 트랜젝션에 의한 중첩 lock은 금지시킨다.<br>
2) exclusive lock 이 걸려있는 경우 같은 row에 대해서 다른 트랜젝션의 read, write 모두 금지 시킨다.<br>

실제로 실험을 해보면 exclusive lock 을 걸어두고 다른 트랜젝션으로 같은 row에 대해서 update를 시도하면
하염없이 exclusive lock이 풀리기만을 기다리면서 시간이 흐른다.

<br>
<br>

### 동시성 제어가 되지 않으면 무슨 일이 발생할까? (신뢰할 수 없는 read에 대해 알아보기)
mysql isolation level를 이해하기 위해서 선행 학습해야할 개념이 lock 말고 하나가 더 있다.
동시성 제어가 되지 않았을 때 발생하는 '신뢰할 수 없는 read'를 유형별로 아는 것이다.

#### [dirty read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dirty_read)
> An operation that retrieves unreliable data, data that was updated by another transaction but not yet committed.
> It is only possible with the isolation level known as read uncommitted.<br>
> This kind of operation does not adhere to the ACID principle of database design.
> It is considered very risky, because the data could be rolled back, or updated further before being committed;
> then, the transaction doing the dirty read would be using data that was never confirmed as accurate.<br>
> Its opposite is consistent read, where InnoDB ensures that a transaction does not read information updated by another transaction,
> even if the other transaction commits in the meantime.

**<red>다른 트랜젝션에 의해서 아직 commit 되지 않은 데이터를 read 하는 경우이다.</red>**
다른 트랜젝션이 변경중이거나 추가한 데이터를 아직 커밋을 하지도 않았는데도 이를 읽어들이는 경우를 뜻한다.
이걸 롤백을 해버리면 무용지물이기 때문에 '정해지지 않은, 확정되지 않은' 데이터라는 의미로 dirty가 붙은 듯 하다.
단어 선택이 좀 확 와닿지 않아서 네이밍이 아쉽다는 생각이 들었다. 뭔가 '불완전의, 확정되지 않은' 뭐 이런 느낌의 네이밍이 더 좋지 않았을까.
**<red>오로지 read uncommitted 레벨에서만 발생하는 오류 유형이다.</red>**

<br>

#### [non-repeatable read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_non_repeatable_read)
> The situation when a query retrieves data, and a later query within the same transaction retrieves what should be the same data, but the queries return different results (changed by another transaction committing in the meantime).<br>
> This kind of operation goes against the ACID principle of database design. Within a transaction, data should be consistent, with predictable and stable relationships.<br>
> Among different isolation levels, non-repeatable reads are prevented by the serializable read and repeatable read levels, and allowed by the consistent read, and read uncommitted levels.

**<red>하나의 트랜젝션 내에서 동일한 쿼리를 두 번 실행했을 때, 나중에 실행한 쿼리가 첫번째 실행한 결과와 다른 결과를 가져오는 경우를 두고 non-reapeatable read 라고 한다.</red>**
이런 현상은 ACID 중 C에 매우 명백하게 위배된다고 할 수 있다. mysql isolation level 중 repeatable read 와 serializable read 는 동일 트랜젝션 내에서 shared lock 을 계속 걸어준 상태로 유지하기 때문에
non-repeatable read 현상이 나타나지 않는다. read commited 레벨에서는 나타날 수 있는데, 이유는 shared lock을 최초에 걸고 이를 트랜젝션이 끝날 때까지 유지하지 않기 때문이다.

<br>

#### [phantom read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_phantom)
> A row that appears in the result set of a query, but not in the result set of an earlier query. For example, if a query is run twice within a transaction, and in the meantime, another transaction commits after inserting a new row or updating a row so that it matches the WHERE clause of the query.<br>
> This occurrence is known as a phantom read. It is harder to guard against than a non-repeatable read, because locking all the rows from the first query result set does not prevent the changes that cause the phantom to appear.<br>
> Among different isolation levels, phantom reads are prevented by the serializable read level, and allowed by the repeatable read, consistent read, and read uncommitted levels.

**<red>하나의 트랜젝션 내에서 동일한 쿼리를 두 번 실행했을 때, 나중에 실행한 쿼리에서 처음 실행한 쿼리에 없었던 row들이 생긴 경우이다.</red>**
트랜젝션 A의 첫번째 select 가 실행되고서 트랜젝션이 완료되기 전에 트랜젝션 B가 같은 테이블에 데이터를 insert-commit 한 경우,
트랜젝션 A가 첫번째 select 문을 다시 실행하면 아까 트랜젝션 B가 넣은 데이터가 추가되어 나오는 현상이다.
**<red>오직 serialize level 에서만 막을 수 있는 현상이다.</red>**


<br>

### [mysql isolation level](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html) 알아보기
이제야 mysql isolation level 을 알아보게 되었다. 위에서 정리한 lock 과 세 가지 read 오류들을 이해하지 않으면
mysql isolation level 을 학습한다는 것이 곧 각 level 에 따라 다르게 적용되는 lock 과 예방할 수 있는 read 오류의 종류를 학습하는 것들이기 때문에
저러한 것들이 먼저 정리가 되지 않으면 mysql isolation level 를 학습할 준비가 되어있지 않은 것으로 판단된다.

<br>

#### READ UNCOMMITTED
shared lock이 적용되지 않고, exclusive lock은 적용됨. 즉, [consistent read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read)를 보장하지 않는다.
그래서 모든 read 오류 유형이 나타날 수 있다.

<br>

#### READ COMMITTED(oracle 기본값)
shared lock,exclusive lock 모두 적용되지만 shared lock의 적용과 해제 시점이 좀 독특하다.

> Each consistent read, even within the same transaction, sets and reads its own fresh snapshot.

공식문서를 보면 위와 같이 나와있는데 동일한 트랜젝션 내에서 매번 새로 lock을 걸고 해제하면서 각각의 fresh한 snapshot을 보여준다는 것이다.
그래서 READ COMMITTED의 경우에는 dirty read는 막을 수 있지만 non-repeatable read, phantom read는 막지 못한다.

<br>

#### REPEATABLE READ(mysql 기본값)
> Consistent reads within the same transaction read the snapshot established by the first read.

동일한 트랜젝션 내에서 Consistent reads가 보장된다는 것이 핵심이다. 즉, 동일한 트랜젝션 내에서라면 처음 select 때 나온 스냅샷을
트랜젝션이 끝날때 까지는 반드시 보여준다는 의미이다.(두번째 select 시 나온 데이터가 첫번째 select와 다르면 undo log를 통해서 스냅샷을 유지한다)

그래서  dirty read, non-repeatable read 모두를 막을 수 있지만 새로운 insert 는 막을 수 없으므로 phantom read는 발생한다.

<br>

#### SERIALIZABLE
phantom read까지 막아주는 isolation 강도가 가장 높은 level이다. 테이블 자체에 RANGE locks 을 걸어서 insert를 막는다고 하는데
공식문서를 봐도 정확하게 파악이 안되서 더 찾아보고 정리해야겠다.

<br>
<br>
