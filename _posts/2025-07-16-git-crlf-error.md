---
layout: post
title: "[Git] LF will be replaced by CRLF in 에러 해결법"
date: '2025-07-16 15:32:27 +0900'
description: crlf error
image: assets\img\content\2025-07-16\image3.png
category: [BootCamp, TIL]
tags: [git]
---

# 이 경고는 무엇인가요?
![git error](/assets/img/content/2025-07-16/image4.png)

### 에러 발생: 

- git add 명령어를 입력하니 위와 같은 에러가 발생했습니다.

에러 원인 검색해보니까

- **LF (Line Feed)**는 Linux나 macOS에서 사용하는 줄 끝 방식이고,
- **CRLF (Carriage Return + Line Feed)**는 Windows에서 사용하는 줄 끝 방식이에요.

> 💡 만든 파일은 원래 LF로 되어 있었지만,
Git 설정에 따라 CRLF로 바뀌게 된다는 경고입니다.

### 에러 해결 방안:

1. Window, DOS 명령어

```
git config --global core.autocrlf true
```

2. Linux, MAC 명령어

```
git config --global core.autocrlf input
```

![solution](/assets/img/content/2025-07-16/image5.png)