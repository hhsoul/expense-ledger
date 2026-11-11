# 내 지출 장부

날짜, 카테고리(식비·교통비·쇼핑·기타), 금액, 메모를 기록하고 월별 요약과 카테고리별 차트로 확인하는 개인 가계부입니다.
`index.html` 한 파일로 동작하고, 로그인과 데이터 저장은 Supabase를 씁니다.

## 구성

- 화면: `index.html` (GitHub Pages로 배포)
- 데이터: Supabase `public.ledger_expenses` 테이블
- 로그인: Supabase Auth (이메일 + 비밀번호)
- 보안: RLS 정책으로 로그인한 본인의 기록만 읽고 쓸 수 있음

## 보안 주의

- `index.html`에 들어 있는 Project URL과 Publishable key는 공개되어도 되는 값입니다.
- Secret key(`sb_secret_...`), `service_role` 키, 데이터베이스 비밀번호는 이 저장소에 절대 넣지 마세요.
- 새 테이블을 추가할 때는 반드시 RLS를 켜고 정책을 만드세요.

## 배포 후 설정

Supabase 대시보드 **Authentication → URL Configuration → Redirect URLs**에
GitHub Pages 주소(예: `https://<아이디>.github.io/expense-ledger/`)를 추가해야 가입 확인 메일 링크가 이 앱으로 돌아옵니다.

## 테이블 정의

```sql
create table public.ledger_expenses (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null default auth.uid() references auth.users(id) on delete cascade,
  date date not null,
  category text not null check (category in ('food','transport','shopping','etc')),
  amount bigint not null check (amount > 0),
  memo text not null default '',
  created_at timestamptz not null default now()
);

create index ledger_expenses_user_date_idx on public.ledger_expenses (user_id, date desc);

alter table public.ledger_expenses enable row level security;

create policy "본인 지출만 접근"
on public.ledger_expenses for all to authenticated
using ((select auth.uid()) = user_id)
with check ((select auth.uid()) = user_id);
```
