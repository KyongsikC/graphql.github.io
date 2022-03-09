---
title: Introduction to GraphQL
sidebarTitle: Introduction
layout: docs
category: Learn
permalink: /learn/
next: /learn/queries/
---

GraphQL은 API용 쿼리 언어이며 데이터에 대해 정의한 type system을 사용하여 쿼리를 실행하기 위한 server-side runtime입니다. GraphQL은 특정 데이터베이스나 스토리지 엔진에 연결되어 있지 않으며 대신 기존 코드와 데이터로 뒷받침됩니다.

GraphQL 서비스는 types과 fields를 정의한 다음 각 유형의 각 field에 대한 functions을 제공하여 생성됩니다.

```graphql
type Query {
  me: User
}

type User {
  id: ID
  name: String
}
```

Along with functions for each field on each type:

```js
function Query_me(request) {
  return request.auth.user
}

function User_name(user) {
  return user.getName()
}
```

For example, the query:

```graphql
{
  me {
    name
  }
}
```

Could produce the following JSON result:

```json
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```

To learn more, click **Continue Reading**.
