# モジマージ 現在仕様 - 綱引き対戦プロトタイプ

更新日: 2026-06-27

## 重要: 参照優先度

実装時の優先参照順は以下とする。

1. ユーザーの最新指示
2. [`docs/source-of-truth-amendment-2026-06-27.md`](source-of-truth-amendment-2026-06-27.md)
3. この `current-gameplay-spec.md` のプレイ画面保護方針
4. [`docs/codex-implementation-spec.md`](codex-implementation-spec.md) の非プレイ画面・バックエンド・保存・PWA・オンライン・テストに関する詳細
5. [`docs/implementation-feasibility-review.md`](implementation-feasibility-review.md)

`docs/codex-implementation-spec.md` は詳細バックログとして有効。ただし、プレイ画面の保護方針と矛盾する箇所は `docs/source-of-truth-amendment-2026-06-27.md` を優先する。

## 重要: プレイ画面の保護方針

- Claude作成のプレイ画面は `prototype/moji-merge-tsunahiki-bold-concept.html` とする。
- Codexは、ユーザーの明示許可なくプレイ画面の見た目・ゲーム仕様・描画・ルールを変更しない。
- Home、難易度選択、Settings、Help、Stats、Achievementsなどの非プレイ画面は `prototype/moji-merge-app.html` で作成・更新できる。
- バックエンド、保存、ランキング、オンライン同期、API、テストはCodexの担当範囲とする。
- Vite / TypeScript移行は将来候補であり、現時点ではClaude版プレイ画面の置換・再実装には使わない。

## 現在の方向性

モジマージは、現時点では「綱引き対戦タイピングゲーム」としてプロトタイプを進める。
スイカゲーム風の世界観は採用せず、太い黒フチ、手描き風キャラクター、YouTubeサムネイルのような強い視認性を持つカジュアル対戦画面を目指す。

## コアルール

- プレイヤーとCPUがロープを引き合う。
- 単語をローマ字入力で打ち切ると、自陣の人数が増える。
- 綱引きの強さは人数に比例する。
- 人数差に応じてロープが左右へ動く。
- ロープ中央の印を自分側へ引き込めば勝ち、相手側へ引き込まれると負け。
- 制限時間は45秒。時間切れ時はロープ位置で勝敗判定する。

## 単語と難易度

- 表示語は漢字混じりの意味ある単語・短文のみ。
- 500件の単語候補を生成する。
- 難易度はL1からL4。
- 入力成功時の追加人数は難易度に比例する。
  - L1: +1人
  - L2: +2人
  - L3: +3人
  - L4: +4人
- ローマ字入力は揺れを許容する。
  - tuchi / tuti / tsuchi を同一扱い
  - shi / si, chi / ti, tsu / tu, fu / hu, ji / zi も許容

## 対戦要素

- CPUにも現在入力中の単語がある。
- CPU側カードには漢字表示、ローマ字進捗、進捗バーを表示する。
- CPUが単語を打ち終えると、CPU側人数が難易度分増える。
- CPU速度は高速寄りに調整済み。
  - 初期速度: 0.00040
  - 単語更新後: 0.00034 + random 0.00016 + lv * 0.000035
- 放置するとCPUが人数を増やし、ロープを引き込む。

## コンボと逆転要素

- プレイヤーが連続成功するとCOMBOが増える。
- 入力ミスでCOMBOは0に戻る。
- 5 COMBOごとに「応援ラッシュ」が発動する。
- 応援ラッシュ中は人数追加が2倍になる。
- ロープが不利側に大きく寄るとステージに DANGER! を表示する。

## ビジュアル仕様

- ロープは単なる線ではなく、黒フチ付きの太い編み込みロープとして描画する。
- ロープには斜めの編み目、ハイライト、左右に垂れ下がる余り紐を入れる。
- 中央には赤い帯状の印を付ける。
- キャラクターは白い棒人間風、太い黒フチ、表情付き。
- 背景はくすんだ紫系、床は木目風。
- UIは太フチ、強い影、漢字が大きく読めるカード構成。

## 現在のプロトタイプ

- アプリ入口: `prototype/moji-merge-app.html`
- Claude版プレイ画面: `prototype/moji-merge-tsunahiki-bold-concept.html`
- ローカルプレイ画面: `outputs/moji-merge-tsunahiki-bold-concept.html`

注意: GitHub上のprototypeが最新です。ローカルHTMLは同期が必要な場合があります。

## Claudeに依頼したいこと

1. まず `docs/source-of-truth-amendment-2026-06-27.md` を確認する。
2. 次に `prototype/moji-merge-tsunahiki-bold-concept.html` を確認する。
3. 必要に応じて `prototype/moji-merge-app.html` で外側画面との接続方針を確認する。
4. `docs/codex-implementation-spec.md` と `docs/implementation-feasibility-review.md` で実装順とリスクを確認する。
5. プレイ画面の世界観を壊さずに魅力を上げる。
6. ロープ、キャラクター、UIカード、エフェクトをよりゲームらしく磨く。
7. 「CPUもタイピングしている」「負けそう」「応援ラッシュ中」が一目で分かる演出を強める。
8. スマホとPCの両方で破綻しないレイアウトにする。
