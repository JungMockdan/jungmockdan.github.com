# 실전 설정 가이드: 단계별 커맨드 & 경로

## 📝 Table of Contents
1. [조직 설정](#1-조직-설정)
2. [GitHub 커넥터 추가 (Organization)](#2-github-커넥터-추가-organization)
3. [개인 커넥터 연동 (각 멤버)](#3-개인-커넥터-연동-각-멤버)
4. [Slack에 Claude 앱 설치](#4-slack-설정)
5. [통합 테스트](#5-통합-테스트)

---

## 1. 조직 설정

### 1-1. Claude Team Plan 생성

#### 웹에서 설정

```
URL: https://claude.ai/upgrade
       ↓
"Team Plan 선택"
       ↓
조직명 입력 (예: "acme-corp")
       ↓
빌링 정보 입력
       ↓
최소 5명 초대
```

#### 결과 확인
```
✅ Settings → Organizations 에서 조직 확인
✅ 조직 Owner로 자동 설정됨
```

---

### 1-2. Team Plan 멤버 관리

#### Organization Settings 접근
```
https://claude.ai → 좌측 "Settings"
                 → "Organizations"
                 → [조직명]
                 → "Members"
```

#### 멤버 추가
```
"Add members" 버튼
  ↓
이메일 입력 (예: dev1@company.com)
  ↓
Seat type 선택:
  □ Standard (추천): $25/월
  □ Premium: $125/월 (고용량 필요시)
  ↓
"Send Invite"
```

#### 역할 설정
```
Organization Settings → "Roles"

Owner (기본 설정됨)
├─ 모든 권한 (커넥터, 멤버, 청구)
│
Standard Member
├─ 개인 커넥터만 설정 가능
├─ 조직 커넥터 사용만 가능 (추가 설정 X)
```

---

## 2. GitHub 커넥터 추가 (Organization)

### ⚠️ 매우 중요
**Organization Owner만 이 단계를 수행할 수 있습니다. Standard Member는 불가능합니다.**

### 2-1. Organization 커넥터 페이지 접근

```
https://claude.ai 
  → Settings 
  → Organizations 
  → [조직명]
  → "Connectors"
```

또는 직접 URL:
```
https://claude.ai/settings/organizations/[org-id]/connectors
```

### 2-2. GitHub 커넥터 추가

#### Step 1: 커넥터 추가 인터페이스
```
"Add connector" 버튼
  ↓
"GitHub" 선택 (기본 제공)
  ↓
"Add" 클릭
```

#### Step 2: GitHub OAuth 인증

```
GitHub 로그인 페이지로 이동
  ↓
GitHub 계정 로그인
  ↓
"Authorize Anthropic" 클릭
```

#### Step 3: 저장소 선택

**선택 옵션 1: 모든 저장소 (권장: 시작 단계)**
```
"All repositories" 선택
  ↓
확인 메시지 표시
  ↓
"Complete" 클릭
```

**선택 옵션 2: 특정 저장소만 (권장: 프로덕션)**
```
"Selected repositories" 선택
  ↓
"Select repositories" 버튼
  ↓
저장소 체크박스:
  ☑️ my-org/doc-repository (필수)
  ☑️ my-org/main-repo (선택)
  ↓
"Install" 클릭
```

### 2-3. 커넥터 상태 확인

```
Organization Connectors 페이지
  ↓
GitHub 항목 확인:
  ✅ Status: "Connected"
  ✅ Last synced: [시간] ago
  ✅ Repositories: [개수]
```

#### 저장소 권한 수정 (나중에)

```
GitHub 커넥터 옆 "..." 메뉴
  ↓
"Configure" 또는 "Edit"
  ↓
저장소 추가/제거
  ↓
"Save"
```

---

## 3. 개인 커넥터 연동 (각 멤버)

### 3-1. Claude Desktop 또는 Web 접근

```
Claude.ai (Web)
  → 좌측 하단 프로필 아이콘
  → "Customize"
  
또는 Claude Desktop
  → 좌측 "Customize"
```

### 3-2. GitHub 커넥터 연동

```
"Customize" 열기
  ↓
"Connectors" 탭
  ↓
GitHub 항목 확인:
  ✅ "Connected" 상태 (Organization이 이미 추가함)
  
만약 "Connect" 버튼만 보이면:
  ↓
  "Connect" 클릭
    ↓
  GitHub 로그인
    ↓
  "Authorize" 클릭
```

### 3-3. Slack 커넥터 연동 (선택)

```
"Connectors" 탭
  ↓
"Slack" 검색
  ↓
"Connect" 클릭
  ↓
Slack 작업공간 선택
  ↓
"Allow" 클릭
```

**권한 확인**:
```
✅ View messages and files in public channels
✅ View direct messages
✅ Search messages
```

---

## 4. Slack 설정

### 4-1. Claude 앱 설치

#### Slack Marketplace 접근

```
Slack 앱
  → "+" (새 앱 추가)
  → "앱 검색"
  → "Claude" 입력
  → "Claude" 공식 앱 선택 (Anthropic 배지 확인)
```

#### 설치 진행

```
"Slack에 추가" 버튼
  ↓
Slack 작업공간 선택 (이미 로그인된 경우 건너뜀)
  ↓
"설치" 클릭
  ↓
권한 화면:
  ✅ Send messages
  ✅ Post messages to channels
  ✅ Access user data
  ↓
"허용" 클릭
```

#### 설치 완료 확인

```
Slack 좌측 "Apps" → "Claude" 표시
또는
@claude 멘션 가능
```

### 4-2. 채널에 Claude 초대

```
채널에서 입력:
/invite @claude

또는

조직 설정 → Channel Settings
         → Integrations
         → Add an integration
         → Claude
```

확인:
```
Channel Details → Apps
  ✅ Claude 앱 표시
```

---

## 5. 통합 테스트

### 5-1. Slack에서 Claude 호출 테스트

#### 테스트 1: 기본 메시지

```
Slack 채널에서:
@claude 안녕! 너는 무엇을 할 수 있어?
```

**예상 응답**:
```
✅ Claude가 능력 설명
✅ Slack 스레드에 응답 표시
```

---

### 5-2. Claude에서 GitHub 접근 테스트

#### 테스트 2: 저장소 읽기

```
Slack 채널에서:
@claude [org]/[repo] 저장소의 README.md 파일을 읽어줄래?
```

**예상 응답**:
```
✅ README 파일 내용 표시
✅ 파일 경로 명시
✅ 마크다운 포매팅 유지
```

**만약 실패하면**:
```
에러: "Cannot access repository"

확인 사항:
1. Organization 수준에서 GitHub 커넥터 추가됨?
   → Settings → Organizations → Connectors → GitHub ✅
   
2. 개인 GitHub 커넥터 연동됨?
   → Settings → Customize → Connectors → GitHub ✅
   
3. 해당 저장소에 읽기 권한?
   → GitHub → [저장소] → Collaborators → 본인 계정 확인
```

---

### 5-3. Claude에서 GitHub에 파일 작성(PR) 테스트

#### 테스트 3: PR 생성

```
Slack 채널에서:
@claude 
[org]/doc-repository 에 새로운 파일을 만드는 PR을 생성해줄래?

파일명: docs/test-automation.md
내용: "이것은 Claude-Slack 자동화 테스트입니다"

PR 제목: "Test: Automation setup"
```

**예상 응답**:
```
✅ Feature branch 생성
✅ 파일 생성 (commit 메시지 포함)
✅ PR 생성 및 링크 제공
✅ Claude가 PR URL을 Slack 스레드에 응답
```

이것이 작동하면 모든 핵심 워크플로우가 가능합니다. ✅

---

## 🔧 트러블슈팅 빠른 참조

### 에러: "Insufficient permissions"

```bash
# Step 1: GitHub 확인
GitHub → [저장소] → Settings → Collaborators

# Step 2: 권한 부여
+ Add people → [username] → Write 선택

# Step 3: Claude 권한 갱신
Claude → Customize → Connectors → GitHub → Reconnect

# Step 4: 5분 대기 후 재시도
```

---

### 에러: "Cannot find connector"

```bash
# Organization이 커넥터를 추가하지 않았을 수 있음

# 확인:
Settings → Organizations → [org] → Connectors

# 없으면:
Organization Owner가 GitHub 커넥터 추가 필요
(본 가이드 섹션 2 참고)
```

---

### 에러: "Cannot find connector"

```bash
# Organization이 GitHub 커넥터를 추가하지 않았을 수 있음

# 확인:
Settings → Organizations → [org] → Connectors

# 없으면:
Organization Owner가 GitHub 커넥터 추가 필요

# 개인 연동 확인:
Customize → Connectors → GitHub → Reconnect
```

---

### 특수 경우: GitHub Enterprise (자체 호스팅)

```
주의: Claude는 기본적으로 github.com만 지원

GitHub Enterprise Server 사용 중인 경우:

1. Team Plan 필수
2. Enterprise Admin이 "GitHub Enterprise Server 커넥터" 활성화 필요
3. 도메인 설정:
   https://your-github-enterprise.company.com
```

---

## 📋 최종 체크리스트

### 조직 Owner 확인사항

- [ ] Team Plan 구독 완료
  ```
  https://claude.ai → Settings → Organizations
  ```

- [ ] Organization 생성 및 멤버 초대
  ```
  Organizations → [org] → Members → 5명 이상 추가
  ```

- [ ] GitHub 커넥터 Organization에 추가
  ```
  Organizations → [org] → Connectors → GitHub ✅
  ```

- [ ] Slack 작업공간 설정 (Admin)
  ```
  Slack → Settings → Apps & integrations → 
  Claude ✅
  ```

### 모든 멤버 확인사항

- [ ] Claude 계정에 GitHub 연동
  ```
  Customize → Connectors → GitHub → Connected ✅
  ```

- [ ] Slack에서 @claude 호출 가능
  ```
  임의 채널 → @claude 안녕! ✓
  ```

- [ ] Claude가 GitHub 저장소 읽기 가능
  ```
  @claude [org]/[repo] 의 README 읽어줄래? ✓
  ```

- [ ] Claude가 GitHub에 PR 생성 가능
  ```
  @claude [org]/[repo] 에 test.md 만드는 PR 생성해줄래? ✓
  ```

---

## 🚀 다음: 실제 사용 사례

설정이 완료되면 다음을 시도해보세요:

### 1. 회의 내용 정리
```
@claude 
위 Slack 스레드의 회의 내용을 정리해서
doc-repository/meeting-notes/ 에 저장해줄래?
```

### 2. PR 코드 리뷰
```
PR URL이 있는 스레드에서:
@claude 이 PR을 검토해주고 개선점을 지적해줄래?
```

### 3. 문서 생성
```
@claude 
"API Integration Guide" 문서를 마크다운으로 작성해서
PRs으로 만들어줄래?

내용 개요:
- REST API 엔드포인트
- 인증 방식
- 예시 코드
```

---

**마지막 수정**: 2026년 6월 5일  
**검증됨**: Claude Team Plan + Slack Free/Pro + GitHub Free/Pro

