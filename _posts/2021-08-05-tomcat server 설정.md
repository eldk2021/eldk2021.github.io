---
title: tomcat server 설정
date: 2021-08-05
tags: settings server
---


톰캣 서버 설정

```markdown
배포 : 운영 중인 서버에 서비스를 등록

배포파일(war 파일 생성 전) 내보내기 전 신경써야할 사항
1. 파일 업로드 경로
2. 로그 설정

log4j 로그레벨

TRACE > DEBUG > INFO > WARN > ERROR >FATAL

TRACE : 초정밀. 모든 세부사항들
DEBUG : 굵직한 흐름 [개발]
INFO : 정보성 출력 [개발]
WARN : 경고성 출력 [운영]
ERROR : 오류, 예외 발생 출력(서비스 에러 + 서버 에러) [운영]
FATAL : 심각한 오류(서버 에러)
```