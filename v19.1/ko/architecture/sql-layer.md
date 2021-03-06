---
title: SQL 계층
summary: 카크로치디비 아키텍처의 SQL 계층은 SQL API를 개발자에게 드러내고 SQL 문을 키-밸류 연산으로 변환합니다.
toc: true
---

카크로치디비 아키텍처의 SQL 계층은 SQL API를 개발자에게 드러내고 SQL 문을 데이터베이스의 나머지 부분에서 사용할 수 있는 키-밸류 연산으로 변환합니다.

{{site.data.alerts.callout_info}}
[아키텍처 개요](overview.html)를 먼저 읽어보는 것을 권장합니다.
{{site.data.alerts.end}}

## 개요

카크로치디비가 배포되면, 개발자는 클러스터에 연결하고 SQL 문법을 사용하는 것만으로 작업을 시작할 수 있습니다.

카크로치디비의 노드는 모두 동일하게 동작하기 때문에, 아무 노드에 요청을 해도 됩니다(로드밸런서와 잘 동작함을 의미). 요청을 수신하는 노드는 다른 계층이 요청을 처리할 때 "게이트웨이 노드"로 작동합니다.

개발자가 클러스터에 요청을 보낼 때, SQL 문을 사용하지만, 궁극적으로 데이터는 스토리지 계층에 키-밸류(KV) 쌍으로 기록됩니다. 이를 처리하기 위해, SQL 계층은 SQL 문을 KV 작업으로 변환해 트랜잭션 계층에 전달합니다.

### 다른 계층과의 상호작용

카크로치디비에서 SQL 계층의 다른 계층과의 상호작용은 다음과 같습니다:

- 트랜잭션 계층에 요청을 보냅니다..

## 구성요소

### 관계형 구조

개발자는 카크로치디비에 저장된 데이터를 관계형 구조로 경험합니다. 즉 행과 열입니다. 행과 열의 집합은 테이블로 구성됩니다. 테이블의 모음은 데이터베이스를 만듭니다. 클러스터는 많은 데이터베이스를 포함할 수 있습니다.

이 구조로 인해 카크로치디비는 제약조건(예: 외래 키)과 같은 일반적인 관계형 기능을 제공합니다. 이를 통해 애플리케이션 개발자는 데이터베이스가 애플리케이션 데이터의 일관된 구조를 보장할 것이라 신뢰합니다. 데이터 유효성 검사를 애플리케이션 로직에 별도로 구축할 필요가 없습니다.

### SQL API

카크로치디비는 ANSI SQL 표준의 상당 부분을 구현하여 관계형 구조를 나타냅니다. [여기에서 카크로치디비 지원하는 모든 SQL 기능](../sql-feature-support.html)을 확인하실 수 있습니다.

또, SQL API를 통해 개발자는 SQL 데이터베이스의 ACID 트랜잭션(`BEGIN`, `END`, `COMMIT`, 등)을 사용할 수 있습니다.

### PostgreSQL 와이어 프로토콜

SQL 쿼리는 PostgreSQL 와이어 프로토콜을 통해 클러스터에 전달됩니다. 따라서 대부분의 PostgreSQL 호환 드라이버는 물론 GORM (Go) 및 Hibernate (Java)와 같은 PostgreSQL ORM을 사용하여 애플리케이션을 클러스터에 간단히 연결할 수 있습니다.

### SQL 구문분석, 계획, 실행

노드가 클라이언트로부터 SQL 요청을 받으면, 카크로치디브는 명령문을 구문분석하고, [쿼리 계획을 만들고](../cost-based-optimizer.html), 계획을 실행합니다.

#### 구문분석

수신된 쿼리는 yacc파일(지원되는 문법을 설명)에 의해 분석되며, 각 쿼리의 문자열 버전을 [추상 구문 트리](https://en.wikipedia.org/wiki/Abstract_syntax_tree) (AST)로 변환합니다.

#### 논리적 계획

그 후 AST는 세 단계로 쿼리 계획으로 변경됩니다:

1. AST는 고수준의 쿼리 계획으로 변경됩니다. 이 변환 과정에서 카크로치디비는 질의가 유효한지 검사하고, 이름을 확인하며, 불필요한 중간 계산을 제거하고, 중간 결과에 사용할 데이터 타입을 결정하는 [의미론적 분석](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))을 수행합니다.

2. 논리적 계획은 항상 유요한 변환 최적화를 사용하여 *단순화*됩니다.

3. 논리적 계획은 쿼리를 실행하는 여러 가지 방법을 평가하고 최소 비용으로 쿼리 계획을 선택하는 [검색 알고리즘](../cost-based-optimizer.html)을 사용해 *최적화*됩니다.

최적화 단계의 결과는 최적화된 논리적 계획입니다. 이것은 [`EXPLAIN`](../explain.html)으로 확인할 수 있습니다.

#### 물리적 계획

물리적 계획 단계에서는 레인지 로컬리티 정보를 기반으로 어떤 노드가 참여할지 결정합니다. 여기서 카크로치디비는 데이터가 저장소 근처에서 계산을 수행하기 위해 쿼리를 배포합니다.

물리적 계획의 결과는 물리적 계획이며 [`EXPLAIN(DISTSQL)`](../explain.html)로 확인할 수 있습니다.

#### 쿼리 실행

물리적 계획의 구성요소는 실행을 위해 하나 이상의 노드로 전송됩니다. 각 노드에서, 카크로치디비는 *논리적 프로세서*를 생성하여 쿼리의 일부를 계산합니다. 논리적 프로세서는 내부 또는 여러 노드 사이에서 데이터의 *논리적 흐름*을 서로 공유합니다. 합쳐진 쿼리 결과가 쿼리를 수신한 첫 번째 노드로 다시 보내지면, SQL 클라이언트로 전송됩니다.

각 프로세서는 쿼리에 의해 조작된 스캌라 값에 대해 인코딩된 형식을 사용합니다. 이것은 SQL에서 사용된 것과 다른 이진 형식입니다. 따라서 SQL 쿼리에 나열된 값은 인코딩되어, 논리 프로세서간에 통신되고, 디스크로부터 읽어 내어야 하며, SQL 클라이언트에 보내기 전에 다시 디코드해야 합니다.

### 인코딩

SQL 쿼리는 구문분석 가능한 문자열로 작성되지만, 카크로치디비의 하위 계층은 주로 바이트를 사용합니다. 즉, SQL 계층에서 쿼리를 실행할 때, 카크로치디비는 행 데이터를 SQL을 문자열에서 바이트로 변환해야 하며, 하위 계층에서 반환된 바이트를 클라이언트로 다시 전달할 SQL 데이터로 변환해야 합니다.

또한--인덱싱 된 열의 경우--이 바이트 인코딩과 데이터 형이 동일한 정렬 순서를 유지하는 것이 중요합니다. 이것은 카크로치디비가 궁극적으로 정렬된 정렬된 키-밸류 저장하는 방식 때문입니다; 바이트들을 정렬하여 저장하면 효율적으로 KV 데이터를 스캔할 수 있습니다.

그러나, 인덱스되지 않은 열(예: `PRIMARY KEY`가 아닌 열)은 공간을 적제 소비하고 순서를 유지하지 않는("값 인코딩"이라고 함) 인코딩을 사용합니다.

[인코딩 기술 노트](https://github.com/cockroachdb/cockroach/blob/master/docs/tech-notes/encoding.md)에서 보다 자세한 내용을 확인할 수 있습니다.

### 분산SQL

카크로치디비는 분산 데이터베이스이므로 일부 쿼리에 대한 분산된 SQL(분산SQL)을 최적화하여, 다수의 레인지에 대한 쿼리의 속도를 획기적으로 향상시킬 수 있습니다. 분산SQL 아키텍처에 대한 설명은 별도 문서에서 자세히 살펴봐야겠지만, 다음의 간단한 설명으로 동작에 대한 통찰을 얻을 수 있습니다.

비 분산 쿼리에서, 관리자 노드는 크 쿼리와 일치하는 모든 행을 받아, 전체 데이터 세트에 대한 계산을 수행합니다.

그러나 분산SQL 쿼리의 경우, 각 노드가 행에 대한 계산을 수행하고 그 결과를 관리자 노드로 전송합니다. 관리자 노드는 각 도드의 결과를 집계해서, 클라이언트에 응답을 보냅니다.

이는 관리자 노드로 가져오는 데이터 양을 놀랍게 줄여주며, 잘 증명된 병렬 컴퓨팅 개념을 활용하여, 궁극적으로 복잡한 쿼리를 완료하는 데 걸리는 시간을 줄입니다. 또, 데이터를 보유한 노드에서 데이터를 처리하여 카크로치디비는 개별 노드의 스토리지보다 큰 행 집합을 처리할 수 있습니다.

분산 방식의 SQL문을 실행하기 위한 몇 가지 개념을 소개합니다:

- **논리적 계획**: 위에 설명한 AST/`planNode` 트리와 마찬가지로, 계산 단계를 통해 추상적인(비분산) 데이터 흐름을 나타냅니다.
- **물리적 계획**: 물리적 계획은 논리적 계획 노드들을 `cockroach`를 실행하는 물리적 장비에 할당하는 것입니다. 논리적 계획 노드들은 클러스터 토폴로지에 따라 복제되고 특화됩니다. 위의 `planNodes`처럼 물리적 계획의 구성요소는 클러스터에서 스케줄되고 실행됩니다.

[분산SQL RFC](https://github.com/cockroachdb/cockroach/blob/master/docs/RFCS/20160421_distributed_sql.md)에서 훨씬 더 자세한 내용을 확인가능합니다.

## 다른 계층과 기술적인 상호작용

### SQL과 트랜잭션 계층

`planNodes`에서 실행된 KV 명령은 트랜잭션 계층으로 보내집니다.

## 무엇을 더 알아볼까요?

카크로치디비가 [트랜잭션 계층](transaction-layer.html)에서 동시 요청을 처리하는 방법을 알아봅시다.