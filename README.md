# 🛒 Shopping List App (Supabase 연동)

Supabase Postgres에 데이터를 저장하는 가볍고 이쁘은 **쇼핑 리스트 웹앱**입니다. 단일 HTML 파일로 동작하며, 동공된 정적 서버(`server.js`)로 로컬에서 손쉽게 실행할 수 있습니다.

---

## ✨ 주요 기능

- ➕ **아이템 추가** — 입력 후 "추가" 또는 `Enter` 키로 등록 → Supabase에 즉시 저장
- ✅ **체크하기 / 해제** — 구매 완료 표시 (취소선 효과) + DB 상태 동기화
- ✖️ **개별 삭제** — 항목 오른쪽 ✕ 버튼으로 제거
- 🧹 **완료 항목 일괄 비우기** — `checked = true` 행만 한 번에 삭제
- 📊 **진행 상황 표시** — 총 항목 수 · 남은 항목 수
- ☁️ **원격 영속 저장** — 어느 브라우저/기기에서도 동일한 리스트를 볼 수 있음
- 📡 **에러/상태 피드백** — 네트워크 실패 시 상단 배너로 알림
- 📱 **반응형 UI** — 모바일/태블릿/데스크톱 대응

---

## 🚀 빠른 시작

### 1. 단일 HTML로 열기

`index.html`을 브라우저에 드래그 하거나 더블클릭하면 증시 실행됩니다. supabase-js는 CDN으로 로드됩니다.

### 2. 로컬 정적 서버로 실행

Node.js가 설치되어 있으면 포함된 경량 서버를 사용할 수 있습니다:

```bash
node server.js
# → http://localhost:8765/
```

포트를 바꾸려면 `server.js` 상단의 `PORT` 상수를 수정하세요.

---

## 🗄️ Supabase 설정

### 접속 정보

`index.html`의 아래 상수를 본인 프로젝트에 맞게 변경하세요:

```js
const SUPABASE_URL = "https://<your-project-ref>.supabase.co";
const SUPABASE_PUBLISHABLE_KEY = "sb_publishable_...";
```

> Publishable / anon 키는 클라이언트 노출을 전제로 한 공개 키입니다. 서비스 키(`service_role`)는 절대 웹에 노출하지 마세요.

### 테이블 스키마

```sql
create table public.shopping_items (
  id uuid primary key default gen_random_uuid(),
  text text not null check (char_length(text) between 1 and 100),
  checked boolean not null default false,
  created_at timestamptz not null default now()
);

create index shopping_items_created_at_idx
  on public.shopping_items (created_at);
```

### Row Level Security (데모용 정책)

인증 없는 데모이므로 `anon` 역할에 전체 CRUD를 허용합니다. **실 서비스에서는 반드시 `auth.uid()` 기반 정책으로 교체하세요.**

```sql
alter table public.shopping_items enable row level security;

create policy "anon_select_shopping_items"
  on public.shopping_items for select to anon using (true);
create policy "anon_insert_shopping_items"
  on public.shopping_items for insert to anon with check (true);
create policy "anon_update_shopping_items"
  on public.shopping_items for update to anon using (true) with check (true);
create policy "anon_delete_shopping_items"
  on public.shopping_items for delete to anon using (true);
```

---

## 📁 프로젝트 구조

```
shopping-listapp/
├── index.html      # UI + Supabase 연동 로직
├── server.js       # Node 정적 서버 (선택)
└── README.md
```

---

## 🔧 기술 스택

- **HTML5 / CSS3 / Vanilla JavaScript** — 프레임워크 없음
- **[Supabase](https://supabase.com/)** — Postgres 기반 BaaS, REST/Realtime API
- **[`@supabase/supabase-js` v2](https://github.com/supabase/supabase-js)** — 브라우저 SDK (CDN 로드)
- **Node.js `http` 모듈** — 제로-시 정적 서버

---

## 🔄 데이터 흐름

```
[브라우저] --(supabase-js)--> [Supabase REST API] --> [Postgres: shopping_items]
```

- 시작 시: `select(*).order(created_at)` 로 전체 항목 로드
- 추가: `insert({text, checked:false}).select().single()`
- 수정: `update({checked}).eq("id", id)`
- 개별 삭제: `delete().eq("id", id)`
- 일괄 삭제: `delete().eq("checked", true)`

---

## 🧩 키보드 조작

| 키 | 동작 |
|---|---|
| `Enter` | 현재 입력한 아이템 추가 |

---

## 🛣️ 향후 개선 아이디어

- [ ] Supabase Auth로 사용자별 리스트 분리 (`user_id` 컬럼 + RLS)
- [ ] Realtime 구독으로 여러 기기 동시 동기화
- [ ] 카테고리/태그 지원
- [ ] 드래그 앤 드롭 정렬
- [ ] 수량 및 단위 입력
- [ ] 다크 모드 / PWA 화

---

## 📄 라이선스

MIT License
