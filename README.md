# 쇼핑 리스트

50세 이상 사용자를 위한 큰 글씨 쇼핑 리스트 웹 애플리케이션

![쇼핑 리스트 메인 화면](shopping-list-test.png)

## 주요 기능

- 쇼핑 항목 추가 / 체크(완료) / 삭제
- 날짜별 필터링 (등록일 또는 완료일 기준)
- 달력에 항목이 있는 날짜 점 표시
- 한글 IME 입력 지원
- 모바일 반응형 디자인

![날짜 필터링 화면](shopping-list-filtered.png)

## 기술 스택

| 구분 | 기술 |
|------|------|
| Frontend | HTML, CSS, JavaScript (단일 파일) |
| Database | [Supabase](https://supabase.com) (PostgreSQL) |
| UI 라이브러리 | [Flatpickr](https://flatpickr.js.org) (달력) |
| 배포 | [Vercel](https://vercel.com) |
| 소스 관리 | [GitHub](https://github.com) |

## 데이터베이스

### `shopping_items` 테이블

| 컬럼 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `id` | `bigint` (PK) | auto increment | 고유 식별자 |
| `text` | `text` | - | 쇼핑 항목 이름 |
| `checked` | `boolean` | `false` | 완료 여부 |
| `created_at` | `timestamptz` | `now()` | 등록 일시 |
| `completed_at` | `timestamptz` | `null` | 완료 일시 (체크 시 자동 설정) |

```sql
CREATE TABLE shopping_items (
    id BIGSERIAL PRIMARY KEY,
    text TEXT NOT NULL,
    checked BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    completed_at TIMESTAMPTZ
);
```

- RLS(Row Level Security) 활성화
- `checked` 변경 시 `completed_at` 자동 설정/해제

## 로컬 실행

```bash
# HTTP 서버 실행
python3 -m http.server 8888

# 브라우저에서 접속
open http://localhost:8888
```

## 프로젝트 구조

```
study-07/
├── index.html          # 전체 애플리케이션 (HTML + CSS + JS)
├── CLAUDE.md           # Claude Code 개발 가이드
└── README.md
```
