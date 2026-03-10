# AGENTS.md — Soccer Kick Game

## プロジェクト概要

サッカーフィールドを舞台にした、ブラウザで動く**ボール蹴り返しゲーム**。  
単一の HTML ファイル（`soccer-game.html`）で完結している。フレームワーク・ビルドツール不要。

---

## ファイル構成

```
soccer-game.html   ← ゲーム本体（HTML / CSS / JS すべてここ）
AGENTS.md          ← このファイル
```

---

## ゲーム仕様

### 基本ルール
- 画面上部からボールが斜め方向に複数転がってくる
- 画面下部のプレイヤー（背番号10）を**左右カーソルキー**で動かしてボールに当てる
- ボールを蹴り返す（当てる）と **+1スコア** ＋パーティクルエフェクト
- ボールを **5個** 見逃すとゲームオーバー
- 400フレームごとにレベルアップ → `spawnInterval` が 6 減少 → ボール出現が速くなる

### 操作
| キー | 動作 |
|------|------|
| `←` (ArrowLeft) | プレイヤーを左移動 |
| `→` (ArrowRight) | プレイヤーを右移動 |

---

## コードアーキテクチャ

```
HTML
 └─ <canvas id="c">  600×520px  ← 描画はすべてここ
 └─ <div id="ui">               ← スコア/レベル/ミス表示（DOM）
 └─ <div id="overlay">          ← タイトル・ゲームオーバー画面

JavaScript（グローバル変数・関数ベース）
 ├─ State
 │   player      { x, y, w, h, speed }
 │   balls[]     { x, y, vx, vy, r, rot, kicked, kickVx, kickVy, kickAlpha, trail[] }
 │   particles[] { x, y, vx, vy, life, color }
 │   score / miss / level / frameCount / spawnInterval / running
 │
 ├─ startGame()      オーバーレイ非表示 → initGame() → loop()
 ├─ initGame()       全 State を初期値にリセット
 ├─ loop()           requestAnimationFrame ループ
 │   ├─ 入力処理（keys オブジェクト）
 │   ├─ spawnBall()  フレームカウントで定期生成
 │   ├─ ボール更新（衝突判定 / ミス判定 / kick 後フライ）
 │   ├─ particles 更新
 │   └─ draw()
 ├─ draw()
 │   ├─ drawField()  芝・ライン・ゴール描画
 │   ├─ trail 描画
 │   ├─ drawBall()   ボール本体（グラデーション＋五角形パッチ）
 │   ├─ particles 描画
 │   └─ drawPlayer() プレイヤーキャラクター
 ├─ burst(x, y)      キック時のパーティクル生成
 ├─ updateHUD()      DOM の数値更新
 └─ gameOver()       overlay を書き換えてゲームオーバー表示
```

---

## 定数・チューニングパラメータ

| 変数 / 箇所 | 初期値 | 説明 |
|---|---|---|
| `canvas` width / height | 600 / 520 | フィールドサイズ |
| `player.speed` | `6` | プレイヤー移動速度（px/frame）|
| `spawnInterval` | `90` | ボール出現間隔（フレーム数）|
| `spawnInterval` 最小値 | `38` | `> 38` の条件でクランプ |
| レベルアップ間隔 | `400` frames | `frameCount % 400 === 0` |
| `spawnInterval` 減少量 | `6` | レベルアップごとに減算 |
| `speed`（ボール） | `2.2 + level*0.3 + rand*1.2` | レベルに比例して速くなる |
| miss 上限 | `5` | これを超えるとゲームオーバー |
| パーティクル数 | `14` | `burst()` 内の for ループ上限 |
| trail 長さ | `8` | `b.trail.length > 8` でシフト |

---

## デザイントークン（CSS 変数の代わりに使われている色）

| 役割 | 値 |
|---|---|
| 背景 | `#0a1a0a` |
| フィールド緑 | `#1a4a1a` |
| ライン | `rgba(255,255,255,0.18)` |
| HUD テキスト | `#e8f5e9` |
| HUD 数値 | `#76ff03` |
| タイトルグロー | `#76ff03` / `#4caf50` |
| ゲームオーバー | `#ff5252` |
| ジャージ | `#e53935`（赤）|
| ショーツ | `#1a237e`（紺）|

フォント: **Oswald**（見出し・数値）/ **Noto Sans JP**（本文）  
Google Fonts から CDN 読み込み。

---

## 今後の開発アイデア（優先度順）

### すぐできる改善
- [ ] **効果音** — Web Audio API で `AudioContext` を使いキック音・ミス音・レベルアップ音を追加
- [ ] **ハイスコア保存** — `localStorage` に `bestScore` を保存して表示
- [ ] **タッチ操作** — `touchstart/touchmove` でスマホ対応（ボタンまたはスワイプ）
- [ ] **ボールの種類** — 速いボール・大きいボール・爆発ボールなどをランダムに混ぜる

### 中規模の機能追加
- [ ] **コンボシステム** — 連続キック数を記録して倍率ボーナス
- [ ] **パワーゲージ** — 長押しで強いキックができる
- [ ] **対戦モード（2P）** — `A/D` キーで 2 人目のプレイヤーを追加
- [ ] **アニメーション強化** — キャラクターにスプライトシートを使う

### 大規模リファクタリング
- [ ] **ES Module 化** — `player.js` / `ball.js` / `renderer.js` などに分割
- [ ] **TypeScript 移行** — 型安全にする
- [ ] **Canvas → WebGL** — PixiJS などを使いパフォーマンス向上

---

## 開発環境のセットアップ

ビルドツール不要。ブラウザで直接開くだけで動く。

```bash
# Live Server（VSCode 拡張）を使う場合
# 拡張: "Live Server" (ritwickdey.liveserver) をインストール
# soccer-game.html を右クリック → "Open with Live Server"

# または Python でローカルサーバーを立てる
python3 -m http.server 8080
# → http://localhost:8080/soccer-game.html
```

### 推奨 VSCode 拡張
| 拡張名 | 用途 |
|---|---|
| Live Server | ホットリロード付きローカルサーバー |
| ESLint | JS の静的解析 |
| Prettier | コードフォーマット |
| HTML CSS Support | CSS クラス補完 |

---

## デバッグ Tips

```javascript
// ブラウザコンソールで即座にテスト可能な変数・関数

// ゲーム状態を確認
console.log({ score, miss, level, balls: balls.length });

// ボールを手動で追加
spawnBall();

// ゲームを強制終了
running = false;

// スポーン間隔を手動で変更（難易度テスト）
spawnInterval = 20;  // 超高速
spawnInterval = 200; // 超低速
```

---

## エージェントへの指示原則

このプロジェクトを AI エージェント（Copilot / Cursor / Claude 等）で開発する際の指針：

1. **単一ファイル原則を守る** — 特別な理由がない限り `soccer-game.html` 1 ファイルに収める
2. **グローバル State を尊重** — `player` / `balls` / `particles` の構造を変える場合はすべての参照箇所を更新する
3. **`requestAnimationFrame` ループを壊さない** — `loop()` の中で `async/await` を使わない
4. **Canvas 座標系を意識** — 原点は左上、Y 軸は下向き
5. **Canvas API の互換性** — `roundRect` は Chrome 99+ / Firefox 112+ 以降のみ対応。古いブラウザをターゲットにする場合は `rect` に置換
6. **パフォーマンス** — `draw()` 内で `new` や大量のオブジェクト生成をしない（GC 負荷）
