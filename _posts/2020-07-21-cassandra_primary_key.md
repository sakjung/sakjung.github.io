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

Cassandra는 Uber, Netflix, Twitter 등등 수많은 기업들이 사용하고 있는 대표적인 NoSQL 데이터 베이스이다. NoSQL도 Relational Database와 같이 primary Key 개념이 있지만 조금 다르게 쓰인다. [Stackoverflow](https://stackoverflow.com/a/24953331/12982476)에 잘 설명 된 답변이 있어서 여기서 풀어 보고자 한다.

## Primary Key

Cassandra에서도 `primary key`는 각 row의 유니크함을 보장하고자 하는 목적도 있지만, 더 나아가 query에서 쓰일 column들을 규정한다는 점에 더 초점을 맞춘다.<br><br>
`Primary key`는 하나의 column으로 구성될 수도 있고 다수의 column들로 구성 될 수도 있다. 전자를 **simple primary key** 라고 하고 후자를 **composite (compound) primary key** 라고 한다.

1. **Simple Primary Key**

	```
	create table simple (
				key text PRIMARY KEY,
				data text      
		);
	```

2. **Composite Primary Key**

	```
	create table composite (
			key_part_one text,
			key_part_two int,
			data text,
			PRIMARY KEY(key_part_one, key_part_two)      
	);
	```
	
	`primary key`의 첫 번째 부분은 ***PARTITION KEY***라고 한다 (i.e. key_part_one)<br>
	`primary key`의 두 번째 부분은 ***CLUSTERING KEY***라고 한다 (i.e. key_part_two) 


	여기서 **partitoin key** 와 **clustering key** 는 여러개의 칼럼으로 구성될 수 있다. 다음 예시를 보자.

	```
	create table multiple (
	      k_part_one text,
	      k_part_two int,
	      k_clust_one text,
	      k_clust_two int,
	      k_clust_three uuid,
	      data text,
	      PRIMARY KEY((k_part_one, k_part_two), k_clust_one, k_clust_two, k_clust_three)      
	);
	```

	(k_part_one, k_part_two): **partition key**<br>
	k_clust_one, k_clust_two, k_clust_three: **clustering key**

각 key들을 정리를 하자면,<br>

- **Partition Key**: 데이터들이 어느 node (machine)에 분배될 것인지를 결정
- **Clustering Key**: Partition 내에서 데이터를 sort (정렬)하는 역할 
- **Primary Key**: 칼럼이 하나 뿐일 때, 즉 simple primary key 일때 Partition Key와 동일한 의미를 가짐
- **Composite (Compound) Key**: 여러개의 칼럼으로 구성된 key를 뜻함

## Query

Cassandra의 query (CQL)에서는 `where` 구문의 사용에 신경을 써야한다. 예시와 함께 알아보자.

1. **Simple Primary Key**

	```
	insert into simple (key, data) VALUES ('jung', 'babo');
	select * from simple where key='jung';
	```
	
	**Output**
	```
	key | data
	----+------
	jung | babo
	```
	
2. **Compostie Primary Key**

	우선 clusering key가 존재 하더라도 오직 partition key만을 where 구문에 사용할 수 있다.
	
	```
	insert into composite (key_part_one, key_part_two, data) VALUES ('ronaldo', 9, 'football player');
	insert into composite (key_part_one, key_part_two, data) VALUES ('ronaldo', 10, 'ex-football player');
	select * from composite where key_part_one = 'ronaldo';
	```
	
	composite 테이블에 데이터를 넣어 주고 partition key (key_part_one)만을 이용해서 query를 해보았다.<br>
	
	**Output**
	```
	key_part_one | key_part_two | data
	--------------+--------------+--------------------
	  ronaldo |            9 |    football player
	  ronaldo |           10 | ex-football player
	```

	 그리고, partition key와 clustering key를 모두 사용하여 query 하는 것도 가능하다
	 
	 ```
	 select * from composite 
	   where key_part_one = 'ronaldo' and key_part_two  = 10;
	 ```

	**Output**
	```
	 key_part_one | key_part_two | data
	--------------+--------------+--------------------
    	  ronaldo |           10 | ex-football player
	```


## 주의할 점

Partition key는 query의 where 구문에 최소한으로 필요한 column들이다. 즉, where 구문을 사용한다면 적어도 partition key들은 ***필수적으로 그리고 순서에 따라*** 들어가 주어야 한다는 뜻이다. 그리고 clsutering key들은 partition key들이 where 구문에 차례로 사용되고 난 다음에 ***primary key에 규정된 순서대로*** 사용될 수 있다. 다음과 같은 primary key가 있다고 가정하자.

```
PRIMARY KEY((col1, col2), col10, col4))
```

**가능한 where 구문** 

- col1 AND col2
- col1 AND col2 AND col10
- col1 AND col2 AND col10 AND col4

**불가능한 where 구문**

- col1 AND col2 AND col4 
- col1 과 col2를 포함하지 않는 모든 구문 (e.g. col1 AND col10 AND col4)

