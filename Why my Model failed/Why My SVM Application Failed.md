# 金融支持向量机应用为何失效（Why My SVM Application Failed）

> **英文摘要（English summary）：** Support vector machines can produce a globally optimal solution to the optimization problem they are given, yet still generalize poorly when labels, feature scales, validation, or kernel choices are wrong. This report derives the soft-margin objective and its dual interpretation, explains the roles of the penalty C and kernel bandwidth, and covers class imbalance, probability calibration, support-vector diagnostics, and time-aware tuning. A-share and U.S. market constraints are included.

**案例等级**：方法诊断模板；任何示例表现均不能替代可复现的时间外证据与账户账本。

## 1. SVM 的保证边界

支持向量机（Support Vector Machine, SVM）通过最大化分类间隔来构造决策边界。常见的软间隔 SVM 优化问题是凸的，因此在给定数据、特征变换、核函数和超参数后，算法可以求到全局最优解。

这个数学保证不代表训练数据无泄漏、特征尺度正确、超参数选择无偏或分类边界具有未来价值。若输入几何或标签错误，SVM 会精确求解错误的问题。

## 2. 从最大间隔到软间隔目标

二分类标签 \(y_i\in\{-1,+1\}\)。线性间隔分类器为 \(f(x)=w^\top x+b\)。硬间隔形式要求：

\[
y_i(w^\top x_i+b)\ge1
\]

并最小化 \(\frac12\|w\|_2^2\)。金融数据通常不可完全分离，因此引入松弛变量 \(\xi_i\ge0\)：

\[
\min_{w,b,\xi}\quad \frac12\|w\|_2^2+C\sum_i \xi_i,\qquad
y_i(w^\top x_i+b)\ge1-\xi_i.
\]

\(C\) 权衡较宽的间隔与训练违例惩罚。\(C\) 大时更重视训练误分类，通常降低偏差但可能提高方差；\(C\) 小时允许更多违例并加强正则化。不同软件对损失归一化和 \(C\) 的标度约定可能不同。

拉格朗日对偶形式依赖样本内积：

\[
\max_{\alpha}\quad
\sum_i\alpha_i-\frac12\sum_{i,j}\alpha_i\alpha_jy_iy_jK(x_i,x_j),
\]

约束 \(0\le\alpha_i\le C\)、\(\sum_i\alpha_i y_i=0\)。只有支持向量（support vectors）对应的 \(\alpha_i\) 非零，决策函数可写成：

\[
f(x)=\sum_i\alpha_i y_iK(x_i,x)+b.
\]

核函数（kernel）将内积替换为隐式特征空间中的相似度。RBF 核常写为 \(K(x,z)=\exp(-\gamma\|x-z\|^2)\)。\(\gamma\) 越大，局部相似度衰减越快，决策边界可能越弯曲。\(C\)、\(\gamma\) 与特征尺度共同决定几何，必须一起调优。

## 3. 金融应用的高频失效机制

### 3.1 特征未标准化

SVM 使用内积或距离，量纲大的特征可能主导边界。标准化器必须在训练折拟合；若在全样本估计均值/方差，未来分布已泄漏进训练流程。异常值会影响均值方差，因此需报告缩放方式与尾部处理规则。

### 3.2 标签未来信息或边界污染

未来收益标签必须从当时可执行的入场开始，并等到完整持有区间成熟。重叠持有期使相邻日期结果共享未来价格；若随机拆分，支持向量也可能成为近重复信息的代表。按标签信息区间隔离比机械删除固定几日更合理。

### 3.3 RBF 默认值和搜索过度

核 SVM 的灵活度会随 \(C\)、\(\gamma\)、特征维度和尺度共同变化。网格尝试得越多，越容易偶然找到某个验证窗口的赢家。应先比较线性 SVM 与透明基线，再在内层前向验证做有界搜索，记录全部候选和停止规则。

### 3.4 类不平衡和概率误读

标准 SVM 的间隔目标不自动关注稀有正类，也不输出经过校准的概率。类别权重会改变错分代价，需报告其定义，并在代表真实基率的时间有效校准块上拟合 Platt scaling 或其他校准器。SVM 分数或间隔不应直接称为违约概率/上涨概率。

### 3.5 支持向量可能暴露数据问题

若支持向量集中在某些年份、低流动性证券、坏报价或事件标签边界上，模型边界可能由这些脆弱观测主导。检查支持向量的日期、证券、质量标记、行业和标签区间。支持向量占比本身既不是过拟合的充分证据，也不是模型质量证明。

## 4. 正确的诊断与调参协议

1. 冻结目标、样本单位、正负类、期限和交易基准；检查退市与不可成交标签。
2. 在每一训练折拟合缺失填补、类别编码、标准化和特征筛选。
3. 按时间留出外层未来区间；同一决策日整体分组，对标签重叠区间 purging。
4. 先与基率、规则、正则化 Logistic 和线性 SVM 比较；仅在有稳定增量时考虑 RBF 等核。
5. 在内层验证共同选择 \(C\)、核带宽与类别权重；限制搜索规模并完整登记尝试。
6. 评估 PR-AUC、ROC-AUC、混淆矩阵、间隔分布、支持向量构成和时间分段稳定性。
7. 仅在独立时间校准段校准概率，再单独确定操作阈值；最终测试期只用一次。
8. 若用于交易，将分数转为组合与订单，纳入延迟、价差、冲击、限价/停牌、借券、部分成交与资金约束。

## 5. A 股与美股适配

| 事项 | A 股 | 美股 | 共同要求 |
|---|---|---|---|
| 横截面切分 | 同一交易日全市场股票为共同冲击组 | 同日多场所与行业受共同冲击 | 日期整组留出，事件和发行人重复样本也要审计 |
| 标签可执行性 | T+1、涨跌限制、停复牌和产品交易规则 | 常规/延长时段、熔断/停牌、借券与场所 | 从可执行入场开始定义结果，列出不可成交处理 |
| 历史标的 | 退市、风险警示、上市板块和成分变化 | 退市、股类、ADR、代码变化与借券历史 | 稳定实体主键与有效期证券映射 |
| 特征几何 | 市场波动和因子尺度可能随板块/状态改变 | 多交易时段与不同流动性标的分布不同 | 每个训练折拟合缩放；跨市场不可默认共用 \(\gamma\) |

跨市场应用需要单独对齐单位、货币、时区、特征定义和验证期。核参数不是可直接迁移的经济常数。

## 6. 放行清单

- [ ] 目标和标签时钟没有未来泄漏，标签成熟与重叠规则已明确。
- [ ] 标准化/特征筛选均只在训练折拟合。
- [ ] 线性基线与核模型在同一时间窗、同一样本上公平比较。
- [ ] \(C\)、\(\gamma\)、权重和校准器在内层/时间有效校准块选择。
- [ ] 输出区分间隔、排序、概率校准和成本后行动效用。
- [ ] 支持向量身份和数据质量已审计，市场分段结果已报告。
- [ ] 最终外层结果及完整尝试记录可追溯。

如果收益优势依赖单一核宽度、某一事件期、全样本缩放或不可复现的阈值，应退回线性基线或停止部署。凸优化的全局收敛不等于市场泛化。

## 参考资料

> 以下文献用于追溯软间隔支持向量机的经典形式；其原始任务结果不构成金融时序的效果证明。

- Cortes & Vapnik (1995), [支持向量网络（Support-Vector Networks）](https://doi.org/10.1007/BF00994018).

## 英文阅读指南（English reading guide）

Sections 2–3 explain the SVM optimization and financial failure modes. Sections 4–6 provide a time-aware tuning protocol, A-share/U.S. adaptation, and release checklist. SVM scores are not calibrated probabilities unless a separate valid calibration step is performed.

