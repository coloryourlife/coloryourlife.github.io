---
layout: post
title: "[MongoDB]Database Migration"
date: 2021-05-24
categories: 
- web
tags: [flask, mongoDB, Database]
---

## 도입
사이드 프로젝트에서 프로젝트 초기 설정을 이것 저것 하던 도중 팀원 중 한명이 본인의 예전 프로젝트에서 `flask-migrate`이라는 라이브러리를 사용했었다고 한번 알아봐달라고 했다. 처음에 해당 라이브러리 이름을 들었을 때, `django`에서 DB 스키마가 바뀔 때마다 항상 입력해줘야 했던 `python manage.py migrate`이 생각났는데 아니나 다를까 해당 라이브러리를 도입했을 때, 장고와 같은 db migration을 편하게 관리해줄 수 있다는 것이다. 그렇게 하여 현재 진행 중인 프로젝트에서도 초기에 Database가 빈번하게 바뀔 것을 대비하여 db schema versioning을 위해 해당 라이브러리에 대해 자세하게 알아보게 되었다. 

## 결과 미리보기
알아보면서 반성하게된 점은 역시나 `django`에서 migrate을 하면서도 이걸 도대체 무엇을 위해 왜 하는지 정확하게 짚고 넘어가지 않았기 때문에 이번 기회에 해당 명령어를 통해 무엇을 어떤 것을 목적으로하는지 알 수 있었다. 하지만 알아보면서 도달하게된 결론은 아이러니하게도 우리 프로젝트에서는 `migrate`을 따로 외부 라이브러리까지 사용해가면서 할 필요가 없다는 것이었다.

## Why
결론부터 얘기하자면 이번 사이드 프로젝트에서 `flask-migrate`을 사용하지 않게된 이유는 우리가 `mongodb`를 사용하기 때문이다. 관련하여 내가 가장 큰 도움을 받은 질문 및 답변, 해당 답변에서 내가 새로히 알게된 패턴에 관해 소개하고자 한다. 우선 mongodb developer community에 올라온 질문이다. (https://developer.mongodb.com/community/forums/t/migration-tool-for-python/6344/) 여기서 질문자는 내가 찾고자하는 내용에 대한 질문을 하게되는데 이는 mongodb를 프로젝트에서 사용하는데 혹시 python 기반의 웹 프레임워크를 사용함에 있어서 migrations을 도와줄 라이브러리가 존재하는지 물어본다. 일단 우리가 앞서 언급했던 `flask-migrate`은 nosql을 위한 라이브러리는 아니고 `Flask-SQLAlchemy`에서 지원하는 데이터베이스에 대해서 migration을 도와준다. 그렇기에 나와 질문자는 mongodb를 사용함에 있어 flask와 연동 가능한 migration 라이브러리를 찾고있었고 여기서 MongoDB 직원분이 남겨주신 답변은 나를 한번 더 반성하게끔 만들었다.

## MongoDB의 핵심은 NoSQL
MongoDB를 설계하면서 내가 가장 고개를 갸우뚱했던 순간들은 모두 NoSQL 기반인 MongoDB를 SQL 기반의 관계형 데이터베이스처럼 설계하고 있음을 느꼈을 때였다. 학교 수업 때 배우고 그래도 규모있는 데이터베이스를 설계했을 때 사용했던 것들이 전부다 SQL 기반의 데이터베이스였기 때문에 `Schemaless`한 NoSQL을 사용하면서도 나도 모르게 관계형 데이터베이스 처럼 설계를 하고있었던 것이다. 그리고 다시 한번 생각해보니 우리가 이번 프로젝트에서 MongoDB를 한번 사용해보기 위해 선택해 놓고, NoSQL이라 이 친구를 무시하고 너무 쉽게 여겨 기본적인 디자인 패턴조차 공부하지 않고 다루려고 했었음을 깨달았다. 이에 다시 한번 개발을 쉽게 생각하고 그냥저냥 넘어가려고 했던 나의 모습을 반성하고 위의 나와 같은 고민을 한 친구의 질문에 직원분이 남겨주신 답변을 얘기해보겠다. 

답변은 간결하면서도 나에게는 묵직하게 다가왔다. 답변의 핵심은 `MongoDB`의 생태계에서 migrations는 그다지 중요하지 않다는 것이었다. 그 이유는 기본적으로 NoSQL이기 때문에 `schema-free nature`한 데이터베이스 이기 때문이다. 이 말로 정리해보면 migration을 해야되는 순간은 우리가 관계형 데이터베이스의 스키마의 변경이 일어났을 때, 해당 수정 사항을 Database에 반영시키기 위해 migration을 사용한다. SQL을 기반으로한 데이터베이스는 기본적으로 정형화된 데이터베이스 스키마를 사용하기 때문에 이와 같이 수정이 일어났을 때, 이를 반영해야하는 과정이 불가피한 것인 것이다. 하지만 이와 달리 MongoDB와 같은 `NoSQL` 기반의 데이터베이스는 기본적으로 정형화된 스키마가 필수도 아니고 사실 데이터베이스 스키마를 정형화하지 않는 것이 SQL과 비교했을 때의 가장 큰 차이이기 때문에 `schema`라는 것 자체를 변경할 필요가 없는 것이고 그렇기 때문에 migration 작업이 별도로 필요하지 않는 것이다. 

NoSQL이 기본적으로 데이터 스키마가 존재하지는 않지만 사실상 우리는 DB의 일관성 유지를 위해 어느정도 Document 내에서의 스키마를 정해놓고 일관적으로 사용하기는 한다. 그렇기 때문에 이러한 변경이 일어났을 때 그래도 Document내의 스키마의 버저닝을 관리를 하긴 해야 명시적으로 작업이 선순환을 이룰 수 있을 것이다. 그래서 답변을 달아준 직원은 `Schema Versioning Pattern`을 사용해보라는 말과 함께 마무리했다.

## Schema Versioning Pattern
먼저 해당 패턴을 제시하는 mongodb측의 posting link를 공유한다. https://www.mongodb.com/blog/post/building-with-patterns-the-schema-versioning-pattern
해당 문서에서 제시하는 Sample Use Case를 빌려와 해당 패턴에 대한 이해도를 높였다. 문서에서 제시하는 상황은 다음과 같다.

```
{
	"_id": "<ObjectId>",  
	"name": "Anakin Skywalker",   
	"home": "503-555-0000",   
	"work": "503-555-0010"
}
```

다음과 같은 original schema를 사용해 서비스를 진행하다가 어느 순간 사용자의 핸드폰 번호까지 저장을 해야되는 시기가 왔다고 생각해보자. 그렇다면 앞선 스키마와 다르게 바뀌어야 할 스키마는 다음과 같을 것이다.

```
{
	"_id": "<ObjectId>",  
	"name": "Anakin Skywalker",   
	"home": "503-555-0000",   
	"work": "503-555-0010",
	"mobile": "503-555-0120"
}
```

앞서 사용하던 Database Schema와 다르게 마지막에 `mobile`이 추가가 되면서 관계형 데이터베이스에서의 migration이 필요한 상황이 만들어졌다. 우리는 이 때 해당 스키마의 버저닝 관리를 위해 단순히 document에 `schema_version`을 명시함으로써 버전을 관리할 수 있고 이를 적용한 document는 다음과 같다.

```
{
	"_id": "<ObjectId>",  
	"schema_version": "2",
	"name": "Anakin Skywalker",   
	"home": "503-555-0000",   
	"work": "503-555-0010",
	"mobile": "503-555-0120"
}
```	

이런식으로 MongoDB에서는 단순히 document 내에 스키마 버전을 명시함으로써 버전 관리를 할 수 있게된다. 이것이 MongoDB에서 나와 질문한 친구에게 최종적으로 던진 solution이며 이러한 버저닝 패턴으로 관리하는 것이 MongoDB스러움을 배울 수 있었다.


