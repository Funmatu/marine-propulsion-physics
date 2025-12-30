# Marine Propulsion Physics Simulation
### 和船の知恵 vs 現代工学：推進方式の物理学的比較検証

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Three.js](https://img.shields.io/badge/Three.js-r128-black)
![Physics](https://img.shields.io/badge/Physics-Momentum%20Theory-red)
![Status](https://img.shields.io/badge/Status-Stable-green)

> **[Live Demo (GitHub Pages) はこちら](https://funmatu.github.io/marine-propulsion-physics/)**  
> ※ ブラウザ上でリアルタイムに動作します。インストール不要。

---

## 📖 概要 (Overview)

本プロジェクトは、船舶推進における3つの異なる方式（**櫓、スクリュープロペラ、ウォータージェット**）の物理的挙動を、流体力学および運動量理論に基づき比較検証するためのシミュレータです。

日本古来の「櫓（Ro）」が持つ低速域での驚異的な効率と、現代の「ウォータージェット」が持つ高速域での優位性が、どのような物理法則によって入れ替わるのか（クロスオーバー現象）を、Three.jsによる3D可視化とリアルタイムグラフ解析によって解明します。

---

## 🧪 物理モデルと理論背景 (Physics Engine & Theory)

本シミュレーションの核心は、単純なアニメーションではなく、**運動量理論（Momentum Theory / Actuator Disk Theory）** に基づく推力計算を行っている点にあります。

### 1. 推力の発生原理
推進器が発生させる推力 $T$ は、単位時間あたりに推進器を通過する水の質量（質量流量 $\dot{m}$）と、その水が得た速度変化（$\Delta V$）の積として定義されます。

$$ T = \dot{m} (V_{jet} - V_{ship}) $$

ここで、
*   $\rho$: 水の密度 ($1025 kg/m^3$)
*   $A$: 推進器の有効断面積 ($m^2$)
*   $V_{ship}$: 船速 ($m/s$)
*   $V_{jet}$: 排出流速 ($m/s$)

質量流量 $\dot{m}$ は以下のように近似されます（アクチュエータディスク理論の簡易モデル）。

$$ \dot{m} \approx \rho A V_{jet} $$

したがって、推力は以下の式で導出されます。

$$ T \approx \rho A V_{jet} (V_{jet} - V_{ship}) $$

### 2. 入力エネルギーと排出流速の関係
入力された仕事率（Power, $P_{in}$）は、機械効率 $\eta_{mech}$ を経て流体の運動エネルギーに変換されます。

$$ P_{eff} = P_{in} \cdot \eta_{mech} = \frac{1}{2} \dot{m} (V_{jet}^2 - V_{ship}^2) $$

本シミュレータでは、入力パワーと断面積 $A$ から排出流速 $V_{jet}$ を逆算し、そこから推力を求めています。これにより、**「断面積が大きいほど、少ないエネルギーで大きな推力を得られる（静止推力）」** という物理則と、**「断面積が小さいほど、高速域まで推力を維持できる」** というトレードオフの関係を再現しています。

### 3. 各推進方式のパラメータ設定

| 推進方式 | 断面積 ($A$) | 特性 | 物理的挙動 |
| :--- | :--- | :--- | :--- |
| **櫓 (Ro)** | 極大 ($1.6 m^2$) | 大流量・低流速 | 漕ぎ出しの推力は最強。しかし、船速 $V_{ship}$ が上がるとすぐに $V_{jet}$ との差がなくなり、推力がゼロになる（低速トルク型）。 |
| **スクリュー** | 中 ($0.18 m^2$) | バランス型 | 櫓とジェットの中間特性。広範な速度域で安定した性能を発揮する。 |
| **ジェット** | 極小 ($0.018 m^2$) | 小流量・高流速 | 低速時はスリップロス（ $V_{jet} - V_{ship}$ ）が大きく効率最悪。しかし、 $V_{ship}$ が上昇しても推力低下が少なく、最終的に最高速に達する。 |

### 4. 流体抵抗 (Hydrodynamic Drag)
船体が受ける抗力 $D$ は、速度の二乗に比例する粘性抵抗と、造波抵抗をモデル化しています。

$$ D = \frac{1}{2} \rho V_{ship}^2 C_d A_{frontal} + k \cdot V_{ship} $$

*   低速域：粘性抵抗が支配的
*   高速域：造波抵抗（波を作るエネルギー損失）が増大

---

## 📊 実装機能 (Features)

### リアルタイム物理演算
*   **クロスオーバー現象の再現:** スタート直後は「櫓」がリードし、中盤で「スクリュー」、最終的に「ウォータージェット」が追い抜く物理現象をシミュレート。
*   **効率計算:** 推進効率 $\eta = \frac{T \cdot V_{ship}}{P_{in}}$ をリアルタイムに算出・グラフ化。

### 高度な3Dビジュアライゼーション
*   **プロシージャル船体生成:** `THREE.ExtrudeGeometry` を用いた流線型の船体モデリング。
*   **動的な海洋表現:** 頂点シェーダー処理による波のうねりと、船体のピッチング（縦揺れ）・ローリング（横揺れ）の連動。
*   **航跡（Wake）パーティクル:** 推力の大きさに応じて発生量と流速が変化する航跡表現。

### インタラクティブUI
*   **レイキャスト判定:** 3D空間上の船体をクリックすることで、詳細な物理パラメータ（推力、抗力、Yaw角、効率）を表示。
*   **エネルギー制御:** 入力ワット数（100W〜2000W）をスライダーで変更し、推力への影響を即座に確認可能。
*   **スマートカメラ:** 先頭船舶への自動追従および、選択した船舶へのフォーカス機能。

---

## 🛠 技術スタック (Tech Stack)

*   **Core Logic:** Vanilla JavaScript (ES6+)
*   **Rendering Engine:** [Three.js (r128)](https://threejs.org/)
*   **Data Visualization:** [Chart.js](https://www.chartjs.org/)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/)
*   **Physics Approach:** Fluid Dynamics / Momentum Theory approximation

---

## 🚀 ローカルでの実行方法 (Installation)

本プロジェクトは静的なHTML/JSで構成されているため、ビルドプロセスは不要です。

1.  リポジトリをクローンします。
    ```bash
    git clone https://github.com/Funmatu/marine-propulsion-physics.git
    ```
2.  ディレクトリに移動します。
    ```bash
    cd marine-propulsion-physics
    ```
3.  `index.html` をお使いのブラウザで開くだけで実行可能です。
    *   ※ VS Codeの "Live Server" 拡張機能などを使用すると、よりスムーズに開発・確認ができます。

---

## 🎮 操作マニュアル (User Guide)

1.  **Simulation Start:**
    *   画面左上の `SIMULATION START` ボタンをクリックすると物理演算が開始されます。
2.  **Input Power:**
    *   スライダーを操作して、入力エネルギー（W）を変更できます。出力が高いほどクロスオーバーのタイミングが変化します。
3.  **Analysis:**
    *   **船体をクリック**すると、画面右側に詳細分析パネルが表示されます。
    *   画面下部のグラフで「速度」と「推進効率」の推移を確認できます。
4.  **Camera:**
    *   マウスドラッグ（左ボタン）：視点の回転
    *   マウスドラッグ（右ボタン）：視点の平行移動
    *   ホイール：ズームイン・アウト
    *   `RESET VIEW` ボタンで初期位置に戻ります。

---

## 📈 今後の展望 (Roadmap)

*   [ ] キャビテーション（気泡発生による推力低下）の物理モデル追加
*   [ ] 風圧抵抗と横風の影響の実装
*   [ ] 船体形状（Vハル、フラットハル）の選択機能

---

## 📜 ライセンス (License)

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Copyright (c) 2025 Funmatu
