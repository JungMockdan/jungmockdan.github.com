# Slack에서 Claude로 Git/PR 자동화하기: 완전한 설정 가이드 (2026년 6월)

## 📋 개요

이 가이드는 Slack에서 Claude를 사용하여 회의 내용, 결정사항 등을 자동으로 문서 저장소에 업로드하는 완전한 설정 방법을 설명합니다. **무료 버전에서도 가능**합니다 (2026년 2월 Slack 정책 업데이트).

---

## 🎯 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                    Slack Workspace                      │
├─────────────────────────────────────────────────────────┤
│  Claude Bot (@claude)                                   │
│     - 회의 내용 작성                                     │
│     - 결정사항 정리                                      │
│     - GitHub에 파일/PR 생성                              │
│     - PR 링크를 스레드에 응답                            │
└──────────────────────┬──────────────────────────────────┘
                       │
              Claude ← GitHub 커넥터
                       │
┌──────────────────────▼──────────────────────────────────┐
│                  GitHub Organization                   │
├─────────────────────────────────────────────────────────┤
│  • Document Repository (문서 저장소)                    │
│    ├─ meeting-notes/                                   │
│    ├─ decisions/                                        │
│    └─ (구조화된 디렉토리)                               │
│                                                          │
│  • Other Repositories (필요시)                          │
│    ├─ main codebase                                     │
│    └─ (PR/Issue 추적)                                   │
└─────────────────────────────────────────────────────────┘
```

> **참고**: Claude 봇만으로 Slack에서 GitHub로 파일 생성·PR 생성이 모두 가능합니다.
> 별도의 GitHub Slack 앱은 필요하지 않습니다.

---

## 4단계 설정 프로세스

### **1단계: Claude Team Plan 설정 (조직 관리자)**

#### 1-1. 조직 생성 및 Team Plan 구독

**비용**: $25/명/월 (Standard seat) 또는 $20/명/월 (연간)

```
조건:
- 최소 5명 이상
- 최대 150명까지 가능 (이상은 Enterprise)
```

**설정 경로**:
```
claude.ai → Settings → Organizations → Create Organization
→ Upgrade to Team Plan
```

**포함 기능**:
- ✅ Slack, GitHub 등 커넥터 사용
- ✅ Claude Code 접근
- ✅ Cowork (협업 기능)
- ✅ 200k context window
- ✅ 조직 수준 커넥터 관리

#### 1-2. Organization 레벨에서 GitHub 커넥터 추가

⚠️ **매우 중요**: Team/Enterprise 조직의 **Owner만** 이 단계를 수행할 수 있습니다.

**설정 경로**:
```
Organization Settings 
  → Connectors 
  → Add Connector
  → GitHub (기본 제공)
```

**절차**:
1. GitHub 커넥터 선택
2. OAuth 인증 (GitHub 계정으로)
3. 권한 부여:
   ```
   - Repository read access
   - Workflow read access (Actions)
   - Pull Request read/write
   - Issue read/write
   - Commit access
   ```
4. **중요**: "All repositories" 또는 특정 저장소 선택
   - 권장: 문서 저장소만 선택 (보안)

**설정 후**:
```
Organization 모든 멤버가 GitHub 커넥터 사용 가능
(개인이 추가로 설정 필요 X, 이미 활성화됨)
```

---

### **2단계: 개인 커넥터 설정 (각 팀 멤버)**

#### 2-1. Claude에 GitHub 개인 계정 연동

**설정 경로**:
```
Claude.ai (Web/Desktop)
  → Customize (좌측 하단)
  → Connectors
  → GitHub (Organization이 이미 추가한 것)
  → Connect
```

**인증 흐름**:
1. GitHub 로그인 화면 이동
2. Claude(Anthropic) GitHub 커넥터 권한 승인
3. 개인이 접근할 수 있는 저장소 자동 표시

**확인사항**:
```
✅ 개인이 접근 권한 있는 저장소만 표시됨
✅ Organization이 추가한 저장소 + 개인 저장소
✅ Private repository도 자동 포함
```

#### 2-2. Slack 커넥터 연동 (Optional but Recommended)

**설정 경로**:
```
Claude.ai
  → Customize
  → Connectors
  → Slack
  → Connect
```

**권한**:
```
- Read public channels
- Read private channels (가입한 것만)
- Read direct messages
- Search messages
- View files
```

**용도**:
- Slack에서 과거 회의 내용 검색
- 채널 컨텍스트 자동 추가
- 결정사항 추적

---

### **3단계: GitHub 설정 (조직 Owner)**

#### 3-1. Slack 앱을 저장소에 설치

**절차**:

1. **GitHub 조직 설정 접근**
   ```
   GitHub Organization 
     → Settings 
     → Integrations 
     → Applications
   ```

2. **Slack 앱 설치** (GitHub Marketplace에서)
   ```
   - "GitHub" Slack app 찾기
   - "Install" 클릭
   - Slack 작업공간 선택
   - 저장소 선택:
     ☑️ 문서 저장소만 선택 (보안 best practice)
     또는
     ☑️ "All repositories" (전체)
   ```

3. **GitHub에서 생성하는 것 아님** ⚠️
   ```
   Slack Marketplace → GitHub 공식 앱 설치
   (github.com에서 직접 설치 불가)
   ```

4. **권한 확인**:
   ```
   ✅ Read-only 권한 (기본값)
   ✅ Webhook 이벤트: PR, Issues, Commits
   ✅ Push events to default branch
   ```

---

### **4단계: Slack 설정 (Workspace Admin)**

#### 4-1. Claude 앱 설치

**설정 경로**:
```
Slack Marketplace
  → 검색: "Claude"
  → "Claude" 공식 앱 찾기
  → "Add to Slack"
```

**설치 후 자동 추가**:
```
❌ 모든 채널에 자동 추가되지 않음 (수동 초대 필요)
```

**초대 방법**:
```
채널에서 입력:
/invite @claude

또는

slack://app?team={workspace_id}&id={app_id}
```

#### 4-2. 연동 확인

```
채널에서 입력:
@claude 안녕! 너는 무엇을 할 수 있어?

→ Claude가 응답하면 설치 완료
```

> **GitHub Slack 앱은 설치할 필요가 없습니다.**
> Claude 봇이 GitHub 커넥터를 통해 직접 파일을 만들고 PR을 생성합니다.
> (PR 생성 시 알림이 별도 봇으로 필요한 경우에만 GitHub Slack 앱을 선택적으로 추가)

---

## 🔧 실제 사용 예시

### **예시 1: 회의 내용을 GitHub에 자동 저장**

#### Slack에서:
```
@claude 어제 제품팀 회의 내용을 정리하고 
decision: 신규 기능 Q3에 개발 시작
을 GitHub의 doc-repo/decisions 폴더에 저장해줘
```

#### Claude의 응답 흐름:
1. ✅ Slack 채널에서 회의 컨텍스트 읽음
2. ✅ 마크다운 형식으로 정리
3. ✅ GitHub 문서 저장소에 푸시
   ```
   branch: main
   path: decisions/2026-06-decision-q3-features.md
   commit message: "Decision: Q3 feature planning (from Slack)"
   ```
4. ✅ Slack에 완료 알림

#### 결과 파일:
```markdown
# Q3 신규 기능 개발 결정

**Date**: 2026-06-05  
**Channel**: #product-team  
**Participants**: [자동 추출]

## 결정사항
신규 기능을 Q3에 개발 시작한다.

## 근거
- 고객 요청 증가
- 경쟁사 대응 필요

## 액션 아이템
- [ ] 기술 스팩 정의 (PM 담당)
- [ ] 팀 리소스 배분 (팀장 담당)
```

---

### **예시 2: PR 리뷰 자동화**

#### Slack에서:
```
PR URL이 있는 스레드에서 @claude를 언급하면:

@claude https://github.com/my-org/main-repo/pull/123 
이 PR을 검토해주고 문제점을 정리해줘

→ Claude가 GitHub 커넥터로 PR 내용을 읽고
→ 코드 리뷰 의견 생성
→ Slack 스레드에 응답
```

---

### **예시 3: PR 자동 생성 (Advanced)**

#### Slack에서:
```
@claude 
제목: "Docs: Update API documentation"
내용: 
- REST API v2 엔드포인트 정의
- 인증 방식 설명
을 문서 저장소에 추가하는 PR 만들어줘
```

#### Claude의 동작:
1. ✅ 마크다운 파일 생성
2. ✅ Feature branch 생성:
   ```
   branch: claude/docs-api-update-{timestamp}
   ```
3. ✅ 커밋:
   ```
   commit: "docs: Update API documentation (from Slack)"
   ```
4. ✅ PR 생성:
   ```
   - Title: "Docs: Update API documentation"
   - Body: Slack 컨텍스트 포함
   - Assignee: 요청자 (자동)
   ```
5. ✅ Slack에 PR 링크 응답

---

## 🔐 보안 및 권한 관리

### **권한 설계 원칙**

#### Organization Level (조직):
```
Owner (1-2명)
├─ Organization 커넥터 관리
├─ 팀 멤버 추가/제거
└─ 청구 관리

Standard Member (나머지)
├─ 개인 커넥터 설정
├─ GitHub 개인 저장소 접근
└─ 조직 저장소는 GitHub 권한 따름
```

#### GitHub Level:
```
Organization Owner
├─ Slack 앱 설치 권한
└─ 저장소 webhook 관리

Repository Collaborator
├─ 읽기: 모든 공개 저장소
├─ 쓰기: 명시적으로 권한 부여된 저장소만
└─ 관리: 저장소 Owner만
```

### **Best Practices**

```yaml
✅ DO:
- GitHub 커넥터: 문서 저장소만 선택 (조직 레벨)
- 제한된 브랜치 보호: main 브랜치에 PR 필수
- 정기적 권한 감사: 월 1회
- Claude가 만든 PR은 사람이 검토 후 병합

❌ DON'T:
- "All repositories" 선택 → 실수로 민감한 코드 노출
- Personal access token 공유 → 보안 위험
- 무제한 커밋 권한 → 실수 방지
- PR 검토 없이 자동 병합
```

### **GitHub 커넥터 권한 최소화**

```
요청하는 권한    사용 목적               최소 권한
─────────────────────────────────────────────
Contents       파일 읽기/쓰기         Read & Write
Pull Requests  PR 생성/업데이트       Read & Write
Issues         Issue 생성/업데이트    Read & Write
Workflows      Actions 실행 상태 확인 Read-only
Commit Status  CI/CD 상태 확인       Read-only
```

---

## 🔍 배포별 상세 단계

### **시나리오 A: 무료 플랜 사용 (1-5명 팀)**

```
1. Claude 무료 버전 사용
   - 커넥터 1개만 가능 → GitHub 선택
   - Slack에 Claude 앱 설치 후 @claude 호출

2. 수동으로 Slack에서 Claude 호출
   @claude {작업 요청}

3. Claude가 GitHub에 파일/PR 생성
   (GitHub 커넥터 통해 직접)
```

**설정 시간**: ~30분
**비용**: $0

---

### **시나리오 B: Claude Team Plan (5-50명 팀)** ← 권장

```
1. Claude Team Plan 구독
   - $25/명/월 (Standard)

2. Organization 수준 GitHub 커넥터 추가
   - Owner만 설정 (1회)

3. 모든 멤버가 개인 커넥터 설정
   - 각자 GitHub 계정 연동

4. Slack에 Claude 앱 설치
   - @claude로 호출 및 PR 생성
```

**설정 시간**: ~1시간
**비용**: $125/월 (5명 기준)
**이점**: 조직 전체 일관성, 권한 중앙화

---

### **시나리오 C: Enterprise (50명+)**

```
1. Claude Enterprise Plan
   - 무제한 커넥터, 고급 SSO/SAML

2. GitHub Enterprise Server (선택)
   - Self-hosted GitHub 지원

3. 부서별 커스텀 플러그인
   - 각 팀 특화 자동화

4. 권한 정책 엔진
   - Slack 채널별 접근 제어
```

---

## 🛠️ 문제 해결 (Troubleshooting)

### **Q1: Claude가 GitHub에 접근할 수 없어요**

```bash
# 확인 사항:
1. ✅ Organization에서 GitHub 커넥터 추가되었나?
2. ✅ 개인이 커넥터 연동했나?
   Settings → Customize → Connectors → GitHub (연동 상태)
3. ✅ GitHub에서 Claude의 App 권한이 있나?
   GitHub → Settings → Applications → Authorized Apps
4. ✅ 해당 저장소에 접근 권한이 있나?
   GitHub → Organization → Teams → [당신의팀] → Repositories
```

**해결**:
```bash
# 1. 다시 권한 허용
Settings → Customize → Connectors → GitHub → Reconnect

# 2. 60초 대기 후 재시도
(토큰 갱신 필요)

# 3. 여전히 안 되면 OAuth 완전 재설정
Settings → Applications → Revoke
→ 다시 Connect
```

---

### **Q2: Slack에서 @claude가 응답하지 않아요**

```bash
# 확인 사항:
1. ✅ Slack에 Claude 앱이 설치되었나?
   Slack → Apps → Claude 확인

2. ✅ 채널에 Claude가 초대되었나?
   /invite @claude

3. ✅ Claude 계정이 Slack과 연동되었나?
   Claude → Customize → Connectors → Slack
```

**해결**:
```bash
# 1. 채널에 명시적 초대
/invite @claude

# 2. Claude 앱 재설치 (필요시)
Slack Marketplace → Claude → Reinstall

# 3. 다시 멘션
@claude 안녕!
```

---

### **Q3: "Insufficient permissions" 에러**

```
원인:
1. Claude의 GitHub 커넥터가 해당 저장소에 대한 쓰기 권한 없음
2. 개인 GitHub 계정이 저장소의 collaborator 아님

해결:

Step 1: GitHub에서 확인
Repository → Settings → Access
→ Collaborators & teams
→ 본인 계정 확인

Step 2: 권한 부여
Owner: + Add people
→ 계정명 입력
→ "Write" 권한 부여

Step 3: Claude 권한 갱신
Claude → Settings → Customize → Connectors
→ GitHub → Disconnect → Reconnect
```

---

## 📊 비용 계산 예시

### **5명 팀 (권장 구성)**

```
Claude Team Plan (Standard)
├─ 5명 × $25/월 = $125/월
│  (또는 5명 × $20/월 = $100/월 연간)
│
├─ 포함 내용:
│  ✅ Claude 무제한 사용
│  ✅ GitHub/Slack 커넥터
│  ✅ Claude Code
│  ✅ 200k context window
│
└─ 추가 비용: $0
   (GitHub, Slack 무료 버전 사용 가능)

────────────────────────
총 월간 비용: $100-125
인당 비용: $20-25
────────────────────────
```

### **20명 팀 (혼합 구성)**

```
Claude Team Plan
├─ Standard seats (15명): 15 × $25 = $375/월
├─ Premium seats (5명): 5 × $125 = $625/월
│  (Power user: 높은 사용량 필요한 경우)
│
└─ 총합: $1,000/월

GitHub 추가 비용
├─ GitHub Team: $4/명/월 × 20 = $80/월
│  (프라이빗 저장소 > 3개 필요 시)
│
└─ Slack Pro: $10.50/명/월 × 20 = $210/월
   (선택사항, 추가 기능 필요 시)

────────────────────────
Claude만: $1,000/월
전체 (GitHub+Slack): $1,290/월
────────────────────────
```

---

## 📚 추가 학습 자료

### **공식 문서**
- [Claude Team Plan - 공식](https://support.claude.com/en/articles/9266767-what-is-the-team-plan)
- [GitHub Integration](https://support.claude.com/en/articles/10167454-use-the-github-integration)
- [Claude Slack 통합](https://www.eesel.ai/blog/claude-cowork-slack-integration)

### **고급 설정**
- GitHub MCP Server: `https://github.com/github/github-mcp-server`
- Custom Connectors: Anthropic 공식 문서 "Get started with custom connectors using remote MCP"

### **자동화 예시**
- Claude Code in Slack: `https://code.claude.com/docs/en/slack`
- PR 자동 리뷰: Slack 스레드에서 `@claude review this PR`

---

## ✅ 체크리스트

### **조직 설정 (Owner)**
- [ ] Claude Team Plan 구독 완료
- [ ] Organization 생성
- [ ] GitHub 커넥터 추가 (Organization 레벨)
- [ ] 팀 멤버 추가 (최소 5명)

### **개인 설정 (각 멤버)**
- [ ] Claude 계정에 GitHub 연동 (OAuth)
- [ ] Claude 계정에 Slack 연동 (선택)
- [ ] 대상 저장소 읽기/쓰기 권한 확인

### **Slack 설정 (Admin)**
- [ ] Claude 앱 설치
- [ ] 채널에 Claude 초대
- [ ] @claude 응답 확인

### **GitHub 설정 (조직 Owner)**
- [ ] GitHub 커넥터에 문서 저장소 포함 확인
- [ ] 브랜치 보호 규칙 설정 (선택)
- [ ] 권한 검토

### **테스트**
- [ ] Slack에서 `@claude` 메시지 가능
- [ ] Claude가 GitHub 저장소 읽기 가능
- [ ] Claude가 GitHub에 파일 작성/PR 생성 가능

---

## 🎓 다음 단계

1. **자동화 스킬 개발**
   - Claude Skills로 회의 내용 자동 정리 스크립트
   - Slack 이벤트 트리거 커스터마이징

2. **고급 워크플로우**
   - GitHub Actions + Claude로 코드 리뷰 자동화
   - Slack threads → PR 자동 생성

3. **기업 규모 확장**
   - Enterprise Plan으로 SSO/SAML 설정
   - 부서별 커스텀 플러그인 개발

---

**마지막 업데이트**: 2026년 6월 5일  
**작성 기준**: Claude Haiku 4.5, Slack 2026 정책, GitHub API v3

