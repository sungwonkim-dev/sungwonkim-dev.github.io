---
layout: post
title: RabbitMQ를 활용한 실시간 데이터 가공 시스템 개선 (3) - 시스템 구성
date: 2020-02-13 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, rabbitmq, database, application-design] # add tag
---

# RabbitMQ를 활용한 실시간 데이터 가공 시스템 개선
## 시스템 구성
### 시스템 구성 고려사항
1. 메시지 유실 방지를 위해 미러 서버가 있어야 한다.
2. 해당 시스템에서 장애가 발생할 경우 전체 시스템의 지연을 유발시킬 수 있기 때문에 프록시 기능이 필요하다.  
2.1. 매니지먼트 어플리케이션, MQ 서버
 
### 해결 방향  
1. RabbitMQ 서버를 2개 생성하고 이 둘을 클러스터로 설정한다.
1.1. 메인 서버는 RAM, 미러 서버는 DISC 모드로 설정
2. 프록시 데몬을 활용하여 장애 발생시 미러 서버를 메인 서버로 사용할 수 있도록 설정

### 시스템 구성 계획
![RabbitMQ_Proxy_System](../assets/img/post/live-data-processing-system-rabbitmq.png)
  
### RabbitMQ 설치 및 매니지먼트 앱 실행  

```
#환경 구축  
yum -y install epel-release # EPEL 저장소 설치  
yum -y update               # 업데이트  
yum -y install erlang socat # rabbitmq 구성 언어, 소켓 통신   

erl -version # 설치 및 적용 확인, Erlang (ASYNC_THREADS,HIPE) (BEAM) emulator version 5.10.4  

# rabbitmq 환경 구축  
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm #  rpm 다운로드  
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc # 키 등록  
rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm # 설치  

systemctl start rabbitmq-server  # 서버 실행  
systemctl enable rabbitmq-server # 서버 부팅마다 실행하도록 설정  
systemctl status rabbitmq-server # 서버 상태 확인  

# 방화벽 설정, 필요시 진행  
firewall-cmd --zone=public --permanent --add-port=4369/tcp
firewall-cmd --zone=public --permanent --add-port=25672/tcp
firewall-cmd --zone=public --permanent --add-port=5671-5672/tcp
firewall-cmd --zone=public --permanent --add-port=15672/tcp
firewall-cmd --zone=public --permanent --add-port=61613-61614/tcp
firewall-cmd --zone=public --permanent --add-port=1883/tcp
firewall-cmd --zone=public --permanent --add-port=8883/tcp

firewall-cmd --reload

# 매니지먼트 설정  
rabbitmq-plugins enable rabbitmq_management
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/

rabbitmqctl add_user admin "password" # 계정 추가  
rabbitmqctl set_user_tags admin administrator  # 태그 등록  
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"  # 권한 설정  

#접속 URL  
http://ip:15672/ # remote server ip or localhost  
```

#### 접속 화면
![RabbitMQ_Management_ui_home](../assets/img/post/live-data-processing-system-rabbitmq-management-ui.png)
**admin과 입력한 패스워드로 로그인해서 위와 같은 화면이 나온다면 구축 성공이다.**  
**서버 하나는 만들었으니 해결 방안을 적용해보자**
### 해결방안 적용
#### RabbitMQ 클러스터 적용 및 이중화
1. 미러서버로 사용할 RabbitMQ 환경 구축  
1.1. 위 "RabbitMQ 설치 및 매니지먼트 앱 실행" 반복  
2. 클러스터 설정    
**클러스터 설정에는 두 가지 방법이 있다. 아래 나오는 문항은 설정 순서가 아닌 방법을 나열한 것이다.**  
2.1. [메뉴얼](https://www.rabbitmq.com/clustering.html) 따라하기  
2.2. 메뉴얼을 정리한 아래 스크립트 따라하기  

```
# 쿠키 복사, 같은 쿠키로 설정해야 통신이 가능하다.
[sungwonkim@rabbit-main]   vi /var/lib/rabbitmq/.erlang.cookie  #복
[sungwonkim@rabbit-mirror] vi /var/lib/rabbitmq/.erlang.cookie  #붙

# 서버 상태 확인, 연결된 노드가 하나임을 확인하는 것임
[sungwonkim@rabbit-main] rabbitmqctl cluster_status
{nodes,[{disc,['rabbit@rabbit-main']}]}... (후략)
[sungwonkim@rabbit-mirror] rabbitmqctl cluster_status
{nodes,[{disc,['rabbit@rabbit-mirror']}]}... (후략)


# 각 서버를 클러스터로 등록, 아래 예제는 mirror를 main에 등록하는 과정
# 참고로 stop하면 RabbitMQ서버가 종료된다. 반드시 stop_app을 하자
[sungwonkim@rabbit-mirror] rabbitmqctl stop_app 

# 디스크 모드로 메인 노드에 등록
[sungwonkim@rabbit-mirror] rabbitmqctl join_cluster --disc rabbit@rabbit-main
[sungwonkim@rabbit-mirror] rabbitmqctl start

# 메인 노드를 DISC 모드에서 RAM 모드로 변경
[sungwonkim@rabbit-main] rabbitmqctl stop_app
[sungwonkim@rabbit-main] rabbitmqctl change_cluster_node_type ram
[sungwonkim@rabbit-main] rabbitmqctl start_app

# 클러스터 등록 확인
[sungwonkim@rabbit-main] rabbitmqctl cluster_status
Cluster status of node 'rabbit@rabbit-main'
[{nodes,[{disc,['rabbit@rabbit-mirror']},{ram,['rabbit-main']}]},
 {running_nodes,['rabbit@rabbit-mirror','rabbit@rabbit-main']},
 {cluster_name,<<"rabbit@temp.temp.temp">>},
 {partitions,[]},
 {alarms,[{'rabbit@rabbit-mirror',[]},{'rabbit@rabbit-main',[]}]}]
```

3. 프록시 설정

```  
# 별도 서버를 구축하지 않고 main에서 프록시 설정
[sungwonkim@rabbit-main] yum install haproxy
[sungwonkim@rabbit-main] vi /etc/haproxy/haproxy.cfg
-------------------------------------------------------------------------
# 기본 설정을 따르되 추가된 설정만 부여
listen rabbitmq
        bind    :5670 
        mode    tcp
        balance first
        timeout client 3h
        timeout server 3h
        server  rabbit-main   your.main.rabbitmq.ip:5672 check inter 3000 rise 2 fall 5
        server  rabbit-mirror your.mirror.rabbitmq.ip:5672 check inter 3000 rise 2 fall 5

frontend front_rabbitmq_management
  bind :15670
  default_backend back_rabbitmq_management

backend back_rabbitmq_management
  balance first
  server rabbitmq-mgmt-main   your.main.rabbitmq.ip:15672 check
  server rabbitmq-mgmt-mirror your.mirror.rabbitmq.ip:15672 check
-------------------------------------------------------------------------

# 실행
[sungwonkim@rabbit-main] /sbin/haproxy -f /etc/haproxy/haproxy.cfg
```

### 메시지 생산과 소비 in Java with Spring Boot
#### 의존성
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
</dependencies>
```
#### 설정 파일
```
spring.rabbitmq.host=your.main.rabbitmq.ip
spring.rabbitmq.port=5670
spring.rabbitmq.username=username
spring.rabbitmq.password=password
```

#### 메시지 생산
```java
@Configuration
public class RabbitMQConfiguration {
    private static final String QUEUE_NAME = "your.queue.name";
    private static final String EXCHAGE_NAME = "";

    @Bean
    Queue queue() {
        return new Queue(QUEUE_NAME, false);
    }

    @Bean
    DirectExchange exchange() {
        return new DirectExchange(QUEUE_NAME);
    }

    @Bean
    Binding binding(Queue queue, DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(QUEUE_NAME);
    }

    @Bean
    RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory,
                                  MessageConverter messageConverter) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter);
        return rabbitTemplate;
    }

    @Bean
    MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}

@Service
@RequiredArgsConstructor
public class RabbitMQMessageProducer {
    private final RabbitTemplate rabbitTemplate;
    public void produce(byte[] messageBytes) {
        MessageProperties properties = new MessageProperties();
        Message message = new Message(messageBytes,properties);
        rabbitTemplate.send("your.queue.name",message);
    }
}
```
#### 메시지 소비
```java
@Service
public class RabbitMQMessageConsumer {
    @RabbitListener(queues = "your.queue.name")
    public void receiveMessage(final Message message){
        System.out.println(message);
    }
}
```
**결론 : 간단한 설정으로 구성과 메시지 생산 소비를 했지만 이것을 했다고 RabbitMQ를 다룰 줄 아는 것은 아니다.**  
**다양한 옵션을 찾아가며 성능에 대한 연구가 필요하다.** 
 
## 다음 포스팅 : RabbitMQ 옵션 및 성능 테스트