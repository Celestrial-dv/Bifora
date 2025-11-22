# BIPLEX Total Monitor – 수정/확장 규칙

## 1. 공통 원칙

1. **신규 개발이 아닌 ‘수정 요청’일 경우:**
   - 기존 파일의 레이아웃, 구조, 함수명, 클래스명, HTML/CSS 계층을
     임의로 재구성하지 않는다.
   - 변경은 항상 **라인 단위 최소 수정(minimal diff)**로 수행한다.

2. **코어 로직 vs 확장 영역**
   - `CORE ZONE`은 절대 수정 금지, `EXTENSION HOOK` 또는 `PATCH ZONE` 안에서만 로직을 확장한다.
   - 예시 주석:
     ```php
     // ===== CORE ZONE (수정 금지) =====
     // ... 기존 공통 로직 ...

     // ===== EXTENSION HOOK: 여기에만 신규 로직 추가 가능 =====
     ```

3. **멀티 계층 동시 점검 의무**
   - `biplex/index.php`에 기능/메뉴/테이블을 추가하거나 수정할 때는  
     아래 4개 계층을 **항상 동시에 점검**해야 한다.
     - 서비스 계층(core)
     - 액션 계층(actions)
     - 템플릿 계층(templates)
     - 프런트 계층(index.php, main.css, main.js)

---

## 2. 파일별 수정 규칙

### 2-1. `/core/BaseService.php`

- 역할:
  - **기존 메뉴(MFDS, CosIng, Merge)**의 베이스 서비스
  - 공통 유틸/인터페이스/라우팅용 베이스 레이어
- 중요한 원칙:
  - **새로운 메뉴/기능(새 탭, 새 카드, InfoEngine 류, Supabase 류 등)을 추가할 때,**
    - `BaseService.php` 안에 해당 메뉴의 비즈니스 로직을 추가하면 안 된다.
    - 반드시 `core/{MenuName}Service.php` 같은 **전용 서비스 파일**을 별도로 생성한다.
- 수정 허용:
  - 기존 MFDS/CosIng/Merge 관련 메서드의 **안전한 보완** (로그 추가, 검증 추가 등)
  - 공통 인터페이스/헬퍼 메서드 추가 (다른 서비스 파일에서 사용할 범용 함수)
- 수정 금지:
  - **새 메뉴/기능의 비즈니스 로직을 BaseService에 직접 구현**
  - 기존 MFDS/CosIng/Merge 메서드 이름/시그니처 변경
  - 다른 서비스에서 의존하는 공통 메서드의 반환 형태를 깨는 변경

---

### 2-2. `core/*Service.php` (메뉴별 전용 서비스)

- 예시:
  - `core/InfoEngineService.php`
  - `core/SupabaseSyncService.php`
  - 앞으로 추가될 `core/KciaService.php`, `core/{MenuName}Service.php` 등

- 역할:
  - 각 메뉴/기능 단위(InfoEngine, Supabase, KCIA, 신규 모듈 등)의 **도메인 전용 서비스 레이어**
- 수정/추가 원칙:
  - **새 메뉴/기능을 index.php에 추가하는 경우,**
    - 반드시 해당 메뉴 전용 서비스 파일을 하나 생성한다.
    - 예: KCIA 메뉴 → `core/KciaService.php`
  - 각 전용 서비스 파일 내부에
    - 데이터 조회용 메서드
    - 통계/집계용 메서드
    - 검증/전처리 메서드
    를 작성한다.
- 수정 금지:
  - 다른 메뉴에서 사용하는 도메인 로직을 BaseService로 다시 옮기거나 중복 구현하는 것
  - 전용 서비스 파일 없이 BaseService에만 새 메뉴 로직을 다 몰아 넣는 것

---

### 2-3. `/actions/start.php`

- 역할: MFDS/CosIng/Merge/InfoEngine/{MenuName} 등 배치/액션 통합 실행 진입점
- 수정 허용:
  - `$_GET['action']` 또는 `$_POST['action']` 값을 기준으로
    **새로운 action 분기 추가**
  - 기존 action 분기 내부에서:
    - 해당 메뉴 전용 서비스 파일을 호출하도록 연결
- 수정 금지:
  - 기존 action 이름 변경 (`action=mfds_sync` 등)
  - 전용 서비스 파일을 만들지 않고 BaseService만으로 모든 메뉴를 처리하도록 재구성하는 것

---

### 2-4. `/actions/export_csv.php`

- 역할: MFDS/CosIng/InfoEngine/KCIA/{MenuName} 데이터 CSV 내보내기 (향후 신규 메뉴 포함 가능)
- 수정 허용:
  - 신규 메뉴에 대한 CSV 내보내기가 필요할 경우:
    - **해당 메뉴 전용 서비스의 메서드를 호출하는 export 분기 추가**
  - 신규 export 타입(예: `type=kcia`, `type=newmenu`) 추가
- 수정 금지:
  - 기존 CSV 타입 이름 변경/삭제로 인해 기존 기능을 깨는 것
  - BaseService에 직접 새 메뉴용 export 로직을 넣고, 전용 서비스 파일은 만들지 않는 것

---

### 2-5. `/biplex/templates/templates.php`

- 역할: SSR 템플릿(뷰) 함수 모음
- 수정 허용:
  - 각 메뉴/기능 전용 **렌더 함수 추가**
    - 예: `render_mfds_panel()`, `render_cosing_panel()`, `render_kcia_panel()`
  - 신규 메뉴를 위해 별도 템플릿 함수를 반드시 만든다.
- 수정 금지:
  - 기존 템플릿 함수 이름 변경
  - BaseService에 새 메뉴 로직을 두고 templates.php에서는 그 메뉴를 따로 구분하지 않는 방식

---

### 2-6. `/biplex/index.php`

- 역할: BIPLEX Total Dashboard 메인 SSR 페이지 (뷰 중심)
- 절대 규칙:
  1. **레이아웃/디자인 구조(그리드, 카드 배치, 섹션 순서)를 AI가 임의로 변경하지 않는다.**
     - 사용자가 “레이아웃을 바꿔도 된다”, “UI 구조를 재설계해줘”라고 명시적으로 허용하지 않는 한,
       기존 HTML 구조와 섹션 순서를 그대로 유지한다.
     - 텍스트, 라벨, 버튼 속성, 클래스 추가 정도만 허용된다.
  2. **`index.php` 안에는 새로운 `<script>` 태그나 인라인 JavaScript를 추가하지 않는다.**
     - 모든 JS 코드는 `_acc/js/main.js`에만 작성하거나 수정해야 한다.
     - index.php에서는 JS를 "호출하는 HTML 속성(id, class, data-attribute 등)"까지만 설정한다.

- 수정 허용:
  - 기존 카드/섹션 안에서:
    - 버튼 텍스트 변경
    - 버튼의 target, href, data-* 속성 변경
    - 기존 레이아웃을 깨지 않는 범위의 작은 마크업 보완
  - 새 메뉴/카드/섹션 추가:
    - 기존 레이아웃 규칙(그리드 구조)을 그대로 따르는 블록 추가
    - 추가된 블록에 맞춰 templates.php, main.css, main.js, 서비스/액션 계층을 동시에 보완

- 수정 금지:
  - 전체 카드를 다시 배치하거나, 그리드/컬럼 구조를 갈아엎는 전면 리팩터링
  - `index.php` 내부에 클릭 이벤트, fetch, AJAX, setTimeout 등 JS 로직을 직접 작성하는 것
  - 기존 카드/섹션을 제거하고 다른 디자인으로 통째 교체하는 것

---

### 2-7. `_acc/css/main.css`, `_acc/js/main.js`

- CSS (`_acc/css/main.css`):
  - 허용: 신규 메뉴/위젯용 클래스 추가
  - 금지: 기존 공통 레이아웃/유틸 클래스 이름 변경/삭제

- JS (`_acc/js/main.js`):
  - 역할: **대시보드에서 사용하는 모든 클라이언트 사이드 JavaScript의 단일 진입점**
  - 허용:
    - 신규 메뉴/위젯용 이벤트 핸들러/초기화 코드 추가
    - 기존 함수에 옵션 파라미터/작은 조건 로직 추가
  - 금지:
    - `biplex/index.php`나 다른 PHP 파일에 JavaScript를 새로 작성하도록 유도하는 것
    - 기존 전역 초기화/핵심 핸들러 삭제/이름 변경
    - 여러 JS 파일로 쪼개거나, script를 흩뿌리는 구조로 변경하는 것

---

## 3. 신규 메뉴/기능 추가 시 필수 절차 (중요)

> `biplex/index.php`에 **새로운 메뉴(탭/카드/섹션)**를 추가하는 경우,  
> AI는 반드시 아래 **6개 파일 레벨**을 모두 확인해야 한다.

1. **서비스 계층 – 전용 서비스 php 필수**
   - [ ] `core/{NewMenuName}Service.php` 전용 서비스 파일이 있는지 확인
   - [ ] 없다면 생성하고, 다음 메서드들을 정의:
     - 데이터 조회 메서드 (예: `getKciaStats()`)
     - 통계/집계 메서드 (필요시)
     - 검증/전처리 메서드

2. **베이스 서비스(BaseService)**
   - [ ] BaseService.php에는 새 메뉴의 비즈니스 로직을 추가하지 않는다.
   - [ ] 필요하다면, 전용 서비스 인스턴스를 생성해 넘겨주는 **헬퍼/팩토리 역할** 정도만 추가 가능

3. **액션 계층**
   - [ ] `actions/start.php` 에서 해당 메뉴용 action 분기 추가
     - 예: `action=kcia_sync`, `action=newmenu_refresh` 등
   - [ ] 메뉴에 파일/CSV 내보내기 기능이 있다면, `actions/export_csv.php`에도 타입 분기 추가

4. **템플릿 계층**
   - [ ] `biplex/templates/templates.php` 에 해당 메뉴 전용 템플릿 함수 추가
     - 예: `render_kcia_section($data)`, `render_newmenu_panel($data)`
   - [ ] index.php는 이 템플릿 함수를 호출해서 화면만 그린다.

5. **프런트 계층(뷰 + 스타일 + JS)**
   - [ ] `biplex/index.php` 에 메뉴/카드/섹션 HTML 추가
   - [ ] `_acc/css/main.css` 에 해당 메뉴 전용 스타일 클래스 추가
   - [ ] `_acc/js/main.js` 에 해당 메뉴 전용 이벤트 핸들러/초기화 코드 추가

6. **체크 마무리**
   - [ ] 위 1~5 항목이 모두 충족되지 않으면, 코드 생성/수정을 진행하지 말고
         어떤 파일/로직이 부족한지 먼저 보고하고, 사용자에게 확인/지시를 요청한다.

---

## 4. AI를 위한 요약 규칙

1. 새 메뉴/기능은 **BaseService에 비즈니스 로직을 추가하지 않는다.**
2. 항상 `core/{MenuName}Service.php` 형태의 전용 서비스 파일을 만든다.
3. `index.php`에서 메뉴를 추가하면,
   - 전용 서비스 + actions + templates + css + js를 **반드시 같이** 다룬다.
4. 이 규칙을 만족시키지 못하는 설계/코드 제안은 잘못된 것이다.
