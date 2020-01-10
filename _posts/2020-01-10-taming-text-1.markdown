---
layout: post
title: 텍스트 길들이기 (1) - 인명, 지명, 사물 식별
date: 2020-01-10 00:00:00 +0300
description: # Add post description (optional)
img: #software.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [java, nlp, text, taming text] # add tag
---

# 인명, 지명, 사물 식별
## 목표 : 개체명 인식에 대한 이해와 OpenNLP를 활용한 예제 구현
 
### 개체명 인식이란?    
1. 비정형 텍스트의 개체명 언급을 인명, 단체, 장소, 의학 코드, 시간 표현, 양, 금전적 가치, 퍼센트 등 미리 정의된 분류로 위치시키고 분류시키는 정보 추출의 하위 태스크.[\^1]
2. NER로도 불림 

#### 조금 더 쉽게 이해해보자
>> 김성원은 오전 12시에 국밥을 먹었다.  

위 문장을 NER 시스템은 아래와 같은 개체명들로 인식할 것이다.  
>> 김성원, 오전 12시, 국밥  

**자동으로 문서의 태그를 생성하는 것에 사용할 수도 있겠다.** 

### Open NLP를 사용해서 개체 인식을 해보자
**예제는 영문으로 작성된 문장만 사용합니다.**
#### 이름 찾기
**maven**
```xml
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-tools</artifactId>
    <version>1.9.1</version>
</dependency>
```
**code**
```{.java       
String sentence = "Nancy Davis Reagan (born Anne Frances Robbins; July 6, 1921 – March 6, 2016) was an American film actress and the wife of Ronald Reagan, the 40th president of the United States. She was the first lady of the United States from 1981 to 1989";
File file = new File("en-ner-person.bin");
FileInputStream stream = new FileInputStream(file);
TokenNameFinderModel model = new TokenNameFinderModel(stream);
NameFinderME finder = new NameFinderME(model);

Tokenizer tokenizer = SimpleTokenizer.INSTANCE;
String[] tokens = tokenizer.tokenize(sentence);
Span[] spans = finder.find(tokens);

for (Span span : spans) {
    StringBuilder builder = new StringBuilder();
    for (int tIndex = span.getStart(); tIndex < span.getEnd(); tIndex++) {
        builder.append(tokens[tIndex]).append(" ");
    }
    System.out.println(builder.toString().trim());
    System.out.println("type : " + span.getType());
}
```
**console**
>>Nancy Davis Reagan  
>>type : person  
>>Anne Frances Robbins  
>>type : person  
>>Ronald Reagan  
>>type : person
  
**결론**  
_최신 기사들로 테스트 한 결과 품질이 떨어진다_  
_고도화하려면 별도로 train을 해야 한다._
  
  
**깊은 내용은 다음 편에..**  
  

[^1]: "개체명 인식 - 위키 백과", 위키피디아, 2020년 01월 10일 접속, https://ko.wikipedia.org/wiki/%EA%B0%9C%EC%B2%B4%EB%AA%85_%EC%9D%B8%EC%8B%9D.
