---
title: "Oracle DB 테이블 스페이스 관리"
date: 2018-01-05 16:05:00 -0600
category: [ ORACLEDB ]
tags: [ PLSQL ]
---
***
## 테이블 스페이스 별 사용량

```sql
SELECT  DF.TABLESPACE_NAME                                              "테이블스페이스명"
       ,SUM(NVL(DF.BYTES, 0))                                           "크기(MB)"
       ,(SUM(NVL(DF.BYTES, 0)) - SUM(NVL(FS.BYTES, 0))) / 1024000       "사용됨(MB)"
       ,(SUM(NVL(FS.BYTES, 0))) / 1024000                               "남음(MB)"
       ,TRUNC((SUM(NVL(F.BYTES, 0)) / SUM(NVL(DF.BYTES, 0))) * 100, 2)  "남음(%)"
  FROM  DBA_FREE_SPACE    FS
       ,DBA_DATA_FILES    DF
 WHERE  FS.FILE_ID(+)   = DF.FILE_ID
GROUP BY DF.TABLESPACE_NAME
ORDER BY DF.TABLESPACE_NAME;
```