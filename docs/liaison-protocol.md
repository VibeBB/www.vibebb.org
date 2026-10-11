# Sister Liaison Protocol (SLP) v2

[English](#english) | [日本語](#日本語)

## English

SLP v2 is how UX-creator asks a sister agent for work and how the sister
answers. Requests and responses are JSON files in the shared workspace
`liaison/` directory. The authoritative models are
`src/ux_creator/requests.py` and `src/ux_creator/responses.py` in
UX-creator-agent; every sister ships an inbox/respond implementation
(`<agent>_ux_inbox`, `<agent>_ux_respond`, or the `ux` CLI subcommand).

### Request: `liaison/<id>.ux-request.json`

| Field | Rule |
| --- | --- |
| `schema_version` | `2` |
| `system` | `"ux-creator"` |
| `id` | `^[a-z0-9][a-z0-9._-]{0,63}$`, unique in `liaison/` |
| `target_agent` | one of `bard`, `circuit`, `dashboard`, `doc`, `firmware`, `fpga`, `mech`, `prodeng`, `sim`, `wire` |
| `stage` | `requirements`, `design`, `manufacturing_handoff`, `build`, `evaluation`, `revision` |
| `risk` | `low` or `high`; a `high` request must cite at least one contract job id in `rationale` |
| `purpose` | at least 20 characters |
| `requested_changes`, `expected_deliverables`, `acceptance` | non-empty lists |
| `inputs[]` | `{path, sha256}` pins of the files the sister must read |
| `depends_on[]` | ids of requests that must be answered first |
| `created_at` | timestamp |

### Response: `liaison/<id>.ux-response.json`

| Field | Rule |
| --- | --- |
| `request` | the request id |
| `responder` | the answering agent; must equal the request `target_agent` |
| `status` | `accepted`, `in_progress`, `done`, `rejected`, `deferred`, `needs_info` |
| `reason` | why, required for anything but routine progress |
| `input_hashes` | the `sha256` of each request input as the sister read it |
| `artifacts[]` | `{path, sha256}` of the delivered files |
| `gate_verdicts[]` | `{gate, verdict}` with `verdict` in `pass`, `fail`, `unknown` |
| `decision_refs[]`, `impression_refs[]` | VRP v2 event ids that justify the answer |
| `questions_for_user[]` | questions only the user can answer |
| `responded_at` | timestamp |

A `done` response must carry artifacts, gate verdicts and VRP references.
The sister's guard hook protects its response file; it never edits the
request or another agent's inputs.

### Liaison states (UX-creator `liaison status`)

| State | Meaning |
| --- | --- |
| `open` | no response yet, dependencies answered |
| `answered` | a valid response whose pins and references all check out |
| `mismatched` | the responder is not the request's target agent |
| `stale` | a request input changed, or the response's `input_hashes` do not match it |
| `broken` | a delivered artifact is missing or changed, or a VRP reference does not resolve in an intact chain |
| `blocked` | a `depends_on` request is not answered yet |
| `circular` | `depends_on` forms a cycle; the requests are never worked |

Every state other than `answered` fails closed for production planning.
Responses never change a sister's deterministic gate verdict; they report
it.

### Rules

- **Nothing waits.** A sister answers with what exists; missing input is
  answered `needs_info` or `deferred` with a reason.
- **No re-routing.** Changes to files other agents pin go through ECR/ECO
  (see [handoffs.md](handoffs.md#change-control-ecr--eco)).
- Records cited in a response follow [VRP v2](records-protocol.md).

## 日本語

SLP v2は、UX-creatorが姉妹agentに作業を頼み、姉妹が答えるための決まりです。
依頼と応答は、共有workspaceの `liaison/` にあるJSONファイルです。正本はUX-creator-agentの
`src/ux_creator/requests.py` と `src/ux_creator/responses.py` です。各姉妹は、受け取りと
応答の機能（`<agent>_ux_inbox`、`<agent>_ux_respond`、または `ux` のCLI）を持っています。

- **依頼**（`<id>.ux-request.json`）には、次のことを書きます。
  - 宛先
  - 工程
  - 危険度（`high` の場合は、理由の欄にcontractのjob idを1つ以上書きます）
  - 目的（20字以上）
  - 求める変更、成果物、受け入れ条件
  - 読むべき入力ファイルとそのsha256
  - 先に答えが必要な依頼（`depends_on`）
- **応答**（`<id>.ux-response.json`）には、次のことを書きます。
  - 状態（`accepted`、`in_progress`、`done`、`rejected`、`deferred`、`needs_info`）
  - 理由
  - 読んだ入力のsha256
  - 成果物とそのsha256
  - ゲートの判定
  - 根拠になったVRPの記録のid
  - ユーザーへの質問

  `done` で答える時は、成果物、ゲートの判定、記録のidが必須です。
- **状態**: UX-creatorは、依頼と応答を突き合わせて、次のどれかに分類します。`answered` 以外は、生産計画で不合格として扱います。
  - `open`: まだ答えが無い
  - `answered`: 正しく答えた
  - `mismatched`: 宛先でないagentが答えた
  - `stale`: 入力が変わった
  - `broken`: 成果物が変わった、または記録のidが壊れていない鎖の中に見つからない
  - `blocked`: 先に必要な依頼が答えられていない
  - `circular`: 依頼同士が互いを待っている
- 応答は、姉妹のゲートの判定を報告するだけで、変えることはありません。待たずに、今あるもので答えます。他のagentが固定しているファイルを変える時は、ECR/ECOを通します。
