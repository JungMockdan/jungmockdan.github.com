---
title: "WINDOWS에 한꺼번에 파일 만들기 BAT"

date: 2022-01-07 13:34:16 -0400

categories: bat 명령어로 템플릿 적용된 파일 만들기

tags: bat batch

# 목차

toc: true  
toc_sticky: true
---

```shell
@echo off
setlocal EnableDelayedExpansion
setlocal EnableExtensions

set arr[0]=foo
set arr[1]=bar
:WHILE_0
for /L %%D in (0,1,1) do (
  set _conts=/* 안녕하세요 */
  set _postfix=.txt
  set _title=!arr[%%D]!!_postfix!
  echo %%D : !arr[%%D]!
  echo %%D : !_conts!
  echo %%D : !_postfix!
  echo %%D : !_title!
  echo.
  echo !_conts!>!_title!
)
```
위 파일을 실행하면 2개의 파일 foo.txt, bar.txt를 가진다.
두파일에는 _conts가 써진 상태이다.

응용하여 템플릿을 적용한 파일을 자동 생성해보면
```shell
@echo off
setlocal EnableDelayedExpansion
setlocal EnableExtensions

set arr[0]=foo
set arr[1]=bar
:WHILE_0
for /L %%D in (0,1,1) do (
  set _conts=/* 안녕하세요 */
  set _postfix=.html
  set _title=!arr[%%D]!!_postfix!
  echo %%D : !arr[%%D]!
  echo %%D : !_conts!
  echo %%D : !_postfix!
  echo %%D : !_title!
  echo.
  echo !_conts!>!_title!
  echo.>> !_title!
  echo ^<^^!DOCTYPE html^>>> !_title!
  echo ^<html lang^=^'en^'^>>> !_title!
  echo ^<head^>>> !_title!
  echo     ^<meta charset^=^'UTF-8^'^>>> !_title!
  echo     ^<title^>!arr[%%D]!^</title^>>> !_title!
  echo ^</head^>>> !_title!
  echo ^<body^>>> !_title!
  echo ^</body^>>> !_title!
  echo ^</html^>>> !_title!
)
```

위의 경우 foo.html, bar.html 2개의 파일이 생성되는데 title 태그안에 파일명이 들어가도록 생성하였다.
한글 깨짐 현상 해결 : $env 를 설정해도 한글이 깨지기 때문에 참조 사이트에서 한데로 진행해야 한다.

https://psychoria.tistory.com/737

관리자권한
```shell
if (!(Test-Path -Path $PROFILE.AllUsersAllHosts)) {
  New-Item -ItemType File -Path $PROFILE.AllUsersAllHosts -Force
  "`$env:LC_ALL='C.UTF-8'" | 
    Out-File -FilePath $PROFILE.AllUsersAllHosts -Append
  "[System.Console]::InputEncoding = [System.Console]::OutputEncoding = [System.Text.Encoding]::UTF8" | 
    Out-File -FilePath $PROFILE.AllUsersAllHosts -Append
}
```

파워셀 재실행 
에러발생시 에러 발생 하면 역시 관리자 권한으로
```shell
Get-ExecutionPolicy -list

Set-ExecutionPolicy RemoteSigned
```

파워셀 재실행 