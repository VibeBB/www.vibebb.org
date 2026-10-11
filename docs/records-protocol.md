# VibeBB Record Protocol (VRP) v2

## English

Every VibeBB agent records why it made each choice, what it saw, and what
it thinks of its own and its sisters' work. VRP v2 is the shared format and
rule set for those records. The canonical specification for implementers is
[`docs/records-protocol.md` in wire-agent](https://github.com/VibeBB/wire-agent/blob/main/docs/records-protocol.md).
All eleven agents (bard, circuit, dashboard, document, firmware, fpga,
mechanical, production-engineering, simulation, UX-creator, wire) ship a
byte copy of the same record core, locked by each repository's
`scripts/check_shared_hooks.py`.

Records are advisory evidence. Only deterministic gates decide pass or
fail. A record, a review or a song can stop work. It cannot turn a failed
or unknown gate into a pass.

### Logs

Each agent appends JSONL logs under `observations/<agent>/`:

| Log | Kind | Written when |
| --- | --- | --- |
| `decisions.jsonl` | `decision` | every non-trivial design choice: question, principles, 2+ options with pros and cons, rationale, evidence, risks, `revisit_when` |
| `impressions.jsonl` | `stage_impression` | at the end of every stage, after the final regeneration |
| `vision-reviews.jsonl` | `vision_review` | every time the model looks at an image (primary, blind or tiebreak) |
| `reads.jsonl` | `upstream_read` | a sister artifact was taken in; lists the sister impressions bound to those bytes |
| `insights.jsonl` | `insight_status` | a hypothesis from an impression is tried, adopted, rejected, deferred or superseded |
| `reconciles.jsonl` | `review_reconcile` | independent reviews of one image were compared |
| `song-receipts.jsonl` | `song_receipt` | a bard song addressed to this agent was read |

### Hash chain

Every record carries `schema_version: 2`, `sequence`, `prev_event_id`,
`recorded_at` and `event_id`, the SHA-256 of the record's canonical JSON.
Each line links to the previous one. A malformed line, an edited record, a
deleted or reordered line, a duplicate id or a timestamp that goes back in
time breaks the chain.

A broken log fails closed:

- writers refuse to append to it;
- readers report it under `integrity_problems` and never mix it into
  results;
- a reference into it does not resolve. This includes a liaison response's
  `decision_refs` or `impression_refs`, even ids that were valid before the
  break.

The chain proves that a log was not changed after writing. It does not prove
who wrote a line or that its content is true.

### Impressions

An impression is a long-form reading of an artifact, tied to the exact bytes
it read:

- at least 400 characters and three sentences;
- facets: `observed`, `works`, `concerns` (each with severity and the agent
  that owns it), `feelings` (maker and user), `next_actions`;
- claims whose anchors occur verbatim in the bound artifact, plus the
  artifact's SHA-256;
- `confidence`, `unknowns`, optional insights (hypotheses with a test) and
  `upstream` answers to sister impressions.

Writers refuse an impression that is 80% similar to an earlier one, and ask
for a `delta` at 60%. Similarity is measured with character 5-grams, so it
works for Japanese and English alike.

### Answers and no re-raise

- An agent that reads a sister's impression must answer it in a later
  impression's `upstream` (`adopted`, `deferred`, `disputed` or `noted`).
- A concern that its owner disputed cannot be raised again while the bound
  artifact bytes are unchanged.
- A rejected hypothesis cannot be proposed again unless it names the old one
  in `revisits` with a reason.
- An insight becomes `adopted` only with a deterministic gate `pass` and the
  adopting decision.

### Blind review

Important images (drawings, schematics, screenshots, plots) get a second
review by a sub-agent that cannot see the first one. Its `blind-guard` hook
denies reads of records, reports, liaison answers and songs. Two reviews
agree when their finding categories overlap by at least half and no `error`
is raised by only one side. A disagreement allows exactly one tiebreak
review. If the split remains after round 2, the outcome is `unresolved`,
which counts as unknown and never as pass.

### Bard songs

bard writes an immutable `liaison/<id>.bard-song.json`, pinned by SHA-256 and
made from sister impressions. The addressed agent records one
`song_receipt`. bard never writes to itself, never receipts its own song,
and never composes in reply to a receipt, so songs cannot loop. Songs never
feed a gate.

### Family hub in UX-creator

UX-creator reads every sister's log. `ux record hub` (MCP
`ux_records_hub`) aggregates impressions per agent and lists cycles in the
"raiser → owner" graph of open concerns and disputes, so a concern bouncing
between agents becomes visible. The hub's `status` is `clear` or
`attention`. It is not a gate verdict.

### Bounds that prevent loops and deadlocks

- The Stop hook refuses to end a session that owes records at most twice,
  then allows the stop and records what was missing.
- Records cite only records that already exist, so no agent waits for
  another.
- One blind review, at most one tiebreak and at most two reconcile rounds
  per image.
- Read receipts and song receipts never require the producer to write a new
  impression.

## 日本語

VibeBBのagentは、どの案をなぜ選んだか、何を見たか、自分と姉妹の成果物を
どう感じたかを記録に残します。VRP v2は、その記録の共通の形式と決まりです。
実装する人向けの正式な仕様は、
[wire-agentの`docs/records-protocol.md`](https://github.com/VibeBB/wire-agent/blob/main/docs/records-protocol.md)
にあります。11のagent（bard、circuit、dashboard、document、firmware、fpga、
mechanical、production-engineering、simulation、UX-creator、wire）は、
同じ記録の中核部分をそのまま写して持っています。写しが同じであることは、各repositoryの
`scripts/check_shared_hooks.py` が確かめます。

記録は参考の証拠です。合格か不合格かを決めるのは、決定論のゲートだけです。
記録、レビュー、歌は作業を止めることはできますが、不合格や「不明」の
ゲートを合格に変えることはできません。

### ログ

各agentは、`observations/<agent>/` にJSONLのログを追記します。

| ログ | 種類 | 書くとき |
| --- | --- | --- |
| `decisions.jsonl` | `decision` | 自明でない設計の判断のたび。問い、原則、長所と短所付きの2つ以上の案、理由、証拠、リスク、見直す条件 |
| `impressions.jsonl` | `stage_impression` | 各段階の終わり（最後の生成のあと） |
| `vision-reviews.jsonl` | `vision_review` | モデルが画像を見るたび（最初、ブラインド、決着） |
| `reads.jsonl` | `upstream_read` | 姉妹の成果物を取り込んだとき。そのファイルに結び付いた姉妹の感想を並べます |
| `insights.jsonl` | `insight_status` | 感想の中の仮説を試した、採用した、却下した、保留した、置き換えたとき |
| `reconciles.jsonl` | `review_reconcile` | 1枚の画像の独立したレビューを突き合わせたとき |
| `song-receipts.jsonl` | `song_receipt` | 自分宛てのbardの歌を読んだとき |

### hashの鎖

どの記録も、`schema_version: 2`、`sequence`、`prev_event_id`、
`recorded_at`、`event_id`（記録の正規化したJSONのSHA-256）を持ちます。
各行は1つ前の行とつながっています。次のような場合に、鎖が切れます。

- 壊れた行がある
- 記録が書き換えられた
- 行が消された、または順番が入れ替えられた
- 同じidが2回ある
- 時刻が過去に戻っている

鎖の切れたログは、安全側に倒して扱います。

- 書く側は、そのログに追記しません。
- 読む側は、`integrity_problems` として報告し、結果には混ぜません。
- そのログへの参照は解決しません。liaisonの返答の `decision_refs` や
  `impression_refs` も同じで、鎖が切れる前に正しく書かれたidでも受け付けません。

鎖で分かるのは、書いた後にログが変えられていないことです。誰が書いたか、
中身が正しいかは分かりません。

### 感想

感想は、成果物を読んだ長い文章で、読んだファイルの中身そのものに結び付きます。

- 400文字以上、3文以上
- 観点: `observed`（観察したこと）、`works`（うまくいっていること）、
  `concerns`（懸念。重さと、対応するagentを書きます）、`feelings`（作り手と
  使い手の気持ち）、`next_actions`（次にすること）
- 根拠（claim）: 結び付けたファイルにそのまま出てくる文字列と、そのファイルのSHA-256
- 確信度、分からないこと。任意で、仮説（確かめ方付き）と、姉妹の感想への
  答え（`upstream`）

書く側は、前の感想と80%以上似ている感想を受け付けません。60%以上似ている
場合は、何が変わったか（`delta`）を求めます。似ている度合いは5文字ずつの
並びで測るので、日本語でも英語でも使えます。

### 答えることと、再提起の禁止

- 姉妹の感想を読んだagentは、後の感想の `upstream` で答えます（`adopted`
  採用、`deferred` 保留、`disputed` 反論、`noted` 了解）。
- 対応する側が反論した懸念は、結び付いたファイルの中身が変わらない限り、もう一度
  出せません。
- 却下された仮説は、`revisits` で元の仮説と理由を示さない限り、もう一度
  出せません。
- 仮説を `adopted`（採用）にするには、決定論のゲートの合格と、採用の判断の
  記録が必要です。

### ブラインドレビュー

重要な画像（図面、回路図、画面、グラフ）は、最初のレビューを見られない
sub-agentがもう一度レビューします。そのsub-agentの `blind-guard` hookは、
記録、レポート、liaisonの返答、歌を読むことを拒否します。2つのレビューは、
指摘の分類が半分以上重なり、片方だけが `error` を出していなければ一致と
みなします。一致しないときは、決着のレビューを1回だけ行えます。2回目でも
割れたら `unresolved`（未解決）で、「不明」として扱い、合格にはしません。

### bardの歌

bardは、姉妹の感想から作った歌を、書き換えられない
`liaison/<id>.bard-song.json` に書き出し、SHA-256で固定します。宛先のagentは、
1つの歌に1回だけ `song_receipt`（受領）を記録します。次のことは起きないので、
歌のやり取りが止まらなくなることはありません。

- bardが自分宛てに歌を書く
- bardが自分の歌の受領を書く
- bardが受領に応えて歌を作る

歌がゲートの入力になることはありません。

### UX-creatorのfamily hub

UX-creatorは、すべての姉妹のログを読みます。`ux record hub`（MCPでは
`ux_records_hub`）は、agentごとに感想を集約します。また、未解決の懸念と反論の
「提起した側 → 対応する側」をつないだグラフの中のループを並べます。これで、agentの間を
行き来し続ける懸念が見えるようになります。hubの `status` は `clear` か
`attention` で、ゲートの判定ではありません。

### 堂々巡りと待ち合わせを防ぐ上限

- Stopのhookは、記録が足りないセッションの終了を2回まで拒否します。その後は
  終了を許し、足りなかったものを記録します。
- 記録が参照するのは、すでにある記録だけです。ほかのagentを待つagentは
  ありません。
- 1枚の画像につき、ブラインドレビューは1回、決着のレビューは1回まで、
  突き合わせは2回までです。
- 既読の受領や歌の受領が、作り手に新しい感想を求めることはありません。
