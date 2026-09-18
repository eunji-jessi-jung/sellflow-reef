<!-- Extracted verbatim from business-rules.xlsx with openpyxl on 2026-09-18 by the reef snorkel pass.
     The .xlsx is the original and remains authoritative; this rendering exists so the
     content is indexable and citable. Regenerate if the spreadsheet changes. -->


# Business Rules (취소정책 · 수수료)


## Sheet: 취소정책

| 취소 사유 코드 | 사유 | 비용 부담 주체 | 재고 복원 | 정산 차감 | 비고 |
|---|---|---|---|---|---|
| 01 | 파트너 귀책 (재고부족·출고지연) | 파트너 | O | O |  |
| 02 | 시스템 오류 | 셀플로우 | O | O |  |
| 03 | 고객 변심 | 셀플로우 | X | O | 배송 시작 후 취소 시 재고 복원 불가 |
| 04 | 배송 실패 (주소불명·수취거부) | 파트너 | O | O | 물류팀 확인 후 처리 |
| 정산 차감 기준 |  |  |  |  |  |
| 정산 실행 전 취소 | 해당 주문을 정산 대상에서 제외 |  |  |  |  |
| 정산 실행 후 취소 | 정산팀이 차월 정산에서 수기 차감 |  |  |  |  |
| 정산 배치 실행 시각 | 매일 02:00 KST |  |  |  |  |


## Sheet: 수수료

| 구분 | 수수료율 | 적용 |
|---|---|---|
| 기본 | 12.0% | 계약 미체결 파트너 전체 |
| 프리미엄 | 9.5% | 월 거래액 1억 이상 |
| 신규 프로모션 | 6.0% | 입점 3개월 이내 |
| 주의 | 계약별 수수료율은 PARTNER_CONTRACT 에 있으나 2021년 이후 미정비. 현재 전 건 기본 수수료율 적용 중. |  |

