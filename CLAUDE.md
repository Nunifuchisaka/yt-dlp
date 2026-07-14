# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このディレクトリについて

ここは yt-dlp の**ソースコードリポジトリではない**。`yt-dlp.exe`（実行バイナリ）を置いて動画をダウンロードするための個人用作業ディレクトリ。ダウンロードした動画ファイル（`*.mp4`, `*.webm`）もこの直下に保存される。

- `yt-dlp.exe` — 実行バイナリ本体（gitignore済み）
- `yt-dlp.conf` — exe と同じディレクトリに置かれたポータブル設定ファイル。**すべての実行に自動で適用される**
- `cookies.txt` — YouTube 認証用クッキー（gitignore済み）
- `tmp/` — 作業用フォルダ

## よく使うコマンド

`D:\yt-dlp` はユーザー環境変数 PATH に追加済みのため、どのディレクトリからでも `yt-dlp` だけで実行できる。`yt-dlp.conf` は exe と同じディレクトリのポータブル設定として、どこから実行しても自動適用される。ただし保存先の指定がないため、**動画は実行時のカレントディレクトリに保存される**点に注意。

```powershell
# ダウンロード（yt-dlp.conf が自動適用される）
yt-dlp "https://www.youtube.com/watch?v=..."

# 利用可能なフォーマット一覧
yt-dlp -F "URL"

# 設定ファイルを無視して素の挙動で実行（切り分け用）
yt-dlp --ignore-config "URL"

# yt-dlp 本体の更新
yt-dlp -U
```

## yt-dlp.conf の要点

設定を変更・診断するときは以下の意図を踏まえること。

- **認証**: `--cookies "D:/yt-dlp/cookies.txt"` でログイン状態を使う
- **SABR 制限回避**: `--js-runtimes node` + `--extractor-args "youtube:player-client=ios,tv,web;player-params=2"`。**Node.js がインストールされている前提**。ダウンロードが壊れたときはまずこの周辺（yt-dlp のバージョンと extractor-args の組み合わせ）を疑う
- **エンコード**: `bestvideo+bestaudio` を mp4 にマージし、ffmpeg の `h264_nvenc`（NVIDIA GPU / RTX 5070 Ti 前提）で再エンコードする。CPU エンコードに変えない
- **副作用**: `--mark-watched` により、ダウンロードした動画は YouTube 上で視聴済みになる

## 注意事項

- `cookies.txt` の中身は認証情報そのもの。**読み取り・表示・コミットを絶対にしない**
- git 管理対象は設定類のみ（`.gitignore` で cookies・exe・動画を除外）。現在この環境では git が「dubious ownership」エラーで動かない。git 操作が必要な場合のみ `git config --global --add safe.directory D:/yt-dlp` で例外を追加する
- ダウンロード済みの動画ファイルを削除・移動しない
