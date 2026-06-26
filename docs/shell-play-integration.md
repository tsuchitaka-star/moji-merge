# Shell ⇔ Play 連携データ契約

更新日: 2026-06-26
対象: Codex（シェル担当）／ Claude（プレイ画面担当）

## 概要

- **Shell**: `prototype/moji-merge-app.html`（Codex 担当）  
  Home / 難易度選択 / Settings / Stats / Achievements / Daily / Online
- **Play**: `prototype/moji-merge-tsunahiki-bold-concept.html`（Claude 担当）  
  実際の試合画面

両者は **LocalStorage キー `moji-merge-shell-save-v1`** を介して連携する。

## データ形式（Source of Truth）

```ts
interface ShellSave {
  selectedDifficulty: 'easy' | 'normal' | 'hard' | 'expert';
  settings: {
    fontSize: string;     // '0.9' | '1' | '1.2' | '1.5'
    shake: string;        // '0' | '0.5' | '1'
    bgm: string;          // '0' ~ '1'
    se: string;           // '0' ~ '1'
    romaji: 'hepburn' | 'kunrei' | 'none';
    contrast: 'true' | 'false';
  };
  stats: {
    totalMatches: number;
    wins: number;
    bestScore: number;
    bestCombo: number;
  };
  achievements: string[];  // IDのリスト
  lastScore?: number;      // 直近の試合スコア
  lastDifficulty?: string;
  lastResult?: 'win' | 'lose' | 'draw';
  lastPlayedAt?: string;   // ISO 8601
}
```

## 設定の効き方（Shell → Play）

| 設定 | Play画面の反映 |
|---|---|
| `selectedDifficulty` | CPU速度・試合時間・DANGER閾値 を切替 |
| `settings.shake` | シェイク強度の倍率（0でシェイク無効） |
| `settings.romaji = 'hepburn'` | ヒント表示は Hepburn 優先（デフォルト） |
| `settings.romaji = 'kunrei'` | Hint: 訓令式を優先（実装余地あり、現状デフォルト） |
| `settings.romaji = 'none'` | ローマ字ヒントを完全に非表示 |
| `settings.contrast = 'true'` | 高コントラストCSSを適用（黒背景・黄縁・赤強調） |
| `settings.fontSize` | 現状は読み込みのみ、適用は今後 |
| `settings.bgm/se` | 現状は読み込みのみ、音響実装後に適用 |

## 難易度プリセット

```ts
const DIFFICULTY_PRESETS = {
  easy:   { cpuSpeedInit: 0.00028, timeLimit: 60000, dangerThreshold: -0.75 },
  normal: { cpuSpeedInit: 0.00040, timeLimit: 45000, dangerThreshold: -0.62 },
  hard:   { cpuSpeedInit: 0.00055, timeLimit: 30000, dangerThreshold: -0.50 },
  expert: { cpuSpeedInit: 0.00070, timeLimit: 30000, dangerThreshold: -0.40 },
};
```

（フル定義はプレイ画面の JS を参照）

## スコア計算（Play → Shell）

```
score =
  (L1ヒット数 × 100) +
  (L2ヒット数 × 250) +
  (L3ヒット数 × 500) +
  (L4ヒット数 × 1000) +
  (最高コンボ × 50) +
  (勝利時 +3000 / 引分時 +1000) ;
score = round(score × (0.6 + 0.4 × 命中率)) ;
```

命中率 = `correctKeys / totalKeys`、ミスタイプは `totalKeys` も増やす。

## 試合終了時の保存処理（Play → Shell）

`triggerGameOver(result)` → `commitMatchResult()` で以下を更新：

- `stats.totalMatches` += 1
- `stats.wins` += 1（勝利時のみ）
- `stats.bestScore` = max(現在, 計算スコア)
- `stats.bestCombo` = max(現在, 最高コンボ)
- `achievements` に該当する ID を追加
- `lastScore`, `lastDifficulty`, `lastResult`, `lastPlayedAt` を書き込み

## 実績ID（共通定義）

| ID | 達成条件 |
|---|---|
| `first_win` | 初めて勝つ |
| `combo_10` | 1試合でコンボ10達成 |
| `combo_20` | 1試合でコンボ20達成 |
| `perfect` | ミス0で勝利 |
| `comeback` | 試合中に rope ≤ -0.5 になってから勝利 |
| `lv4_killer` | 1試合で L4 単語を3つ撃破 |

これらは Shell の `ACHIEVEMENTS` 配列とも一致している。**新規実績を追加する際は両方を同時に更新する。**

## 画面間ナビゲーション

- Shell → Play: `<a href="moji-merge-tsunahiki-bold-concept.html">`（直接遷移）
- Play → Shell: 終了オーバーレイの「ホームへ」ボタン → `location.href = 'moji-merge-app.html#/'`

Play 画面は単体で開いても動く（shell save が無ければ DEFAULT_SAVE 相当で normal 難易度）。

## 役割分担の原則

- **Shell**（Codex）: 起動入口、画面ルーティング、設定 UI、統計表示、実績一覧表示、データ書き込みの読み取り側
- **Play**（Claude）: 試合中の画面、ロープ・棒人間・タイピング・スコア計算、データ書き込み側
- 両者で**同じキー・同じ形式**を使う約束。スキーマ変更時は両者を同時に更新する。

## バージョニング

LocalStorage キーに `-v1` を入れている。スキーマを破壊的に変更する場合：

1. `moji-merge-shell-save-v2` に切替
2. 旧データからの migration ロジックを両側に追加

## 既知の TODO

- Settings.fontSize の適用（現状読み込みのみ）
- Settings.bgm / se の適用（音響実装後）
- Settings.romaji = 'kunrei' の Hint 優先表示
- ポーズ機能との連携
- 試合履歴（history[]）の保存（codex-implementation-spec.md 4.1 参照）
