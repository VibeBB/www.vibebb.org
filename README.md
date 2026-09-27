# VibeBB — Vibe BreadBoarding

<img src="assets/vibebb-silkscreen.svg" alt="VibeBB — Vibe BreadBoarding (silkscreen logo)" width="320">

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/VibeBB/www.vibebb.org)

[English](#english) | [日本語](#日本語)

## English

VibeBB is *Vibe BreadBoarding*: describe what you want to build in
natural language, and AI designs the circuit board, enclosure, and
firmware. Pass or fail is decided by deterministic gates and
real-hardware evidence — not by vibes. This repository is the source of
the site at [vibebb.org](https://vibebb.org).

- [Overview](#overview)
- [Why "Vibe BreadBoarding"?](#why-vibe-breadboarding)
- [How it works](#how-it-works)
- [Recording design rationale](#recording-design-rationale)
- [Learning across conversations](#learning-across-conversations)
- [How it differs from LLM-only CAD](#how-it-differs-from-llm-only-cad)
- [Future vision: "printing" boards at home](#future-vision-printing-boards-at-home)

### Overview

VibeBB aims to make AI the primary designer. The AI handles requirements
elicitation, part selection, circuit design, board layout, enclosure
design, firmware design, manufacturing data generation, and iteration
driven by manufacturing and real-hardware feedback. The human takes part
as the owner of the requirements, and can also act as a reviewer or make
direct fixes when needed.

**Scope.** Mass-production quality and certification are not assumed.
The goal is to reach a working prototype as fast as possible.

### Why "Vibe BreadBoarding"?

The name is modeled after *Vibe Coding*. VibeBB brings the idea Andrej
Karpathy expressed in his
[February 2025 post](https://x.com/karpathy/status/1886192184808149383) —
"fully give in to the vibes, embrace exponentials, and forget that the
code even exists" — together with the interactive "see stuff, say stuff,
run stuff" loop, into the prototyping and verification of boards,
enclosures, and firmware. As
[Collins Dictionary's 2025 Word of the Year](https://blog.collinsdictionary.com/language-lovers/collins-word-of-the-year-2025-ai-meets-authenticity-as-society-shifts/)
shows, the development experience of stating intent in natural language,
looking at the result, and giving the next instruction is spreading.

Just as the breadboard made it possible to "try as you think" without
soldering, VibeBB lets you simply say what you want in words. The AI
drives part selection, circuit design, board layout, enclosure,
firmware, and manufacturing data, while the human looks at the working
prototype board and its fitting enclosure and gives feedback. Not
drawing a schematic is the counterpart of not reading code in Vibe
Coding.

VibeBB does not mean that design and verification are light; it means
that the heavy verification is hidden from human hands. Following
[Simon Willison](https://simonwillison.net/2025/Mar/19/vibe-coding/)'s
distinction between true Vibe Coding, where the artifact is never
reviewed, and AI-assisted work that includes review, VibeBB moves the
reviewer's role from humans to deterministic gates and real-hardware
evidence.

### How it works

#### The experience loop

**Describe → AI designs and verifies with deterministic gates → build
and try → feed measurements into the next design.** The basic cycle is
to build first, check on real hardware, and turn the next change around
immediately, rather than spending a long time on desk study.

#### Design flow

From the requirements dialogue through manufacturing and real-hardware
feedback, the agents proceed in parallel, exchanging opinions and
requests with one another. Firmware is delegated to OpenHands' native
software development capability, so board, enclosure, and firmware are
designed and verified within the same interactive flow.

#### Design principles

- Each stage produces machine-readable and visual projections, which
  SDK subagents and vision review on a best-effort basis. Findings are
  passed to the fix loop as natural-language text; reviews hold no
  pass/fail authority.
- Human review is optional. By default the AI runs end to end, from
  requirements to manufacturing data. What the user verifies is not
  schematics or artwork, but whether the delivered board and enclosure
  actually work and fit.

### Recording design rationale

VibeBB does not rely on conversation logs or memory to preserve *why*
the design was made this way. It stores the record in the same change as
the design input (git). Values chosen by the designer (the AI) — parts,
placement, trace widths, silkscreen, stackup, design rules, net classes,
safety boundaries, mechanical dimensions, firmware pin assignments — are
recorded together with the reason for adoption, the rejected
alternatives, the driving requirements, and their provenance.

Missing records are detected deterministically. Attributes that express
design decisions are classified as either required or exempt; attributes
in neither class become `unclassified`. When a record no longer matches
its subject, the reason must be recorded again.

As a result, even several revisions later or for a different reader,
"why this part", "why this trace width", and "why this dimension" can be
traced after the fact. Design rationale explains reasons; it holds no
pass/fail authority. Pass or fail is decided by the deterministic gates
and evidence.

### Learning across conversations

VibeBB writes what it learns while working into OpenHands' persistent
memory and reads it back at the start of the next conversation. What
accumulates with use includes: the parts and part numbers frequently
used in the repository, how footprints and stackups are chosen, the
clearance and trace-width values that actually passed, know-how about
enclosure fastening and wall thickness, points repeatedly flagged by
silkscreen or review, and the user's design preferences. Proposals can
therefore start from the conventions of previous designs, without the
same explanation being repeated every time.

This persistent memory is disabled by default. To use it, enable
*Persistent memory* in the OpenHands Local GUI settings. Memos assist the
working context; pass or fail is decided by the deterministic gates and
evidence.

### How it differs from LLM-only CAD

The difference from LLM-only CAD is not that VibeBB produces the same
solution every time, but that the produced design can be re-verified
afterwards. Even if each run yields a different solution, that is
acceptable as long as it can be verified by deterministic measurement
and by independent parsers re-reading the output.

### Future vision: "printing" boards at home

VibeBB's premise — cheap and fast manufacturing — can go further. If
printed electronics matures, boards could be made on the spot like a
home 3D printer, shortening "build and try" from days to tens of
minutes, and VibeBB would truly approach breadboard speed. The same
applies to enclosures, brackets, and mechanical parts: 3D printing and
CNC quoting, DFM, and ordering services become procurement channels on
par with board fabs.

Soberly speaking, in-home manufacturing of simple one- and two-layer
boards is already real (desktop milling, conductive ink printing). On
the other hand, high-density multilayer boards, plated through holes,
high current, and certified mass production will remain the domain of
professional fabs for the time being. Technologies such as conductive
filament, conductive paste, and 3D-MID/LDS/IME are blurring the boundary
between enclosure and circuit, but they presuppose verification of
conductivity, solderability, contact resistance, and durability.

VibeBB builds this future in from the start:

- **DRC with mechanical and material profiles**: minimum trace width and
  spacing, tool diameter, ink or filament resistivity and current
  capacity, via method, substrate, and curing/sintering conditions are
  verified in a separate profile within the same framework as
  fab-facing DRC.
- **Material-aware electrical analysis**: trace resistance, voltage
  drop, and temperature rise are estimated from measured material data
  rather than by assuming copper foil.
- **Hybrid manufacturing dispatch**: from the same design graph, a
  locally prototyped version that can be built at hand is generated
  alongside a conventional fab mass-production version for when density
  or current exceeds what local methods can deliver.
- **Closed-loop inspection**: alignment, continuity, and resistance
  measurements are fed back into the design, and the quirks of each
  machine and material lot are accumulated as knowledge.
- **Room for structural electronics**: `Layout` is not fixed to planar
  rigid boards. A design graph that does not preclude future extension
  to non-planar circuits — including enclosure surfaces and embedded
  wiring — is preferred.

Beyond that lie personally fitted wearables that take a 3D body scan as
a mechanical constraint and produce non-planar, stretchable circuits on
the spot. For designs whose shape changes from person to person, any
approach that treats schematics or artwork drawings as the source of
truth breaks down. VibeBB regenerates manufacturing data every time from
a design graph containing requirements, constraints, and design
rationale as the source of truth, precisely so that the same mechanism
can reach this far.

This is a vision-level future direction, not a promise about the current
implementation scope.

## 日本語

VibeBBは「Vibe BreadBoarding」です。作りたいものを自然言語で伝えると、
AIが回路基板・筐体・ファームウェアを設計し、合否は決定論的ゲートと
実機エビデンスが判定します――バイブスではなく。このリポジトリは
[vibebb.org](https://vibebb.org)のサイトのソースです。

- [概要](#概要)
- [なぜ「Vibe BreadBoarding」なのか](#なぜvibe-breadboardingなのか)
- [仕組み](#仕組み)
- [設計根拠を残す](#設計根拠を残す)
- [会話をまたいで学習する](#会話をまたいで学習する)
- [LLM-only CADとの違い](#llm-only-cadとの違い)
- [将来展望：家庭で基板を「印刷」する時代へ](#将来展望家庭で基板を印刷する時代へ)

### 概要

VibeBBは、AIが主たる設計者となることを目指します。AIは要件のヒアリング、
部品選定、回路設計、基板レイアウト、筐体設計、ファームウェア設計、
製造データの生成、そして製造・実機フィードバックを受けた反復までを担います。
人間は要件のオーナーとして関わり、必要に応じてレビュアーとなることも、
直接修正することもできます。

**対象範囲。** 量産品質や認証は前提にしません。動く試作へ最短で到達することを
目的とします。

### なぜ「Vibe BreadBoarding」なのか

この名前はVibe Codingになぞらえたものです。Andrej Karpathyが
[2025年2月の投稿](https://x.com/karpathy/status/1886192184808149383)で示した
「完全にバイブスに身を委ね、指数関数的な進歩を受け入れ、コードの存在すら忘れる
（fully give in to the vibes, embrace exponentials, and forget that the code
even exists）」という発想と、「see stuff, say stuff, run stuff」の対話的な
ループを、VibeBBは基板・筐体・ファームウェアの試作と検証へ持ち込みます。
[Collins英語辞典の2025年Word of the Year](https://blog.collinsdictionary.com/language-lovers/collins-word-of-the-year-2025-ai-meets-authenticity-as-society-shifts/)
が示すように、自然言語で目的を伝え、結果を見て、次の指示を返すという開発体験は
広がりつつあります。

ブレッドボード（BreadBoard）がハンダ付けなしに「考えながら試す」ことを可能に
したように、VibeBBではやりたいことを言葉で伝えるだけで済みます。AIが部品選定、
回路設計、基板レイアウト、筐体、ファームウェア、製造データを進め、人間は
動く試作基板とそれが収まる筐体を見てフィードバックを返します。回路図を描かない
ことは、Vibe Codingにおいてコードを読まないことと対になる発想です。

VibeBBは、設計や検証が軽いという意味ではありません。重い検証を人間の手作業から
隠す、という意味です。
[Simon Willison](https://simonwillison.net/2025/Mar/19/vibe-coding/)が区別した、
生成物をレビューしない本来のVibe Codingと、レビューを伴うAI活用とを踏まえ、
VibeBBはレビュアーの役割を人間から決定論的ゲートと実機エビデンスへ移します。

### 仕組み

#### 体験のループ

**語る → AIが設計し、決定論的ゲートで検証する → 作って試す → 測定結果を次の
設計へ返す。** 長時間の机上検討よりも、まず作って実機で確かめ、すぐに次の
変更を回すことを基本サイクルとします。

#### 設計フロー

要件対話から製造・実機フィードバックまでを、各エージェントが互いに意見や
要望を出し合いながら並行して進めます。ファームウェアはOpenHands本来の
ソフトウェア開発能力へ委譲するため、基板・筐体・ファームウェアを同じ対話的な
流れの中で設計・検証できます。

#### 設計原則

- 各工程で機械可読な投影と視覚的な投影を生成し、SDKのsubagentとvisionが
  best-effortでレビューします。所見は自然文で修正ループへ渡され、レビューは
  合否の権限を持ちません。
- 人間によるレビューは任意です。既定ではAIが要件から製造データまで走り切ります。
  ユーザーが確かめるのは回路図やアートワークではなく、届いた基板と筐体が
  実際に動き、収まるかどうかです。

### 設計根拠を残す

VibeBBは「なぜその設計にしたか」を会話ログや記憶に頼らず、設計入力（git）と
同じ変更の中に保存します。部品、配置、配線幅、シルク、stackup、design rule、
net class、安全境界、機構寸法、ファームウェアのピン割当のように設計者（AI）が
選んだ値は、採用理由、却下した代替案、その値を導いた要求、出所とともに
記録されます。

記録漏れは決定論的に検出します。設計判断を表す属性は「必須」と「免除」に
分類され、どちらにも分類されない属性は`unclassified`となります。記録が対象と
一致しなくなった場合は、理由の再記述を求めます。

これにより、数リビジョン後であっても、別の読み手であっても、「なぜこの部品か」
「なぜこの配線幅か」「なぜこの寸法か」を後から辿れます。設計根拠は理由を
説明するものであり、合否の権限は持ちません。合否は決定論的ゲートとエビデンスが
判定します。

### 会話をまたいで学習する

VibeBBは作業の中で分かったことをOpenHandsの永続メモリへ書き残し、次の会話の
開始時に読み込みます。リポジトリでよく使う部品や型番の傾向、footprintや
stackupの選び方、clearanceや配線幅で実際に通った値、筐体の締結や肉厚の勘所、
シルクやレビューで毎回指摘される観点、ユーザーが好む設計の癖などが、使うほど
溜まっていきます。その結果、同じ説明を毎回繰り返さなくても、以前の設計の流儀を
引き継いだ提案から始められます。

この永続メモリは既定で無効です。利用するには、OpenHands Local GUIの設定で
永続メモリ（Persistent memory）を有効にしてください。メモは作業文脈の補助で
あり、合否は決定論的ゲートとエビデンスが判定します。

### LLM-only CADとの違い

LLM-only CADとの違いは、毎回同じ解を出すことではなく、出た設計を後から
再検証できることにあります。実行ごとに解が異なっても、決定論的な実測と、
独立したparserによる再読込で検証できればよい、と考えます。

### 将来展望：家庭で基板を「印刷」する時代へ

VibeBBの前提である「製造の安さと速さ」は、さらに先へ進む可能性があります。
プリンテッドエレクトロニクスが成熟すれば、家庭用3Dプリンタのように基板を
その場で作れるようになり、「作って試す」は数日から数十分へ短縮され、VibeBBは
本当にブレッドボードの速度に近づきます。同じことは筐体・ブラケット・機械部品
にも当てはまり、3Dプリント／CNCの見積・DFM・発注サービスは、基板fabと並ぶ
調達経路になります。

冷静に見れば、単純な1〜2層基板の宅内製造はすでに現実です（卓上切削、
導電性インク印刷）。一方で、高密度多層、メッキスルーホール、大電流、認証を
要する量産は、当面プロのfabの領域です。導電性フィラメント、導電性ペースト、
3D-MID/LDS/IMEのような技術は筐体と回路の境界を曖昧にしつつありますが、
導電率、はんだ付け性、接触抵抗、耐久性の検証が前提になります。

VibeBBはこの未来を最初から織り込みます。

- **機械・材料プロファイル対応のDRC**: 最小線幅・間隔、工具径、インクや
  フィラメントの抵抗率と電流容量、ビア方式、基材、硬化・焼結条件を、
  fab向けDRCと同じ枠組みの中の別プロファイルとして検証します。
- **材料を考慮した電気解析**: 銅箔を前提とせず、実測の材料データから配線抵抗、
  電圧降下、温度上昇を見積もります。
- **ハイブリッド製造の振り分け**: 同じ設計グラフから、手元で作れるローカル
  試作版と、密度や電流が要求を超える場合の従来fab向け量産版とを生成します。
- **クローズドループ検査**: 位置合わせ、導通、抵抗の測定結果を設計へ戻し、
  機体や材料ロットの癖も知識として蓄積します。
- **構造エレクトロニクスへの余地**: `Layout`を平面のリジッド基板に固定せず、
  筐体表面や埋め込み配線を含む非平面回路への将来拡張を妨げない設計グラフを
  優先します。

その先には、身体の3Dスキャンを機械的制約として取り込み、非平面・伸縮回路を
その場で作る、個人適合ウェアラブルがあります。個人ごとに形状が変わる設計では、
回路図やアートワーク図を正とする方式は破綻します。VibeBBが、要件・制約・
設計根拠を含む設計グラフを正とし、製造データを毎回再生成するのは、同じ仕組みで
この方向まで届かせるためです。

これはvision-levelの将来方向であり、現在の実装範囲についての約束ではありません。
