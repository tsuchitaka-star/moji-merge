# モジマージ オンライン環境仕様

更新日: 2026-06-26

## 目的

モジマージを、スマホとPCの両方で遊べるWeb/PWAゲームとして公開し、将来的にランキングとオンライン対戦へ拡張する。

基本方針は「できるだけ無料枠で進める」。最初から有料サーバーを持たず、Cloudflareの無料枠を中心に構成する。

## 採用方針

### 第1リリース

- スマホ/PC両対応のWeb版
- PWA対応
- 個人プレイ
- ローカルスコア
- GitHub + Cloudflare Pagesで公開

### 第2リリース

- Cloudflare D1でランキング
- デイリーチャレンジ
- 匿名プレイヤーID
- ログインなしで遊べる状態を維持

### 第3リリース

- Cloudflare Workers + Durable Objectsで1対1対戦
- 招待URL
- ルーム制
- WebSocket同期

## 技術スタック

```text
Frontend:
- TypeScript
- Vite
- Phaser 3
- PWA

Hosting:
- Cloudflare Pages

API:
- Cloudflare Workers
- Hono

Realtime Battle:
- Cloudflare Durable Objects
- WebSocket

Database:
- Cloudflare D1

Repository:
- GitHub

Assets:
- GitHub repository
- 必要になったら Cloudflare R2
```

## ディレクトリ構成案

```text
apps/
  web/
    Vite + Phaser + PWA

  worker/
    Cloudflare Workers + Hono
    API
    matchmaking
    ranking
    battle room gateway

packages/
  core/
    merge rules
    scoring
    random seed
    word generation
    battle rules

  types/
    shared TypeScript types

assets/
  generated images
  sounds

docs/
  game design
  graphic spec
  online environment
```

## 無料枠を使う理由

### Cloudflare Pages

Web/PWAの配信先。GitHubと連携して、mainブランチへpushすると自動でビルド/デプロイする。

確認済みの無料枠:

- Free planで月500ビルド
- 1サイト最大20,000ファイル
- 1ファイル最大25MiB
- プレビュー環境も使える

参照:

- https://developers.cloudflare.com/pages/platform/limits/

### Cloudflare Workers

API、ランキング登録、マッチング、対戦入口に使う。

確認済みの無料枠:

- Free planで100,000 requests/day
- CPU timeは1 invocationあたり10ms目安

参照:

- https://developers.cloudflare.com/workers/platform/pricing/

### Cloudflare Durable Objects

リアルタイム対戦のルーム管理に使う。1対1対戦の部屋ごとにDurable Objectを持たせる。

確認済みの無料枠:

- Workers Free planでも利用可能
- Free planではSQLite storage backendのDurable Objectsが利用可能
- Free planでDurable Objects requests 100,000/day

参照:

- https://developers.cloudflare.com/durable-objects/platform/pricing/

### Cloudflare D1

ランキング、日別スコア、匿名ユーザーID、試合履歴の保存に使う。

想定テーブル:

```sql
players
scores
daily_challenges
matches
```

## なぜColyseus/Fly.io/Renderを最初に使わないか

Colyseusはリアルタイムゲーム向けで便利だが、Node.jsの常駐サーバーが必要になる。

無料運用を強く優先する場合、常駐サーバー型は以下の問題がある。

- 無料枠が縮小/終了しやすい
- スリープで初回接続が遅くなることがある
- WebSocket運用で制限に当たりやすい
- 後から料金が発生しやすい

Cloudflare Durable Objectsなら、Cloudflare Workers上でルーム単位の状態管理ができるため、無料開始との相性が良い。

## Supabaseの扱い

Supabaseは便利だが、最初の対戦基盤には使わない。

使うなら将来:

- Auth
- プロフィール
- 管理画面
- 大きくなった後のDB運用

Supabase RealtimeのFree planは、同時接続200、100 messages/secなどの制限がある。ランキングやログインには良いが、対戦の権威サーバーとしてはCloudflare Durable Objectsの方が向いている。

参照:

- https://supabase.com/docs/guides/realtime/limits

## 対戦同期方針

### 基本

サーバー権威型にする。

クライアントは操作を送る。サーバーが結果を決める。

### クライアントが送るもの

- プレイヤー入力
- ブロック配置
- ブロック選択
- 入力完了イベント
- リトライ/退出

### サーバーが管理するもの

- 乱数シード
- ブロック生成順
- スコア
- コンボ
- おじゃま発生
- 勝敗
- 試合時間

### 重要な設計判断

完全な物理同期は避ける。

スイカゲーム風の見た目は維持しつつ、内部ロジックはできるだけ決定論的にする。

候補:

- グリッド寄りの落下
- 簡易物理
- 同レベル隣接で合体
- 見た目だけ跳ねる/揺れる

## PWA対応

スマホとPCの両方をWebで扱う。

必要なもの:

- `manifest.webmanifest`
- アプリアイコン
- Service Worker
- オフライン時の練習モード
- 縦画面9:16のゲームキャンバス
- PCでは中央に縦画面ゲーム、左右にランキング/ステータス

## アカウント/ログイン方針

最初はログインなし。

```text
anonymous_player_id
localStorage
D1 ranking
```

後から必要になったら:

- Googleログイン
- GitHubログイン
- Supabase Auth
- Cloudflare Accessは管理者向けだけに使う

## セットアップ手順

### 1. Cloudflareアカウント作成

- https://dash.cloudflare.com/sign-up
- Free plan

### 2. GitHub連携

- Cloudflare Dashboard
- Workers & Pages
- Pages
- Connect to Git
- `tsuchitaka-star/moji-merge` を選択

### 3. Pages設定

初期実装後の想定:

```text
Framework preset: Vite
Build command: npm run build
Build output directory: apps/web/dist
Root directory: apps/web
```

モノレポ構成にする場合は、後で設定調整する。

### 4. Workers作成

```text
apps/worker
Hono
Cloudflare Workers
```

初期API:

- `GET /health`
- `POST /scores`
- `GET /rankings/daily`
- `POST /matchmaking`

### 5. D1作成

初期テーブル:

```sql
CREATE TABLE players (
  id TEXT PRIMARY KEY,
  display_name TEXT,
  created_at TEXT NOT NULL
);

CREATE TABLE scores (
  id TEXT PRIMARY KEY,
  player_id TEXT NOT NULL,
  score INTEGER NOT NULL,
  combo_max INTEGER NOT NULL,
  accuracy REAL,
  mode TEXT NOT NULL,
  seed TEXT NOT NULL,
  created_at TEXT NOT NULL
);

CREATE TABLE daily_challenges (
  date TEXT PRIMARY KEY,
  seed TEXT NOT NULL,
  word_set TEXT NOT NULL
);

CREATE TABLE matches (
  id TEXT PRIMARY KEY,
  player_a_id TEXT,
  player_b_id TEXT,
  winner_id TEXT,
  seed TEXT NOT NULL,
  started_at TEXT NOT NULL,
  ended_at TEXT
);
```

### 6. Durable Objects作成

対戦用:

```text
BattleRoom
```

責務:

- WebSocket接続管理
- プレイヤー2人の入室
- 試合開始
- 入力イベント処理
- スコア同期
- おじゃま送信
- 勝敗決定

## 直近のTODO

- Cloudflareアカウントを作る
- Cloudflare PagesとGitHubを接続する
- Web/PWAの最小実装を作る
- `packages/core` にゲームロジックを分離する
- 個人プレイをPagesで公開する
- D1ランキングを追加する
- Durable Objects対戦を追加する

## 参照リンク

- Cloudflare Pages limits: https://developers.cloudflare.com/pages/platform/limits/
- Cloudflare Workers pricing: https://developers.cloudflare.com/workers/platform/pricing/
- Cloudflare Durable Objects pricing: https://developers.cloudflare.com/durable-objects/platform/pricing/
- Supabase Realtime limits: https://supabase.com/docs/guides/realtime/limits
- Render pricing: https://render.com/pricing
- Fly.io pricing: https://fly.io/docs/about/pricing/
