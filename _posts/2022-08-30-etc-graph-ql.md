---
title:  "GraphQL 간략 정리"
excerpt: "GraphQL 간략 정리"

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2022-08-30
last_modified_at: 2022-08-30

published: true
---


# intro
## 우선 예시부터 보자
* GraphQL을 파악하려면 샘플 쿼리와 그 응답을 살펴보는 것이 가장 좋습니다. GraphQL 프로젝트 웹사이트인 https://graphql.org/ 에서 수정한 다음 3개의 예시를 살펴보겠습니다.
### 첫 번째 예시
* 지정한 모양의 특정 필드를 반환하는 API를 요청해 클라이언트가 GraphQL 쿼리를 구성하는 방법을 보여줍니다.
```
{
  me {
    name
  }
}
```
* GraphQL API는 다음과 같은 결과를 JSON 형식으로 반환합니다.

```
{
  "me": {
    "name": "Dorothy"
  }
}
```

### 두 번째 예시

* 클라이언트는 GraphQL 쿼리의 일부로 인수를 전달할 수 있습니다. 

```
{
  human(id: "1000") {
    name
    location
  }
}
```

* 결과는 다음과 같습니다.

```
{
  "data": {
    "human": {
      "name": "Dorothy,
      "location": "Kansas"
    }
  }
}
```

### 세 번째 예시
* 여기서부터는 더욱 흥미롭습니다. GraphQL은 사용자에게 재사용 가능한 파편을 정의하고 변수를 할당할 수 있도록 합니다.
* ID 목록을 요청해야 하며, 각 ID에 대한 일련의 기록을 요청해야 한다고 가정해 보겠습니다. GraphQL을 사용하면 단일 API 호출로 원하는 모든 요소를 풀링하는 쿼리를 구성할 수 있습니다.
* 다음과 같은 쿼리가 있다고 가정해 보겠습니다.
```
query HeroComparison($first: Int = 3) {
  leftComparison: hero(location: KANSAS) {
    ...comparisonFields
  }
  rightComparison: hero(location: OZ) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

* 이는 다음과 같은 결과로 이어질 수 있습니다.

```
{
  "data": {
    "leftComparison": {
      "name": "Dorothy",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Aunt Em"
            }
          },
          {
            "node": {
              "name": "Uncle Henry"
            }
          },
          {
            "node": {
              "name": "Toto"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "Wizard",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Scarecrow"
            }
          },
          {
            "node": {
              "name": "Tin Man"
            }
          },
          {
            "node": {
              "name": "Lion"
            }
          }
        ]
      }
    }
  }
}
```

# GraphQL 이란?
* GraphQL은 페이스북에서 만든 쿼리 언어입니다
* Graph QL 은 Structed Query Language(이하 sql)와 마찬가지로 쿼리 언어
* 일반적으로 GraphQL 의 인터페이스간 송수신은 네트워크 레이어 L7의 HTTP POST 메서드와 웹소켓 프로토콜을 활용합니다.
    * 필요에 따라서는 얼마든지 L4의 TCP/UDP를 활용하거나 심지어 L2 형식의 이더넷 프레임을 활용 할 수도 있음.

# REST API와 비교
* REST API는 URL, METHOD등을 조합하기 때문에 다양한 Endpoint가 존재 합니다. 반면, gql은 단 하나의 Endpoint가 존재
* GraphQL API에서는 불러오는 데이터의 종류를 쿼리 조합을 통해서 결정
    * 예를 들면, REST API에서는 각 Endpoint마다 데이터베이스 SQL 쿼리가 달라지는 반면, GraphQL API는 gql 스키마의 타입마다 데이터베이스 SQL 쿼리가 달라짐

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-08-30-etc-graph-ql-01.png)


* REST에서는 Resource에 대한 형태 정의와 데이터 요청 방법이 연결되어 있지만, GraphQL에서는 Resource에 대한 형태 정의와 데이터 요청이 완전히 분리되어 있습니다
* REST에서는 Resource의 크기와 형태를 서버에서 결정하지만, GraphQL에서는 Resource에 대한 정보만 정의하고, 필요한 크기와 형태는 client단에서 요청 시 결정합니다.
* REST에서는 URI가 Resource를 나타내고 Method가 작업의 유형을 나타내지만, GraphQL에서는 GraphQL Schema가 Resource를 나타내고 Query, Mutation 타입이 작업의 유형을 나타냅니다.
* REST에서는 여러 Resource에 접근하고자 할 때 여러 번의 요청이 필요하지만, GraphQL에서는 한번의 요청에서 여러 Resource에 접근할 수 있습니다.
* REST에서 각 요청은 해당 엔드포인트에 정의된 핸들링 함수를 호출하여 작업을 처리하지만, GraphQL에서는 요청 받은 각 필드에 대한 resolver를 호출하여 작업을 처리합니다.

## Resources
* REST의 핵심 아이디어는 리소스입니다. GET 각 리소스는 URL로 식별되며 해당 URL에 요청을 보내 해당 리소스를 검색합니다  . 요즘 대부분의 API가 사용하는 JSON 응답을 받게 될 것입니다. 따라서 다음과 같이 보입니다.

* GET /books/1 
```json
{
  "title": "Black Hole Blues",
  "author": { 
    "firstName": "Janna",
    "lastName": "Levin"
  }
}
```

* GraphQL은 이 두 개념이 완전히 별개이기 때문에 GraphQL은 이와 관련하여 상당히 다릅니다. 스키마에는 다음  Book 과  Author 같은 유형이 있을 수 있습니다.

```
type Book {
  id: ID
  title: String
  published: Date
  price: String
  author: Author
}

type Author {
  id: ID
  firstName: String
  lastName: String
  books: [Book]
}
```
* 특정 책이나 저자에 실제로 액세스할 수 있으려면  Query 스키마에 유형을 생성해야 합니다.

```
type Query {
  book(id: ID!): Book
  author(id: ID!): Author
}
```

* 이제 위의 REST 요청과 유사한 요청을 보낼 수 있지만 이번에는 GraphQL을 사용합니다.
```
// GET /graphql?query={ book(id: "1") { title, author { firstName } } }

{
  "title": "Black Hole Blues",
  "author": {
    "firstName": "Janna",
  }
}
```

* 둘 다 URL을 통해 요청할 수 있고 둘 다 동일한 모양의 JSON 응답을 반환할 수 있지만 REST와 상당히 다른 GraphQL에 대한 몇 가지 사항을 즉시 확인할 수 있습니다.
* 우선 GraphQL 쿼리가 있는 URL이 요청하는 리소스와 관심 있는 필드를 지정한다는 것을 알 수 있습니다. 또한 관련  author 리소스가 포함되어야 한다고 서버 작성자가 결정하는 것이 아니라 API의 소비자가 결정합니다.
* 그러나 가장 중요한 것은 리소스의 ID, Books 및 Authors의 개념이 가져오는 방식과 연결되어 있지 않다는 것입니다. 다양한 유형의 쿼리와 다양한 필드 집합을 통해 동일한 책을 잠재적으로 검색할 수 있습니다.

## URL Routes vs GraphQL Schema
* 오늘날의 REST API에서 API는 일반적으로 엔드포인트 목록으로 설명됩니다.

```
GET /books/:id
GET /authors/:id
GET /books/:id/comments
POST /books/:id/comments
```


* GraphQL에서는 위에서 다룬 것처럼 API에서 사용 가능한 항목을 식별하기 위해 URL을 사용하지 않습니다. 대신 GraphQL 스키마를 사용합니다.

```
type Query {
  book(id: ID!): Book
  author(id: ID!): Author
}

type Mutation {
  addComment(input: AddCommentInput): Comment
}

type Book { ... }
type Author { ... }
type Comment { ... }
input AddCommentInput { ... }
```


* 유사한 데이터 세트에 대한 REST 경로와 비교할 때 여기에 몇 가지 흥미로운 부분이 있습니다. 
    * 첫째, GraphQL은 읽기와 쓰기를 구별하기 위해 동일한 URL에 다른 HTTP 동사를 보내는 대신 다른  초기 유형  인 Mutation vs. Query를 사용합니다. 
    * GraphQL 문서에서 키워드로 보낼 작업 유형을 선택할 수 있습니다.

```
query { ... }
mutation { ... }
```



## 참고
* 이 자료 괜찮은듯
    * https://www.apollographql.com/blog/graphql/basics/graphql-vs-rest/
* 대략 위의 내용을 이해하고 좀더 자세한 자료는 아래를 참고하자. 
    * https://www.redhat.com/ko/topics/api/what-is-graphql
    * https://tech.kakao.com/2019/08/01/graphql-basic/
    * https://hwasurr.io/api/rest-graphql-differences/
    * https://cloud.google.com/blog/ko/products/api-management/interacting-with-apis-rest-and-graphql