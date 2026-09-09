---
theme: neversink
layout: cover
title: Error-Correction of Matrix Multiplication Algorithms
slide_info: false
neversink_slug: 'dentsu_2026'
info: |
  最適化，計算機科学，代数幾何 2026

  2026年9月24〜26日, 電気通信大学

author: Nobutaka Shimizu
mdc: true
css: unocss
style: |
  @import './styles/custom.css';
addons:
  - '@/addons/slidev-addon-bomb'
bomb:
  slideNum: true
fonts:
  sans: 'Roboto'
  mono: 'Fira Code'
  weights: '400,500,700'
  italic: true
favicon: 'https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6/svgs/solid/book.svg'
themeConfig:
  primary: '#1976d2'

---

# 行列積アルゴリズムの誤り訂正

<div class="grid grid-cols-10 gap-8 items-center h-56 mt-2">

<div class="col-span-6 cover-authors-wrap">

<div class="cover-authors">
  <div class="cover-author">
    <a class="cover-author-name" href="https://sites.google.com/view/nobutaka-shimizu/home">清水 伸高</a>
    <span class="cover-author-affil">東京科学大学</span>
  </div>
</div>

</div>
<div class="col-span-4">

  <QRCode value="https://nobutakashimizu.github.io/dentsu_2026/1" :size="120" render-as="svg"/>

</div>

</div>

:: note ::
<div class="text-slate-500">

  2026年9月24〜26日, [最適化・計算機科学・代数幾何](https://sites.google.com/view/oca2026/)@電気通信大学

</div>


---
layout: top-title
color: amber-light
---

::title::

# トークの内容



::content::

<div class="topic-box">

最近書いた論文
- Error-Correction of Matrix Multiplication Algorithms, Hirahara and **S.** (STOC 25)
- An Optimal Error-Correcting Reduction for Matrix Multiplication, Hirahara and **S.** (ICALP 25)
- Error-Correction of Matrix Multiplication Algorithms, Hirahara and **S.** (FOCS 26)

</div>

- STOC, ICALP, FOCSに採択
  - **STOC, FOCS**: 理論計算機科学分野の一番すごい国際会議
  - 色んな数学（主に組合せ論）の道具や概念がよく出てくる
    - 上記の論文で出てくる道具：正則化補題（加法的組合せ論）,エクスパンダー,誤り訂正符号

---
layout: top-title
color: amber-light
---

::title::

# 行列積


::content::

<div class="topic-box">

入力として与えられた二つの行列$A,B\in\mathbb{F}^{n\times n}$に対して$AB$を計算せよ（$\mathbb{F}$は有限体）.

</div>

愚直に計算すると,$n^3$に比例する回数の演算が必要（計算量$O(n^3)$）

<NaiveMatrixMulAnimation />

<v-click>

- [Strassen (1968)](https://link.springer.com/article/10.1007/BF02165411)によって $O(n^{\textcolor{#c2185b}{2.807}})$時間アルゴリズムが与えられた
- 以降,現在も指数部分の改善が続いている

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# 行列積


::content::

<div class="topic-box">

入力として与えられた二つの行列$A,B\in\mathbb{F}^{n\times n}$に対して$AB$を計算せよ ($\mathbb{F}$は有限体).

</div>

行列積の計算量を$O(n^\omega)$とする

<div style="display: flex; gap: 1em; font-size: 0.7em; border: 1px solid #ccc; padding: 0.5em; border-radius: 5px;">
<div>

| 年 | $\omega$ | 論文 |
|:--:|:--|:--|
| 1968 | $2.807$ | [Strassen](https://link.springer.com/article/10.1007/BF02165411) |
| 1978 | $2.795$ | [Pan](https://ieeexplore.ieee.org/document/4567976) |
| 1979 | $2.779$ | [Bini, Capovani, Romani, Lotti](https://www.sciencedirect.com/science/article/pii/0020019079901133) |
| 1981 | $2.522$ | [Schönhage](https://epubs.siam.org/doi/10.1137/0210032) |
| 1981 | $2.517$ | [Romani](https://epubs.siam.org/doi/10.1137/0211020) |

</div>
<div>

| 年 | $\omega$ | 論文 |
|:--:|:--|:--|
| 1981 | $2.496$ | [Coppersmith, Winograd](https://ieeexplore.ieee.org/document/4568320) |
| 1986 | $2.479$ | [Strassen](https://ieeexplore.ieee.org/document/4568194) |
| 1990 | $2.3755$ | [Coppersmith, Winograd](https://www.sciencedirect.com/science/article/pii/S0747717108800132?via%3Dihub) |
| 2010 | $2.3737$ | [Stothers](https://era.ed.ac.uk/handle/1842/4734) |
| 2012 | $2.3729$ | [Williams](https://dl.acm.org/doi/10.1145/2213977.2214056) |

</div>
<div>

| 年 | $\omega$ | 論文 |
|:--:|:--|:--|
| 2014 | $2.3728639$ | [Le Gall](https://dl.acm.org/doi/10.1145/2608628.2627493) |
| 2020 | $2.3728596$ | [Alman, Williams](https://theoretics.episciences.org/14213) |
| 2022 | $2.371866$ | [Duan, Wu, Zhou](https://ieeexplore.ieee.org/document/10353208) |
| 2024 | $2.371552$ | [Williams, Xu, Xu, and Zhou](https://epubs.siam.org/doi/10.1137/1.9781611977912.134) |
| 2025 | $2.371339$ | [Alman, Duan, Williams, Xu, Xu, and Zhou](https://epubs.siam.org/doi/10.1137/1.9781611978322.63) |

</div>
</div>

<style>
th {
  background-color: #f0f0f0;
}
</style>

---
layout: top-title
color: amber-light
---

::title::

# 行列積の計算量の推移

::content::

<OmegaProgressChart />

---
layout: top-title
color: amber-light
---

::title::

# 行列積の近似


::content::


<div class="question">

入力として与えられた二つの**一様ランダム**な行列 $A,B\sim\mathbb{F}^{n\times n}$ に対して, 行列$C\in\mathbb{F}^{n\times n}$であって,
$AB$の**できるだけ多くの成分**を計算せよ.

</div>

<ApproxMatrixMulAnimation />

---
layout: top-title
color: amber-light
---

::title::

# 行列積の近似


::content::

<div class="definition">

アルゴリズム $\mathsf{Algo}$ の**平均近似率** $\alpha$ を以下で定義

$$
  \alpha:=\Pr_{\substack{A,B\sim\F^{n\times n}\\ i,j\sim[n]}}[\mathsf{Algo}(A,B)_{i,j}=(AB)_{i,j}]
$$

</div>

- $\alpha = 1$はランダム行列に対する**厳密**行列積
- $\alpha = \frac{1}{\abs{\F}}$なら簡単 (ランダムな行列を出力すればよい)

<div class="question">

$n^{2+o(1)}$時間で非自明な **$\alpha > \frac{1}{|\F|}+\varepsilon$** を達成できるか?

＊ $|\F|$ や $\varepsilon$ は $n$ に依存しない定数として扱う

</div>


---
layout: top-title
color: amber-light
---

::title::

# 動機


::content::

- 高速行列積の多くのアルゴリズムは非実用的
  - 定数倍が非常に大きい ($n>10^{155}$じゃないとStrassenより早くならない <a href="https://epubs.siam.org/doi/10.1137/1.9781611978322.61" class="cite-reference">\[Alman, Yu, 2025\]</a>)

<v-clicks>

- AI技術の多くがGPU上で大規模な行列積（例えば勾配計算）に依存するため,電力消費量が世界的に増加傾向にある <a class="cite-reference" href="https://www.iea.org/reports/electricity-2024/executive-summary"> \[International Energy Agency\] </a>


- 近年, 物理系を利用した**省エネ**行列積アルゴリズムが提案されている: 
  - 水流 <a href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITCS.2024.96" class="cite-reference">\[Valinat, 2024\]</a>
  - 熱力学系 <a href="https://openreview.net/forum?id=6flkWTzK2H" class="cite-reference">\[Coles et al, 2023\]</a>
  - 光学デバイス <a href = "https://www.nature.com/articles/s41377-022-00717-8" class="cite-reference">\[Zhou et al, 2022\]</a>

- 物理系に依拠するアルゴリズムはホワイトノイズによるエラーが発生しうる (つまり近似行列積を解いてる)

<div class="topic-box" style="margin: 0; text-align: center;">

省エネな近似行列積アルゴリズム + 我々の誤り訂正 = ？？？

</div>

</v-clicks>


---
layout: top-title
color: amber-light
---

::title::

# 問題設定


::content::

<div class="definition">

二つの行列$C,D\in \mathbb{F}^{n\times n}$の**近似率** $\agr(C,D)$を以下で定義する:

$$
  \begin{align*}
    \agr(C,D) &:= \Pr_{i,j\sim[n]}[C(i,j) = D(i,j)]
  \end{align*}
$$

</div>

<v-click>

- $1-$**ハミング距離** (異なる値をとっている成分の割合) に等しい
- 平均近似率 $\alpha$ は以下のように表せる：
$$
  \begin{align*}
    \Exp_{\substack{A,B\sim\Fp^{n\times n}}}[\agr(M(A,B),AB)] &\ge \alpha
  \end{align*}
$$

</v-click>

---
layout: top-title
color: amber-light
---
::title::

# 主結果


::content::

<div class="theorem">

任意の定数$\varepsilon>0$に対し, もし平均近似率 $\textcolor{c2185b}{\alpha\ge \frac{1}{\abs{\F}}+\varepsilon}$ を達成する $\textcolor{c2185b}{T(n)}$ 時間アルゴリズム$M$が存在するならば, **厳密行列積**を解く $\textcolor{c2185b}{T(n)\cdot (\log n)^{O(1)}}$ 時間乱択アルゴリズム$M'$が存在する. すなわち

$$

\forall A,B\in\F^{n\times n},\quad \Pr_{M'}\left[ M'(A,B) = AB \right] \ge \frac{2}{3}.

$$

</div>

<v-clicks>

- ランダムな推測 $\alpha=1/|\F|$ よりちょっとでも良い近似率が達成できたら,行列積が解ける
- **平均時近似から最悪時厳密への帰着**: ランダムな入力で近似的に解ける $\Rightarrow$ 任意の入力で厳密に解ける
- $|\F|,\varepsilon$に対する依存度は大きい: $M'$の計算量は$\textcolor{c2185b}{2^{2^{\poly(|\F|,1/\varepsilon)}}}\cdot T(n) \cdot (\log n)^{O(1)}$

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::

# 関連結果


::content::


- <a class="cite-reference" href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2024.34">\[Gola, Shinkar, Singh, RANDOM'24\]</a>: 近似行列積の問題設定を最初に考えた論文
  - (追加の仮定の下で) $\alpha>\frac{8}{9}$が達成可能$\Rightarrow$ $\alpha=1$を達成可能.
  - $\alpha\ge\frac{1}{\abs{\F}}+\varepsilon$にできるかは未解決だったが,我々はこれを解決

<v-clicks>


- 厳密行列積に対する**最悪時から平均時への帰着**
  - <a href="https://www.sciencedirect.com/science/article/pii/002200009390044W?via%3Dihub" class="cite-reference">\[Blum, Luby, Rubinfeld, JCSS'93\]</a>
  - <a href="https://dl.acm.org/doi/10.1145/3519935.3520041" class="cite-reference">\[Asadi, Golovnev, Gur, Shinkar, STOC'22\]</a>
  - <a href="https://dl.acm.org/doi/10.1145/3564246.3585189" class="cite-reference">\[Hirahara, Shimizu, STOC'23\]</a>
  
- 後続研究: $\varepsilon$への依存度の改善
  - <a class="cite-reference" href="https://arxiv.org/abs/2502.13065">\[Vaikuntanathan, Zamir, '25\]</a> (ただし, Learning with Parityの計算量的困難性に依拠)
  - <a class="cite-reference" href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2025.29">\[Shinkar, Singh, RANDOM'25\]</a>: 我々の$2^{2^{\poly(1/\varepsilon)}}$を$2^{\poly(1/\varepsilon)}$に改善


</v-clicks>

---
layout: top-title
color: amber-light
---

::title::

# アイデア: 行列の符号化


::content::

任意の入力 $A,B$ に対して近似を計算するアルゴリズム $M$ を仮定する

<EncodingReductionAnimation />

<v-click>

<div class="topic-box">

ポイント: **誤り訂正符号**を使ってencoding/decodingを設計

</div>

</v-click>

---
layout: section
color: amber-light
---

# アルゴリズム的誤り訂正符号の入門



---
layout: top-title
color: amber-light
---

::title::

# 符号の基礎


::content::

- 有限体上の線型写像 $\Enc\colon \F^n\to\F^N$ を **符号化関数**という
  - 符号化関数 $\Enc$ に対し, $\calC = \Enc(\F^n)$を **(線形)符号** という (要はただの部分空間)
  - 常に $N\ge n$ を仮定し、$r=n/N$ を**レート**と呼ぶ
  - ランク落ちはしないと仮定 ($\dim\calC=n$)

- ベクトル同士の距離: $\dist(x,y) = \frac{1}{n}\sum_{i\in[n]} \mathbf{1}_{x(i)\ne y(i)}$ (ハミング距離)
- 符号$\calC$の**距離**: $\delta := \min_{\substack{x\ne y \\x,y\in \calC}}\dist(x,y)$
  

<div style="display: flex; justify-content: center; align-items: center;">

![code](./images/code.svg)

</div>      

---
layout: top-title
color: amber-light
---

::title::

# 符号の基礎 (一意復号)


::content::


- $\mathrm{ball}(x,\rho) = \{ y\in\F^n\colon \dist(x,y)\le\rho \}$: 中心$x$半径$\rho$のハミングボール

<div class="definition">

符号$\calC$は, 任意の$y\in\F^N$に対して **$\abs{\ball(y,\rho)\cap \calC} \le 1$** であるとき, **半径 $\rho$ で一意復号可能である**.

</div>

<ListDecodingBall unique />


---
layout: top-title
color: amber-light
---

::title::

# 符号の基礎 (一意復号)


::content::


- $\mathrm{ball}(x,\rho) = \{ y\in\F^n\colon \dist(x,y)\le\rho \}$: 中心$x$半径$\rho$のハミングボール

<div class="definition">

符号$\calC$は, 任意の$y\in\F^N$に対して **$\abs{\ball(y,\rho)\cap \calC} \le 1$** であるとき, **半径 $\rho$ で一意復号可能である**.

</div>

<ListDecodingBall unique packing />

---
layout: top-title
color: amber-light
---

::title::

# 符号の基礎 (一意復号の限界)


::content::

- 符号の距離が$\delta$であるとき, $\rho < \delta/2$でなければならない
  - 特に, 訂正できるエラーの割合は $\rho < 1/2$


- 50\%を超えるエラーを一意に復号するのは不可能
  - 復号の一意性を緩めることによって対処 -> リスト復号

<div style="display: flex; justify-content: center; align-items: center;">

![一意復号の限界](./images/unique_radius.svg)

</div>

<figcaption style="text-align: center; font-size: 0.8em; color: #666;">

半径$\rho$で一意復号可能な符号を作るには, 半径$\rho$のHammingボールを$\F^n$内に敷き詰めればよい.

</figcaption>

---
layout: top-title
color: amber-light
---

::title::

# 符号の基礎 (リスト復号)


::content::

<div class="definition">

符号$\calC\subseteq \F^N$ は, 全ての$y\in \F^N$に対して

$$

\abs{\ball(y,\rho)\cap \calC} \le L

$$

を満たすとき, **リストサイズ$L$で$\rho$-リスト復号可能である**という。
- 特に $L=O(1)$ となる最大の $\rho^*$ を **リスト復号半径** という (符号の族を考え $n\to\infty$ の漸近評価)
</div>

<ListDecodingBall />

---
layout: top-title
color: amber-light
---

::title::

# アルゴリズム的側面


::content::

- **リスト復号**アルゴリズム: $y\in \F^N$を入力として受け取り, $\mathrm{ball}(y,\rho)\cap\mathcal{C}$の元を**全て**出力
  - 出力する個数は全部で$L$個 (リストサイズ)
- 自明なアルゴリズム: 全ての $x\in\F^n$ を列挙して $\dist(\Enc(x),y)\le \rho$ かチェック
  - ランダム符号は高速化に役立ちそうな構造がなく,効率的な一意復号すら難しいと予想（LPN問題）

<v-clicks>

<div class="question">

最適なパラメータおよび構造的性質をもち, 効率的にリスト復号できる符号があるか?
  1. 距離:$\delta\approx 1-\frac{1}{|\F|}$
  2. リスト可能半径 $\rho\approx \delta\approx 1-\frac{1}{|\F|}$
  3. 構造的性質を持ち, 効率的にリスト復号可能な符号

</div>

-> **エクスパンダーウォーク符号** <a href="https://dl.acm.org/doi/10.1145/3055399.3055408" class="cite-reference">\[Ta-Shma, 2017\]</a>

</v-clicks>


---
layout: section
color: amber-light
---

# 行列の符号化

---
layout: top-title
color: amber-light
---

::title::

# テンソル符号

::content::

<div class="definition">

符号化関数 $\mathrm{Enc}\colon \F^n \mapsto \F^{N}$ が, 行列 $L\in\F^{N\times n}$ を用いて $\Enc(x)=Lx$ と表せるとき、これに対応する
**テンソル符号化関数** $\Enc'\colon \F^{n\times n}\to \F^{N\times N}$ を

$$
\Enc'\colon X \mapsto L X L^\top
$$

とし、$\Enc'(\F^{n\times n})$ を **テンソル符号** という.

</div>

- 元の符号 $\Enc$ が効率的に（リスト）復号できるとき、そのテンソル符号もほぼ同様 <a href="https://epubs.siam.org/doi/10.1137/090778274" class="cite-reference">\[Gopalan, Guruswami, Raghavendra, 2011\]</a>.

<div class="theorem">

元の符号が半径 $1-c$ でリスト復号できるなら、ほぼ同じ計算量でテンソル符号は半径 $1-2c$ でリスト復号できる。

</div>

---
layout: top-title
color: amber-light
---

::title::

# テンソル符号による行列の符号化

::content::

<TensorEncodingDiagram />

<figcaption style="text-align: center; font-size: 0.8em; color: #666; margin-top: 0.85rem;">

アルゴリズム$M$のエラー率が復号可能半径内に収まっていれば, リスト復号可能

</figcaption>

---
layout: top-title
color: amber-light
---

::title::

# リスト復号 -> $AB$ の復元

::content::

- リスト復号によって, 複数の行列 $C_1,\dots,C_\ell\in\F^{n\times n}$ が得られる
  - ある $i$ に対して $C_i = AB$ が保証されている -> そのような $i$ はどれか？
- <a class="cite-reference" href="https://link.springer.com/chapter/10.1007/3-540-09526-8_5">\[Freivalds, 1979\]</a> のアルゴリズムを適用すればよい

<div class="theorem">

与えられた三つの行列 $A,B,C\in\F^{n\times n}$ に対し、**$AB=C$ かどうか** を $O(n^2)$ 時間で判定する乱択アルゴリズムが存在する。

</div>

- アルゴリズム：一様ランダムなベクトル $x,y\sim \F^n$ に対して $x^\top (AB) y = x^\top C y$ かどうかを確認
- $AB\ne C$ ならば、$(\textcolor{#c2185b}{x},\textcolor{#c2185b}{y}) \mapsto \textcolor{#c2185b}{x}^\top (AB-C) \textcolor{#c2185b}{y}$ は非ゼロな次数$2$の他変数多項式
  - 確率 $1-2/|\F|$ で $x^\top (AB-C) y \ne 0$ となる (Schwartz--Zippelの補題)

---
layout: top-title
color: amber-light
---

::title::

# テンソル符号化の問題点

::content::

- リスト復号の半径にロスが生じる

<div class="theorem">

元の符号が半径 $1-c$ でリスト復号できるなら、ほぼ同じ計算量でテンソル符号は半径 $1- \textcolor{#c2185b}{2c}$ でリスト復号できる。

</div>

- 非自明な近似率では、$\alpha = \frac{1}{|\F|}+\varepsilon$
- テンソル符号のリスト復号半径を $1-\frac{1}{|\F|}-\varepsilon$ にする必要がある
  - $c\approx \frac{1}{2|\F|}$ にしなければならない
- しかし、半径$1-\frac{1}{2|\F|}$ は一般にリスト復号可能ではない (半径が大きすぎる)

<div class="topic-box">

テンソル符号は使わず、エクスパンダーグラフを用いた符号で代替する.

</div>

---
layout: section
color: amber-light
---

# エクスパンダーグラフと符号


---
layout: top-title
color: amber-light
---

::title::

# エクスパンダーグラフ

::content::

- $d$-正則グラフ $G=(V,E)$（全ての頂点にちょうど$d$本の辺が繋がってる）
- 隣接行列を $A\in\binset^{V\times V}$
  - 頂点ペア $u,v\in V$ に対し,$A_{u,v}=\mathbf{1}_{\{u,v\}\in E}$
- 実行列$A$は対称なので,実固有値 $\lambda_1\ge \lambda_2 \ge \dots \ge \lambda_{|V|}$ を持つ
  - 最大固有値は $\lambda_1=d$ となる（全成分$1$のベクトルが固有ベクトル）

<div class="definition">

$\max\{|\lambda_2|,|\lambda_{|V|}|\} \le \textcolor{#c2185b}{\gamma}\cdot d$ を満たすグラフを **$\gamma$-エクスパンダーグラフ** という.

</div>

- $\gamma$ は小さいほど「強いエクスパンダー性」を持つ
- 直感的には、「疎なのに連結性が強い」グラフ

---
layout: top-title
color: amber-light
---

::title::

# エクスパンダーグラフ

::content::

<ExpanderCompare />

---
layout: top-title
color: amber-light
---

::title::

# 直和符号

::content::

<div class="definition">

パラメータ $\ell\in\Nat$ に対して以下の $\Enc\colon \F^n\to\F^{n^\ell}$ を **直和符号** という:
$$
x\mapsto (x(i_1)+\dots+x(i_\ell))_{i_1,\dots,i_\ell\in[n]}
$$

</div>

<DirectSumEncodingAnimation />

<div class="topic-box">

1. 直和符号は非常に高い誤り訂正能力をもつ (Yao's XOR Lemma)
2. 冗長性が非常に高い

</div>

-> 「良いとこどり」の符号を作れないか？

---
layout: top-title
color: amber-light
---
::title::
# エクスパンダーウォーク符号

::content::

<div class="definition">

$d$-正則**エクスパンダーグラフ** $G=(V,E)$ を考える.
パラメータ $\ell\in\Nat$ に対して、$G$上の長さ$\ell-1$のウォーク全体の集合を $W\subseteq V^\ell$ とする ($\abs{W}=n\cdot d^{\ell}$).

以下の符号化関数 $\Enc\colon \F^V\to\F^W$ で定まる符号を**エクスパンダーウォーク符号**という:

$$
  \Enc(x) = \rbra{ x(v_0)+x(v_1)+\dots+x(v_{\ell-1}) }_{(v_0,v_1,\dots,v_{\ell-1})\in W }.
$$

</div>

<ExpanderWalkEncodingAnimation />

---
layout: top-title
color: amber-light
---
::title::

# エクスパンダーウォーク符号の性質


::content::


- グラフの次数$d$と歩数$\ell$が定数ならば, **レートは$\frac{|V|}{|W|}=d^{-\ell}=\Omega(1)$**
  - 直和符号だと $n^{-\ell+1}$

- **半径 $\rho=1-\frac{1}{\abs{\F}}-\varepsilon$** で 効率的に**近似**リスト復号が可能 <a href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2023.60" class="cite-reference">\[Jeronimo, RANDOM'23\]</a>
  - 計算時間は$2^{2^{\poly(1/\varepsilon)}}\cdot |V|(\log |V|)^{O(1)}$
  - $\varepsilon$への依存度は大きい (Frieze-Kannanの弱正則化補題)
- 符号化 $z \mapsto \Enc(z)$ の計算も$O(|V|)$時間でできる

<div class="topic-box">

- 符号化も復号も高速
- 行列積を「符号化」できる（後述）

</div>

---
layout: top-title
color: amber-light
---

::title::
# 行列積の符号化

::content::

<div class="topic-box">

行列$A,B\in\F^{n\times n}$に対し, うまく$A',B'$を構成して
$A'B'=\Enc(AB)$となるようにしたい.

</div>

<ExpanderWalkReductionAnimation />


---
layout: top-title
color: amber-light
---
::title::
# 行列$A',B'$の構成

::content::

<div style="display: flex; justify-content: center; align-items: center;">

![行列の構成](./images/expander_construction.svg)

</div>

<figcaption style="text-align: center; font-size: 0.8em; color: #666;">

行列$A'$は$\abs{W}\times \ell n$行列となる. 第$\mathbf{i}=(i_1,\dots,i_\ell)$行目には, $A$の第$i_1$行ベクトル, 第$i_2$行ベクトル, ... を並べる.
<br>
$B'$は$B$に対し, 行と列を入れ替えて同じ操作を行って構成する.

</figcaption>

---
layout: top-title
color: amber-light
---
::title::
# 行列$A',B'$の構成

::content::

<div style="display: flex; justify-content: center; align-items: center;">

![行列の構成](./images/lifting.svg)

</div>

- $A'B' \in \F^{W\times W}$の第$(\mathbf{i},\mathbf{j})$成分は, $(AB)_{i_1,j_1} + \dots + (AB)_{i_\ell,j_\ell}$に一致する ($\mathbf{i},\mathbf{j}$は$G$上のウォーク)
- これは, テンソル積$G^2$上のウォーク$(i_1,j_1)\to \dots \to (i_\ell,j_\ell)$に沿った和とみなせる.
- すなわち, $AB$を, **$G^2$上のエクスパンダーウォーク符号**で符号化したものとみなせる.

---
layout: top-title
color: amber-light
---
::title::

# まとめ


::content::

- 行列積に対する近似アルゴリズムが設計できたら, ほぼ同程度の時間で全成分を計算する行列積アルゴリズムが構成できる
- 証明手法: 誤り訂正符号のリスト復号を行列に適用
  - **エクスパンダーウォーク符号** + 近似リスト復号アルゴリズム
- **体が大きい時**でも別の符号を使えば同様の結果を示せる
  - 例: $\abs{\F} \ge n/\alpha$のとき, リードソロモン符号+テンソル符号 <a href="https://dl.acm.org/doi/10.1145/3717823.3718244" class="cite-reference">\[Hirahara, Shimizu, STOC'25\]</a>
- 今後の方向性
  - **実数**上の行列積で同様のことができないか? (実用的には実数上の行列積が主流のはず)
    - どのような定式化が良いか?
  - $\varepsilon$への依存度の改善 <a class="cite-reference" href="https://arxiv.org/abs/2502.13065">\[Vaikuntanathan, Zamir, '25\]</a>, <a class="cite-reference" href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2025.29">\[Shinkar, Singh, RANDOM'25\]</a>