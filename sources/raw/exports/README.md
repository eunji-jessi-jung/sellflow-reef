# 데이터 추출 기록

이 폴더의 파일은 특정 시점에 운영 DB 에서 뽑은 것이다. **재추출 없이 그대로 인용하지 말 것.**

## cancel_recon_queue_monthly_20260901.csv

- **추출일:** 2026-09-01
- **환경:** settlement_prod (read replica)
- **요청자:** 재무기획팀 문지영 / **추출:** 데이터팀 윤서진
- **조건:** `SELECT DATE_FORMAT(RECV_DTM,'%Y-%m') AS ym, COUNT(*), SUM(EXPECTED_AMT)
  FROM CANCEL_RECON_QUEUE WHERE STATUS='PENDING' GROUP BY 1 ORDER BY 1`
- **결과:** 41개월 · 합계는 파일 하단 참조
- **비고:** STATUS 가 PENDING 외의 값을 가진 행은 조회되지 않았다. 수기 정정분이
  시스템 밖에서 처리되었다면 이 수치에 반영되지 않는다.

## 재추출 방법

```
bin/export-queue.sh --table CANCEL_RECON_QUEUE --group-by month --env prod
```

(스크립트 위치 TBD — 현재는 DBA 에게 요청)
