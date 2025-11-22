# BIPLEX Total Monitor – SSR 아키텍처
## 1. 프로젝트 개요
- 프로젝트명: **BIPLEX Total Monitor**
- 목표: MFDS, CosIng, Merge, InfoEngine, Supabase, KNAG 데이터를 하나의 **PHP SSR 대시보드**에서 통합 모니터링
- 렌더링 방식: **서버 사이드 렌더링(SSR, PHP)**  
- 클라이언트 JS: 보조적 인터랙션 (약 700줄, `_acc/js/main.js`)
- 주요 SSR 진입점: `biplex/index.php`

---

## 2. 디렉토리 구조 (SSR 관련)

```text
/www
│
├─ _acc/
│   ├─ css/
│   │   └─ main.css                   # SSR 공통 스타일시트
│   └─ js/
│       └─ main.js                    # SSR 공통 JavaScript
│
├─ actions/
│   ├─ start.php                      # 모든 액션 통합 (MFDS/CosIng/Merge/InfoEngine)
│   └─ export_csv.php                 # CSV 내보내기 (MFDS/CosIng/InfoEngine/KCIA)
│
├─ core/
│   ├─ BaseService.php                # 공통 헬퍼 + 기본 섹션 렌더링 (MFDS, CosIng, Merge, API Logs)
│   ├─ InfoService.php                # Info 서비스 - Data sync, info status
│   ├─ InfoEngineService.php          # Info Engine 서비스 - Info Engine Status
│   ├─ InfoEnrichedService.php        # Info Engine 서비스 - Latest Enriched Ingredients
│   ├─ InfoKicaService.php            # Info KCIA 서비스
│   ├─ InfoFunctionManageService.php  # Info Function Manager 서비스
│   ├─ KnagService.php                # KNAG 동기화 서비스
│   └─ SupabaseSyncService.php        # Supabase 동기화 서비스
│
├─ biplex/
│   ├─ templates/
│   │   └─ templates.php              # 재사용 컴포넌트 함수 집합 (card, badge, tableRow 등)
│   │
│   ├─ myconfig.php                   # 환경 설정 파일
│   ├─ monitor_api.php                # 기존 API (내부 전용)
│   ├─ merge_api.php                  # 기존 API (내부 전용)
│   ├─ info_status.php                # 기존 API (내부 전용)
│   ├─ index.php              # 메인 대시보드 (SSR 엔트리)
│   ├─ diagnostic.php                 # BIPLEX Total Monitor 진단(서버환경+DB구조+파일시스템)
│   └─ diagnostic2.php                # BIPLEX Total Monitor 진단 (백업)
│
├─ api/
│   ├─ .htaccess                      # 내부 API 직접 접근 차단
│   └─ info_status.php                # 기존 API (내부 전용)
│
└─ .htaccess                          # 전역 접근 제어
``` :contentReference[oaicite:1]{index=1}

---

## 3. 요청 처리 흐름 (Request → Response)
### 3-1. 일반 대시보드 요청 흐름
---
브라우저 요청: GET /biplex/index.php
      ↓
Apache + 전역 .htaccess
      ↓
PHP: /biplex/index.php
      ↓
(1) 환경설정 로드: myconfig.php
(2) 템플릿 로드: templates.php (컴포넌트 함수만)
(3) 서비스 인스턴스 생성: BaseService($pdo)
(4) 서비스 메서드 호출: 
    - renderGlobalStats() → 전역 통계 HTML
    - renderMfdsSection() → MFDS 섹션 HTML
    - renderCosingSection() → CosIng 섹션 HTML
    - renderMergeSection() → Merge 섹션 HTML
(5) 레이아웃에 HTML 조립
      ↓
응답: 완성된 HTML 반환

※ 핵심 변경사항:
- ❌ 삭제: index.php의 getStatsData(), getTabData() 함수
- ✅ 추가: BaseService의 render*() 메서드들
- ✅ 개선: templates.php는 작은 컴포넌트만 제공
---

### 3-2. AJAX 통계 새로고침 흐름
브라우저: fetch('/biplex/index.php?partial=stats')
      ↓
index.php: partial 파라미터 감지
      ↓
BaseService::renderGlobalStats() 호출
      ↓
응답: Global Stats HTML만 반환

### 3-3. 액션/CSV 요청 흐름
브라우저: POST /actions/start.php (action=mfds_start)
      ↓
start.php: action 파라미터 분기
      ↓
해당 서비스 호출 (BaseService 또는 개별 서비스)
      ↓
JSON 응답

## 4. 인증/세션/보안 처리 위치
⚠ 아래는 실제 구현에 맞춰 지속적으로 갱신 필요
### 4-1.인증/세션 시작 위치:
(예: biplex/index.php 상단 또는 공통 인클루드 파일)

### 4-2.권한 체크 위치:
(예: index.php, actions/start.php, export_csv.php 상단에서 is_logged_in() 확인)

### 4-3.직접 접근 차단:
(1) /api/.htaccess 에서 외부 접근 차단 (내부 PHP만 호출 가능) 
(2) 루트 .htaccess 에서 index 이외 접근 제어

## 5. 레이어 정의
### 5-1.Service Layer (데이터 + HTML 생성)
BaseService.php의 역할:

(1) 공통 헬퍼 함수 (모든 서비스에서 재사용)
- fmt(): 숫자 포맷팅
- formatDate(): 날짜 포맷팅
- escape(): XSS 방지

(2)기본 도메인 섹션 렌더링 (MFDS, CosIng, Merge, API Logs)
- renderGlobalStats(): 전역 통계 카드 HTML
- renderMfdsSection(): MFDS 섹션 HTML
- renderCosingSection(): CosIng 섹션 HTML
- renderMergeSection(): Merge 섹션 HTML
- renderApiFetchLogs(): API Logs 섹션 HTML

(3) 데이터 조회 메서드 (DB 쿼리 실행)
- getGlobalStats(): 전역 통계 배열
- getMfdsSample(): MFDS 샘플 배열
- getCosingSample(): CosIng 샘플 배열
- 등...

(4)개별 도메인 서비스 (InfoEngineService 등):
BaseService를 상속받지 않고 독립 운영
복잡한 도메인 로직 담당
필요 시 BaseService의 헬퍼 함수 호출 가능

### 5-2.Template Layer (재사용 컴포넌트만)
templates.php의 역할:
❌ 삭제: renderMfdsSection, renderCosingSection 등 (→ BaseService로 이동)
✅ 유지: 작은 재사용 컴포넌트만
-card(): 통계 카드
-badge(): 상태 배지
-tableRow(): 테이블 행
-progressBar(): 진행바
-sectionHeader(): 섹션 헤더

### 5-3.Controller + View Layer (index.php)
index.php의 역할:
-세션/인증 처리
-서비스 인스턴스 생성
-서비스 메서드 호출
-레이아웃 조립 (HTML 구조만)

❌ 금지사항:
-DB 쿼리 직접 실행
-복잡한 비즈니스 로직
-HTML 생성 로직

### 5-4.Actions/Jobs Layer
actions/start.php:
-액션 파라미터에 따라 서비스 호출
-JSON 응답 반환

actions/export_csv.php:
-CSV 내보내기 전담
-서비스에서 데이터 조회 후 CSV 변환

### 5-5.Client Layer (main.js)
main.js의 역할:
-자동 통계 새로고침 (30초마다)
-탭 전환 로직
-AJAX 액션 처리 (submitAction)
-Merge 진행바 폴링
-Info Function 탭 관리

- 실제 세션/로그인 처리 위치 (나중에 구현되면 섹션 4에 구체 경로/함수명 추가)
- index 에서 사용하는 주요 “데이터 소스”(MFDS, CosIng, Merge, InfoEngine, Supabase, KNAG)의 개요
- 나중에 라우터를 따로 뺄 계획이라면 “향후 routes.php 도입 시 흐름” 미리 주석으로 메모

## 6. 레이어 정의
┌─────────────────────────────────────────────────┐
│  브라우저 요청 (GET /biplex/index.php)             │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  index.php (Controller + Layout)                │
│  ----------------------------------------       │
│  $service = new BaseService($pdo);              │
│  $globalStats = $service->renderGlobalStats();  │
│  $mfdsSection = $service->renderMfdsSection();  │
│  ...                                            │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  BaseService (Data + HTML)                      │
│  ----------------------------------------       │
│  public function renderMfdsSection() {          │
│      $sample = $this->getMfdsSample(5);         │
│      // templates.php의 컴포넌트 사용              │
│      return "<section>...</section>";           │
│  }                                              │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  templates.php (Components)                     │
│  ----------------------------------------       │
│  function card(...) { return "..."; }           │
│  function tableRow(...) { return "..."; }       │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  완성된 HTML 응답                                 │
└─────────────────────────────────────────────────┘

## 7. 향후 확장 계획
### 7-1.도메인별 서비스 분리 (Phase 2)
현재 BaseService에 MFDS, CosIng, Merge가 혼재되어 있지만, 향후 다음과 같이 분리 가능:
class MfdsService extends BaseService {
    public function renderSection() { ... }
}

class CosingService extends BaseService {
    public function renderSection() { ... }
}

class MergeService extends BaseService {
    public function renderSection() { ... }
}

### 7-2.라우터 도입 (Phase 3)
```text
routes.php → Controller → Service → View
```

## 8. 주요 변경 이력
버전	날짜	      변경 내용
v1.0	2025-11-10    초기 아키텍처 정의
v2.0	2025-11-20    BaseService 역할 재정의, templates.php 슬림화



---
