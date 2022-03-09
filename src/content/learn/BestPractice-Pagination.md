---
title: Pagination
layout: docs
category: Best Practices
permalink: /learn/pagination/
---

## Plurals

가장 간단한 방법은 복수형을 반환하는 필드를 사용하는 것

```graphql
# { "graphiql": true }
{
  hero {
    name
    friends {
      name
    }
  }
}
```

## Slicing

가져올 친구 수를 지정

```graphql
{
  hero {
    name
    friends(first: 2) {
      name
    }
  }
}
```

## Pagination and Edges

페이지 매김을 할 수 있는 방법

- `friends(first:2 offset:2)` 목록에서 다음 두 개를 요청
- `friends(first:2 after:$friendId)`, 가져온 마지막 친구 다음에 다음 두 개를 요청
- `friends(first:2 after:$friendCursor)`, 마지막 항목에서 커서를 가져와 페이지 매김에 사용(**cursor-based pagination**)

```graphql
{
  hero {
    name
    friends(first: 2) {
      edges {
        node {
          name
        }
        cursor
      }
    }
  }
}
```

## End-of-list, counts, and Connections

연결 끝에 도달했을 때를 어떻게 알 수 있습니까?

```graphql
{
  hero {
    name
    friends(first: 2) {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

이 PageInfo 개체에 endCursor 및 startCursor도 포함할 수 있습니다. 이는 연결에 대한 잠재적인 사용성 개선으로 이어집니다.

## Complete Connection Model

분명히 이것은 복수형을 갖는 원래 디자인보다 더 복잡합니다! 그러나 이 디자인을 채택하여 클라이언트를 위한 여러 기능을 풀 수 있습니다.

- 목록을 페이지 매김하는 기능.
- `totalCount` 또는 `pageInfo`와 같은 연결 자체에 대한 정보를 요청할 수 있는 기능.
- `cursor` 또는 `friendshipTime`과 같은 에지 자체에 대한 정보를 요청할 수 있는 기능.
- 사용자가 opaque cursors를 사용하기 때문에 백엔드에서 페이지 매김을 수행하는 방법을 변경할 수 있습니다.

```graphql
# { "graphiql": true }
{
  hero {
    name
    friendsConnection(first: 2, after: "Y3Vyc29yMQ==") {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

## Connection Specification

이 패턴의 일관된 구현을 보장하기 위해 Relay 프로젝트에는 커서 기반 연결 패턴을 사용하는 GraphQL API를 빌드하기 위해 따를 수 있는 공식 [specification](https://facebook.github.io/relay/graphql/connections.htm)이 있습니다.
