## 1. 课程定位与目标
- 定位：人工智能/数据挖掘/机器学习方向的基础课程。
- 目标：
  - 了解机器学习定义、应用场景。
  - 掌握机器学习基础环境安装与使用。
  - 掌握 Matplotlib、Numpy、Pandas 对数据进行展示、分析。

## 2. 人工智能概述
### 2.1 应用场景
网络安全、电子商务、计算模拟、交通模式、社交网络、传感器网络等。

### 2.2 人工智能必备三要素
- 数据
- 算法
- 计算力：CPU、GPU、TPU
  - CPU：适合 IO 密集型任务。
  - GPU：适合计算密集型、易于并行的程序，SIMD 架构。

### 2.3 AI、机器学习、深度学习关系
- 机器学习是人工智能的一个实现途径。
- 深度学习是机器学习的一个方法发展而来。

## 3. 人工智能发展历程
### 3.1 起源
- 图灵测试：测试者与被测试者隔开，通过键盘提问；多次测试中超过 30% 测试者不能确定是人还是机器，则机器通过测试。
- 达特茅斯会议：1956 年，约翰·麦卡锡、马文·闵斯基、克劳德·香农、艾伦·纽厄尔、赫伯特·西蒙等讨论“用机器模仿人类学习及其他智能”，提出“人工智能”，1956 年成为人工智能元年。

### 3.2 六个阶段
1. 起步发展期：1956—20世纪60年代初，机器定理证明、跳棋程序。
2. 反思发展期：20世纪60年代—70年代初，目标不切实际，进入低谷。
3. 应用发展期：20世纪70年代初—80年代中，专家系统在医疗、化学、地质等领域成功。
4. 低速发展期：20世纪80年代中—90年代中，专家系统问题暴露。
5. 稳步发展期：20世纪90年代中—2010年，互联网技术推动；IBM 深蓝战胜卡斯帕罗夫；IBM 提出“智慧地球”。
6. 蓬勃发展期：2011 年至今，大数据、云计算、物联网、GPU、深度神经网络推动。

## 4. 人工智能主要分支
- 计算机视觉 CV：图像形成、处理、提取、三维推理；物体检测、人脸识别。
- 语音识别：语音转文本、TTS；受声纹识别、鸡尾酒会效应影响。
- 文本挖掘/分类：句法分析、情绪分析、垃圾信息检测。
- 机器翻译：源语言到目标语言；神经机器翻译进展明显。
- 机器人：固定机器人、移动机器人；工业机器人、软体机器人。

## 5. 机器学习工作流程
1. 获取数据
2. 数据基本处理：缺失值、异常值等
3. 特征工程：特征提取、特征预处理、特征降维
4. 机器学习/模型训练
5. 模型评估
6. 达到要求上线服务；否则重新迭代

### 5.1 数据集
- 一行 = 一个样本；一列 = 一个特征。
- 数据类型：
  - 特征值 + 目标值（目标值连续/离散）
  - 只有特征值，无目标值
- 数据分割：训练集 70%~80%，测试集 20%~30%。

### 5.2 特征工程
- 定义：用专业背景知识和技巧处理数据，使特征在算法上发挥更好作用。
- 意义：直接影响机器学习效果。
- 包含：
  - 特征提取：文本/图像转数字特征。
  - 特征预处理：转换函数使特征更适合模型。
  - 特征降维：降低随机变量个数，得到不相关主变量。

## 6. 机器学习算法分类
- 监督学习：输入有特征值和目标值。
  - 回归：输出连续值，如房价预测。
  - 分类：输出有限离散值，如肿瘤良恶性。
- 无监督学习：只有特征值，无目标值；根据相似性划分。
- 半监督学习：部分有标签、部分无标签。
- 强化学习：自动决策、连续决策；agent、action、reward、environment、observation；目标是获得最多累计奖励。
- 监督学习与强化学习对比：
  - 监督学习反馈映射输出关系，反馈及时，输入独立同分布。
  - 强化学习反馈 reward，结果反馈有延时，输入随行为变化。

## 7. 模型评估
### 7.1 分类模型评估
- 准确率：预测正确数 / 样本总数。
- 其他：精确率、召回率、F1-score、AUC。

### 7.2 回归模型评估
- RMSE 均方根误差：
  \[
  RMSE=\sqrt{\frac{\sum_{i=1}^{n}(p_i-a_i)^2}{n}}
  \]
- 其他：RSE、MAE、RAE。

### 7.3 拟合
- 欠拟合：模型学习太粗糙，训练集特征关系都没学出来。
- 过拟合：训练样本表现过优，测试集表现不佳。

## 8. Azure 机器学习实验
- Azure Machine Learning 是微软公有云 Azure 上的 Web 机器学习服务。
- 目的：简化机器学习过程，让开发人员、业务分析师、数据科学家便捷应用。
- 实验目的：了解从数据到建模并最终评估预测的流程。

## 9. 深度学习简介
- 深度学习是机器学习分支，是一类算法集合。
- 神经网络：受生物神经元启发。
- 各层负责内容：
  - 第1层：颜色及简单纹理。
  - 第2层：更细化纹理，布纹、刻纹、叶纹。
  - 第3层：烛光、高光、萤火、鸡蛋黄等。
  - 第4层：萌狗脸、宠物形貌、圆柱体、七星瓢虫等。
  - 第5层：花、黑眼圈动物、鸟、键盘、原型屋顶等。
- 演示：http://playground.tensorflow.org

## 10. 环境安装与 Jupyter Notebook
- 常用库：Matplotlib、Numpy、Pandas、tables、jupyter。
- 创建虚拟环境：`mkvirtualenv ai`
- 安装：`pip3 install -r requirements.txt`
- Jupyter Notebook：
  - 原名 IPython Notebook，开源 Web 应用。
  - 名字源自 Julia、Python、R。
  - 文件格式 `.ipynb`，JSON 文档格式。
  - 适合数据探索：代码、叙事文本、可视化整合。
- 启动：`jupyter notebook`，默认 `http://localhost:8888`。
- cell 两种模式：
  - 编辑模式：Enter。
  - 命令模式：Esc。
- 常用快捷键：
  - `Shift+Enter`：执行本单元并跳下一单元。
  - `Ctrl+Enter`：执行本单元并留在本单元。
  - 命令模式：`Y` 代码，`M` Markdown，`A` 上方添加，`B` 下方添加，双击删除 cell，`Z` 回退，`L` 行号。
  - 编辑模式：Tab 补全，`Ctrl+/` 注释。
- Markdown：标题 `#`，缩进 `-`。

## 11. Matplotlib
### 11.1 作用
专门用于开发 2D/3D 图表，渐进、交互式数据可视化；可视化是数据挖掘关键辅助工具。

### 11.2 绘图流程
1. 创建画布：`plt.figure(figsize=(10,10), dpi=100)`
2. 绘制图像：`plt.plot(x, y)`
3. 显示图像：`plt.show()`

### 11.3 折线图完善
- 设置 x/y 刻度：`plt.xticks()`、`plt.yticks()`
- 网格：`plt.grid(True, linestyle="--", alpha=0.5)`
- 描述信息：`plt.xlabel()`、`plt.ylabel()`、`plt.title()`
- 保存：`plt.savefig("test.png")`；注意 `plt.show()` 会释放 figure 资源，保存应在 show 前。
- 中文显示：
  ```python
  from pylab import mpl
  mpl.rcParams["font.sans-serif"] = ["SimHei"]
  mpl.rcParams["axes.unicode_minus"] = False
  ```
- 多次 plot：`plt.plot(x, y1, label="上海")`、`plt.plot(x, y2, color="r", linestyle="--", label="北京")`
- 图例：`plt.legend(loc="best")`
- 多坐标系：`fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(20,8), dpi=100)`
  - 面向对象：`axes[0].plot()`、`axes[0].set_xticks()`、`axes[0].set_title()` 等。

### 11.4 常见图形
- 折线图：变化趋势，`plt.plot(x,y)`
- 散点图：变量关联、离群点，`plt.scatter(x,y)`
- 柱状图：离散数据对比，`plt.bar(x, width, align='center')`
- 直方图：连续数据分布，`plt.hist(x, bins=None)`
- 饼图：分类占比，`plt.pie(x, labels, autopct, colors)`

## 12. Numpy
### 12.1 优势
- 开源 Python 科学计算库，快速处理任意维度数组。
- ndarray 对象：N 维数组，相同类型元素集合。
- 优势：
  - 内存块风格：数据地址连续，批量操作快。
  - 支持并行化/向量化运算。
  - 底层 C 语言，解除 GIL，效率高。

### 12.2 ndarray 属性
- `shape`：维度元组
- `ndim`：维数
- `size`：元素数量
- `itemsize`：一个元素长度（字节）
- `dtype`：元素类型

### 12.3 创建数组
- `np.ones(shape, dtype)`、`np.ones_like(a)`
- `np.zeros(shape, dtype)`、`np.zeros_like(a)`
- `np.array(object, dtype)` 深拷贝；`np.asarray(a, dtype)` 浅拷贝
- `np.linspace(start, stop, num, endpoint)` 等差指定数量
- `np.arange(start, stop, step, dtype)` 等差指定步长
- `np.logspace(start, stop, num)` 等比数列
- 随机：
  - 正态：`np.random.randn()`、`np.random.normal(loc=0, scale=1, size=None)`、`np.random.standard_normal(size=None)`
  - 均匀：`np.random.rand()`、`np.random.uniform(low, high, size)`、`np.random.randint(low, high, size, dtype)`

### 12.4 索引切片
- 直接索引、切片，先行后列。
- 二维：`stock_change[0, 0:3]`
- 三维：`a1[0,1,2]` 等。

### 12.5 形状修改
- `ndarray.reshape(shape, order)`：返回视图，行列不互换。
- `ndarray.resize(new_shape)`：修改本身。
- `ndarray.T`：转置。

### 12.6 类型修改与去重
- `ndarray.astype(type)`
- `ndarray.tostring()` / `tobytes()`
- `np.unique()` 去重。

### 12.7 逻辑运算与统计
- 布尔索引：`test_score[test_score>60]=1`
- `np.all()`、`np.any()`
- `np.where(condition, x, y)`
- 复合逻辑：`np.logical_and`、`np.logical_or`
- 统计：`np.min/max/median/mean/std/var(a, axis)`
- 下标：`np.argmax/argmin(temp, axis)`
- axis=0 代表列，axis=1 代表行。

### 12.8 数组间运算与广播
- 数组与数：逐元素运算。
- 数组与数组：形状相同或满足广播机制。
- 广播条件：
  - 后缘维度轴长相等；
  - 或其中一方长度为 1。
- 示例：`(4,1)+(3,) -> (4,3)`

### 12.9 矩阵
- 矩阵是二维数组；向量是特殊矩阵（列向量）。
- 加法：行列数相等。
- 标量乘法：每个元素乘。
- 矩阵乘法：`(M行,N列)*(N行,L列)=(M行,L列)`
- 性质：不满足交换律，满足结合律；单位矩阵 I/E；逆矩阵；转置。
- API：`np.matmul`、`np.dot`；`np.matmul` 禁止矩阵与标量乘法；矢量内积两者无区别。

## 13. Pandas
### 13.1 介绍
- 2008 年 Wes McKinney 开发，基于 Numpy，基于 Matplotlib 画图，独特数组结构。
- 优势：增强图表可读性、便捷数据处理、读取文件方便、封装画图和计算。

### 13.2 数据结构
- Series：一维，数据和索引。
  - 创建：`pd.Series(data=None, index=None, dtype=None)`
  - 属性：`index`、`values`
- DataFrame：二维表格，行索引 index，列索引 columns。
  - 创建：`pd.DataFrame(data=None, index=None, columns=None)`
  - 属性：`shape`、`index`、`columns`、`values`、`T`、`head()`、`tail()`
  - 索引修改：整体修改 `data.index = stu`；`reset_index(drop=False)`；`set_index(keys, drop=True)`
- MultiIndex：多级/层次化索引，三维结构。
  - `pd.MultiIndex.from_arrays(arrays, names=[])`
- Panel：老版本三维结构，0.20.0 起弃用，推荐 DataFrame + MultiIndex。

### 13.3 基本数据操作
- 读取：`pd.read_csv()`
- 索引：
  - 直接索引先列后行：`data['open']['2018-02-27']`
  - `loc`：先行后列，按名字。
  - `iloc`：先行后列，按下标。
  - `ix` 已弃用。
- 赋值：`data['close']=1` 或 `data.close=1`
- 排序：
  - `df.sort_values(by, ascending=True)`
  - `df.sort_index()`
  - `series.sort_values()`、`series.sort_index()`

### 13.4 DataFrame 运算
- 算术：`add()`、`sub()`
- 逻辑：`data[data["open"]>23]`
- 函数：`query(expr)`、`isin(values)`
- 统计：`describe()`、`max/min/mean/std/var/median/mode/idxmax/idxmin/cumsum`
- 自定义：`apply(func, axis=0)`

### 13.5 Pandas 画图
- `DataFrame.plot(kind="line")`
- kind：line、bar、barh、hist、pie、scatter。
- `Series.plot()`

### 13.6 文件读取与存储
- CSV：`read_csv`、`to_csv`
  - `sep`、`usecols`、`columns`、`header`、`index`、`mode`
- HDF5：`read_hdf`、`to_hdf`
  - 需安装 tables：`pip install tables`
  - 支持压缩 blosc，跨平台，节省空间。
- JSON：`read_json`、`to_json`
  - `orient`：split、records、index、columns、values
  - `lines=True` 每行一个对象。

### 13.7 缺失值处理
- 判断：`pd.isnull(df)`、`pd.notnull(df)`
- 删除：`dropna(axis="rows")`，不修改原数据，需接收返回值。
- 填充：`fillna(value, inplace=True)`
- 替换：`replace(to_replace, value)`
- 非 NaN 标记如 `?`：先 `replace(to_replace="?", value=np.nan)`，再处理。

### 13.8 数据离散化
- 目的：简化数据结构，减少连续属性值个数。
- 分组：
  - `pd.cut(data, bins)` 自定义区间
  - `pd.qcut(data, q)` 大致等频分组
  - `value_counts()` 统计分组次数
- One-hot 编码：`pd.get_dummies(data, prefix=None)`

### 13.9 合并
- `pd.concat([data1,data2], axis=1)`：按行/列合并。
- `pd.merge(left, right, how='left', on=['key1','key2'])`
  - how：left、right、outer、inner。

### 13.10 交叉表与透视表
- 交叉表：`pd.crosstab(value1, value2)`，计算分组个数/频率。
- 透视表：`data.pivot_table([], index=[])`，指定列作为行/列索引并聚集。
- 案例：股票涨跌与星期几关系：
  - `date = pd.to_datetime(data.index).weekday`
  - `data['posi_neg'] = np.where(data['p_change']>0,1,0)`
  - `count = pd.crosstab(data['week'], data['posi_neg'])`
  - 求比例：`count.div(count.sum(axis=1).astype(np.float32), axis=0)`
  - `pro.plot(kind='bar', stacked=True)`
  - 透视表：`data.pivot_table(['posi_neg'], index='week')`

### 13.11 分组与聚合
- `groupby(['color'])['price'].mean()`
- `groupby(['color'], as_index=False)['price'].mean()`
- 星巴克案例：
  - `starbucks.groupby(['Country']).count()`
  - `starbucks.groupby(['Country','State/Province']).count()`

### 13.12 电影案例
- 评分平均分：`df["Rating"].mean()`
- 导演人数：`df["Director"].unique().shape[0]`
- 分布：`df["Rating"].plot(kind='hist')`
- 电影分类统计：
  - 分割 genre，得到分类列表。
  - 创建全 0 DataFrame。
  - 遍历电影，将出现分类置 1。
  - 求和排序画图。

## 14. 完整机器学习项目流程
1. 抽象成数学问题：分类/回归/聚类。
2. 获取数据：代表性、数据偏斜、量级评估。
3. 特征预处理与特征选择：归一化、离散化、因子化、缺失值、去共线性；相关系数、卡方检验、互信息、条件熵、后验概率、逻辑回归权重。
4. 训练模型与调优：调整超参数。
5. 模型诊断：过拟合/欠拟合、交叉验证、学习曲线、误差分析。
6. 模型融合：前端特征清洗/采样，后端模型融合。
7. 上线运行：准确率、误差、时间复杂度、空间复杂度、稳定性。

## 15. 独立同分布 IID
- 独立：每次抽样互不影响。
- 同分布：每次抽样服从同一分布。
- 独立同分布：每次抽样独立且同分布。
- 机器学习重要假设：训练数据和测试数据满足相同分布，是模型在测试集获得好效果的基本保障。
- 目前并非所有问题都要求独立同分布。