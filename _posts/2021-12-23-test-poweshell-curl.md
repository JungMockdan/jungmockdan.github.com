---
title: "Window powershell에서 웹리퀘스트 테스트하기"
date: 2021-12-23 13:26:28 -0400
categories: roadmap
tags: 테스트 제한된 개발환경 파워쉘


# 목차
toc: true  
toc_sticky: true 
---

- 보안강화로 프로그램설치등이 어려운 상황에서 curl 테스트와 비슷하게 테스트 가능함.

##  window > powershell 에서 webrequest 테스트 해보기
```bash
$URL = "api 주소";
$text = "key=value&key=value&key=value&key=value...."
$postParam=[System.Text.Encoding]::UTF8.GetBytes($text)
(Measure-Command -Expression { Invoke-WebRequest -Method POST -Body $postParam -ContentType "application/x-www-form-urlencoded; charset=utf-8" -Uri $URL }).Milliseconds
```
