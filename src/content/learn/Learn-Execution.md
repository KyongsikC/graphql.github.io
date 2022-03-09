---
title: Execution
layout: docs
category: Learn
permalink: /learn/execution/
next: /learn/thinking-in-graphs/
---

```graphql
type Query {
  human(id: ID!): Human
}

type Human {
  name: String
  appearsIn: [Episode]
  starships: [Starship]
}

enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

type Starship {
  name: String
}
```

```graphql
# { "graphiql": true }
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

각 유형의 각 필드는 GraphQL 서버 개발자가 제공하는 *resolver*라는 함수로 뒷받침됩니다. 필드가 실행되면 해당 *resolver*가 호출되어 다음 값을 생성합니다.

필드가 문자열이나 숫자와 같은 스칼라 값을 생성하면 실행이 완료됩니다. 그러나 필드가 object value을 생성하는 경우 쿼리에는 해당 개체에 적용되는 다른 필드 선택이 포함됩니다. 이것은 스칼라 값에 도달할 때까지 계속됩니다. GraphQL 쿼리는 항상 스칼라 값에서 끝납니다.

## Root fields & resolvers

GraphQL API에 대한 가능한 모든 entry points을 나타내는 유형이 있으며, 이를 종종 _Root_ type or the _Query_ type이라고 합니다.

이 예에서 쿼리 유형은 `id` 인수를 허용하는 `human`이라는 필드를 제공합니다.

```js
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

리졸버 함수는 4개의 인수를 받습니다.:

- `obj` The previous object, root Query type에선 사용하지 않음.
- `args` GraphQL 쿼리의 필드에 제공된 인수입니다.
- `context` 모든 resolver에 제공되며 현재 로그인한 사용자 또는 데이터베이스에 대한 액세스와 같은 중요한 컨텍스트 정보를 보유하는 값
- `info` 현재 쿼리 및 스키마 세부 정보와 관련된 필드별 정보를 보유하는 값

## Asynchronous resolvers

Let's take a closer look at what's happening in this resolver function.

```js
human(obj, args, context, info) {
  return context.db.loadHumanByID(args.id).then(
    userData => new Human(userData)
  )
}
```

## Trivial resolvers

```js
Human: {
  name(obj, args, context, info) {
    return obj.name
  }
}
```

GraphQL 라이브러리에서 이렇게 간단한 리졸버를 생략할 수 있으며 필드에 리졸버가 제공되지 않으면 동일한 이름의 속성을 읽고 반환합니다.

## Scalar coercion

`name` 필드가 해결되는 동안 `appearsIn` 및 `starships` 필드는 concurrently하게 resolved될 수 있습니다.

```js
Human: {
  appearsIn(obj) {
    return obj.appearsIn // returns [ 4, 5, 6 ]
  }
}
```

내부적으로 4, 5, 6과 같은 숫자를 사용하지만 GraphQL 유형 시스템에서 Enum 값으로 나타내는 Enum이 서버에 정의되어 있을 수 있습니다.

## List resolvers

```js
Human: {
  starships(obj, args, context, info) {
    return obj.starshipIDs.map(
      id => context.db.loadStarshipByID(id).then(
        shipData => new Starship(shipData)
      )
    )
  }
}
```

GraphQL은 이러한 모든 비동기 요청을 동시에 기다리며, 이러한 각 항목에 대한 `name` 필드를 로드하기 위해 동시에 다시 계속 resolved됩니다.

## Producing the result

각 필드가 확인되면 결과 값은 필드 이름(또는 별칭)을 키로, 확인된 값을 값으로 사용하여 키-값 맵에 배치됩니다. 이것은 쿼리의 맨 아래 리프 필드에서 루트 쿼리 유형의 원래 필드까지 계속됩니다.

```graphql
# { "graphiql": true }
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```
