---
name: backend-jobs-brief
description: 한국 IT 백엔드/서버/인프라(SRE·DevOps·Platform·Cloud) 직군의 주요 회사(네카라쿠배당토 + 무신사·야놀자·컬리·두나무·직방 + Series B↑ 핫스타트업 ~30개) 채용 동향과 신규 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 생성합니다. 사용자가 "백엔드 채용", "서버 개발자 채용", "한국 IT 채용 동향", "이번 주 채용 소식", "주간 채용 브리핑", "월간 채용 정리", "네카라쿠배 채용", "백엔드 이직 시장", "DevOps 채용", "SRE 채용", "Platform Engineer 채용", "개발자 채용 뉴스", "한국 백엔드 채용 트렌드", "어떤 회사가 백엔드 뽑고 있어", "이번 달 채용 어떤 식이야" 같은 직·간접 표현을 쓸 때 반드시 이 skill을 실행하세요. 단순 단일 공고 조회가 아니라 시장 트렌드 + 회사별 신규 공고 + 핵심 인사이트를 엮은 종합 브리핑이 필요한 모든 상황에서 트리거합니다.
---

# Backend Jobs Brief

한국 IT 백엔드/서버/인프라 직군의 채용 동향과 주요 회사 신규 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 만듭니다.

## 왜 이 skill이 필요한가

한국 백엔드 개발자가 시장 흐름을 파악하려면 보통 점핏·원티드·사람인을 따로 보고, 회사별 채용 페이지를 별도로 확인하고, 트렌드 뉴스도 별도로 검색해야 합니다. 5~6개 검색을 병렬도 아니라 순차로 하면 30분이 훌쩍 갑니다. 이 skill은 ~30개 화이트리스트 회사를 한 번에 훑고, 트렌드·공고·인사이트를 한 화면에 엮어 3~5분 안에 본인이 시장에서 어디쯤 있는지 파악하게 합니다.

매주 호출하면서 vault에 누적되면, 같은 plugin 안의 monthly digest skill (추후 v0.2)이 4주치를 합성해 큰 그림을 보여줄 수 있게 설계되어 있습니다.

## 참조 문서 (Progressive disclosure)

자세한 내용은 필요할 때만 `Read`로 로드하세요. 메인 SKILL.md는 워크플로우 뼈대만 담고, 자주 갱신되는 화이트리스트·쿼리·템플릿은 reference 파일로 빼두었습니다.

- `references/companies.md` — 회사 화이트리스트 ~30개 (티어별, Tier-1 채용 페이지 URL 포함). Step 1에서 Read.
- `references/sources.md` — 4채널 수집 쿼리 템플릿 (Tavily / WebSearch / WebFetch). Step 1에서 Read.
- `references/output-template.md` — frontmatter + 본문 마크다운 템플릿. Step 4에서 Read.

## 출력 위치

기본 경로: `~/workspace/backend-jobs-brief/{weekly|monthly}/<filename>.md` (오늘 한국 날짜 기준).

### 경로 결정 절차
1. `~/.claude/data/backend-jobs-brief/config.json`이 있고 `output_dir` 필드가 존재하면 그 값을 우선 사용 (예: `~/Documents/obsidian/backend-jobs`)
2. 없으면 기본 경로 사용
3. 디렉토리가 없으면 먼저 `mkdir -p`로 생성
4. 같은 주차/월 파일이 이미 존재하면 덮어쓰지 않고 `<filename>-HHMM.md` 형태로 시간 suffix

### config.json 예시
```json
{
  "output_dir": "~/Documents/obsidian/backend-jobs"
}
```

사용자가 "출력 경로 바꿔줘" / "저장 위치를 X로" 같이 요청하면 이 파일을 Write해 설정.

## 실행 순서

### Step 0. Preflight

`ToolSearch` 도구로 Tavily MCP 핵심 도구 2개를 로드 시도:

```
ToolSearch(query="select:mcp__tavily__tavily_search,mcp__tavily__tavily_extract", max_results=2)
```

- 성공 (2개 도구 스키마 반환) → Step 1로 진행
- 실패 → 아래 메시지 출력 후 **즉시 halt** (메인 워크플로우 진입 금지):

  > Tavily MCP가 등록되지 않아 Backend Jobs Brief를 실행할 수 없습니다.
  > 아래 명령으로 먼저 Tavily MCP를 설치해 주세요:
  >
  > `claude mcp add tavily <Tavily API key, 자세한 방법은 Tavily 공식 문서 참조>`
  >
  > 설치 후 세션을 재시작하고 다시 실행해 주세요.

### Step 1. 컨텍스트 결정

1. 오늘 한국 날짜·시각(KST) 확인 (예: `2026-04-25 Sat 10:30`)
2. **모드 결정** — 사용자 발화 키워드 매칭:

| 키워드 | 모드 | 기간 | 출력 파일명 |
|---|---|---|---|
| "월간", "이번 달", "지난 달", "monthly", "한 달", "30일" | `monthly` | 직전 30일 | `YYYY-MM-backend-jobs-brief.md` |
| 그 외 | `weekly` (default) | 직전 7일 | `YYYY-Www-backend-jobs-brief.md` (ISO 주차, 월요일 기준) |

모호한 발화(예: "채용 동향")면 default `weekly`를 채택하고, frontmatter `mode: weekly`와 TL;DR 첫 줄에 모드를 명시해 사용자가 모드를 알 수 있게 하세요.

3. 출력 디렉토리 결정/생성 (위 "경로 결정 절차" 참조) + 파일 충돌 시 시간 suffix
4. `references/companies.md` Read → 화이트리스트 로드
5. `references/sources.md` Read → 쿼리 템플릿 로드

### Step 2. 병렬 수집 (한 메시지에서 동시 호출)

`references/sources.md`의 4채널 쿼리 템플릿을 그대로 사용해 **반드시 한 메시지 안에서 병렬**로 호출하세요. 순차 호출하면 wall-clock이 6~10배 늘어나 사용자 경험이 망가집니다.

| 채널 | 도구 | 호출 수 | 목적 |
|---|---|---|---|
| A | `WebFetch` | 10 | Tier-1 회사 채용 페이지 직접 fetch — 가장 정확한 정보 |
| B | `mcp__tavily__tavily_search` | 15 | 회사명 × 채용 키워드 — Tier-2/3 + Tier-1 보강 |
| C | `mcp__tavily__tavily_search` + `WebSearch` | 5 | 트렌드/뉴스 — 시장 흐름 파악 |
| D | `mcp__tavily__tavily_search` (`site:` 연산자) | 5 | 점핏·원티드·사람인·프로그래머스·잡플래닛 사이트 검색 |

총 ~35회. 모드별 `days` 파라미터: `weekly`→7, `monthly`→30.

### Step 3. 합성

1. URL 정규화 + 제목 유사도로 중복 제거 (`references/sources.md`의 "결과 통합 규칙" 참조)
2. 결과를 카테고리로 분류:
   - **회사별 신규 공고** (A·B 결과)
   - **시장 트렌드** (C 결과)
   - **기술 스택/연봉 시그널** (B+C 교차)
   - **핵심 이벤트** (회사 인력 확장·축소·신설팀 발표)
3. **출처 번호 normalize**: 본문 등장 순서대로 `[[1]]..[[N]]` 재할당, 본문에 인용되지 않은 URL은 sources에서 제외. gap 없는 연속 번호여야 함 (`[[6]], [[8]], [[12]]` 누락 금지).
4. YAML frontmatter `sources` 배열에 `[{id, url, title}, ...]` 기록 — 추후 monthly digest skill이 이 배열을 파싱해 합성합니다.

### Step 4. 마크다운 작성·저장

`references/output-template.md` Read → 그 구조 그대로 작성. 본문 길이 가이드:
- weekly: ~3,000자 (TL;DR 3~5줄, 회사별 공고 표 ~20행, 트렌드 200~400자, 인사이트 200~300자)
- monthly: ~5,000자 (회사별 공고 표 ~40행, 트렌드 500~800자, 인사이트 500자, 추가 "주차별 변화" 섹션)

저장 후 `git add` 같은 추가 동작은 하지 마세요 (vault는 별도 git 추적이 아님).

### Step 5. 사용자 보고

저장 경로 + TL;DR(3~5줄)만 짧게 보고하세요. 본문 전체를 채팅에 다시 붙여넣지 마세요. 사용자는 vault 파일을 직접 열어 보는 패턴이라, 채팅창에 본문 반복은 노이즈입니다.

## 구조 준수 사항 (사용자 메모리)

`feedback_report_structure.md` 메모리에 명시된 사용자 선호:
- **TL;DR을 본문 최상단에** 배치
- **"데이터 품질 & 수집 도구" 섹션을 본문 최말단에** 배치 — 메타 정보가 의사결정 흐름을 깨므로 말미로 보내고, TL;DR이 가장 먼저 눈에 들어와야 시간이 절약됩니다
- 인과 주장에는 inline `[[n]](url)` 출처 링크 — 사용자가 출처를 직접 추적할 수 있도록

## 에러 처리

| 상황 | 대응 |
|---|---|
| WebFetch 차단/timeout (HTTP 403/timeout) | `mcp__tavily__tavily_extract`로 같은 URL fallback → 그래도 실패 시 채널 B 검색 결과로 그 회사를 보강. "데이터 품질" 섹션에 fetch 실패 회사 명시 |
| Tavily/WebSearch 결과 < 5건 | 광범위 키워드(`백엔드 채용 2026 한국`)로 1회 재시도 |
| 모드 모호 발화 | default `weekly`, frontmatter `mode: weekly` 명시, TL;DR 첫 줄에 모드 표기 |
| 회사 채용 페이지 404 | 데이터 품질 섹션에 명시, 본문 회사별 표에서 해당 회사 행 생략 |
| 신규 공고 0건 (전체) | TL;DR에 "이번 주 신규 공고 없음 — 진행 중 공고 위주" 명시 후 진행 중 공고로 채움 |
| 화이트리스트 회사 정보 0건 | "데이터 품질" 섹션에 "{회사명} 정보 없음" 명시, 본문 회사별 표에서 해당 회사 행 생략 |

## 화이트리스트 동적 조정

사용자가 ad-hoc으로 호출할 때:

| 발화 | 동작 |
|---|---|
| "Tier-3 빼고" / "스타트업 빼고" | Tier-3 10개 제외 |
| "[회사명] 추가" | 그 회차 한정으로 화이트리스트에 추가 |
| "게임사 포함" | 넥슨·엔씨·크래프톤·펄어비스·스마일게이트 임시 추가 |
| "외국계 포함" | 구글/MS/AWS/메타 코리아 임시 추가 |
| "빅테크만" | Tier-1만 사용 |

이런 임시 조정은 `references/companies.md`를 수정하지 말고, 그 회차의 Step 2 수집 단계에서만 처리하세요. 영구 변경이 필요하면 사용자에게 명시 확인 후 reference 파일 업데이트.
