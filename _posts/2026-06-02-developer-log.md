---
title: "개발일지 — 2026-06-02 (OWASP CVSS gate · Boot 3.5.14)"
excerpt: "dependency-check CVSS≥9 fail gate 1단계, Boot 3.5.14·Tomcat/Netty BOM override, weekly OWASP CI fail 활성."
categories: [deVlog]
tags: [planet645, spring, security, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-02 21:29:46 +0900
last_modified_at: 2026-06-02 21:30:36 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** OWASP **CVSS≥9** 게이트 1단계를 적용하고, Boot **3.5.14**·BOM override(Tomcat **10.1.55**, Netty **4.1.133**)로 `dependencyCheckAnalyze` **PASS**를 확인했다.

## 1. 오늘 목표

### Planet645

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `AD-MISSION-BETA-01`

### 개발 운영

- [x] `SECURITY-DEPENDENCY-CVSS-01` — CVSS fail gate 1단계 · baseline · allowlist

---

## 2. 주요 변경 사항

### 2.1 OWASP CVSS fail gate 1단계 (개발 운영)

| 산출 | 경로 |
|------|------|
| Triage·게이트 SoT | `artifact/runbook/security/dependency-cvss-baseline.md` |
| Runbook §4.1 | `artifact/runbook/security/dependency-supply-chain.md` |
| Allowlist (Cassandra N/A) | `com.mockdan.life-saver-lotto/config/dependency-check-suppressions.xml` |
| Gradle gate | `failBuildOnCVSS=9`, `failOnError=true` |
| Weekly CI | `.github/workflows/dependency-supply-audit.yml` — OWASP `continue-on-error: false` |
| 보드 | `SECURITY-DEPENDENCY-CVSS-01` archived |

| 단계 | 내용 |
|------|------|
| 정리 | Boot 3.3.13 스캔 63취약/425 CVE → 패치·override 계획 |
| 패치 | Spring Boot **3.5.14**, springdoc **2.8.17**, POI **5.4.1** |
| BOM override | `tomcat.version=10.1.55`, `netty.version=4.1.133.Final` |
| 검증 | `./gradlew dependencyCheckAnalyze` **PASS** (취약 12·최고 CVSS **7.5**) |

### 2.2 Planet645

_(해당 없음)_

---

## 3. 문제와 해결

### 3.1 CVSS gate 실패 → BOM override (개발 운영)

| 문제 | 원인 | 해결 |
|------|------|------|
| `dependencyCheckAnalyze` FAILED @ CVSS≥9 | Boot 3.5.14 BOM에 Tomcat **10.1.54**, Netty **4.1.132** 잔존 | `ext` override → Tomcat **10.1.55**, Netty **4.1.133.Final** |
| 6 CVE (41293, 43512, 43515, 42579, 42581, 42584) | 패치 버전 미반영 | 위 override 후 재스캔 PASS |

### Planet645

_(해당 없음)_

---

## 4. 배운 점

- **정리 → 패치 → 게이트** 순서가 맞다. baseline(63 CRITICAL) 상태에서 `failBuildOnCVSS=9`만 켜면 CI가 바로 빨갛다.
- Boot minor 상향만으로 BOM이 최신 CVE 패치선에 못 미치면 **`ext` tomcat/netty override**가 필요하다.
- Stage 1 통과 후에도 **CVSS 7.5**(Netty CVE-2026-42582 등)는 남는다 — stage 2(≥7)는 별도 태스크.

---

## 5. 다음 액션

### 5.1 미완

> **이월:** [2026-06-01-developer-log.md](2026-06-01-developer-log.md) §5.1.

#### Planet645

**2026-06-01 이월**

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [ ] (선택) `AD-MISSION-BETA-01`

#### 개발 운영

**2026-06-01 이월**

_(이월 항목 없음 — CVSS gate 완료)_

**당일 신규**

- [ ] (후속) OWASP stage 2 — `failBuildOnCVSS=7` (Netty 42582 등 잔여 HIGH 정리 후)

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| `SECURITY-DEPENDENCY-CVSS-01` — CVSS≥9 gate · baseline · Boot 3.5.14 | §2.1 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-06-01-developer-log.md](2026-06-01-developer-log.md) |
| dependency-check | `com.mockdan.life-saver-lotto/build/reports/dependency-check-report.html` |
