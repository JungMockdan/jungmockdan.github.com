---
title: "개발일지 — 2026-05-18 (Flyway baseline 통합·로컬 DB·엑셀 당첨번호 import)"
excerpt: "Flyway checksum 불일치 해소를 위해 V1–V29를 V1__baseline_schema.sql로 스쿼시·로컬 DB reset. local 프로파일 엑셀 import 비활성 원인 정리 후 APP_LOTTO_EXCEL_IMPORT_ENABLED로 VS Code·Docker 기동 시 1217회차 적재."
date: 2026-05-18 12:00:00 +0900
last_modified_at: 2026-05-18 12:00:00 +0900
categories: [deVlog]
tags: [planet645, flyway, mariadb, excel-import, spring-boot, docker-compose, 개발일지]
toc: true
toc_sticky: true
---

> [이전 2026-05-17 일지](/2026-05-17-developer-log) 이후, **로컬 Spring 기동 실패(Flyway)** 와 **당첨번호 테이블 공백(엑셀 import 미실행)** 을 같은 날에 정리했다.

---

## 1. 오늘 목표

### Planet645

- [x] VS Code 디버그로 Spring 앱 정상 기동
- [x] `t_win_base_info` / `t_win_numbers`에 과거 회차 데이터 적재
- [x] Flyway 마이그레이션 이력 정리(파일 수 축소·팀 규칙)

### 개발 운영

- [x] Flyway 불변·로컬 DB reset 절차 문서화
- [x] Docker Compose / VS Code launch에서 엑셀 import 옵션 명시

---

## 2. 주요 변경 사항

### 2.1 Flyway checksum 불일치 · baseline 스쿼시

| 항목 | 내용 |
|------|------|
| 증상 | 기동 시 `Migration checksum mismatch` (V16, V25, V26, V27) |
| 원인 | DB에는 **예전 파일 내용**으로 적용된 이력(`terms consent core` 등)이 남아 있는데, 워크스페이스는 **동일 버전 번호의 SQL을 교체·수정**한 상태 |
| 조치 | 로컬 `lottery` DB **drop/create** 후, 기존 `V1`–`V29` **29개 파일 → `V1__baseline_schema.sql` 1개**로 통합·삭제 |
| 검증 | `./gradlew clean bootRun` → `Successfully applied 1 migration` · `Started LifeSaverLottoApplication` |

팀 규칙 추가·갱신:

- `com.mockdan.life-saver-lotto/.claude/rules/database.md` — Flyway policy, 로컬 reset 명령
- `com.mockdan.life-saver-lotto/.claude/rules/how-to.md` — checksum 오류 대응
- `artifact/runbook/backend/05-data-model-and-migrations.md` — 타임라인을 baseline + `V2+` 신규만으로 정리

> **주의:** 이미 적용된 `V1__baseline_schema.sql`은 수정하지 말고, 스키마 변경은 `V2__<desc>.sql`부터 추가. squash/rebase 후 팀원은 **반드시 로컬 DB reset + `./gradlew clean`**.

### 2.2 엑셀 당첨번호 기동 import

`WinningNumberExcelImportRunner`는 `app.lotto.excel-import.enabled=true`일 때만 등록된다.

```properties
# application-local.properties (변경 후)
app.lotto.excel-import.enabled=${APP_LOTTO_EXCEL_IMPORT_ENABLED:false}
```

| 구분 | 내용 |
|------|------|
| 왜 안 돌았나 | `local` 프로파일 기본값이 **`false`** → `CommandLineRunner` 빈 미등록 |
| 데이터 파일 | `src/main/resources/file/all_result_20260401132352.xlsx` (classpath `file/...`) |
| 로그 마커 | `엑셀 파일 로드 시작` → `엑셀 데이터 로드 완료 (신규 N건 추가)` |

반영한 설정:

| 위치 | 설정 |
|------|------|
| `.vscode/launch.json` | `APP_LOTTO_EXCEL_IMPORT_ENABLED=true` |
| `docker-compose.yml` (`spring-app`) | 동일 env |
| `.env.example` | 문서·예시 값 |

검증 (`APP_LOTTO_EXCEL_IMPORT_ENABLED=true`):

- `t_win_base_info` / `t_win_numbers` 각 **1217건** (회차 1~1217)

### 2.3 개발 운영

- WSL에서 `gradlew` CRLF(`\r`)로 실행 실패 → `sed -i 's/\r$//' gradlew`로 line ending 정리(로컬 1회성 이슈)

---

## 3. 문제와 해결

### 3.1 Flyway Validate failed (checksum)

| | |
|---|---|
| **문제** | `entityManagerFactory` 생성 실패, Flyway `Validate failed` |
| **원인** | 적용 후 마이그레이션 SQL 편집·파일명 변경(V25 `terms_consent_core` → `terms_consent_schema` 등) |
| **해결** | 로컬 DB 초기화 + 마이그레이션 단일 baseline으로 rebase. `flyway repair`만으로는 스키마 불일치(`terms_policy_revision` vs `terms_policy_state` 등) 위험 |

### 3.2 기동은 됐는데 당첨 테이블이 비어 있음

| | |
|---|---|
| **문제** | VS Code 디버그 기동 후 `t_win_*` 행 0건 |
| **원인** | `app.lotto.excel-import.enabled=false` (의도적 기본값, 빠른 로컬 기동) |
| **해결** | 환경변수로 import 활성화. 엑셀 파일은 repo에 존재 — 설정만 켜면 됨 |

### 3.3 clean 없이 bootRun 시 V25 중복 컬럼

| | |
|---|---|
| **문제** | `build/resources/main`에 **삭제 전** migration 캐시 잔존 → `Duplicate column name 'title'` |
| **원인** | 소스는 `V1`만인데 빌드 산출물에 구 `V25`…`V29` 포함 |
| **해결** | `./gradlew clean` 후 재기동 |

### 개발 운영

_(해당 없음 — §2.3 gradlew CRLF는 본문에 기록)_

---

## 4. 배운 점

- Flyway는 **버전 번호**만 본다. 파일 이름·내용을 바꾸면 checksum이 깨진다. 운영 DB가 생기기 전 로컬 단계에서는 **baseline 스쿼시 + DB reset**이 가장 단순하다.
- `@ConditionalOnProperty`로 꺼 둔 `CommandLineRunner`는 로그에 아무것도 안 남는다 — “기능이 안 돈다”고 느끼기 전에 **빈 등록 조건**을 먼저 본다.
- 마이그레이션 파일을 줄인 뒤에는 **`gradlew clean`** 이 캐시 불일치를 막는 필수 단계다.

---

## 5. 다음 액션

### Planet645

- [ ] Docker Compose 노캐시 빌드 완료 후 `spring-app` 기동·엑셀 import 로그·회차 수 확인
- [ ] 통계 warm-up(`app.lotto.draw-statistics.warmup-on-startup`)과 추천 화면 smoke
- [ ] 스키마 변경 시 `V2__...` 신규 migration 추가( V1 수정 금지)

### 개발 운영

- [ ] 팀원 공유: Flyway squash PR 머지 시 **로컬 lottery DB reset** 필수
- [ ] PR 본문에 `APP_LOTTO_EXCEL_IMPORT_ENABLED`·엑셀 파일 경로 한 줄 안내

---

## 6. 기준

- 저장소: `com.mockdan.life-saver-lotto-workspace` (Spring `com.mockdan.life-saver-lotto`, MariaDB `lottery`)
- 환경 정책: 스테이징 없음 — 로컬 `local` / 회귀는 로컬 `prod` profile (`CLAUDE.md`)
