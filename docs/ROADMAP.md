# Notion 기반 견적서 웹 뷰어 개발 로드맵

Notion 데이터베이스에 입력된 견적서를 고객이 웹 브라우저에서 확인하고 PDF로 저장할 수 있는 단일 목적 뷰어 애플리케이션

## 개요

견적서 웹 뷰어는 비로그인 고객을 위한 견적서 조회 서비스로 다음 기능을 제공합니다:

- **Notion API 견적서 조회**: 고유 URL(Notion 페이지 ID)로 견적서 데이터를 서버사이드에서 조회
- **견적서 웹 렌더링**: 헤더(발주처, 발행일, 유효기간), 항목 테이블(품목/수량/단가/금액), 합계를 보기 좋게 표시
- **PDF 다운로드**: 브라우저 인쇄 API(window.print())를 활용한 PDF 저장
- **에러 처리**: 존재하지 않거나 만료된 견적서에 대한 안내 메시지 표시

## 기술 스택

- Next.js 16 (App Router), React 19, TypeScript 5
- Tailwind CSS v4, shadcn/ui (new-york 스타일)
- @notionhq/client (Notion API 공식 클라이언트)
- Lucide React (아이콘)
- Vercel 배포

## 개발 워크플로우

1. **작업 계획**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 생성**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - `/tasks` 디렉토리에 새 작업 파일 생성
   - 명명 형식: `XXX-description.md` (예: `001-setup.md`)
   - 고수준 명세서, 관련 파일, 수락 기준, 구현 단계 포함
   - API/비즈니스 로직 작업 시 "## 테스트 체크리스트" 섹션 필수 포함 (Playwright MCP 테스트 시나리오 작성)
   - 예시를 위해 `/tasks` 디렉토리의 마지막 완료된 작업 참조. 초기 상태의 샘플로 `000-sample.md` 참조.

3. **작업 구현**

   - 작업 파일의 명세서를 따름
   - 기능과 기능성 구현
   - **API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수**
   - 각 단계 후 작업 파일 내 단계 진행 상황 업데이트
   - 구현 완료 후 Playwright MCP를 사용한 E2E 테스트 실행
   - **⛔ 테스트 미통과 시 다음 단계 진행 절대 금지** — 실패 원인 파악 후 수정
   - 테스트 통과 확인 후 다음 단계로 진행
   - 각 단계 완료 후 중단하고 추가 지시를 기다림

   **🧪 API/비즈니스 로직 구현 후 필수 테스트 절차 (Playwright MCP)**

   ```
   Step 1. 개발 서버 실행 확인 (npm run dev)
   Step 2. Playwright MCP로 해당 기능 페이지 탐색
   Step 3. 정상 케이스 테스트 — 예상 결과와 실제 결과 비교
   Step 4. 경계값/엣지 케이스 테스트 — 빈 값, 최대값, 잘못된 형식 등
   Step 5. 에러 케이스 테스트 — 네트워크 오류, 인증 실패, 권한 없음 등
   Step 6. 테스트 결과를 작업 파일의 "## 테스트 체크리스트"에 기록
   Step 7. 모든 체크 통과 시에만 해당 단계를 완료로 표시
   ```

4. **로드맵 업데이트**

   - 로드맵에서 완료된 작업을 완료로 표시

## 개발 단계

### Phase 1: 프로젝트 골격 - 완료

> **왜 이 순서인가?**
> 디렉토리 구조, 라우팅, 빌드 환경을 먼저 확립해야 이후 모든 작업이 충돌 없이 진행된다.
> 골격 없이 기능을 먼저 만들면 나중에 구조를 뜯어고치는 비용이 급격히 커진다.

- **Task 001: 프로젝트 초기 설정 및 라우팅 구조** - 완료
  - See: 기존 코드베이스에 반영됨
  - Next.js App Router 기반 라우트 구조 생성 (`/`, `/quote/[id]`)
  - 루트 레이아웃 (`layout.tsx`) 및 프로바이더 구성 (NextThemesProvider, TooltipProvider, Toaster)
  - 글로벌 CSS 및 Tailwind CSS v4 설정
  - Geist 폰트 적용

### Phase 2: 공통 모듈 - 완료

> **왜 이 순서인가?**
> 타입 정의와 공통 유틸리티를 먼저 확립해야 핵심 기능 구현 시 타입 불일치나 중복 코드가 생기지 않는다.
> 모든 기능이 의존하는 "계약서"를 먼저 작성하는 것과 같다.

- **Task 002: 타입 정의 및 인터페이스 설계** - 완료
  - See: `src/types/index.ts`, `src/types/ui.ts`
  - Invoice, InvoiceItem, InvoiceStatus 타입 정의
  - NavItem, FeatureCard 등 공통 타입 정의
  - ButtonVariant, ButtonSize UI 타입 및 BREAKPOINTS 상수 정의

### Phase 3: 핵심 기능 - 완료

> **왜 이 순서인가?**
> 이 앱의 존재 이유인 "Notion에서 견적서 조회 → 화면 렌더링 → PDF 저장" 플로우를 먼저 완성해야
> 전체 가치를 빠르게 검증할 수 있다. 데이터(API) → 표시(UI) → 출력(PDF) 순으로
> 의존성 방향을 따라 구현한다.

- **Task 006: Notion API 연동 및 데이터 조회** - 완료
  - See: `src/lib/notion.ts`
  - @notionhq/client 기반 Notion 클라이언트 싱글톤 구성
  - Invoice 페이지 조회 (pages.retrieve) 및 파싱 헬퍼 함수 (title, rich_text, date, select, number)
  - InvoiceItems 데이터베이스 쿼리 (relation 필터링)
  - 환경변수: NOTION_API_KEY, NOTION_DATABASE_ID, NOTION_ITEMS_DATABASE_ID

- **Task 003: 견적서 조회 페이지 UI 구현** - 완료
  - See: `src/app/quote/[id]/page.tsx`
  - 견적서 헤더 영역 (견적번호, 고객명, 상태 Badge, 발행일, 유효기간, 비고)
  - 항목 테이블 (품목명, 수량, 단가, 금액) shadcn/ui Table 컴포넌트 활용
  - 소계/부가세(10%)/합계 금액 표시 영역
  - Card 기반 레이아웃 (max-w-3xl 중앙 정렬)

- **Task 004: PDF 저장 버튼 및 인쇄 스타일시트** - 완료
  - See: `src/app/quote/[id]/_print-button.tsx`, `src/app/globals.css`
  - PrintButton 클라이언트 컴포넌트 (window.print() 호출)
  - @media print CSS 규칙 (불필요 요소 숨김, 여백 최적화, 페이지 나누기 방지)
  - 인쇄 시 Card 테두리 유지 처리

### Phase 4: 추가 기능

> **왜 이 순서인가?**
> 핵심 기능이 동작함을 확인한 후 에러 처리, 보안, 사용자 안내를 추가한다.
> 이 단계는 "동작하는 앱"을 "믿을 수 있는 앱"으로 만드는 과정이다.
> 핵심 기능 없이 에러 처리만 먼저 만드는 것은 방향이 없는 작업이 된다.

- **Task 005: 에러 페이지 및 루트 안내 페이지** - 완료
  - See: `src/app/quote/[id]/not-found.tsx`, `src/app/page.tsx`
  - Not Found 페이지 (존재하지 않는 견적서 안내)
  - 루트 경로 안내 페이지 (견적서 조회 방법 가이드, URL 형식 예시, 문의 연락처)

- **Task 007: 에러 바운더리 및 서버 에러 처리** - 우선순위
  - `/quote/[id]/error.tsx` 에러 바운더리 컴포넌트 생성 (Notion API 장애, 네트워크 오류 등 런타임 에러 대응)
  - `/quote/[id]/loading.tsx` 로딩 UI 컴포넌트 생성 (Skeleton 기반 견적서 로딩 상태 표시)
  - 만료된 견적서(status === 'expired') 별도 처리 로직 추가 (조회 가능하되 만료 안내 배너 표시)
  - Notion API 응답 실패 시 에러 유형별 분기 처리 (API 키 오류, 페이지 접근 권한, 네트워크 타임아웃)
  - **[구현 후 필수]** Playwright MCP로 에러 케이스 시나리오 전수 테스트 (API 오류, 만료 견적서, 잘못된 ID)

- **Task 008: 입력값 검증 및 보안 강화**
  - 견적서 ID(UUID) 형식 검증 미들웨어 또는 페이지 레벨 유효성 검사
  - Notion API 응답 데이터 검증 (필수 프로퍼티 누락 시 안전한 기본값 적용)
  - 환경변수 미설정 시 빌드 타임 경고 또는 런타임 안전 처리
  - HTTP 보안 헤더 설정 (next.config 활용)
  - **[구현 후 필수]** Playwright MCP로 잘못된 입력값 및 경계값 케이스 테스트

### Phase 5: 최적화 및 배포

> **왜 이 순서인가?**
> 기능이 완성된 후 품질을 검증(테스트)하고, 실제 환경에 배포한 뒤 성능을 개선하는 것이 효율적이다.
> 배포 전 테스트로 회귀를 방지하고, 배포 후 실측 데이터를 바탕으로 최적화 우선순위를 정한다.
> 미완성 기능을 최적화하는 것은 낭비다.

- **Task 009: E2E 테스트 환경 구축 및 테스트 작성**
  - Playwright 설치 및 설정 (`playwright.config.ts`)
  - **정상 케이스**: 견적서 정상 조회 플로우 (페이지 접근 → 헤더/테이블/합계 표시 확인)
  - **에러 케이스**: 잘못된 ID → Not Found 페이지, 서버 에러 → 에러 페이지 표시
  - **경계값 케이스**: 항목 0개, 최대 항목 수, 금액 0원, 소수점 단가 처리
  - PDF 저장 버튼 클릭 테스트 (window.print 호출 검증)
  - 반응형 레이아웃 테스트 (모바일/태블릿/데스크톱 뷰포트별 렌더링 확인)
  - ⛔ 모든 테스트 케이스 통과 확인 후 완료 처리

- **Task 010: Notion API 통합 테스트**
  - Notion API 모킹 전략 수립 (테스트 환경에서 실제 API 호출 없이 검증)
  - getInvoiceById 함수 단위 테스트 (정상 응답, null 반환, 에러 발생 케이스)
  - 프로퍼티 파싱 헬퍼 함수 단위 테스트 (각 타입별 정상/비정상 입력)
  - **Playwright MCP를 활용한 전체 사용자 플로우 통합 테스트**
  - 테스트 체크리스트: 정상/경계값/에러 케이스 전수 통과 확인 후 완료 처리

- **Task 011: Vercel 배포 설정 및 환경변수 구성**
  - Vercel 프로젝트 연결 및 환경변수 설정 (NOTION_API_KEY, NOTION_DATABASE_ID, NOTION_ITEMS_DATABASE_ID)
  - Notion 데이터베이스 구성 가이드 문서 작성 (프로퍼티명, 타입, relation 설정 방법)
  - `.env.example` 파일 생성 (필수 환경변수 목록 및 설명)
  - 배포 후 프로덕션 환경 동작 확인

- **Task 012: 성능 최적화 및 캐싱 전략**
  - Next.js 서버 컴포넌트 캐싱 전략 적용 (revalidate 설정으로 Notion API 호출 빈도 제어)
  - 메타데이터 최적화 (OG 이미지, 동적 title/description)
  - 번들 사이즈 분석 및 불필요한 의존성 정리
  - Lighthouse 성능 점수 측정 및 개선 (목표: Performance 90+)
