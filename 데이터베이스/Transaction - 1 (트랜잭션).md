## 트랜잭션

트랜잭션은 **하나의 작업을 수행하기 위해 필요한 데이터베이스의 연산들을 모아놓은 것**으로, 데이터베이스에서 **논리적인 작업의 단위**이며 장애가 발생했을 때 **데이터를 복구하는 작업의 단위**이다. 

트랜잭션을 관리함으로써 데이터베이스의 회복과 병행 제어가 가능해져 데이터의 정합성을 보장해준다.

## 트랜잭션의 특징

트랜잭션이 성공적으로 처리되어 데이터베이스의 무결성과 일관성을 보장하려면 4가지 특성을 만족해야 한다.

1. 원자성(Atomicity)
    - 트랜잭션을 구성하는 연산들이 모두 정상적으로 실행되거나 하나도 실행되지 않아야 한다는 `all-or-nothing` 방식을 의미한다.
    - 이처럼 트랜잭션의 원자성을 보장하려면 장애가 발생했을 때 데이터베이스의 원래 상태로 복구하는 회복 기능이 필요하다.
2. 일관성(Consistency)
    - 트랜잭션이 성공적으로 완료되면 데이터베이스는 일관적인 상태를 유지해야 함을 의미한다.
    - 쉽게 말하자면, 트랜잭션이 제약조건, `cascade` , `triggers` 를 포함한 정의된 모든 조건에 맞게 데이터의 값이 변경되야 함을 뜻한다.
3. 격리성(Isolation)
    - 현재 수행 중인 트랜잭션이 완료될 때까지 트랜잭션이 생성한 중간 연산 결과에 다른 트랜잭션들이 접근할 수 없음을 의미한다.
4. 지속성(Durability)
    - 트랜잭션이 성공적으로 완료된 후 데이터베이스에 반영한 수행 결과는 어떠한 경우에도 손실되지 않고 영구적이여야 함을 의미한다.
    

## 트랜잭션의 연산

데이터베이스에 날린 쿼리가 반영되는 시점은 트랜잭션이 성공적으로 완료되는 시점에 반영이 되는데, 트랜잭션에는 두 가지 과정이 있다.

**commit 연산**

- 트랜잭션이 성공적으로 수행되었을 때 선언한다(작업 완료).
- `commit` 연산이 실행된 후에야 트랜잭션의 수행 결과가 데이터베이스에 반영된다.

**rollback 연산**

- 트랜잭션이 수행을 실패했을 때 선언한다(작업 취소).
- `rollback` 연산이 실행되면 트랜잭션이 지금까지 실행한 연산의 결과를 취소하고 수행 전의 상태로 돌아간다.

## 트랜잭션의 상태

![image](https://user-images.githubusercontent.com/89899249/185291908-29f17fd7-6c58-49d2-b817-e9462a3f669d.png)

- 활동(Active) : 트랜잭션이 실행중인 상태
- 부분 완료(Parially Committed) : 트랜잭션의 마지막 연산까지 실행했지만, Commit 연산이 실행되기 직전의 상태
- 완료(Commit) : 트랜잭션이 성공적으로 종료되어 `commit` 연산을 실행한 후의 상태
- 실패(Failed) : 트랜잭션 실행에 오류가 발생해 중단된 상태
- 철회(Aborted) : 트랜잭션이 비정상적으로 종료되어 `rollback` 연산을 수행한 상태

## MySQL에서의 트랜잭션

`MySQL` 서버에서는 `MyISAM` 이나 `Memory` 스토리지 엔진이 더 빠르다고 생각하고 `InnoDB` 스토리지 엔진은 사용하기 복잡하고 번거롭다고 생각한다. 하지만 사실은 `MyISAM` 이나 `Memory` 같이  트랜잭션을 지원하지 않는 스토리지 엔진의 테이블이 더 많은 고민거리를 만들어 낸다.

트랜잭션은 꼭 여러 개의 변경 작업을 수행하는 쿼리가 조합됐을 때만 의미 있는 개념은 아니다. 

트랜잭션(하나의 논리적인 작업 셋)에 하나의 쿼리가 있든 두 개 이상의 쿼리가 있든 관계없이 **트랜잭션 자체가 100% 적용되거나(`commit` 을 실행했을 때) 아무것도 적용되지 않도록 해서(`rollback` 됐을 때) 데이터의 정합성을 보장할 수 있다.**

간단한 예제로 트랜잭션 관점에서 `InnoDB` 와 `MyISAM` 테이블의 차이를 살펴보겠다.

```sql
create table tab_myisam ( fdpk int not null, primary key (fdpk)) engine=MyISAM;
insert into tab_myisam (fdpk) values (3);

create table tab_innodb ( fdpk int not null, primary key (fdpk)) engine=InnoDB;
insert into tab_innodb (fdpk) values (3);
```

테스트용 테이블에 각각 레코드를 1건씩 저장한 후 다음 쿼리를 `InnoDB` 테이블과 `MyISAM` 테이블에서 각각 실행해 보겠다.

```sql
insert into tab_myisam (fdpk) values (1), (2), (3);
insert into tab_innodb (fdpk) values (1), (2), (3);
```

**실행 결과**

![image](https://user-images.githubusercontent.com/89899249/185292003-a2118486-024a-4434-abba-654ef84b723e.png)

**select * from tab_myisam 결과**

![image](https://user-images.githubusercontent.com/89899249/185292085-e8637f2d-da6e-4e3d-aa49-b3320084cf46.png)

**select * from tab_innodb 결과**

![image](https://user-images.githubusercontent.com/89899249/185292204-053fd498-f9fe-452f-a89f-7efe50a6f981.png)

두 문장 모두 `primary` 키 중복 오류로 쿼리가 실패했다. 그런데 두 테이블의 레코드를 조회해보면 `MyISAM` 테이블에는 오류가 발생했음에도 1과 2는 `insert` 한 상태로 남아 있는 것을 볼 수 있다. `MEMORY` 스토리지 엔진을 사용해도 이와 동일한 결과가 나온다. 

이 현상을 **부분 업데이트(Partial Update)**라고 표현하며, 이러한 부분 업데이트 현상은 테이블 데이터의 정합성을 맞추는데 상당히 어려운 문제를 만들어 낸다.

만약 트랜잭션 없이 이 문제를 해결하려면 단순한 `insert` 쿼리 하나에도 수 많은 `if-elif-else` 문을 작성해야 한다.

### 주의사항

트랜잭션 또한 `DBMS` 의 커넥션과 동일하게 꼭 필요한 최소의 코드에만 적용(트랜잭션 범위를 최소화)하는 것이 좋다.

아래 예시는 사용자가 게시판에 게시물을 작성한 후 저장 버튼을 클릭했을 때 서버에서 처리하는 내용을 순서대로 정리한 것이다.

```
1) 처리 시작
	==> 데이터베이스 커넥션 생성
	==> 트랜잭션 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
9) 알림 메일 발송 이력을 DBMS에 저장
	<== 트랜잭션 종료(COMMIT)
	<== 데이터베이스 커넥션 반납
10) 처리 완료
```

 위 처리 절차 중에서 `DBMS` 의 트랜잭션 처리에 좋지 않은 영향을 미지는 부분은 다음과 같다.

- 데이터베이스의 커넥션을 생성하는 코드를 1번과 2번 사이에 구현하며 그와 동식에 `START TRANSACTION` 명령으로 트랜잭션을 시작한다. 그리고 9번과 10번 사이에서 트랜잭션을 `COMMIT` 하고 커넥션을 종료한다. 그러나 실제로 `DBMS` 에서 데이터를 저장하는 작업(트랜잭션)은 5번부터 시작된다는 것을 알 수 있다. 일반적으로 데이터베이스 커넥션은 개수가 제한적이기 때문에 2번과 3번, 4번의 절차가 아무리 빨라도 `DBMS` 의 트랜잭션에 포함시킬 필요가 없다.
- 더 큰 위험은 8번 작업이다. 메일 전송이나 `FTP` 파일 전송 작업 또는 네트워크를 통해 원격 서버와 통신하는 등과 같은 작업은 어떻게 해서든 `DBMS` 의 트랜잭션 내에서 제거하는 것이 좋다.
- 또한 이 처리 절차에는 `DBMS` 의 작업이 크게 4개가 있다.
    - 사용자가 입력한 정보를 저장하는 5번과 6번은 반드시 하나의 트랜잭션에 묶어야 한다.
    - 7번 작업은 저장된 데이터의 단순 확인 및 조회이므로 트랜잭션에 포함할 필요가 없다.
    - 9번 작업은 조금 성격이 다르기 때문에 이전 트랜잭션(5번과 6번 작업)에 함께 묶지 않아도 무방하다.

문제가 될 만한 부분 세 가지를 보완해서 위의 처리 절차를 다시 설계해보겠다.

```
1) 처리 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
	==> 데이터베이스 커넥션 생성
	==> 트랜잭션 시작
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
	<== 트랜잭션 종료(COMMIT)
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
	==> 트랜잭션 시작
9) 알림 메일 발송 이력을 DBMS에 저장
	<== 트랜잭션 종료(COMMIT)
	<== 데이터베이스 커넥션 반납
10) 처리 완료
```

보여준 예제가 최적의 트랜잭션 셜계는 아닐 수 있다. 여기서 설명하려는 바는 프로그램의 코드가 데이터베이스 **커넥션을 가지고 있는 범위와 트랜잭션이 활성화돼 있는 프로그램의 범위를 최소화**해야 한다는 것이다. 또한 **네트워크 작업이 있는 경우에는 반드시 트랜잭션에서 배제**해야 한다.

## 참고
- [https://jeong-pro.tistory.com/241](https://jeong-pro.tistory.com/241)
- [https://victorydntmd.tistory.com/129](https://victorydntmd.tistory.com/129)
- [https://do-my-best.tistory.com/50](https://do-my-best.tistory.com/50)
- [https://brunch.co.kr/@skeks463/27](https://brunch.co.kr/@skeks463/27)
- Real MySQL

## 예상 면접 질문 및 답변

### Q.  트랜잭션이란 무엇인가?

트랜잭션은 **하나의 작업을 수행하기 위해 필요한 데이터베이스의 연산들을 모아놓은 것**으로, 데이터베이스에서 **논리적인 작업의 단위**이며 장애가 발생했을 때 **데이터를 복구하는 작업의 단위**이다.

### Q. 트랜잭션의 특성에 대해 설명

1. 원자성(Atomicity)
    - 트랜잭션을 구성하는 연산들이 모두 정상적으로 실행되거나 하나도 실행되지 않아야 한다는 `all-or-nothing` 방식을 의미한다.
2. 일관성(Consistency)
    - 트랜잭션이 제약조건, `cascade` , `triggers` 를 포함한 정의된 모든 조건에 맞게 데이터의 값이 변경되야 함을 뜻한다.
3. 격리성(Isolation)
    - 현재 수행 중인 트랜잭션이 완료될 때까지 트랜잭션이 생성한 중간 연산 결과에 다른 트랜잭션들이 접근할 수 없음을 의미한다.
4. 지속성(Durability)
    - 트랜잭션이 성공적으로 완료된 후 데이터베이스에 반영한 수행 결과는 어떠한 경우에도 손실되지 않고 영구적이여야 함을 의미한다.

### Q. 트랜잭션의 연산에 대해 설명

1. **commit 연산**
- 트랜잭션이 성공적으로 수행되었을 때 선언하며, `commit` 연산이 실행된 후에야 트랜잭션의 수행 결과가 데이터베이스에 반영된다.
2. **rollback 연산**
- 트랜잭션이 수행을 실패했을 때 선언하며, `rollback` 연산이 실행되면 트랜잭션이 지금까지 실행한 연산의 결과를 취소하고 수행 전의 상태로 돌아간다.

### Q. 트랜잭션의 상태에 대해 설명

본문 참고

### Q.  `MyISAM` 스토리지 엔진에 대해 설명

`MyISAM` 스토리지 엔진은 트랜잭션에 안전하지 않은 테이블을 제공하는 스토리지 엔진이다. 장점으로는 항상 테이블에 `ROW COUNT`를 갖고 있기 때문에 `SELECT count(*)` 명령시 빠르고, `SELECT` 명령시에도 빠른 속도를 지원한다. 그러나  트랜잭션을 지원하지 않는다는 점과 레코드 락을 지원하지 않아 쿼리 실행 시 테이블 전체에 락이 걸린다는 단점이 있다.

### Q. `InnoDB` 스토리지 엔진에 대해 설명

`InnoDB` 스토리지 엔진은 트랜잭션에 안전한 테이블을 제공하는 스토리지 엔진이다. 트랜잭션과 레코드 락을 지원하기 때문에 대용량의 데이터를 처리하는데 있어 유리한 점이 있다. 따라서 `CRUD`가 많은 서비스에 유리하다.

### Q. 트랜잭션을 적용할 때 주의해야할 점

데이터베이스 커넥션을 가지고 있는 범위와 트랜잭션이 활성화돼 있는 프로그램의 범위를 최소화해야 하며, 네트워크 작업이 있는 경우에는 반드시 트랜잭션에서 배제해야 한다.
