---
title: "테이블 스페이스 요량 확인"
date: 2019-01-05 16:05:00 -0600
tags: [ PL-SQL ]
---
### Tablespace 관리
###### Tablespace 별 사용량
```sql
SELECT  U.TABLESPACE_NAME "테이블스페이스명",
        SUM(NVL(U.BYTES, 0)) "크기(MB)",
        (SUM(NVL(U.BYTES, 0)) - SUM(NVL(F.BYTES, 0))) / 1024000 "사용됨(MB)",
        (SUM(NVL(F.BYTES, 0))) / 1024000 "남음(MB)",
        TRUNC((SUM(NVL(F.BYTES, 0)) / SUM(NVL(U.BYTES, 0))) * 100, 2) "남음(%)"
  FROM  DBA_DATA_FILES U,
        DBA_FREE_SPACE F
 WHERE  U.FILE_ID = F.FILE_ID(+)
GROUP BY U.TABLESPACE_NAME
ORDER BY U.TABLESPACE_NAME;
```