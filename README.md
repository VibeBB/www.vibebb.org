# VibeBB — Vibe BreadBoarding

<img src="assets/vibebb-silkscreen.svg" alt="VibeBB — Vibe BreadBoarding (silkscreen logo)" width="320">

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/VibeBB/www.vibebb.org)

[English](#english) | [日本語](#日本語)

## English

VibeBB is "Vibe BreadBoarding": describe what you want to build in
natural language, and AI designs the circuit board, enclosure, and
firmware — with pass/fail decided by deterministic gates and
real-hardware evidence, not by vibes. This repository is the source of
the site at [vibebb.org](https://vibebb.org).

### What is VibeBB?

VibeBB aims to make AI the primary designer. The AI handles requirements
elicitation, part selection, circuit, board layout, enclosure, and
firmware design, manufacturing data generation, and iteration driven by
manufacturing and real-hardware feedback, while the human is involved as
the owner of the requirements and can also serve as a reviewer or make
direct fixes when needed.

### Scope

Mass-production quality and certification are not assumed; the goal is
to reach a working prototype as fast as possible.

### VibeBB — Vibe BreadBoarding

VibeBB is "Vibe BreadBoarding", modeled after Vibe Coding. It brings the
idea Andrej Karpathy expressed in his
[February 2025 post](https://x.com/karpathy/status/1886192184808149383) —
"fully give in to the vibes, embrace exponentials, and forget that the
code even exists" — and the "see stuff, say stuff, run stuff"
interactive loop into board, enclosure, and FW prototyping and
verification. As
[Collins Dictionary's 2025 Word of the Year](https://blog.collinsdictionary.com/language-lovers/collins-word-of-the-year-2025-ai-meets-authenticity-as-society-shifts/)
shows, the development experience of stating intent in natural language,
looking at the result, and returning the next instruction is spreading.

Just as the breadboard made it possible to "try as you think" without
soldering, with VibeBB you only state what you want in words, and the AI
drives part selection, circuit, board layout, enclosure, firmware, and
manufacturing data, while the human looks at the working prototype board
and fitting enclosure and gives feedback. Not drawing a schematic is the
counterpart of Vibe Coding's not reading code.

VibeBB does not mean that design and verification are light; it means
the heavy verification is hidden from human hands. Following
[Simon Willison](https://simonwillison.net/2025/Mar/19/vibe-coding/)'s
distinction between true Vibe Coding without reviewing the artifact and
AI-assisted work with review, VibeBB moves the review role from humans
to deterministic gates and real-hardware Evidence.

The experience loop is: **describe → AI designs and verifies with
deterministic gates → build and try → feed measurements into the next
design**. The basic cycle is to build first, check on real hardware, and
turn the next change immediately, rather than long desk study.

### Design principles

- Each stage produces machine-readable and visual projections, which SDK
  subagents/vision review on a best-effort basis. Findings go to the fix
  loop in natural text; reviews hold no pass/fail authority.
- Human review is optional. The default is for AI to run from
  requirements to manufacturing data end to end; what the user verifies
  is not schematics or artwork but whether the delivered board and
  enclosure actually work and fit.

### Recording design rationale

VibeBB does not rely on conversation logs or memory for "why the design
was made this way"; it stores the record in the same change as the
design input (git). Values chosen by the designer (AI) — parts,
placement, trace widths, silkscreen, stackup, design rules, net classes,
safety boundaries, mechanical dimensions, FW pin assignments — are
recorded with the reason for adoption, rejected alternatives, driving
requirements, and provenance.

Missed records are detected deterministically. Attributes that express
design decisions are classified as required or exempt; attributes in
neither class become `unclassified`. When a record no longer matches its
subject, it asks for the reason to be re-recorded.

As a result, even several revisions later or to a different reader, "why
this part", "why this trace width", "why this dimension" can be traced
afterwards. Design rationale explains reasons; it holds no pass/fail
authority. Pass/fail is decided by the deterministic gates and Evidence.

### Learning across conversations

VibeBB writes what it learns while working into OpenHands' persistent
memory and reads it at the start of the next conversation. Tendencies in
parts and part numbers frequently used in this repository, how
footprints and stackups are chosen, clearance and trace-width values
that actually passed, enclosure fastening and wall-thickness know-how,
points repeatedly flagged by silkscreen or review, and the user's design
preferences accumulate with use. As a result, proposals can start from
the conventions of previous designs without repeating the same
explanation each time.

This persistent memory is disabled by default. To use it, enable
Persistent memory in the OpenHands Local GUI settings. Memos assist the
working context; pass/fail is decided by the deterministic gates and
Evidence.

### Design flow

From requirements dialogue to manufacturing and real-hardware feedback,
the agents proceed in parallel, exchanging opinions and requests with
each other. Firmware is delegated to OpenHands' native software
development capability, and board, enclosure, and FW are designed and
verified in the same interactive flow.

### Difference from LLM-only CAD

The difference from LLM-only CAD is not that it produces the same
solution every time, but that the produced design can be re-verified
afterwards. Even if each run yields a different solution, that is fine
as long as it can be verified by deterministic measurement and
independent parser re-reading.

### Future vision — toward the era of "printing" boards at home

VibeBB's premise — cheap and fast manufacturing — can go further. If
printed electronics matures, boards could be made on the spot like a
home 3D printer, shortening "build and try" from days to tens of
minutes, and VibeBB truly approaches breadboard speed. The same applies
to enclosures, brackets, and mechanical parts; 3D printing/CNC quoting,
DFM, and ordering services become procurement channels on par with board
fabs.

Soberly speaking, in-home manufacturing of simple 1–2 layer boards is
already real (desktop milling, conductive ink printing). On the other
hand, high-density multilayer, plated through holes, high current, and
certified mass production remain the domain of professional fabs for the
time being. Technologies such as conductive filament, conductive paste,
and 3D-MID/LDS/IME are blurring the boundary between enclosure and
circuit, but they presuppose verification of conductivity,
solderability, contact resistance, and durability.

VibeBB builds this future in from the start.

- **DRC with mechanical/material profiles**: minimum trace
  width/spacing, tool diameter, ink or filament resistivity and current
  capacity, via method, substrate, and curing/sintering conditions are
  verified in a separate profile within the same framework as
  fab-facing DRC.
- **Material-aware electrical analysis**: estimates trace resistance,
  voltage drop, and temperature rise from measured material data rather
  than assuming copper foil.
- **Hybrid manufacturing dispatch**: from the same design graph,
  generates a locally prototyped version buildable at hand and a
  conventional fab mass-production version when density or current
  exceeds requirements.
- **Closed-loop inspection**: feeds alignment, continuity, and
  resistance measurement results back into the design, and accumulates
  machine and material lot quirks as knowledge.
- **Room for structural electronics**: `Layout` is not fixed to planar
  rigid boards; a design graph that does not preclude future extension
  to non-planar circuits including enclosure surfaces and embedded
  wiring is preferred.

Beyond that lies personally fitted wearables that take a 3D body scan as
a mechanical constraint and produce non-planar, stretchable circuits on
the spot. For designs whose shape changes per person, an approach that
treats schematics or artwork drawings as the source of truth breaks
down. VibeBB regenerates manufacturing data each time with a design
graph containing requirements, constraints, and design rationale as the
source of truth precisely so the same mechanism can reach this
direction.

This is a vision-level future direction, not a promise about the current
implementation scope.

## 日本語

VibeBBは「Vibe BreadBoarding」――作りたいものを自然言語で伝えると、
AIが回路基板・筐体・ファームウェアを設計し、合否は決定論的ゲートと
実機エビデンスが判定します。このリポジトリは
[vibebb.org](https://vibebb.org)のサイトのソースです。

### VibeBBとは？

VibeBBはAIが主たる設計者となることを目指します。AIは要件のヒアリング、
部品選定、回路・基板レイアウト・筐体・ファームウェアの設計、製造データの
生成、製造・実機フィードバックを受けた反復までを担い、人間は要件の
オーナーとして関わり、必要に応じてレビュアーの役割や直接修正も担えます。

### 対象範囲

量産品質や認証を前提にせず、動く試作へ最短で到達することを目的とします。

### VibeBB — Vibe BreadBoarding

VibeBBは、Vibe Codingになぞらえた「Vibe BreadBoarding」です。Andrej Karpathyが
[2025年2月の投稿](https://x.com/karpathy/status/1886192184808149383)で示した
「完全にバイブスに身を委ね、コードの存在すら忘れる（fully give in to the vibes,
embrace exponentials, and forget that the code even exists）」という発想と、
「see stuff, say stuff, run stuff」の対話的なループを、基板・筐体・FWの試作と
検証へ持ち込みます。
[Collins英語辞典の2025年Word of the Year](https://blog.collinsdictionary.com/language-lovers/collins-word-of-the-year-2025-ai-meets-authenticity-as-society-shifts/)
が示すように、自然言語で目的を伝え、結果を見て、次の指示を返す開発体験は
広がっています。

ブレッドボード（BreadBoard）がハンダ付けなしに「考えながら試す」ことを可能に
したように、VibeBBではやりたいことを言葉で伝えるだけで、AIが部品選定、回路、
基板レイアウト、筐体、ファームウェア、製造データを進め、人間は動く試作基板と
収まる筐体を見てフィードバックを返します。回路図を描かないことは、コードを
読まないVibe Codingと対になる発想です。

VibeBBは、設計や検証が軽いという意味ではなく、重い検証を人間の手作業から
隠すという意味です。
[Simon Willison](https://simonwillison.net/2025/Mar/19/vibe-coding/)が区別した
生成物をレビューしない本来のVibe Codingとレビューを伴うAI活用を踏まえ、
VibeBBはレビューの役割を人間から決定論的ゲートと実機Evidenceへ移します。

体験のループは、**語る → AIが設計し決定論的ゲートで検証する → 作って試す →
測定結果を次の設計へ返す**です。長時間の机上検討よりも、まず作って実機で
確かめ、すぐ次の変更を回すことを基本サイクルにします。

### 設計原則

- 各工程で機械可読投影と視覚投影を生成し、SDKのsubagent／visionが
  best-effortでレビューします。所見は自然文で修正ループへ渡し、レビューは
  合否権限を持ちません。
- 人間レビューは任意です。既定はAIが要件から製造データまで走り切ることであり、
  ユーザーが確かめるのは回路図やアートワークではなく、届いた基板と筐体が
  実際に動き、収まるかどうかです。

### 設計根拠を残す

「なぜその設計にしたか」を会話ログや記憶に頼らず、設計入力（git）と同じ変更で
保存します。部品、配置、配線幅、シルク、stackup、design rule、net class、
安全境界、機構寸法、FWピン割当のように設計者（AI）が選んだ値は、採用理由、
却下した代替案、駆動している要求、出所とともに記録されます。

記録漏れは決定論的に検出します。設計判断を表す属性は必須と免除に分類され、
どちらにも分類されない属性は`unclassified`とします。対象と一致しなくなった
場合は理由の再記述を求めます。

これにより、数リビジョン後や別の人が見たときでも、「なぜこの部品なのか」
「なぜこの配線幅か」「なぜこの寸法か」を後から辿れます。設計根拠は理由の
説明であり、合否の権限は持ちません。合否は決定論的ゲートとEvidenceが
判定します。

### 会話をまたいで学習する

作業して分かったことをOpenHandsの永続メモリへ書き残し、次の会話の開始時に
読み込みます。このリポジトリでよく使う部品や型番の傾向、footprintやstackupの
選び方、clearanceや配線幅で実際に通った値、筐体の締結・肉厚の勘所、シルクや
レビューで毎回指摘される観点、ユーザーが好む設計の癖などが、使うほど
溜まっていきます。結果として、同じ説明を毎回しなくても、以前の設計の流儀を
引き継いだ提案から始められます。

この永続メモリは既定で無効です。利用するにはOpenHands Local GUIの設定で
永続メモリ（Persistent memory）を有効にしてください。メモは作業文脈の補助で
あり、合否は決定論的ゲートとEvidenceが判定します。

### 設計フロー

要件対話から製造・実機フィードバックまでを、各エージェントが互いに意見や
要望を出し合いながら並行して進みます。ファームウェアはOpenHands本来の
ソフトウェア開発能力へ委譲し、基板・筐体・FWを同じ対話的な流れで
設計・検証します。

### LLM-only CADとの違い

LLM-only CADとの違いは、毎回同じ解を出すことではなく、出た設計を後から
再検証できることです。実行ごとに解が異なっても、決定論的な実測と独立
parser再読込で検証できればよいとします。

### 将来展望 — 家庭で基板を「印刷」する時代へ

VibeBBの前提である「製造の安さと速さ」は、さらに先へ進む可能性があります。
プリンテッドエレクトロニクスが成熟すれば、家庭用3Dプリンタのように基板を
その場で作れるようになり、「作って試す」が数日から数十分へ短縮され、VibeBBは
本当にブレッドボードの速度に近づきます。同じことは筐体・ブラケット・機械部品に
も当てはまり、3Dプリント／CNCの見積・DFM・発注サービスは基板fabに並ぶ
調達経路になります。

冷静に見れば、単純な1〜2層基板の宅内製造はすでに現実です（卓上切削、
導電性インク印刷）。一方で高密度多層、メッキスルーホール、大電流、認証が
必要な量産は当面プロのfabが優位です。導電性フィラメント、導電性ペースト、
3D-MID/LDS/IMEのような技術は、筐体と回路の境界を曖昧にしつつありますが、
導電率、はんだ付け性、接触抵抗、耐久性の検証が前提になります。

VibeBBはこの未来を最初から織り込みます。

- **機械・材料プロファイル対応DRC**: 最小線幅・間隔、工具径、インクや
  フィラメントの抵抗率と電流容量、ビア方式、基材、硬化・焼結条件を、
  fab向けDRCと同じ枠組みの別プロファイルで検証します。
- **材料を考慮した電気解析**: 銅箔前提ではなく実測の材料データから配線抵抗、
  電圧降下、温度上昇を見積もります。
- **ハイブリッド製造の振り分け**: 同じ設計グラフから、手元で作れるローカル
  試作版と、密度や電流が要求を超える場合の従来fab向け量産版を生成します。
- **クローズドループ検査**: 位置合わせや導通・抵抗の測定結果を設計へ戻し、
  機体や材料ロットの癖も知識として蓄積します。
- **構造エレクトロニクスへの余地**: `Layout`を平面リジッド基板に固定せず、
  筐体表面や埋め込み配線を含む非平面回路への将来拡張を妨げない設計グラフを
  優先します。

その先には、身体の3Dスキャンを機械制約として取り込み、非平面・伸縮回路を
その場で作る個人適合ウェアラブルがあります。個人ごとに形状が変わる設計では、
回路図やアートワーク図を正とする方式は破綻します。VibeBBが要件・制約・
設計根拠を含む設計グラフを正とし、製造データを毎回再生成するのは、この
方向まで同じ仕組みで届かせるためです。

これはvision-levelの将来方向であり、現在の実装範囲の約束ではありません。
