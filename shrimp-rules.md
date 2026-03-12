# Development Guidelines

## Project Overview

- Notion 데이터베이스 기반 견적서 단일 목적 웹 뷰어
- 경로: `/quote/[id]` — 인증 없음, 고유 URL 직접 접근
- 스택: Next.js 16 App Router, TypeScript strict, Tailwind CSS v4, shadcn/ui, @notionhq/client
- 경로 별칭: `@/` → `src/`

---

## Project Architecture

### 핵심 파일 위치

| 역할 | 파일 경로 |
|------|-----------|
| Notion API 클라이언트 + 데이터 조회 | `src/lib/notion.ts` |
| 도메인 타입 (Invoice, InvoiceItem) | `src/types/index.ts` |
| UI 타입 (ButtonVariant, BREAKPOINTS) | `src/types/ui.ts` |
| 견적서 렌더링 (서버 컴포넌트) | `src/app/quote/[id]/page.tsx` |
| PDF 저장 버튼 (클라이언트 컴포넌트) | `src/app/quote/[id]/_print-button.tsx` |
| 존재하지 않는 견적서 안내 | `src/app/quote/[id]/not-found.tsx` |
| 전역 스타일 + @media print | `src/app/globals.css` |
| 공통 재사용 컴포넌트 | `src/components/common/` |
| shadcn/ui 컴포넌트 | `src/components/ui/` |
| 전역 레이아웃 + 프로바이더 | `src/components/layout/root-layout.tsx` |

---

## Server / Client Component Rules

- **`src/app/quote/[id]/page.tsx`는 서버 컴포넌트 유지 필수** — `'use client'` 추가 절대 금지
- Notion API 호출은 **서버사이드 전용** — `src/lib/notion.ts`를 클라이언트 컴포넌트에서 import 금지
- 클라이언트 인터랙션이 필요하면 같은 디렉토리에 `_` 접두사 파일로 분리
  - ✅ `src/app/quote/[id]/_some-client.tsx`
  - ❌ `src/app/quote/[id]/some-client.tsx` (접두사 없음)

---

## Notion Database Property Names (변경 금지)

### Invoice DB (`NOTION_DATABASE_ID`)

| 프로퍼티명 | Notion 타입 | 파싱 함수 |
|-----------|-------------|----------|
| `quoteNumber` | title | `extractTitle` |
| `clientName` | rich_text | `extractRichText` |
| `issueDate` | date | `extractDate` |
| `validUntil` | date | `extractDate` |
| `status` | select | `extractSelect` |
| `note` | rich_text | `extractRichText` |

### InvoiceItem DB (`NOTION_ITEMS_DATABASE_ID`)

| 프로퍼티명 | Notion 타입 | 파싱 함수 |
|-----------|-------------|----------|
| `itemName` | rich_text | `extractRichText` |
| `quantity` | number | `extractNumber` |
| `unitPrice` | number | `extractNumber` |
| `amount` | number | `extractNumber` |
| `invoiceId` | relation | 직접 파싱 |

---

## Multi-File Sync Rules

**Invoice 타입 변경 시 아래 3개 파일 동시 수정 필수:**
1. `src/types/index.ts` — 타입 인터페이스
2. `src/lib/notion.ts` — `parseInvoice()` 함수
3. `src/app/quote/[id]/page.tsx` — 렌더링 코드

**InvoiceItem 타입 변경 시 아래 3개 파일 동시 수정 필수:**
1. `src/types/index.ts` — 타입 인터페이스
2. `src/lib/notion.ts` — `parseInvoiceItem()` 함수
3. `src/app/quote/[id]/page.tsx` — 테이블 렌더링 코드

---

## Type Rules

- `any` 타입 사용 **절대 금지**
- `InvoiceStatus` = `'draft' | 'sent' | 'expired'` — 이 3가지 외 값 추가 금지
- Notion API 응답 타입은 `@notionhq/client/build/src/api-endpoints`에서 import
- `isFullPage()` 가드 통과한 페이지만 파싱

---

## Styling Rules

- Tailwind CSS v4: `tailwind.config` 파일 **없음** — `@import "tailwindcss"` 문법 사용
- 인쇄 시 숨길 요소: `print:hidden` 클래스 또는 `className="... print:hidden"`
- `@media print` 추가 스타일: `src/app/globals.css` 하단에만 작성
- Card 컴포넌트 인쇄 타겟: `[data-slot="card"]` 선택자 사용
- 반응형 레이아웃 **필수** — Tailwind 반응형 접두사(`sm:`, `md:`, `lg:`) 사용

---

## Format Functions

- **원화 포맷**: `formatKRW(amount: number)` — `src/app/quote/[id]/page.tsx` 위치
- **날짜 포맷**: `formatDate(dateStr: string)` — `src/app/quote/[id]/page.tsx` 위치
- **부가세 계산**: `Math.round(subtotal * 0.1)` — 반올림 필수

---

## Adding Components

- shadcn/ui 컴포넌트 추가: `npx shadcn add <component-name>`
- 공통 재사용 컴포넌트: `src/components/common/` 에 생성
- 페이지별 클라이언트 컴포넌트: `_` 접두사 + 같은 디렉토리

---

## Environment Variables

- 필수 3개: `NOTION_API_KEY`, `NOTION_DATABASE_ID`, `NOTION_ITEMS_DATABASE_ID`
- `.env.local`에 설정 — 서버사이드 전용, 클라이언트에서 접근 불가
- 환경변수 미설정 시 Notion API 호출 불가 — 빌드 전 반드시 확인

---

## E2E Testing

- API 연동 및 비즈니스 로직 구현 후 **Playwright MCP로 E2E 테스트 필수**
- 테스트 순서: 정상 케이스 → 경계값/엣지 케이스 → 에러 케이스
- **⛔ 모든 케이스 통과 전까지 완료 처리 금지**

---

## Prohibited Actions

- `src/app/quote/[id]/page.tsx`에 `'use client'` 추가 금지
- `src/lib/notion.ts`를 클라이언트 컴포넌트에서 import 금지
- `any` 타입 사용 금지
- `console.log` 직접 사용 금지 (로깅 라이브러리 사용)
- Notion DB 프로퍼티명 임의 변경 금지 (실제 Notion DB와 동기화 필수)
- 파일 삭제 전 사용자 동의 없이 진행 금지
- `tailwind.config.ts` 파일 생성 금지 (v4는 설정 파일 없음)
- 요청 범위 외 코드 수정 금지
- 사용되지 않는 추상화·확장성 추가 금지

---

## AI Decision Tree

```
새 기능 추가?
├── 인터랙션 필요?
│   ├── YES → 같은 디렉토리 `_` 접두사 파일 생성 + 'use client'
│   └── NO  → page.tsx 서버 컴포넌트에서 직접 구현
│
├── Notion 데이터 필요?
│   └── 반드시 src/lib/notion.ts에 함수 추가 → page.tsx에서 호출
│
├── 새 타입 필요?
│   └── src/types/index.ts에 추가
│
└── UI 컴포넌트 필요?
    ├── shadcn/ui 있음? → npx shadcn add <name>
    ├── 재사용? → src/components/common/
    └── 페이지 전용? → 같은 디렉토리 `_` 접두사

타입 변경?
└── Invoice/InvoiceItem 변경 → types/index.ts + notion.ts + page.tsx 동시 수정

스타일 추가?
├── 인쇄 관련? → src/app/globals.css @media print 섹션
└── 일반 UI? → Tailwind 클래스 직접 사용
```
