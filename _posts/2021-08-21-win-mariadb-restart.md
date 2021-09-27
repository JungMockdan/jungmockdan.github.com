---
layout: post
title: "윈도우에서 마리아db 재시작하기"
date: 2021-08-21 08:37:24 -0400
categories: database
tags: mariadb restart window


# 목차
toc: true  
toc_sticky: true
---
    
## 마리아 db 재시작하기
회사에서 집의 로컬db에 접속하기 위해 접속허용설정과 권한 변경을 하고 나서 서비스를 재시작하려고 했는데 잘 되지 않았다.
아래 처럼 해결하였다.
![관리자 권한 cmd](https://github.com/JungMockdan/jungmockdan.github.com/blob/gh-pages/assets/images/post/cmd-mariadb-restart.PNG?raw=true)
```shell
// 바로 서비스를  중지해봅니다.
C:\Program Files\MariaDB 10.5\bin> net stop mysql
서비스 이름이 잘못되었습니다.

NET HELPMSG 2185을(를) 입력하면 도움말을 더 볼 수 있습니다.
// 서비스 등록이 안되어 있습니다.
// 관리자권한으로 cmd를 실행합니다. 설치합니다.
C:\Program Files\MariaDB 10.5\bin>mysqld -install
Service successfully installed.
// 서비스 중지
C:\Program Files\MariaDB 10.5\bin>net stop mariadb
MariaDB 서비스를 멈춥니다..
MariaDB 서비스를 잘 멈추었습니다.

//서비스 시작
C:\Program Files\MariaDB 10.5\bin>net start mariadb
MariaDB 서비스를 시작합니다...
MariaDB 서비스가 잘 시작되었습니다.
```

위처럼 하면 됩니다.