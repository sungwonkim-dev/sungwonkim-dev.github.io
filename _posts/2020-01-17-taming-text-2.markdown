---
layout: post
title: 텍스트 길들이기 (2) - 텍스트 클러스터링
date: 2020-01-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, nlp, text, taming text] # add tag
---

# 인명, 지명, 사물 식별
## 목표 : 텍스트 클러스터링에 대한 이해와 아파치 머하웃을 사용한 클러스터링, Carrot^2를 사용한 검색 결과 클러스터링 예제 구현
 
### 텍스트 클러스터링이란?    
1. 기계학습을 통해 유사한 텍스트를 그룹핑하는 것.
2. 라벨이 붙지 않은 문서를 어떤 유사도에 따라 그룹으로 묶는 일 (비지도 학습)

#### 텍스트 클러시터링 예시
>> 구글에 특정 문장이나 단어를 검색하면 내용이 거의 다 비슷하다.  
>> -> "how to solve handshake failure in java"  

**구글은 뉴스나 문서 제공시 클러스터링을 사용한다고 밝혔었다. (2011년)** 

### 클러스터링 기초를 알아보자
#### 검색에서 클러스터링은 만병통치약인가?
**비슷한 결과만 모아서 보여주면 그게 검색이지 않냐?**

미완성