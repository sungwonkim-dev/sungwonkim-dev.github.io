---
layout: post
title: 나만의 코드 컨벤션 (2) - 패키지, 클래스, 변수
date: 2020-01-06 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, convention] # add tag
---
## 패키지, 클래스
### 구조  

![package]({{site.baseurl}}/assets/img/post/package.PNG)

#### 패키지에 대한 나만의 약속과 예시
1. 관점별로 분리하자.
    - 관리자 입장에서 사용할 클래스들은 admin 패키지
    - 회원 입장에서 사용할 클래스들은 user 패키지
    - 비회원 입장또는 공용으로 사용할 클래스들은 common 패키지
     
2. 컴포넌트별로 분리하자.
    -  Controller, Service, Repository

3. 전역으로 영향을 미치는 항목은 root 패키지에서 관리하자.
    - Security Config
    - Application  
    
#### 클래스에 대한 나만의 약속과 예시  
1. Entity는 DB 테이블의 카멜 표기법으로 작성한다.  
    - discussion_post -> DiscussionPost 
    
2. 추상 클래스를 선호한다.
    - __동일한 속성__ 이 있다면 추상클래스를 활용하는 것을 선호한다.  
    
3. Helper 클래스보다 Util 클래스를 선호한다.  

4. 컴포넌트 클래스는 suffix에 컴포넌트명을 사용한다.
    - Admin __Controller__
    - Log __Service__
    - Log __Repository__
    
    
**상황에 따라 지키지 못 하는 경우도 있겠지만 최대한 지키려고 한다.**