---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://scikit-learn.org/stable/modules/clustering.html
  - https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html
  - https://theory.stanford.edu/~serie/papers/kMeansPP-soda.pdf
  - https://scikit-learn.org/stable/modules/mixture.html
tags:
  - ai
---

K 均值聚类是在给定簇数 $k$ 下，寻找把样本划成 $k$ 个簇的一种划分，使簇内平方和（SSE，scikit-learn 里叫 inertia）最小。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

## 目标函数

设样本集 $X=\{x_1,\dots,x_N\}$，簇中心集合 $C=\{\mu_1,\dots,\mu_k\}$，K 均值要解的是：

$$
\min_{C}\ \mathrm{SSE}(C)=\sum_{i=1}^{N}\ \min_{\mu_j\in C}\ \lVert x_i-\mu_j\rVert_2^2
$$

注意这个式子是「每个点只记离它最近的那个中心的距离平方」，所以 SSE 同时是「划分质量」和「中心位置」的函数。scikit-learn 用户指南给的正是这个形式，并称它为 inertia 或 within-cluster sum-of-squares。

两个性质决定了后面所有讨论：

- **它不是凸问题**。对「划分」这层离散变量而言，可行解个数是第二类 Stirling 数级别（把 $N$ 个点分成 $k$ 个非空组的方案数），规模稍大就不可能穷举。
- **它没有归一化**。inertia 的绝对数值随样本量、量纲、维度一起变，只能在同一数据集内比较不同 $k$ 或不同初始化，不能跨数据集比。

正因为非凸，K 均值不「求解」而是「下降」：从一个初始中心出发，反复执行两步，使 SSE 单调不增，最终停在某个局部极小。

## Lloyd 算法的两步交替

**分配步（assignment）**：固定中心 $\mu_j$，把每个点分给最近的中心，

$$
c_i \leftarrow \arg\min_{j}\ \lVert x_i-\mu_j\rVert_2^2
$$

**更新步（update）**：固定分配 $c_i$，把每个中心换成该簇的样本均值，

$$
\mu_j \leftarrow \frac{1}{|C_j|}\sum_{i:\,c_i=j} x_i
$$

两步交替直到分配不再变化（或达到 `max_iter`、SSE 变化小于 `tol`）。

### 为什么每一步都不升高目标函数

这是判断「K 均值实现对不对」的核心，也是面试常问的点：

1. **分配步不会升高 SSE**。固定中心后，SSE 是 $N$ 个独立项之和，第 $i$ 项只依赖 $c_i$。把 $c_i$ 换成让该项最小的那个 $j$，只会让这一项变小或不变，其余项不动，所以总和不会变大。
2. **更新步不会升高 SSE**。固定分配后，SSE 拆成 $k$ 个互不耦合的簇内平方和 $\sum_{i\in C_j}\lVert x_i-\mu_j\rVert^2$。对 $\mu_j$ 求梯度：

$$
\nabla_{\mu_j}\sum_{i\in C_j}\lVert x_i-\mu_j\rVert^2=-2\sum_{i\in C_j}(x_i-\mu_j)=0\ \Longrightarrow\ \mu_j=\frac{1}{|C_j|}\sum_{i\in C_j}x_i
$$

   二阶导是 $2|C_j|I$，正定，所以均值点是该簇平方和的**全局**极小点，换过去一定不升。

两步都不升，SSE 单调不增且有下界 0，加上「划分方案数有限」，算法必然在有限步内停住。但停住只说明到达局部极小：某个点稍微挪到隔壁簇、或某个中心稍微偏离均值，SSE 都会上升。

**判据**：如果你的实现里 SSE 出现上升，一定是这两步之一写错了——典型错误是更新中心时把「空簇」的旧中心保留成了某个远离数据的位置，或者距离用了非平方的欧氏距离却拿平方和比较。

## k 的选择

### 肘部法

把 $k=1,2,\dots$ 的 inertia 画出来，找曲线由陡变缓的拐点。机制上它成立是因为增加一个中心必然降低 SSE（最坏情况新中心放在某个点上，SSE 不升），所以曲线单调下降，肘部是「新增中心带来的边际收益开始递减」的位置。

**判据与局限**：肘部法没有客观定义，拐点常常很模糊；当簇本身重叠或数据维度较高时，曲线近似平滑下降，读不出拐点。它只适合作为初筛，不应当作结论。

### 轮廓系数

对单个样本 $i$：

- $a(i)$：$i$ 到**同簇**其他样本的平均距离；
- $b(i)$：$i$ 到**最近的其他簇**中所有样本的平均距离。

$$
s(i)=\frac{b(i)-a(i)}{\max\{a(i),\,b(i)\}},\qquad S=\frac{1}{N}\sum_{i=1}^{N}s(i)
$$

$s(i)\in[-1,1]$：接近 1 表示「离本簇近、离邻簇远」，接近 0 表示处在两簇边界，负值表示可能被分错了簇。取 $S$ 最大的 $k$。

**判据**：轮廓系数对凸形簇友好，对长条形、密度差异大的簇会低估；它是 $O(N^2)$ 的成对距离计算，$N$ 大时要抽样。

## 初始化为什么敏感，以及 k-means++

K 均值的目标函数非凸，不同的初始中心会收敛到不同的局部极小。scikit-learn 文档的原话是「K-means will always converge, however this may be to a local minimum. This is highly dependent on the initialization of the centroids」，官方给的对策就是 k-means++。

k-means++ 的机制是**按距离平方加权抽样**：第一个中心从数据里均匀随机选；之后每个新中心按概率

$$
P(x)=\frac{D(x)^2}{\sum_{x'}D(x')^2}
$$

抽样，其中 $D(x)$ 是 $x$ 到已选中心集合的最短距离。这样离已有中心越远的点越可能被选成新中心，避免了「两个初始中心落在同一个真实簇里」这种最常见的坏初始化。Arthur 与 Vassilvitskii 的原始论文证明该初始化在期望意义下给出 $O(\log k)$ 量级的近似保证（常数因子为 $8(\ln k+2)$）。

**工程判据**：即便用了 k-means++，单次运行也只保证局部最优，所以还要多次重启取 inertia 最小的一次。scikit-learn 的 `n_init` 就是这个「不同随机种子跑几次」的参数，默认值随版本变化（早期默认 `10`，当前稳定版为 `'auto'`，具体起版待核），`KMeans` 的完整签名是：

```python
KMeans(n_clusters=8, *, init='k-means++', n_init='auto', max_iter=300,
       tol=0.0001, verbose=0, random_state=None, copy_x=True, algorithm='lloyd')
```

**易错点**：把 `random_state` 固定后重跑得到完全相同的结果，于是误以为「算法稳定」；实际上这只说明同一个局部极小被重复命中，换一个种子结果可能差很多。

## 为什么必须标准化

K 均值用的是欧氏距离，而欧氏距离**直接相加各维度的差值平方**，量纲大的维度会支配整个距离。

算个具体的：两个样本年龄差 5 岁、年收入差 5000 元，若原样计算，距离平方是 $5^2+5000^2=25{,}000{,}025$，年龄维度的贡献占 $0.0001\%$——聚类结果等价于「只按收入分」。标准化（减去均值、除以标准差）之后两维各自变成无量纲的 z 分数，才谈得上「两维同等重要」。

**判据**：只要各特征的量纲或取值范围不同（金额 vs 比例、计数 vs 比率），就必须标准化；如果所有特征本来就同量纲同尺度（比如都是 0–1 的概率），可以不标准化。用 `StandardScaler` 而不是 `MinMaxScaler` 更常见，因为 K 均值对离群点敏感，标准化不改变这一点，但至少让各维的敏感度可比。

**易错点**：标准化要在**训练集**上 fit，再 transform 全量；先对全量数据 fit 会引入轻微的泄漏——在纯无监督场景影响小，但在「先聚类再当特征喂给下游模型」的流程里会变成真实的泄漏。

## 硬聚类与软聚类

K 均值输出的是**硬划分**：每个点属于且仅属于一个簇。高斯混合模型（GMM）输出**软划分**：每个点对每个簇有一个后验概率（responsibility），再做 EM 迭代。

两者的关系可以精确表述：若把 GMM 中所有簇的协方差都取成同方差的各向同性矩阵 $\sigma^2 I$，并令 $\sigma\to 0$，后验概率会退化成「最近中心的那个簇为 1、其余为 0」，EM 的 E 步退化成 K 均值的分配步，M 步退化成均值更新。所以 **K 均值是「等方差各向同性 GMM 在方差趋于 0 时的硬分配极限」**。

**判据**：需要「不确定性」或「重叠簇」时用 GMM；需要「每个点必须归属唯一簇」且数据量大时用 K 均值（K 均值每轮是 $O(Nkd)$，GMM 每轮还要算协方差与行列式，常数更大）。

## 适用前提与失效场景

| 前提 | 含义 | 违反时的表现 |
|---|---|---|
| 簇是凸的、各向同性 | 每簇大致是球状团块 | 长条形、环形（如 two-moons）被切开 |
| 各簇大小相近 | 簇的样本量与半径可比 | 大簇被小簇「抢走」边界，或大簇被拆成两个 |
| 各簇密度相近 | 点分布疏密一致 | 稀疏簇的中心被密集簇吸走 |
| 用均值代表簇 | 均值有意义 | 类别型特征（无序）无法取均值 |
| 对离群点稳健 | 均值受极值影响 | 单个远点能把一个中心拽离整个簇 |

对应的替代方案：非凸形状用谱聚类或 DBSCAN；簇大小差异大用 GMM 或层次聚类；有离群点先做剔除或改用基于密度的算法；类别型特征用 k-modes 或先做嵌入。

**高维的额外问题**：维度升高时欧氏距离趋于集中（各点间距离差异变小），inertia 的区分度下降，官方文档明确提到高维下欧氏距离会「inflated」，这也是 K 均值在高维稀疏数据上效果差的原因之一。实践中先做 PCA 降维再聚类。

## 关联

- [[机器学习]]：K 均值是无监督学习里最基础的划分式聚类，属于「无标签、按相似度分组」这一支。
- [[深度学习]]：深度聚类（如自编码器 + K 均值）把特征学习与聚类交替进行，本页的 SSE 与 Lloyd 迭代是其中的固定组件。
- [[2026-10-27-机器学习]]：本机笔记只记了算法步骤，本页补的是目标函数、单调性证明与失效判据。
- [[人工智能]]：聚类是「无监督发现结构」的代表任务，与分类/回归的区别在于没有标签。
- [[神经网络]]：向量量化（VQ-VAE 的码本更新）本质上就是一次 K 均值式的中心更新。

## 来源

- scikit-learn, Clustering — 2.3.2 K-means（inertia 定义与公式、非凸/各向同性假设、k-means++ 说明）：https://scikit-learn.org/stable/modules/clustering.html ，访问日期 2026-10-27。
- scikit-learn, Clustering — 2.3.11.5 Silhouette Coefficient（$a$、$b$ 的定义与 $s=(b-a)/\max(a,b)$）：同上页面，访问日期 2026-10-27。
- scikit-learn, `sklearn.cluster.KMeans` API（`init='k-means++'`、`n_init`、`max_iter=300`、`tol=1e-4`、`algorithm='lloyd'` 等默认值）：https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html ，访问日期 2026-10-27。
- David Arthur, Sergei Vassilvitskii, *k-means++: The Advantages of Careful Seeding*, SODA 2007（按 $D^2$ 加权抽样与期望近似保证）：https://theory.stanford.edu/~serie/papers/kMeansPP-soda.pdf ，访问日期 2026-10-27。
- scikit-learn, Gaussian Mixture Models（软划分、与 K 均值的关系）：https://scikit-learn.org/stable/modules/mixture.html ，访问日期 2026-10-27。

> 说明：Lloyd 迭代的单调性证明与「K 均值是等方差 GMM 的硬分配极限」属于教科书级推导，本页自行给出完整式子以供验算；`n_init` 默认值的确切起版、以及轮廓系数原始论文（Rousseeuw, 1987）的页码未逐一核对，标「待核」的部分以官方 API 文档为准。
