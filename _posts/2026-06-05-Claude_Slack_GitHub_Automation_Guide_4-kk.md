# Claude Team Plan 조직 자동화 가이드
## Slack 회의록/결정사항/ADR을 Git에 자동 Publish하기

> **대상**: Claude Team Plan 사용 중인 조직 (Standard + Premium seats 혼합)  
> **용도**: Slack의 회의록, 결정사항, ADR을 Git 문서저장소에 자동 저장  
> **기준**: 2026년 6월

---

## 🎯 현재 상황

```
✅ 이미 완료됨:
├─ Claude Team Plan 구독
├─ Organization 생성
├─ GitHub 커넥터 추가 (조직 레벨)
├─ Slack 앱 설치
└─ 팀 멤버 5+ 명

🎯 지금부터 할 것:
├─ 효율적인 워크플로우 구축
├─ Slack → Git 자동화 설정
├─ 팀이 쉽게 사용할 수 있는 프롬프트
└─ 운영 규칙 수립
```

---

## 📂 Git 저장소 구조

### 문서 저장소 디렉토리 레이아웃

```
doc-repository/
├── meeting-notes/          # 회의 기록
│   ├── 2026-06/
│   │   ├── 2026-06-05-product-team.md
│   │   ├── 2026-06-05-engineering-team.md
│   │   └── 2026-06-03-all-hands.md
│   └── README.md           # 회의록 관리 가이드
│
├── decisions/              # 결정사항 (의사결정 기록)
│   ├── 2026-06/
│   │   ├── 2026-06-decision-api-v2-rollout.md
│   │   ├── 2026-06-decision-database-migration.md
│   │   └── 2026-06-decision-tech-stack-update.md
│   └── README.md           # 의사결정 프로세스
│
├── adr/                    # Architecture Decision Records
│   ├── 2026-06/
│   │   ├── 0001-microservices-architecture.md
│   │   ├── 0002-event-driven-design.md
│   │   └── 0003-database-sharding-strategy.md
│   └── README.md           # ADR 작성 가이드
│
└── templates/              # 마크다운 템플릿
    ├── meeting-notes-template.md
    ├── decision-template.md
    └── adr-template.md
```

---

## 🚀 Slack에서 바로 쓸 수 있는 명령어

### 패턴 1: 회의록 작성

#### Slack에서 입력:

```
@claude
다음은 Product Team 회의 내용이야. 
회의록으로 정리해서 doc-repository의 
meeting-notes/2026-06 폴더에 PR로 올려줄래?

회의명: Product Roadmap Planning
참석자: Alice, Bob, Charlie
주요 논의:
- Q3 신규 기능 3개 우선순위 결정
- 기술 부채 정리 계획 수립
- 마케팅팀과 협업 방향 논의
```

#### Claude의 응답:

```
✅ PR 생성 완료!

브랜치: claude/meeting-notes-2026-06-05-product-team
파일: meeting-notes/2026-06/2026-06-05-product-team.md
PR: https://github.com/my-org/doc-repository/pull/1234

내용 요약:
- 📅 날짜: 2026-06-05
- 👥 참석자: 3명
- 📝 아이템: 3개

검토 후 병합해주세요.
```

---

### 패턴 2: 의사결정 기록

#### Slack에서 입력:

```
@claude 
이거 의사결정으로 기록해줄래?

제목: "API v2로 완전 마이그레이션 결정"
결정사항: Q3부터 API v1 지원 중단, v2로 통일
근거:
- v1 버그 유지보수 비용 높음
- v2가 성능 30% 향상
- 클라이언트 90% 이미 v2 사용
영향도: 모든 백엔드 서비스, 모바일팀
액션 아이템:
1. 마이그레이션 가이드 작성 (Josh)
2. 호환성 레이어 개발 (Engineering)
3. 클라이언트 공지 (Product)

decisions 폴더에 저장해줄래?
```

#### Claude의 응답:

```
✅ PR 생성 완료!

파일: decisions/2026-06/2026-06-decision-api-v2-migration.md

구조:
- 의사결정명
- 날짜, 담당자
- 결정 내용
- 배경 및 근거
- 영향도 분석
- 액션 아이템 (체크박스)
- 검토 사항

PR#: https://github.com/my-org/doc-repository/pull/1235
```

---

### 패턴 3: ADR (Architecture Decision Record)

#### Slack에서 입력:

```
@claude
아키텍처 결정을 ADR로 작성해줄래?

제목: "이벤트 기반 아키텍처로 전환"
상황:
- 현재 Request-Response 방식으로 서비스간 강한 결합
- 실시간 동기화 필요한 기능 증가
- 확장성 한계

결정:
- 모든 도메인 이벤트를 Kafka로 발행
- 서비스는 구독하는 이벤트만 처리
- 데이터베이스는 Event Sourcing 적용

결과:
- 장점: 서비스 독립성 증대, 확장 용이
- 단점: 초기 개발 비용 증가, 운영 복잡도 증가
- 위험: 이벤트 순서 보장, 중복 처리

adr/2026-06 폴더에 0002-event-driven-architecture.md로 저장해줄래?
```

#### Claude의 응답:

```
✅ ADR 작성 완료!

파일: adr/2026-06/0002-event-driven-architecture.md

ADR 구조 (따라감):
1. 상황 (Context)
2. 결정 (Decision)
3. 결과 (Consequences)
   - 장점
   - 단점  
   - 위험 요소
4. 연관 ADR
5. 참고 자료

PR#: https://github.com/my-org/doc-repository/pull/1236

💡 팁: 다른 ADR과 링크되는 부분을 확인했습니다.
```

---

## 🛠️ Standard vs Premium Seat 역할 분담

### Standard Seat (일반 팀원)
```
사용 패턴:
- 회의록 작성 요청
- 의사결정 기록 생성
- PR 검토 및 병합

제약사항:
- 사용량 제한 있음 (충분함)
- Claude Code 고급 기능 미사용
```

### Premium Seat (전문가/리더)
```
사용 패턴:
- 회의록 검토 및 편집
- 템플릿 개선 및 스킬 작성
- 자동화 프롬프트 개발
- 복잡한 다중 문서 처리

추천 대상:
- Tech Lead / Architect
- Product Manager
- Engineering Manager
```

---

## 📝 Claude 내 Skills 작성

### Skill 1: 회의록 정리

#### 작성 방법

```
Claude.ai → Customize → Skills → Create Skill
```

#### Skill 정의

```
이름: "Meeting Notes Formatter"

지침:
당신은 회의 기록을 프로페셔널 마크다운 문서로 변환합니다.

입력:
- 회의 이름
- 참석자 목록
- 회의 내용 (자유 형식)

출력:
마크다운 형식:

---
# [회의명]

**Date**: YYYY-MM-DD  
**Attendees**: [참석자들]  
**Duration**: [소요시간]  

## 📋 Agenda

1. [주제 1]
2. [주제 2]
...

## 💬 Discussion

### [주제 1]
- 내용
- 내용

### [주제 2]
- 내용
- 내용

## ✅ Decision Items

- [ ] [액션 아이템 1] (@담당자)
- [ ] [액션 아이템 2] (@담당자)

## 📌 Next Steps

1. ...
2. ...

## 📎 Attachments

- [관련 링크]
- [참고 자료]

---

규칙:
- 마크다운 구조 유지
- 액션 아이템은 체크박스로
- 담당자를 @로 명시
- 날짜는 YYYY-MM-DD 형식
```

### Skill 2: 의사결정 기록 작성

```
이름: "Decision Record Writer"

지침:
조직의 의사결정을 구조화된 마크다운으로 기록합니다.

입력:
- 결정 제목
- 배경/컨텍스트
- 결정 내용
- 근거
- 영향도 (High/Medium/Low)

출력:
---
# Decision: [제목]

**Date**: YYYY-MM-DD  
**Decision Maker**: [직책/이름]  
**Status**: Active  
**Impact Level**: [High/Medium/Low]  

## 📌 Summary

[한 문단 요약]

## 🎯 Decision

[결정 내용]

## 🔍 Context

[배경 및 상황]

## ✔️ Rationale

- 근거 1
- 근거 2
- 근거 3

## 📊 Impact Analysis

| 영역 | 영향 | 설명 |
|------|------|------|
| Backend | High | API 변경 필요 |
| Frontend | Medium | UI 수정 |
| Ops | Low | 배포 프로세스 무변 |

## 🎬 Action Items

- [ ] [액션 1] (@담당자) - Due: YYYY-MM-DD
- [ ] [액션 2] (@담당자) - Due: YYYY-MM-DD

## 🔗 Related Decisions

- [Decision: ...]
- [ADR: ...]

---

규칙:
- Impact Level은 3단계로만 분류
- 액션 아이템은 데드라인 포함
- 관련된 다른 결정사항 링크
```

### Skill 3: ADR 템플릿

```
이름: "ADR Writer"

지침:
Architecture Decision Record를 작성합니다.
(Nygard의 ADR 형식 준수)

입력:
- ADR 번호 (자동 생성 가능)
- 제목
- 상황
- 결정
- 결과 (장점/단점/위험)

출력:
---
# ADR [번호]: [제목]

**Status**: Proposed | Accepted | Deprecated | Superseded  
**Context Date**: YYYY-MM-DD  
**Decision Maker**: [Architecture Team]  

## Context

[현재 상황 및 문제점]

## Decision

우리는 [결정]를 선택했다.

## Consequences

### Positive ✅

- 장점 1
- 장점 2
- 장점 3

### Negative ⚠️

- 단점 1
- 단점 2

### Risks & Mitigations 🚨

| 위험 | 완화 방안 |
|------|----------|
| 위험 1 | 대응 방안 |
| 위험 2 | 대응 방안 |

## Alternatives Considered

1. **[대안 1]**
   - 장점: ...
   - 단점: ...

2. **[대안 2]**
   - 장점: ...
   - 단점: ...

## Related ADRs

- ADR [N]: ...
- ADR [N]: ...

## References

- [참고 자료 1]
- [참고 자료 2]

---

규칙:
- 번호는 4자리 (0001, 0002, ...)
- Status 필드는 5가지만 사용
- 모든 결정은 근거 포함
```

---

## 💬 Slack에서 자주 쓸 프롬프트

### 프롬프트 1: 빠른 회의록

```
@claude /meeting-notes

[여기에 회의 내용 붙여넣기]
```

### 프롬프트 2: 의사결정 기록

```
@claude /decision
제목: [제목]
영향도: [High/Medium/Low]
설명: [내용]
```

### 프롬프트 3: ADR 작성

```
@claude /adr
제목: [제목]
상황: [상황 설명]
결정: [무엇을 결정했나]
결과: [어떤 결과가 있을까]
```

### 프롬프트 4: 여러 문서 한번에

```
@claude /batch
- 회의록: [회의명] [내용]
- 결정사항: [제목] [내용]
- ADR: [제목] [내용]

모두 PR로 올려줄래?
```

---

## 🔧 실제 워크플로우

### 사례 1: 주간 엔지니어링 미팅

#### 상황:
```
매주 금요일 오전 10시 - Engineering Sync
참석: 개발팀 8명
주요 목적: 기술 결정, 이슈 트래킹
```

#### 현재 (자동화 전):
```
1. 회의 중: 누군가 노트 작성 (30분 낭비)
2. 회의 후: 노트 정리 (30분 소요)
3. Slack에 공유 (수동)
4. 분산된 정보 (찾기 어려움)
```

#### 자동화 후:
```
1. 회의 중: Claude에 실시간 메시지로 내용 공유
2. 회의 직후: @claude가 회의록 정리 (자동)
3. PR 생성: doc-repository에 자동 PR
4. 팀 검토 및 병합: 5분 내 완료
5. GitHub에 자동 저장: 검색/추적 용이
```

#### 실제 Slack 채팅:

```
[회의 중]

Alice: "Q3 로드맵 업데이트"
Bob: "신규 기능 API 설계 논의"
Charlie: "데이터베이스 마이그레이션 진행상황"

[회의 말미]

Engineering Manager:
@claude 위 회의 내용을 정리해서 
meeting-notes/2026-06/friday-sync로 PR 올려줄래?
```

---

### 사례 2: 마이크로서비스 아키텍처 결정

#### 상황:
```
Tech Lead가 아키텍처 개선 제안
이를 조직의 아키텍처 문서로 기록해야 함
```

#### 프로세스:

```
1. Slack에서 Tech Lead가 개요 설명
   @claude
   "Kafka 이벤트 기반 아키텍처로 전환하는 ADR 작성해줄래?
   [상황, 결정, 결과 설명]"

2. Claude가 정식 ADR 작성
   - 마크다운 포맷 (ADR 번호 0003)
   - 모든 섹션 완성

3. PR 생성 및 리뷰
   - 아키텍처팀 검토
   - 의견 반영 (GitHub PR 댓글)
   - 병합

4. 자동 기록 완성
   - adr/2026-06/0003-event-driven-arch.md
   - 검색 가능, 버전 관리됨
```

---

## 📊 조직별 설정 예시

### Engineering Team의 설정

```
Slack 채널: #engineering
구독 저장소: doc-repository
주요 문서:
- meeting-notes/engineering/
- adr/
- decisions/tech-*

담당자: Tech Lead (Premium seat)
```

### Product Team의 설정

```
Slack 채널: #product
구독 저장소: doc-repository
주요 문서:
- meeting-notes/product/
- decisions/product-*

담당자: Product Manager (Standard seat)
```

### All Hands의 설정

```
Slack 채널: #general (또는 #all-hands)
구독 저장소: doc-repository
주요 문서:
- meeting-notes/all-hands/
- decisions/org-*
- company-updates/

담당자: CEO/COO (Premium seat)
```

---

## ✅ 체크리스트: 시작하기

### Phase 1: 준비 (1주)

- [ ] GitHub 저장소 디렉토리 구조 생성
  ```bash
  mkdir -p meeting-notes/2026-06
  mkdir -p decisions/2026-06
  mkdir -p adr/2026-06
  mkdir -p templates
  
  # 템플릿 파일 추가
  touch templates/meeting-notes-template.md
  touch templates/decision-template.md
  touch templates/adr-template.md
  ```

- [ ] README 파일 작성 (각 폴더 하위)
  ```markdown
  # Meeting Notes
  
  Slack의 회의 내용을 Claude가 자동으로 정리하여 저장합니다.
  
  ## 사용 방법
  
  Slack에서:
  @claude 회의 내용을 정리해서 저장해줄래?
  
  ## 폴더 구조
  
  YYYY-MM/ 폴더로 월별 정리
  ```

- [ ] Claude Skills 생성
  - [ ] "Meeting Notes Formatter"
  - [ ] "Decision Record Writer"
  - [ ] "ADR Writer"

- [ ] 팀 교육 자료 준비

### Phase 2: 시범 운영 (2주)

- [ ] 한 팀에서 시작 (예: Engineering)
- [ ] 주간 회의에서 테스트
- [ ] 피드백 수집
- [ ] 프롬프트 개선

### Phase 3: 전사 확대 (1주)

- [ ] 다른 팀 적용
- [ ] 표준화된 가이드라인 배포
- [ ] Premium seat 담당자 지정

### Phase 4: 운영 (지속)

- [ ] 월간 정리 (아카이브)
- [ ] 템플릿 개선
- [ ] 사용 통계 모니터링

---

## 📈 기대 효과

### 시간 절감

```
Before (자동화 전):
회의 1시간 + 기록 30분 + 정리 30분 = 2시간

After (자동화 후):
회의 1시간 + Claude 자동정리 + 검토 5분 = 1시간 5분

월간 절감: 10회의 × 55분 = 9시간 / 팀
연간 절감: 108시간 / 팀 ($5,400 가치)
```

### 정보 접근성 개선

```
Before: 메일 / Slack 메시지 산재
After: GitHub 중앙화 + 검색 가능 + 버전 관리

찾기 시간: 평균 15분 → 2분
효율성: 7배 향상
```

### 의사결정 추적

```
Before: 왜 이렇게 결정했나? (기억에 의존)
After: 결정사항 문서 + 근거 + 영향도 기록

의사결정 재논의율: 40% → 5% 감소
```

---

## 🚨 주의사항

### 1. 민감한 정보 관리

```
❌ 피할 것:
- 급여 정보
- 개인 신상정보
- 기밀 비즈니스 데이터
- 고객 민감 정보

✅ 필터링 규칙:
- 민감 정보 마스킹 (Claude가 자동 제안)
- Private 저장소 사용
- 팀 내부 문서만 Git에 저장
```

### 2. PR 검수

```
✅ 모든 Claude 생성 문서는 반드시:
1. 사람이 검토 필수
2. GitHub PR에서 댓글로 수정 요청
3. Claude가 수정 반영
4. 병합 전 한 번 더 검토
```

### 3. 데이터 일관성

```
문제: 같은 결정사항을 여러 곳에 기록
해결:
- 한 곳에만 저장 (decisions/)
- 다른 곳에서 링크로 참조
- ADR도 마찬가지
```

---

## 🎓 고급 팁

### Tip 1: ADR 자동 번호 매기기

```
현재: 수동으로 번호 입력
개선:
@claude
다음 ADR 번호를 확인해서 
"0004-[제목]"으로 작성해줄래?
→ Claude가 기존 ADR 스캔 후 자동 번호
```

### Tip 2: 월간 아카이브

```
매월 말:
@claude
2026-06의 모든 문서를 
2026-06-archive.md 로 인덱싱해줄래?

→ 검색 성능 향상
```

### Tip 3: 크로스 레퍼런싱

```
@claude
adr/0001과 decision/api-v2-migration 사이의
관계를 분석해서 링크 추가해줄래?
```

### Tip 4: 정기 검토 요청

```
@claude
Q2 결정사항 중 
실제로 실행된 것과 미루어진 것을
분류해줄래?
```

---

## 📞 문제 해결

### Q: Claude가 GitHub에 접근하지 못해요

```
확인:
1. Organization에서 GitHub 커넥터 활성화됨?
   → Organization Settings → Connectors → GitHub ✅

2. 저장소가 커넥터에 포함되어 있나?
   → 저장소명 확인

3. 개인 GitHub 계정에 쓰기 권한이 있나?
   → [저장소] → Settings → Collaborators 확인
```

### Q: PR이 만들어졌는데 형식이 이상해요

```
해결:
1. Skills 다시 확인
   → Customize → Skills → [Skill명]

2. 프롬프트에 더 명확한 지침 추가
   @claude
   "[위의 내용을 마크다운으로 변환하되]
   [섹션 제목은 ##로, 리스트는 -로]"
```

### Q: 팀원들이 프롬프트를 일관되게 쓰지 않아요

```
해결:
1. Slack 채널에 "친대"로 고정 메시지
   ```
   🤖 Claude 사용 가이드
   
   회의록: @claude /meeting-notes [내용]
   결정사항: @claude /decision 제목: [제목] ...
   ADR: @claude /adr 제목: [제목] ...
   ```

2. 주간 리마인더 (Workflow로 자동화)

3. 템플릿 Slack 북마크 공유
```

---

## 📚 추가 학습 자료

### 문서 작성 표준
- [GitHub README 작성법](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-on-github/about-readmes)
- [마크다운 가이드](https://www.markdownguide.org/)

### ADR 참고 자료
- [Nygard의 ADR 형식](https://adr.github.io/)
- [ADR Examples](https://github.com/adr/adr.github.io)

### 조직 문서화
- [The Twelve-Factor App](https://12factor.net/)
- [Google Engineering Practices](https://google.github.io/eng-practices/)

---

## 🎉 다음 단계

### Week 1-2: 기본 설정
```
✓ 디렉토리 구조
✓ Skills 생성
✓ 한 팀 파일럿
```

### Week 3-4: 피드백 & 개선
```
✓ 프롬프트 개선
✓ 템플릿 개선
✓ 운영 가이드 수립
```

### Week 5+: 전사 확대
```
✓ 모든 팀에 적용
✓ 자동화 확대 (GitHub Actions 등)
✓ 분석 및 최적화
```

---

**적용 조직**: Claude Team Plan (Standard + Premium seats)  
**마지막 수정**: 2026년 6월 5일  
**소유**: 귀 조직

