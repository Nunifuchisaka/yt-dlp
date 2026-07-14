---
name: video-to-obsidian
description: YouTube動画をダウンロードし、Whisperで文字起こしして、内容を資料化しObsidianに保存する一連のワークフロー。「この動画をダウンロードして文字起こししてObsidianにまとめて」「動画を資料化して」「URLをObsidianにまとめて」のような依頼、またはその一部（ダウンロードのみ、文字起こしのみ等）で使う。
---

# 動画ダウンロード → 文字起こし → Obsidian資料化

YouTube動画のURLを受け取り、以下の3ステップを順に実行する。一部のステップだけ依頼された場合は該当ステップのみ実行する。

## ステップ1: yt-dlp でダウンロード

```powershell
& "D:\yt-dlp\yt-dlp.exe" "<URL>"
```

- **フルパスで実行する**。`yt-dlp` だけだとシェルセッションによっては PATH が効かず CommandNotFoundException になる
- 保存先はカレントディレクトリになるため、**D:\yt-dlp で実行する**（動画ファイルはそこに置く運用）
- `yt-dlp.conf` が自動適用される（cookies認証・NVENC再エンコード・`--mark-watched` 等）。**実行すると動画がYouTube上で視聴済みになる**副作用がある
- ダウンロードは数分かかることがあるのでバックグラウンド実行にする

### 「Requested format is not available」エラーが出た場合（SABR/DRM制限）

1. まず `& "D:\yt-dlp\yt-dlp.exe" -U` で本体を更新して再試行
2. それでもダメなら player-client を差し替えて再試行（2026-07時点の実績ある回避策）:

```powershell
& "D:\yt-dlp\yt-dlp.exe" --extractor-args "youtube:player-client=android_vr,web_safari;player-params=2" "<URL>"
```

- 切り分けには `-F --no-mark-watched` でフォーマット一覧が取れるかを先に確認するとよい（`--no-mark-watched` を付けないと確認だけで視聴済みになる）

## ステップ2: Whisper で文字起こし

```powershell
& "E:\Tools\Whisper\venv\Scripts\python.exe" "E:\Tools\Whisper\Whisper.py" "<動画ファイルのフルパス>"
```

- **Whisper.bat は使わない**。末尾に `pause` があり非対話シェルでハングする。上記のように venv の python を直接呼ぶ（処理内容は同一）
- faster-whisper の large-v3 / CUDA / 日本語設定。動画と同じ場所に同名の `.txt` が保存される
- 数分かかるのでバックグラウンド実行にする
- 注意: yt-dlp の出力ファイル名には `[動画ID]` が含まれる。PowerShell で `Get-Content` 等を使うときは `[ ]` がワイルドカード扱いされるため **`-LiteralPath` を使う**（エージェントのファイル読み取りツールなら通常問題ない）

## ステップ3: 資料化して Obsidian に保存

保存先 Vault: `C:\Users\nunif\works\i\Obsidian`

1. **文字起こし全文を読む**（`.txt` をファイル読み取りツールで読む）
2. **Vault の構成を確認し、トピックに合うフォルダを選ぶ**（例: `Codex/`, `VRChat/`, `YouTube/`, `References/` など）。迷ったらフォルダ一覧を見て判断する
3. **同テーマの既存ノートがないか確認する**。あれば:
   - 出典が別（別動画・別チャンネル）なら別ノートとして作成し、双方に `関連: [[ノート名]] — 一言説明` の相互リンクを張る
   - 既存ノートのスタイル（見出し構成・書き方）に合わせる
4. ノートの基本構成:

```markdown
# タイトル（内容がわかる日本語見出し）

導入段落（何の手法・情報のまとめかを2〜3文で）。

> 出典: 動画タイトル（チャンネル名 / 発表者）
> https://www.youtube.com/watch?v=...

関連: [[既存ノート]] — 違いの一言説明（あれば）

## セクション（概要・手順・比較・所感など内容に応じて）
...
```

5. 資料化の方針:
   - **技術情報・手順・具体的な数値を抽出する**。話し言葉の再現ではなく、読んで使える資料にする
   - 宣伝部分（LINE登録特典・コミュニティ誘導・自社講座の宣伝など）は本文から除外し、除外した旨を末尾の「所感・注意」に一行記す
   - 動画内で口頭で述べられたプロンプト例はコードブロックで整形して載せる
   - Whisperの誤変換と思われる固有名詞（例: 「コーデックス」→ Codex、「ファスタービスパー」→ faster-whisper）は正しい表記に直す。確証がない固有名詞は無理に直さず文脈を保つ
