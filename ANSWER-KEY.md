# 셀플로우 — Answer Key

**Verifiability.** Every fragment of the keystone answer, and where it lives.
A human must be able to check any claim in this reef against this document.

> **Spoiler.** This is the grading key for the
> [sellflow](https://github.com/eunji-jessi-jung/sellflow) fixture. If you intend
> to work the brief yourself, stop here.
>
> Paths are relative to that repository: code paths resolve under `repos/`,
> document paths under `sources/`. So `handover/2025-03_정산팀_인수인계.md` is
> `sellflow/sources/handover/2025-03_정산팀_인수인계.md`, and
> `legacy/OrderCancelServiceV1.java` is under
> `sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/`.

## Keystone question

> 셀플로우가 정산 정정 업무를 AI 에이전트로 자동화하려고 합니다.
> 현재 프로세스가 어떻게 도는지, 무엇을 자동화하면 되는지 3일 안에 파악해 주세요.

## Correct answer

**자동화할 프로세스가 존재하지 않는다.** 대기열은 3년간 채워지기만 했고,
이상 탐지 모델은 1년 넘게 같은 건들을 지적해 왔으며, 아무도 보지 않았다.

## Fragment map

| # | Fragment | Lives in | Type | Tier |
|---|---|---|---|---|
| 1 | 취소 API가 정산 상태를 확인하지 않음 | `OrderCancelService.CHWISO_BULGA` | code | medium |
| 2 | `JUNGSAN_WANRYO` 상태는 존재하나 취소 경로에서 미검사 | `OrderStatus` | code | **hard (함정)** |
| 3 | 정산은 배송완료일 기준으로만 추출 — 취소 여부 무관 | `DailySettlementJobConfig.settlementTargetReader` | code, 타 repo | hard |
| 4 | 지급은 되돌릴 수 없음 | `SettlementItemWriter` javadoc | code | easy |
| 5 | 취소 이벤트는 아웃박스 → 큐로만 이동 | `OrderEventRelayJob` | code, 타 repo | medium |
| 6 | **큐를 비우는 것이 없음** | `CancelReconciler` — `@Component`, 호출자 0 | code, **부재** | **hard** |
| 7 | 사유 03은 재고 미복원 | `inventory-api.RESTOCKABLE_REASONS` | code, 타 repo | medium |
| 8 | 사유 03 비용은 셀플로우 부담 | `business-rules.xlsx` | **non-code** | easy |
| 9 | **정정 소유자 = 정산팀** | `org-chart.xlsx` | **non-code, 유일** | medium |
| 10 | 2023년 의도적 결정, 예상 월 10건 미만 | `SF-2287` | **non-code, 유일** | medium |
| 11 | 위키는 틀렸다 (2021-03 기준) | `confluence-snapshots/주문-취소-정책_48213.html` §4 | **non-code, 상충, HTML export** | medium |
| 11b | **위키는 사실 `OrderCancelServiceV1` 을 정확히 기술한다** — 해당 클래스는 아직 저장소에 있고 `SETTLEMENT_DTL` 을 확인하며 예외를 던진다. 다만 호출자가 없다 | `legacy/OrderCancelServiceV1.java` | code, **함정** | **hard** |
| 11c | 2024-08 누군가 이미 위키 댓글로 "반영이 안 된 것 같다"고 물었으나 답이 없다 | 같은 HTML export 댓글 블록 | **non-code** | medium |
| 12 | API 스펙도 틀렸다 (409 문서화, 2022) | `order-service-openapi.json` | **non-code, 상충** | medium |
| 13 | **AI가 이미 탐지 중, 조치 0건** | `settlement-anomaly` + `SETTLEMENT_ANOMALY.STATUS` | code + data | medium |
| 14 | 실제 적체 4,127건 / 1.89억원 / 월평균 101건 | `cancel_recon_queue_monthly_20260901.csv` | **non-code, 데이터** | easy |
| 15 | 현행 기획서가 잘못된 전제 위에 있음 | `2026_정산정정_AI에이전트_자동화_기획.md` §2 | **non-code, 상충** | medium |

**17 fragments · 9 from non-code · 2 provable only by absence (#6, #11b) · 3 mutually contradicting sources (#11, #12, #15)**

### 추가 조각 — 비코드 소스 확장 (2026-09-18)

실무 저장소와 지식 베이스에서 일반적으로 관찰되는 문서 구성을 따라 유형을 보강했다.
아래 조각은 전부 **코드로는 복원할 수 없다.**

| # | Fragment | Lives in | Type | Tier |
|---|---|---|---|---|
| 16 | 인계 시점(2025-03)에 「대기열 자동 처리 여부 확인 필요」가 미해결로 남았고 담당자는 퇴사했다 | `handover/2025-03_정산팀_인수인계.md` | **non-code, 유일** | medium |
| 17 | 2025-07 회고의 「취소 건 정산 제외 점검」 재발방지 항목이 미체크로 남아 있다 | `runbooks/장애회고_2025-07-12_정산배치_중복실행.md` | **non-code, 부재** | medium |
| 18 | 2023-04-24 「스케줄 등록만 남았다」 이후 같은 지연이 3년간 반복된다 | `slack/settlement-dev_2023-04_2026-08.json` | **non-code, 유일** | **hard** |
| 19 | 재무기획팀이 2026-08 에 이미 절차와 잔액의 불일치를 지적했고 답을 받지 못했다 | `mail/RE_정산_미정정_금액_문의.eml` | **non-code, 유일** | medium |
| 20 | 차감이 자동인지에 대해 주문팀과 정산팀의 인지가 서로 다르다 (2026-06 킥오프) | `minutes/2026-06-18_정산정정_자동화_킥오프.md` | **non-code, 상충** | medium |
| 21 | 레지스트리에 `CANCEL_RECON_QUEUE` 의 consumer 가 `TODO` 로 비어 있다 | `registry/services.yaml` | **non-code, 부재** | medium |
| 22 | 업무절차 v1.1 은 「월 1회 이상 대기 건 확인」을 규정하지만 수행 기록이 없다 | `policy/정산_정정_업무절차_v1.1.md` | **non-code, 상충** | medium |
| 23 | `SF-4512`(Quartz 스케줄 등록)는 2023-04-24 생성 후 3년 넘게 To Do, 담당자 없음 | `sprints/tickets_2026-S17.csv` | **non-code, 데이터** | **hard** |
| 24 | 2024 검토안의 보류 사유가 「처리량이 많지 않음」 — 적체 데이터와 정면으로 어긋난다 | `_archive/2024_정산정정_자동화_검토안_초안.md` | **non-code, 상충** | medium |

**확장 후: 26 fragments · 18 from non-code · 4 provable only by absence (#6, #11b, #17, #21) ·
7 mutually contradicting sources (#11, #12, #15, #20, #22, #24, 그리고 v0.3 vs v1.1 절차서)**

문서 유형(회의록·인수인계·회고·채팅·메일·레지스트리·스프린트·절차서 버전 병존·아카이브)은
실무에서 일반적으로 쓰이는 구성을 따른 것이며, 내용은 전부 가상이다.

### ⚠️ 최대 함정 — #11b

위키가 "틀렸다"고 결론내면 **절반만 맞다.** 위키는 `OrderCancelServiceV1` 을
정확하게 기술하고 있으며, 그 클래스는 지금도 저장소에 존재하고 정산 여부를
확인해 `IllegalStateException` 을 던진다.

문제는 **아무도 그것을 호출하지 않는다**는 것이다. 2.8.0(2023-04)에서
`service.OrderCancelService` 로 교체되었고, `@Deprecated` 주석에 "배치에서 참조
가능성이 있어 남겨둠"이라 적혀 있다 — 실제 참조는 없다.

코드 검색만으로는 "정산 확인 로직이 있다"는 **정반대 결론**에 도달한다.
정확한 답은 두 클래스의 존재와 호출 관계를 모두 확인해야 나온다.

## Known unknowns (F4) — the reef must say it cannot determine these

| Question | Why undeterminable |
|---|---|
| 4,127건 중 실제로 수기 정정된 건이 있는가 | `CANCEL_RECON_QUEUE.STATUS` 는 PENDING 뿐. 정산팀이 시스템 밖에서 처리했는지 확인할 방법이 없음 |
| `SETTLEMENT_ADJUSTMENT` 테이블이 존재하는가 | `CancelReconciler` 가 INSERT 하지만 어떤 마이그레이션에도 정의가 없음 |
| 파트너별 계약 수수료율 | `PARTNER_CONTRACT` 테이블은 V3 에 정의돼 있으나 데이터를 알 수 없고, 읽는 `PartnerContractRepository` 는 호출자가 없다. `SettlementItemProcessor` 는 `DEFAULT_FEE_RATE = 0.12` 를 모든 파트너에 적용한다 |
| 정산 배치가 실제로 매일 도는가 | Quartz 설정만 있고 실행 이력이 없음 |

## Second finding (S5) — 되돌림 패턴과 무관

**(1) 용어 충돌:** 주문 도메인의 `SANGPUM_CD` 와 재고 도메인의 `sku` 가 동일 개념이나
`inventory-api` 는 `ORD_NO` 를 보관하지 않는다. 주문↔재고 추적은 조인이 불가능하며,
이 사실은 `inventory-api/README.md` 에만 적혀 있다.

**(2) 복제 후 분기한 유틸:** `DateUtil.settlementBaseDate()` 가 두 저장소에
각각 존재하며 동작이 다르다. `order-service` 판은 무조건 전일을 반환하고,
`settlement-batch` 판은 02시 이전 수동 실행 시 **전전일**을 반환한다.
settlement-batch 쪽 주석에 "order-service 의 DateUtil 을 복사해 온 뒤 고쳤다"고
적혀 있다. 두 파일을 나란히 놓고 봐야만 드러난다.

**(3) 테스트 공백:** `OrderCancelServiceTest` 에 `// TODO 정산 완료 주문 취소
케이스 테스트 필요` 가 남아 있고, CI 워크플로는 테스트 단계 자체가 주석 처리되어
있다 (`2023-05-11`). 3년간 테스트가 돌지 않았다.
