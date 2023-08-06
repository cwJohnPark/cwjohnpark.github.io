---
title: "데이터 중심의 애플리케이션 설계 - 그래프 데이터 모델 "
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
  - graph database
---

다대다 관계는 서로 다른 데이터 모델을 구분하는 중요한 기능이다.
애플리케이션의 대부분이 일대다 관계(트리 구조 데이터)로 구성되었거나, 레코드 간에 관계가 없는 경우 문서 모델이 적합하다.
하지만 데이터에 다대다 관계가 애플리케이션에 일반적 데이터 모델이라면 그래프로 모델링하는 것이 더 자연스럽다.

그래프는 두 가지 종류의 개체로 구성된다.

- 노드 또는 엔티티라고도 하는 버텍스(vertices)
- 관계 또는 호라고도 하는 엣지(edges)

그래프로 모델링할 수 있는 대표적인 예는 다음과 같다.

- 소셜 그래프: 버텍스(vertex)은 사람이고, 엣지(edge)는 어떤 사람들이 서로를 알고 있는지를 나타낸다.
- 웹 그래프: 버텍스는 웹 페이지이고 엣지는 다른 페이지로 연결되는 HTML 링크를 나타낸다.
- 도로 또는 철도 네트워크: 버텍스는 교차점이고 엣지는 교차점 사이의 도로 또는 철도 노선을 나타낸다.

그래프의 또 다른 강력한 용도는 완전히 다른 유형의 객체를 단일 데이터 저장소에 일관성 있게 저장하는 것이다.
예를 들어, Facebook은 다양한 유형의 버텍스와 엣지로 구성된 단일 그래프를 유지한다.

- 버텍스는 사람, 위치, 이벤트, 체크인 및 사용자의 댓글을 나타낸다.
- 엣지는 어떤 사람이 서로 친구인지, 어떤 위치에서 어떤 체크인을 했는지, 누가 어떤 게시물에 댓글을 달았는지, 누가 어떤 이벤트에 참석했는지 등을 나타낸다.

## 속성 그래프

프로퍼티 그래프 모델에서 각 버텍스는 다음으로 구성된다.

- 고유 식별자
- 나가는 에지 세트
- 들어오는 에지 세트
- 속성 모음(키-값 쌍)

각 엣지는 다음으로 구성됩니다:

- 고유 식별자
- 에지가 시작되는 버텍스(꼬리 버텍스)
- 두 정점 사이의 관계를 설명하는 레이블
- 속성 모음(키-값 쌍)

그래프 데이터모델을 관계형 스키마로 표현하면 다음과 같다.

```sql
CREATE TABLE vertices (
vertex_id integerPRIMARYKEY, properties json
);
CREATE TABLE edges (
edge_id integer PRIMARY KEY,
tail_vertex integer REFERENCES vertices (vertex_id), head_vertex integer REFERENCES vertices (vertex_id), label text,
properties json
);
CREATE INDEX edges_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```

1. 모든 버텍스는 다른 버텍스와 연결되는 엣지를 가질 수 있다.
2. 어떤 정점이 주어지면 그 정점의 들어오는 가장자리와 나가는 가장자리를 모두 효율적으로 찾을 수 있으므로 그래프를 앞뒤로, 즉 정점 사슬을 통해 경로를 따라 이동하는 것이 가능하다.
3. 서로 다른 종류의 관계에 서로 다른 레이블을 사용하면 깔끔한 데이터 모델을 유지하면서 여러 종류의 정보를 단일 그래프에 저장할 수 있다.

## Cypher 쿼리 언어

Cypher는 속성 그래프를 위한 선언적 쿼리 언어로, Neo4j 그래프 데이터베이스를 위해 만들어졌다.

다음은 싸이퍼 쿼리 언어의 예제다.

```
CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location      {name:'United States', type:'country'  }),
  (Idaho:Location    {name:'Idaho',         type:'state'    }),
  (Lucy:Person       {name:'Lucy' }),
  (Idaho) -[:WITHIN]->  (USA)  -[:WITHIN]-> (NAmerica),
  (Lucy)  -[:BORN_IN]-> (Idaho)
```

각 버텍스에는 미국(NAmerica) 또는 아이다호(Idaho)와 같은 기호 이름이 주어진다.
쿼리의 다른 부분에서는 화살표 표기법을 사용하여 이러한 이름을 사용하여 정점 사이에 에지를 생성할 수 있다.
`(Idaho) -[:WITHIN]-> (USA)`는 아이다호를 꼬리 노드로, USA를 머리 노드로 하여 WITHIN이라는 레이블이 지정된 엣지를 생성한다.

다음 쿼리는 Cypher에서 미국에서 유럽으로 이주한 모든 사람의 이름을 구하는 방법을 보여준다.
좀 더 정확하게 말하자면, 여기서는 미국 내 위치에 대한 `BORN_IN` 엣지와 유럽 내 위치에 대한 `LIVES_IN` 에지를 가진 모든 정점을 찾고 각 정점의 이름 속성을 반환하고자 한다.

```
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```

이 쿼리는 다음 조건을 모두 충족하는 정점(이 예제에서는 사람)을 찾는다.

1.`person`은 어떤 정점으로 나가는 `BORN_IN` 엣지를 가지고 있다.
이 버텍스에서 나가는 `WITHIN` 에지 체인을 따라가면 결국 이름 속성이 "United States"인 `Location` 유형의 버텍스에 도달할 수 있다.

2. 동일한 사람의 버텍스에는 나가는 `LIVES_IN` 에지가 있다. 이 에지를 따라 나가는 `WITHIN` 에지 체인을 따라가면 결국 이름 프로퍼티가 "Europe"인 `Location` 유형의 정점에 도달한다.

3. 마지막으로 도달한 사람 버텍스 모두에 대해 이름 속성을 반환한다.
