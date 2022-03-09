---
title: Thinking in Graphs
layout: docs
category: Best Practices
permalink: /learn/thinking-in-graphs/
next: /learn/serving-over-http/
---

## It's Graphs All the Way Down [\*](https://en.wikipedia.org/wiki/Turtles_all_the_way_down)

> GraphQL을 사용하면 비즈니스 도메인을 그래프로 모델링할 수 있습니다.

스키마 내에서 다양한 유형의 노드와 노드가 서로 연결/관련되는 방식을 정의합니다. 클라이언트에서 이것은 객체 지향 프로그래밍과 유사한 패턴을 생성합니다. 다른 유형을 참조하는 유형입니다. 서버에서 GraphQL은 인터페이스만 정의하므로 모든 백엔드(신규 또는 기존 !)에서 자유롭게 사용할 수 있습니다.

## Business Logic Layer

> 비즈니스 논리 계층은 비즈니스 도메인 규칙을 적용하기 위한 단일 정보 소스 역할을 해야 합니다.

실제 비즈니스 로직을 어디에서 정의해야 합니까? 어디에서 유효성 검사 및 권한 부여 확인을 수행해야 합니까?  
답: 전용 비즈니스 로직 계층 내부.  
비즈니스 논리 계층은 비즈니스 도메인 규칙을 적용하기 위한 단일 정보 소스 역할을 해야 합니다.

다이어그램에서 시스템의 모든 진입점(REST, GraphQL 및 RPC)은 동일한 유효성 검사, 권한 부여 및 오류 처리 규칙으로 처리됩니다.

![Business Logic Layer Diagram](/img/diagrams/business_layer.png)
