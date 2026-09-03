# 할일

반복 일정(매일/매주/매월/매년/특정일), 프로젝트 진행률, 오늘 모아보기, 캘린더 보기를 지원하는 할 일 관리 앱이에요. 서버 없이 정적 파일(HTML 한 장)로 동작하고, 로그인하면 여러 기기 간에 자동으로 동기화돼요.

가계부 앱(household-ledger)과는 완전히 독립된 앱이에요 — 다른 저장소, 다른 로그인 계정, 다른 데이터 저장 공간을 씁니다.

## 실제 주소

배포되면 다음 주소에서 쓸 수 있어요 (GitHub Pages 활성화 후):

```
https://myeonggyu77.github.io/todo-app/
```

## 처음 설정하기 (클라우드 동기화용 Supabase 연결)

지금은 `index.html` 안의 Supabase 키가 비어있는 상태(`YOUR_SUPABASE_PROJECT_URL` 등)라, 로그인 없이 이 기기의 브라우저에만 저장돼요. 여러 기기에서 동기화하려면 아래 순서로 본인 계정의 무료 Supabase 프로젝트를 만들어서 연결해주세요 (5분 정도 걸려요).

### 1. Supabase 프로젝트 생성

1. https://supabase.com 에서 회원가입 후 **New project** 생성
2. 프로젝트가 만들어지면 왼쪽 메뉴 **Project Settings > API** 로 이동
3. **Project URL** 과 **anon public** key를 복사해두기

### 2. 데이터 테이블 만들기

Supabase 프로젝트의 **SQL Editor**에서 아래 SQL을 실행하세요:

```sql
create table todo_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null,
  updated_at timestamptz not null default now()
);

alter table todo_data enable row level security;

create policy "본인 데이터만 조회"
  on todo_data for select
  using (auth.uid() = user_id);

create policy "본인 데이터만 추가"
  on todo_data for insert
  with check (auth.uid() = user_id);

create policy "본인 데이터만 수정"
  on todo_data for update
  using (auth.uid() = user_id);
```

### 3. 이메일/비밀번호 로그인 활성화

**Authentication > Providers**에서 **Email** 이 켜져 있는지 확인하세요 (기본값으로 켜져 있어요). 가입 시 이메일 인증을 요구할지 여부는 **Authentication > Settings**에서 조절할 수 있어요.

### 4. 앱에 키 연결하기

`index.html`을 열어서 최상단 `<script>` 블록에 있는 아래 두 줄을 1번에서 복사한 값으로 바꿔주세요:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_PROJECT_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

저장하고 다시 배포(main 브랜치에 push)하면 로그인/클라우드 동기화가 바로 동작해요.

## 기능

- **목록**: 일반 리스트 / 프로젝트 리스트(진행률 %) 두 형식
- **반복**: 매일 · 매주 · 매월 · 매년 · 달력에서 여러 날짜를 직접 지정하는 반복
- **빠른 입력 자연어 인식**: "내일 우유 사기", "매주 월요일 청소" 처럼 입력하면 날짜/반복을 자동으로 인식
- **오늘 보기**: 모든 리스트를 가로질러 오늘 마감·지난 할 일을 한 화면에
- **캘린더 보기**: 날짜별 할 일 개수를 달력으로, 날짜 클릭 시 그날 할 일 모아보기
- **하위 체크리스트**: 모든 리스트에서 할 일 아래에 하위 할 일 추가 가능
- **반복 완료 스트릭**: 반복 할 일을 연속으로 완료한 횟수 기록
- **완료 항목 숨기기/보이기**: 목록별로 토글
- **드래그로 순서 변경**
- **PWA**: 홈 화면에 추가해서 앱처럼 사용 가능
