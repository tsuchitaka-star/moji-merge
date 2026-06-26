# モジマージ綱引き 残機能・実装仕様書

**更新日**: 2026-06-26
**対象**: Codex（実装担当）
**前提**: `prototype/moji-merge-tsunahiki-bold-concept.html` は単一HTMLでvs CPU戦のみ実装済み。本ドキュメントは「公開可能なゲームとして完成」させるために必要な機能を網羅する。

## 0. 優先度区分

| 区分 | 意味 |
|---|---|
| P0 | リリースブロッカー。Phase 1（個人プレイ版公開）に必須 |
| P1 | 公開直後に追加したい（体験向上に直結） |
| P2 | 第2〜3リリースで追加（継続性・対戦のため） |
| P3 | あったら嬉しいレベル（後回し可） |

実装するかしないかは人間が判断する。本書は「全選択肢を提示する」ことを目的とする。

---

## 1. 画面構成と遷移

### 1.1 画面フロー全体図

```
[起動]
   ↓
[Splash / Loader] ─── PWA インストール案内
   ↓
[Title / Home画面] ←─── 全画面から戻れる
   ├── 🆚 Play vs CPU       → [難易度選択] → [In-game]
   ├── 🎯 Daily Challenge   → [In-game (固定シード)]
   ├── 🌐 Online Match      → [Matchmaking] → [In-game]
   ├── 📊 Stats / 統計
   ├── 🏆 Leaderboard
   ├── 🏅 Achievements
   ├── ⚙️ Settings
   ├── ❓ How to Play
   └── ℹ️ Credits

[In-game]
   ├── ESC で [Pause Menu]
   │     ├── Resume
   │     ├── Restart
   │     ├── Settings
   │     └── Quit to Home
   └── 試合終了 → [Result]
         ├── もう一度（同設定）
         ├── 違う設定で
         ├── リプレイを見る (P3)
         ├── シェア
         └── ホームへ
```

### 1.2 ルーティング (P1)

SPA で hash router を採用。

| URL | 画面 |
|---|---|
| `#/` | Home |
| `#/play/easy\|normal\|hard\|expert` | vs CPU |
| `#/daily` | Daily Challenge |
| `#/online` | Matchmaking |
| `#/room/{roomId}` | Online Room |
| `#/stats` | Statistics |
| `#/leaderboard/{period}` | Leaderboard |
| `#/achievements` | Achievements |
| `#/settings` | Settings |
| `#/help` | How to Play |

ブラウザバック対応必須。

---

## 2. 画面別仕様

### 2.1 Title / Home画面 (P0)

**レイアウト**:
- 上部: 巨大ロゴ「モジマージ 綱引き」（既存CSS流用、bold-concept路線維持）
- 中央: メニューボタン群（大きく、太フチ）
  - vs CPU（メイン）
  - Daily Challenge
  - Online（Phase 3）
  - Stats / Achievements / Settings 等は脇
- 下部:
  - 自己ベストスコア表示
  - 連続ログイン日数
  - バージョン番号、SNSリンク
- 背景: **デモプレイ**（CPU vs CPU が無音で進行、操作不可）

**動作**:
- 初回起動時は自動的に How to Play へ
- 2回目以降はそのまま Home

**ファイル**: `src/screens/home.ts`

### 2.2 難易度選択画面 (P1)

**CPU難易度プリセット**:

| 難易度 | cpuSpeed初期 | 加算係数 | 試合時間 | DANGER しきい値 |
|---|---|---|---|---|
| Easy | 0.00028 | 0.000025 | 60秒 | -0.75 |
| Normal | 0.00040 | 0.000035 | 45秒 | -0.62（現状） |
| Hard | 0.00055 | 0.000045 | 30秒 | -0.50 |
| Expert | 0.00070 | 0.000055 | 30秒 | -0.40 |

**表示要素**:
- 各難易度カード（5秒のデモプレイループを背景）
- 「初心者おすすめ」「自己ベスト挑戦」などのラベル
- 戻るボタン

### 2.3 In-game (既存) — 必要な追加 (P0)

現状実装済み。以下を追加:

- **ポーズボタン**: 右上に小さく ☰ ボタン、押すとポーズメニュー
- **ESC** で同様
- ポーズ中はキー入力ブロック、`requestAnimationFrame` 停止
- 再開時に `state.start` をずらして経過時間補正
- **試合中の入力統計を記録**:
  ```ts
  state.metrics = {
    totalKeys: 0,
    correctKeys: 0,
    errorCount: 0,
    wordsCleared: { L1: 0, L2: 0, L3: 0, L4: 0 },
    maxCombo: 0,
    ropeHistory: [], // [{ts, rope}] - 100ms間隔
  };
  ```

### 2.4 Result画面 (P0)

**現状**: WIN/LOSE/DRAW テキストのみ。

**追加要素**:

#### ヘッダー
- 結果バッジ（WIN/LOSE/DRAW）+ アニメーション付き
- 「自己ベスト更新！」ラベル（該当時）
- 「○連勝」「○連勝ストップ」表示

#### スコア統計（カード型）
- **総スコア**（大きく中央に）
- **詳細**:
  - 撃破した単語数（レベル別）
  - 最高コンボ
  - 平均タイピング速度（kana/秒）
  - 正確率（正タイプ / 全タイプ）
  - 試合時間
  - 累計味方人数 vs CPU人数

#### グラフ
- **綱位置の時系列**: 試合中の rope の動き（折れ線、X軸=時間、Y軸=rope）
- **コンボ推移**: 棒グラフ

#### 獲得メダル/アチーブメント
- 当該試合で達成した実績を一覧
- 例: 「20コンボ達成」「L4を3つ撃破」「ノーミス勝利」

#### アクションボタン群
- もう一度（同じ難易度）
- 違う設定で（難易度選択へ）
- リプレイを見る (P3)
- 結果をシェア:
  - X (Twitter) 投稿テンプレ
  - 画像ダウンロード（canvas で生成）
- 統計を見る
- ホームへ戻る

**データ保存**:
- 試合完了時、`stats` と `history[]` を LocalStorage に書き込み
- `bestScore`, `longestStreak` 等を更新

### 2.5 Statistics画面 (P1)

**タブ構成**:

#### 概要タブ
- 累計プレイ数、勝率、累計プレイ時間（hh:mm 表示）
- 自己ベスト（スコア、コンボ、最長連勝）
- 最近の戦績（直近10戦、結果アイコン）

#### 試合履歴タブ
- 一覧（日付・モード・結果・スコア、ソート可能）
- 各行クリックで詳細モーダル

#### ワード分析タブ
- よく出会う単語ランキング（撃破数 top 20）
- 苦手単語（ミス率 高 top 10）
- 早打ちチャンピオン単語（平均タイム最速 top 10）

#### グラフタブ
- 日別プレイ数（過去30日）
- 平均スコア推移
- 平均コンボ推移

**ファイル**: `src/screens/stats.ts`

### 2.6 Settings画面 (P0)

#### 表示
- 文字サイズ: 90% / 100% / 120% / 150%
- 高コントラストモード（白黒＋黄色）
- シェイク強度（0% / 50% / 100%）
- ダークモード（OS連動 / 強制ON / 強制OFF）

#### 音響
- BGM音量（0–100%）
- 効果音音量（0–100%）
- 全ミュート切替

#### 入力
- ローマ字方式の表示優先（Hepburn / Kunrei / 表示しない）
- キーマップ表示（変更は P3）

#### ゲームプレイ
- 試合終了後の自動リトライ
- DANGER 表示の感度（早め / 標準 / 遅め）
- カウントダウン秒数（3 / 5 / 10）

#### データ
- 統計データをエクスポート（JSONダウンロード）
- 統計データをインポート
- **全データリセット**（確認ダイアログ二段階）

### 2.7 How to Play / チュートリアル (P0)

**初回起動時に自動表示**、後から `#/help` で再表示可能。

**ステップ構成**（左右スワイプで進む）:
1. ゲームの目的（綱引きで相手を引き込む）
2. ローマ字入力の基本（`chi` でも `ti` でもOKという話）
3. 単語打ち切りで味方が増える、難しいほど大量に
4. COMBOを5まで繋ぐと応援ラッシュ
5. CPU も同時に打っている、放置するとやられる
6. DANGER / CHANCE の見方
7. 「やってみる！」ボタン → 練習モード or 通常モード

**実装**: フルスクリーンオーバーレイ、各ステップで該当UIをハイライト＋矢印。

### 2.8 Pause Menu (P1)

ゲーム中 ESC か ☰ ボタンで開く。背景はゲーム画面を暗くしてオーバーレイ。

メニュー:
- ▶ 続ける（Resume）
- 🔄 やり直す（Restart）— 確認ダイアログ
- ⚙️ 設定（簡易版）
- 🏠 ホームに戻る — 確認ダイアログ

ポーズ中の挙動:
- `requestAnimationFrame` 停止
- キー入力ブロック（Resume ボタンのみ反応）
- BGM はそのままフェードアウト or 継続

### 2.9 Leaderboard画面 (P2)

タブ:
- グローバル（全期間）
- ウィークリー
- デイリー（当日 Daily Challenge）
- フレンド（P3）

各行: 順位、表示名、スコア、難易度、日付。

自分の順位は固定表示（上部 or 下部）。

### 2.10 Achievements画面 (P2)

実績一覧（取得済み / 未取得を区別表示）。

カテゴリ:
- 戦績系（勝利数、連勝、ノーミス）
- コンボ系（10/20/30）
- 単語系（撃破数、特定単語）
- デイリー系（連続日数）
- イースターエッグ系

---

## 3. ゲーム本体の追加機能

### 3.1 ポーズ機能 (P0)
仕様: 2.8 参照。

### 3.2 自動セーブ (P0)
- 試合終了時、`stats` 更新と `history[]` 追加
- セーブ失敗（容量超過）時はサイレントに古い履歴を削除

### 3.3 入力統計の収集 (P0)
試合中、以下を `state.metrics` に蓄積:
- 全打鍵数 / 正タイプ / 誤タイプ
- レベル別撃破数
- 最高コンボ
- rope の時系列（100ms毎）

リザルト画面で使用。

### 3.4 リプレイ (P3)
- 試合中の `metrics.inputs[]` を記録（`{ts, char, result}`）
- 再生時は同じ単語シードで再現
- 共有用にエンコードして URL に埋め込み

---

## 4. データモデル（LocalStorage）

### 4.1 スキーマ (P0)

```ts
interface LocalSave {
  version: 1;
  playerId: string;       // 'anon-' + UUID
  displayName: string | null;
  createdAt: string;      // ISO 8601
  settings: Settings;
  stats: Stats;
  achievements: string[]; // ID list
  history: MatchRecord[]; // 直近 100 件
  dailyChallenge: {
    lastPlayedDate: string;
    bestToday: number;
  };
}

interface Settings {
  bgmVolume: number;      // 0-1
  seVolume: number;       // 0-1
  shake: number;          // 0-1
  preferredRomaji: 'hepburn' | 'kunrei' | 'none';
  highContrast: boolean;
  fontSize: number;       // 0.9 / 1.0 / 1.2 / 1.5
  autoRetry: boolean;
  dangerThreshold: 'early' | 'normal' | 'late';
  countdownSec: 3 | 5 | 10;
  darkMode: 'system' | 'light' | 'dark';
}

interface Stats {
  totalMatches: number;
  wins: number;
  losses: number;
  draws: number;
  totalPlayMs: number;
  bestScore: number;
  bestCombo: number;
  longestStreak: number;
  currentStreak: number;
  wordHits: { L1: number; L2: number; L3: number; L4: number };
  totalKeystrokes: number;
  totalErrors: number;
}

interface MatchRecord {
  id: string;
  date: string;
  mode: 'cpu' | 'daily' | 'online';
  difficulty: 'easy' | 'normal' | 'hard' | 'expert';
  result: 'win' | 'lose' | 'draw';
  score: number;
  maxCombo: number;
  accuracy: number;       // 0-1
  durationMs: number;
  wordsCleared: { L1: number; L2: number; L3: number; L4: number };
  ropeHistory: number[];  // 100ms 毎のサンプル
}
```

### 4.2 マイグレーション (P1)

`version` フィールドで管理。フォーマット変更時:
```ts
function migrate(save: any): LocalSave {
  if (save.version === 1) return save;
  // 将来のフォーマット変更に備える
  // ...
}
```

### 4.3 ストレージ容量管理 (P1)

- `history[]` は最新100件のみ保持、古いものは要約のみ保存
- 容量超過時、ユーザーに通知して古い履歴削除を提案

---

## 5. デイリーチャレンジ (P2)

### 5.1 仕様
- 毎日 0:00 (JST) に新規チャレンジ
- 全プレイヤー共通のシード（単語・出現順）
- 1日1回のみランキング登録（再挑戦可、ベストのみ）
- 残り時間 / 試合時間は固定（30秒 など）

### 5.2 シード生成
```ts
const dateSeed = new Date().toLocaleDateString('ja-JP', { timeZone: 'Asia/Tokyo' });
// "2026/6/26"
const rng = seededRng(hashString(dateSeed));
```

`seededRng` は決定的な PRNG（mulberry32 等）。

### 5.3 共有
シェア用テンプレ:
```
モジマージ綱引き 2026-06-26
スコア 5,800 / コンボ 23
WIN!
#モジマージ
{url}
```

URL は固定（毎日のチャレンジ URL）。

---

## 6. アチーブメント (P2)

### 6.1 一覧（例）

| ID | 名前 | 達成条件 |
|---|---|---|
| `first_win` | 初勝利 | 初めて勝つ |
| `combo_10` | コンボ10 | 1試合でコンボ10達成 |
| `combo_20` | 二十連 | 1試合でコンボ20達成 |
| `combo_30` | 神域 | 1試合でコンボ30達成 |
| `perfect` | 完全無欠 | ミス0で勝利 |
| `comeback` | 大逆転 | rope < -0.5 から勝利 |
| `daily_streak_7` | 七日連続 | デイリーを7日連続 |
| `daily_streak_30` | 一ヶ月皆勤 | デイリーを30日連続 |
| `wordmaster_100` | 百打突破 | 通算100語撃破 |
| `wordmaster_1000` | 千打突破 | 通算1000語撃破 |
| `lv4_killer` | 大物喰い | 1試合でL4を3つ撃破 |
| `speedster` | 高速タイパー | 平均5kana/秒以上で勝利 |
| `comeback_king` | 逆境の王 | comeback × 10 |
| `early_bird` | 早起き | 朝6時にプレイ |
| `night_owl` | 夜更かし | 深夜2時にプレイ |

### 6.2 UI
- トースト通知（達成時、画面上部から滑り込み）
- Achievements 画面で全一覧
- 未取得は灰色＋ヒント表示

---

## 7. 対戦システム

### 7.1 マッチング基本仕様 (P2)

#### クイックマッチ
- ボタン1つでマッチング開始
- 待機画面 → マッチ成立 → 試合開始
- レーティング近い相手を優先
- 30秒で見つからなければ CPU フォールバック

#### フレンドマッチ
- 招待 URL 生成（`https://app.url/#/room/abc123`）
- URL を相手に共有 → 開いた相手がルーム入室
- 両者揃ったら自動開始（5秒カウントダウン）
- ルームは10分で expire

#### ランクマッチ (P3)
- 月次シーズン制
- ティア: Bronze / Silver / Gold / Platinum / Diamond / Master
- 試合結果でレーティング変動（Elo K=32）

### 7.2 サーバー API (P2)

Cloudflare Workers + Hono を想定。

```
POST /api/matchmaking/queue
  body: { playerId, rating?, mode: "quick" | "ranked" }
  response: { ticket }

GET /api/matchmaking/status?ticket=XXX
  response: { status: "waiting" | "matched", roomId?, wsUrl? }

DELETE /api/matchmaking/queue?ticket=XXX
  キャンセル

POST /api/room/create
  body: { playerId }
  response: { roomId, joinUrl, wsUrl, expiresAt }

GET /api/room/:roomId/status
  response: { status, players, expiresAt }

WS /ws/:roomId
  双方向メッセージング
```

### 7.3 同期メッセージ (P2)

サーバー権威型。

クライアント → サーバー:
```ts
{ type: "ready" }
{ type: "input", char: "a", ts: 12345 }
{ type: "leave" }
```

サーバー → クライアント:
```ts
{ type: "match_start", word, opponentWord, seed, startAt }
{ type: "opponent_progress", typed: "ame", word }
{ type: "opponent_finish", word, lvl, newWord, ropeDelta }
{ type: "tick", rope, time, playerPeople, opponentPeople }
{ type: "match_end", result, stats }
{ type: "opponent_left" }
```

### 7.4 切断・再接続 (P2)
- 30秒以内に再接続できれば継続
- それ以上は不戦敗、相手は不戦勝
- 切断時の BOT 引き継ぎ（P3）

### 7.5 アンチチート (P2 軽量, P3 本格)
- 入力レートの上限（人間が打てる速度を超えたら警告）
- 単語完了の最低時間チェック
- リクエスト署名（HMAC）

---

## 8. グローバルランキング (P2)

### 8.1 データモデル

Cloudflare D1。

```sql
CREATE TABLE players (
  player_id TEXT PRIMARY KEY,
  display_name TEXT,
  rating INTEGER DEFAULT 1000,
  created_at TEXT
);

CREATE TABLE leaderboard (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  player_id TEXT,
  mode TEXT,           -- 'all_time' | 'weekly' | 'rated'
  score INTEGER,
  combo INTEGER,
  difficulty TEXT,
  recorded_at TEXT
);

CREATE TABLE daily_challenge (
  date TEXT,
  player_id TEXT,
  score INTEGER,
  stats TEXT,          -- JSON
  PRIMARY KEY (date, player_id)
);

CREATE INDEX idx_leaderboard_mode_score ON leaderboard(mode, score DESC);
CREATE INDEX idx_daily_date_score ON daily_challenge(date, score DESC);
```

### 8.2 API

```
GET /api/leaderboard/all_time?limit=100&offset=0
GET /api/leaderboard/weekly?limit=100
GET /api/leaderboard/daily/:date?limit=100
GET /api/leaderboard/me
POST /api/leaderboard/submit
  body: { mode, score, combo, difficulty, signature }
```

`signature` で改竄防止（playerId + score + ts を HMAC）。

---

## 9. 音響

### 9.1 BGM (P1)

| 状態 | 曲調 | ループ |
|---|---|---|
| タイトル | 軽快・明るい | ◯ |
| 試合中（通常） | テンポ早め | ◯ |
| 試合中（DANGER） | 緊張感UP（オーバーラップ） | ◯ |
| 試合中（応援ラッシュ） | 高揚感（オーバーラップ） | ◯ |
| 勝利 | ファンファーレ | ✗ |
| 敗北 | 残念フレーズ | ✗ |
| 引き分け | ニュートラル | ✗ |

**実装**:
- Web Audio API
- `<audio>` タグ + GainNode で音量制御
- BGM切替はクロスフェード 0.5秒

### 9.2 SE (P1)

| イベント | 候補音 |
|---|---|
| 打鍵（1文字） | ピッ（軽め） |
| 単語完了 | パパッ＋拍手 |
| ミス | ブッ |
| 撃破 L1 | 軽い拍手 |
| 撃破 L2 | やや派手 |
| 撃破 L3 | ジャラン |
| 撃破 L4 | ドンッ＋大拍手 |
| 応援ラッシュ開始 | チャラララ |
| DANGER 警告 | ピロピロ |
| CHANCE | 上昇音 |
| CPU 撃破 | 重い「ぐっ」 |
| 試合終了 WIN | パフッ＋紙吹雪 |
| 試合終了 LOSE | 「あー」 |
| カウントダウン | 鐘 |
| ボタン | クリッ |
| メニュー開閉 | スッ |

### 9.3 アセット (P1)

- 形式: OGG (Chromium/Firefox) + MP3 (Safari)
- 容量目安: BGM × 4曲、各 60–90秒、< 1.5MB／曲。SE × 12種、各 < 50KB
- ライセンス: CC0 or 自作

ファイル配置:
```
public/audio/bgm/
  title.ogg, title.mp3
  match_normal.ogg, ...
public/audio/se/
  type.ogg, type.mp3
  ...
```

---

## 10. PWA / モバイル対応

### 10.1 PWA 設定 (P1)

`public/manifest.json`:
```json
{
  "name": "モジマージ 綱引き",
  "short_name": "モジマージ",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#82616d",
  "theme_color": "#e60012",
  "orientation": "any",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

`public/sw.js`:
- Workbox 推奨
- 静的アセット（HTML/JS/CSS/音/画像）は cache-first
- API は network-first + cache fallback

インストール促進:
- 3回目訪問時にバナー表示

### 10.2 モバイル UX 改善 (P0)

現状の課題:
- `hidden-input` が見えにくく、フォーカスしにくい
- ソフトキーボード表示時に画面下半分が隠れる

改善:
- **モバイル縦画面では別レイアウト**:
  - キャンバスを上 50%
  - 単語カードを真ん中
  - 入力欄を下 30%（タップで明示的にキーボード呼び出し）
- 横画面は従来通り
- 「タップでキーボード表示」を大きく案内
- iOS の zoom on focus を防ぐ (`input { font-size: 16px+ }`)

### 10.3 タッチ補助操作 (P2)

完全タイピング以外の選択肢:
- 単語をタップで「一文字進める」（タイピング苦手な人向け）
- フリック入力対応（P3）

---

## 11. アクセシビリティ

### 11.1 視覚 (P1)
- DANGER/CHANCE: 色だけでなく形・アイコンも変える（⚠ / ⭐）
- 高コントラストモード: 白黒＋黄色アクセント、棒人間は白＋太い黒
- 文字サイズ: 設定で選択可
- 色覚多様性配慮: 青/赤の区別が苦手な人向けにオレンジ/紫の代替パレット

### 11.2 聴覚 (P1)
- 全効果音に視覚代替（既存の画面フラッシュ＋アイコン）
- 字幕は不要（ナレーション無し）

### 11.3 入力 (P2)
- キーボードのみで全画面操作可能（Tab / Enter / Esc）
- スクリーンリーダー対応（`aria-label`, `aria-live`）
- フォーカス可視化（focus ring を明示的に）

### 11.4 動作軽減 (P1)
- `prefers-reduced-motion` 対応
- ON時: シェイクOFF、確認のための minimum animation のみ

---

## 12. アナリティクス・運用

### 12.1 テレメトリ (P3, opt-in)

匿名統計の例:
- 試合数、勝率、平均試合時間
- 平均コンボ、平均スコア
- 単語別ヒット/ミス率
- 端末カテゴリ（mobile / desktop, brand）

実装: Cloudflare Analytics or Plausible（Cookieless）。

初回起動時に「匿名統計に協力するか」を聞く。

### 12.2 エラーログ (P1)

- フロント: try/catch + `window.addEventListener('error', ...)`
- Worker 側に POST して KV に保存（30日でTTL）
- 個人情報を含まない

### 12.3 A/B テスト (P3)
- 難易度バランス検証
- UI 配置の効果検証
- 確率分割は LocalStorage で固定

---

## 13. テスト・QA

### 13.1 単体テスト (P1)

Vitest 推奨。

**対象**:
- `tokenizeRoma`: 全カナ、拗音（しゃ/しゅ/しょ）、長音、促音（っ）
- `checkInput`: prefix/complete/invalid の境界
- `buildHint`: 入力途中での表示が正しい
- 単語辞書の整合性（重複なし、全単語が tokenize 成功）
- スコア計算、コンボ計算、応援ラッシュ判定

ファイル:
```
tests/romaji.test.ts
tests/words.test.ts
tests/game.test.ts
```

### 13.2 E2E テスト (P2)

Playwright。

シナリオ:
- 起動 → ホーム表示
- 難易度選択 → 試合 → 終了 → リザルト
- 設定変更 → 反映確認
- レイアウト崩れチェック（Visual Regression）

### 13.3 体感バランステスト (P0)
- 50人規模のクローズドβ（友人・SNS）
- 難易度別の勝率分布をモニタ
- 「初心者が連敗しすぎないか」「上級者が圧勝しすぎないか」

---

## 14. 技術基盤

### 14.1 リポジトリ構成 (P1)

現状の HTML プロトタイプから、TypeScript + Vite の構造へ移行:

```
/
├── public/
│   ├── manifest.json
│   ├── sw.js
│   ├── icons/
│   ├── audio/
│   └── og.png
├── src/
│   ├── main.ts                 # エントリ
│   ├── router.ts               # hash router
│   ├── game/
│   │   ├── state.ts
│   │   ├── words.ts            # POOL, makeWords, shuffle
│   │   ├── romaji.ts           # SYLLABLE_VARIANTS, tokenizeRoma, checkInput, buildHint
│   │   ├── physics.ts          # rope state update
│   │   ├── render.ts           # canvas drawing
│   │   └── ai.ts               # CPU 入力シミュ
│   ├── screens/
│   │   ├── home.ts
│   │   ├── selectDifficulty.ts
│   │   ├── game.ts
│   │   ├── result.ts
│   │   ├── stats.ts
│   │   ├── settings.ts
│   │   ├── help.ts
│   │   └── leaderboard.ts
│   ├── audio/
│   │   ├── bgm.ts
│   │   └── se.ts
│   ├── storage/
│   │   ├── localdb.ts          # LocalSave 読み書き
│   │   └── migrate.ts
│   ├── net/
│   │   ├── api.ts
│   │   └── ws.ts
│   ├── ui/
│   │   ├── components/
│   │   │   ├── Button.ts
│   │   │   ├── Modal.ts
│   │   │   ├── Toast.ts
│   │   │   └── Pill.ts
│   │   └── theme.ts
│   ├── achievements/
│   │   ├── catalog.ts
│   │   └── checker.ts
│   └── i18n/
│       ├── ja.ts
│       └── en.ts
├── workers/
│   ├── matchmaking/
│   │   ├── src/index.ts
│   │   └── wrangler.toml
│   ├── room/                   # Durable Objects
│   ├── leaderboard/
│   └── shared/
│       └── types.ts
├── shared/
│   └── protocol.ts             # WS メッセージ型
├── tests/
├── docs/
└── prototype/                  # 現状の HTML を保管
```

### 14.2 ビルド (P1)

- **Vite + TypeScript**
- Phaser 3 は不要（既存の Canvas 直書きで十分軽い）
- CSS は CSS Modules or vanilla で

`vite.config.ts`:
```ts
export default {
  plugins: [VitePWA({ ... })],
  build: { target: 'es2020' }
};
```

### 14.3 CI (P2)

GitHub Actions:
- PR時: lint, test, build
- main merge時: Cloudflare Pages 自動デプロイ
- Workers は `wrangler deploy` を別ジョブで

### 14.4 環境変数 (P2)

- `VITE_API_URL`: Workers エンドポイント
- `VITE_WS_URL`: WebSocket エンドポイント
- `VITE_ANALYTICS_KEY`: Plausible 用

---

## 15. セキュリティ・コンプライアンス

### 15.1 入力検証 (P1)
- ユーザー入力（display name など）は HTML escape
- XSS 対策: `innerHTML` 利用箇所を見直し、`textContent` 優先

### 15.2 個人情報 (P0)
- 個人情報は収集しない方針
- `playerId` は端末ローカル UUID、サーバーに紐づくがメール等は無し
- プライバシーポリシーページ（P1）

### 15.3 利用規約 (P1)
- 簡潔な利用規約（不適切な表示名禁止等）

---

## 16. ロードマップ・リリース計画

### Phase 1 — 個人プレイ完成版

**目標**: 単一プレイヤーでも楽しめる完成度。

含まれる:
- タイトル画面（2.1）
- 難易度選択（2.2）
- ポーズ機能（2.8）
- リザルト画面詳細化（2.4）
- 設定画面（2.6）
- チュートリアル（2.7）
- LocalStorage 保存（4.1）
- PWA 対応（10.1）
- BGM/SE（9.1, 9.2）
- 基本テスト（13.1）
- アクセシビリティ基本（11.1, 11.4）

**リリース**: Cloudflare Pages
**期間目安**: 2〜4週間

### Phase 2 — 統計・継続性

含まれる:
- Statistics画面（2.5）
- Achievements（6.1）
- Daily Challenge（5.1）
- Leaderboard（8）
- マイグレーション（4.2）
- E2E テスト（13.2）

**期間目安**: 2〜3週間

### Phase 3 — オンライン対戦

含まれる:
- フレンドマッチ（7.1）
- クイックマッチ（7.1）
- WebSocket同期（7.3）
- 切断対応（7.4）
- 軽量アンチチート（7.5）

**期間目安**: 3〜4週間

### Phase 4 — 拡張

- ランクマッチ（7.1）
- リプレイ（3.4）
- 多言語（11以外）
- A/B テスト（12.3）

---

## 17. 優先度マトリクス（全機能一覧）

| ID | 機能 | 優先度 | Phase |
|---|---|---|---|
| 2.1 | Title/Home画面 | P0 | 1 |
| 2.2 | 難易度選択 | P1 | 1 |
| 2.3 | In-game追加（ポーズボタン、metrics） | P0 | 1 |
| 2.4 | Result画面詳細化 | P0 | 1 |
| 2.5 | Statistics画面 | P1 | 2 |
| 2.6 | Settings画面 | P0 | 1 |
| 2.7 | How to Play | P0 | 1 |
| 2.8 | Pause Menu | P1 | 1 |
| 2.9 | Leaderboard画面 | P2 | 2 |
| 2.10 | Achievements画面 | P2 | 2 |
| 3.1 | ポーズ機能 | P0 | 1 |
| 3.2 | 自動セーブ | P0 | 1 |
| 3.3 | 入力統計 | P0 | 1 |
| 3.4 | リプレイ | P3 | 4 |
| 4.1 | LocalStorageスキーマ | P0 | 1 |
| 4.2 | マイグレーション | P1 | 2 |
| 4.3 | 容量管理 | P1 | 2 |
| 5 | Daily Challenge | P2 | 2 |
| 6 | アチーブメント | P2 | 2 |
| 7.1 | マッチング | P2 | 3 |
| 7.2 | API | P2 | 3 |
| 7.3 | 同期メッセージ | P2 | 3 |
| 7.4 | 切断対応 | P2 | 3 |
| 7.5 | アンチチート | P2/P3 | 3 |
| 8 | グローバルランキング | P2 | 2 |
| 9.1 | BGM | P1 | 1 |
| 9.2 | SE | P1 | 1 |
| 9.3 | 音響アセット | P1 | 1 |
| 10.1 | PWA設定 | P1 | 1 |
| 10.2 | モバイルUX | P0 | 1 |
| 10.3 | タッチ補助 | P2 | 4 |
| 11.1 | 視覚アクセシビリティ | P1 | 1 |
| 11.2 | 聴覚 | P1 | 1 |
| 11.3 | キーボードのみ | P2 | 2 |
| 11.4 | reduced-motion | P1 | 1 |
| 12.1 | テレメトリ | P3 | 4 |
| 12.2 | エラーログ | P1 | 1 |
| 12.3 | A/B テスト | P3 | 4 |
| 13.1 | 単体テスト | P1 | 1 |
| 13.2 | E2E テスト | P2 | 2 |
| 13.3 | 体感バランステスト | P0 | 1 |
| 14.1 | TS+Vite 移行 | P1 | 1 |
| 14.2 | ビルド | P1 | 1 |
| 14.3 | CI | P2 | 2 |
| 14.4 | 環境変数 | P2 | 2 |
| 15.1 | 入力検証 | P1 | 1 |
| 15.2 | 個人情報 | P0 | 1 |
| 15.3 | 利用規約 | P1 | 2 |

---

## 18. 既存コードからの移行ガイド

### 18.1 現状ファイル
- `prototype/moji-merge-tsunahiki-bold-concept.html` (979行)
- 単一HTML内に CSS, HTML, JS が全部混在
- グローバル変数で state 管理

### 18.2 移行戦略
1. `src/game/romaji.ts` に `SYLLABLE_VARIANTS`, `tokenizeRoma`, `checkInput`, `buildHint` を抽出
2. `src/game/words.ts` に POOL, makeWords, shuffle を抽出
3. `src/game/state.ts` に `state` の型定義と reset 関数
4. `src/game/render.ts` に draw, drawPerson, drawRopePath, drawBraidedRope, curvePoints を抽出
5. `src/game/ai.ts` に CPU 入力シミュレーション抽出
6. `src/screens/game.ts` で上記を組み合わせて試合ループ

### 18.3 互換性
- 既存の HTML プロトタイプは `prototype/` に残し、Phase 1 で `public/index.html` に置き換える
- 単語辞書、ローマ字判定ロジックは破壊的変更を避ける

---

## 19. 補足: 体感調整パラメータ一覧

将来チューニングしたい数値:

| 変数 | 現在値 | 範囲 |
|---|---|---|
| `cpuSpeed (initial)` | 0.00040 | 0.00020〜0.00080 |
| `cpuSpeed (per word)` | 0.00034 + rand 0.00016 + lv×0.000035 | 同上比率 |
| `rope drift force` | (player − cpu) × 0.000019 | 同 × 0.5〜2.0 |
| `rope per word` | 0.045 × lv | 0.030〜0.060 |
| `combo for rush` | 5 | 3〜7 |
| `rush duration` | 4300ms | 3000〜6000 |
| `DANGER threshold` | -0.62 | -0.40〜-0.80 |
| `time limit` | 45000ms | 30000〜90000 |
| `max people` | 60 | 30〜100 |

これらは Phase 1 体感βで調整。

---

## 20. 最後に — このドキュメントの使い方

- Codex は本書を**章単位で着手**できる粒度で書いてある
- 各機能の見出し（2.1, 2.2…）を Issue や PR タイトルに使うと進捗管理しやすい
- 不明点があれば人間に確認、Phase 1 を最優先で進める
- 仕様書自体に矛盾があれば、本書を「Source of Truth」とし、`current-gameplay-spec.md` と差分が出たら本書を優先する

実装するかしないかは**人間が最終判断**する。
