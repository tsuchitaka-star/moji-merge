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

1. 実装仕様書 / Source of Truth: [`docs/codex-implementation-spec.md`](docs/codex-implementation-spec.md)
2. 実装可能性レビュー: [`docs/implementation-feasibility-review.md`](docs/implementation-feasibility-review.md)
3. 現在仕様メモ: [`docs/current-gameplay-spec.md`](docs/current-gameplay-spec.md)
4. グラフィック制作仕様: [`docs/graphic-spec-for-claude.md`](docs/graphic-spec-for-claude.md)
5. オンライン環境仕様: [`docs/online-environment.md`](docs/online-environment.md)

`docs/codex-implementation-spec.md` は、単一HTMLプロトタイプを公開可能なWeb/PWAゲームへ移行するための本実装仕様です。`current-gameplay-spec.md` と差分が出た場合は、`codex-implementation-spec.md` を優先します。ただし、プレイ画面のデザイン・挙動はClaude作成版を勝手に変更しません。

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

1. `prototype/moji-merge-tsunahiki-bold-concept.html`
2. `prototype/moji-merge-app.html`
3. `docs/codex-implementation-spec.md`
4. `docs/implementation-feasibility-review.md`
5. `docs/current-gameplay-spec.md`
6. `docs/graphic-spec-for-claude.md`

まずはClaude版プレイ画面の魅力を保ちながら、ホーム・設定・保存・統計などの外側を整えます。
