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
1. 비정형 텍스트의 개체명 언급을 인명, 단체, 장소, 의학 코드, 시간 표현, 양, 금전적 가치, 퍼센트 등 미리 정의된 분류로 위치시키고 분류시키는 정보 추출의 하위 태스크. [위키피디아]
2. NER로도 불림 

#### 조금 더 쉽게 이해해보자
>> 김성원은 오전 12시에 국밥을 먹었다.  

위 문장을 NER 시스템은 아래와 같은 개체명들로 인식할 것이다.  
>> 김성원, 오전 12시, 국밥  

**자동으로 문서의 태그를 생성하는 것에 사용할 수도 있겠다.** 

### Open NLP를 사용해서 개체 인식을 해보자
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
```.java       
String sentence = "Nancy Davis Reagan (born Anne Frances Robbins; July 6, 1921 – March 6, 2016) was an American film actress and the wife of Ronald Reagan, the 40th president of the United States. She was the first lady of the United States from 1981 to 1989";
File file = new File("en-ner-person.bin"); // 미리 만들어진 훈련 모델
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

#### 개체명 인식 모델 고도화
1. 특정 도메인에 속하는 데이터로 모델을 만들어라
  * 기사에 사용할 것이면 기사만, 논문에 사용할 것이라면 논문에서 추출한 데이터만 사용
  * 참고로 Open NLP의 모델들은 뉴스 기사로 훈련된 모델이다.
2. 1번으로 만들어진 모델로 예측한 결과를 결합하라
  * 재현율이 높아질 것이다.  

#### 고도화 영역을 **혼자** 진행하기 어려운 이유
* 다량, 양질의 데이터를 모으기 어려움
  * GIGO
  * 개인용 검색 엔진이면 Ok, 단체용 검색 엔진이면 글쎄..  
      * 혼자서 모은 데이터가 단체에게도 적용되는 데이터일까
  
#### 개체명 인식 모델 훈련하기
```.java
InputStreamFactory inputStreamFactory = new InputStreamFactory() {
    @Override
    public InputStream createInputStream() throws IOException {
        return new FileInputStream("ko-person.train");
    }
};
    
NameSampleDataStream nameSampleDataStream = new NameSampleDataStream(new PlainTextByLineStream(inputStreamFactory, "UTF-8"));

TokenNameFinderFactory tokenNameFinderFactory = new TokenNameFinderFactory(); // 커스텀 가능    
TokenNameFinderModel model = NameFinderME.train(
    languageCode,
    "person",
    nameSampleDataStream,
    TrainingParameters.defaultParams(),
    tokenNameFinderFactory);
    
File output = new File("ko-person.bin");
FileOutputStream fileOutputStream = new FileOutputStream(output);
model.serialize(fileOutputStream);
```

#### 과연 개체명 인식이 필요할까?
**필요하다**  
- 문장형 검색이 가능하다
    - 2019년 11월 14일 목요일에 시행된 시험이 뭔가요? -> '2019년 11월 14일', '목요일', '시험' -> **대학수학능력시험**  
- 목적이 명확하지 않은 검색 쿼리에 강하다
    - 아 이 사이트에서 뭔가 본 거 같은데 정확히는 모르겠고..
        - 깃헙 개발자 김씨 컨벤션
        - ex) [컨벤션] site:[깃헙 개발자 김씨]
   
**필요하지 않다**  
- 사용자가 키워드 또는 exactly query만 사용하도록 요구한다.
    -  단, 사용 목적이 확실할 경우에만 적용된다.
    - ex) 책 제목 검색 등

**모르겠다**  
- 사람에 따라 다르다
   - 모든 사람의 입맛에 맞게 만들 수 없으니 있어서 나쁠 거 있나?
   - 쓸 사람만 써라라는 마인드면 없어도 좋다

**결론 : 답은 없다. 요구에 맞게 설계해라.**  

[위키피디아]: "개체명 인식 - 위키 백과", 위키피디아, 2020년 01월 10일 접속, https://ko.wikipedia.org/wiki/%EA%B0%9C%EC%B2%B4%EB%AA%85_%EC%9D%B8%EC%8B%9D.
