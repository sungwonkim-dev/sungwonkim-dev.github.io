---
layout: post
title: RabbitMQ를 활용한 실시간 데이터 가공 시스템 개선 (1) - 이슈와 개선 방향
date: 2020-01-31 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, database, application-design] # add tag
---

# 카프카를 활용한 실시간 데이터 가공 시스템 개선
## 실시간 데이터 가공 시스템??
### [AS-IS]
![실시간 데이터 가공 시스템 요약도](../assets/img/post/live-data-processing-system-old.png)

**시스템 요약**
1. A 어플리케이션은 사용자의 실시간 요청에 따라 100 ~ 10,000개의 데이터를 생성하고 T1 테이블에 저장
2. B 어플리케이션은 생성된 데이터를 조회하여 데이터 가공 후 T2 테이블에 저장   
2.1. 약 100개의 B 어플리케이션 운용 중
3. B 어플리케이션은 진행 상황에 따라 T1 테이블을 갱신
4. A 어플리케이션은 진행 상황을 확인하기 위해 T1 테이블을 조회 
5. C 어플리케이션은 T2 테이블에 저장된 데이터를 재가공 후 T1 테이블을 갱신  
5.1. A, B, C 어플리케이션들은 각 역할이 다르기 때문에 통합할 수 없음  

## 이슈 
1. A 어플리케이션에서 진행 상황 조회시 응답 지연 이슈 발생  
1.1. **시스템 요약** 중 1, 3, 4, 5에 영향을 받음
2. B 어플리케이션에서 데이터 가공을 위해 T1 테이블 조회시 중복으로 가공하는 이슈 발생     
2.1. T1 테이블 조회시 진행 상황 플래그를 변경하지만 동시에 접근하는 경우가 많음  
2.2. TABLE LOCK으로 해결하고자 했으나 사용자에게 반환되는 시간이 지연돼 시스템 장애 유발 가능성이 생김  

## 필요한 기능
1. 로드 밸런싱  
1.1. B 어플리케이션에서 빠른 데이터 가공을 할 수 있도록 하기 위함  
2. 대기열 기능  
2.1. 현재 시스템에선 테이블을 대기열로 사용하고 있어 응답 지연 등 부하 이슈가 발생하고 있음    
2.2. 응답 지연 방지, 시스템 장애 방지  
2.3. 중복 데이터 처리 방지
3. 많은 데이터에 대한 실시간 처리 가능  
3.1. 데이터의 양이 정해진 것이 아니라 스케일러블하게 운용이 가능하면 좋겠음

## 개선 방안
### 요약 : 해당 요청들을 메시지로 처리하기 위해 RabbitMQ를 도입하자
![RabbitMQ](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)  
**[RabbitMQ 레퍼런스](https://www.rabbitmq.com/documentation.html)와 블로그글이 많으니 다음 포스팅에선 아주 자세한 설명은 생략하고 간단한 설명만 작성합니다.**

### [TO-BE]
![실시간 데이터 가공 시스템 요약도](../assets/img/post/live-data-processing-system-new.png)
1. 로드 밸런싱 - RabbitMQ에 내장된 Message에 대한 병렬 처리 기능 활용
2. 대기열 기능 - 메시지 큐  활용
3. 많은 데이터에 대한 실시간 처리 기능 - 대용량 요청을 파티션 기능으로 빠르고 스케일러블하게 처리 가능
4. DB CONNECTION 어플리케이션 추가로 DB 부하 및 응답 지연 개선

## 다음 포스팅  : Kafka vs RabbitMQ