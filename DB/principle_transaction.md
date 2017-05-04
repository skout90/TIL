## 트랜잭션의 원리

트랜잭션이란 DB의 논리적 기능을 수행하기 위한

작업의 논리적 단위이다.

이 트랜잭션은 어떻게 이루어 지는 것일까?

기본적인 개념을 이해해보자.



#### DB 시스템의 구조

비휘발성 디스크에 데이터 저장, 일부 데이터 메인 메모리 유지.

![dbms1](http://d2.naver.com/content/images/2015/06/helloworld-407507-1.png)



보통 위와 같이 질의처리기와 저장시스템으로 나눠진다.

MySQL의 경우 저장시스템으로 InnoDB와 MyISAM 과 같은 엔진이 있다.



DBMS는 데이터를 Page 단위로 입출력한다.

페이지 관리하는 모듈을 페이지 버퍼관리자라고 함.



> 버퍼 관리 정책에 따라 트랜잭션의 UNDO복구와 REDO 복구가 이루어짐



#### UNDO

트랜잭션이 정상 종료 되지 않을때.

원상복구 되어야함 = UNDO



메모리 버퍼 정책에 따라

- 수정된 페이지들을 최소한 트랜잭션 종료시점(EOT) 까지는 버퍼에 유지(STEAL)
- 수정된 페이지를 언제든지 디스크에 씀.(¬STEAL)

트랜잭션 종료전에, 디스크에 쓴다면. 버퍼 공간은 절약하지만

UNDO 작업이, 메모리 + 디스크 까지 수정해주야해서 복잡해지는 문제가 있음.



#### REDO

커밋한 트랜잭션의 수정을 재반영하는 복구 작업(REDO 복구)

- 수정했던 모든 페이지를 트랜잭션 커밋시점에 디스크에 반영(Force)
- 수정했던 페이지를 트랜잭션 커밋시점에 디스크에 반영하지 않음(¬FORCE)



#### UNDO와 REDO 복구를 위한 로그

- 물리적인 상태로깅(physical state logging)

갱신 이전 이미지와 이후 이미지를 모두 가지고 있음.

UNDO : 이전 이미지로 현재 이미지 대체

REDO : 이후 이미지로 대체

> UPDATE 문장에 대한 로깅은 수정 전 레코드와, 새로 갱신 레코드를 모두 기록하고 위와 같이 UNDO나 REDO 실행



- 논리적인 전이로깅(logical transition logging)

물리적인 로깅이 결과값을 기록하는 방식이라면

논리적인 로깅은 어떤 일을 했었는가를 기록하는 방식이다.



> a = a + 1와 같은 연산을 로깅할 때, 
>
> 이전 값 0, 이후 값 1을 기록하거나, 연산 자체를 기록하기도 한다.



이 논리적인 전이 로깅은, 로그 레코드의 크기를 줄여주며,

물리적으로 복구하기 힘든 자료구조(B+=tree)의 복구를 쉽게 해준다.



#### 로그가 쓰여지는 방식

규칙이 있다.

- WAL(Write Ahead Logging) 원칙

업데이트가 데이터베이스에 써지기 전에 UNDO 정보가 로그에 써져야함.

- 트랜잭션이  정상적으로 종료 처리 되기 위해서는 REDO 정보가 로그에 써져야함.



## Reference

[Naver D2 : DBMS는 어떻게 트랜잭션을 관리할까?](http://d2.naver.com/helloworld/407507)