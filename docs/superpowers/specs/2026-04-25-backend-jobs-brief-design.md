# Backend Jobs Brief — Design Spec

- **Created**: 2026-04-25
- **Owner**: 류욱상 (wooksang.ryu@musinsa.com)
- **Status**: Approved (brainstorming complete, ready for implementation)
- **Plugin**: `backend-jobs-brief-plugin`
- **Marketplace**: `wooksang-marketplace` (today8934/wooksang-marketplace)
- **Source repo (planned)**: `today8934/backend-jobs-brief-plugin`

---

## 1. 목적

한국 IT 백엔드/서버/인프라(SRE·DevOps·Platform·Cloud) 직군 채용 동향과 주요 회사(빅테크·미드·주요 스타트업 ~30개) 채용 공고를 한국어 마크다운 브리핑으로 합성한다. 주간(default) + 월간 듀얼 모드. vault에 누적 저장하여 추후 분기/연간 digest 확장 여지 확보.

## 2. 결정 요약 (브레인스토밍 결과)

| 항목 | 결정 | 근거 |
|---|---|---|
| 카테고리 | 정보 수집·분석 → 한국어 리포트 | 사용자의 기존 skill 패턴 (`overnight-market-report`, `arsenal-brief`)과 일치 |
| 시장 범위 | 한국 중심 | 사용자 위치 (한국, 무신사 도메인), 한국 백엔드 채용 시장 모니터링 |
| 결과물 무게 | 트렌드 + 공고 혼합 (TL;DR → 트렌드 → 회사별 공고 → 인사이트) | 기존 `overnight-market-report` 톤·구조와 일관 |
| 직무 범위 | 백엔드 + 인프라/플랫폼 (Backend, SRE, DevOps, Platform Engineer, Cloud Engineer) | 한국 채용 시장에서 자주 묶이고, 백엔드 출신 자연스러운 이동 경로 |
| 회사 set | 빅테크 10 + 미드 10 + 주요 스타트업 10 = 30개 (3 tier) | 시장 시그널의 80% 커버, 사용자 회사(무신사) 포함 |
| 호출 빈도 | 주간(default) + 월간 듀얼 모드 | 채용 시장 시간 상수, 기존 `overnight-weekly-digest` 패턴 재사용 |
| 패키징 | plugin (별도 git repo) | 사용자의 기존 plugin 컨벤션 |
| 저장 | vault에 마크다운 + YAML frontmatter | 추후 monthly digest를 위한 누적 |
| 자동화 | cron 없음 (수동 호출) | 첫 N주 결과 품질 안정화 후 승격 |
| 구현 깊이 | 균형형 (B) — 상위 10개 회사 직접 fetch + 검색 | token 비용/품질 균형, A↔C 슬라이딩 가능 |
| Marketplace 등록 | wooksang-marketplace `.claude-plugin/marketplace.json`에 entry 추가 | 사용자의 standing 컨벤션 |

## 3. 파일 레이아웃

### 3.1 Plugin repo (`today8934/backend-jobs-brief-plugin`)

```
backend-jobs-brief-plugin/
├── README.md
├── LICENSE                                  # MIT (overnight·arsenal과 동일)
├── .claude-plugin/
│   └── plugin.json                          # name·version·description·author·keywords
├── skills/
│   └── backend-jobs-brief/
│       ├── SKILL.md                         # frontmatter + 워크플로우
│       └── references/
│           ├── companies.md                 # 화이트리스트 ~30개 (티어별, 채용 페이지 URL 포함)
│           ├── sources.md                   # Tavily/WebSearch/WebFetch 쿼리 템플릿
│           └── output-template.md           # frontmatter + 본문 마크다운 템플릿
└── docs/superpowers/specs/
    └── 2026-04-25-backend-jobs-brief-design.md   # 본 문서
```

### 3.2 vault (plugin 외부, 사용자 환경)

```
~/workspace/backend-jobs-brief/              # 디폴트 누적 vault
├── weekly/
│   └── 2026-W17-backend-jobs-brief.md      # ISO 주차 (월요일 기준)
└── monthly/
    └── 2026-04-backend-jobs-brief.md
```

**경로 결정 절차** (overnight 패턴):
1. `~/.claude/data/backend-jobs-brief/config.json`이 있고 `output_dir` 필드가 존재하면 그 값을 우선 사용
2. 없으면 `~/workspace/backend-jobs-brief/{weekly|monthly}/` 사용
3. 디렉토리가 없으면 `mkdir -p`
4. 같은 주차/월 파일이 이미 존재하면 덮어쓰지 않고 시간 suffix 추가 (`...-HHMM.md`)

### 3.3 Marketplace 등록 위치

`/Users/wooksangryu/workspace/wooksang-marketplace/.claude-plugin/marketplace.json`의 `plugins` 배열 말미에 추가:

```json
{
  "name": "backend-jobs-brief-plugin",
  "source": {
    "source": "github",
    "repo": "today8934/backend-jobs-brief-plugin"
  },
  "description": "한국 IT 백엔드/인프라/플랫폼 직군 채용 동향과 주요 IT 회사(빅테크·미드·주요 스타트업 ~30개) 채용 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 생성합니다. 주간(default) + 월간 듀얼 모드, vault 누적 저장으로 추후 분기 digest 확장 가능.",
  "version": "0.1.0"
}
```

## 4. plugin.json (manifest)

기존 `overnight-market-report-plugin`/`arsenal-brief-plugin`의 양식 그대로:

```json
{
  "name": "backend-jobs-brief-plugin",
  "version": "0.1.0",
  "description": "한국 IT 백엔드/서버/인프라(SRE·DevOps·Platform·Cloud) 직군의 채용 동향과 주요 IT 회사 채용 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 생성합니다. 빅테크·미드·주요 스타트업 ~30개 화이트리스트 + 주간(default) + 월간 듀얼 모드 + vault 누적 저장.",
  "author": {
    "name": "류욱상",
    "email": "wooksang.ryu@musinsa.com",
    "url": "https://github.com/today8934"
  },
  "license": "MIT",
  "keywords": [
    "korea",
    "backend",
    "server-developer",
    "devops",
    "sre",
    "platform-engineer",
    "jobs",
    "recruiting",
    "korean",
    "briefing",
    "tavily",
    "weekly-digest"
  ]
}
```

## 5. SKILL.md frontmatter (트리거 description)

```yaml
---
name: backend-jobs-brief
description: 한국 IT 백엔드/서버/인프라(SRE·DevOps·Platform·Cloud) 직군의 주요 회사(네카라쿠배당토 + 무신사·야놀자·컬리·두나무·직방 + Series B↑ 핫스타트업 ~30개) 채용 동향과 신규 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 생성합니다. 사용자가 "백엔드 채용", "서버 개발자 채용", "한국 IT 채용 동향", "이번 주 채용 소식", "주간 채용 브리핑", "월간 채용 정리", "네카라쿠배 채용", "백엔드 이직 시장", "DevOps 채용", "SRE 채용", "Platform Engineer 채용", "개발자 채용 뉴스", "한국 백엔드 채용 트렌드", "어떤 회사가 백엔드 뽑고 있어", "이번 달 채용 어떤 식이야" 같은 직·간접 표현을 쓸 때 반드시 이 skill을 실행하세요. 단순 단일 공고 조회가 아니라 시장 트렌드 + 회사별 신규 공고 + 핵심 인사이트를 엮은 종합 브리핑이 필요한 모든 상황에서 트리거합니다.
---
```

## 6. 워크플로우 (SKILL.md 본문)

### Step 0. Preflight

```
ToolSearch(query="select:mcp__tavily__tavily_search,mcp__tavily__tavily_extract", max_results=2)
```
- 성공 (2개 도구 스키마 반환) → Step 1
- 실패 → halt + 설치 안내 메시지 출력 (overnight `Step 0` 패턴 동일)

### Step 1. 컨텍스트 결정

1. 오늘 한국 날짜·시각 확인 (KST)
2. **모드 결정** (사용자 발화 키워드 매칭):

| 키워드 | 모드 | 기간 | 출력 파일명 |
|---|---|---|---|
| "월간", "이번 달", "지난 달", "monthly", "한 달", "30일" | `monthly` | 직전 30일 | `YYYY-MM-backend-jobs-brief.md` |
| 그 외 | `weekly` (default) | 직전 7일 | `YYYY-Www-backend-jobs-brief.md` (ISO 주차, 월요일 기준) |

3. 출력 디렉토리 결정 (3.2 절차) + 기존 파일 충돌 시 시간 suffix
4. `references/companies.md` Read → 화이트리스트 로드
5. `references/sources.md` Read → 쿼리 템플릿 로드

### Step 2. 병렬 수집 (한 메시지에서 동시 호출, 4채널)

| 채널 | 도구 | 호출 수 | 파라미터 |
|---|---|---|---|
| **A. Tier-1 회사 채용 페이지** | `WebFetch` | 10 | `references/companies.md`의 Tier-1 URL × 10 / prompt: "Extract recent job postings (within {days}일) for backend/server/SRE/DevOps/Platform engineers. List title, level, tech stack, deadline if any." |
| **B. 회사별 검색 (Tier-2/3 + Tier-1 보강)** | `mcp__tavily__tavily_search` | 15 | `query="{회사명} 백엔드 개발자 채용 2026"` (또는 "서버 개발자 채용", "SRE 채용") / `days=7` (weekly) or `30` (monthly) / `max_results=5` |
| **C. 트렌드/뉴스** | `mcp__tavily__tavily_search` + `WebSearch` | 5 | `"한국 IT 백엔드 채용 동향 2026 {월}"`, `"개발자 채용 시장 트렌드 2026"`, `"네이버 카카오 토스 채용 동향"` |
| **D. 채용 사이트 사이트 검색** | `mcp__tavily__tavily_search` | 5 | `"site:jumpit.saramin.co.kr 백엔드"`, `"site:wanted.co.kr 백엔드 시니어"`, `"site:saramin.co.kr 서버 개발자"`, `"site:programmers.co.kr 채용 백엔드"`, `"site:jobplanet.co.kr 백엔드 개발자"` |

**총 호출**: ~35 (모두 한 메시지 내 병렬 발사, 순차 호출 금지).

### Step 3. 합성

1. 결과 통합 → URL 정규화 + 제목 유사도로 중복 제거
2. 결과를 5개 카테고리로 분류:
   - 회사별 신규 공고 (A + B에서 추출)
   - 시장 트렌드 (C에서 추출)
   - 기술 스택/연봉 시그널 (B + C 교차)
   - 핵심 이벤트 (회사별 인력 확장·축소·신설팀 발표 등)
   - 메타 (사용 도구·누락·한계)
3. 출처 번호 normalize (overnight 패턴): 등장 순서대로 `[[1]]..[[N]]` 재할당, 본문 미사용 URL은 sources에서 제외
4. YAML frontmatter `sources` 배열에 `[{id, url, title}, ...]` 기록

### Step 4. 마크다운 작성·저장

`references/output-template.md` 구조 그대로 작성하고 vault 경로에 저장. 본문 길이 가이드:
- weekly: ~3,000자 (TL;DR 3~5줄, 회사별 공고 표 ~20행, 트렌드 200~400자, 인사이트 200~300자)
- monthly: ~5,000자 (회사별 공고 표 ~40행, 트렌드 500~800자, 인사이트 500자, 추가 "주차별 변화" 섹션)

### Step 5. 사용자 보고

저장 경로 + TL;DR만 짧게 보고. 본문 전체를 채팅에 다시 붙여넣지 않음 (overnight 패턴).

## 7. 출력 템플릿 (`references/output-template.md`)

```markdown
---
title: "한국 백엔드 채용 브리핑 — 2026-W17 (주간)"
date_kst: 2026-04-25
period_start: 2026-04-19
period_end: 2026-04-25
mode: weekly                              # weekly | monthly
job_scope: [backend, devops, sre, platform, cloud]
companies_covered:
  bigtech: [naver, kakao, line, coupang, woowahan, daangn, toss_group, kakaobank, kakaopay, hyundai_autoever]
  mid:     [musinsa, yanolja, kurly, dunamu, zigbang, 11st, ssg, 29cm, socar, linenext]
  startup: [liner, sendbird, moloco, megazone, tosslab, hyperconnect, buzzvil, vbridge, classum, idus]
new_postings_count: 0                     # 합성 후 채움
sources:
  - { id: 1, url: "...", title: "..." }
---

# 한국 백엔드 채용 브리핑 — 2026-W17

## TL;DR
- 시장 한 줄 요약
- 두드러진 회사 변화 한 줄
- 기술 스택/트렌드 한 줄

## 시장 트렌드 (W17)
- 채용 규모 변화 (전주 대비 / 전월 대비)
- 두드러진 기술 스택 변화
- 신입/경력 비중 변화
- 도메인별 흐름 (핀테크·커머스·콘텐츠·O2O)

## 회사별 신규 공고
| 회사 | 직무 | 경력 | 기술 스택 | 비고 | 출처 |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | [[n]] |

## 핵심 인사이트
- 시그널 1
- 시그널 2
- 시그널 3 (커리어 함의 포함)

---

## 데이터 품질 & 수집 도구
- **사용 도구**: Tavily (`tavily_search` × N), WebSearch × M, WebFetch × K
- **직접 fetch 성공**: ?개 회사 채용 페이지
- **누락/접근 실패**: ?개 — 사유 명시
- **한계**: ...
- **다음 호출 권장**: ...

## 면책
- 본 브리핑은 공개 채용 정보 합성본이며, 실제 지원 여부 결정은 각 회사 공식 채널 확인 후 진행하세요.
- 비공개 채용·내부 추천 채널은 포함되지 않습니다.
```

**구조 준수 사항** (사용자 메모리 `feedback_report_structure.md`):
- TL;DR 최상단
- "데이터 품질 & 수집 도구" 섹션 최말단
- 인과 주장에는 inline `[[n]](url)` 출처 링크

## 8. 회사 화이트리스트 (`references/companies.md`)

### Tier-1 빅테크 (10) — WebFetch 우선

| 회사 | 채용 페이지 | 메모 |
|---|---|---|
| 네이버 (Naver) | https://recruit.navercorp.com/ | |
| 카카오 (Kakao) | https://careers.kakao.com/ | |
| 라인플러스 (LINE Plus) | https://careers.linecorp.com/ko | |
| 쿠팡 (Coupang) | https://www.coupang.jobs/kr/ | |
| 우아한형제들 (배민) | https://career.woowahan.com/ | |
| 당근마켓 (Daangn) | https://team.daangn.com/jobs/ | |
| 토스 그룹 (비바리퍼블리카·토스증권·토스뱅크) | https://toss.im/career/jobs | 통합 페이지 — 1회 fetch로 3개사 커버 |
| 카카오뱅크 (KakaoBank) | https://kakaobank.career.greetinghr.com/ | |
| 카카오페이 (KakaoPay) | https://careers.kakaopay.com/ | |
| 현대오토에버 (Hyundai AutoEver) | https://www.hyundai-autoever.com/recruit/ | 자동차 IT 백엔드 채용 활발 |

### Tier-2 미드 (10) — Tavily 검색 only (v0.1.0)

무신사, 야놀자, 컬리(마켓컬리), 두나무(업비트), 직방, 11번가, SSG.COM, 29CM, 쏘카(Socar), 라인넥스트(LINE NEXT)

> v0.1.0에서 Tier-2는 WebFetch 없이 Tavily `tavily_search` (회사명 + "백엔드 개발자 채용") 으로만 수집한다. 추후 v0.2.0+에서 fetch 성공률·정보 품질 보고 Tier-2 일부를 WebFetch 풀로 승격 검토.

### Tier-3 스타트업 (10) — Tavily 검색 only

라이너(Liner), 센드버드(Sendbird), 몰로코(Moloco), 메가존클라우드(MegazoneCloud), 토스랩(Tosslab/잔디), 하이퍼커넥트(Hyperconnect), Buzzvil, 비브리지(VBridge), 클라썸(Classum), 아이디어스(idus)

> 사용자가 ad-hoc으로 "Tier-3 빼고", "[회사명] 추가/제거" 같이 호출하면 그 회차에 한해 화이트리스트 동적 조정 (SKILL.md Step 1 보강).

## 9. 에러 처리 / 엣지 케이스

| 상황 | 대응 |
|---|---|
| Tavily MCP 미로드 | preflight halt + 설치 안내 메시지 (overnight 패턴) |
| WebFetch 차단/timeout (HTTP 403/timeout) | `mcp__tavily__tavily_extract`로 fallback → 그래도 실패 시 검색 결과로 보강 |
| 검색 결과 < 5건 | 광범위 키워드 (`백엔드 채용 2026 한국`) 1회 재시도 |
| 모드 모호 발화 (예: "채용 동향") | default `weekly` 채택, frontmatter `mode: weekly` 명시, TL;DR에 모드 표기 |
| 회사 채용 페이지 404 | `references/companies.md`에 `last_failed_at` 메모 (선택) → 사용자에게 보고 섹션에 명시 |
| 화이트리스트 회사 정보 0건 | "데이터 품질" 섹션에 "{회사명} 정보 없음" 명시, 본문 회사별 표에서 해당 회사 행 생략 |
| 신규 공고 0건 (전체) | TL;DR에 "이번 주 신규 공고 없음 — 진행 중 공고 위주" 명시 후 진행 중 공고로 채움 |
| 같은 주차/월 파일 충돌 | 시간 suffix 추가 (`...-HHMM.md`), 덮어쓰기 금지 |
| Tavily 빈 응답 | WebSearch fallback 1회 |

## 10. 검증 체크리스트 (자체 검토 + 첫 실행 검증)

- [ ] frontmatter YAML 파싱 가능 (`mode`, `period_start`, `period_end`, `companies_covered`, `sources`, `new_postings_count`)
- [ ] TL;DR 3~5줄, 최상단 위치 (메모리 준수)
- [ ] 데이터 품질 섹션 최말단 (메모리 준수)
- [ ] 모든 인과 주장에 `[[n]](url)` 출처 인라인
- [ ] 회사별 공고 테이블 6컬럼 (회사·직무·경력·기술스택·비고·출처)
- [ ] vault 저장 경로 정상 (`~/workspace/backend-jobs-brief/{weekly|monthly}/...`)
- [ ] 트리거 발화 5~10개로 SKILL 자동 발동 (수동 호출 외)
- [ ] preflight Tavily 미로드 시 즉시 halt
- [ ] weekly→monthly 모드 전환이 발화로 정상 작동
- [ ] sources 배열 normalize 결과 gap 없는 1..N 연속 번호
- [ ] 화이트리스트 회사 30곳 모두 Step 2에서 시도 (직접 fetch 또는 검색)

## 11. 배포 흐름

| 단계 | 액션 | 사용자 confirm |
|---|---|---|
| 1 | 로컬 `backend-jobs-brief-plugin/` 디렉토리 + 파일 작성 | ❌ |
| 2 | 본 spec 문서 작성 (`docs/superpowers/specs/...`) | ❌ |
| 3 | skill-creator skill로 SKILL.md + references/* 생성 | ❌ |
| 4 | 로컬 git init + 초기 commit | ✅ |
| 5 | github repo 생성 (today8934/backend-jobs-brief-plugin) | ✅ |
| 6 | push origin main | ✅ |
| 7 | wooksang-marketplace의 `.claude-plugin/marketplace.json` 수정 | ✅ |
| 8 | wooksang-marketplace commit + push | ✅ |
| 9 | 로컬 plugin 등록·세션 재시작 후 첫 실행 검증 | 사용자 직접 |

## 12. 미해결 / 추후 (Out of scope for v0.1.0)

- **자동 cron 등록** — 첫 N주 결과 품질 안정화 후 별도 PR로 추가 (`/schedule` 또는 OMC cron skill)
- **분기/연간 digest** — `overnight-weekly-digest`처럼 별 skill로 추가. weekly/monthly vault를 입력으로 받아 합성
- **외국계 한국 지사 (구글/MS/AWS 코리아)** — 채용 패턴이 글로벌 기준이라 별도 화이트리스트로 분리 가능
- **게임사 (넥슨/엔씨/크래프톤)** — 도메인 특이성 강해 default에서 제외, 사용자 발화 ("게임사 포함") 시 ad-hoc 추가
- **연봉 정보 수집** — 한국 채용 공고에 연봉 명시 적음. 잡플래닛·블라인드 같은 별도 소스 필요. v0.1.0에서는 "연봉 비공개" 표기만
- **이메일/Slack 발송** — vault 저장만, 외부 발송 없음

## 13. 성공 기준 (Acceptance criteria)

본 plugin v0.1.0은 다음을 만족하면 ship.

1. 사용자가 "백엔드 채용 동향 정리해줘"라고 발화하면 SKILL이 자동 트리거된다
2. 주간 모드 호출 시 ~3,000자 본문 + frontmatter가 vault `weekly/`에 저장된다
3. 월간 모드 발화 시 자동으로 `monthly/`에 저장된다
4. 회사별 공고 표에 최소 10행 이상 (수집 결과가 그만큼 있는 경우)
5. preflight Tavily 미로드 시 즉시 halt + 설치 안내 출력
6. wooksang-marketplace에 entry 등록되어 다른 환경에서도 plugin 설치 가능
