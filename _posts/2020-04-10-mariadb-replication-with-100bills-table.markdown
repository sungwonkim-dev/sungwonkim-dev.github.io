---
layout: post
title:  100억개의 데이터와 함께하는 MariaDB Replication & 데이터 병합 (1)  
date: 2020-04-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [mysql, mariadb, index, big table] # add tag
---

# 100억개의 데이터와 함께하는 MariaDB Replication & 데이터 병합
## 이슈 파악
### 현재 상황
1. 2개의 서버, 3개의 데이터베이스에 특정 성격의 데이터를 저장하고 있다.
2. 데이터는 테이블 단위로 관리되며 테이블은 도메인을 기준으로 생성했다.  
3. 데이터는 각 테이블에서는 고유한 값(UNIQUE)이지만 테이블 간, DB간 비교를 진행할 땐 고유한 값이 아니다.
4. 결과적으로 중복된 데이터를 저장했을 가능성이 있다
5. 테이블의 ROW가 늘어남에 따라 DML의 수행 시간 또한 늘어나고 있다. (약 100억 ROW) 
6. 데이터 백업이 이루어지고 있지 않다.
 
### 해결 방향  
1. 2개의 서버에 각각 Maria DB 설치 후 Replication 환경 구성 
2. 저장되는 데이터에 대한 중복 검사를 할 수 있도록 시스템 구성


## 각 해결 방향에 대한 주의 사항
1. 2개의 서버에 각각 Maria DB 설치 후 Replication 환경 구성  
1.1. 데이터를 활용하는 어플리케이션 Read/Write 비율을 조사하여 DB 글로벌 옵션 튜닝 필요    
1.2. 테이블 성격에 맞는 스토리지 엔진 적용 필요
2. 저장되는 데이터에 대한 중복 검사를 할 수 있도록 시스템 구성
2.1. 데이터 병합 필요  
2.2. 테이블 파티셔닝 필요  
2.3. 데이터 성격에 맞는 스토리지 엔진 적용 필요  
2.4. 데이터 성격과 스토리지 엔진에 맞는 인덱스 설정 필요  

**중복 검사용 테이블에 적용될 스토리지 엔진은 TokuDB가 될 가능성이 크다. 이유는 차근차근...** 
 
## 다음 포스팅 : 데이터 중복 검사 방안 찾기