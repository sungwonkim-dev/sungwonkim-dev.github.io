---
layout: post
title: 카프카를 활용한 실시간 데이터 가공 시스템 개선 (2) - RabbitMQ Tutorial
date: 2020-02-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, rabbitmq, database, application-design] # add tag
---

# 카프카를 활용한 실시간 데이터 가공 시스템 개선
## 갑자기 RabbitMQ??
**포스팅에선 다루지 않았지만 Apache Kafka와 RabbitMQ에 대한 리서치를 진행했었다.**  
**요약과 함께 특징을 나열해보겠다**
### 공통점  
1. RabbitMQ, Apache Kafka 모두 상업적으로 사용 가능한 오픈 소스이다.  
2. 모두 pub - sub 시스템으로 설계되어있다.

### RabbitMQ
1. 다양한 요청과 응답, 포인트 투 포인트 및 Pub - Sub 패턴을 사용 
2. 푸시 기반 메시지 처리  
3. SOA를 위한 플랫폼
3.1. SOA는 Service Oriented Architecture의 약자
3.2. 마이크로 서비스에 적용성이 좋음
4. pub과 sub의 시간차가 매우 작다.

### Apache Kafka   
1. 대용량 Pub - Sub 및 스트리밍에 특화
2. Key - Value 와 Timestamp로 구성된 메시지  
2.1. 타임윈도우 적용이 매우 쉽다.  
2.2. 일정 시간 내에 메시지가 소멸되지 않는다.  
3. 다양한 기능을 사용하기 위해서는 외부 서비스가 동반돼야 한다.    
3.1. Apache Zookeeper 등.
4. 풀 기반 메시지 처리  

#### 이해를 돕기 위한 자료
![Kafka vs RabbitMQ_1](../assets/img/post/kafka_vs_rabbitmq_message_management.PNG)  

![Kafka vs RabbitMQ_2](../assets/img/post/kafka_vs_rabbitmq_message_routingPNG.PNG)  

![Kafka vs RabbitMQ_3](../assets/img/post/kafka_vs_rabbitmq_message_processing.PNG)  

##### 풀 vs 푸시
*   풀 기반 : 소비자가 메시지 브로커에서 메시지를 요청하고 이를 받는 형식
* 푸시 기반 : 메시지 브로커가 소비자에게 메시지를 전달하고 이를 받는 형식  

## 결론
1. 대용량 트래픽은 아직 필요없다. 
2. 메시지 대기량에 따라 소비자를 능동적으로 줄이고 늘리는 기능을 개발할 것이니 푸시 기반이 더 어울린다.
3. 현재 어플리케이션들을 마이크로 서비스화하는 작업을 진행 중이다.
4. 소비자에서 실행되는 로직이 종료 시간을 예상할 수 없는 구조이다. 순차적으로 수행하게 되면 메시지 처리에 지연이 올 수 있다.

**Kafka 보단 RabbitMQ가 더 적절하다고 판단된다.**
 
## 다음 포스팅 : RabbitMQ 테스트 환경 구축 및 메시지 처리 테스트