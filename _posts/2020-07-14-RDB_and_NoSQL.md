---
layout: post
title: Relational Database와 NoSQL
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

## Relational Database와 NoSQL

`Relational Database`는 1970년 Edgar Codd에 의해 처음으로 고안된 데이터 베이스 형태를 의미한다. 데이터들간의 관계를 기반으로 여러 테이블들이 한 데이터 베이스 안에서 하나의 스키마 (schema)로 구성되어 있기 때문에 붙은 이름이다. `Relational Database`들을 관리하기 위한 소프트웨어를 Relational Database Management System (RDBMS)라고 한다.  
대표적으로는 Oracle, Teradata, MySQL, PostgreSQL, SQLite등이 있다.
<br>
`NoSQL`은 'Not Only SQL'을 뜻하며 'NonRelational' 이라고도 한다. `Relational Database`가 가지는 한계점을 극복하고자 나온 데이터 베이스라고 볼 수 있다. 더 간단한 디자인, 더 간단한 horizontal scaling, 더 나은 availability를 추구한다.

> **Horizontal scaling**: 처리하고자 하는 업무에 더 많은 서버, 머신, 노드등을 추가하여 더 효율적인 처리가 가능한가에 관한 특성을 말한다.  
> **Availability**: 작업자의 database의 이용 가능 여부를 말한다. 여러 조각으로 분산된 database일 수록 높은 availability를 가진다고 할 수 있다.

대표적으로는 Apache Cassandra (Parition Row store), MongoDB (Document store), DynamoDB (Key-Value store), Apache HBase (Wide Column Store), Neo4J (Graph Database)  
각각의 `NoSQL`들은 저마다 유니크한 특성을 가지고 있다
<br>
그렇다면, 언제 `Relational Database`를 쓰는게 좋고 언제 `NoSQL`을 쓰는게 좋을까? 각자의 장점에 대해서 알아보자.

******
 
 ## Relational Database 장점
 
 1. **유연한 SQL query의 활용이 가능함**:
 
 	SQL (Structured Query Language)은 가장 대표적인 쓰이는 database query 언어이다. `Relational Database` 에서는 SQL을 활용하여 아주 쉽게 database에 다양한 작업을 할 수 있다.   
	
 2. **Query가 아닌 데이터를 모델링 함**:
 
 	데이터가 query와는 독립적으로 모델링 된다. 이는 다양한 query에 대한 유연성을 보장한다.
	
 3. **테이블간의 JOIN**:  
 	
	JOIN을 통해 테이블들을 서로 합치는게 용이하다.
	
 4. **Aggregation과 analytics**:
 	
	GROUP BY, ORDER BY, SUM 등등 추후에 데이터 분석에 필수적인 aggregation이 용이하다.
 	
 5. **상대적으로 작은 데이터 사이즈에 유리함**:
 
 	Relational Database는 상대적으로 간편한 편이기 때문에, 적은 양의 데이터에 사용되기에 유리하다.
 
 6. **ACID transactions**:
 
 	[ACID의 정의](https://namu.wiki/w/Acid)<br>
	ACID특성은 에러나 전원 장애같은 상황에서도 Database transaction의 안정성과 타당성을 보장해준다. 
	
 
 7. **변화하는 비즈니스 요구에 맞추기 유리함**:
 
 	비즈니스에서 요구되는 데이터의 형태가 바뀌더라도 이에 맞춰 유연하게 database를 조작할 수 있다.



