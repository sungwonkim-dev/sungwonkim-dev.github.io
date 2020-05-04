---
layout: post
title: gRPC Load Balancing with Eureka (Java)
date: 2020-05-04 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, gRPC, load-balancing, eureka] # add tag
---

# gRPC Load Balancing with Eureka
## 목표 : gRPC 서버에 대한 클라이언트 사이드 로드 밸런싱을 구현한다.
 
### gRPC  
#### gRPC란?  
##### 어떠한 환경에서도 실행할 수 있는 고성능 RPC 프레임워크입니다.  
##### 원격 프로시저 호출은 별도의 원격 제어를 위한 코딩 없이 다른 주소 공간에서 함수나 프로시저를 실행할 수 있게하는 프로세스 간 통신 기술입니다.    

#### 언제 사용합니까?  
##### 마이크로 서비스화 된 시스템에서 각 어플리케이션의 Role에 맞는 함수를 원격으로 호출하고 싶을 때 사용합니다. (쓰고 보니 위에 설명과 동일한 말이네요...)  

#### 그러면 다수의 클라이언트가 하나의 서버를 참조할 수도 있고 하나의 클라이언트가 다수의 서버를 참조할 수도 있겠네요?
##### 맞습니다. 그래서 후자의 경우에 특정 서버에 요청이 집중될 수 있으니 클라이언트 사이드 로드 밸런싱을 하려고 합니다.

#### 그러면 동적으로 마이크로 서비스를 담당하는 서버가 생성될 때는 어떻게 처리하나요?
<pre>
1. 어플리케이션 서버 로드  
2. Spring Eureka에 어플리케이션 서버 정보 등록  
3. 어플리케이션 클라이언트 로드  
4. 클라이언트에서 Spring Eureka에 등록된 어플리케이션 서버 목록 획득
5. 로드 밸런싱하여 RPC 요청
6. 4 ~ 5 반복
</pre>
##### 위와 같은 순서로 처리합니다.    

### Spring Eureka  
#### Spring Eureka란?  
##### 마이크로 서비스의 정보를 레지스트리에 등록하여 관리하는 코디네이터입니다.  

#### 그러면 마이크로 서비스들이 Eureka 서버에 등록되고 클라이언트들은 해당 정보를 조회해서 동적으로 할당할 수 있겠네요.  
##### 맞습니다. 해당 기능이 존재했으면 좋았겠으나... 2020년 04월 28일 기준으로 존재하지 않아 직접 구현해야 하는 상황입니다... 그래서 이 글이 올라가는 것입니다.  

### 구현
```java
@Configuration
@RequiredArgsConstructor
public class RpcMultiAddressNameResolverFactory extends NameResolver.Factory {

    private final EurekaClient eurekaClient;

    @Value("${grpc.server.name}")
    private String gRpcServerName;
    private List<EquivalentAddressGroup> equivalentAddressGroups = new ArrayList<>();



    @PostConstruct
    public void init() {
        synchronized (this) {
            List<InstanceInfo> instanceInfos = eurekaClient.getApplications().getInstancesByVirtualHostName(gRpcServerName);

            if (instanceInfos.size() <= 0) {
                ServiceLogger.warn("Applications for" + gRpcServerName + " is empty... set refreshed : false...");
                refreshed = false;
                return;
            }

            equivalentAddressGroups.clear();

            for (InstanceInfo instanceInfo : instanceInfos) {
                equivalentAddressGroups.add(new EquivalentAddressGroup(new InetSocketAddress(instanceInfo.getIPAddr(), instanceInfo.getPort())));
                ServiceLogger.info("completed to add gRpc server. app name : " + instanceInfo.getAppName() + ", server : " + instanceInfo.getIPAddr(), ":" + instanceInfo.getPort());
            }
        }
    }

    public NameResolver newNameResolver(URI notUsedUri, NameResolver.Args args) {
        return new NameResolver() {
            @Override
            public String getServiceAuthority() {
                return "authority";
            }

            public void start(Listener2 listener) {
                listener.onResult(ResolutionResult.newBuilder().setAddresses(equivalentAddressGroups).setAttributes(Attributes.EMPTY).build());
            }

            public void shutdown() {
            }
        };
    }

    @Override
    public String getDefaultScheme() {
        return "name-resolver";
    }
}
```

```java

@EnableEurekaClient
@RequiredArgsConstructor
@Configuration
public class RpcLoadBalancerFactory {

    private final RpcMultiAddressNameResolverFactory rpcMultiAddressNameResolverFactory;
    private ManagedChannel managedChannel;

    @PostConstruct
    public void init() {

        if (rpcMultiAddressNameResolverFactory.isNotRefreshed()) {
            ServiceLogger.error("EurekaMultiAddressNameResolverFactory was not refreshed... skip EurekaLoadBalancerFactory initializing");
            return;
        }

        synchronized (this) {
            managedChannel = ManagedChannelBuilder
                    .forTarget("service")
                    .nameResolverFactory(rpcMultiAddressNameResolverFactory)
                    .defaultLoadBalancingPolicy("round_robin")
                    .usePlaintext()
                    .build();

            //Stub은 각 서비스에 맞게 직접 설정할 수 있도록 한다.  
        }
    }
}
```

#### 구성도
![Spring + gRPC + Eureka](../assets/img/post/2020-04-28-grpc-load-balancing-with-eureka-1.PNG)

#### 주의할 점
위 설정을 ctrl + c, v 한다고 로드 밸런싱이 완벽히 되진 않습니다.    
딱 로드 밸런싱 흉내만 낼 수 있습니다.  
eureka registry refresh interval 등 영향을 미치는 요인이 많으니 이러한 사항은 본인의 시스템에 맞게 할 수 있도록 하면 좋겠습니다.
