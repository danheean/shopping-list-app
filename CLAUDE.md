# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

쇼핑 리스트 관리 웹 애플리케이션 (단일 HTML 파일 구조)
- **Target Users**: 50세 이상 사용자를 위한 큰 폰트 크기 (22-26px)
- **Database**: Supabase (shopping_items 테이블)
- **UI Library**: Flatpickr (달력 컴포넌트)
- **Language**: 한국어 UI, 한국식 날짜 형식 (YYYY년 M월)

## Database Schema

```sql
CREATE TABLE shopping_items (
    id BIGSERIAL PRIMARY KEY,
    text TEXT NOT NULL,
    checked BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    completed_at TIMESTAMPTZ  -- 체크 시 자동 설정
);
```

## Development Setup

### Supabase MCP Connection
프로젝트는 `.mcp.json`에 정의된 Supabase MCP 서버를 사용합니다.
```bash
# Claude Code가 자동으로 연결 (mcp 명령어)
/mcp
```

### Local Testing
```bash
# HTTP 서버 실행 (포트 8888)
python3 -m http.server 8888

# 브라우저에서 접속
# http://localhost:8888/index.html
```

### Database Operations
Supabase MCP 도구를 사용:
```javascript
// SQL 실행
mcp__supabase__execute_sql

// 마이그레이션 적용
mcp__supabase__apply_migration
```

## Architecture

### Single-File Structure
모든 코드가 `index.html`에 포함:
- HTML 구조
- CSS 스타일 (노트북 디자인)
- JavaScript (ES6 모듈, Supabase 클라이언트)

### Key Components

1. **데이터 로딩** (`loadItems`)
   - Supabase에서 shopping_items 조회
   - created_at 오름차순 정렬
   - 로딩 상태 관리 (`isLoading` 플래그)

2. **아이템 추가** (`addItem`)
   - 중복 방지: `isAdding` 플래그
   - 한글 IME 지원: `e.isComposing` 체크
   - Supabase insert 후 자동 리로드

3. **체크박스 토글** (`toggleItem`)
   - checked 상태 변경 시 completed_at 자동 설정/해제
   - completed_at = checked ? NOW() : NULL

4. **날짜 필터** (Flatpickr)
   - 등록일 또는 완료일 기준 필터링
   - 점 표시: 해당 날짜에 항목이 있는 경우
   - 타임존 주의: 문자열에서 직접 날짜 추출 (`.split('T')[0]`)

### Critical Implementation Details

**한글 입력 처리**
```javascript
// Enter 키 이벤트에서 반드시 isComposing 체크
itemInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && !e.isComposing) {
        e.preventDefault();
        addItem();
    }
});
```

**타임존 이슈**
```javascript
// ❌ 잘못된 방법 (타임존 변환으로 날짜가 바뀔 수 있음)
const date = new Date(item.created_at);
dates.add(date.toISOString().split('T')[0]);

// ✅ 올바른 방법 (문자열에서 직접 추출)
const dateStr = item.created_at.split('T')[0];
dates.add(dateStr);
```

**Flatpickr 한국어 설정**
- 월/연도 순서: `flex-direction: row-reverse` 사용
- "년" 추가: JavaScript로 동적 요소 삽입
- 폰트 크기: 22px (가독성 최적화)

## Testing

Playwright MCP를 사용한 E2E 테스트:
```javascript
// 로컬 서버 실행 필요
python3 -m http.server 8888 &

// Playwright 도구 사용
mcp__playwright__browser_navigate
mcp__playwright__browser_snapshot
mcp__playwright__browser_click
mcp__playwright__browser_fill_form
```

## UI/UX Guidelines

- **폰트 크기**: 최소 18px, 중요 텍스트 20-26px
- **체크박스**: 26x26px
- **버튼**: padding 14px 이상
- **날짜 형식**: "2026년 2월" (한국식)
- **아이콘**: 📅 (등록), ✅ (완료)

## Common Issues

1. **달력에 잘못된 날짜 점 표시**
   - 원인: 타임존 변환 버그
   - 해결: 문자열 직접 파싱 사용

2. **한글 입력 시 중복 추가**
   - 원인: IME 조합 중 Enter 이벤트 발생
   - 해결: `e.isComposing` 체크 필수

3. **데이터 로딩 전 빈 화면 깜빡임**
   - 원인: 초기 렌더링 후 데이터 로드
   - 해결: `isLoading` 플래그로 로딩 상태 표시
