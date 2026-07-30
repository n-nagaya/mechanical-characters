# 機械キャラクタの計算論的設計 — ブラウザ実装
**Computational Design of Mechanical Characters — Interactive Browser Implementation**

**[▶ Live Demo / デモを開く](https://n-nagaya.github.io/mechanical-characters/)** — インストール不要、ブラウザでそのまま動きます / Runs directly in your browser, nothing to install.

Coros らによる SIGGRAPH 2013 論文 *Computational Design of Mechanical Characters* の対話的設計パイプラインを、単一の HTML ファイルとして実装した非公式のブラウザデモです。目標とする閉曲線(モーションカーブ)をスケッチすると、その軌道を最もよく再現するリンク機構をデータベースから検索し、寸法パラメータを連続最適化して、単一のクランクで駆動するアニメーションとして再生します。

An unofficial, single-file browser implementation of the interactive design pipeline from *Computational Design of Mechanical Characters* (Coros et al., SIGGRAPH 2013). Sketch a closed target curve, retrieve the best-matching linkage mechanism from a precomputed database, refine its parameters via continuous optimization, and watch the mechanism trace the curve driven by a single crank.

> **原著を知るには / Start with the original:** Disney Research のプロジェクトページに論文 PDF と解説動画があります。**動画（約5分）を先に見るのが手法の全体像をつかむ一番の近道です。**
> [Disney Research — Computational Design of Mechanical Characters](https://studios.disneyresearch.com/2013/07/21/computational-design-of-mechanical-characters/)（[論文 PDF](https://studios.disneyresearch.com/wp-content/uploads/2019/03/Computational-Design-of-Mechanical-Characters-1.pdf) / [解説動画](https://www.youtube.com/watch?v=DfznnKUwywQ)）

<!-- スクリーンショット / デモGIFをここに配置 -->
<!-- ![demo](docs/demo.gif) -->

## 動かす / Running it

**オンライン / Online** — 上の **[▶ Live Demo](https://n-nagaya.github.io/mechanical-characters/)** から開けます。

**ローカル / Local** — リポジトリの `index.html` をダウンロードしてブラウザで開くだけで動作します。外部ライブラリ・ビルド・サーバは不要です。
Download `index.html` and open it in a browser. No external libraries, build steps, or server required.

> **Note:** GitHub のファイル一覧で `index.html` をクリックするとソースコードが表示されるだけです。デモは Live Demo リンクから開いてください。
> Clicking the file on GitHub shows the source code only — use the Live Demo link to run it.

動作確認 / Tested on: Chrome / Edge / Safari / Firefox の最新版(デスクトップ・タブレット)

## 使い方 / Usage

1. **目標曲線を描く / Sketch** — 図面キャンバス上にドラッグで閉曲線を描きます。プリセット(楕円・豆型・8の字・おむすび・歩行軌道)も選べます。
   Draw a closed target curve on the drafting canvas, or pick a preset.
2. **機構を検索 / Search** — 事前サンプリング済みデータベースから、曲線特徴量メトリックにより上位3候補の機構を提示します。カードをクリックして候補を切り替えられます。
   The top-3 candidate mechanisms from the precomputed database are shown as clickable cards.
3. **パラメータ最適化 / Optimize** — 選択した機構の寸法・取付点・配置を最適化し、マーカー軌跡と目標曲線の二乗誤差を最小化します。
   The selected mechanism's dimensions, attachment point, and placement are refined to minimize the squared error between the traced curve and the target.
4. **駆動・確認 / Play** — 単一の入力クランクで機構を駆動し、軌跡と目標曲線を重ねて確認します。速度調整と表示切替(目標曲線 / 軌跡 / 歯車)が可能です。
   The mechanism is driven by a single input crank, with speed control and display toggles.

5. **実寸で出力 / Export** — 寸法をミリメートルに換算し、JSON / CSV / DXF で書き出します。
   Dimensions are converted to millimeters and exported as JSON / CSV / DXF.

### 出力 / Export

**実寸の決め方:** 「機構全体の長辺」を mm で指定します。1周期分の機構全体（歯車のピッチ円を含む）の外接矩形の長辺が、その値になるよう換算されます。最適化された機構は目標曲線の数倍になるため、板材サイズから逆算できるこの基準を採用しています。

**出力座標系:** 機構ローカル系 — 原点は入力クランクの固定ピボット、x 軸は固定節（軸間）方向、**y は上向き**（CAD 標準。画面キャンバスとは y の向きが逆）。図面上の配置（回転・平行移動）は加工に無関係なので含めません。鏡映表示が選ばれている場合は y 座標の符号を反転し、画面の見え方と一致させます。

| 形式 | 内容 |
|---|---|
| JSON | リンク長・固定ピボット座標・位相0°のジョイント座標・組立角・マーカー取付寸法・歯車諸元・軌跡・目標曲線・精度。§5-C の Fusion 360 アドインへの入力を兼ねる |
| 寸法 CSV | 上記を「区分・項目・記号・値・単位・備考」の表にしたもの。表計算での確認用（Excel 用に BOM 付き） |
| 軌跡 CSV | 入力クランク角ごとのマーカー座標と目標曲線座標 |
| DXF | R12 (AC1009)。LINK / PIVOT / JOINT / MARKER / TRACE / TARGET / GEAR のレイヤに分けた 2D 図面。単位は mm |

**歯車について（重要）:** 歯車比が運動学に効くのは**歯車駆動5節リンクのみ**です。この機構については、指定した基準歯数と軸間距離から `d = m·z`、`a = (d1+d2)/2` を満たすようモジュールを逆算し、歯数・基準円・歯先円・歯底円を出力します。軸間距離はリンク寸法で決まっているため、モジュールは一般に非標準値になります（歯形を生成して加工する前提）。あわせて次を自動判定します。

- **回転方向の成立性** — 歯車比が正（同方向回転）の場合、外歯車1組では実現できません。アイドラ（遊び歯車）が必要である旨を警告します
- **アンダーカット** — 歯数が 17 未満の場合に警告します

4節リンク・クランクスライダの画面上のクランク円板は**描画上の装飾**であり、モジュールも歯数も物理的意味を持ちません。したがって歯車諸元を出力しません（論文 §6.1 の歯車列生成は本実装では未対応）。

### 表示操作 / View navigation

最適化された機構は目標曲線より数倍大きくなることが多く(歯車を含めると図面の 2〜6 倍になる場合があります)、既定の縮尺では画面外にはみ出します。図面右上のツールバーで縮尺を操作してください。

Optimized mechanisms are often several times larger than the target curve, so the view can be zoomed and panned:

| 操作 / Action | 方法 / How |
|---|---|
| 拡大・縮小 / Zoom | マウスホイール、トラックパッドのピンチ、ツールバーの `＋` `−`、キー `+` `-`(カーソル位置を中心に拡大) |
| 移動 / Pan | **Space** または **Shift** + ドラッグ、中ボタン・右ボタンのドラッグ、ツールバーの `✎ 描画 / ✥ 表示` を「表示」に切り替えてドラッグ |
| 全体表示 / Fit | ツールバーの `全体表示`、キー `F`(1周期分の全フレームの外接矩形に自動フィット) |
| 等倍に戻す / Reset | ツールバーの `1:1`、キー `0` |

現在の縮尺は図面枠の **尺度 SCALE** 欄とツールバーに表示されます。既定では「結果を自動で全体表示」が有効で、候補選択後・最適化完了後に機構が画面からはみ出している場合のみ自動でフィットします(手順04のチェックボックスで無効化できます)。

## 実装内容 / What is implemented

論文の対話的パイプラインの中核部分(論文 Fig. 2 の a–c 相当)を実装しています。

| パイプライン段階 | 論文 | 本実装 |
|---|---|---|
| 機構ライブラリ | パラメータ化された駆動機構群(§4, Fig. 4) | 4種:4節リンク、歯車駆動5節リンク(歯車比 −2, −1, 1, 2 の4変種)、クランク・スライダ、Stephenson 型6節リンク |
| 機構シミュレーション | 制約ベースシミュレーション+Newton–Raphson(§3) | 閉形式の順運動学(円交点計算+分岐の連続性チェック) |
| パラメータ空間探索 | メトリック空間での Poisson-disk サンプリング(§4.1) | 特徴量空間での Poisson-disk 型棄却サンプリング(起動時に約1,040曲線を構築) |
| 曲線メトリック | 特徴量ベース+係数の対話学習(§5) | 特徴量(長さ・面積・楕円率・自己交差数)+頂点距離。係数は固定値 |
| 連続最適化 | 陰関数定理による勾配+BFGS(§4.2, 式(2)–(4)) | Nelder–Mead 法(機構パラメータ+相似変換を同時最適化) |
| 駆動 | 単一入力ドライバ(§6.1) | 単一クランク駆動のアニメーション |
| 表示 | ―(論文の対象外) | 図面ビューのズーム・パン、1周期分の外接矩形への自動フィット |
| 出力 | 3Dプリント用出力(§6.4) | 実寸(mm)換算した機構パラメータの JSON / CSV / DXF 出力。歯車は5節リンクのみ実諸元(モジュール・歯数・基準円)を算出 |

**対象外(論文にあるが未実装)/ Not implemented:**
非円形歯車によるタイミング制御(§4.3)、歯車列の自動生成(§6.1)、衝突回避のレイヤ配置(§6.2)、支持構造生成(§6.3)、3Dプリント用出力(§6.4)、メトリック係数の対話学習(§5.2)。

Timing control via non-circular gears (§4.3), automatic gear-train generation (§6.1), collision-free layering (§6.2), support-structure generation (§6.3), fabrication export (§6.4), and interactive metric training (§5.2) are out of scope.

## 技術メモ / Technical notes

- 依存ライブラリなしの単一 HTML(Canvas 2D + Vanilla JS、約1,830行)
- 曲線は等弧長で 72 点にリサンプリングして比較。位相(タイミング)情報は論文同様に破棄し、巡回オフセットと向きを総当たりで対応付け
- 検索は特徴量距離で上位40件に絞ってから頂点距離で再ランキング
- 最適化は最大450反復の Nelder–Mead をフレーム分割実行(UI を停止させない)。プリセット曲線に対して RMS 誤差 1〜16 px(目標曲線 約360 px 規模)、実行時間 0.1 秒未満
- 鏡映対称の曲線に対応するため、初期化時に回転 {0, π} × 鏡映 {あり, なし} の4通りを評価して最良の初期値を選択
- 候補の**提示順は曲線メトリック順**だが、**自動選択は初期目的関数 F が最良の候補**にしている。メトリックの順位と実際の当てはまりは一致しないことがあるため(候補カードに両方の値を表示)
- 6節リンクは Stephenson 型。4節ループを解いて連結節上の点 C を求め、それを入力とする第2ダイアド(C-D-O6)を再び円交点で解く2段構成。閉形式のまま解ける
- 6節リンクの DB サンプリングは、12個のパラメータを独立に振ると第2ループがほぼ成立しないため、**先に4節ループを解いて C の軌跡を求め、`|C-O6|` の最小・最大から第2ループが必ず解ける `e`, `f` を逆算**している。これで実現可能率が 96% になった
- 変数が増えると Nelder–Mead の収束が遅くなるため、6節リンク(機構12 + 配置4 = 16次元)のみ最大反復数を 900 にしている(他の機構は 450 のまま)
- 目標曲線・機構の座標はすべて図面座標(world)で保持し、描画の直前にのみ `screen = origin + scale × world` の1段のビュー変換を適用。線幅・ピボット記号・マーカーは画面基準で一定サイズを保つため、拡大しても図面としての可読性が落ちない
- 出力は機構の局所座標系をそのまま mm 倍する方式。最適化変数のうち配置(回転 psi・平行移動)は捨て、相似変換のスケール成分 `exp(x[dim+1]) / lmax` だけを実寸換算に使う。目標曲線は逆写像で機構座標系へ戻して同梱する

## 引用 / Citation

本実装は下記論文の非公式実装です。原著は Disney Research Zurich / Disney Research Boston / MIT CSAIL によるものです。研究・教育目的でお使いの際は原論文を引用してください。

This is an **unofficial** implementation. The original work is by Disney Research Zurich / Disney Research Boston / MIT CSAIL. Please cite the original paper:

**原著の資料 / Original resources:**

- プロジェクトページ / Project page: https://studios.disneyresearch.com/2013/07/21/computational-design-of-mechanical-characters/
- 論文 PDF / Paper PDF: https://studios.disneyresearch.com/wp-content/uploads/2019/03/Computational-Design-of-Mechanical-Characters-1.pdf
- 解説動画 / Video: https://www.youtube.com/watch?v=DfznnKUwywQ
- DOI: https://doi.org/10.1145/2461912.2461953

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

## 謝辞 / Acknowledgments

本実装のコードは Anthropic の Claude を用いて作成しました。
This implementation was developed with the assistance of Anthropic's Claude.

## ライセンス / License

本リポジトリのコードは MIT License で公開しています([LICENSE](LICENSE) を参照)。論文の手法・図・本文の著作権は原著者および ACM に帰属します。

The code in this repository is released under the MIT License (see [LICENSE](LICENSE)). The method, figures, and text of the original paper remain the property of the original authors and ACM.
