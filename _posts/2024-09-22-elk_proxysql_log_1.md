---
title: "[ELK] 로그 수집시스템 - proxySQL 수집"
excerpt: "ELK 스택을 이용하여 proxySQL의 로그를 수집하고 검색하는 시스템을 구성합니다."

categories:
  - Elk
tags:
  - [elk]
#permalink: mysql-first
toc: true
toc_sticky: true
 
date: 2024-09-22
last_modified_at: 2024-09-22
comments: true
---

### 🚀 MongoDB Lock Free 알고리즘(내부 잠금경합 최소화)
Real MongoDB 서적의 이론을 토대로 MongoDB의 동시성을 높이기 위한 대표적인 개념인 하자드 포인터와 스킵 리스트에 대해 정리해보았습니다. 
<br/>

### 🚀 하자드 포인터
---
이빅션 쓰레드는 한정된 공유캐시의 공간을 확보하기 위해 적절히 제거할 수 있는 페이지를 디스크 영역으로 동기화 시키는 작업을 수행하는데 이 때 하자드 포인터를 참조하여 공유 캐시에 제거 가능한지 여부를 확인합니다. 

사용자 쓰레드는 사용자의 쿼리를 처리하기 위해 WiredTiger 의 공유캐시를 참조할 때 먼저 하자드 포인터에 자신이 참조하는 페이지를 등록합니다. 그리고 사용자 쓰레드가 쿼리를 처리하는 동안 이빅션 쓰레드는 동시에 캐시에서 제거해야 할 데이터 페이지를 골라 캐시에서 삭제하는 작업을 실행합니다. 이때 "이빅션 쓰레드"는 적절히 제거할 수 있는 페이지(자주 사용되지 않는 페이지)를 골라 먼저 하자드 포인터에 등록돼 있는지 확인합니다.

WiredTiger 스토리지 엔진에서 사용할 수 있는 하자드 포인터의 최대 개수는 기본적으로 1,000개로 제한돼 있습니다. 만약 하자드 포인터의 개수가 부족해 WiredTiger 스토리지 엔진의 처리량이 느려진다면 WiredTiger 스토리지 엔진의 옵션을 변경하여 하자드 포인터의 최대 개수를 1,000개 이상으로 설정할 수 있습니다.

MongoDB 서버의 설정 파일을 이용해 하자드 포인터의 개수를 변경하려면, 다음과 같이 `configString` 옵션에 `hazard_max` 옵션을 설정합니다.



{% assign posts = site.categories.Elk %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}