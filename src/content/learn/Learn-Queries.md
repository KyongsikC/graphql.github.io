---
title: Queries and Mutations
layout: docs
category: Learn
permalink: /learn/queries/
next: /learn/schema/
sublinks: Fields,Arguments,Aliases,Fragments,Operation Name,Variables,Directives,Mutations,Inline Fragments
---

On this page, you'll learn in detail about how to query a GraphQL server.

## Fields

```graphql
# { "graphiql": true }
{
  hero {
    name
  }
}
```

필드는 객체를 참조할 수도 있습니다. 이 경우 해당 개체에 대한 필드의 sub-selection을 만들 수 있습니다.

클라이언트가 classic REST architecture에서 필요로 하는 것처럼 여러 번 왕복하는 대신 한 번의 요청으로 많은 관련 데이터를 가져올 수 있습니다.

```graphql
# { "graphiql": true }
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

## Arguments

```graphql
# { "graphiql": true }
{
  human(id: "1000") {
    name
    height
  }
}
```

REST와 같은 시스템에서는 single set of arguments(query parameters and URL segments)만 전달.

GraphQL에서는 모든 필드와 중첩 객체가 고유한 인수 집합을 얻을 수 있으므로 GraphQL을 여러 API 가져오기가 가능합니다.

```graphql
# { "graphiql": true }
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

[Read more about the GraphQL type system here.](/learn/schema)

## Aliases

필드 결과의 이름을 원하는 대로 바꿀 수 있음

별칭을 지정할 수 있으므로 한 번의 요청으로 두 결과를 모두 얻을 수 있음

```graphql
# { "graphiql": true }
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

## Fragments

재사용 가능한 단위

```graphql
# { "graphiql": true }
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

### Using variables inside fragments

선언된 변수에 액세스

```graphql
# { "graphiql": true }
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
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

## Operation name

`query` 키워드와 query name을 모두 생략하는 축약형 구문을 사용했지만 프로덕션 앱에서는 코드를 덜 모호하게 만드는 데 사용하는 것이 유용합니다.

문제가 발생하면(네트워크 로그 또는 GraphQL 서버의 로그에 오류가 표시됨, 디버깅 및 서버 측 로깅에 매우 유용) 이름으로 식별하는 것이 더 쉽습니다.

```graphql
# { "graphiql": true }
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

## Variables

동적 인수를 쿼리 문자열에 직접 전달하는 것은 좋은 생각이 아닙니다.

그러면 클라이언트 측 코드가 런타임에 쿼리 문자열을 동적으로 조작하고 GraphQL 명세로 serialize해야 하기 때문입니다.

대신 GraphQL에는 쿼리에서 동적 값을 전달하는 방법이 있습니다. 이러한 값을 Variables라고 합니다.

When we start working with variables, we need to do three things:

1. 쿼리 값을 `$variableName`으로 바꿉니다.
2. `$variableName` 변수를 선언
3. 일반적으로 JSON형태로 변수 값을 전달 `variableName: value`

Here's what it looks like all together:

```graphql
# { "graphiql": true, "variables": { "episode": "JEDI" } }
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

### Default variables

```graphql
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

## Directives

변수를 사용하여 쿼리의 구조와 모양을 동적으로 변경하는 방법도 필요할 수 있습니다.

Let's construct a query for such a component:

```graphql
# { "graphiql": true, "variables": { "episode": "JEDI", "withFriends": false } }
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

Directives은 fields 또는 Fragments에 첨부될 수 있으며 서버가 원하는 방식으로 쿼리 실행에 영향을 줄 수 있습니다.

GraphQL 서버 구현에서 지원해야 하는 정확히 두 개의 지시문이 포함되어 있습니다.

- `@include(if: Boolean)` Only include this field in the result if the argument is `true`.
- `@skip(if: Boolean)` Skip this field if the argument is `true`.

완전히 새로운 지시문을 정의하여 실험적 기능을 추가할 수도 있습니다.

## Mutations

서버 측 데이터도 수정할 수 있는 방법이 필요합니다.

쿼리를 구현하여 데이터 쓰기할 수도 있지만, Mutations을 통해 명시적으로 보내야 한다는 규칙을 설정하는 것이 유용합니다.쿼리와 마찬가지로 결과 객체 유형을 반환할 수 있습니다.

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI", "review": { "stars": 5, "commentary": "This is a great movie!" } } }
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

### Multiple fields in mutations

쿼리와 변형 사이에는 이름 외에 한 가지 중요한 차이점이 있습니다:

**query fields가 parallel로 실행되는 동안 mutation fields는 차례대로 실행됩니다.**

첫 번째는 두 번째가 시작되기 전에 완료되도록 보장하고 race condition이 발생되지 않도록 합니다.

## Inline Fragments

Interface or a union type을 반환하는 필드를 쿼리하는 경우 *inline fragments*을 사용하여 concrete type의 데이터에 액세스해야 합니다.

`hero` field는 에피소드 인수에 따라 `Human` 또는 `Droid`가 될 수 있는 `Character` 유형을 반환합니다. 직접 선택에서는 이름과 같이 `Character` 인터페이스에 존재하는 필드만 요청할 수 있습니다.

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```

### Meta fields

`__typename`을 요청하여 해당 지점의 개체 유형 이름을 가져올 수 있습니다.

GraphQL 서비스는 몇 가지 메타 필드를 제공하며 나머지는 [Introspection](../introspection/) system을 노출하는 데 사용됩니다.

```graphql
# { "graphiql": true}
{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}
```
