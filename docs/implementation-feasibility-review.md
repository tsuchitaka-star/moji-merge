# モジマージ綱引き 実装可能性レビュー

更新日: 2026-06-26
対象仕様: `docs/codex-implementation-spec.md`

## 結論

実装は可能です。ただし、仕様書の全内容を一気に作ると範囲が大きすぎます。

現実的には、まず Phase 1 の「個人プレイ完成版」を完成させ、その後に統計・ランキング・オンライン対戦へ進める構成が安全です。

## 実装難易度の見立て

| 領域 | 実装可能性 | 難易度 | 判断 |
|---|---:|---:|---|
| 既存HTMLプロトタイプの維持改善 | 高 | 低 | すぐ可能 |
| TypeScript + Vite 移行 | 高 | 中 | 早めに着手すべき |
| Home / Result / Settings / Help | 高 | 中 | Phase 1で実装可能 |
| LocalStorage保存 | 高 | 低〜中 | Phase 1必須 |
| PWA / モバイルUX | 高 | 中 | スマホ対応のため必須 |
| 音響 | 高 | 中 | アセット準備が別途必要 |
| 統計 / 実績 / Daily Challenge | 高 | 中 | Phase 2向き |
| グローバルランキング | 中〜高 | 中〜高 | Cloudflare D1前提で可能 |
| オンライン対戦 | 中 | 高 | Durable Objects/WebSocket設計が必要 |
| ランクマッチ / リプレイ / A/B | 中 | 高 | Phase 4でよい |

## Phase 1で実装すべき範囲

最初の公開版では、次に絞るのが現実的です。

- Vite + TypeScript へ移行
- `romaji.ts`, `words.ts`, `state.ts`, `render.ts`, `ai.ts` へ分割
- Home画面
- 難易度選択
- In-gameのポーズとmetrics収集
- Result画面詳細化
- Settings画面
- How to Play
- LocalStorage保存
- スマホ縦画面レイアウト改善
- PWA基本対応
- 最低限の単体テスト

## 先に直すべき技術リスク

### 1. 単一HTMLのままだと拡張が危険

現在のプロトタイプは見た目検証には良いですが、画面遷移・保存・設定・テスト・PWAを入れるには肥大化します。

対応:
- まず Vite + TypeScript に移行する
- 既存HTMLは `prototype/` に残す
- ゲームロジックと描画を分ける

### 2. ローマ字判定は最初にテスト化する

`tuchi / tuti / tsuchi` のような揺れ許容はゲーム体験の核です。ここが壊れると入力ストレスが強くなります。

対応:
- `romaji.ts` を最初に抽出
- Vitestで `tokenizeRoma`, `checkInput`, `buildHint` を固定する

### 3. モバイル入力は早期検証が必要

スマホではソフトキーボードが画面を隠します。PC版の見た目だけで進めると後で作り直しになります。

対応:
- Phase 1からモバイル縦画面レイアウトを実装
- 入力欄は16px以上
- キャンバス、単語カード、入力欄の縦分割を先に決める

### 4. オンライン対戦は後回しが妥当

リアルタイム対戦は、同期・切断・チート・遅延・部屋管理が絡みます。ゲーム本体が固まる前に入れると作り直しが増えます。

対応:
- Phase 1/2ではCPU戦を完成させる
- Phase 3でCloudflare Workers + Durable Objects + WebSocketに進む

## 仕様上の調整提案

### P0とP1の整理

`難易度選択` と `Pause Menu` は仕様書上 P1 ですが、公開版では実質P0に近いです。

理由:
- 難易度がないと初心者と上級者の両方に合いにくい
- ポーズがないとスマホ/PCどちらでも中断に弱い

### 音響は仮音でよい

BGM/SEは体験向上に大きいですが、アセット制作待ちで実装を止める必要はありません。

対応:
- まずWeb Audio APIの簡易SEで実装
- 後からOGG/MP3アセットに差し替える

### 体感βは本当に必要

綱引きの面白さは数値調整に依存します。

確認すべき数値:
- CPU速度
- 人数差によるrope移動量
- 単語難易度ごとの人数追加
- 応援ラッシュ倍率
- DANGERしきい値

## 推奨実装順

1. Vite + TypeScript の土台作成
2. ローマ字判定と単語辞書を抽出してテスト化
3. 既存Canvas描画を `render.ts` に移植
4. CPU戦を `game.ts` と `state.ts` で再現
5. Home / 難易度選択 / Result を追加
6. LocalStorage保存と統計値の最低限対応
7. Settings / How to Play / Pause を追加
8. スマホ縦画面とPWA対応
9. 音響、アクセシビリティ、細部演出を追加
10. Phase 2以降で統計、実績、Daily、Leaderboard、Onlineへ進む

## 実装可否まとめ

Phase 1は十分実装可能です。仕様の粒度もCodex/Claudeが分担しやすい状態です。

一方で、Phase 2以降のランキング・オンライン対戦は、ゲーム本体の完成後に切り出すべきです。特にオンライン対戦は無料枠でも構築できますが、Durable ObjectsとWebSocketの設計が必要なため、後回しが妥当です。

## 次の実装単位

最初のPRまたは作業単位は以下がよいです。

- `src/game/romaji.ts`
- `src/game/words.ts`
- `tests/romaji.test.ts`
- `tests/words.test.ts`
- Vite + TypeScript の最小起動

この単位なら、既存プロトタイプの見た目を壊さずに、本実装への土台を作れます。
