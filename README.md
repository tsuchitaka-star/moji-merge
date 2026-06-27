# モジマージ

モジマージは、言葉入力の速さと判断を使って遊ぶカジュアルタイピングゲームです。

現在は、まず遊びとして分かりやすい「綱引き対戦タイピング」プロトタイプを進めています。

## 現在のプロトタイプ

### モジマージ 綱引き

- プレイヤーとCPUがロープを引き合う対戦タイピングゲーム
- 単語を打ち切ると味方人数が増える
- 綱引きの強さは人数に比例する
- 難しい言葉ほど追加人数が多い
- CPUにも入力中の単語と進捗バーがある
- 5コンボごとに「応援ラッシュ」が発動し、人数追加が2倍になる
- ロープは線ではなく、編み込みと余り紐のある太い紐として描画する

## 画面構成

- アプリ入口 / 非プレイ画面: [`prototype/moji-merge-app.html`](prototype/moji-merge-app.html)
- Claude作成のプレイ画面: [`prototype/moji-merge-tsunahiki-bold-concept.html`](prototype/moji-merge-tsunahiki-bold-concept.html)

重要: プレイ画面の見た目・ゲーム仕様はClaude作成版を維持します。Codex側はホーム、設定、統計、実績、保存、API、ランキング、オンライン同期など、プレイ画面外または裏側の処理を担当します。

## 最新仕様

実装時の優先参照順は以下です。

1. 最新の上書き補遺: [`docs/source-of-truth-amendment-2026-06-27.md`](docs/source-of-truth-amendment-2026-06-27.md)
2. 実装仕様書 / 詳細バックログ: [`docs/codex-implementation-spec.md`](docs/codex-implementation-spec.md)
3. 実装可能性レビュー: [`docs/implementation-feasibility-review.md`](docs/implementation-feasibility-review.md)
4. 現在仕様メモ: [`docs/current-gameplay-spec.md`](docs/current-gameplay-spec.md)
5. グラフィック制作仕様: [`docs/graphic-spec-for-claude.md`](docs/graphic-spec-for-claude.md)
6. オンライン環境仕様: [`docs/online-environment.md`](docs/online-environment.md)

`docs/codex-implementation-spec.md` は、単一HTMLプロトタイプを公開可能なWeb/PWAゲームへ広げるための詳細バックログです。ただし、プレイ画面の保護方針と矛盾する箇所は `docs/source-of-truth-amendment-2026-06-27.md` を優先します。特に、Claude作成のプレイ画面をCodexが勝手に置換・再実装することは禁止です。

## Notion

進捗管理ページ:
https://app.notion.com/p/38a245c08105810ea3b1c94730567aac

オンライン環境仕様:
https://app.notion.com/p/38a245c0810581498e33d2c6a0100309

## ディレクトリ予定

```text
assets/       画像・音などの素材
docs/         仕様書・制作メモ
prototype/    ClaudeやAIに渡す検証用HTML
src/          本実装
```

## Claudeに参照してほしいもの

1. `docs/source-of-truth-amendment-2026-06-27.md`
2. `prototype/moji-merge-tsunahiki-bold-concept.html`
3. `prototype/moji-merge-app.html`
4. `docs/codex-implementation-spec.md`
5. `docs/implementation-feasibility-review.md`
6. `docs/current-gameplay-spec.md`
7. `docs/graphic-spec-for-claude.md`

まずはClaude版プレイ画面の魅力を保ちながら、ホーム・設定・保存・統計などの外側を整えます。
