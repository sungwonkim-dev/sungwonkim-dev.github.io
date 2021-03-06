---
layout: post
title: 텍스트 길들이기 (3) - 야생의 텍스트?
date: 2020-01-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, nlp, text, taming text] # add tag
---

# 야생의 텍스트
## 목표 : 다음 개척지에 대한 탐구
 
### 문서 클러스터링까지 왔는데 다음은 어디?
**지금까지는 텍스트를 다루는 기초 지식의 이해만 했다.**  
**예를 들자면 아기들에게 "저건 아빠야", "저건 엄마야" 정도의 지식 이해라고 보면 된다.**  

**조금 고수준으로 가보자 (물론 학습은 하지 않고 겉핥기만)**
1. 의미론
1.1. 지금까지 한 것은  syntax, 앞으로는 semantics  
1.2. 단어의 의미, 더 큰 의미 단위를 구성하기 위한 단어간 상호작용에 대한 연구  
1.3. 검색을 넘어 대화 수준까지
2. 담론  
2.1. 의미를 기반으로 해서 담론 분석은 문장 간의 관계를 알아내는 것이 목적  
2.2. 의미와 같은 범주라고 생각할 수도 있다.
3. 화용론  
3.1. 맥락, 세계에 대한 지식, 언어 관습, 그 외의 기타 추상적 속성이 어떻게 텍스트의 의미에 기여하는지에 대한 연구

### 의미론, 담론, 화용론의 예시
1. 의미론  
1.1. 동의어, 반의어, 상위어/하위에 등과 같은 단어 의미와 단어 의미 명확화        
   * 금융기관을 뜻하는 bank, 강둑을 뜻하는 river bank
   * 너 정말 대단하다!!, 너 정말 대단하다~~, 너 정말 대단하다??
1.2. 관형구에 대한 의미 추론  
   * bit the dust -> (단어상 의미) 먼지를 먹다, (관형적 의미) 실패하다, 죽다
   * 해가 서쪽에서 뜨려나? -> (단어상 의미) 해가 서쪽에서 뜰지 궁금하다, (관형적 의미) 얘가 왜이래?  
   
2. 담화
2.1. 의미론이 문장 내에서 작동한다면 담화는 문장과 문장 간 관계를 본다.
   * 그럼 그렇지 -> (의미론) 그럼 그렇다, (담화) 앞에 존재하는 사건 또는 의견에 대한 동의  

3. 화용론  
3.1. 맥락과 맥락이 의사소통에 미치는 영향에 대한 것
   * 말하는 사람, 듣는 사람, 시간과 장소에 따른 영향  
   * Q : 김성원은 어디에 살고 있지?  
   * A: (대답 전) '김성원은 안산에 살고 있다', '김성원은 아파트에 살고 있다'
   * A: 안산, 아파트
   * 대답하는 사람의 배경지식이 다르니 답변도 다르다
   
### 그 외 분야
1. 문서와 컬렉션 요약
   * 단어, 문장의 중요도, 중복 단어 등 여러 요인를 활용한 요약
2. 관계 추출
   * Bill Gates is cofounder of Microsoft
   * Bill Gate, Microsoft
3. 중요 콘텐츠와 인물 식별  
   * 사람에 따라 문서에 대한 흥미가 다름
4. 정서 분석을 통한 감정 감지  
   * 영화 댓글 분석
5. 교차 언어 정보 검색
   * 한국어로 검색해도 영문 자료가 나올 수 있도록  
   * 의미 해석이 필요  