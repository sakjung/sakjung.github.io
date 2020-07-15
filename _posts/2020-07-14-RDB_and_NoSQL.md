---
layout: post
title: Relational Database (관계형 데이터베이스) VS NoSQL (비관계형 데이터베이스)
#subtitle: 
#cover-img: /assets/img/path.jpg
#image: /assets/img/path.jpg
#share-img: /assets/img/path.jpg
categories: data_engineering
comments: true
---
<style>
	* {
	text-align: justify}
</style>

`Relational Database`는 1970년 Edgar Codd에 의해 처음으로 고안된 데이터베이스 형태이다. 데이터들간의 관계를 기반으로 여러 테이블들이 한 데이터베이스 안에서 특정한 스키마 (schema)로 구성되어 있기 때문에 붙은 이름이다. `Relational Database`들을 관리하기 위한 소프트웨어를 Relational Database Management System (RDBMS)라고 한다. 대표적으로는 Oracle, Teradata, MySQL, PostgreSQL, SQLite등이 있다.
<br><br>
`NoSQL`은 'Not Only SQL'을 뜻하며 'NonRelational' 이라고도 한다. `Relational Database`가 가지는 한계점을 극복하고자 나온 데이터베이스라고 볼 수 있다. 더 간단한 디자인, 더 간단한 horizontal scaling, 더 나은 availability를 추구한다. 대표적으로는 Apache Cassandra (Parition Row store), MongoDB (Document store), DynamoDB (Key-Value store), Apache HBase (Wide Column Store), Neo4J (Graph Database). 괄호 속에 나타나 있듯이, 각각의 `NoSQL`들은 저마다 유니크한 특성을 가지고 있다.

> **Horizontal scaling**: 처리하고자 하는 업무에 더 많은 서버, 머신, 노드등을 추가하여 더 효율적인 처리가 가능한가에 관한 특성을 말한다.  
> **Availability**: 작업자의 데이터베이스 이용 가능 여부를 말한다. 여러 조각으로 분산된 데이터베이스일 수록 높은 availability를 가진다고 할 수 있다.


그렇다면 언제 `Relational Database`를 쓰는게 좋고, 언제 `NoSQL`을 쓰는게 좋을까? 둘의 상대적인 장단점을 간략하게 비교해보자.

******

## Relational Database VS NoSQL

|| **요구 조건** | **Relational Database** | **NoSQL** |
|:--------:|:--------:|:---------:|:---------------:|
|1|방대한 데이터 양|불리|**유리**|
|2|자주 변하는 비즈니스 요구사항|**유리**| 불리|
|3|데이터 타입이나 스키마 구조에 대한 유연성|불리|**유리**|
|4|높은 Throughput (빠른 read와 write)|불리|**유리**|
|5|Horizontal scaling|불리|**유리**|
|6|높은 availability|불리|**유리**|
|7|Data transaction의 안정성과 타당성|**유리**|불리|
|8|테이블간의 JOIN|**유리**|불리|
|9|Data Aggregation과 Analytics|**유리**|불리|

- **부연 설명**

	**요구조건 2**: `Relational Database`의 경우 테이블들 간의 관계를 이용하여 유연하게 query구문을 작성할 수 있다. 하지만 `NoSQL`의 경우 보통 사용하고자 하는 query문을 먼저 정의한 후 이에 맞춘 데이터베이스를 구성하기 때문에 추후에 query에 변화를 주기 힘들 수 있다. 

	**요구조건 4 & 7**: ACID transaction을 지원하느냐의 여부와 관련이 깊다. ACID transaction의 정의는 [나무위키](https://namu.wiki/w/Acid) 에서 확인 가능하다. `Relational Database`는 ACID transaction을 보장해 주지만 `NoSQL`은 그렇지 못하다 (하지만, MongoDB처럼 예외인 경우도 있다). ACID특성은 에러나 전원 장애같은 상황에서도 Database transaction의 안정성과 타당성을 보장해 주지만, 낮은 throughput의 원인이 되기도 한다.


