---
title: Schemas and Types
layout: docs
category: Learn
permalink: /learn/schema/
next: /learn/validation/
sublinks: Type System,Type Language,Object Types and Fields,Arguments,The Query and Mutation Types,Scalar Types,Enumeration Types,Lists and Non-Null,Interfaces,Union Types,Input Types
---

GraphQL 서비스는 해당 서비스에서 쿼리할 수 있는 가능한 데이터 집합을 완전히 설명하는 유형 집합을 정의합니다. 그런 다음 쿼리가 들어오면 해당 스키마에 대해 유효성이 검사되고 실행됩니다.

### Type language

GraphQL services can be written in any language.

We'll use the "GraphQL schema language" - it's similar to the query language.

### Object types and fields

The most basic components of a GraphQL schema are object types:

```graphql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

The language is pretty readable, but let's go over it so that we can have a shared vocabulary:

- `Character`는 a *GraphQL Object Type*으로 some fields를 가질 수 있다. 대부분 스키마 타입은 object type일 것이다.
- `name` and `appearsIn`은 `Character` type의 _fields_ 이다. 모든 `Character` type의 부분에서 나타날 수 있다.
- `String` is one of the built-in _scalar_ types
- `String!` 필드는 _non-nullable_ 이다.
- `[Episode!]!`는 `Episode` objects array이다. 또한 *non-nullable*이다(with zero or more items).

### Arguments

Every field on a GraphQL object type can have zero or more arguments.

```graphql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

### The Query and Mutation types

스키마의 대부분의 유형은 object types 이지만 두 가지 special type이 있습니다.

GraphQL 쿼리의 진입점을 정의하기 때문에 특별합니다. Mutation도 Query와 동일하게 정의되고 동작합니다.

```graphql
schema {
  query: Query
  mutation: Mutation
}
```

```graphql
# { "graphiql": true }
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

### Scalar types

those fields don't have any sub-fields - they are the leaves of the query.

In the following query, the `name` and `appearsIn` fields will resolve to scalar types:

```graphql
# { "graphiql": true }
{
  hero {
    name
    appearsIn
  }
}
```

GraphQL comes with a set of default scalar types out of the box:

- `Int`: A signed 32‐bit integer.
- `Float`: A signed double-precision floating-point value.
- `String`: A UTF‐8 character sequence.
- `Boolean`: `true` or `false`.
- `ID`: The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String; however, defining it as an `ID` signifies that it is not intended to be human‐readable.

사용자 지정 스칼라 유형을 지정하는 방법도 있습니다. 그런 다음 해당 유형을 직렬화, 역직렬화 및 검증하는 방법을 정의하는 것은 구현에 달려 있습니다.

```graphql
scalar Date
```

### Enumeration types

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

This means that wherever we use the type `Episode` in our schema, we expect it to be exactly one of `NEWHOPE`, `EMPIRE`, or `JEDI`.

### Lists and Non-Null

```graphql
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

여기에서는 `String` 유형을 사용하고 느낌표 !를 추가하여 *Non-Null*로 표시합니다.

해당 인수로 null 값이 전달되면 GraphQL 서버가 유효성 검사 오류를 반환합니다.

```graphql
# { "graphiql": true, "variables": { "id": null } }
query DroidById($id: ID!) {
  droid(id: $id) {
    name
  }
}
```

```graphql
myField: [String!]
```

This means that the _list itself_ can be null, but it can't have any null members. For example, in JSON:

```js
myField: null // valid
myField: [] // valid
myField: ["a", "b"] // valid
myField: ["a", null, "b"] // error
```

Now, let's say we defined a Non-Null List of Strings:

```graphql
myField: [String]!
```

This means that the list itself cannot be null, but it can contain null values:

```js
myField: null // error
myField: [] // valid
myField: ["a", "b"] // valid
myField: ["a", null, "b"] // valid
```

You can arbitrarily nest any number of Non-Null and List modifiers, according to your needs.

### Interfaces

For example, you could have an interface `Character` that represents any character in the Star Wars trilogy:

```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

```graphql
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

For example, note that the following query produces an error:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}
```

To ask for a field on a specific object type, you need to use an inline fragment:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

### Union types

```graphql
union SearchResult = Human | Droid | Starship
```

```graphql
# { "graphiql": true}
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

```graphql
{
  search(text: "an") {
    __typename
    ... on Character {
      name
    }
    ... on Human {
      height
    }
    ... on Droid {
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

`Starship`의 경우 `Character`가 아니기 때문에 `name`을 빼면 결과에 나오지 않는다.

### Input types

In the GraphQL schema language, input types look exactly the same as regular object types, but with the keyword `input` instead of `type`:

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}
```

Here is how you could use the input object type in a mutation:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI", "review": { "stars": 5, "commentary": "This is a great movie!" } } }
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```
