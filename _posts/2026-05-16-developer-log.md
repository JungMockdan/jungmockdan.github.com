---
title: "개발일지 — 2026-05-16 (Application 부트스트랩 리팩터링)"
date: 2026-05-16 08:45:00 +0900
last_modified_at: 2026-05-16 08:45:00 +0900
categories: [deVlog]
tags: [planet645, spring, refactoring, bootstrap, excel-import, testing, board]
excerpt: "LifeSaverLottoApplication의 엑셀 당첨번호 적재 책임을 runner/service/parser로 분리하고, prod-profile import 비활성화와 단위 테스트를 추가했다."
toc: true
toc_sticky: true
---

## 1. 오늘 목표

### Planet645

- [x] `LifeSaverLottoApplication`에 몰려 있던 엑셀 당첨번호 적재·파싱·DB 병합 책임 분리.
- [x] Spring Boot 엔트리포인트를 `main()`과 전역 애노테이션 중심으로 단순화.
- [x] prod-profile에서 시작 시 엑셀 import가 실행되지 않도록 명시 설정.
- [x] 신규 회차 저장과 기존 회차 base info 병합 동작 단위 테스트 추가.

### 개발 운영

- [x] `artifact/ops/tasks/board.md`에 `PHASE22-APP-BOOTSTRAP-REFACTOR-01` 등록.
- [x] 코드리뷰 지적 사항 반영 후 관련 테스트 성공 기록.

---

## 2. 주요 변경 사항

### 2.1 Spring Boot 엔트리포인트 정리

- `LifeSaverLottoApplication`에서 Apache POI 파싱, `CommandLineRunner`, DB 저장/병합 로직을 제거했다.
- 애플리케이션 클래스는 `@SpringBootApplication`, 캐시/JPA auditing/스케줄링 설정, `main()`만 갖도록 정리했다.

### 2.2 엑셀 당첨번호 import 책임 분리

- `WinningNumberExcelImportRunner`: 앱 시작 시 import 실행을 담당한다.
- `WinningNumberExcelImportService`: classpath 리소스 로드, 신규 회차 저장, 기존 `TWinBaseInfo` 병합을 담당한다.
- `WinningNumberExcelParser`: Apache POI 기반 엑셀 행 파싱을 담당한다.
- `WinningNumberExcelRow`: 파싱 결과를 전달하는 package-private record로 분리했다.

### 2.3 prod-profile 실행 가드

- `application-prod.properties`에 `app.lotto.excel-import.enabled=false`를 추가했다.
- 로컬 기본 동작은 유지하되, prod-profile 실행 시 오래된 엑셀 seed import가 자동 실행되지 않게 했다.

### 2.4 테스트 보강

- `WinningNumberExcelImportServiceTest` 추가.
- 신규 회차는 base info 저장 후 win numbers를 저장하는지 검증했다.
- 기존 회차는 win numbers를 추가하지 않고 base info의 당첨자 수·당첨금·일자를 병합하는지 검증했다.

---

## 3. 문제와 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| `LifeSaverLottoApplication`이 비대함 | 부팅 진입점에 엑셀 파싱·저장·병합 책임이 섞임 | runner/service/parser/record로 분리 |
| prod-profile에서도 import runner가 기본 실행될 수 있음 | `@ConditionalOnProperty(matchIfMissing=true)`로 기존 동작 유지 | `application-prod.properties`에서 명시적으로 비활성화 |
| 리팩터링 검증이 context load에 치우침 | 분리된 service 동작을 직접 검증하는 테스트 부족 | `WinningNumberExcelImportServiceTest` 추가 |
| `./gradlew` 직접 실행 실패 | wrapper 파일 실행 권한 없음 | `bash ./gradlew ...`로 테스트 수행 |

---

## 4. 검증

- IDE linter: 변경 파일 기준 오류 없음.
- `bash ./gradlew test --tests com.mockdan.lifesaverlotto.LifeSaverLottoApplicationTests` — `BUILD SUCCESSFUL`.
- `bash ./gradlew test --tests com.mockdan.lifesaverlotto.cmm.lottery.WinningNumberExcelImportServiceTest --tests com.mockdan.lifesaverlotto.LifeSaverLottoApplicationTests` — `BUILD SUCCESSFUL`.

---

## 5. 다음 액션

### Planet645

- [ ] `PHASE22-APP-BOOTSTRAP-REFACTOR-01` PR merge 후 보드 아카이브 여부 검토.
- [ ] 향후 seed/import 류 부팅 작업은 `@ConditionalOnProperty`와 profile별 명시 설정을 기본 패턴으로 둘 것.

### 개발 운영

- [ ] 현재 브랜치 변경을 PR로 올리고 merge 완료까지 추적.
