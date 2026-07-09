# 機械キャラクタの計算論的設計 — ブラウザ実装
**Computational Design of Mechanical Characters — Interactive Browser Implementation**

Coros らによる SIGGRAPH 2013 論文 *Computational Design of Mechanical Characters* の対話的設計パイプラインを、単一の HTML ファイルとして実装した非公式のブラウザデモです。目標とする閉曲線(モーションカーブ)をスケッチすると、その軌道を最もよく再現するリンク機構をデータベースから検索し、寸法パラメータを連続最適化して、単一のクランクで駆動するアニメーションとして再生します。

An unofficial, single-file browser implementation of the interactive design pipeline from *Computational Design of Mechanical Characters* (Coros et al., SIGGRAPH 2013). Sketch a closed target curve, retrieve the best-matching linkage mechanism from a precomputed database, refine its parameters via continuous optimization, and watch the mechanism trace the curve driven by a single crank.

<!-- スクリーンショット / デモGIFをここに配置 -->
<!-- ![demo](docs/demo.gif) -->

**[▶ Live Demo](https://n-nagaya.github.io/mechanical-characters/mechanical_characters.html)**

## デモ / Demo

`mechanical_characters.html` をブラウザで開くだけで動作します。外部ライブラリ・ビルド・サーバは不要です。

Just open `mechanical_characters.html` in a browser. No external libraries, build steps, or server required.

- GitHub Pages を有効にすると、そのままオンラインデモとして公開できます。
- 動作確認: Chrome / Edge / Safari / Firefox の最新版(デスクトップ・タブレット)

## 使い方 / Usage

1. **目標曲線を描く** — 図面キャンバス上にドラッグで閉曲線を描きます。プリセット(楕円・豆型・8の字・おむすび・歩行軌道)も選べます。
2. **機構を検索** — 事前サンプリング済みデータベースから、曲線特徴量メトリックにより上位3候補の機構を提示します。カードをクリックして候補を切り替えられます。
3. **パラメータ最適化** — 選択した機構の寸法・取付点・配置を最適化し、マーカー軌跡と目標曲線の二乗誤差を最小化します。
4. **駆動・確認** — 単一の入力クランクで機構を駆動し、軌跡と目標曲線を重ねて確認します。速度調整と表示切替(目標曲線 / 軌跡 / 歯車)が可能です。

1. **Sketch** a closed target curve on the drafting canvas (or pick a preset).
2. **Search** the precomputed mechanism database; the top-3 candidates are shown as clickable cards.
3. **Optimize** the selected mechanism's dimensions, attachment point, and placement to minimize the squared error between the traced curve and the target.
4. **Play** the mechanism driven by a single input crank, with speed control and display toggles.

## 実装内容 / What is implemented

論文の対話的パイプラインの中核部分(論文 Fig. 2 の a–c 相当)を実装しています。

| パイプライン段階 | 論文 | 本実装 |
|---|---|---|
| 機構ライブラリ | パラメータ化された駆動機構群(§4, Fig. 4) | 3種:4節リンク、歯車駆動5節リンク(歯車比 −2, −1, 1, 2 の4変種)、クランク・スライダ |
| 機構シミュレーション | 制約ベースシミュレーション+Newton–Raphson(§3) | 閉形式の順運動学(円交点計算+分岐の連続性チェック) |
| パラメータ空間探索 | メトリック空間での Poisson-disk サンプリング(§4.1) | 特徴量空間での Poisson-disk 型棄却サンプリング(起動時に約780曲線を構築) |
| 曲線メトリック | 特徴量ベース+係数の対話学習(§5) | 特徴量(長さ・面積・楕円率・自己交差数)+頂点距離。係数は固定値 |
| 連続最適化 | 陰関数定理による勾配+BFGS(§4.2, 式(2)–(4)) | Nelder–Mead 法(機構パラメータ+相似変換を同時最適化) |
| 駆動 | 単一入力ドライバ(§6.1) | 単一クランク駆動のアニメーション |

**対象外(論文にあるが未実装)/ Not implemented:**
非円形歯車によるタイミング制御(§4.3)、歯車列の自動生成(§6.1)、衝突回避のレイヤ配置(§6.2)、支持構造生成(§6.3)、3Dプリント用出力(§6.4)、メトリック係数の対話学習(§5.2)。

Timing control via non-circular gears (§4.3), automatic gear-train generation (§6.1), collision-free layering (§6.2), support-structure generation (§6.3), fabrication export (§6.4), and interactive metric training (§5.2) are out of scope.

## 技術メモ / Technical notes

- 依存ライブラリなしの単一 HTML(Canvas 2D + Vanilla JS、約1,100行)
- 曲線は等弧長で 72 点にリサンプリングして比較。位相(タイミング)情報は論文同様に破棄し、巡回オフセットと向きを総当たりで対応付け
- 検索は特徴量距離で上位40件に絞ってから頂点距離で再ランキング
- 最適化は最大450反復の Nelder–Mead をフレーム分割実行(UI を停止させない)。プリセット曲線に対して RMS 誤差 1〜16 px(目標曲線 約360 px 規模)、実行時間 0.1 秒未満
- 鏡映対称の曲線に対応するため、初期化時に回転 {0, π} × 鏡映 {あり, なし} の4通りを評価して最良の初期値を選択

## 引用 / Citation

本実装は下記論文の非公式実装です。原著は Disney Research Zurich / Disney Research Boston / MIT CSAIL によるものです。研究・教育目的でお使いの際は原論文を引用してください。

This is an **unofficial** implementation. The original work is by Disney Research Zurich / Disney Research Boston / MIT CSAIL. Please cite the original paper:

> S. Coros, B. Thomaszewski, G. Noris, S. Sueda, M. Forberg, R. W. Sumner, W. Matusik, and B. Bickel, "Computational design of mechanical characters," *ACM Transactions on Graphics*, vol. 32, no. 4, Article 83, pp. 1–12, Jul. 2013. doi: [10.1145/2461912.2461953](https://doi.org/10.1145/2461912.2461953)

```bibtex
@article{coros2013mechanical,
  author  = {Coros, Stelian and Thomaszewski, Bernhard and Noris, Gioacchino and Sueda, Shinjiro and Forberg, Moira and Sumner, Robert W. and Matusik, Wojciech and Bickel, Bernd},
  title   = {Computational Design of Mechanical Characters},
  journal = {ACM Transactions on Graphics},
  volume  = {32},
  number  = {4},
  pages   = {83:1--83:12},
  year    = {2013},
  doi     = {10.1145/2461912.2461953}
}
```

## ライセンス / License

本リポジトリのコードは MIT License で公開しています。論文の手法・図・本文の著作権は原著者および ACM に帰属します。

The code in this repository is released under the MIT License. The method, figures, and text of the original paper remain the property of the original authors and ACM.
