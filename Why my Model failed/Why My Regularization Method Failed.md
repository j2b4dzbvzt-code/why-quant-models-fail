# 金融正则化方法为何失效（Why My Regularization Method Failed）

> **英文摘要（English summary）：** Regularization controls model complexity by shrinking coefficients, but it cannot repair leakage, unstable labels, misspecified targets, or non-representative validation. This report derives Ridge, Lasso, and Elastic Net, explains scaling and correlated-factor effects, and gives a nested time-validation workflow for A-share and U.S. financial panels. The goal is stable out-of-time evidence, not merely a sparse coefficient table.

**案例等级**：方法报告与审查清单；示例不代表任何实证策略结果。

## 1. 正则化解决的是哪一类问题

正则化（regularization）在拟合损失之外加入参数惩罚，以限制模型自由度、降低估计方差或鼓励稀疏性。它不能自动解决：

- 特征在历史决策时不可得；
- 标签、证券池或目标期限定义错误；
- 训练/验证切分泄漏；
- 数据生成过程发生结构变化；
- 预测信号无法映射到可成交净收益。

如果这些问题存在，正则化可能把错误关系缩小、筛选或稳定地复制，并不会让它变成真实信号。

## 2. Ridge、Lasso 与 Elastic Net 的目标函数

在平方损失线性模型中，令中心化后的设计矩阵为 \(X\)，响应为 \(y\)。Ridge 回归（岭回归）求：

\[
\hat\beta_{\mathrm{ridge}}
=\arg\min_\beta \left\{\frac{1}{2n}\|y-X\beta\|_2^2+
\lambda\|\beta\|_2^2\right\}.
\]

若无额外约束，闭式解为：

\[
\hat\beta_{\mathrm{ridge}}=(X^\top X+2n\lambda I)^{-1}X^\top y,
\]

这里系数取决于损失的归一化约定；不同软件的 \(\lambda\) 标度未必相同。Ridge 通常缩小相关特征的系数，但不会将系数精确置零。

Lasso 回归（最小绝对收缩与选择算子）使用 \(L_1\) 惩罚：

\[
\hat\beta_{\mathrm{lasso}}
=\arg\min_\beta\left\{\frac{1}{2n}\|y-X\beta\|_2^2+
\lambda\|\beta\|_1\right\}.
\]

在标准化、正交列的特殊情形下，它对应软阈值操作：

\[
\hat\beta_j=\operatorname{sign}(z_j)(|z_j|-\lambda)_+.
\]

Elastic Net（弹性网络）结合两种惩罚：

\[
\arg\min_\beta\left\{\frac{1}{2n}\|y-X\beta\|_2^2+
\lambda\left[\alpha\|\beta\|_1+
\frac{1-\alpha}{2}\|\beta\|_2^2\right]\right\}.
\]

\(\lambda\) 控制整体收缩，\(\alpha\) 控制稀疏成分占比。截距通常不惩罚；标准化、损失缩放与参数约定必须记录，才能复现结果。

## 3. 金融研究中常见的失效路径

### 3.1 尺度让惩罚失去公平性

惩罚直接作用于系数。如果某列用百分点、另一列用小数、第三列用美元计量，相同经济效应可能对应不同系数大小。通常在每个训练折内估计均值/尺度，再变换验证折；任何全样本标准化会泄漏未来分布信息。

哑变量、稀疏变量、已有单位意义的特征是否标准化，需按模型定义事前决定。不能因为某列“看起来不重要”而在测试期后重新缩放规则。

### 3.2 相关因子让稀疏选择不稳定

行业、规模、价值、动量和流动性特征常高度相关。Lasso 可能在相近变量中任意择一；更换窗口、样本或微小扰动后，入选变量就变了。Ridge 往往把权重分摊到相关因子，Elastic Net 可在某些条件下缓解分组不稳定，但三者都不能证明某个因子有经济因果含义。

应报告路径稳定性、选择频率、系数符号和相关特征组的整体预测贡献，而不是只展示一次拟合的非零变量。

### 3.3 惩罚参数被错误验证方式选出

随机 K 折交叉验证在面板/时序中会打散日期与事件，把相关行情同时分给训练和验证。由此选择出的 \(\lambda\) 往往优化了泄漏后的插值任务。嵌套验证中，外层模拟未来部署；内层只在可用历史上选择 \(\lambda\)、\(\alpha\)、特征和预处理。

前向验证也需按标签信息区间处理重叠。Purging 应基于实际特征/标签区间；隔离期（embargo）应有依赖结构理由，而不是机械使用任意固定天数。

### 3.4 预筛选造成隐藏自由度

先对全部日期做相关性筛选、单变量显著性筛选或 PCA，再对筛后的特征做正则化，相当于用验证/测试信息参与筛选。后续的惩罚路径不会替先前的数据窥探补缴复杂度。预处理、筛选和变换要在每个训练折内重做，记录所有候选与访问次数。

### 3.5 惩罚不能修复结构问题

正则化的偏差—方差权衡（bias–variance trade-off）是在既定模型族和抽样条件下进行的。若市场断点、标签错位、缺失非随机或目标关系错误，系数更小不等于泛化更好。弱信号被收缩到零也可能合理，不能为追求非零因子而反复降低惩罚。

## 4. 如何证明系数和预测具有稳定性

- 在滚动窗口、扩展窗口、市场状态和合理样本扰动下重估完整流程。
- 报告 Ridge/Lasso/Elastic Net 的时间外误差与基线差异，不仅比较交叉验证均值。
- 统计每个因子的入选频率、符号一致率、系数分布和相关组聚合结果。
- 对重采样进行依赖感知设计；随机抽行 bootstrap 不等同于未来风险区间。
- 区分“特征重要性”“预测贡献”和“因果效应”。非零系数不代表可交易或可解释的因果驱动。
- 若目标是概率或排序，还应检查校准、分位段表现和实际决策效用。

最终测试期必须锁定。反复比较不同惩罚路径、因子集和收益标签后再报最优结果，会使最终测试退化成新的训练集。

## 5. 可复现的重估流程

1. 定义目标、预测期限、证券池、交易时钟和经济基准。
2. 建立点时特征面板，明确历史证券、修订记录、缺失含义、退市和标签成熟状态。
3. 冻结外层未来评估窗；同一决策日证券整组切分，重叠标签区间按设计 purging。
4. 在每个内层训练折拟合填补、编码、缩放、筛选和候选模型；记录随机种子和全部搜索尝试。
5. 对比 OLS/简单基线、Ridge、Lasso、Elastic Net；在内层选惩罚结构，不看外层结果挑参数。
6. 检查正则化路径、特征组选择稳定性和跨窗系数漂移；如果变量身份不稳定，报告组级结论或不作变量解释。
7. 冻结模型后，在外层时间段评价预测；交易用途须进一步映射至换手、成本、容量、约束和账户现金流。
8. 用合理费用、延迟、证券池和市场状态做敏感性分析，报告失败的规格和试验总数。

## 6. A 股与美股因子面板适配

| 维度 | A 股检查 | 美股检查 | 共同原则 |
|---|---|---|---|
| 历史覆盖 | 退市、风险警示、停牌、板块与上市规则变化 | 退市、股类、ADR、代码变化和历史成分 | 稳定标识与有效期历史不能缺失 |
| 因子时点 | 财报披露时间、供应商延迟和后续修订 | SEC 文件接收时间、修订和标准化延迟 | 特征只取决策时点可得版本 |
| 横截面依赖 | 同日市场/行业冲击、涨跌限制导致的标签删失 | 同日共振、行业事件、借券和多场所交易 | 以日期/事件组织验证，而非随机拆行 |
| 成本后映射 | T+1、涨跌停、停牌、费用与可交易量 | 价差、冲击、卖空借券、场所路由和延长时段 | 预测分数经过组合与真实订单才能成为经济证据 |

若将美股因子训练后直接迁移到 A 股，货币、单位、行业映射、交易日历、市场暴露和因子定义均需重新审计。标准化的“同一列名”不保证含义相同。

## 7. 放行条件与停止条件

- [ ] 所有特征和标签在历史决策时点可重建。
- [ ] 缺失处理、标准化、特征筛选和惩罚参数完全嵌套在训练数据内。
- [ ] 验证同时尊重时间、日期群组和标签信息区间。
- [ ] 报告惩罚路径、系数/选择稳定性、时间外增量和完整搜索记录。
- [ ] 预测结论与变量解释分开；无因果证据时不写因果措辞。
- [ ] 交易主张经过成本、容量、可成交状态和账户账本验证。

若增量只在某一个惩罚参数、某一窄窗口或一个市场出现，或成本后完全消失，应回退基线并收缩结论。正则化不是给已筛选出的漂亮因子盖章。

## 参考资料

> 以下文献用于追溯 Lasso 与 Elastic Net；变量稀疏不等同于经济解释或样本外稳定性。

- Tibshirani (1996), [Lasso 回归收缩与变量选择（Regression Shrinkage and Selection via the Lasso）](https://doi.org/10.1111/j.2517-6161.1996.tb02080.x).
- Zou & Hastie (2005), [Elastic Net 正则化与变量选择（Regularization and Variable Selection via the Elastic Net）](https://doi.org/10.1111/j.1467-9868.2005.00503.x).

## 英文阅读指南（English reading guide）

Sections 2–3 define the penalties and common failure mechanisms; Sections 4–7 cover stability, nested time validation, market adaptation, and release criteria. A sparse model is not automatically a stable or economically useful model.

