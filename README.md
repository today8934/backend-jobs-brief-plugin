# Backend Jobs Brief Plugin

한국 IT 백엔드/서버/인프라(SRE·DevOps·Platform·Cloud) 직군의 주요 회사 채용 동향과 신규 공고를 Tavily·WebSearch·WebFetch로 병렬 수집해 한국어 마크다운 브리핑을 생성하는 Claude Code plugin입니다.

## 무엇을 하나

- **빅테크 + 미드 + 주요 스타트업 ~30개** 화이트리스트 회사를 정기적으로 훑고
- **시장 트렌드 + 회사별 신규 공고 + 핵심 인사이트**를 한 화면에 합성
- **주간 (default)** + **월간** 듀얼 모드 — 사용자 발화로 자동 전환
- vault에 누적 저장 — 추후 분기/연간 digest 확장 가능

## 트리거 발화

다음과 같은 표현으로 자동 발동됩니다:

- "백엔드 채용 동향 정리해줘"
- "이번 주 한국 IT 채용 소식"
- "월간 채용 정리"
- "네카라쿠배 채용 어떻게 돼?"
- "DevOps 채용 좀 봐줘"
- "백엔드 이직 시장 어때?"
- "어떤 회사가 백엔드 뽑고 있어?"

## 출력 위치

- **디폴트**: `~/workspace/backend-jobs-brief/{weekly|monthly}/<filename>.md`
- **사용자 설정**: `~/.claude/data/backend-jobs-brief/config.json`의 `output_dir` 필드로 오버라이드

```json
{
  "output_dir": "~/Documents/obsidian/backend-jobs"
}
```

## 의존성

- **Tavily MCP** (필수) — `claude mcp add tavily <api-key>`. 미설치 시 preflight halt + 안내 메시지.
- WebSearch / WebFetch (Claude Code 기본 도구)

## 구성

```
backend-jobs-brief-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── backend-jobs-brief/
        ├── SKILL.md                     # 메인 워크플로우 (Step 0~5)
        └── references/
            ├── companies.md             # 화이트리스트 30개 (Tier-1/2/3)
            ├── sources.md               # 4채널 수집 쿼리 템플릿
            └── output-template.md       # frontmatter + 본문 템플릿
```

## 화이트리스트 동적 조정

ad-hoc 호출:
- "Tier-3 빼고" → 스타트업 10개 제외
- "[회사명] 추가" → 그 회차 한정 추가
- "게임사 포함" → 넥슨·엔씨·크래프톤·펄어비스·스마일게이트 임시 추가
- "외국계 포함" → 구글/MS/AWS/메타 코리아 임시 추가
- "빅테크만" → Tier-1만

## 추후 (v0.2.0+)

- monthly digest skill (overnight-weekly-digest 패턴) — 4주치 weekly 파일 합성
- Tier-2 일부 회사 WebFetch 풀 승격
- 외국계·게임사 별도 화이트리스트
- 자동 cron 등록 (매주 월요일 오전)

## 라이선스

MIT (see [LICENSE](./LICENSE)).
