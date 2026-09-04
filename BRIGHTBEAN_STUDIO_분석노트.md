# BrightBean Studio 분석 노트 🫘

> 이 문서는 BrightBean Studio 프로젝트를 처음 받아본 뒤,
> **"이게 뭔지 → 어떻게 설치·사용하는지 → 어떻게 돈을 벌 수 있는지 → 다른 언어로 바꿀 수 있는지"**
> 순서로 분석한 내용을 정리한 것이다.

## 🔗 관련 링크

| 구분 | 주소 |
|---|---|
| **원본 저장소 (upstream)** | https://github.com/brightbeanxyz/brightbean-studio |
| **내 저장소 (fork/작업본)** | https://github.com/bmshin94/brightbean-studio |
| **AI 에이전트 스킬 (공식 컴패니언)** | https://github.com/brightbeanxyz/brightbean-studio-agent |
| **무료 호스팅 버전 (체험용)** | https://brightbean.xyz/studio/ |

---

## 목차

1. [이게 뭐 하는 프로젝트인가](#1-이게-뭐-하는-프로젝트인가)
2. [폴더 구조 뜯어보기](#2-폴더-구조-뜯어보기)
3. [설치 방법](#3-설치-방법)
4. [실행 방법](#4-실행-방법)
5. [SNS 연결 방법](#5-sns-연결-방법)
6. [실제 사용 흐름](#6-실제-사용-흐름)
7. [AI(MCP) 연동](#7-aimcp-연동)
8. [자주 터지는 문제](#8-자주-터지는-문제)
9. [수익화 아이디어](#9-수익화-아이디어)
10. [React / PHP로 다시 만들 수 있나](#10-react--php로-다시-만들-수-있나)
11. [동기별 최종 판단 (실제 상황)](#11-동기별-최종-판단-실제-상황)
12. [핵심 요약](#12-핵심-요약)

---

## 1. 이게 뭐 하는 프로젝트인가

**한 줄 요약: 여러 SNS 계정을 한 화면에서 기획 → 작성 → 예약 → 승인 → 발행 → 성과분석까지 하는 오픈소스 웹앱.**

Sendible, SocialPilot, ContentStudio 같은 **월 15~40만원짜리 유료 SaaS**를 대체하는 무료·자체호스팅 버전이다.
계정 수, 채널 수, 팀원 수 제한이 전혀 없다.

### 지원 플랫폼 (13개)

| 플랫폼 | 발행 | 댓글 | DM | 통계 |
|---|:---:|:---:|:---:|:---:|
| Facebook | ✓ | ✓ | ✓ | ✓ |
| Instagram | ✓ | ✓ | ✓ | ✓ |
| Instagram (Direct) | ✓ | ✓ | ✓ | ✓ |
| LinkedIn (개인) | ✓ | ✓ | — | ✓ |
| LinkedIn (회사) | ✓ | ✓ | — | ✓ |
| TikTok | ✓ | — | — | ✓ |
| YouTube | ✓ | ✓ | — | ✓ |
| Pinterest | ✓ | — | — | ✓ |
| Threads | ✓ | ✓ | — | ✓ |
| Bluesky | ✓ | ✓ | — | — |
| Google 비즈니스 프로필 | ✓ | — | — | ✓ |
| Mastodon | ✓ | ✓ | — | — |
| DEV.to | ✓ | — | — | — |

> 중간 대행 API(aggregator)를 쓰지 않고 **각 플랫폼 공식 API에 직접** 붙는다. 데이터가 제3자를 거치지 않는다.

### 기술 스택

| 계층 | 기술 |
|---|---|
| 백엔드 | Django 5.x (Python 3.12+, 이 프로젝트는 3.13 기준) |
| 프론트엔드 | Django 템플릿 + HTMX + Alpine.js (React 미사용) |
| CSS | Tailwind CSS 4 (django-tailwind) |
| DB | PostgreSQL 16+ (로컬은 SQLite 가능) |
| 백그라운드 작업 | django-background-tasks (**Redis/Celery 불필요**) |
| 인증 | django-allauth (이메일 + 구글 OAuth) |
| API | django-ninja (REST) + MCP SDK |
| 배포 | Docker + Gunicorn + Caddy(자동 HTTPS) |

### 프로젝트 규모 (실측)

| 항목 | 수치 |
|---|---|
| 파이썬 코드 | **76,115줄** |
| HTML 화면 | 240개 |
| DB 테이블(모델) | 65개 |
| 화면 처리 로직(뷰) | 356개 |
| 플랫폼 연동 코드 | 8,154줄 |
| DB 마이그레이션 | 83개 |

> 취미 프로젝트가 아니라 **실제 서비스 중인 제품 규모**다.

### 라이선스

**AGPL-3.0** — 자세한 내용은 [9. 수익화 아이디어](#9-수익화-아이디어) 참고. (수익화 전략에 직접 영향)

---

## 2. 폴더 구조 뜯어보기

```
brightbean-studio/
├── apps/                  # 핵심 기능 23개 (Django 앱)
├── providers/             # 플랫폼별 API 연동 "통역사" (8,154줄)
├── templates/             # 화면 HTML 240개
├── theme/                 # Tailwind CSS 소스
├── static/                # 이미지, JS (htmx, alpine)
├── config/                # Django 설정 (base/development/production/test)
├── development_specs/     # 설계 문서 3,300줄 ← 구조 파악에 최고
├── tests/                 # 테스트
├── .env.example           # 환경변수 템플릿 ← 설치 시 필수
├── docker-compose.yml     # Docker 실행 설정
├── Makefile               # make setup / make server 등 단축 명령
├── README.md              # 공식 사용설명서 (40KB, 매우 상세)
└── CLAUDE.md              # AI 어시스턴트 지침
```

### `apps/` 안의 주요 기능 (코드량 순)

| 앱 | 줄 수 | 하는 일 |
|---|---:|---|
| `composer/` | 10,508 | 글쓰기 에디터, 플랫폼별 문구 override, 템플릿, 아이디어 칸반보드 |
| `api/` | 7,073 | 외부 에이전트용 REST API |
| `social_accounts/` | 6,000 | 각 SNS OAuth 연결 플로우 |
| `calendar/` | 5,179 | 달력, 반복 발행 슬롯, 자동 큐 |
| `intelligence/` | 4,492 | **유료 AI 분석 애드온** (원작자 수익모델) |
| `media_library/` | 4,080 | 미디어 보관함, 폴더, 플랫폼별 변형 자동생성 |
| `mcp/` | 4,066 | **MCP 서버** — AI 에이전트가 조종하는 통로 |
| `inbox/` | 3,695 | 통합 댓글/DM/멘션함, 감정분석 |
| `analytics/` | 3,732 | 성과 대시보드 |
| `api_keys/` | 3,081 | 워크스페이스 범위 API 키 발급/폐기 |
| `publisher/` | 2,411 | 발행 엔진, 자동 재시도, 90일 감사로그 |
| `members/` | 2,172 | RBAC 권한, 초대 |
| `approvals/` | 2,168 | 승인 워크플로우 |
| `notifications/` | 1,540 | 알림 (인앱/이메일/웹훅) |
| `onboarding/` | 1,148 | 첫 사용자 체크리스트 |
| `client_portal/` | 1,091 | 클라이언트 매직링크 포털 |
| `accounts/` | 1,037 | 로그인/회원가입 |
| `credentials/` | 930 | 플랫폼 API 키 암호화 저장 |
| `oauth_server/` | 456 | MCP용 OAuth 2.1 인증 서버 |
| `organizations/` `workspaces/` `settings_manager/` `common/` | — | 조직/워크스페이스/설정/공용 유틸 |

### `providers/` — 이 프로젝트의 진짜 자산

```
providers/
├── base.py              # 추상 인터페이스 (모든 플랫폼의 공통 규격)
├── types.py             # 공통 타입 정의
├── exceptions.py        # 에러 정의
├── facebook.py          # 페이스북
├── instagram.py         # 인스타 (Facebook Login 방식)
├── instagram_login.py   # 인스타 Direct (Instagram Login 방식)
├── threads.py           # 스레드
├── linkedin.py / linkedin_personal.py / linkedin_company.py
├── tiktok.py
├── youtube.py           # 671줄
├── pinterest.py
├── bluesky.py
├── mastodon.py
├── google_business.py
├── devto.py
├── meta_insights.py     # 메타 통계
├── meta_comments.py     # 메타 댓글
└── meta_messaging.py    # 메타 DM
```

플랫폼마다 업로드 방식·토큰 만료·rate limit·에러코드·웹훅 서명 방식이 전부 다르다.
**이 8,154줄이 "각 플랫폼의 지랄맞은 규칙을 다 겪어본 경험치" 그 자체**다.

---

## 3. 설치 방법

### 준비물

| 필요한 것 | 버전 | 확인 명령어 |
|---|---|---|
| Python | 3.12+ (이 프로젝트는 3.13) | `python3 --version` |
| Node.js | 20+ | `node --version` |
| Git | 아무거나 | `git --version` |

> PostgreSQL은 **설치 안 해도 된다.** 로컬은 SQLite 파일 하나로 굴러간다.

### 방법 A — 로컬 설치 (가장 쉬움)

**1) 환경파일 만들기**
```bash
cd brightbean-studio
cp .env.example .env
```

**2) `.env`에서 딱 3줄 수정**
```bash
SECRET_KEY=랜덤한-긴-문자열-50자이상
ENCRYPTION_KEY_SALT=다른-랜덤-문자열
DATABASE_URL=sqlite:///db.sqlite3
```

랜덤 문자열 생성:
```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

> ⚠️ `ENCRYPTION_KEY_SALT`는 **SNS 토큰 암호화 소금**이다. 나중에 바꾸면 저장된 연결이 전부 깨진다. 한 번 정하면 절대 건드리지 말 것.

나머지 항목(플랫폼 API 키 등)은 일단 비워둬도 실행된다.

**3) 한방에 설치**
```bash
make setup
```

이게 하는 일 = 아래 4단계와 동일:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cd theme/static_src && npm install && cd ../..
python manage.py migrate
```

**4) 관리자 계정 만들기**
```bash
python manage.py createsuperuser
```
> 비밀번호 입력 시 화면에 아무것도 안 보이는 건 정상.

### 방법 B — Docker (터미널 하나로 끝)

```bash
cp .env.example .env
# .env에서 DATABASE_URL만 아래로 변경
#   DATABASE_URL=postgres://postgres:postgres@postgres:5432/brightbean

docker compose up -d --build
docker compose exec app python manage.py createsuperuser
```

마이그레이션과 Tailwind 빌드가 자동으로 돌아간다. 첫 빌드 60~90초.
로그 확인: `docker compose logs -f`

---

## 4. 실행 방법

로컬 설치의 경우 **터미널 3개**를 켜야 한다.

### 터미널 1 — 웹서버 (필수)
```bash
source .venv/bin/activate
python manage.py runserver
```

### 터미널 2 — 백그라운드 일꾼 (필수)
```bash
source .venv/bin/activate
python manage.py process_tasks
```

> 🚨 **이걸 안 켜면 예약 발행이 절대 안 된다.** 가장 흔한 실수 1위.

### 터미널 3 — Tailwind CSS 감시 (개발 시)
```bash
cd theme/static_src && npm run start
```

접속: **http://localhost:8000**

### 첫 화면에서 할 일 (순서 중요)

```
조직(Organization) 생성
      ↓
워크스페이스(Workspace) 생성   ← 클라이언트/브랜드 하나당 하나
      ↓
SNS 계정 연결
      ↓
글 작성 / 예약
```

- **조직** = 우리 회사 (예: "OO마케팅")
- **워크스페이스** = 담당 브랜드 (예: "A카페", "B헬스장")
  → 브랜드별로 SNS 계정, 글, 팀원 권한이 완전히 분리된다.

대시보드에 **"Get Started" 체크리스트 5개**가 뜬다 (`apps/onboarding/checklist.py`):
1. SNS 계정 연결
2. 첫 글 작성
3. 첫 아이디어 등록
4. 팀원 초대
5. (마지막 항목)

---

## 5. SNS 연결 방법

### 난이도 하 — 개발자 등록 불필요 (여기부터 시작 추천)

| 플랫폼 | 방법 |
|---|---|
| **Bluesky** | Bluesky → Settings → Privacy and Security → **App Passwords** 생성 → 핸들 + 앱비번 입력 |
| **Mastodon** | 인스턴스 주소(`mastodon.social`)만 입력. OAuth 앱이 자동 등록됨 |
| **DEV.to** | DEV.to → Settings → Extensions → API Key 생성 → 붙여넣기 |

> 👉 **처음엔 Bluesky로 연결해서 글 하나 올려보는 걸 강력 추천.** 전체 흐름을 5분이면 파악한다.

### 난이도 상 — 개발자 앱 등록 필요

Instagram, Facebook, TikTok, YouTube, LinkedIn, Pinterest, Threads, Google 비즈니스 프로필.

**공통 규칙 — 콜백(Redirect URI) 형식:**
```
{APP_URL}/social-accounts/callback/{플랫폼}/
```
예: `http://localhost:8000/social-accounts/callback/facebook/`

> ⚠️ **끝의 슬래시(`/`) 필수.** 이거 하나 빠져서 실패하는 경우가 매우 많다.
> ⚠️ **TikTok만 예외** — 주소에 `tiktok` 문자열을 넣을 수 없어서 `social1`을 쓴다. (TikTok이 자기 브랜드명 포함 URI를 거부함)

### Meta (Facebook / Instagram / Threads) 설정 요약

1. [Meta for Developers](https://developers.facebook.com/) → 앱 생성 (타입: **Business**)
2. App Settings → Basic 에서 **App ID / App Secret** 복사
3. **Use cases** 4개 추가 + 각각 권한 추가
   - "Manage everything on your Page" → `pages_manage_posts`, `pages_manage_engagement`, `pages_read_engagement`, `pages_read_user_content`, `pages_manage_metadata`, `read_insights`
   - "Messenger from Meta" → `pages_messaging`
   - "Manage messaging & content on Instagram" → `instagram_basic`, `instagram_content_publish`, `instagram_manage_comments`, `instagram_manage_insights`
   - "Access the Threads API" → `threads_content_publish`, `threads_manage_insights`, `threads_manage_replies`
4. Facebook Login → Settings → Valid OAuth Redirect URIs 에 콜백 등록
5. **Webhooks 설정** (댓글 수신에 필수)
   - Callback URL: `{APP_URL}/webhooks/facebook/`
   - Verify token: `.env`의 `FACEBOOK_WEBHOOK_VERIFY_TOKEN` 값
   - Page 객체에 `feed`, `mention`, `messages` 구독
   - Instagram 객체에 `comments`, `mentions` 구독
6. `.env`에 입력 후 **서버 재시작**
   ```bash
   PLATFORM_FACEBOOK_APP_ID=...
   PLATFORM_FACEBOOK_APP_SECRET=...
   FACEBOOK_WEBHOOK_VERIFY_TOKEN=랜덤문자열
   ```

#### ⚠️ 함정 2가지

1. **Threads는 App ID가 다르다.** 페북 App ID를 넣으면 에러 `4476002`.
   → `PLATFORM_THREADS_APP_ID` / `PLATFORM_THREADS_APP_SECRET`을 따로 설정해야 한다.
   설정 전까지 연결 페이지에 "Not Configured"로 표시된다.

2. **웹훅 미설정은 조용히 실패한다.** 계정은 정상으로 보이는데 댓글만 안 들어온다.
   README 원문에도 *"This step is easy to miss and fails silently"* 라고 적혀 있다.
   진단 명령어:
   ```bash
   python manage.py diagnose_facebook --account-id <UUID>
   python manage.py diagnose_facebook --account-id <UUID> --subscribe   # 구독 복구
   ```

### API 키 넣는 두 가지 위치

| 위치 | 범위 | 우선순위 |
|---|---|---|
| `.env` 파일 | 배포 전체 | **높음 (우선)** |
| `{APP_URL}/admin/` → Credentials → Platform credentials | 조직별 | 낮음 |

> Django admin은 **superuser만** 접근 가능하다.

---

## 6. 실제 사용 흐름

1. **아이디어 수집** — 칸반보드에 포스트잇처럼 등록, 드래그로 상태 이동
2. **글 작성 (Composer)** — 사진/영상 첨부, 여러 계정 동시 선택, 플랫폼별 문구 개별 수정, 미리보기, 첫 댓글(해시태그) 예약
3. **일정 배치 (Calendar)** — 드래그로 날짜/시간 지정
   - **반복 슬롯**: "매주 화·목 19시" 고정
   - **큐(Queue)**: 글을 큐에 넣으면 다음 빈 슬롯에 자동 배치
4. **승인 (Approvals)** — 워크스페이스 설정에서 `없음 / 선택 / 내부검토 / 내부검토+클라이언트` 선택
   - 클라이언트는 **가입·로그인 없이 매직링크(30일 유효)** 로 접속해 승인/반려
5. **자동 발행 (Publisher)** — 백그라운드 일꾼이 처리, 실패 시 자동 재시도, 90일 감사로그
6. **댓글 관리 (Inbox)** — 전 플랫폼 댓글/DM/멘션 통합, 감정분석, 담당자 배정, 스레드 답글
   ```bash
   python manage.py backfill_inbox --days 7                      # 과거 메시지 가져오기
   python manage.py backfill_inbox --platform facebook --days 30
   ```
7. **성과 분석 (Analytics)** — 조회수/참여율/팔로워 증감, 7·30·90일 추이 그래프

---

## 7. AI(MCP) 연동

이 프로젝트에는 **MCP 서버가 내장**되어 있어서 AI 에이전트가 직접 조종할 수 있다.

### 1) API 키 발급
스튜디오 → **Organization → API Keys**
- 권한 선택: `create_posts`, `publish_directly`, `upload_media`, `view_analytics`
- 워크스페이스 범위 지정, 특정 SNS 계정만 허용 가능
- 즉시 폐기(revoke) 가능

### 2) Claude Code에 등록
```bash
claude mcp add --transport http brightbean {APP_URL}/api/v1/mcp \
  --header "Authorization: Bearer bb_studio_..."
```

### 3) Claude Desktop은 API 키 없이도 가능
Settings → Connectors → Add custom connector → 서버 URL `{APP_URL}/api/v1/mcp`
→ DCR(동적 클라이언트 등록) + OAuth 로그인. **단, 공개 https 주소 필요.**

### MCP 도구 12개

| 도구 | 기능 | 필요 권한 |
|---|---|---|
| `list_accounts` | 계정 목록 | — |
| `create_draft` | 초안 작성 | `create_posts` |
| `schedule_post` | 작성 + 예약 한번에 | `create_posts` + `publish_directly` |
| `schedule_draft` | 기존 초안 예약 | `create_posts` + `publish_directly` |
| `get_post` / `list_posts` | 글 조회 | — |
| `cancel_post` | 예약 취소 → 초안 복귀 | `create_posts` |
| `search_media` / `get_media` | 미디어 검색/조회 | — |
| `upload_media` | 업로드 (base64, ≤1MB) | `upload_media` |
| `get_account_analytics` | 채널 분석 (7~90일) | `view_analytics` |
| `get_post_analytics` | 글별 분석 | `view_analytics` |

### REST API 엔드포인트

Base URL: `{APP_URL}/api/v1/` · 인증: `Authorization: Bearer bb_studio_...`
문서: `{APP_URL}/api/v1/docs`

| 메서드 | 경로 | 기능 |
|---|---|---|
| GET | `/me` | 키 범위/권한 확인 |
| GET | `/accounts` | 연결된 계정 목록 |
| POST | `/posts` | 글 생성 |
| GET/PATCH | `/posts/{id}` | 글 조회/수정 |
| POST | `/posts/{id}/schedule` | 예약 |
| POST | `/posts/{id}/cancel` | 예약 취소 |
| GET | `/analytics/accounts/{id}` | 채널 분석 |
| GET | `/analytics/posts/{id}` | 글 분석 |
| POST/GET | `/media`, `/media/{id}` | 미디어 |
| POST | `/mcp` | MCP JSON-RPC 엔드포인트 |

**Rate limit:** 쓰기 120/분, 읽기 300/분, 워크스페이스 합계 1000/분
**멱등성:** 모든 쓰기 요청에 `Idempotency-Key` 지원

---

## 8. 자주 터지는 문제

| 증상 | 원인 & 해결 |
|---|---|
| 예약했는데 글이 안 올라감 | 백그라운드 일꾼 미실행 → `python manage.py process_tasks` |
| `redirect URI mismatch` | `.env`의 `APP_URL`과 플랫폼 등록 주소 불일치. http/https, 포트, **끝 슬래시**까지 완전히 동일해야 함 |
| 화면이 깨져 보임 | Tailwind 미실행 → `cd theme/static_src && npm run start` (또는 `npm run build`) |
| `.env` 수정이 반영 안 됨 | 서버 재시작 필요 |
| 댓글이 안 들어옴 | 웹훅 설정 누락 → `python manage.py diagnose_facebook --account-id <UUID>` |
| Threads `4476002` 에러 | Threads 전용 App ID 미설정 |
| Docker postgres unhealthy | `docker compose up` 후 10~15초 대기, `docker compose logs postgres` 확인 |

---

## 9. 수익화 아이디어

### ⚖️ 먼저: AGPL-3.0 라이선스 이해

LICENSE 파일 **13조 (Remote Network Interaction)** 가 핵심:

> "프로그램을 수정해서 네트워크로 서비스하면, 그 사용자들에게 **수정된 전체 소스코드를 무료로 제공**해야 한다."

일반 오픈소스(MIT)와 결정적으로 다른 지점이다.

| ❌ 불가 | ✅ 가능 |
|---|---|
| 코드 수정 후 소스 숨기고 SaaS 판매 | **서비스 판매** (대행, 운영, 컨설팅) |
| BrightBean 이름/로고 사용 (상표는 별개) | 소스 공개하며 **호스팅 + 지원** 과금 |
| 코드 자체를 재판매 | **API로 붙는 별도 서비스** 판매 |

> 💡 **핵심: AGPL은 "소프트웨어를 파는 것"을 막지, "소프트웨어로 하는 일을 파는 것"은 막지 않는다.**
> (법률 자문은 아님. 실제 사업화 시 전문가 확인 필요.)

### 원작자의 수익모델 = 힌트

`apps/intelligence/`(4,492줄)를 뜯어본 결과:
- **Stripe 결제 + 크레딧 기반** 유료 AI 분석 애드온
- 유료 도구 6개: 콘텐츠 갭 분석, 채널 벤치마킹, 영상 후킹 점수, 썸네일/제목 점수, 니치 분석, 영상 벤치마킹
- `.env`의 5개 변수를 모두 설정해야 메뉴가 나타나고, 안 하면 순수 OSS 모드로 동작

→ **"본체는 무료 오픈소스, 부가가치는 별도 유료 SaaS"** 가 이 판의 정답.

### 아이디어 7개 (현실성 순)

#### 🥇 1위. SNS 운영 대행사
| 항목 | 내용 |
|---|---|
| 원가 | 서버비 월 1~2만원 (툴값 0원) |
| 단가 | 업체당 월 50~150만원 (시장가) |
| 시작까지 | 1~2주 |
| 라이선스 | 문제 없음 (내부 사용) |

경쟁사는 툴에 월 30~40만원을 쓴다. 그 차액이 통째로 마진.
무제한 워크스페이스라 거래처 100개를 받아도 추가비용 0원.
클라이언트 승인 포털 덕에 "사장님 컨펌" 프로세스가 자동화된다.

#### 🥈 2위. Meta 앱 심사 + 셋업 대행
- 인스타/페북 연동은 진입장벽이 매우 높다 (Use case 4개 + 권한 10개+ + 웹훅 + 심사)
- 웹훅 미설정은 조용히 실패해서 초보자가 원인을 못 찾는다
- 단가: 셋업 1회 50~200만원 + 유지보수 월 10~30만원
- **한 번 뚫어놓은 노하우가 그대로 자산**이 되고, 1위 아이디어와 세트로 판매 가능

#### 🥉 3위. 한국형 커넥터 개발
- 현재 지원 13개 중 **한국 플랫폼이 0개** (네이버 블로그, 카카오채널, 브런치 전부 없음)
- `providers/base.py` 상속 구조라 새 플랫폼 추가가 비교적 쉽다
- 원작자에게 PR로 기여하면 포트폴리오가 되고, "한국 유일" 타이틀로 영업 가능
- ⚠️ **선결 과제**: 네이버·카카오가 외부 자동 발행 API를 공식 제공하는지 반드시 먼저 확인.
  없으면 불가능하며, 크롤링/자동화 편법은 계정 정지 위험이 크다.

#### 4위. AI 자동화 파이프라인 ⭐ 추천
```
사장님이 "이번 주 신메뉴 나왔어요" 한마디
   ↓ AI가 플랫폼별 문구 생성
   ↓ 자동으로 초안 등록 + 최적 시간대 예약
   ↓ 사장님은 승인 링크에서 클릭만
```
- 이미 내장된 MCP 서버(도구 12개)로 바로 구현 가능
- 단가: 업체당 월 20~50만원, 사람 손이 거의 안 들어가 마진이 매우 높음
- **AGPL 안전**: AI 부분을 별개 서비스로 두고 API로만 통신하면 소스 공개 의무 없음
- 1위 대행사업의 원가를 크게 낮추는 무기이기도 함

#### 5위. 매니지드 호스팅
- 월 3~10만원 구독. 설치/운영을 대신해준다
- 수정했다면 소스 공개 의무 발생 (수정 없이 그대로 쓰면 원본 링크 안내로 충족)
- 경쟁 포인트는 코드가 아니라 **안정성 + 빠른 CS + 한글 지원**
- 단점: 서버 장애 대응, 고객 SNS 토큰 보관에 따른 보안 책임

#### 6위. 별도 유료 애드온 SaaS
원작자의 `intelligence` 구조를 한국판으로. 한국어 카피라이팅, 해시태그 추천, 경쟁사 분석, 최적 발행시간 예측 등. 크레딧 충전제. 별도 서버 API 제공이면 AGPL 전염 없음.

#### 7위. 교육/콘텐츠
설치 강의, Meta 앱 심사 통과 가이드 전자책, 템플릿 팩. 소액이지만 1·2위 사업의 **리드 유입 통로**로 유효.

### 추천 진행 순서

```
[2위 셋업 대행] 첫 매출 + 노하우 축적
        ↓
[1위 대행사] 안정적 월 매출
        ↓
[4위 AI 자동화] 원가 절감 + 규모 확장
        ↓
[3위 한국 커넥터] 경쟁자가 못 넘는 해자
```

### ⚠️ 리스크 체크리스트

| 리스크 | 대응 |
|---|---|
| Meta 앱 심사 탈락 | 대행 용도 명확히 소명. 심사 기간 수 주 소요 |
| 플랫폼 정책 변경 | 인스타/틱톡 API 정책이 자주 바뀜. 기능 중단 가능성 상존 |
| 고객 SNS 토큰 보관 책임 | 코드는 `EncryptedTextField`로 암호화 저장하지만, 유출 시 법적 책임은 운영자 |
| AGPL 위반 | 수정 후 서비스하면 반드시 소스 공개 |
| 상표권 | "BrightBean" 이름·로고 사용 불가. 자체 브랜드로 리브랜딩 필요 |
| 노동집약 | 대행업은 결국 사람 손 → 4위(AI 자동화)가 해법 |

> 참고: AGPL 프로젝트는 보통 **듀얼 라이선스**(유료 상용 라이선스로 소스 공개 의무 면제)를 판매한다.
> 큰 사업을 그린다면 원작자에게 직접 문의하는 것도 옵션.

---

## 10. React / PHP로 다시 만들 수 있나

### 먼저: 두 질문은 서로 다른 층이다

```
🎨 화면 (프론트엔드)  ← React
        ↕
⚙️ 로직 (백엔드)      ← PHP / Python(Django)
        ↕
🗄️ 데이터베이스
```

### React로 화면을 바꾸는 것 → 가능하지만 함정이 있다

현재는 **HTMX + Alpine.js** (서버가 HTML 조각을 보내는 방식).
React로 바꾸려면 API가 필요한데, `django-ninja` 기반 REST API가 이미 있다. **다만:**

| | 개수 |
|---|---|
| 화면 처리 로직(뷰) | **356개** |
| API로 노출된 것 | **약 13개** |

API는 AI 에이전트용으로 만든 것이라 최소한만 있다.
- ✅ 있음: 계정 목록, 글 CRUD·예약·취소, 미디어, 분석
- ❌ 없음: **댓글함, 승인 워크플로우, 달력, 팀원관리, 미디어 폴더, 클라이언트 포털**

→ 전체를 React로 갈아엎으려면 **API를 수백 개 새로 만들어야 한다.**
   (다행히 `django-ninja` 뼈대가 있어 추가 자체는 어렵지 않다.)

#### 💡 현실적인 React 활용법
```
기존 Django 화면 (유지)      ← 관리자용, 복잡한 설정
        +
새 React 앱 (신규 개발)      ← 클라이언트가 보는 화면만
```
클라이언트 승인 포털 하나만 React로 만들어도 **작업량 1/20에 체감 효과는 절반 이상**.

### PHP로 재작성 → 기술적으로 가능하지만 비추천

| | Django 그대로 | PHP 재작성 |
|---|---|---|
| 기존 코드 활용 | 100% | 0% |
| 13개 플랫폼 연동 | 완성됨 | 8,154줄 신규 작성 |
| 65개 테이블 설계 | 완성됨 | 처음부터 |
| MCP(AI 연동) 서버 | 완성됨 | PHP SDK 빈약 |
| 예상 기간 | 0일 | **8개월 ~ 2년** |
| 원작자 업데이트 | 자동 수신 | 영영 못 받음 |

**진짜 어려운 건 언어가 아니라 `providers/` 8,154줄이다.**
- 인스타는 컨테이너 생성 → 상태 폴링 → 발행의 3단계
- 토큰이 60일마다 만료되어 자동 갱신 로직 필요
- 플랫폼마다 rate limit·에러코드·웹훅 서명 방식이 전부 다름

### 🚨 중요한 함정: "다른 언어로 짜면 AGPL 피할 수 있나?"

| 방식 | AGPL 적용? |
|---|---|
| 코드를 보면서 다른 언어로 **번역/이식** | ⚠️ **적용될 가능성 높음** (2차적 저작물) |
| 코드를 안 보고 기능만 보고 **독자 개발** | ✅ 미적용 (기능·아이디어는 저작권 대상 아님) |

실무에서 "봤지만 안 보고 만들었다"를 증명하기는 매우 어렵다.
**라이선스 회피 목적의 재작성은 시간 낭비 + 법적 리스크.**

### ✅ 권장 결론

**1단계** — 백엔드는 Django 그대로 사용. Python을 몰라도 설치·운영에는 지장 없다.

**2단계** — 필요한 화면만 React로 추가 (클라이언트 포털, 리포트 대시보드, 모바일 화면).
부족한 API는 `django-ninja`로 몇 개만 추가.

**3단계** — PHP가 편하다면 재작성 대신 **옆에 붙인다.**
```
[PHP로 만든 우리 서비스]  ←→ REST API / MCP ←→  [BrightBean Studio]
  랜딩페이지, 결제,                              SNS 발행 담당
  고객관리, 리포트
```
- ✅ 익숙한 언어로 개발
- ✅ Django 코드 미수정 → **AGPL 소스 공개 의무 미발생**
- ✅ 어려운 SNS 연동은 검증된 코드가 처리

> 이것은 9장의 "4위 AI 자동화" 및 원작자의 `intelligence`와 **정확히 같은 구조**다.

### 목적별 판단표

| 목적 | 권장 |
|---|---|
| 익숙한 언어로 고치고 싶다 | 3단계 방식 (옆에 붙이기) |
| 화면이 촌스럽다 | Tailwind CSS만 수정. 재작성 불필요 |
| 라이선스를 피하고 싶다 | 재작성으로는 못 피함. 다른 전략 필요 |
| 공부 목적 | 기능 1개만 골라 따라 만들기 |
| 내 브랜드로 팔고 싶다 | 리브랜딩(로고/색/이름)은 현재 코드에서 바로 가능 |

> 👉 실제 동기가 **"익숙한 언어 / 라이선스 회피 / 내 브랜드 판매"** 였다면
> [11장](#11-동기별-최종-판단-실제-상황)에서 각각을 자세히 다룬다.

---

## 11. 동기별 최종 판단 (실제 상황)

React/PHP 재작성을 고민한 이유는 세 가지였다. **결론부터: 세 가지 모두 재작성 없이 해결된다.**

### 동기 1 — "내가 아는 언어라 고치기 편해서"

고치려는 대상이 무엇이냐에 따라 답이 갈린다.

| 고치고 싶은 것 | 필요한 기술 | 재작성 필요? |
|---|---|---|
| 화면 디자인, 색상, 로고, 문구 | **HTML + CSS만** | ❌ 불필요 |
| 새 페이지·리포트·랜딩·결제 | 익숙한 언어로 **사이드카** 개발 | ❌ 불필요 |
| 기존 로직 세부 수정 | Django 템플릿 + Python 조금 | ❌ 불필요 |
| 새 SNS 플랫폼 연동 추가 | **Python 필수** (`providers/`) | ❌ 불필요 (이 부분만 Python) |

> 💡 Django 템플릿 문법은 PHP와 상당히 비슷하다. `{{ 변수 }}`, `{% if %}...{% endif %}` 구조라서
> PHP를 아는 사람이면 **화면 수정은 반나절이면 적응**한다. 굳이 언어를 바꿀 이유가 없다.

**권장:** 백엔드는 Django 유지 + 새로 만들 것만 익숙한 언어로 사이드카.

### 동기 2 — "라이선스 피하려고"

**이 목적으로는 재작성이 효과가 없다.** 명확히 정리한다.

| 방식 | AGPL 회피 가능? | 비고 |
|---|---|---|
| 코드 보고 PHP/React로 이식 | ❌ 불가 | 2차적 저작물로 판단될 가능성 높음 |
| 코드를 전혀 안 보고 독자 개발 | ⭕ 이론상 가능 | 이미 코드를 본 사람은 사실상 증명 불가 |
| 기능만 참고해 완전히 새 설계 | ⭕ 가능 | 그러면 8개월~2년짜리 신규 개발 |

#### 먼저, AGPL에 대한 흔한 오해부터 정정

| 오해 | 실제 |
|---|---|
| "전 세계에 소스를 공개해야 한다" | ❌ **서비스 사용자에게 제공**하면 된다. GitHub 공개 의무 아님 |
| "고객 데이터·API 키도 공개해야 한다" | ❌ 소스코드만. 데이터·설정값·비밀키는 대상 아님 |
| "수정 안 해도 공개해야 한다" | ❌ 수정을 안 했으면 **공개할 것 자체가 없다.** 원본 링크 안내로 충족 |
| "오픈소스면 돈을 못 번다" | ❌ GitLab, Mattermost, Grafana 전부 오픈소스로 수천억 매출 |

> 즉 **"소스 공개"의 실제 부담은 생각보다 훨씬 작다.**
> 그대로 쓰면 공개할 것이 아예 없고, 고쳐도 고친 부분만 고객에게 주면 된다.

#### 진짜 해법 3가지

1. **AGPL을 준수하면서 서비스로 번다** (가장 현실적)
   - 대행·운영·컨설팅은 라이선스와 무관하게 자유롭게 판매 가능
   - 코드를 안 고치면 공개 의무 자체가 발생하지 않는다

2. **사이드카 구조 — 내가 만든 코드는 100% 내 것**
   ```
   [내 서비스: PHP/React]  ←→ REST API / MCP ←→  [BrightBean Studio: 무수정]
     결제, 고객관리, AI,                          SNS 발행만 담당
     랜딩, 리포트, 브랜딩
   ```
   - Studio를 **수정하지 않고** 별도 프로세스로 통신 → AGPL 전염 없음
   - 내 코드는 비공개 유지 가능, 익숙한 언어 사용 가능
   - **동기 1·2·3을 한 번에 해결하는 구조**

3. **원작자에게 상용 듀얼 라이선스 문의**
   - AGPL 프로젝트는 통상 "유료 상용 라이선스 = 소스 공개 의무 면제"를 판매한다
   - 규모 있는 사업을 그린다면 이메일 한 통으로 확인해볼 가치가 있다

> ⚠️ 법률 자문이 아니다. 실제 사업화 전 반드시 전문가 확인이 필요하다.
> 다만 **"재작성으로 라이선스를 피한다"는 전략만은 확실히 비추천**이다.

### 동기 3 — "내 브랜드로 팔려고"

**좋은 소식: 화이트라벨 기능이 이미 내장되어 있다.** 재작성이 전혀 필요 없다.

```python
apps/workspaces/models.py:26    primary_color = models.CharField(...)   # 워크스페이스별 색상
apps/organizations/models.py:10 logo_url = models.URLField(...)         # 조직별 로고
```

- 워크스페이스마다 **로고와 브랜드 색상을 다르게** 설정 가능
- 워크스페이스별 기본 해시태그, 첫 댓글, 포스팅 템플릿도 지정 가능
- 즉 **클라이언트 A에게는 A사 브랜드로, B에게는 B사 브랜드로** 보여줄 수 있다

#### 리브랜딩 작업 범위 (실측)

| 작업 | 대상 | 난이도 |
|---|---|---|
| 로고 교체 | `static/img/`, `.github/assets/` | 매우 쉬움 |
| 파비콘 교체 | `static/favicon/` (7개 파일) | 매우 쉬움 |
| 색상 변경 | `theme/static_src/tailwind.config.js`, `styles.css` | 쉬움 |
| 서비스명 문자열 교체 | `brightbean` 문자열이 포함된 **82개 파일** (대부분 문서·테스트) | 보통 (일괄 치환) |
| 도메인/메일 설정 | `.env` (`APP_URL`, `DEFAULT_FROM_EMAIL`) | 쉬움 |

> 실제 화면에 보이는 부분만 바꾸면 **반나절이면 충분**하다.

#### 🚨 리브랜딩 시 반드시 지켜야 할 3가지 (AGPL)

1. **저작권 고지는 제거하면 안 된다.** LICENSE 파일과 소스 내 저작권 표시는 유지해야 한다.
   (서비스 이름·로고를 바꾸는 것과, 원저작자 표시를 지우는 것은 완전히 다른 문제다.)
2. **코드를 수정했다면 사용자에게 소스를 제공해야 한다.** 푸터에 "소스코드" 링크 하나면 충족된다.
3. **"BrightBean" 상표는 쓰지 않는다.** 반대로 내 브랜드명으로 바꾸는 것은 자유다.

> ✅ **정리: AGPL 하에서 리브랜딩 + 유료 판매는 합법이다.**
> 조건은 "원저작자 표시 유지 + 수정 시 소스 제공"이며, 브랜드를 바꾸는 것 자체는 아무 문제가 없다.

### 세 동기에 대한 통합 결론

```
동기 1 (익숙한 언어)  ─┐
동기 2 (라이선스)     ─┼─→  [사이드카 구조] 하나로 전부 해결
동기 3 (내 브랜드)    ─┘     + 내장 화이트라벨 기능 활용
```

| | 재작성 | 사이드카 + 리브랜딩 |
|---|---|---|
| 기간 | 8개월~2년 | **1~2주** |
| 내 코드 비공개 | ✅ (단, 이식이면 AGPL 적용) | ✅ 확실히 가능 |
| 익숙한 언어 사용 | ✅ | ✅ |
| 내 브랜드 판매 | ✅ | ✅ |
| 13개 플랫폼 연동 | 직접 개발 | **이미 완성** |
| 원작자 업데이트 수신 | ❌ | ✅ |
| 라이선스 리스크 | ⚠️ 높음 | ✅ 낮음 |

**→ 재작성이 정답인 경우는 사실상 없다.**

---

## 12. 핵심 요약

| 질문 | 답 |
|---|---|
| **이게 뭔가?** | 13개 SNS를 한 곳에서 관리하는 오픈소스 웹앱. 월 15~40만원 SaaS의 무료 대체재 |
| **누가 쓰나?** | 여러 채널·여러 클라이언트를 운영하는 대행사, 팀. 개인 계정 1개면 오버스펙 |
| **얼마나 큰가?** | 파이썬 76,115줄 / 화면 240개 / 테이블 65개. 실제 제품 규모 |
| **설치 난이도?** | 앱 실행 자체는 쉬움(`make setup`). **SNS 연동이 진짜 관문** |
| **가장 흔한 실수?** | 백그라운드 일꾼(`process_tasks`) 미실행 → 예약 발행 안 됨 |
| **가장 큰 함정?** | 웹훅 미설정이 **조용히 실패**해서 댓글만 안 들어옴 |
| **돈은 어떻게 버나?** | 코드가 아니라 **서비스**를 판다. 대행 → 셋업 컨설팅 → AI 자동화 순 |
| **React/PHP 재작성?** | 재작성 말고 **옆에 붙여라.** REST API + MCP가 이미 열려 있다 |
| **라이선스?** | AGPL-3.0. 수정 후 네트워크 서비스 시 소스 공개 의무 |
| **재작성으로 라이선스 회피?** | ❌ 불가. 이식은 2차적 저작물. **사이드카 구조**가 정답 |
| **내 브랜드로 팔 수 있나?** | ✅ 화이트라벨 내장. 저작권 고지만 유지하면 합법 |

### 지금 당장 해볼 5단계

```
1. make setup                      # 설치
2. 터미널 2개 실행                  # runserver + process_tasks
3. 로그인 → 조직/워크스페이스 생성
4. Bluesky 하나만 연결              # 개발자 등록 불필요
5. 글 하나 써서 5분 뒤 예약 → 확인
```

이 5단계가 성공하면 나머지는 전부 같은 패턴이다.

---

*이 문서는 저장소 코드를 직접 분석해서 작성했다. 원문 상세 설명은 `README.md`, 설계 배경은 `development_specs/architecture.md` 참고.*
