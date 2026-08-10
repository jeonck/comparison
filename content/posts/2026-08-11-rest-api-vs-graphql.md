---
title: "REST API vs GraphQL: 주요 기능 및 특징 비교"
date: 2026-08-11T02:05:43.552272+09:00
tags: ["rest-api", "graphql", "api-design", "web-development"]
---
## Overview

REST와 GraphQL은 클라이언트-서버 간 데이터 통신을 위한 두 가지 대표적인 API 설계 방식입니다. <strong class="kw">REST</strong>는 리소스 중심의 다중 엔드포인트 구조를, <strong class="kw">GraphQL</strong>은 단일 엔드포인트에서 클라이언트가 필요한 데이터를 직접 명시하는 쿼리 구조를 사용합니다.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">REST</text><text x="480" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">GraphQL</text><rect x="40" y="60" width="90" height="260" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="85" y="200" text-anchor="middle" font-size="13" style="fill:var(--content)">Client</text><rect x="200" y="70" width="110" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="95" text-anchor="middle" font-size="11" style="fill:var(--content)">/users</text><rect x="200" y="130" width="110" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="155" text-anchor="middle" font-size="11" style="fill:var(--content)">/posts</text><rect x="200" y="190" width="110" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="215" text-anchor="middle" font-size="11" style="fill:var(--content)">/comments</text><rect x="200" y="250" width="110" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="275" text-anchor="middle" font-size="11" style="fill:var(--content)">/likes</text><line x1="130" y1="90" x2="200" y2="90" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="130" y1="150" x2="200" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="130" y1="210" x2="200" y2="210" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="130" y1="270" x2="200" y2="270" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="335" text-anchor="middle" font-size="11" style="fill:var(--secondary)">여러 엔드포인트, 다중 요청</text><rect x="390" y="60" width="90" height="260" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="435" y="200" text-anchor="middle" font-size="13" style="fill:var(--content)">Client</text><rect x="550" y="150" width="70" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="585" y="175" text-anchor="middle" font-size="10" style="fill:var(--content)">/graphql</text><text x="585" y="192" text-anchor="middle" font-size="9" style="fill:var(--secondary)">단일 엔드포인트</text><line x1="480" y1="180" x2="550" y2="180" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="585" y="335" text-anchor="middle" font-size="11" style="fill:var(--secondary)">단일 요청, 선택적 필드</text></svg>
</div>

## Comparison Table

| 항목 | REST API | GraphQL |
| --- | --- | --- |
| 요청 진입점 | 리소스별 다중 엔드포인트 (/users, /posts 등) | 단일 엔드포인트 (보통 /graphql) |
| 데이터 요청 방식 | URL 경로와 HTTP 메서드로 리소스 지정 | 쿼리 문서로 필요한 필드를 명시적으로 선언 |
| 응답 데이터 범위 | 서버가 정의한 고정 응답 구조 반환 | 클라이언트가 요청한 필드만 반환 |
| 오버/언더패칭 | 발생 가능 (불필요한 필드 포함 또는 추가 요청 필요) | 필드 단위 선택으로 최소화 |
| 캐싱 방식 | HTTP 캐싱(GET, ETag 등) 자연스럽게 지원 | 엔드포인트가 단일해 별도 캐싱 계층 필요 |
| 버전 관리 | URL 버전(/v1, /v2) 또는 헤더로 관리 | 스키마에 필드 추가/폐기(deprecate)로 점진적 진화 |
| 오류 처리 | HTTP 상태 코드(404, 500 등)로 표현 | HTTP 200과 함께 응답 본문의 errors 필드로 표현 |
| 파일 업로드/스트리밍 | 멀티파트 폼, 스트리밍 등 HTTP 표준 방식 활용 | 표준 스펙 밖이라 별도 확장이나 라이브러리 필요 |

## Key Differences

- REST는 <strong class="kw">다중 엔드포인트</strong>, GraphQL은 <strong class="kw">단일 엔드포인트</strong> 구조를 사용한다
- GraphQL은 <strong class="kw">필드 단위 조회</strong>로 오버/언더패칭을 줄이지만 REST는 고정 응답 구조를 반환한다
- REST는 <strong class="kw">HTTP 캐싱</strong>을 자연스럽게 활용하지만 GraphQL은 별도 캐싱 전략이 필요하다
- 오류 처리 시 REST는 <strong class="kw">HTTP 상태 코드</strong>를, GraphQL은 <strong class="kw">응답 본문 errors</strong>를 사용한다

## When to Use Each

**REST API**

- **공개 웹 API 설계**: HTTP 표준과 캐싱, 문서화 도구 생태계가 성숙해 외부 개발자에게 익숙한 인터페이스를 제공한다
- **단순 CRUD 서비스**: 리소스 구조가 단순하고 요청 패턴이 예측 가능할 때 엔드포인트 설계가 직관적이다
- **파일 업로드/스트리밍 중심 서비스**: HTTP의 멀티파트, 스트리밍 기능을 그대로 활용할 수 있어 별도 확장이 필요 없다

**GraphQL**

- **복잡한 화면별 데이터 조합**: 여러 리소스를 조합해야 하는 화면에서 한 번의 요청으로 필요한 필드만 가져올 수 있다
- **모바일 등 대역폭 제약 환경**: 필드 단위 선택으로 응답 크기를 최소화해 네트워크 비용을 줄인다
- **빠르게 변화하는 프론트엔드 요구사항**: 백엔드 배포 없이도 클라이언트가 필요한 필드 조합을 자유롭게 바꿀 수 있다
