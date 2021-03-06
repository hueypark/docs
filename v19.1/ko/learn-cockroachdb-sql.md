---
title: CockroachDB SQL을 배우자
summary: 가장 필요한 CockroachDB SQL 구문들을 배우자
toc: true
build_for: [standard, managed]
---

이 페이지는 여러분이 가장 중요한 CockroachDB SQL 구문을 쉽게 알 수 있게 도와줄 겁니다. 완전한 목록 그리고 연관된 더 자세한 사항을 보고 싶으시다면 [SQL Statements](sql-statements.html)을 참고하세요.

{% unless site.managed %}
{{site.data.alerts.callout_success}}
이 구문들을 시도하려면 interactive SQL shell을 사용해보세요. 만약 당신의 클러스터가 이미 실행되고 있다면, [`cockroach sql`](use-the-built-in-sql-client.html) 이 명령을 사용하세요. 다른 방법을 사용해보자면 메모리 클러스터 안에 있는 일시적인 쉘을 열기위해서 [`cockroach demo`](cockroach-demo.html) 명령어를 사용해보세요. 
{{site.data.alerts.end}}
{% endunless %}

{{site.data.alerts.callout_info}}
CockroachDB는 확장성 있는 standard SQL을 제공하는 것을 목표로 합니다. 하지만 어떤 standard SQL 기능은 아직 가능하지 않습니다. 자세한 사항을 확인하고 싶으시다면 다음 링크를 참고해주세요. [SQL Feature Support](sql-feature-support.html) 
{{site.data.alerts.end}}

{% if site.managed %}
## 시작하기 전에

여러분이 여러분의 클러스터로  [the CockroachDB SQL client](managed-connect-to-your-cluster.html#use-the-cockroachdb-sql-client)를 연결했는지 여부를 꼭 확인하셔야 합니다.

## 데이터 베이스 만들기

당신에게 관리되는 CockroachDB 클러스터는  `defaultdb` 와 어떤 내부의 데이터베이스들과 합께 있습니다.

새로운 데이터베이스를 만들기 위해서 당신의 초기 "admin" 사용자와 연결하고  [`CREATE DATABASE`](create-database.html) 뒤에 이름을 붙여서 사용하십시오.

{% include copy-clipboard.html %}
~~~ sql
> CREATE DATABASE bank;
~~~

데이터베이스의 명명법은 다음 링크를 따라야 합니다. [these identifier rules](keywords-and-identifiers.html#identifiers). 데이터베이스가 존재하는 경우 에러를 피하기 위해,  `IF NOT EXISTS` 를 포함시킬 수 있습니다:

{% include copy-clipboard.html %}
~~~ sql
> CREATE DATABASE IF NOT EXISTS bank;
~~~

더이상 데이터베이스가 필요하지 않을 때,   [`DROP DATABASE`](drop-database.html) 를 사용하세요. 그리고 데이터베이스와 그 안에 있는 모든 것을 지우기 위해 뒤에 데이터베이스의 이름을 붙이세요:

{% include copy-clipboard.html %}
~~~ sql
> DROP DATABASE bank;
~~~

## 데이터베이스 보여주기

모든 데이터베이스를 보기 위해서, [`SHOW DATABASES`](show-databases.html) 구문을 사용하세요:

{% include copy-clipboard.html %}
~~~ sql
> SHOW DATABASES;
~~~

~~~
  database_name
+---------------+
  bank
  defaultdb
  postgres
  system
(4 rows)
~~~

## 초기 데이터베이스 설정

초기 데이터 베이스 설정을 위해 가장 좋은 방법은 [connection string](managed-sign-up-for-a-cluster)을 넣어주시면 됩니다.

{% include copy-clipboard.html %}
~~~ sql
> SET DATABASE = bank;
~~~

기본 데이터베이스에서 작업 할 때는 명령문에서 명시적으로 참조 할 필요가 없습니다. 어떤 데이터베이스가 현재 초기 디폴트인지 알기위해선  `SHOW DATABASE` 명령을 사용하시면 됩니다. (단수형인 것에 주목해야 해요.):

{% include copy-clipboard.html %}
~~~ sql
> SHOW DATABASE;
~~~

~~~
  database
+----------+
  bank
(1 row)
~~~
{% endif %}

## 테이블 

테이블을 만들기 위해서는 [`CREATE TABLE`](create-table.html)을 쓰고 뒤에 테이블 이름, column 이름 그리고 [데이터 타입](data-types.html) and [제약](constraints.html) 이 존재하는 경우에 대하여 사용하세요:

{% include copy-clipboard.html %}
~~~ sql
> CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL
);
~~~

테이블과 column의 이름 붙이는 것은 이 [규칙](keywords-and-identifiers.html#identifiers)을 따라야 합니다. 또한 만약 당신이 명시적으로 [primary key](primary-key.html)를 정의하지 않았다면, CockroachDB는 자동적으로 primary key로서 숨겨진 `rowid` column을 자동으로 생성할 것입니다.

테이블이 이미 존재하는 경우 에러를 피하기 위하여 당신은 `IF NOT EXISTS`를 포함시킬 수 있습니다:

{% include copy-clipboard.html %}
~~~ sql
> CREATE TABLE IF NOT EXISTS accounts (
    id INT PRIMARY KEY,
    balance DECIMAL
);
~~~

테이블로부터 모든 column들을 보여주기 위해서 [`SHOW COLUMNS FROM`](show-columns.html) 이란 명령어를 사용하시고 뒤에 테이블의 이름을 적어주세요:

{% include copy-clipboard.html %}
~~~ sql
> SHOW COLUMNS FROM accounts;
~~~

~~~
  column_name | data_type | is_nullable | column_default | generation_expression |   indices   | is_hidden
+-------------+-----------+-------------+----------------+-----------------------+-------------+-----------+
  id          | INT       |    false    | NULL           |                       | {"primary"} |   false
  balance     | DECIMAL   |    true     | NULL           |                       | {}          |   false
(2 rows)
~~~

더이상 테이블을 필요가 없다면 [`DROP TABLE`](drop-table.html)을 사용하십시오. 그리고 뒤에 테이블의 이름을 붙이면 그 테이블과 그 안에 있는 모든 데이터들이 사라지게 됩니다:

{% include copy-clipboard.html %}
~~~ sql
> DROP TABLE accounts;
~~~

## 테이블 보여주기

유효한 모든 데이터베이스를 보기 위해서 [`SHOW TABLES`](show-tables.html) 명령어를 사용하세요.:

{% include copy-clipboard.html %}
~~~ sql
> SHOW TABLES;
~~~

~~~
  table_name
+------------+
  accounts
(1 row)
~~~

## 테이블에 행 넣기

테이블에 행을 넣기 위해서 [`INSERT INTO`](insert.html)명령어를 사용하세요. 그리고 뒤에 테이블 이름을 붙이고 테이블에 열이 나타나는 순서대로 나열된 열 값을 넣으세요.:

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO accounts VALUES (1, 10000.50);
~~~

만약 그대가 column 값을 다른 순서로 넘기고 싶다면 column 이름을 명시적으로 나열하고 해당 순서로 column 값을 넣어주시면 됩니다:

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO accounts (balance, id) VALUES
    (25000.00, 2);
~~~

테이블에 여러 행을 삽입하려면 쉼표로 구분 된 괄호 목록을 사용하시면 됩니다. 참고로 각 괄호는 한 행의 column 값을 포함합니다.

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO accounts VALUES
    (3, 8100.73),
    (4, 9400.10);
~~~

[Defaults values](default-value.html)는 그대가 명령문에서 어떤 특정 column을 제외할경우 사용이 됩니다. 아니면 명시적으로 초기값을 요청할 때 쓰이죠. 예를 들어보죠. 이어서 오는 두개의 명령문들 모두 `balance`가 기본값으로 채워진 행을 만들어줍니다.(이 경우는 `NULL`):

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO accounts (id) VALUES
    (5);
~~~

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO accounts (id, balance) VALUES
    (6, DEFAULT);
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM accounts WHERE id in (5, 6);
~~~

~~~
  id | balance
+----+---------+
   5 | NULL
   6 | NULL
(2 rows)
~~~

## 인덱스 만들기

[Indexes](indexes.html)는 테이블의 모든 행들을 거칠 필요 없이 데이터를 위치시킬 수 있게 도와줍니다. 테이블의 [primary key](primary-key.html)와  [`UNIQUE` constraint](unique.html)가 있는 모든 열에 대해서 자동으로 생성이 됩니다.

특별하지 않은 즉, 일반적인 column에 대해 인덱스를 만드는 방법은 [`CREATE INDEX`](create-index.html) 명령을 사용하는 것입니다. 그리고 명령어 뒤에 인덱스 이름과 테이블과 column을 확인할 수 있는 'ON' 구문이 옵니다. 각각의 column에 대해서 여러분은 ascending (`ASC`) or descending (`DESC`) 즉 오름차순, 내림차순 정렬을 원하시는 대로 할 수 있습니다.

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX balance_idx ON accounts (balance DESC);
~~~

여러분은 테이블을 만드는 과정 동안에도 인덱스를 만들 수 있습니다. 간단하게 `INDEX` 키워드를 사용하고 뒤에 선택적 인덱스 이름 그리고 인덱싱 되는 column을 넣어주세요:

{% include copy-clipboard.html %}
~~~ sql
> CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL,
    INDEX balance_idx (balance)
);
~~~

## 테이블에 인덱스 보여주기

인덱스 들을 보여주는 방법이 있습니다. [`SHOW INDEX FROM`](show-index.html) 명령어를 사용하세요. 그리고 뒤에 테이블의 이름을 넣으세요:

{% include copy-clipboard.html %}
~~~ sql
> SHOW INDEX FROM accounts;
~~~

~~~
  table_name | index_name  | non_unique | seq_in_index | column_name | direction | storing | implicit
+------------+-------------+------------+--------------+-------------+-----------+---------+----------+
  accounts   | primary     |   false    |            1 | id          | ASC       |  false  |  false
  accounts   | balance_idx |    true    |            1 | balance     | DESC      |  false  |  false
  accounts   | balance_idx |    true    |            2 | id          | ASC       |  false  |   true
(3 rows)
~~~

## 테이블 

테이블을 쿼리하려면 [`SELECT`](select-clause.html)를 사용하시고 그 뒤에 테이블을 쿼리하려면 SELECT 다음에 쉼표로 구분 된 리턴 할 열 목록과 데이터를 검색 할 테이블을 넣으시면 됩니다:

{% include copy-clipboard.html %}
~~~ sql
> SELECT balance FROM accounts;
~~~

~~~
  balance
+----------+
  10000.50
  25000.00
   8100.73
   9400.10
  NULL
  NULL
(6 rows)
~~~

모든 열을 검색하려면, `*` 이 와일드카드를 사용하십시오:

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM accounts;
~~~

~~~
  id | balance
+----+----------+
   1 | 10000.50
   2 | 25000.00
   3 |  8100.73
   4 |  9400.10
   5 | NULL
   6 | NULL
(6 rows)
~~~

결과를 필터링 하고 싶으시다면 column과 필터링 할 열 및 값 식별을 하는 `WHERE` 절을 추가하십시오:

{% include copy-clipboard.html %}
~~~ sql
> SELECT id, balance FROM accounts WHERE balance > 9000;
~~~

~~~
  id | balance
+----+----------+
   2 | 25000.00
   1 | 10000.50
   4 |  9400.10
(3 rows)
~~~

결과들을 정렬하기 위해서는 정렬을 할 열을 식별해주는 `ORDER BY` 절을 추가하세요. 각각의 column에 대해 여러분은 오름차순(ascending (`ASC`))으로 정렬할지 내림차순((descending (`DESC`)))으로 정렬할지에 대해 선택이 가능합니다. 

{% include copy-clipboard.html %}
~~~ sql
> SELECT id, balance FROM accounts ORDER BY balance DESC;
~~~

~~~
  id | balance
+----+----------+
   2 | 25000.00
   1 | 10000.50
   4 |  9400.10
   3 |  8100.73
   5 | NULL
   6 | NULL
(6 rows)
~~~

## 테이블 안에 행 업데이트하기

테이블에 있는 행을 업데이트하기 위해선 [`UPDATE`](update.html)명령어를 사용하고 뒤에 테이블 이름을 붙이세요. 그리고 그 뒤에 업데이트 할 columns와 새로운 값을 식별해주는 명령어인 `SET`을 써주시고 업데이트할 rows를 확인해주는 `WHERE` 명령어를 사용하세요:

{% include copy-clipboard.html %}
~~~ sql
> UPDATE accounts SET balance = balance - 5.50 WHERE balance < 10000;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM accounts;
~~~

~~~
  id | balance
+----+----------+
   1 | 10000.50
   2 | 25000.00
   3 |  8095.23
   4 |  9394.60
   5 | NULL
   6 | NULL
(6 rows)
~~~

테이블에 기본 키가있는 경우,`WHERE` 절에서 그것을 사용하여 특정 행을 확실하게 업데이트 할 수 있습니다. 그렇지 않은 경우,`WHERE` 절과 일치하는 행이 업데이트됩니다. `WHERE` 절이 없으면 테이블의 모든 행이 업데이트 됩니다.

## 테이블에 행 지우기

테이블에서 행을 지우기 위해선 [`DELETE FROM`](delete.html) 명령어를 사용하고 그 뒤에 테이블 이름을 쓰고 그 뒤에 또 지울 행을 확인해주는 절인 `WHERE`를 써주시면 됩니다:

{% include copy-clipboard.html %}
~~~ sql
> DELETE FROM accounts WHERE id in (5, 6);
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM accounts;
~~~

~~~
  id | balance
+----+----------+
   1 | 10000.50
   2 | 25000.00
   3 |  8095.23
   4 |  9394.60
(4 rows)
~~~

`UPDATE` 문과 마찬가지로 테이블에 기본 키가있는 경우,`WHERE` 절에서 그것을 사용하여 특정 행을 확실하게 삭제할 수 있습니다. 그렇지 않은 경우,`WHERE` 절과 일치하는 행이 삭제됩니다. `WHERE`절이 없으면 테이블의 모든 행이 삭제됩니다.


{% unless site.managed %}
## 그래서 다음은요?

- 모든 [SQL Statements](sql-statements.html)를 알아보아요.
- 쉘에서 또는 커맨드 라인에서 직접 [built-in SQL client](use-the-built-in-sql-client.html)를 사용해봅시다. 
- [Install the client driver](install-client-drivers.html)에서는 원하는 언어로 그리고 [build an app](build-an-app-with-cockroachdb.html)을 해보아요.
- [core CockroachDB features](demo-data-replication.html)에 대해 알아보아요. 예를 들어 automatic replication, rebalancing, and fault tolerance
{% endunless %}
