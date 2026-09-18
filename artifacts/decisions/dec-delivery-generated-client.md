---
id: "DEC-DELIVERY-GENERATED-CLIENT"
type: "decision"
title: "Committing a Generated Order Client"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Reconstructed on 2026-09-19 by reading every file in delivery-bff (11 files, no node_modules), the 2022 order spec the client names as its source, the code-derived live surface and its extraction metadata, and order-service's OrderController, OrderCancelService, OrderStatus and CancelReason. No architecture decision record, ticket or thread describing this decision exists anywhere in the reef — the entire written trace is one four-line javadoc in src/orderClient.ts. This artifact therefore records an inferred decision, not a minuted one. If SF-4901 or any codegen runbook surfaces, it supersedes the Rationale section. Goes stale if delivery-bff gains a codegen script, a lockfile, a CI workflow, or a regenerated src/generated/orderApi.ts."
freshness_triggers:
  - "delivery-bff:package.json"
  - "delivery-bff:src/generated/orderApi.ts"
  - "delivery-bff:src/orderClient.ts"
  - "delivery-bff:tsconfig.json"
  - "order-service:build.gradle"
  - "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "sellflow-docs:apis/order/openapi.json"
  - "sellflow-docs:apis/order/openapi.meta.json"
known_unknowns:
  - "The rationale itself. No ADR, ticket, minute, Slack thread or README line in the reef says why a generated client was committed rather than regenerated on build or written by hand. Everything in the Rationale section below is inferred from the artefacts and is labelled as such."
  - "Who took the decision. The repo has no CODEOWNERS and no author metadata; the registry records delivery-bff's owner_team as TODO with the note 물류팀 이관 논의 중 (2025-11~), 확정 전 ('transfer to the logistics team under discussion since 2025-11, not yet confirmed'), while package.json and README claim 커머스본부 물류팀."
  - "What SF-4901 says, who filed it and when. src/orderClient.ts names it as the regeneration ticket and marks it 미착수 (not started). A grep of all five repos and the whole of sources/ finds the string only in that file and in the reef's own extracted delivery spec; sources/context/tickets/ does not hold it."
  - "Whether the file was hand-edited after generation. Three of its assumptions — the /api/v1 prefix, the CancelResponse shape and the OrderStatus union — cannot be emitted by any generator from the spec the header names. Either it was edited despite its own 직접 수정하지 마세요 banner, or it came from an earlier spec this reef does not hold. Both readings fit the artefacts equally."
  - "Whether openapi-typescript-codegen 0.23.0 was ever installed anywhere. It is not in dependencies or devDependencies, there is no lockfile and no node_modules, and no script, Makefile or CI job invokes it. The generator may have been run from a developer machine, a different repo, or not at all."
  - "Whether the same client was generated for other consumers. The 2022 spec carries the contact 커머스본부 주문팀 / order-dev@sellflow.co.kr, which suggests it was published beyond delivery-bff, but delivery-bff is the only consumer in this reef."
  - "Why the checked-in client was never deleted once it became clear nothing called it. requestCancel has no caller in the repo and no importer in any other repo; the file is still compiled by tsc on every build."
tags:
  - "adr-reconstructed"
  - "code-generation"
  - "contract-drift"
  - "delivery"
  - "openapi"
  - "orphaned-component"
aliases:
  - "orderApi.ts decision"
  - "checked-in generated client"
  - "생성 클라이언트 커밋"
relates_to:
  - type: "constrains"
    target: "[[API-DELIVERY]]"
  - type: "depends_on"
    target: "[[API-ORDER]]"
  - type: "refines"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "depends_on"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[RISK-DELIVERY]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "ORDER_API_BASE declared here and read by no file in src/."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "Describes the service only as a carrier proxy; says nothing about an order client."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "Two scripts, build and start. No codegen dependency, no test runner."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "The hand-written client for the other upstream, for comparison."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "The checked-in artefact: header, eslint-disable, status union, 409 throw."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "One route; does not import orderClient."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "The only written trace of the decision — the SF-4901 deferral comment."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tests/deliveryStatus.test.ts"
    notes: "expect(true).toBe(true); imports vitest, which is not a declared dependency."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tsconfig.json"
    notes: "include: [\"src\"], strict: false — the generated file is compiled, unchecked."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "@RequestMapping(\"/orders\"), ResponseEntity<Void>."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "Post-SF-2287 guard: CHWISO and BANPUM only."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.json"
    notes: "The 2022-11-04 springdoc artefact named in the client's header."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.meta.json"
    notes: "Records that springdoc-openapi is no longer a declared dependency, so no fresh spec can be produced."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "delivery-bff entry: owner_team TODO, Node 16, db none."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The policy repeal the client did not follow."
notes: "This ADR is reconstructed. Its Context and Consequences are evidenced; its Rationale is inference, and says so. The contract itself — assumption by assumption, with verdicts — lives in CON-ORDER-DELIVERY and is deliberately not restated here."
---

# Committing a Generated Order Client

## Context

delivery-bff talks to two upstreams and treats them in two entirely different ways.

The carrier is consumed by hand. `src/deliveryStatus.ts` is fourteen lines of `axios` with a declared `DeliveryStatus` interface, a 3s timeout, three attempts and a backoff, written by a person and maintained by a person → delivery-bff:src/deliveryStatus.ts

order-service is consumed through a machine artefact. `src/generated/orderApi.ts` opens with a banner that is unambiguous about what it is and how it should be treated:

```ts
/* eslint-disable */
/**
 * 자동 생성 파일입니다. 직접 수정하지 마세요.
 *
 * generator: openapi-typescript-codegen 0.23.0
 * source:    order-service-openapi.json
 * generated: 2022-11-08T04:12:33Z
 */
```

"This is an auto-generated file. Do not edit it directly." → delivery-bff:src/generated/orderApi.ts

Two facts about that banner matter more than its content. It is dated, so the artefact carries its own age. And it is checked into `src/`, inside `tsconfig.json`'s `include`, so it is compiled and shipped on every build like any hand-written module → delivery-bff:tsconfig.json

**There is no record of why.** No ADR exists in this reef for delivery-bff. `sources/context/decisions/` is an empty directory. The README is four lines and describes only the carrier proxy — 배송사 API를 프록시한다 ("it proxies the carrier API") — and never mentions order-service, a client or a generator → delivery-bff:README.md, sellflow-docs:context/registry/services.yaml

The whole written trace of the decision is one javadoc in the wrapper that imports the file:

```ts
/**
 * 주문 API 래퍼.
 * 생성 클라이언트가 2022 스펙 기준이라 상태코드 목록이 현재와 다르다.
 * 재생성은 SF-4901 에서 다루기로 함. (미착수)
 */
```

"Order API wrapper. The generated client is based on the 2022 spec, so its status-code list differs from today's. Regeneration will be handled in SF-4901. (Not started.)" → delivery-bff:src/orderClient.ts

That comment is the decision record. It establishes three things in four lines: somebody knew the file was stale, somebody decided not to fix it now, and somebody filed a ticket that has not moved. It does not establish why the file was committed in the first place, and nothing else in the reef does either. This artifact is therefore an ADR for a decision inferred from artefacts, not minuted by anyone. The inference is confined to the Rationale section and marked there.

## Decision

Generate a TypeScript client from order-service's OpenAPI document once, commit the output to `src/generated/`, and consume it through a thin hand-written wrapper — with no regeneration step in the build, no version pin on the spec, and no contract test on either side.

Reconstructed from the artefacts, the decision has four separable parts:

- **Generate rather than hand-write.** The cancel call goes through generated types (`OrderStatus`, `CancelRequest`, `CancelResponse`) while the carrier call, in the same repo and the same week's codebase, uses a hand-declared interface. The two upstreams were treated by two different policies → delivery-bff:src/generated/orderApi.ts, delivery-bff:src/deliveryStatus.ts
- **Commit the output rather than produce it at build time.** `package.json` declares exactly two scripts, `"build": "tsc"` and `"start": "node dist/index.js"`. `openapi-typescript-codegen` appears in neither `dependencies` (`axios`, `express`) nor `devDependencies` (`typescript`, `@types/express`) → delivery-bff:package.json
- **Wrap rather than call directly.** `requestCancel` in `orderClient.ts` adds nothing except the wrapping — it builds a `CancelRequest` and forwards. Its one genuine contribution is the comment quoted above, which is where the staleness is acknowledged → delivery-bff:src/orderClient.ts
- **Defer regeneration indefinitely once the drift was noticed.** The comment names SF-4901 and marks it 미착수. The ticket is not in `sources/context/tickets/`, and the artefact is unchanged → delivery-bff:src/orderClient.ts

The date the decision was taken is the date in the header, 2022-11-08 — four days after the spec it names was generated (2022-11-04), and 164 days before [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] repealed the business rule the file hard-codes → delivery-bff:src/generated/orderApi.ts, sellflow-docs:apis/order/openapi.json, sellflow-docs:context/tickets/SF-2287.md

## Key Facts

- The artefact is 1,411 days old as of 2026-09-19, and carries its own generation timestamp `2022-11-08T04:12:33Z` plus the generator name and version, `openapi-typescript-codegen 0.23.0` → delivery-bff:src/generated/orderApi.ts
- **No regeneration step exists anywhere.** `package.json` has two scripts, `build` and `start`, and four declared packages in total; no generator, no `openapi` script, no `prebuild` hook, no Makefile in the repo → delivery-bff:package.json
- **No CI exists either.** A full file listing of delivery-bff returns 11 files and no `.github` directory, no Dockerfile, no lockfile and no `node_modules`. order-service carries six workflow files and settlement-batch three, so the absence is specific to this repo, not an estate-wide convention → delivery-bff (full repo listing), order-service:.github/workflows/ci.yml
- **Nothing could have caught the drift.** The one test file asserts `expect(true).toBe(true)` under the Korean title 재시도 후 마지막 저장 값을 반환한다 ("returns the last stored value after retries"), and it imports `vitest`, which is not a declared dependency and has no runner in `scripts` → delivery-bff:tests/deliveryStatus.test.ts, delivery-bff:package.json
- **The generated file is exempted from the repo's own lint rules** by the `/* eslint-disable */` on its first line, so `eslint:recommended` and the repo's `no-console` rule never look at it → delivery-bff:src/generated/orderApi.ts, delivery-bff:.eslintrc.json
- **It is not exempted from the build.** `tsconfig.json` sets `include: ["src"]` and `strict: false`, so the file compiles into `dist/` on every `npm run build`, and does so unchecked — `res.json()` returning `any` satisfies the declared `Promise<CancelResponse>` only because strict mode is off → delivery-bff:tsconfig.json
- **The one business rule the client hard-codes came straight from the spec.** The 2022 document describes the cancel operation as 주문을 취소한다. 정산이 완료된 주문은 취소할 수 없다 ("cancels an order; an order whose settlement is complete cannot be cancelled") and its 409 as 취소 불가 상태 (배송완료·정산완료 등) ("a state in which cancellation is not possible — delivery complete, settlement complete, and so on"). The client turned that prose into control flow: `if (res.status === 409) throw new Error('정산이 완료된 주문은 취소할 수 없습니다.')` → sellflow-docs:apis/order/openapi.json, delivery-bff:src/generated/orderApi.ts
- **order-service stopped meaning that on 2023-04-21.** `OrderCancelService` now blocks exactly `CHWISO` and `BANPUM` under the comment 이미 취소되었거나 반품 프로세스로 넘어간 주문만 차단한다 ("block only orders already cancelled or moved into the return process"), so a 409 today means "already cancelled or returned" and never "settled". The client has asserted the opposite for 1,247 days → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- **Three of the client's assumptions cannot have come from the spec it names.** Verified directly against `sources/apis/order/openapi.json`: `/api/v1` occurs 0 times in the whole 30-path document, `JUMUN_WANRYO` occurs 0 times, and the `components.schemas` list has no `OrderStatus` entry at all (`OrderMst.sangtaeCd` is a bare `{"type":"string","description":"주문상태코드"}`). The spec does emit enums elsewhere — `taekBaeSaCd`, `chaenNelCd`, `gyeolJeCd`, `changgoCd`, `chulGoSangtae`, `baesongSangtae` all carry `enum` arrays — so enum emission was working and order status specifically was untyped → sellflow-docs:apis/order/openapi.json
- **The response type is wrong in both directions.** The spec's 200 for cancel is `OrderCancel {ordNo, chwisoIlsi, chwisoSayuCd, choriSangtae, bigo}`; the client declares `CancelResponse {ordNo, sangtaeCd}`; the live controller returns `ResponseEntity.ok().build()`, an empty body, on which the client's closing `return res.json()` throws → sellflow-docs:apis/order/openapi.json, delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- **The decision is now irreversible in its own terms.** Regeneration needs a current spec, and order-service can no longer emit one: `springdoc-openapi` is not a declared dependency in `build.gradle`, which is why the reef's extraction fell back to tier 3 (copy the 2022 file) and tier 4 (derive from controllers) → sellflow-docs:apis/order/openapi.meta.json
- **The deferral comment understates the drift by a factor it does not hint at.** It names the status-code list. It does not name the path prefix, the 409 policy, the response shape, the relative `fetch` URL, or the missing `Authorization` header that the spec's global `bearerAuth` requires → delivery-bff:src/orderClient.ts, delivery-bff:src/generated/orderApi.ts
- **Nothing calls any of it.** `src/index.ts` registers one route, `app.get('/delivery/:ordNo', ...)`, which calls `syncDeliveryStatus` and never imports `orderClient`; no other repo in `sellflow/repos` imports delivery-bff → delivery-bff:src/index.ts
- **The runtime configuration for the decision was written and then never wired.** `.env.template` declares `ORDER_API_BASE=http://order-service.internal`, and no file under `src/` reads it; the generated `fetch` uses a relative path instead → delivery-bff:.env.template, delivery-bff:src/generated/orderApi.ts

## Rationale

No recorded rationale exists. What follows is inference from the artefacts, offered as the most economical explanation of what is in the repo, and it should be read as such — the corresponding certainty is recorded in `known_unknowns`.

**Why generate at all.** In November 2022 order-service published a springdoc document with 30 paths and 32 operations (the reef's extraction record says 45; recounting the file gives 32, and the 45 could not be reproduced — see [[API-ORDER]]) and 32 schemas, complete with Korean field descriptions and six enums → sellflow-docs:apis/order/openapi.json. Against a contract that size, generating types is the cheap and conventional choice, and it is the choice a team would make precisely because the contract looks authoritative. The carrier, by contrast, published no spec at all — the reef holds none, and `deliveryStatus.ts` declares its own four-field interface — so hand-writing was the only option there. The split in treatment follows the availability of a document, not a judgement about the two upstreams.

**Why commit the output.** The evidence is circumstantial but consistent. There is no lockfile and no `node_modules`; the two runtime dependencies are pinned with carets; and the generator is not installed. A build step that regenerates would have made the build depend on order-service being reachable and on a generator version nobody pinned. Committing the output makes `npm run build` hermetic. That is a real benefit, and it is the standard argument for checked-in generated code. Its standard cost is exactly what happened here: the artefact stops tracking its source and nothing notices.

**Why wrap it.** `requestCancel` adds no behaviour. What it adds is a place to write a comment — and the comment it was eventually given is the only documentation the decision has. Whether the wrapper was created for that purpose or for the conventional reason (a seam to swap the generated call behind) cannot be determined; only the comment survives.

**Why defer regeneration.** The comment says 재생성은 SF-4901 에서 다루기로 함 ("regeneration will be handled in SF-4901"). Two conditions make the deferral rational at the moment it was written: the code was already unreachable, so the staleness cost nothing today; and, as the extraction metadata shows, regeneration was no longer a mechanical act — the provider could not produce a fresh spec, so SF-4901 was scoped as a regeneration but was in fact a rewrite → sellflow-docs:apis/order/openapi.meta.json. A ticket for a rewrite against a contract nobody can emit is a ticket that does not get picked up.

What cannot be inferred, and is not guessed at here: who decided, whether anyone reviewed the file's three non-derivable assumptions at the time, and whether the checked-in-generated-code policy was a team convention or a one-off. No CODEOWNERS file, no PR record and no convention document exists in the repo or in `sources/`.

## Consequences

**A file that claims a provenance it does not have.** The header names a generator, a version, a source document and a timestamp. Three of the file's contents cannot be produced by running that generator over that document, and the output shape argues the same way: `openapi-typescript-codegen` emits a `core/` directory with an `OpenAPI.BASE`, a shared `request()` helper and per-model files, not a single module with an inline `fetch` and a hardcoded relative path. The practical consequence is that **regenerating from `sources/apis/order/openapi.json` would not reproduce this file**, so any review that expects SF-4901 to be a no-op diff will be wrong. The assumption-by-assumption verdicts are tabulated in [[CON-ORDER-DELIVERY]] and are not repeated here.

**A repealed business rule preserved in a `throw`.** This is the consequence that connects the decision to the rest of the estate. SF-2287 removed the settled-order cancel block after CS logged 214 enquiries in one month; the fix was confirmed working three days after deployment. The checked-in client kept the abolished rule alive as a user-facing Korean sentence, in code, for 1,247 days and counting. Nothing in the client reads the 409 body that order-service actually sends — `GlobalExceptionHandler` returns `Map.of("message", e.getMessage())` with the real reason, and the client discards it in favour of its own → [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]], order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java

**Drift with no detector.** The decision removed the only mechanism that would have surfaced the drift automatically — regeneration — and replaced it with nothing. The table below lists every mechanism that could in principle have caught it, and its state in this repo:

| Mechanism | State in delivery-bff | Source |
|---|---|---|
| Regeneration on build | Absent. Scripts are `build` and `start` only | delivery-bff:package.json |
| Generator as a dev dependency | Absent. devDependencies are `typescript`, `@types/express` | delivery-bff:package.json |
| CI workflow | Absent. No `.github` directory in the repo | delivery-bff (repo listing) |
| Contract test | Absent on both sides | delivery-bff:tests/, order-service (no contract test) |
| Unit test touching the client | Absent. One test, `expect(true).toBe(true)` | delivery-bff:tests/deliveryStatus.test.ts |
| Type checking strict enough to complain | `strict: false` | delivery-bff:tsconfig.json |
| Lint | Explicitly disabled in the file's first line | delivery-bff:src/generated/orderApi.ts |
| Spec version pin | Absent. Neither side pins a version | sellflow-docs:apis/order/openapi.meta.json |

**The drift is one-directional and unrecoverable from the provider.** `springdoc-openapi` left `build.gradle` at some point after 2022, so the contract cannot be refreshed at source. Whoever eventually picks up SF-4901 must either regenerate against a 2022 document whose other 29 paths no longer have handlers, or write the client by hand against `sources/apis/order/openapi.code-derived.json` — the surface derived from today's controllers → sellflow-docs:apis/order/openapi.meta.json

**The damage today is zero and the trap is real.** Nothing calls `requestCancel`, so no customer has ever been told that a settled order cannot be cancelled by this code path. What the decision leaves behind is a loaded import: the next engineer who wires `requestCancel` into a route inherits, in one line, a path that 404s, a URL a Node process cannot resolve, a success branch that throws on an empty body, a status union missing three live values and inventing one, and an error message asserting a policy the company deliberately abolished. The wrapper's comment will tell them that only the status list is stale → [[RISK-DELIVERY]]

**One more instance of the estate's dominant failure mode.** A written artefact describing behaviour the code no longer has, left standing because nothing owns it and nothing tests it. The 2021 Confluence cancel policy, the 2022 OpenAPI document and this client are the same shape of problem in three different media → [[RISK-SELLFLOW-DOC-DRIFT]]

**What would have to change for the decision to be safe rather than reversed.** Not regeneration alone. A checked-in client is defensible when something re-derives it and fails loudly on a diff. Here that would mean a pinned generator in `devDependencies`, a `codegen` script, a CI job that regenerates and fails on a dirty tree, a pinned spec version on the provider side, and a provider able to emit that spec at all. The last item is the blocker, and it is not in delivery-bff's control.

## Related

- [[SYS-DELIVERY]] — the service that holds the checked-in artefact
- [[CON-ORDER-DELIVERY]] — the contract itself, assumption by assumption, with verdicts
- [[API-ORDER]] — the 30-path 2022 spec against the 2-endpoint reality, and why it cannot be regenerated
- [[API-DELIVERY]] — what delivery-bff exposes, and the outbound call it never makes
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — the decision that invalidated this one 164 days later
- [[PROC-ORDER-CANCEL]] — the cancellation rule as it works today
- [[RISK-DELIVERY]] — the stale-client exposure in full
- [[RISK-SELLFLOW-DOC-DRIFT]] — the estate-wide pattern this is one instance of
