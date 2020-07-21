---
layout: post
title: Apache Cassandra - Primary Key, Partition Key 그리고 Clustering Key
#subtitle: 
#cover-img: /assets/img/path.jpg
#image: /assets/img/path.jpg
#share-img: /assets/img/path.jpg
categories: data_engineering
comments: true
---

`Cassandra`는 Uber, Netflix, Twitter 등등 수많은 기업들이 사용하고 있는 대표적인 NoSQL 데이터 베이스이다. NoSQL도 Relational Database와 같이 primary Key 개념이 있지만 조금 다르게 쓰인다. 간단하게 알아보자.

## Primary Key

`Cassandra`에서도 primary key는 각 row의 유니크함을 보장하고자 하는 목적도 있지만, 더 나아가 query에서 쓰일 column들로 구성되어야 한다는 점에 더 초점을 맞춰야 한다.  
