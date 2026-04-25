# 수집 소스 + 쿼리 템플릿

Step 2의 4채널 병렬 수집을 가이드합니다. **모든 호출은 한 메시지 안에서 병렬**로 발사하세요. 순차 호출은 wall-clock을 6~10배 늘려 사용자 경험을 망칩니다.

총 호출 수: ~35 (channel A 10 + B 15 + C 5 + D 5).

## 채널 A: Tier-1 회사 채용 페이지 직접 fetch

- **도구**: `WebFetch`
- **호출 수**: 10 (Tier-1 회사 entity 수와 일치)
- **파라미터**:
  - `url`: `references/companies.md`의 Tier-1 URL (10개)
  - `prompt`: 아래 공통 prompt

### 공통 prompt

```
Extract recent job postings (within {N} days, where N=7 for weekly mode, N=30 for monthly mode) for these roles:
- backend / server developer
- SRE (Site Reliability Engineer)
- DevOps engineer
- platform engineer
- cloud engineer

For each posting, extract:
- title (직무명)
- level (junior/mid/senior or 신입/주니어/미들/시니어)
- tech stack keywords (Java, Spring, Kotlin, Python, Go, Node.js, AWS, K8s, Kafka 등)
- deadline if visible
- direct apply URL if visible

Korean text is OK. Return as a structured list.
```

### Fallback 체인 (HTTP 403 / timeout / blank response)

1. `mcp__tavily__tavily_extract`로 같은 URL 시도
2. 그래도 실패하면 채널 B의 검색 결과로 그 회사를 보강
3. "데이터 품질 & 수집 도구" 섹션에 fetch 실패 회사 명시

## 채널 B: 회사별 검색 (Tier-2/3 + Tier-1 보강)

- **도구**: `mcp__tavily__tavily_search`
- **호출 수**: 15 (Tier-2 10 + Tier-3 우선 5)
- **파라미터**:
  - `query`: `references/companies.md`의 검색 키워드 (회사별)
  - `days`: weekly→7, monthly→30
  - `max_results`: 5
  - `topic`: `general`

### 예시

```python
# Tier-2 회사들 (10개)
mcp__tavily__tavily_search(query="무신사 백엔드 개발자 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="야놀자 서버 개발자 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="컬리 백엔드 개발자 채용 2026", days=7, max_results=5)
# ... (Tier-2 나머지)

# Tier-3 우선순위 5개 (전체 30 회사 합산이 너무 많아지지 않게 5개만)
mcp__tavily__tavily_search(query="라이너 백엔드 개발자 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="센드버드 백엔드 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="몰로코 백엔드 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="메가존클라우드 SRE DevOps 채용 2026", days=7, max_results=5)
mcp__tavily__tavily_search(query="하이퍼커넥트 서버 개발자 채용 2026", days=7, max_results=5)
```

남은 Tier-3 5개는 채널 D의 채용 사이트 검색에서 자연스럽게 잡힙니다.

## 채널 C: 트렌드 / 뉴스

- **도구**: `mcp__tavily__tavily_search` × 3 + `WebSearch` × 2
- **호출 수**: 5
- **목적**: 시장 분위기·기술 스택 변화·산업 뉴스 — 회사별 공고와 별개로 "왜 이 시장이 이렇게 움직이는가"를 설명할 재료

### 예시

```python
# Tavily search (3개)
mcp__tavily__tavily_search(query="한국 IT 백엔드 채용 동향 2026", days=7, max_results=8)
mcp__tavily__tavily_search(query="개발자 채용 시장 트렌드 2026 한국", days=7, max_results=8)
mcp__tavily__tavily_search(query="네이버 카카오 토스 채용 동향", days=7, max_results=6)

# WebSearch (2개)
WebSearch(query="한국 백엔드 채용 2026 4월")
WebSearch(query="개발자 이직 시장 한국 2026")
```

월간 모드에서는 `days=30`으로 늘리고 쿼리에 "2026 4월" 같은 명시 표현을 추가하세요.

## 채널 D: 채용 사이트 사이트 검색

- **도구**: `mcp__tavily__tavily_search` (`site:` 연산자 포함)
- **호출 수**: 5
- **목적**: 점핏·원티드·사람인·프로그래머스·잡플래닛의 누적 채용 데이터 — Tier-2/3 누락 보완

### 예시

```python
mcp__tavily__tavily_search(query="site:jumpit.saramin.co.kr 백엔드 개발자", days=7, max_results=10)
mcp__tavily__tavily_search(query="site:wanted.co.kr 백엔드 시니어", days=7, max_results=10)
mcp__tavily__tavily_search(query="site:saramin.co.kr 서버 개발자 백엔드", days=7, max_results=10)
mcp__tavily__tavily_search(query="site:career.programmers.co.kr 채용 백엔드", days=7, max_results=10)
mcp__tavily__tavily_search(query="site:jobplanet.co.kr 백엔드 개발자 채용", days=7, max_results=10)
```

## 결과 통합 규칙

1. **URL 정규화**: 쿼리 파라미터 제거 (`?utm_source=...`), 끝 슬래시 통일, http→https
2. **제목 유사도 dedup**: 같은 회사·같은 직무명·같은 기술 스택 조합이면 1개로 합치고, 출처는 가장 권위 있는 것 (회사 공식 > 점핏/원티드 > 뉴스) 우선
3. **회사 매칭**: Tier-2/3 회사명을 search 결과 제목/본문에서 substring 매칭. 매칭 실패 시 "기타" 그룹으로 분류 (본문 회사별 표에 별도 행으로 추가, 회사명 컬럼에 회사명 그대로 표기)
4. **신선도 필터**: monthly 모드에서도 60일 이상 된 공고는 제외 (Tavily `days=30`이 기본 필터지만 일부 결과는 더 오래된 게 섞임)

## 토큰 예산 가이드

| 채널 | input | output |
|---|---|---|
| A | ~10K | ~5K |
| B | ~15K | ~7K |
| C | ~10K | ~5K |
| D | ~12K | ~6K |
| **합계** | **~50K** | **~25K** |

합성 단계에서 ~10K output 마크다운 본문이 추가됩니다.

수집 결과가 너무 많으면 (e.g., 단일 회사에서 30+ 공고) 본문 표에는 시니어/주니어 균형 잡힌 10개 내외만 남기고 나머지는 "데이터 품질 & 수집 도구" 섹션에서 합산만 보고하세요. 본문 표가 너무 길면 사용자가 안 읽습니다.

## 재시도 정책

- 채널 A 단일 호출 timeout: 1회 fallback (Tavily extract) 후 검색 결과로 보강
- 채널 B/C/D 결과 < 5건: 광범위 키워드 (`백엔드 채용 2026 한국`)로 1회 재시도
- 전체 호출 중 30% 이상 실패: 즉시 중단 + 사용자에게 보고 ("Tavily 응답 불안정, 잠시 후 다시 시도해주세요")
