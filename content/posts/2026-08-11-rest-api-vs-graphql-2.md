---
title: "REST API vs GraphQL: 어떤 것을 선택해야 할까?"
date: 2026-08-11T02:11:54.145058+09:00
tags: ["rest-api", "graphql", "api-design", "backend-architecture"]
---
## Overview

REST API와 GraphQL은 둘 다 클라이언트와 서버 간 데이터를 주고받는 표준화된 방식이지만, 요청과 응답을 설계하는 철학이 근본적으로 다르다. <strong class="kw">REST</strong>는 리소스 단위의 고정된 엔드포인트로 단순함과 캐싱 효율을 추구하는 반면, <strong class="kw">GraphQL</strong>은 단일 엔드포인트에서 클라이언트가 필요한 데이터만 정확히 요청하는 유연성을 제공한다. 프로젝트의 데이터 복잡도, 팀 규모, 클라이언트 다양성에 따라 선택 기준이 달라진다.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="40" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">REST API</text><text x="480" y="40" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">GraphQL</text><rect x="40" y="70" width="110" height="36" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="93" text-anchor="middle" font-size="12" style="fill:var(--content)">/users/1</text><rect x="40" y="120" width="110" height="36" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="143" text-anchor="middle" font-size="12" style="fill:var(--content)">/users/1/posts</text><rect x="40" y="170" width="110" height="36" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="193" text-anchor="middle" font-size="12" style="fill:var(--content)">/users/1/friends</text><path d="M150 88 L230 130" style="stroke:var(--compare-a)" stroke-width="1.5" fill="none"/><path d="M150 138 L230 140" style="stroke:var(--compare-a)" stroke-width="1.5" fill="none"/><path d="M150 188 L230 150" style="stroke:var(--compare-a)" stroke-width="1.5" fill="none"/><rect x="230" y="115" width="70" height="40" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="265" y="140" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Client A</text><text x="95" y="230" text-anchor="middle" font-size="11" style="fill:var(--secondary)">고정 응답: 여러 번 호출 필요</text><rect x="340" y="120" width="90" height="40" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="385" y="145" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Client B</text><path d="M430 140 L470 100" style="stroke:var(--compare-b)" stroke-width="1.5" fill="none"/><rect x="470" y="75" width="120" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="96" text-anchor="middle" font-size="12" style="fill:var(--content)">POST /graphql</text><text x="530" y="112" text-anchor="middle" font-size="10" style="fill:var(--secondary)">단일 엔드포인트</text><path d="M340 160 L470 110" style="stroke:var(--compare-b)" stroke-width="1.5" fill="none"/><rect x="340" y="170" width="90" height="40" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="385" y="195" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Client C</text><path d="M430 190 L470 120" style="stroke:var(--compare-b)" stroke-width="1.5" fill="none"/><text x="530" y="150" text-anchor="middle" font-size="11" style="fill:var(--secondary)">필요한 필드만 선택 요청</text><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="95" y="270" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Over/Under-fetching 발생 가능</text><text x="530" y="250" text-anchor="middle" font-size="11" style="fill:var(--secondary)">쿼리 하나로 정확히 필요한 데이터만 획득</text></svg>
</div>

## Comparison Table

| Aspect | REST API | GraphQL |
| --- | --- | --- |
| 요청 구조 | 리소스별 다수의 고정 엔드포인트(URL) | 단일 엔드포인트에 쿼리를 보내는 구조 |
| 데이터 조회 방식 | 서버가 정의한 응답 형태를 그대로 수신 | 클라이언트가 필요한 필드만 선택적으로 요청 |
| 연관 데이터 처리 | 여러 번의 왕복 호출(N+1) 또는 중첩 엔드포인트 필요 | 한 번의 쿼리로 중첩된 연관 데이터까지 조회 가능 |
| 캐싱 | HTTP 캐시(브라우저, CDN)를 그대로 활용 가능 | 쿼리마다 응답이 달라 별도의 캐싱 계층 설계 필요 |
| 버전 관리 | /v1, /v2 등 엔드포인트 버전 분리로 대응 | 스키마에 필드를 추가/폐기하며 점진적으로 진화 |
| 학습 곡선 및 도입 비용 | HTTP 메서드와 리소스 개념만 알면 바로 사용 가능 | 스키마, 리졸버, 쿼리 언어 학습이 선행되어야 함 |
| 에러 및 상태 처리 | HTTP 상태 코드(404, 500 등)로 명확히 구분 | 대부분 200 OK로 응답하며 errors 필드로 별도 표현 |
| 적합한 팀/규모 | 단순한 CRUD, 소규모 팀, 공개 API에 적합 | 다양한 클라이언트(웹/모바일)와 복잡한 데이터 그래프에 적합 |

## Key Differences

- REST는 <strong class="kw">리소스 중심</strong> 엔드포인트를, GraphQL은 <strong class="kw">쿼리 중심</strong> 단일 엔드포인트를 사용한다.
- REST는 <strong class="kw">HTTP 캐싱</strong>을 그대로 활용할 수 있지만, GraphQL은 <strong class="kw">커스텀 캐싱</strong> 로직이 필요하다.
- 연관 데이터 조회 시 REST는 여러 번의 호출이 필요할 수 있는 반면 GraphQL은 <strong class="kw">단일 쿼리</strong>로 해결한다.
- GraphQL은 <strong class="kw">오버페칭/언더페칭</strong> 문제를 줄이지만 서버 측 <strong class="kw">스키마 설계</strong> 부담이 커진다.

## When to Use Each

**REST API**

- **단순한 CRUD API**: 리소스 구조가 단순하고 엔드포인트 수가 적을 때 REST의 직관적인 구조가 유리하다.
- **공개 API 및 외부 개발자 대상**: HTTP 표준과 캐싱을 그대로 활용할 수 있어 문서화와 온보딩이 쉽다.
- **CDN/브라우저 캐싱이 중요한 서비스**: URL 기반 캐싱을 그대로 활용해 인프라 비용을 절감할 수 있다.

**GraphQL**

- **다양한 클라이언트(웹/모바일) 지원**: 각 클라이언트가 필요한 필드만 선택적으로 요청해 데이터 전송량을 최적화할 수 있다.
- **복잡하게 연관된 데이터 그래프**: 여러 리소스를 한 번의 쿼리로 조회해 프론트엔드의 호출 횟수를 줄일 수 있다.
- **빠르게 변화하는 프론트엔드 요구사항**: 스키마에 필드를 점진적으로 추가/제거하며 API 버저닝 부담을 줄일 수 있다.
