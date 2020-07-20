---
layout: post
title: COPY를 이용한 BULK INSERT
#subtitle: 
#cover-img: /assets/img/path.jpg
#image: /assets/img/path.jpg
#share-img: /assets/img/path.jpg
categories: data_engineering
comments: true
---

RDBMS 운용시 아주 방대한 양의 데이터를 집어넣어야 할 때는 퍼포먼스를 신경써야 하기 마련이다. 데이터 양이 커질 수록 데이터를 기입하는 방식에 따라 소요되는 시간과 메모리 양은 많은 차이를 보이기 때문이다. 본 포스팅 에서는 `COPY`를 통한 `BULK INSERT` (한 번에 대량의 데이터를 집어넣는 방식)에 대해서 알아 보고자 한다. 이 방법은 `COPY`를 통해 대량의 데이터를 데이터 베이스에 빠르게 집어 넣을 수 있을 뿐만 아니라, file-like object 인 중간 매개체 역할의 Buffer (csv 형태)를 만들고 이를 이용함으로써 메모리 사용까지 줄일 수 있는 방법이다. 아래 그림은 이러한 일련의 과정을 시각적으로 보여준다.

<span style="display:block;text-align:center">![Copy Data From a String Iterator](/img/copy_bulk_insert.png)</span>
<div style="text-align:center">
	Copy Data From a String Iterator <a href="https://yuml.me/edit/64d10485">(yuml.me)</a>
</div>

******

**RDBMS: PostgreSQL**<br>
**Python - psycopg2**

```python
import psycopg2
from typing import Iterator, Optional
import io
from typing import Any
import pandas as pd
```

우선 필요한 package들을 import 해준다. 아래 두 코드 블록은 [Haki Benita](https://hakibenita.com/fast-load-data-python-postgresql) 의 블로그를 참고 했다.

```python
def clean_csv_value(value: Optional[Any]) -> str:
    if value is None:
        return r'\N'
    return str(value).replace('\n', '\\n')
```

`clean_csv_vale function` 을 만들어 준다. 이 function은 json 형태로 들어온 API 데이터를 csv 형태로 바꿔주기 위해 데이터를 정돈(?)하는 역할을 한다. 추후에 쓰일 PostgreSQL은 COPY를 할 때 default로 NULL값을 '\N'로 받아 들이기 때문에 None값 (empty values)들을 '\N'으로 바꿔 준다. 그리고 csv내의 실제 줄바꿈 ('\n')과 구별하기 위하여 데이터 내 모든 줄바꿈 에 대해서 backslash를 한번 더 입혀 escape 해준다 (\n -> \\n).

```python
class StringIteratorIO(io.TextIOBase):
    def __init__(self, iter: Iterator[str]):
        self._iter = iter
        self._buff = ''

    def readable(self) -> bool:
        return True

    def _read1(self, n: Optional[int] = None) -> str:
        while not self._buff:
            try:
                self._buff = next(self._iter)
            except StopIteration:
                break
        ret = self._buff[:n]
        self._buff = self._buff[len(ret):]
        return ret

    def read(self, n: Optional[int] = None) -> str:
        line = []
        if n is None or n < 0:
            while True:
                m = self._read1()
                if not m:
                    break
                line.append(m)
        else:
            while n > 0:
                m = self._read1(n)
                if not m:
                    break
                n -= len(m)
                line.append(m)
        return ''.join(line)
```

`StringIteratorIO` class는 string iterator buffer 를 만드는 역할을 한다. 나중에 **file** (e.g. csv)을 직접적으로 읽어들여 이용하는 대신 이 **file-like object** buffer가 대신 copy_from에 사용된다. file을 직접적으로 읽어 들이지 않고도 file형태를 가지는 buffer를 만들어 내기 때문에 메모리 사용량을 획기적으로 줄일 수 있다.

```python
conn = psycopg2.connect("host=127.0.0.1 dbname=mydb user=myusername password=mypassword")
cur = conn.cursor()
```

이제 db에 연결을 해주고 cursor를 만들어 주자. 이제 user들의 log json데이터에서 user 데이터를 추출하여 db에 집어넣는 상황을 가정해보자.

```python
user_table_create = ("""CREATE TABLE users (
        user_id int,
        first_name varchar,
        last_name varchar,
        gender varchar,
        level varchar,
        PRIMARY KEY (user_id)
);
""")

cur.execute(user_table_create)
conn.commit()
```

users table을 만들어 주었다. 본 포스팅은 필자의 local environment에 저장된 json 파일을 가지고 실험을 해보았다. API를 통해서 json 파일을 받는 상황이라면 그에 맞게 코드를 조정하면 되겠다. 우선 데이터들을 살펴보고 추가적인 처리가 필요한지 알아보기 위해서 dataframe 으로 읽어들여 보았다.

```python
filepath = './data/log_data/2018/11/2018-11-29-events.json' 
# 여러개의 json 파일들이 한줄 씩 나열되어 있는 파일 즉 '{key:value pairs}\n{key:value pairs}\n...' 형태 

df = pd.read_json(filepath, lines=True)
user_df = df[['userId', 'firstName', 'lastName', 'gender', 'level']]

# 다양한 방법으로 dataset들을 살펴보자 
```

dataframe을 확인하여 데이터를 전반적으로 살펴보았다. 이제 csv file-like object를 만들어 보도록 하자. API를 통해 json 파일을 받는 상황을 가정하였으므로, dataframe을 다시 json 형식으로 바꾸어 주었다.

```python
user_log = user_df.to_json(orient='values')
users = json.loads(user_log)
# users를 'list of lists' iterator로 변환 시켰다 

user_string_iterator = StringIteratorIO((
    '|'.join(map(clean_csv_value, (
        user[0],
        user[1],
        user[2],
        user[3],
        user[4],
    ))) + '\n'
    for user in users
))
```

**부연설명:**
1. for loop을 통해 users내의 list들을 차례로 추출한다. 각각의 list는 총 5개의 element를 가진다.
2. clean_csv_value를 각 element에 mapping 하고, 각 element들을 '\|'로 join한다. element가 ','를 포함하고 있을 수 있으므로 '\|'를 이용했다.
3. 마지막으로 줄바꿈 '\n'을 끝에 붙여주고 다음 iteration으로 넘어간다

본 예시에서는 저장된 json파일을 dataframe으로 변환 후 다시 values만 뽑아 냈기 때문에 users가 list of lists 이다. 하지만, 보통 API를 통해 json 형태의 데이터를 받는 경우 dictionary 형태인 경우가 대부분 일 것이다. 그러므로 json 데이터의 구조를 먼저 잘 파악하자.

```python
create_user_temp_table = """
    CREATE TABLE temp_u (
        user_id int,
        first_name varchar,
        last_name varchar,
        gender varchar,
        level varchar);
        """
	
temp_to_user = """
        INSERT INTO users(user_id, first_name, last_name , gender , level)
        SELECT *
        FROM temp_u
        ON CONFLICT (user_id) 
        DO UPDATE 
        SET level = excluded.level;
        
        DROP TABLE temp_u;
"""
```

users table의 경우 user_id가 primary key이다. Primary key는 테이블 내에서 중복을 허용하지 않는다. 그러므로, COPY를 통해 집어 넣으려는 데이터 중 이미 테이블내에 존재하는 user_id를 가진 데이터가 있다면 에러가 나게 된다. 이 경우 upsert개념을 이용해야하는데, COPY는 INSERT 구문과 달리 한번에 대량의 데이터를 넣으므로 ON CONFLICT DO UPDATE SET 구문을 바로 이용할 수 없다. 따라서, temporary table을 만들고 여기에 다가 COPY를 한 후, ON CONFLICT DO UPDATE SET 구문을 이용하여 temporary table 전체를 users 테이블에 insert하는 방법을 써야한다. 본 예시에서는 user_id가 중복될 경우 level을 업데이트하는 경우를 가정했다. 

```python
cur.execute(create_user_temp_table)
cur.copy_from(user_string_iterator, 'temp_u', sep='|')
# chunk size를 지정하고 싶다면 size argument를 지정해 주면 된다. default는 8192.
cur.execute(temp_to_user)

conn.commit()
conn.close()
```

위를 python이 아닌 query로써 구현한다면 다음과 같을 것이다

```
-- Update records
UPDATE users
SET level = t.level
FROM temp_u t
WHERE users.user_id = t.user_id;

-- Insert records
INSERT INTO users
SELECT t.* FROM temp_u t LEFT JOIN users
ON t.user_id = users.user_id
WHERE users.user_id IS NULL;
```

### 'psycopg2.ProgrammingError: ON CONFLICT DO UPDATE command cannot affect row a second time' 에러 가 날 경우

copy_from function 도중 '**psycopg2.ProgrammingError: ON CONFLICT DO UPDATE command cannot affect row a second time**' 에러가 발생 하는 경우가 생길 수 있다. error의 부연설명을 보면 '**HINT:  Ensure that no rows proposed for insertion within the same command have duplicate constrained values.**'라고 힌트가 주어진 걸 알 수 있다. 즉, COPY 커맨드를 한 번 실행할 때 중복이 없어야 하는 조건의 칼럼 (user_id)에 두번 이상의 UPDATE는 불가능 하다는 뜻이다. 그러므로, 이러한 경우 data cleaning을 통해 중복되는 user_id를 미리 없앤 후에 temporary table로 옮기는 작업 필요하다. 그렇다면 temporary table내에서 user_id 들은 unique해 질 것이고 insert시 필요한 user_id에 단 한번의 update를 부과할 수 있다.

```python
user_df = user_df.drop_duplicates(subset='userId', keep="last")
```

위와 같은 코드를 통하여, 중복되는 user_id중 마지막 record (row)만 남기고 모두 제거하는 방법을 쓸 수 있다. 이는 마지막에 있는 log 데이터가 가장 최신의 것이라고 가정한다면 타당한 프로세스 이다.


