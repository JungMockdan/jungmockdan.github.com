---
title: "개발일지 — 2026-06-02 (CVSS gate · 자체 미션 베타)"
excerpt: "OWASP CVSS≥9 gate·Boot 3.5.14, v1 자체 미션 베타(compose·verify), 문자 시드 B 잔여."
categories: [deVlog]
tags: [planet645, spring, security, 개발일지]
toc: true
toc_sticky: true
date: 2026-06-02 21:29:46 +0900
last_modified_at: 2026-06-02 21:52:05 +0900
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

> **하루 요약:** CVSS≥9 gate·Boot 3.5.14 정리에 이어, v1 **자체 미션 베타**(`AD-MISSION-BETA-01`) — compose·verify·보드 아카이브까지 마쳤다.

## 1. 오늘 목표

### Planet645

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기
- [x] `AD-MISSION-BETA-01` — v1 자체 미션 베타

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

### 2.2 v1 자체 미션 베타 (Planet645)

| 산출 | 경로 |
|------|------|
| prod+미션 compose | `docker-compose.mission-beta.yml` (`MISSION_ENABLED=true`) |
| Verify runbook / run | `artifact/ops/verify/AD-MISSION-BETA-01-runbook.md` · `runs/2026-06-02-AD-MISSION-BETA-01.md` |
| 보드 | `AD-MISSION-BETA-01` archived |

| 단계 | 내용 |
|------|------|
| BM | v1 **자체 미션** + (선택) AdSense — 오퍼월은 v2 후보(AdiSON·NAS 등) |
| 약관 D5 | `V5__terms_ad_tracking.sql` · `TERMS-AD-TRACKING-01` 선행 ✅ |
| 검증 | B01·B04 PASS · B02–B03·B05 Docker 미기동 `skip` (AD-REWARD R02–R03 2026-05-24 + Gradle) |

---

## 3. 문제와 해결

### 3.1 CVSS gate 실패 → BOM override (개발 운영)

| 문제 | 원인 | 해결 |
|------|------|------|
| `dependencyCheckAnalyze` FAILED @ CVSS≥9 | Boot 3.5.14 BOM에 Tomcat **10.1.54**, Netty **4.1.132** 잔존 | `ext` override → Tomcat **10.1.55**, Netty **4.1.133.Final** |
| 6 CVE (41293, 43512, 43515, 42579, 42581, 42584) | 패치 버전 미반영 | 위 override 후 재스캔 PASS |

### Planet645

| 문제 | 원인 | 해결 |
|------|------|------|
| Docker verify `skip` | WSL에서 Docker Desktop daemon 미기동 | compose·runbook 커맨드 문서화 · Desktop 기동 후 §2.1–2.2 재실행 |

---

## 4. 배운 점

- **정리 → 패치 → 게이트** 순서가 맞다. baseline(63 CRITICAL) 상태에서 `failBuildOnCVSS=9`만 켜면 CI가 바로 빨갛다.
- Boot minor 상향만으로 BOM이 최신 CVE 패치선에 못 미치면 **`ext` tomcat/netty override**가 필요하다.
- Stage 1 통과 후에도 **CVSS 7.5**(Netty CVE-2026-42582 등)는 남는다 — stage 2(≥7)는 별도 태스크.
- **매체 오퍼월**(AdiSON·NAS)은 웹 S2S 가능(v2) — v1 베타는 **자체 미션 플래그**만.

---

## 5. 다음 액션

### 5.1 미완

> **이월:** [2026-06-01-developer-log.md](2026-06-01-developer-log.md) §5.1.

#### Planet645

**2026-06-01 이월**

- [ ] `FEAT-TEXT-SEED-PREVIEW-B-01` — 문자 시드 범위 B · 고정/제외 번호 채우기

#### 개발 운영

**2026-06-01 이월**

_(이월 항목 없음 — CVSS gate 완료)_

**당일 신규**

- [ ] (후속) OWASP stage 2 — `failBuildOnCVSS=7` (Netty 42582 등 잔여 HIGH 정리 후)

### 5.2 완료 (인덱스)

| 항목 | §2 |
|------|-----|
| `SECURITY-DEPENDENCY-CVSS-01` — CVSS≥9 gate · baseline · Boot 3.5.14 | §2.1 |
| `AD-MISSION-BETA-01` — v1 자체 미션 베타 · compose · verify | §2.2 |

---

## 6. 기준

| 항목 | 값 |
|------|-----|
| 보드 | `artifact/ops/tasks/board.md` |
| 환경 표기 | 로컬 prod-profile 회귀 (staging 표기 없음) |
| 전일 일지 | [2026-06-01-developer-log.md](2026-06-01-developer-log.md) |
| dependency-check | `com.mockdan.life-saver-lotto/build/reports/dependency-check-report.html` |
