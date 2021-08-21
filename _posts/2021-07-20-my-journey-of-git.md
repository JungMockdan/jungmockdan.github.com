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
```shell
> git log --branches --not --remotes // 예쁘지가 않다. 알아보기가 힘들다.
> git log --branches --not --remotes --oneline --graph --decorate  // 휴..겨우 읽고 싶은 마음이 생겼다. 
```
깔끔하게도 그런 파일이 없으니 바로 다음 단계다.
2. 일단 기존 로컬로 딴 브랜치 temp 는 delete 했다.
```shell
> git branch -d  feature/temp  // 실패했다. 그래서 강제로 삭제했다.
> git branch -D  feature/temp  // 성공이다.
```
3. 원격의 feature/temp를 checkout 했다.
```shell
> git remote update  // 원격저장소를 한번 업데이트 해줬다. 혹시 생성한 브랜치가 안보이면 함 해주면 보일것이다.
> git branch // 로컬저장소의 브랜치 목록확인이다
> git branch -r // 원격저장소의 브랜치 목록확인이다
> git branch -s // 로컬/원격저장소의 브랜치 모든 목록확인이다
> git checkout -t origin/feature/temp // 드디어 checkout 했다.
```
4. 모두 잘 해결되었다. 


## 2021-07-29

### 상황 : intelij gui에서 commit and push 를 하다 실패되어 커밋내용을 확인하고 싶었다.

### 해결 :
1. xxx
```shell
> git log
> git log -2 // 숫자는 최근 2개의 커밋이라는 의미 
```

## 2021-08-21

### 상황
- 브랜치 이름을 잘못 만들어 원격에  push 했다. 협업프로젝트라서 수정해야 하는 상황이 발생했다. 해당 프로젝트는 브랜치 디렉토리 전략이 있다. 각 모듈마다 이름을 두고, 모듈/master가 있고 그하위로  feature이름을 붙이게 되었다. 나의 경우....모듈명에 오타가 났다. 
### 해결 :
1. 로컬 저장소 이름변경
```shell
git branch -m <new_name_branch>
```
2. 원격 저장소 이름변경

```shell
1. 변경할 브랜치로 체크아웃한다.
git checkout <old_name_branch>

2. 새이름으로 push 한다.
git push origin -u <new_name_branch>

3. 잘못된 이름의 원격 branch를 삭제한다.
git push origin --delete <old_name_branch>
4. 현재 브랜치의 정보를 확인하면 head 정보가 바뀌었음을 확인가능하다.
git log --graph --pretty=oneline --abbrev-commit --all --decorate
* 8b1eb3c (HEAD -> vos-biz/web-api, origin/vos-biz/web-api) response
```


## 2021-xx-xx

### 상황
- 
### 해결 :

```shell

```