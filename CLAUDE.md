# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

- PRD 문서: @docs/PRD.md
- 개발 로드맵: @docs/ROADMAP.md

## 언어 및 커뮤니케이션 규칙

- **기본 응답 언어**: 한국어
- **코드 주석**: 한국어로 작성
- **커밋 메시지**: 한국어로 작성
- **문서화**: 한국어로 작성
- **변수명/함수명**: 영어 (코드 표준 준수)

## 주요 명령어

```bash
npm run dev      # Turbopack으로 개발 서버 실행 (localhost:3000)
npm run build    # Turbopack으로 프로덕션 빌드
npm run start    # 프로덕션 서버 실행
npm run lint     # ESLint 검사
```

테스트 설정 없음.

shadcn/ui 컴포넌트 추가:
```bash
npx shadcn add <component-name>
```

## 프로젝트 개요

Next.js App Router 기반의 모던 웹 스타터킷. 주요 라이브러리: Next.js 16, React 19, TypeScript 5, Tailwind CSS v4, shadcn/ui (new-york 스타일), TanStack Table, Zustand, React Hook Form + Zod, Sonner, next-themes.

## 아키텍처

### 디렉토리 구조

```
src/
  app/              # Next.js App Router 페이지
    examples/       # 컴포넌트 예제 페이지 (/examples)
  components/
    ui/             # shadcn/ui 컴포넌트 (직접 수정 가능)
    common/         # 재사용 공통 컴포넌트 (DataTable, FeaturesSection 등)
    layout/         # 레이아웃 컴포넌트 (Header, Footer, RootLayout 등)
  store/            # Zustand 스토어
  types/            # TypeScript 타입 정의
  hooks/            # 커스텀 훅
  data/             # 정적 데이터
  lib/              # 유틸리티 함수
```

경로 별칭: `@/` → `src/`

### 서버/클라이언트 컴포넌트 패턴

- 페이지(`page.tsx`)는 기본적으로 서버 컴포넌트
- 인터랙션이 필요한 클라이언트 컴포넌트는 별도 파일로 분리
- 동일 디렉토리 내 클라이언트 파일은 `_` 접두사 사용 (예: `_user-table.tsx`)
- `'use client'` 범위를 최소화하여 서버 렌더링 최대화

### 전역 프로바이더 구조

`src/components/layout/root-layout.tsx`에서 세 개 프로바이더를 한 번에 설정:
- `NextThemesProvider` (next-themes, 시스템 테마 기본값)
- `TooltipProvider` (전역 단일 마운트)
- `Toaster` (Sonner 토스트)

### 상태 관리

`src/store/ui-store.ts`: Zustand로 글로벌 UI 상태(사이드바 열림/닫힘) 관리. 새 스토어 추가 시 같은 디렉토리에 파일을 추가.

### 데이터 테이블

`src/components/common/data-table.tsx`가 제네릭 `DataTable<TData, TValue>` 컴포넌트를 제공. TanStack Table 위에 shadcn/ui Table을 래핑하며 정렬 기능을 내장. 사용처에서 `ColumnDef[]`와 `data[]`를 props로 전달.

### 타입 정의

`src/types/index.ts`에서 프로젝트 공통 타입(`NavItem`, `FeatureCard`, `TableRow` 등)을 중앙 관리.

## 코딩 규칙

- `any` 타입 사용 금지
- 컴포넌트는 기능별로 분리하여 재사용
- 반응형 레이아웃 필수 (Tailwind 반응형 접두사 활용)
- 파일 삭제 시 반드시 사용자 동의 후 진행
