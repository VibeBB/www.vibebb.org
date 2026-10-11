# Agent handoffs in the VibeBB family

[English](#english) | [日本語](#日本語)

## English

This page is the map of artifacts that actually flow between the eleven
VibeBB agents today. Every arrow is backed by code on the `main` branch of
the receiving repository: an import kind, a liaison file, or a contract
field. Planned handoffs are listed separately and move into the map only
when their code lands.

### Rules every handoff follows

- **Artifacts, not imports.** Sisters exchange JSON files in the shared
  workspace. No agent imports another agent's code or edits another
  agent's inputs; a change goes back as a request.
- **Pinned by hash.** The receiver records the producer artifact's
  `sha256`. A changed upstream file is detected, never silently reused.
- **Gates decide.** Only the receiver's deterministic gates issue a
  verdict. Impressions, vision reviews and songs travel with the artifact
  as advisory evidence and never change a verdict.
- **Unknown fails closed.** A missing, unparseable or stale input makes
  the dependent gate `unknown`, which fails the design verdict.
- **Nothing waits.** Records reference only records that already exist;
  no agent blocks waiting for a sister. Missing input is recorded as
  unresolved and work continues (VRP, below).

### Design artifacts

```mermaid
flowchart LR
  CIR["electrical-circuit"]
  MECH["mechanical"]
  WIRE["wire"]
  FW["firmware"]
  FPGA["fpga"]
  SIM["simulation"]
  DASH["dashboard"]
  UX["UX-creator"]
  BARD["bard"]

  CIR -- "*.firmware.json, pinmap" --> FW
  CIR -- "*.firmware.json" --> FPGA
  CIR -- "*.connectivity.json" --> WIRE
  CIR -- "*.connectivity.json" --> SIM
  CIR -- "*.board-geometry.json" --> MECH
  MECH -- "*.mech-board-envelope.json" --> CIR
  MECH -- "*.envelope.json + keepouts" --> WIRE
  MECH -- "*.envelope.json" --> SIM
  WIRE -- "*.contract.json" --> SIM
  FW -- "*.fw-power.json" --> SIM
  BARD -- "song / cue artifacts" --> SIM
  BARD -- "cues.json" --> FW
  FPGA -- "*.fpga-regmap.json" --> FW
  FW -- "*.fw.json + faults" --> DASH
  UX -- "*.ux.json" --> DASH
  UX -- "*.ux.json" --> WIRE
  UX -- "*.ux.json CMF" --> MECH
  UX -- "cue request" --> BARD
```

| From | To | Artifact | Where the receiver binds it |
| --- | --- | --- | --- |
| circuit | firmware, fpga | `*.firmware.json` (pins, buses, clocks) | firmware / fpga contract import |
| circuit | wire, sim | `*.connectivity.json` (nets, connector pins) | wire `imports`, sim `ImportRef.system` |
| circuit | mechanical | `*.board-geometry.json` (outline, holes, part heights) | mech `board_geometry` gate |
| mechanical | circuit | `*.mech-board-envelope.json` (height limits, side openings) | circuit `circuit.enclosure` gate |
| mechanical | wire, sim | `*.envelope.json` (fixing points, 3D keep-out boxes) | wire `imports` + `route_keepout` gate, sim `ImportRef.system` |
| wire | sim | `*.contract.json` | sim `ImportRef.system` |
| firmware | sim | `*.fw-power.json` (supply-net draw) | sim `import:firmware:<net>` PDN load |
| bard | sim | song and cue artifacts | sim `ImportRef.system` |
| bard | firmware | `cues.json` playback table | firmware `fw_cues.h` generation |
| fpga | firmware | `*.fpga-regmap.json` | firmware `fw.fpga_regmap` gate |
| firmware | dashboard | `*.fw.json` with `firmware_sha256`, fault table | dashboard `firmware_contract`, `service.faults` gate |
| UX | dashboard, wire | `*.ux.json` | contract import |
| UX | mechanical | `*.ux.json` CMF part | mech `appearance_cmf` gate |

### Simulation requests and answers

```mermaid
flowchart LR
  SIM["simulation"]
  REQ["mechanical, circuit, wire, fpga"]
  PE["production-engineering"]
  REQ -- "*.sim-request.json" --> SIM
  SIM -- "*.sim-response.json (margin, corrective guidance)" --> REQ
  SIM -- "*.sim-response.json predictions" --> PE
```

| From | To | Artifact | Where the receiver binds it |
| --- | --- | --- | --- |
| mechanical | simulation | drop, vibration and IP study request | sim ruggedness analysis |
| circuit | simulation | per-part dissipation, electrolytic lifetime | sim thermal and Arrhenius analyses |
| wire | simulation | supply loops (PDN) | sim PDN analysis |
| fpga | simulation | package thermal facts | sim thermal analysis |
| simulation | requester | hash-pinned response with margins and corrective guidance on failure | each requester's sim-response gate |
| simulation | production-engineering | predicted characteristic values | prodeng `sim.prediction` gate |

A failing simulation answer goes straight back to the requesting agent as a
change driver; it does not route through UX-creator.

### Manufacturing

```mermaid
flowchart LR
  PE["production-engineering"]
  CIR["electrical-circuit"]
  MECH["mechanical"]
  WIRE["wire"]
  FW["firmware"]
  FPGA["fpga"]
  UX["UX-creator"]
  DASH["dashboard"]

  CIR -- "brief, connectivity, circuit-assembly, circuit-handling" --> PE
  MECH -- "mech-envelope, mech-appearance" --> PE
  WIRE -- "wire-contract, wire-crimp, wire-formboard" --> PE
  FPGA -- "fpga-production" --> PE
  FW -- "fw-production" --> PE
  UX -- "ux-contract (oobe.in_box)" --> PE
  PE -- "factory-test-spec.json" --> FW
  PE -- "factory-test-spec.json" --> CIR
  PE -- "factory-test-spec.json" --> DASH
```

| From | To | Artifact | Where the receiver binds it |
| --- | --- | --- | --- |
| circuit, mechanical, wire, UX | production-engineering | brief, connectivity, envelope, contracts | prodeng `ImportRef.kind` |
| circuit | production-engineering | `*.circuit-assembly.json` (per-side SMT/THT), `*.circuit-handling.json` (MSL, reflow peak, ESD) | prodeng `circuit.assembly`, `circuit.handling` gates |
| mechanical | production-engineering | `*.mech-appearance.json` (cosmetic criteria, limit samples) | prodeng `appearance.limit_samples` gate |
| wire | production-engineering | `*.wire-crimp.json`, `*.wire-formboard.json` | prodeng `wire.crimp`, `wire.formboard` gates |
| fpga, firmware | production-engineering | `*.fpga-production.json`, `*.fw-production.json` (gated image sha256, loader options) | prodeng `fpga.programming`, `firmware.programming` gates |
| UX | production-engineering | `oobe.in_box` in the UX contract | prodeng `packaging.oobe` gate |
| production-engineering | firmware | `factory-test-spec.json` | firmware `fw.ftm` gate |
| production-engineering | circuit | `factory-test-spec.json` test nets | circuit `circuit.dft` gate |
| production-engineering | dashboard | `factory-test-spec.json` | dashboard `factory.spec` gate and factory-test panel |

Checks that stay inside one agent (circuit derating, second sources and
fabrication stackup; fpga corner guard band and power estimate) are gates,
not handoffs, and are not drawn here.

### Coordination: liaison (SLP v2) and records (VRP)

```mermaid
flowchart LR
  UX["UX-creator"] -- "*.ux-request.json" --> S["10 sisters"]
  S -- "*.ux-response.json" --> UX
  S -. "decisions, impressions, vision reviews" .-> DOC["document"]
  S -. "artifacts as sources" .-> DOC
```

- **Sister Liaison Protocol (SLP v2).** UX-creator writes requests into
  each sister's `liaison/` inbox; the sister answers with a response file
  that its guard hook protects. UX-creator detects circular request
  chains, so a request cannot bounce forever. See
  [liaison-protocol.md](liaison-protocol.md).
- **VibeBB Record Protocol (VRP v2).** Each agent appends hash-chained
  decisions, stage impressions, vision reviews, read receipts, insight
  status, blind-review reconciles and bard song receipts to
  `observations/<agent>/*.jsonl`. A reader must answer a sister impression
  it read, a disputed concern cannot be raised again on unchanged bytes,
  and UX-creator's family hub shows concerns that cycle between agents.
  The Stop hook refuses to end a session that owes a record at most twice,
  then allows the stop and records what was missing. See
  [records-protocol.md](records-protocol.md).
- **document** reads sister agents, their artifacts and their records as
  typed sources for reports and launch material.

### Change control (ECR / ECO)

A sister that must change a file other agents pin by sha256 does not
re-pin silently. It writes an engineering change request (`*.ecr.json`)
and order (`*.eco.json`); UX-creator's `ux change` / `ux_change_check`
finds every consumer pin, rejects stale pins and omitted consumers, and
requires passing gate evidence and dispositioned AI review findings. AI
findings can block a change but never pass one. There is no human
approval step.

### Planned handoffs

None. Every handoff from the previous plan is on `main` and drawn above.
Inputs that enter VibeBB from outside (supplier data, field returns,
regulatory updates) are out of scope for this map.

## 日本語

このページは、11のVibeBB agentの間で、今実際に流れている成果物の地図です。
矢印はすべて、受け取る側のrepositoryの `main` にあるコード（取り込みの種類、
liaisonのファイル、contractの欄）で確かめたものです。計画中の受け渡しは別の
表に分けてあり、コードがmergeされた時点で図に移します。

### すべての受け渡しが守る決まり

- **コードではなく成果物で渡す**: 姉妹agentは、共有workspaceのJSONファイルを
  やり取りします。他のagentのコードをimportしたり、入力を書き換えたりはしません。
  変更が必要な時は依頼として返します。
- **hashで固定する**: 受け取る側は、作成側の成果物の `sha256` を記録します。
  元のファイルが変われば検出され、黙って使い回されることはありません。
- **判定はゲートだけが行う**: 判定を出すのは、受け取る側の決定論ゲートだけです。
  impression、vision review、うたは参考情報として成果物と一緒に渡り、判定を
  変えることはありません。
- **不明は不合格として扱う**: 入力が無い、読めない、古い場合、その入力を使う
  ゲートは `unknown` になり、設計の判定は不合格になります。
- **待たない**: 記録が参照するのは、すでに存在する記録だけです。姉妹の返事を
  待って止まるagentはありません。足りない入力は未解決として記録し、作業を
  続けます。

図と表は英語の節を見てください。agentの間の記録（VRP v2）の決まりは
[records-protocol.md](records-protocol.md)にあります。以前「計画中」だった
受け渡しは、すべて `main` に入り、図と表に移しました。シミュレーションの不合格の
答えは、UXを通さずに依頼したagentへ直接戻ります。他のagentがsha256で固定して
いるファイルを変える時は、ECR/ECOを通します（人による承認はありません）。
外部からVibeBBに入る情報は、この地図の対象外です。
