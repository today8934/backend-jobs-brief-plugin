# 화이트리스트 회사 ~30개

3 tier로 분류. **Tier-1**은 WebFetch로 직접 채용 페이지 수집, **Tier-2·3**은 Tavily 검색만 사용 (v0.1.0). 추후 v0.2.0+에서 Tier-2 일부를 WebFetch 풀로 승격 검토.

## Tier-1 빅테크 (10) — WebFetch 우선

각 회사의 채용 페이지를 직접 fetch해 신규 공고를 수집합니다. 페이지 차단/timeout 시 `mcp__tavily__tavily_extract`로 fallback.

| # | 회사 | 채용 페이지 URL | 메모 |
|---|---|---|---|
| 1 | 네이버 (Naver) | https://recruit.navercorp.com/ | |
| 2 | 카카오 (Kakao) | https://careers.kakao.com/ | 카카오 본사 |
| 3 | 라인플러스 (LINE Plus) | https://careers.linecorp.com/ko | 한국 법인 |
| 4 | 쿠팡 (Coupang) | https://www.coupang.jobs/kr/ | |
| 5 | 우아한형제들 (배민) | https://career.woowahan.com/ | |
| 6 | 당근마켓 (Daangn) | https://team.daangn.com/jobs/ | |
| 7 | 토스 그룹 (비바리퍼블리카·토스증권·토스뱅크) | https://toss.im/career/jobs | 통합 페이지 — 1회 fetch로 3개사 커버 |
| 8 | 카카오뱅크 (KakaoBank) | https://kakaobank.career.greetinghr.com/ | |
| 9 | 카카오페이 (KakaoPay) | https://careers.kakaopay.com/ | |
| 10 | 현대오토에버 (Hyundai AutoEver) | https://www.hyundai-autoever.com/recruit/ | 자동차 IT 백엔드 채용 활발 |

### WebFetch prompt 템플릿 (Tier-1 공통)

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

## Tier-2 미드 (10) — Tavily 검색 only

`mcp__tavily__tavily_search`만 사용. 회사명 + 채용 키워드 조합으로 검색.

| # | 회사 | 검색 키워드 (예시) |
|---|---|---|
| 11 | 무신사 (Musinsa) | "무신사 백엔드 개발자 채용" |
| 12 | 야놀자 (Yanolja) | "야놀자 서버 개발자 채용" |
| 13 | 컬리 / 마켓컬리 (Kurly) | "컬리 백엔드 개발자 채용" |
| 14 | 두나무 / 업비트 (Dunamu) | "두나무 백엔드 채용" |
| 15 | 직방 (Zigbang) | "직방 서버 개발자 채용" |
| 16 | 11번가 (11st) | "11번가 백엔드 개발자 채용" |
| 17 | SSG.COM (SSG) | "SSG 백엔드 개발자 채용" |
| 18 | 29CM | "29CM 백엔드 개발자 채용" |
| 19 | 쏘카 (Socar) | "쏘카 서버 개발자 채용" |
| 20 | 라인넥스트 (LINE NEXT) | "라인넥스트 백엔드 채용" |

## Tier-3 스타트업 (10) — Tavily 검색 only

Series B 이상 또는 시장 영향력 있는 한국·한국계 스타트업.

| # | 회사 | 검색 키워드 (예시) |
|---|---|---|
| 21 | 라이너 (Liner) | "라이너 백엔드 개발자 채용" |
| 22 | 센드버드 (Sendbird) | "센드버드 백엔드 채용" |
| 23 | 몰로코 (Moloco) | "몰로코 백엔드 채용" |
| 24 | 메가존클라우드 (MegazoneCloud) | "메가존클라우드 SRE DevOps 채용" |
| 25 | 토스랩 / 잔디 (Tosslab) | "토스랩 백엔드 채용" |
| 26 | 하이퍼커넥트 (Hyperconnect) | "하이퍼커넥트 서버 개발자 채용" |
| 27 | Buzzvil | "Buzzvil 백엔드 개발자 채용" |
| 28 | 비브리지 (VBridge) | "비브리지 백엔드 채용" |
| 29 | 클라썸 (Classum) | "클라썸 백엔드 개발자 채용" |
| 30 | 아이디어스 (idus) | "아이디어스 백엔드 개발자 채용" |

## 동적 조정 (사용자 발화)

사용자가 다음과 같이 호출하면 그 회차의 화이트리스트를 임시 조정:

| 발화 | 동작 |
|---|---|
| "Tier-3 빼고" / "스타트업 빼고" | Tier-3 10개 제외 |
| "[회사명] 추가" | 그 회차 한정으로 화이트리스트에 추가 |
| "게임사 포함" | 넥슨·엔씨소프트·크래프톤·펄어비스·스마일게이트 임시 추가 |
| "외국계 포함" | 구글 코리아·MS 코리아·AWS 코리아·메타 코리아 임시 추가 |
| "빅테크만" | Tier-1만 사용 |

이런 임시 조정은 본 파일을 수정하지 마세요. 그 회차의 Step 2 수집 단계에서만 처리하면 됩니다.

## 화이트리스트 갱신 정책

회사 도산·이름 변경·채용 정책 큰 변동이 있을 때만 본 파일 업데이트. 정기 갱신은 분기 1회 정도. 사용자가 "[회사 X]가 없어졌어요" / "[회사 Y] 추가하면 좋겠어요" 같이 보고하면 한 줄 메모 후 다음 갱신 시 반영.
