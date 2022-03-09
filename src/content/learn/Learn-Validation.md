---
title: Validation
layout: docs
category: Learn
permalink: /learn/validation/
next: /learn/execution/
---

GraphQL 쿼리가 유효한지 여부를 미리 결정할 수 있습니다. 이를 통해 서버와 클라이언트는 런타임 검사에 의존하지 않고도 유효하지 않은 쿼리가 생성되었을 때 개발자에게 효과적으로 알릴 수 있습니다.

```graphql
# { "graphiql": true }
{
  hero {
    ...NameAndAppearances
    friends {
      ...NameAndAppearances
      friends {
        ...NameAndAppearances
      }
    }
  }
}

fragment NameAndAppearances on Character {
  name
  appearsIn
}
```

프래그먼트는 자신을 참조하거나 순환을 생성할 수 없습니다. 결과가 제한되지 않을 수 있기 때문입니다:

```graphql
# { "graphiql": true }
{
  hero {
    ...NameAndAppearancesAndFriends
  }
}

fragment NameAndAppearancesAndFriends on Character {
  name
  appearsIn
  friends {
    ...NameAndAppearancesAndFriends
  }
}
```

필드를 쿼리할 때 주어진 유형에 존재하는 필드를 쿼리해야 합니다. 따라서 `Hero`가 `Character`를 반환하면 `Character`에 대한 필드를 쿼리해야 합니다.

```graphql
# { "graphiql": true }
# INVALID: favoriteSpaceship does not exist on Character
{
  hero {
    favoriteSpaceship
  }
}
```

필드에서 반환하려는 데이터를 지정해야 합니다.

```graphql
# { "graphiql": true }
# INVALID: hero is not a scalar, so fields are needed
{
  hero
}
```

필드가 스칼라인 경우 추가 필드를 쿼리하는 것은 의미가 없으며 그렇게 하면 쿼리가 무효화됩니다.

```graphql
# { "graphiql": true }
# INVALID: name is a scalar, so fields are not permitted
{
  hero {
    name {
      firstCharacterOfName
    }
  }
}
```

`Character`를 반환하는 Hero를 쿼리할 때 `Character`가 `Droid`이면 `primaryFunction`을 가져오고 그렇지 않으면 해당 필드를 무시한다는 것을 나타내는 방법이 필요합니다. 이를 위해 앞서 소개한 Fragment을 사용할 수 있습니다.

```graphql
# { "graphiql": true }
# INVALID: primaryFunction does not exist on Character
{
  hero {
    name
    primaryFunction
  }
}
```

```graphql
# { "graphiql": true }
{
  hero {
    name
    ...DroidFields
  }
}

fragment DroidFields on Droid {
  primaryFunction
}
```

인라인 fragment 사용

```graphql
# { "graphiql": true }
{
  hero {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```
