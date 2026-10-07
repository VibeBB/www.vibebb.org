# Running VibeBB on Agent Canvas / OpenHands

[English](#english) | [日本語](#日本語)

## English

This page collects practices for running the VibeBB plugins on your own
Agent Canvas (OpenHands) setup. It was written against Agent Canvas
1.25.x and agent-server 1.53.x — the September–October 2026 releases.
Older versions work, but several features mentioned below need 1.18 or
newer.

- [Prerequisites](#prerequisites)
- [Install and enable the plugins](#install-and-enable-the-plugins)
- [Model lanes: author and review](#model-lanes-author-and-review)
- [Agent profiles](#agent-profiles)
- [Secrets](#secrets)
- [Conversation runtime: keep it local](#conversation-runtime-keep-it-local)
- [Working practice](#working-practice)
- [Automations](#automations)
- [Apps](#apps)
- [Reading results](#reading-results)
- [Safety notes](#safety-notes)
- [Troubleshooting](#troubleshooting)

### Prerequisites

- Docker on the machine that runs the agent. Every VibeBB tool executes
  inside a pinned container image; nothing else needs to be installed on
  the host.
- Agent Canvas 1.24 or newer is recommended. Agent profiles, the Docker
  conversation runtime, and the automation template library arrived
  across 1.17–1.24; 1.25 adds the model router, bulk LLM profiles, and
  persona support on agent profiles.

### Install and enable the plugins

- Add each plugin by repository URL with the plugin folder, for example
  `https://github.com/VibeBB/wire-agent`, folder `plugins/wire`. Since
  Agent Canvas 1.21 you can paste the folder's GitHub URL directly
  (`.../tree/main/plugins/wire`).
- Keep every sister plugin you use **enabled**. Sisters exchange design
  files through the shared workspace; a disabled plugin leaves its
  liaison input `unknown`, and unknown inputs fail closed at the
  receiving gate rather than being skipped.

### Model lanes: author and review

- Each plugin's SessionStart hook seeds two LLM profiles,
  `vibebb-author` and `vibebb-review`, cloned from your active profile
  on first use. Generation and review can then run on different models.
- Point the lanes at different models deliberately: a strong model for
  `vibebb-author`, and a strong **vision-capable** model for
  `vibebb-review` — the review lane reads drawings and photos.
- Keep the lanes meaningfully separate. A review that comes from the
  same model configuration as the author is a weaker second opinion.
- Deterministic gates — not the models — issue pass/fail verdicts.
  Choosing a cheaper model degrades design quality, never safety;
  choosing a stronger model never weakens a gate.

### Agent profiles

Agent profiles (v1.18+) scope which tools, MCP servers, and secrets an
agent may use, and automations can pick a profile since 1.20. For
VibeBB work, a dedicated profile keeps design conversations least-
privilege:

- `mcp_server_refs`: only the sister plugins you actually use. MCP
  server names are `ux`, `bard`, `dashboard`, `doc`, `circuit`,
  `firmware`, `fpga`, `mech`, `prodeng`, `sim`, and `wire`.
- `secret_refs`: only the LLM credentials the conversation needs — see
  the next section.
- `disabled_skills`: skills unrelated to the design flow.
- `persona` / `system_message_suffix` (v1.25): optional project
  conventions the whole team shares.

Create profiles in the canvas settings or, as an advanced path, as JSON
files in `~/.openhands/agent-profiles/` (schema version 2).

### Secrets

- VibeBB tools run with `--network none` inside their tool containers.
  They never see LLM keys or cloud credentials; a design conversation
  only needs the LLM credentials of its active profile(s).
- Scope secrets with `secret_refs` on the profile instead of exposing
  them globally. Leave GitHub tokens and other unrelated secrets out of
  a design profile.
- Never place credentials in workspace files — workspace content is part
  of the conversation context.

### Conversation runtime: keep it local

- Keep `OH_CONVERSATION_RUNTIME` at `local` (the default). VibeBB tool
  containers need the host Docker daemon; a conversation running inside
  the Docker runtime has no Docker available, so the plugins' launchers
  cannot work there.
- The Docker conversation runtime (v1.20–1.21) isolates each
  conversation in its own container. It is a good fit for *untrusted*
  workloads — e.g., automations that process inbound GitHub issues — not
  for design conversations today.

### Working practice

- Use **Plan Mode** (`/plan`) for requirements and design briefs; leave
  it (or use `/code`) once you want files generated. The brief you
  approve is what the sisters verify against.
- Tag conversations with the project name. Artifacts carry `sha256`
  references back to their inputs, and tags make the producing
  conversation easy to find later.
- Enable *Persistent memory* in the Local GUI settings. VibeBB reads
  back what it learned in previous conversations — preferred stackups,
  part numbers, clearances that actually passed.
- Attach photos, datasheets, and sketches in chat. The vision review
  reads them and its findings enter the design file as assumptions for
  you to confirm.
- When a sister asks a question, answer it in the conversation — an
  unanswered question stays `unresolved` and blocks the verdict.

### Automations

- Automations can attach plugins directly (the plugin run type), so
  design-side automations are possible — e.g., a scheduled run that
  re-verifies generated artifacts after upstream inputs change, or that
  regenerates derived views when a sister's output lands. Give the
  automation its own agent profile scoped to only the plugins it needs.
- Version automations with Git Sync (bidirectional, with optional
  encryption), or share them as exported `.automation.json` files.
- Since 1.19 a disabled automation carries a structured reason and
  timestamp — check the automation's page before re-enabling.

### Apps

- Apps (Beta) are trusted, **unisolated** JavaScript pages inside the
  canvas. Install only sources you control. Install by repository URL,
  including folder (tree) URLs; *Update* re-installs in place.
- A VibeBB status App is a natural fit for the dashboard sister and may
  come later. Today Apps are optional for running VibeBB.

### Reading results

- Gates write the pass/fail report and machine-readable verdicts under
  `out/<name>/`. Design rationale lives in `records/`; liaison state in
  `observations/`.
- Vision reviews, impressions, and songs are advisory evidence. They can
  stop a design but never pass one — the gates decide.
- Generated files cannot be edited by hand; change the design input and
  regenerate. Committing the design files to git is encouraged — the
  records explain *why* the design is what it is.

### Safety notes

- A passing report is not a safety certification. Mains voltage,
  vehicles, medical and other regulated uses need review by a qualified
  person.
- Keep generated and recorded files public-safe: artifacts you commit or
  publish can contain design details but must never contain secrets.

### Troubleshooting

- `plugin root unresolved` in a tool call: the plugin is not installed
  or enabled — or the conversation is running under the Docker runtime,
  where the plugin's host paths do not exist. Install/enable the plugin
  and keep the runtime `local`.
- Docker errors when a tool starts: the host lacks Docker, or the pinned
  image is still pulling. Pre-pull the image if pulls are slow.
- Profile changes do not retro-apply: after editing a profile, switch
  the conversation's profile or start a new conversation.
- An automation stopped running: since 1.19 the canvas records why it
  was disabled; read the disablement reason on the automation page.

## 日本語

このページは、自分の Agent Canvas（OpenHands）環境で VibeBB
プラグインを動かすための実践的な指針をまとめたものです。Agent Canvas
1.25.x と agent-server 1.53.x（2026年9〜10月のリリース）を前提に書か
れています。古いバージョンでも動きますが、以下の一部機能には 1.18
以降が必要です。

- [前提条件](#前提条件)
- [プラグインのインストールと有効化](#プラグインのインストールと有効化)
- [モデルのレーン分け：作成とレビュー](#モデルのレーン分け作成とレビュー)
- [エージェントプロファイル](#エージェントプロファイル)
- [シークレット](#シークレット)
- [会話ランタイム：localのままにする](#会話ランタイムlocalのままにする)
- [作業の進め方](#作業の進め方)
- [オートメーション](#オートメーション)
- [Apps](#apps)
- [結果の読み方](#結果の読み方)
- [安全に関する注意](#安全に関する注意)
- [トラブルシューティング](#トラブルシューティング)

### 前提条件

- エージェントを実行するマシンに Docker。VibeBB のツールはすべて固定
  （digest ピン）されたコンテナイメージ内で動くため、ホストには Docker
  以外のインストールは不要です。
- Agent Canvas 1.24 以降を推奨。エージェントプロファイル、Docker 会話
  ランタイム、オートメーションのテンプレート群は 1.17〜1.24 で導入
  され、1.25 でモデルルーター、LLM プロファイルの一括登録、プロファイル
  へのペルソナ設定が追加されました。

### プラグインのインストールと有効化

- 各プラグインはリポジトリ URL とフォルダを指定して追加します。例：
  `https://github.com/VibeBB/wire-agent`、フォルダ `plugins/wire`。
  1.21 以降はフォルダの GitHub URL（`.../tree/main/plugins/wire`）を
  そのまま貼ってもインストールできます。
- 使う姉妹プラグインはすべて **有効** のままにしてください。姉妹は共有
  ワークスペース内の設計ファイルで連携します。無効化されたプラグインが
  あると、その liaison 入力は `unknown` になり、受け側のゲートで合格に
  なりません（スキップではなく fail closed です）。

### モデルのレーン分け：作成とレビュー

- 各プラグインの SessionStart フックが、2 つの LLM プロファイル
  `vibebb-author` と `vibebb-review` を初回起動時に有効なプロファイル
  から複製して作成します。生成とレビューを別モデルで実行できます。
- レーンは意図的に分けてください：`vibebb-author` には強いモデルを、
  `vibebb-review` には強い **視覚対応** モデルを。レビューレーンは図面や
  写真を読みます。
- レーンは意味のある形で分離してください。作成と同じモデル設定からの
  レビューは、セカンドオピニオンとして弱くなります。
- 合否を出すのは決定論的ゲートであり、モデルではありません。安いモデルを
  選んでも下がるのは設計品質だけで安全性は下がりません。逆に強いモデルを
  選んでもゲートは弱まりません。

### エージェントプロファイル

エージェントプロファイル（v1.18+）は、エージェントが使えるツール・
MCP サーバー・シークレットを範囲限定します。1.20 以降はオートメーション
にもプロファイルを指定できます。VibeBB の作業には専用プロファイルを作ると
最小権限にできます。

- `mcp_server_refs`：実際に使う姉妹プラグインだけ。MCP サーバー名は
  `ux`、`bard`、`dashboard`、`doc`、`circuit`、`firmware`、`fpga`、
  `mech`、`prodeng`、`sim`、`wire` です。
- `secret_refs`：会話が必要とする LLM 認証情報のみ（次節参照）。
- `disabled_skills`：設計フローと無関係なスキル。
- `persona` / `system_message_suffix`（v1.25）：任意。チームで共有したい
  プロジェクトの約束事を書けます。

プロファイルはキャンバスの設定画面で作成するか、上級者向けには
`~/.openhands/agent-profiles/` に JSON（スキーマバージョン 2）として
置きます。

### シークレット

- VibeBB のツールはツール用コンテナ内で `--network none` で動きます。
  LLM キーやクラウド認証情報を見ることはありません。設計の会話に必要な
  のは、有効なプロファイルが参照する LLM 認証情報だけです。
- グローバルに公開するのではなく、プロファイルの `secret_refs` で範囲
  限定してください。GitHub トークンなど無関係なシークレットは設計用
  プロファイルから外します。
- ワークスペースのファイルに認証情報を置かないでください。ワークスペース
  の内容は会話コンテキストの一部になります。

### 会話ランタイム：localのままにする

- `OH_CONVERSATION_RUNTIME` は `local`（デフォルト）のままにして
  ください。VibeBB のツール用コンテナはホストの Docker デーモンを必要と
  します。Docker ランタイム内の会話では Docker が使えないため、
  プラグインのランチャーが動作しません。
- Docker 会話ランタイム（v1.20〜1.21）は各会話を専用コンテナに隔離します。
  今日の段階では、GitHub Issue の受信処理のような**信頼できない**作業には
  適していますが、設計の会話には向きません。

### 作業の進め方

- 要件や設計ブリーフには **Plan モード**（`/plan`）を使い、ファイル生成に
  入ったら外す（または `/code`）形にします。あなたが承認したブリーフが、
  姉妹たちの検証基準になります。
- 会話にはプロジェクト名のタグを付けてください。成果物は入力の `sha256`
  を持ち、タグがあると生成元の会話を後で見つけやすくなります。
- Local GUI の設定で *Persistent memory* を有効にしてください。VibeBB は
  以前の会話で学んだこと（よく使う積層構成、型番、実際に通ったクリアランス
  など）を次の会話で読み戻します。
- 写真・データシート・スケッチは会話に添付してください。視覚レビューが
  読み取り、結果は設計ファイルに「あなたが確認する仮定」として入ります。
- 姉妹からの質問には会話で回答してください。未回答の質問は `unresolved`
  のまま残り、判定を止めます。

### オートメーション

- オートメーションはプラグインを直接アタッチ（plugin 実行形式）できるので、
  設計側の自動化が可能です。例：上流の入力が変わったときに生成物を再検証
  する定時実行や、姉妹の出力が置かれたときに派生ビューを再生成する実行。
  オートメーションには必要なプラグインだけを範囲指定した専用プロファイルを
  割り当ててください。
- Git Sync（双方向、暗号化オプションあり）でオートメーションを版管理するか、
  `.automation.json` をエクスポートして共有します。
- 1.19 以降、無効化されたオートメーションには構造化された理由とタイム
  スタンプが記録されます。再有効化の前にオートメーション画面で理由を
  確認してください。

### Apps

- Apps（ベータ）はキャンバス内で動く信頼済み・**非隔離** の JavaScript
  ページです。管理できるソースのみインストールしてください。リポジトリ
  URL（フォルダ / tree URL を含む）でインストールでき、*Update* でその場
  更新されます。
- VibeBB の状態表示 App は dashboard 姉妹に自然に載るため、今後出てくる
  可能性があります。現時点で VibeBB の実行に Apps は不要です。

### 結果の読み方

- ゲートは `out/<name>/` 配下に合否レポートと機械可読の判定を書きます。
  設計根拠は `records/`、liaison の状態は `observations/` に残ります。
- 視覚レビュー・所感・歌は助言的エビデンスです。設計を止めることはでき
  ますが、合格にすることはありません。合否はゲートが決めます。
- 生成ファイルは手編集できません。設計入力を変えて再生成してください。
  設計ファイルを git にコミットすることを推奨します。records が「なぜ
  この設計か」を説明します。

### 安全に関する注意

- 合格レポートは安全認証ではありません。商用電源・車載・医療などの規制
  用途は、有資格者によるレビューが必要です。
- 生成物・記録ファイルは公開可能な状態に保ってください。コミットや公開
  する成果物には設計の詳細を含めて構いませんが、シークレットを含めては
  いけません。

### トラブルシューティング

- ツール呼び出しで `plugin root unresolved`：プラグインが未インストール
  または無効、もしくは会話が Docker ランタイムで動いていてプラグインの
  ホスト側パスが存在しません。プラグインをインストール・有効化し、
  ランタイムは `local` のままにしてください。
- ツール起動時の Docker エラー：ホストに Docker がないか、ピンされた
  イメージを取得中です。取得が遅い場合は事前に pull してください。
- プロファイル変更は遡及しません。編集後は会話のプロファイルを切り替えるか、
  新しい会話を始めてください。
- オートメーションが止まった：1.19 以降は無効化の理由が記録されます。
  オートメーション画面で理由を確認してください。
