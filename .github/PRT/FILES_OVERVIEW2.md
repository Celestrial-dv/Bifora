# BIPLEX Total Monitor – 파일 개요 및 연관성

> 위치: `/www/FILES_OVERVIEW.md`  
> 목적: **“index.php 수정 시 어떤 파일들을 같이 봐야 하는지”**를 강제로 인지시키는 문서

## 1. 핵심 SSR 관련 파일 목록

| 경로 | 역할 | 주요 기능 | 비고 |
|------|------|-----------|------|
| `_acc/css/main.css` | 전체 대시보드 스타일 | 공통 스타일 정의 | 신규 컴포넌트 추가 시 수정 |
| `_acc/js/main.js` | 클라이언트 인터랙션 | 통계 새로고침, 탭 전환, AJAX 처리, Merge 폴링 | 신규 기능 추가 시 수정 |
| `actions/start.php` | 액션 통합 처리 | MFDS/CosIng/Merge/InfoEngine 액션 분기 | action 파라미터에 따라 서비스 호출 |
| `actions/export_csv.php` | CSV 내보내기 | MFDS/CosIng/InfoEngine/KCIA 데이터 CSV 변환 | 테이블 컬럼 변경 시 수정 |
| **`core/BaseService.php`** | **공통 헬퍼 + 기본 섹션 렌더링** | **1) 공통 포맷팅 함수<br>2) MFDS/CosIng/Merge/API Logs 섹션 HTML 생성<br>3) DB 쿼리 실행** | ⭐ v2.0: 섹션 렌더링 추가 |
| `core/InfoEngineService.php` | Info Engine 전용 로직 | Info 영역 통계, 샘플 조회 | 독립 운영 (BaseService 미상속) |
| `core/InfoService.php` | Info 데이터 동기화 | total_cosmetic → infodata 동기화 | 독립 운영 |
| `core/InfoFunctionService.php` | Info Function 탭 로직 | Function 관리 탭 데이터 | 독립 운영 |
| `core/InfoEnrichedService.php` | Info Enriched 탭 로직 | AI 매칭 데이터 조회 | 독립 운영 |
| `core/InfoKciaService.php` | KCIA 탭 로직 | KCIA 스크래핑 및 매칭 | 독립 운영 |
| `core/InfoFunctionManageService.php` | Function Manage 탭 로직 | Function 모니터링 | 독립 운영 |
| `core/KnagService.php` | KNAG 동기화 로직 | INCI 한글화 관리 | 독립 운영 |
| `core/SupabaseSyncService.php` | Supabase 동기화 로직 | 벡터 DB 동기화 | 독립 운영 |
| **`biplex/templates/templates.php`** | **재사용 컴포넌트 함수 집합** | **card(), badge(), tableRow(), progressBar() 등** | ⭐ v2.0: 작은 컴포넌트만 남김 |
| **`biplex/index.php`** | **SSR 컨트롤러 + 레이아웃** | **1) 서비스 인스턴스 생성<br>2) render*() 메서드 호출<br>3) HTML 레이아웃 조립** | ⭐ v2.0: DB 쿼리 제거 |
| `biplex/myconfig.php` | 환경 설정 | DB 연결, API 키 등 | 민감정보 포함 |
| `biplex/diagnostic.php` | 시스템 진단 | 서버 환경 + DB 구조 검증 | 운영 전용 |
| `biplex/monitor_api.php` | 내부 API | 통계 조회용 (내부 전용) | 외부 접근 차단 |
| `biplex/merge_api.php` | Merge API | 진행률 조회용 (내부 전용) | 외부 접근 차단 |
| `/api/.htaccess` | API 접근 제어 | 외부 접근 차단 | 보안 핵심 |

---

## 2. 계층별 파일 역할 (v2.0)

```text
┌─────────────────────────────────────────────────┐
│  Client Layer                                   │
│  - main.js: AJAX, 탭 전환, 통계 새로고침        │
│  - main.css: 스타일 정의                        │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  Controller + Layout Layer                      │
│  - index.php: 서비스 호출 + HTML 레이아웃       │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  Service Layer (데이터 + HTML)                  │
│  - BaseService.php: 기본 섹션 렌더링            │
│  - InfoEngineService.php: Info 영역             │
│  - SupabaseSyncService.php: Supabase 영역       │
│  - 기타 도메인별 서비스...                      │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  Template Layer (컴포넌트만)                    │
│  - templates.php: 작은 재사용 컴포넌트          │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  Database Layer                                 │
│  - mfds_cosmetic_ingredients                    │
│  - cosing_ingredients                           │
│  - total_cosmetic                               │
│  - merge_logs                                   │
│  - infodata                                     │
│  - api_fetch_logs                               │
└─────────────────────────────────────────────────┘
```

---

## 3. 파일별 수정 허용 범위 (v2.0)

### 3-1. BaseService.php

**허용:**
- ✅ 신규 render*() 메서드 추가 (섹션 렌더링)
- ✅ 신규 get*() 메서드 추가 (데이터 조회)
- ✅ 공통 헬퍼 함수 추가 (fmt, formatDate, escape 등)

**금지:**
- ❌ 기존 메서드의 시그니처 변경 (반환 타입, 파라미터)
- ❌ 기존 메서드의 로직 대폭 수정 (버그 수정 제외)
- ❌ Info 영역 로직 추가 (→ InfoEngineService에서 처리)

**v2.0 주요 변경:**
- ✅ 추가: renderGlobalStats(), renderMfdsSection(), renderCosingSection(), renderMergeSection()
- ✅ 유지: getGlobalStats(), getMfdsSample(), getCosingSample() 등

### 3-2. templates.php

**허용:**
- ✅ 신규 컴포넌트 함수 추가 (예: alert(), modal())
- ✅ 기존 컴포넌트 함수의 선택적 파라미터 추가

**금지:**
- ❌ 기존 컴포넌트 함수 이름 변경
- ❌ 복잡한 섹션 렌더링 함수 추가 (→ BaseService로 이동)
- ❌ DB 쿼리 실행
- ❌ 비즈니스 로직 추가

**v2.0 주요 변경:**
- ❌ 삭제: renderMfdsSection, renderCosingSection 등 (→ BaseService로 이동)
- ✅ 유지: card(), badge(), tableRow(), progressBar() 등 작은 컴포넌트

### 3-3. index.php

**허용:**
- ✅ 신규 탭 추가 (HTML 구조만)
- ✅ 서비스 메서드 호출 추가
- ✅ 레이아웃 조정 (그리드, 섹션 순서)

**금지:**
- ❌ DB 쿼리 직접 실행
- ❌ 복잡한 비즈니스 로직
- ❌ HTML 생성 로직 (단순 조립만 허용)
- ❌ 인라인 JavaScript 추가 (→ main.js로 분리)

**v2.0 주요 변경:**
- ❌ 삭제: getStatsData(), getTabData() 함수
- ✅ 추가: $service->renderGlobalStats() 등 서비스 메서드 호출

### 3-4. main.js

**허용:**
- ✅ 신규 이벤트 핸들러 추가
- ✅ 신규 AJAX 함수 추가
- ✅ 기존 함수 내부 로직 수정 (버그 수정, 기능 개선)

**금지:**
- ❌ 전역 함수명 변경 (refreshGlobalStats, showTab 등)
- ❌ 기존 이벤트 핸들러 삭제
- ❌ index.php에서 호출하는 함수 시그니처 변경

**v2.0 주요 변경:**
- 변경 없음 (기존 동작 100% 유지)

### 3-5. main.css

**허용:**
- ✅ 신규 클래스 추가
- ✅ 기존 클래스 스타일 수정 (색상, 여백 등)

**금지:**
- ❌ 공통 클래스명 변경 (.stat-card, .tab-button 등)
- ❌ 리셋/베이스 스타일 삭제

---

## 4. 신규 메뉴 추가 시 체크리스트

| 순서 | 계층 | 작업 내용 | 파일 |
|------|------|-----------|------|
| 1 | Service | 신규 도메인 서비스 생성 (예: KnagService.php) | `/core/{Domain}Service.php` |
| 2 | Service | 데이터 조회 메서드 작성 | `getStats()`, `getSample()` |
| 3 | Service | 섹션 렌더링 메서드 작성 | `renderSection()` |
| 4 | Template | 필요한 컴포넌트 확인/추가 | `templates.php` |
| 5 | Controller | index.php에 탭 추가 | HTML 구조만 |
| 6 | Controller | 서비스 인스턴스 생성 및 호출 | `$service->renderSection()` |
| 7 | Client | main.js에 탭 전환 로직 추가 (필요시) | `showTab()` 함수 |
| 8 | Client | main.css에 스타일 추가 (필요시) | 컴포넌트 스타일 |
| 9 | Action | start.php에 액션 분기 추가 (필요시) | `action` 파라미터 처리 |

---

## 5. 파일 간 호출 관계 (v2.0)

```text
index.php
  ├─ require templates.php (컴포넌트 함수)
  ├─ new BaseService($pdo)
  │   ├─ renderGlobalStats()
  │   │   ├─ getGlobalStats() [DB 쿼리]
  │   │   └─ card() [templates.php]
  │   ├─ renderMfdsSection()
  │   │   ├─ getMfdsSample() [DB 쿼리]
  │   │   └─ tableRow() [templates.php]
  │   ├─ renderCosingSection()
  │   │   ├─ getCosingSample() [DB 쿼리]
  │   │   └─ tableRow() [templates.php]
  │   └─ renderMergeSection()
  │       ├─ getMergeProgress() [DB 쿼리]
  │       ├─ getMergeSample() [DB 쿼리]
  │       └─ progressBar() [templates.php]
  │
  ├─ new InfoEngineService($pdo)
  │   └─ (독립 운영)
  │
  └─ new SupabaseSyncService($pdo)
      └─ (독립 운영)

main.js
  ├─ refreshGlobalStats() → AJAX 호출
  ├─ showTab() → 탭 전환
  ├─ submitAction() → 액션 실행
  └─ updateMergeProgress() → 진행바 폴링
```

---

## 6. 주요 변경 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| v1.0 | 2025-11-10 | 초기 파일 개요 정의 |
| v2.0 | 2025-11-20 | BaseService 역할 재정의, templates.php 슬림화, index.php 간소화 |

---
