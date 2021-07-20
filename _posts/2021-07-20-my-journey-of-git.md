---
title: "나와 GIT"
date: 2021-07-20 13:26:28 -0400
categories: git
tags: git command
# 목차
toc: true  
toc_sticky: true
---
## 2021-07-20

### 상황 
- 계속 재택을 하다가 어제 간만에 출근을 하는데, 로컬에서 작업한 브랜치가 있었고, 이것은 PUSH하고 싶지 않았다. 그 상태로 새브랜치(temp)를 따고, commit 했다. 어제 회사에서 작업을 잘 하고,  commit을 했고, 오늘 아침에 땡기려고 하니....난리다....난리... 충돌도 나고...무슨 commit 도 안된다고 하고.... 원인을 파악하는것은 포기했다.
### 해결 :
1. 일단 로컬에 혹시 commit하고 push 하지 않은 파일이 있는지 확인했다.
```bash
> git log --branches --not --remotes // 예쁘지가 않다. 알아보기가 힘들다.
> git log --branches --not --remotes --oneline --graph --decorate  // 휴..겨우 읽고 싶은 마음이 생겼다. 
```
- 깔끔하게도 그런 파일이 없으니 바로 다음단계다.
2. 일단 기존 로컬로 딴 브랜치 temp 는 delete 했다.
```bash
> git branch -d  feature/temp  // 실패했다. 그래서 강제로 삭제했다.
> git branch -D  feature/temp  // 성공이다.
```
3. 원격의 feature/temp를 checkout 했다.
```bash
> git remote update  // 원격저장소를 한번 업데이트 해줬다. 혹시 생성한 브랜치가 안보이면 함 해주면 보일것이다.
> git branch // 로컬저장소의 브랜치 목록확인이다
> git branch -r // 원격저장소의 브랜치 목록확인이다
> git branch -s // 로컬/원격저장소의 브랜치 모든 목록확인이다
> git checkout -t origin/feature/temp // 드디어 checkout 했다.
```
4. 모두 잘 해결되었다. 

## 2021-??-??

### 상황
- 
### 해결 :
1. 
```bash

```
