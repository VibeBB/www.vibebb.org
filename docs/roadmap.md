# VibeBB family roadmap

[English](#english) | [日本語](#日本語)

## English

These are directions the family intends to take once the technology and
the safety story are ready. Nothing here is implemented; each item lists
what blocks it today and what we watch to decide when to start.

### 1. On-target debugging for firmware and fpga

**Goal.** firmware-agent and fpga-agent attach to real hardware through a
debug probe, read state, and use what they see in the fix loop.

- **Probes and tools to support:** SWD and JTAG through OpenOCD,
  probe-rs and pyOCD for microcontrollers; openFPGALoader for FPGA
  configuration and JTAG access.
- **Boundaries.** Reading registers, memory, RTT / semihosting output and
  breakpoints is in scope. Writing flash, erasing, fuse or option-byte
  changes and FPGA configuration stay human-confirmed and host-only, as
  today: no gate or MCP tool drives programming hardware.
- **Evidence.** Probe output is advisory evidence recorded with the probe
  serial, tool version and target image sha256. Like GDB output today, it
  never changes a deterministic verdict.
- **Why not yet.** The agents run in network-less containers; USB
  passthrough into Docker, probe permissions (udev) and a per-host
  allow-list need a design that keeps an agent from touching hardware it
  was not given.
- **Watch.** probe-rs (target coverage, RTT, DAP API stability), OpenOCD
  releases, pyOCD CMSIS-Pack support, openFPGALoader board coverage.

### 2. Vision-informed, any-angle placement and routing

**Goal.** circuit and mechanical agents turn insights from vision reviews
and impressions into layout changes, including placement and routing that
are not limited to 90° and 45° steps.

- **Today.** Placement is rule-based and routing uses Freerouting, which
  produces orthogonal and 45° tracks. Mechanical layout follows the brief.
- **Why not yet.** Free angles make placement and routing a far larger
  combinatorial search. LLMs cannot search it reliably on their own, and
  their geometric reasoning is not yet dependable enough to place parts.
- **Intended split.** The LLM proposes intent (which part should be near
  which, which nets are critical, where heat or noise matters); a
  deterministic optimiser (constraint, mixed-integer or any-angle router)
  produces geometry; the existing DRC, ERC and DFM gates judge it. The
  insight ledger of VRP v2 records each idea as a hypothesis with its
  outcome, so ideas are tried, adopted or rejected with evidence.
- **Shadow mode first.** A future optimiser runs next to the current flow
  and its results are compared, not shipped, until it beats the baseline
  on the gates and on measured hardware.
- **Watch.** Any-angle and topological routers, learning-based placement
  (reinforcement learning and graph models for macro and standard-cell
  placement), LLM plus solver hybrids, and KiCad's routing and API
  roadmap.

### 3. Listening review for bard

**Goal.** Songs and product sound cues are reviewed by listening, in the
same record format as vision reviews.

- **Why not yet.** The agents' model providers do not yet offer an
  audio-input path that VibeBB can call with the same provenance as image
  review. Until then bard checks cues with deterministic rules
  (frequency range, duration, SPL limits) and documents what was not
  heard.
- **Watch.** Audio-input support in the model providers configured for
  OpenHands.

## 日本語

技術と安全の仕組みが整った時点で、familyとして取り組む予定の方向です。
ここに書いたものは、まだ実装していません。項目ごとに、今は何が障害になって
いるか、始める時期を決めるために何を見続けるかを書いています。

### 1. firmwareとfpgaの実機デバッグ

- **目標**: firmware-agentとfpga-agentが、デバッグプローブで実機につなぎ、
  状態を読み、見えたことを修正のループに使えるようにします。
- **使う道具**: マイコンにはSWDとJTAG（OpenOCD、probe-rs、pyOCD）を使います。
  FPGAの書き込みとJTAGにはopenFPGALoaderを使います。
- **範囲**: レジスタ、メモリ、RTTやsemihostingの出力の読み取りと、ブレーク
  ポイントは範囲に入れます。flashの書き込み、消去、fuseやoption byteの変更、
  FPGAの書き込みは、今と同じく人が確認し、hostだけで行います。ゲートやMCPの
  ツールから書き込み装置を動かすことはしません。
- **証拠の扱い**: プローブの出力は参考情報として記録します。記録には、プローブの
  シリアル番号、ツールの版、対象imageのsha256を含めます。今のGDBの出力と
  同じく、決定論の判定は変えません。
- **まだ実装しない理由**: agentはネットワークの無いコンテナで動いています。
  Dockerへの USB の受け渡し、プローブの権限（udev）、hostごとの許可リストを、
  agentが渡されていない機器に触れない形で設計する必要があります。

### 2. visionで得たひらめきを使う、任意角の配置と配線

- **目標**: circuitとmechanical agentが、vision reviewやimpressionで得た
  ひらめきを、配置や配線の変更に反映できるようにします。配置と配線の角度は、
  90°や45°刻みに限りません。
- **今の状態**: 配置はルールに基づいて行い、配線はFreeroutingで行っています。
  Freeroutingの配線は、直交か45°です。
- **まだ実装しない理由**: 角度を自由にすると、探す組み合わせが桁違いに
  増えます。LLMだけではこれを確実に探せず、幾何の推論も部品の配置を任せられる
  水準にまだありません。
- **想定している分担**: LLMは意図を出します（どの部品を近くに置くか、どのnetが
  重要か、熱やノイズが問題になる場所はどこか）。形は決定論の最適化ソルバ
  （制約ソルバ、混合整数計画、任意角の配線器）が作ります。判定は今のDRC、ERC、
  DFMのゲートが行います。VRP v2のひらめき台帳は、それぞれの案を仮説として記録し、
  結果（採用、却下）を証拠と一緒に残します。
- **まず並走で試す**: 将来の最適化ソルバは、今の流れと並べて動かし、結果を
  比べるだけにします。ゲートと実機の測定で今のやり方を上回るまでは、成果物には
  使いません。

### 3. bardの「耳で聴く」レビュー

- **目標**: うたと製品の効果音（cue）を、耳で聴いてレビューします。記録の形は
  vision reviewと同じにします。
- **まだ実装しない理由**: VibeBBが使うモデルには、画像のレビューと同じ出典の
  記録を付けて呼べる、音声入力の経路がまだありません。それまでbardは、cueを
  決定論のルール（周波数の範囲、長さ、音圧の上限）で確かめ、聴いていないことを
  記録に明記します。
