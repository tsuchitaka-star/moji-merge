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

## 最新仕様

- 現在仕様: [`docs/current-gameplay-spec.md`](docs/current-gameplay-spec.md)
- グラフィック制作仕様: [`docs/graphic-spec-for-claude.md`](docs/graphic-spec-for-claude.md)
- オンライン環境仕様: [`docs/online-environment.md`](docs/online-environment.md)

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

1. `docs/current-gameplay-spec.md`
2. `prototype/moji-merge-tsunahiki-bold-concept.html`
3. `docs/graphic-spec-for-claude.md`

まずは現在の綱引き対戦プロトタイプの魅力を上げることを優先します。
