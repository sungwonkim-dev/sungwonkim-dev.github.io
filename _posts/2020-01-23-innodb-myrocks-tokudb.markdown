---
layout: post
title: InnoDB vs MyRocks vs TokuDB
date: 2020-01-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [database, maria db, MySQL] # add tag
---

#InnoDB vs MyRocks vs TokuDB
##목표 : InnoDB, MyRocks, TokuDB의 특징을 이해, 이슈 개선안 도출 
###최근 발생한 이슈    
1. 로그성 데이터를 저장하는 테이블이 커지기 시작하면서 저장소의 용량이 부족해짐
    - 거의 full 수준  
2. 1번으로 인해서 로그성 데이터가 저장되지 못 하는 장애 상황 발생
<br>
<br> 
###목표
**데이터의 압축률이 높은 Storage Engine 또는 기능을 찾고자 함**  
   - 로그성 데이터에 적합한 Storage Engine 또는 기능
   - NoSQL은 최후의 수단으로..
      - 시스템  구조상..
<br>
<br>  

###DB와 Table 정보
1. DB 정보 
    - mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1  
    
2. table 정보
    - `log_msg` mediumtext // 주된 데이터를 저장하는 컬럼
    -  ENGINE=InnoDB, ROW_FORMAT=COMPRESSED, KEY_BLOCK_SIZE=8
    -  약 640,000,000개의 로우가 존재
        - 16개 테이블
    - 각 테이블당 최대 199GB, 최소 58GB의 용량을 차지  
        - 전체 1,726GB
    - 주로 insert와 select 쿼리만 사용한다.
<br>
<br>
###Storage Engine 후보
####InnoDB와 후보들의 공통 특징
   - Transaction 지원
   - ACID
   - Row Level Locking
   - MVCC 
      - DBMS에서 Lock을 사용하지 않고 데이터의 읽기 일관성을 보장해주는 기법
      - 읽기, 쓰기 작업이 많은 DB에는 매우 유용
      - [[MVCC]](https://mysqldba.tistory.com/335)
   -  XA 
      - 2 Phase Commit을 통해 분산 트랜잭션 처리가 가능
      - 하나 이상의 DB간 2PC가 보장돼야 하는 경우에 유용
      - [[XA 개념 정리]](https://heni.tistory.com/10)
   - automatic crash recovery
   
**아래부턴 각 엔진별 비교이기 때문에 사용되는 강조 표현에는 `각 엔진에 비해`가 있다고 생각해주세요.**  
####**InnoDB**  
   - 만능이다
      - 어느 상황에나 성능 이슈에 크게 신경쓰지 않아도 된다.
   - MySQL 5.5.5 버젼부턴 기본 엔진으로 사용된다.
      - 만능이라는 얘기  
<br>
<br>
####**TokuDB**
   - 쓰기 작업에 최적화된 엔진  
   - 꽤 괜찮은 성능의 압축 기능  
   - 빅데이터를 다루는 상황에서는 가장 좋은 선택  
<br>     
#####**조금 더 자세하게 알아보자**   
   - 읽기 최적화
      - 보조 인덱스 사용
      - Read Free Replication
        - [[TokuDB Read Free Replication : Details and Use Cases]](https://www.percona.com/blog/2014/09/25/tokudb-read-free-replication-details-and-use-cases/)
      - NO 인덱스 단편화
        - [[How to Identify and Fix SQL Server Index Fragmentation]](https://logicalread.com/2015/10/30/fix-sql-server-index-fragmentation-mc11/#.XikYQsj7SUl)
   - 쓰기 최적화
      - 빠른 INSERT
      - Bulk loader
      - 데이터 압축
   - **외래키가 지원되지 않음**
   - InnoDB에 비해 Crash Recovery기능이 부족함
<br>
<br>     
####**MyRocks**
   - 쓰기와 저장 공간 구성에 최적화된 엔진
   - 최고 성능의 압축 기능
   - SSD를 사용하는 상황에서 가장 좋은 선택
   - [[MyRocks: A space- and write-optimized MySQL database]](https://engineering.fb.com/core-data/myrocks-a-space-and-write-optimized-mysql-database/)  
<br>
#####**조금 더 자세하게 알아보자**
   - LSM Tree 사용
       - 쓰기와 저장 공간 구성에 최적화 된 방법
       - 모든 쓰기 동작은 메모리 테이블과 WAL을 먼저 수행
       - 작은 트랜잭션을 위해 디자인된 방법
   - **외래키가 지원되지 않음**
   - 일부 타입에만 인덱스를 적용할 수 있다.
<br>
<br>
<br> 
###**결론**
1. **이론상 로그성 데이터를 저장하기엔 InnoDB보단 TokuDB나 MyRocks를 사용하는 것이 이점이 더 많아보인다.**  
2. **여러가지 운영 환경과 시스템 구조에 대한 것을 종합해서 적용 여부를 판단하는 것이 좋겠다.**  
   - **MySQL -> MariaDB, Batch 어플리케이션 이슈 등**  
3. **이론도 중요하지만 실제로 벤치마킹 해봐야 할 거 같다.**  
4. **다음 글은 실제 벤치마킹에 대한 글이 될 것이다.**  
<br>
#
#  
참고 :   
[[choosing-the-right-storage-engine]](https://mariadb.com/kb/en/choosing-the-right-storage-engine/)  
[[MySQL Storage Engines - which do you use? TokuDB? MyRocks? InnoDB?]](https://www.slideshare.net/SvetaSmirnova/mysql-storage-engines-which-do-you-use-tokudb-myrocks-innodb)    
[[InnoDB, MyRocks and TokuDB on the insert benchmark]](http://smalldatum.blogspot.com/2017/05/innodb-myrocks-and-tokudb-on-insert.html)  
[[InnoDB vs TokuDB in LinkBench benchmark]](https://www.percona.com/blog/2015/07/24/innodb-vs-tokudb-in-linkbench-benchmark/)  