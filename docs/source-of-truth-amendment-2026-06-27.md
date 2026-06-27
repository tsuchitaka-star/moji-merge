# Source of Truth 補遺 2026-06-27

この補遺は、2026-06-27時点の最新方針を明文化するものです。
`docs/codex-implementation-spec.md`、`docs/current-gameplay-spec.md`、README、Notionの記載に差分がある場合、この補遺とユーザーの最新指示を最優先します。

## 1. 最優先ルール

- Claude作成のプレイ画面は `prototype/moji-merge-tsunahiki-bold-concept.html` を正とする。
- Codexは、ユーザーの明示許可なくプレイ画面を変更しない。
- 「プレイ画面」には、ゲーム中キャンバス、ロープ描画、キャラクター描画、入力UI、単語カード、CPU進捗、コンボ、勝敗演出、リザルト表示、ゲームルール、単語辞書、難易度計算、CPU速度、ローマ字判定を含む。
- Codexがプレイ画面に触る必要がある場合は、実装前に「どのファイルのどの挙動をどう変えるか」を明記し、ユーザーの明示許可を得る。

## 2. Codexが進めてよい範囲

Codexは、プレイ画面を維持したまま以下を進めてよい。

- ホーム画面、設定、ヘルプ、統計、実績、ランキング、オンライン待機画面などの非プレイ画面
- `prototype/moji-merge-app.html` など、プレイ画面外のアプリ入口
- LocalStorage保存、設定保存、戦績保存の設計と実装。ただしプレイ画面への接続は明示許可後
- API、ランキング、マッチング、WebSocket、Cloudflare Workersなどのバックエンド
- テスト、CI、PWA、デプロイ設定。ただしテスト目的でもプレイ画面を書き換えない
- Claudeや他AIが参照する仕様書・共有資料の整理

## 3. Vite / TypeScript移行の扱い

`docs/codex-implementation-spec.md` の14章・18章には、単一HTMLをTypeScript + Viteへ移行し、`src/screens/game.ts` などへ分割する案が残っている。
これは現在の最新方針では **将来候補** として扱い、現時点の実装指示ではない。

現時点では以下を禁止する。

- `prototype/moji-merge-tsunahiki-bold-concept.html` の置換
- Claude版プレイ画面の描画・ルール・入力処理の再実装
- `public/index.html` やVite画面でプレイ画面を別物に差し替えること
- ゲーム本体を `src/screens/game.ts` に抽出して正本化すること

Vite / TypeScriptを使う場合でも、対象は非プレイ画面、バックエンド連携、データモデル、テスト可能な純粋関数、または将来検証用の別ブランチに限定する。

## 4. 仕様書の読み方

優先順は以下とする。

1. ユーザーの最新指示
2. この補遺 `docs/source-of-truth-amendment-2026-06-27.md`
3. `docs/current-gameplay-spec.md` のプレイ画面保護方針
4. `docs/codex-implementation-spec.md` の非プレイ画面・バックエンド・保存・PWA・オンライン・テストに関する詳細
5. その他の制作メモ

`docs/codex-implementation-spec.md` は詳細なバックログとして有効だが、プレイ画面保護と矛盾する箇所はこの補遺で上書きする。

## 5. 次に実装可能なこと

プレイ画面を触らずに進められる優先タスクは以下。

- ホーム画面からClaude版プレイ画面への導線強化
- 設定・ヘルプ・統計・実績の非プレイ画面の完成度向上
- LocalStorageの保存スキーマ作成
- 結果データ連携の方式検討。ただしプレイ画面への埋め込みは許可後
- Cloudflare Workers + D1 + Durable Objectsを前提にしたランキング・オンライン対戦APIの実装準備
- GitHub / Notion / Claude向け共有資料の同期
