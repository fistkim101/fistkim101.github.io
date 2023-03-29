---
layout: default
title: CH 19 Data Layer - Caching (Local Data Source)
parent: Flutter Advanced Course - Clean Architecture With MVVM
nav_order: 19
---

<br>

홈 화면 진입시 네트워트 통신하지 않고 기존에 저장해둔 로컬 데이터로 보여주는 방식이었다. 특별히 캐시 관련 라이브러리를 쓴 것도 아니고 그냥 map 만들고 만료시각 넣어서 캐시로 처리한 것.

캐시관련 처리는 특별한게 없었고 dataSource 도 localDataSource, remoteDataSource 로 나눠서 처리한 점을 설계 관점에서 인지할 필요는 있다. 