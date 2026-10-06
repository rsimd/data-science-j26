---
kernelspec:
  name: python3
  display_name: Python 3
  language: python
---

# 1. 正規分布と多変量正規分布

授業の進め方・評価方法は[授業ガイダンス](index.md)を参照。

## なぜ正規分布を学ぶのか

すべてのデータが正規分布に従うわけではない。それでも正規分布を学ぶのは、平均とばらつきを結びつける基本的な基準モデルだからである。たとえば測定値には、読み取りや環境など複数の小さな誤差が加わる。そうした要因が互いに独立で、極端に大きな要因が支配しないなどの条件のもとでは、その和は中心極限定理により正規分布に近づく。

$$
X=\theta+\epsilon,\qquad \epsilon\sim\mathcal{N}(0,\sigma^2)
$$

ここで $\theta$ は測りたい値、$\epsilon$ は平均0、標準偏差 $\sigma$ の誤差である。正規分布では平均 $\mu$ と標準偏差 $\sigma$ を基準に、値がどれほど離れているかを

$$
z=\frac{x-\mu}{\sigma}
$$

で表せる。正規分布を仮定したとき、平均から $\sigma$, $2\sigma$, $3\sigma$ 以内に入る割合は、それぞれ約68%、95%、99.7%である。したがって、標準化した値は、異なる単位のデータを比べたり、平均から大きく離れた値に気づいたりする手がかりになる。

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja

x = np.linspace(-4, 4, 500)
density = np.exp(-x**2 / 2) / np.sqrt(2 * np.pi)
fig, ax = plt.subplots(figsize=(7, 3.8))
ax.plot(x, density, color="#3264a8", linewidth=2.5)
inside = (x >= -1) & (x <= 1)
ax.fill_between(x[inside], density[inside], alpha=0.28, color="#2a8f6a", label="平均から±1標準偏差：約68%")
ax.axvline(0, color="#333333", linewidth=1)
ax.set_xticks([-1, 0, 1], [r"$\mu-\sigma$", r"$\mu$", r"$\mu+\sigma$"])
ax.set(xlabel="値", ylabel="確率密度", title="正規分布では、平均の近くに値が集まりやすい")
ax.grid(axis="y", alpha=0.25)
ax.legend()
fig.tight_layout()
plt.show()
```

ただし、この割合や標準化の解釈は、データを正規分布で近似してよい場合に限られる。実際のデータでは、ヒストグラムやQ-Qプロットなどで分布を確かめ、正規分布だと最初から決めつけない。

## 今日の問い

> データの「中心」「ばらつき」「変数同士の関係」を、数学的にはどう表せばよいだろうか。

```{mermaid}
flowchart LR
    A[データ] --> B[平均・分散]
    B --> C[正規分布]
    C --> D[変数が複数なら？]
    D --> E[共分散]
    E --> F[共分散行列]
    F --> G[多変量正規分布]
    G --> H[正規性]
    H --> I[2. 相関分析]
```

## 到達目標

授業後、次のことを自分の言葉で説明できることを目指す。

- 平均と分散、正規分布の形とパラメータの意味
- 標準正規分布と一般の正規分布の関係
- 共分散の符号と散布図の傾向
- 共分散行列の対角要素・非対角要素の意味
- 多変量正規分布と1変量正規分布の対応
- 正規性と、その視覚的な確認方法

## 記号の約束

数式では、スカラーを斜体、ベクトルを太字小文字、行列を太字大文字で表す。確率変数は大文字の斜体、確率変数ベクトルは大文字の太字で表す。

- スカラーの観測値 $x$、確率変数 $X$、平均 $\mu$、分散 $\sigma^2$
- ベクトルの観測値 $\boldsymbol{x}$、確率変数ベクトル $\boldsymbol{X}$、平均ベクトル $\boldsymbol{\mu}$
- 共分散行列 $\boldsymbol{\Sigma}$

演習ノートブックでは、同じ内容を自分で動かして確かめられる。

## この回の核

> 正規分布は、平均と分散によって中心と広がりを記述する。

> 多変量では、分散は共分散行列へ拡張され、各変数のばらつきと変数間の関係を表す。

> 実際の解析では、データを正規分布として扱ってよいかという「正規性」も考える。

---

## 平均と分散

### 平均：データの中心

データを $x_1,x_2,\ldots,x_n$ とすると、標本平均は

$$
\bar{x}=\frac{1}{n}\sum_{i=1}^{n}x_i
$$

である。データの中心を表す基本的な要約量である。

### 平均だけでは足りない

次の2つのデータを比べる。

$$
A=[48,49,50,51,52], \qquad B=[10,30,50,70,90].
$$

どちらの平均も50だが、値の散らばり方は大きく異なる。データの特徴を表すには、中心だけでなく、中心からのばらつきも必要になる。

### 分散：平均からのばらつき

確率変数 $X$ の平均を $\mu=E[X]$ とすると、分散は

$$
\operatorname{Var}(X)=E\left[(X-\mu)^2\right]
$$

である。

1. 各値から平均を引き、平均との差を作る。
2. 差を二乗し、正負が打ち消し合わないようにする。
3. 二乗した差の平均を取る。

データ全体を母集団とみなす分散は $\frac{1}{n}\sum_i(x_i-\bar{x})^2$、標本から母分散を推定する不偏分散は $\frac{1}{n-1}\sum_i(x_i-\bar{x})^2$ と分母が異なる。ここでは分布の形を記述する母分散の考え方を中心に扱う。

標準偏差は分散の平方根 $\sigma=\sqrt{\sigma^2}$ であり、元データと同じ単位でばらつきを表す。

```{code-cell} python
import numpy as np

A = np.array([48, 49, 50, 51, 52])
B = np.array([10, 30, 50, 70, 90])

for name, data in [("A", A), ("B", B)]:
    print(f"{name}: 平均={data.mean():.1f}, 分散={data.var():.1f}, 標準偏差={data.std():.1f}")
```

---

## 正規分布

### 形から考える

測定値などの連続データを多数集めると、平均付近の値が多く、平均から遠い値ほど少なくなる形が現れることがある。その代表的なモデルが正規分布である。ただし、どのデータも正規分布になるわけではない。

$$
X\sim\mathcal{N}(\mu,\sigma^2)
$$

と書き、確率変数 $X$ が平均 $\mu$、分散 $\sigma^2$ の正規分布に従うことを表す。$\mu$ は分布の中心、$\sigma$ は広がりの尺度、$\sigma^2$ は分散である。

### パラメータで形が変わる

- $\mu$ を変えると、分布全体が左右に移動する。
- $\sigma$ を大きくすると、分布は広く低くなる。
- $\sigma$ を小さくすると、分布は狭く高くなる。

正規分布の確率密度関数の構造は次章で確認する。

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja

x = np.linspace(-5, 8, 500)
for mu, sigma in [(0, 1), (2, 1), (0, 2)]:
    density = np.exp(-0.5 * ((x - mu) / sigma) ** 2) / (sigma * np.sqrt(2 * np.pi))
    plt.plot(x, density, label=fr"$\mu={mu}, \sigma={sigma}$")
plt.xlabel("x")
plt.ylabel("確率密度")
plt.legend()
plt.grid(alpha=0.25)
plt.show()
```

### 乱数で確かめる

以下では平均0、標準偏差1の正規分布から標本を作り、ヒストグラムを確率密度として描く。NumPyの `scale` 引数は**標準偏差**であり、分散ではない。

```{code-cell} python
rng = np.random.default_rng(7)
sample = rng.normal(loc=0, scale=1, size=5000)

plt.hist(sample, bins=45, density=True, alpha=0.65, label="標本のヒストグラム")
plt.plot(x, np.exp(-x**2 / 2) / np.sqrt(2 * np.pi), color="black", label="理論密度")
plt.xlabel("x")
plt.ylabel("確率密度")
plt.legend()
plt.show()
print(f"標本平均={sample.mean():.3f}, 標本標準偏差={sample.std(ddof=0):.3f}")
```

ヒストグラムは乱数による標本の揺らぎを含む。標本数を増やすと、一般に理論密度の形が見えやすくなる。

---

## 正規分布の確率密度関数

正規分布の密度関数を丸暗記するのではなく、式の各部分が何をしているかを確認する。平均と分散を指定すれば、あらゆる分布が正規分布に決まるわけではない。ここでは**正規分布というモデルを選んだとき**の密度関数を扱う。

### 1. 山型の関数

まず標準正規分布の形を作る関数

$$
\exp\left(-\frac{x^2}{2}\right)
$$

を見る。$x=0$ で最大となり、正負対称で、原点から離れると0へ近づく。

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja

x = np.linspace(-4, 4, 500)
y = np.exp(-x**2 / 2)
fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(x, y, color="#3264a8", linewidth=2.5, label=r"$\exp(-x^2/2)$")
ax.fill_between(x, y, alpha=0.16, color="#3264a8")
ax.scatter([0], [1], color="#c44e52", zorder=3)
ax.annotate("最大値 1", (0, 1), xytext=(0.5, 0.88), arrowprops={"arrowstyle": "->"})
ax.set(xlabel="x", ylabel="関数の値", title="1. 山型の関数")
ax.set_ylim(0, 1.1)
ax.grid(alpha=0.25)
ax.legend()
fig.tight_layout()
plt.show()
```

### 2. 面積を1にする => 標準正規分布の確率密度関数

連続型確率分布の密度は、全範囲で積分した面積が1である。ガウス積分

$$
\int_{-\infty}^{\infty}\exp\left(-\frac{x^2}{2}\right)dx=\sqrt{2\pi}
$$

より、山型の関数を $\sqrt{2\pi}$ で割れば面積1になる。

$$
\phi(x)=\frac{1}{\sqrt{2\pi}}\exp\left(-\frac{x^2}{2}\right)
$$

これを標準正規分布 $Z\sim\mathcal{N}(0,1)$ の確率密度関数とする。

```{code-cell} python
fig, ax = plt.subplots(figsize=(7, 4))
kernel = np.exp(-x**2 / 2)
density = kernel / np.sqrt(2 * np.pi)
ax.plot(x, kernel, linestyle="--", color="#888888", label=r"$\exp(-x^2/2)$（面積 $\sqrt{2\pi}$）")
ax.plot(x, density, color="#2a8f6a", linewidth=2.5, label=r"$\phi(x)$（面積 1）")
ax.fill_between(x, density, alpha=0.18, color="#2a8f6a")
ax.set(xlabel="x", ylabel="関数の値", title="2. 面積が1になるように正規化")
ax.grid(alpha=0.25)
ax.legend()
fig.tight_layout()
plt.show()
```

### 3. 平均と標準偏差を変える => 正規分布の確率密度関数

標準正規分布の確率密度関数を $\phi(z)$ とする。ここで求めたいのは、平均を $\mu$、標準偏差を $\sigma$ とする確率変数 $X$ の密度関数である。標準正規変数 $Z$ を使い、

$$
X=\mu+\sigma Z \qquad (\sigma>0)
$$

と変換すると、分布の中心が0から $\mu$ へ移り、横方向の広がりが $\sigma$ 倍になる。変換後の値 $x$ に対応する標準化した値は $z=\frac{x-\mu}{\sigma}$ である。

$z$ の小さな幅 $dz$ は、変換後には $x$ の幅 $dx=\sigma dz$ に広がる。変換の前後でその範囲に入る確率は同じであり、確率は「密度 $\times$ 幅」だから、

$$
\phi(z)\,dz=f_X(x)\,dx.
$$

したがって、変換後の密度は

$$
f_X(x)=\frac{\phi(z)}{\sigma}
=\frac{1}{\sigma}\phi\left(\frac{x-\mu}{\sigma}\right).
$$

このように、変換前後で確率を保つために密度を調整する関係を「変数変換の公式」という。ここでは横幅が $\sigma$ 倍になるので、密度の高さは $\frac{1}{\sigma}$ 倍になる。

標準正規分布の密度 $\phi(z)=\frac{1}{\sqrt{2\pi}}\exp\left(-\frac{z^2}{2}\right)$ を代入すると、

$$
f_X(x)=\frac{1}{\sigma\sqrt{2\pi}}
\exp\left[-\frac{1}{2}\left(\frac{x-\mu}{\sigma}\right)^2\right]
=\frac{1}{\sqrt{2\pi\sigma^2}}
\exp\left[-\frac{(x-\mu)^2}{2\sigma^2}\right].
$$

指数部は平均からの距離を標準偏差単位で測り、前の係数は横方向の伸縮後も密度の面積を1に保つ。

```{code-cell} python
fig, ax = plt.subplots(figsize=(7, 4))
for mu, sigma, label in [
    (0, 1, r"$\mu=0,\;\sigma=1$"),
    (2, 1, r"$\mu=2,\;\sigma=1$"),
    (0, 2, r"$\mu=0,\;\sigma=2$"),
]:
    density = np.exp(-0.5 * ((x - mu) / sigma) ** 2) / (sigma * np.sqrt(2 * np.pi))
    ax.plot(x, density, linewidth=2.3, label=label)
ax.set(xlabel="x", ylabel="確率密度", title="3. 平均と標準偏差による形の変化")
ax.grid(alpha=0.25)
ax.legend()
fig.tight_layout()
plt.show()
```

### 発展：ガウス積分

ガウス積分とは、ガウス型の関数を実数全体で積分し、その曲線の下の総面積を求める広義積分である。ここでは $e^{-x^2/2}$ の面積を求める。この値が分かると、標準正規分布の密度の面積を1にする係数 $\frac{1}{\sqrt{2\pi}}$ が得られる。

$$
I=\int_{-\infty}^{\infty}e^{-x^2/2}\,dx
$$

と置くと、積の積分として

$$
I^2=\int_{-\infty}^{\infty}\int_{-\infty}^{\infty}e^{-(x^2+y^2)/2}\,dx\,dy.
$$

極座標 $x=r\cos\theta$, $y=r\sin\theta$ では面積要素が $r\,dr\,d\theta$ になるので、

$$
I^2=\int_0^{2\pi}\int_0^\infty e^{-r^2/2}r\,dr\,d\theta=2\pi.
$$

$I>0$ より $I=\sqrt{2\pi}$ である。この結果は、直前の密度関数を面積1にする計算につながる。一般形や別の証明は、[ガウス積分の公式の２通りの証明（高校数学の美しい物語）](https://manabitimes.jp/math/754)を参照。

---

## 共分散：2つの変数の関係

### それぞれのばらつきだけで足りるか

2変量の観測ベクトル $\boldsymbol{x}=(x_1,x_2)^\mathrm{T}$ を散布図にすると、同じ平均と各変数の分散を持っていても、点群が右上がりか右下がりかで形が違うことがある。各変数の分散だけでは、2つが一緒にどう変化するかを表せない。

### 共分散

共分散は、2つの変数がそれぞれの平均からどちら向きにずれているかを、ひとつの数にまとめた量である。各データ組で「$X$ の平均との差」と「$Y$ の平均との差」を掛け、その積を平均する。両方が平均より上（または両方が平均より下）なら積は正、片方が上でもう片方が下なら積は負になる。したがって、同じ向きにずれる組が多いか、反対向きにずれる組が多いかが共分散の符号に表れる。

確率変数 $X,Y$ の平均を $\mu_X,\mu_Y$ とすると、共分散は

$$
\operatorname{Cov}(X,Y)=E[(X-\mu_X)(Y-\mu_Y)]
$$

である。ここで $E[\,\cdot\,]$ は、偏差の積を平均することを表す。

- 共分散が正：一方が大きいとき他方も大きい傾向（右上がり）
- 共分散が負：一方が大きいとき他方は小さい傾向（右下がり）
- 共分散が0付近：線形な共変動が弱い

共分散0でも、一般に変数が独立とは限らない。また共分散の大きさは測定単位に依存する。

### 点群で見る

以下の点群は、共分散行列の非対角成分だけを変えた例である。

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja

rng = np.random.default_rng(12)
for i, rho in enumerate([0.0, 0.8, -0.8], start=1):
    cov = [[1, rho], [rho, 1]]
    points = rng.multivariate_normal([0, 0], cov, size=800)
    ax = plt.subplot(1, 3, i)
    ax.scatter(points[:, 0], points[:, 1], s=5, alpha=0.45)
    ax.set_title(f"共分散={rho}")
    ax.set_aspect("equal")
    ax.grid(alpha=0.2)
plt.tight_layout()
plt.show()
```

次回は共分散を各変数の標準偏差で割り、尺度に依存しない相関係数へ進む。

---

## 共分散行列と多変量正規分布

### 平均ベクトル

2変量の中心は、平均を並べたベクトルで表す。

$$
\boldsymbol{\mu}=\begin{pmatrix}\mu_1\\\mu_2\end{pmatrix}.
$$

$d$ 個の変数なら $\boldsymbol{\mu}=(\mu_1,\ldots,\mu_d)^\mathrm{T}$ となる。

### 共分散行列

2変量の共分散行列は

$$
\boldsymbol{\Sigma}=\begin{pmatrix}
\sigma_1^2 & \operatorname{Cov}(X_1,X_2)\\
\operatorname{Cov}(X_2,X_1) & \sigma_2^2
\end{pmatrix}.
$$

対角成分は各変数の分散、非対角成分は変数間の共分散である。共分散は対称なので $\operatorname{Cov}(X_1,X_2)=\operatorname{Cov}(X_2,X_1)$ であり、$\boldsymbol{\Sigma}$ も対称行列になる。

**ちなみに4変量の場合**、$i\ne j$ に対して $\sigma_{ij}=\operatorname{Cov}(X_i,X_j)$ と置くと、共分散行列は次の $4\times4$ 行列になる。

$$
\boldsymbol{\Sigma}=\begin{pmatrix}
\sigma_1^2 & \sigma_{12} & \sigma_{13} & \sigma_{14}\\
\sigma_{12} & \sigma_2^2 & \sigma_{23} & \sigma_{24}\\
\sigma_{13} & \sigma_{23} & \sigma_3^2 & \sigma_{34}\\
\sigma_{14} & \sigma_{24} & \sigma_{34} & \sigma_4^2
\end{pmatrix}.
$$

対角成分は4変数それぞれの分散、非対角成分は変数どうしの共分散を表す。左右対称なので、異なる変数の組の共分散は6組である。

### 多変量正規分布

$d$ 次元ベクトル $\boldsymbol{X}$ の多変量正規分布を

$$
\boldsymbol{X}\sim\mathcal{N}(\boldsymbol{\mu},\boldsymbol{\Sigma})
$$

と書く。ここでは共分散行列 $\boldsymbol{\Sigma}$ が正定値である場合を考える。正定値なので、逆行列 $\boldsymbol{\Sigma}^{-1}$ が存在する。正定値行列の定義や性質については、[正定値行列・半正定値行列の定義と性質（数学の景色）](https://mathlandscape.com/positive-definite-matrix/)を参照。また、平均ベクトルとの偏差 $\boldsymbol{x}-\boldsymbol{\mu}$ がゼロベクトルでなければ、次の二次形式は正の値をとる。

$$
(\boldsymbol{x}-\boldsymbol{\mu})^\mathrm{T}\boldsymbol{\Sigma}^{-1}(\boldsymbol{x}-\boldsymbol{\mu})
$$

この二次形式をマハラノビス距離の二乗として使える。また、行列式 $|\boldsymbol{\Sigma}|$ も正なので、密度の面積を1にする係数を定められる。正定値でない特異な場合は逆行列が存在せず、以下の通常の $d$ 次元密度の式はそのまま使えない。

この条件のもとで、密度関数は

$$
p(\boldsymbol{x})=\frac{1}{(2\pi)^{d/2}|\boldsymbol{\Sigma}|^{1/2}}
\exp\left[-\frac12(\boldsymbol{x}-\boldsymbol{\mu})^\mathrm{T}\boldsymbol{\Sigma}^{-1}(\boldsymbol{x}-\boldsymbol{\mu})\right].
$$

指数関数の括弧内にある二次形式は、マハラノビス距離の二乗である。

$$
d_M^2(\boldsymbol{x},\boldsymbol{\mu})=(\boldsymbol{x}-\boldsymbol{\mu})^\mathrm{T}\boldsymbol{\Sigma}^{-1}(\boldsymbol{x}-\boldsymbol{\mu})
$$

したがって指数部は $-\frac{1}{2}d_M^2(\boldsymbol{x},\boldsymbol{\mu})$ となる。マハラノビス距離は、平均からの差を共分散行列に合わせて測る距離であり、変数ごとの尺度や変数間の相関を考慮する。共分散があると等密度線は傾き、各方向の分散が異なると伸び方も変わる。直感的な説明は、[マハラノビス距離とは（スタビジ）](https://toukei-lab.com/mahalanobis)も参照。

### 1変量正規分布との対応

$d=1$ とすれば、$\boldsymbol{\mu}\to\mu$, $\boldsymbol{\Sigma}\to\sigma^2$ となり、

$$
(\boldsymbol{x}-\boldsymbol{\mu})^\mathrm{T}\boldsymbol{\Sigma}^{-1}(\boldsymbol{x}-\boldsymbol{\mu})
\longrightarrow \frac{(x-\mu)^2}{\sigma^2}.
$$

行列式は $|\boldsymbol{\Sigma}|=\sigma^2$、前の係数は $\frac{1}{\sqrt{2\pi\sigma^2}}$ になる。つまり多変量正規分布は、1変量正規分布をベクトル・行列へ拡張した形である。

### 共分散行列を変えて観察する

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja

rng = np.random.default_rng(23)
covariances = [
    [[1, 0], [0, 1]],
    [[1, 0.8], [0.8, 1]],
    [[1, -0.8], [-0.8, 1]],
    [[4, 0], [0, 1]],
]
titles = ["無相関", "正の共分散", "負の共分散", "第1方向の分散が大きい"]
fig, axes = plt.subplots(1, 4, figsize=(12, 3))
for ax, cov, title in zip(axes, covariances, titles):
    points = rng.multivariate_normal([0, 0], cov, size=900)
    ax.scatter(points[:, 0], points[:, 1], s=4, alpha=0.4)
    ax.set_title(title)
    ax.set_aspect("equal")
    ax.grid(alpha=0.2)
plt.tight_layout()
plt.show()
```

図を見て、対角成分と非対角成分のどちらが点群の広がり・傾きを変えたか説明してみよう。

---

## 正規性

### 正規性とは

正規分布というモデルを学んだからといって、実データが必ずその分布に従うわけではない。データの分布が正規分布とみなせるか、という性質を**正規性**と呼ぶ。

後の推定・検定では、手法がどのような前提のもとで使えるかを確認することがある。第1回では「統計手法を使う前に、その前提を確かめる」という考え方を持つ。

### ヒストグラム

ヒストグラムは分布の形をつかむ最初の方法である。しかし、階級幅や標本数によって見え方が変わるため、山型に見えるだけで正規分布と断定してはいけない。

### Q-Qプロット

Q-Qプロットは、データの分位点と、正規分布を仮定した場合の理論分位点を比べる。点が概ね直線に沿えば正規分布との整合性が視覚的に高く、曲がりや端でのずれがあれば、歪みや裾の違いなどを疑う。

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib_fontja
from scipy import stats

rng = np.random.default_rng(31)
normal_data = rng.normal(size=600)
uniform_data = rng.uniform(size=600)

fig, axes = plt.subplots(1, 2, figsize=(9, 4))
stats.probplot(normal_data, dist="norm", plot=axes[0])
axes[0].set_title("正規分布からの標本")
stats.probplot(uniform_data, dist="norm", plot=axes[1])
axes[1].set_title("一様分布からの標本")
plt.tight_layout()
plt.show()
```

Q-Qプロットも視覚的な診断であり、これだけで正規性が証明されるわけではない。標本数や分析目的も合わせて判断する。正規性の検定方法の詳細は本回の必須範囲に含めない。

Gaussian Naive BayesによるIris分類は、[発展ノート：Gaussian Naive Bayes](pymc-iris.ipynb)にまとめた。

---

## まとめ

| | 1変量 | 多変量 |
|---|---|---|
| データ | $x$ | $\boldsymbol{x}$ |
| 中心 | $\mu$ | $\boldsymbol{\mu}$ |
| ばらつき | $\sigma^2$ | $\boldsymbol{\Sigma}$ |
| 分布 | 正規分布 | 多変量正規分布 |

### 持ち帰る3点

1. 正規分布は平均 $\mu$ と分散 $\sigma^2$ で中心と広がりを表す。
2. 多変量では分散が共分散行列 $\boldsymbol{\Sigma}$ に拡張され、変数ごとのばらつきと変数間の関係を表す。
3. 実データを正規分布として扱ってよいかという正規性を確認する。

### 第2回「相関分析」へ

共分散は2変数の共変動を表すが、値は測定単位や尺度に左右される。標準化によって尺度に依存しない関係の強さを表せないだろうか。次回は相関係数 $r$（$-1\leq r\leq1$）を学ぶ。

```text
共分散 → 標準化 → 相関係数 → 相関分析
```

### Exit Ticket

1. 正規分布の $\mu$ は何を表すか。
2. $\sigma^2$ を大きくすると、分布の形はどう変わるか。
3. 共分散行列の対角成分は何を表すか。
4. 共分散が正のとき、散布図にはどのような傾向が現れるか。
5. 正規性とは何か。またQ-Qプロットでは何を確認するか。
6. 今日、理解しにくかった点は何か。

---

## 演習ノートブック

図を自分で生成し、平均・標準偏差・共分散行列・正規性を確かめる。演習ノートブックは独立ページで開ける。
