# 金融分类模型为何失效（Why My Financial Classification Model Failed）

> **英文摘要（English summary）：** This report explains why a financial classifier can converge, achieve strong in-sample scores, and still fail out of time. It covers target design, point-in-time data, class imbalance, logistic regression and related classifiers, calibration, threshold selection, temporal validation, and the conversion of probabilities into executable decisions. A-share and U.S. equity sections highlight market-specific data and trading constraints. All numerical illustrations must be treated as hypothetical unless a reproducible dataset, code, and account-level ledger are supplied.

**范围说明**：本文是分类方法诊断框架，不代表真实信用风险或股票策略的实证结果。数值示例仅用于讲解计算逻辑，不应视为回测证据。

## 1. 分类模型到底要回答什么

分类（classification）将观测映射为类别或类别概率。金融研究至少要区分：

- 信用违约、风险事件识别与欺诈筛查：估计事件是否发生；
- 涨跌方向或收益区间预测：估计未来结果落在哪个区间；
- 证券排序：在同一决策日的可投资集合中比较机会；
- 交易动作：在成本、风险和容量约束下决定是否下单。

这些任务的标签基率（base rate）、预测期限、漏报/误报成本和行动单位不同。把“未来上涨”设成正类并不完整；必须同时规定决策时点、收益区间、基准、公司行为、退市回收、无法成交时的标签处理及标签成熟时间。

## 2. 从似然看 Logistic 回归

二分类 Logistic 回归（logistic regression）令：

\[
p_i=P(Y_i=1\mid X_i)=\sigma(X_i^\top\beta),\qquad
\sigma(z)=\frac{1}{1+e^{-z}}.
\]

对 \(Y_i\in\{0,1\}\)，对数似然为：

\[
\ell(\beta)=\sum_i\left[Y_i\log p_i+(1-Y_i)\log(1-p_i)\right].
\]

估计量通过最大化似然得到。数值优化收敛只表示算法找到给定样本与假设下的驻点；它不证明目标定义正确、样本有代表性、概率校准良好或未来可迁移。多分类通常用多项 Logistic（multinomial logistic）；有序信用等级还需区分有序模型与无序类别模型。

常见替代模型也有专属假设：线性判别分析（LDA）要求类条件分布共享协方差；二次判别分析（QDA）分别估计各类协方差，数据需求更高；K 近邻（KNN）依赖距离、尺度与局部样本密度。换模型不会修复标签泄漏或错误抽样。

## 3. 失败链：从数据到决策

### 3.1 点时样本与标签错误

- 当前指数成分回填历史，会删除已退市公司并造成幸存者偏差（survivorship bias）。
- 财报期末、公告公开、数据供应商入库和交易决策不是同一时间；把期末值当成当时已知会造成前视偏差（look-ahead bias）。
- 未来收益标签未等待完整持有期成熟，或跨过验证边界，可能让同一未来事件同时影响训练和测试。
- 对“无法成交”样本直接删行，会让分类器只学习容易成交的幸存观测。

每条样本应保留决策时刻、信息可得时刻、事件区间、标签成熟时刻、证券身份和数据版本。

### 3.2 类别不平衡与分离

少数类比例很低时，全部预测多数类也可能取得很高准确率（accuracy）。稀有违约、退市或极端下跌事件还可能集中在少数年份或行业，导致样本内有效信息远少于行数。

完全分离或准完全分离（complete / quasi-complete separation）会使 Logistic 系数趋于极大，优化器却仍可能显示收敛。应检查系数与标准误是否异常、迭代警告、不同窗口下符号是否稳定，并考虑收缩估计或 Firth 修正；这些方法不能增加缺失的独立事件。

过采样、欠采样和类别权重会改变训练目标中的有效类别先验。经过这些变换的原始输出一般不能直接解释为自然总体中的事件概率，需在符合真实基率的时间有效校准集上重新校准。

### 3.3 线性 Logit、交互和非线性边界

Logistic 回归在线性预测变量 \(X^\top\beta\) 上施加线性结构。风险阈值、交互作用和状态依赖若未建模，模型可能排序尚可但在高风险区域系统低估。增加样条、交互或树模型前应先提出经济机制，再通过训练期内嵌套验证评估，不能反复看最终测试集决定形状。

### 3.4 随机拆分制造虚假的泛化

金融面板中的同一天股票共享市场冲击，同一公司重复出现，同一事件的新闻和多窗口标签还会高度重叠。随机拆行或仅保持类别比例的分层拆分（stratified split）会让近重复的日期、发行人或事件穿越训练/测试边界。

外层评估应按未来日期推进；同一决策日的证券整体留在同一折。标签信息区间重叠时按区间清除（purging），只有仍存在短程依赖时才增加隔离期（embargo）。分层可以改善折间类别构成，但不替代时间隔离。

## 4. 指标、校准与阈值分别回答不同问题

- 混淆矩阵、精确率（precision）、召回率（recall）依赖阈值，回答特定操作点下的错误构成。
- ROC-AUC 衡量随机正例排在随机负例之前的概率；对稀有事件，精确率—召回率曲线和 PR-AUC 常更贴近告警负担。
- 对数损失（log loss）和 Brier 分数评价概率误差；可靠性图检查“预测 20% 的事件是否大约发生 20%”。
- 校准器（calibrator）应在独立且时间靠后的校准块拟合。校准后的概率仍需按交易收益、损失、成本、容量和风险预算选择行动阈值。
- Hosmer–Lemeshow 检验对分箱和样本量敏感，不能作为校准的唯一证据。

AUC 高不等于概率准确；概率校准好也不代表收益为正；准确率高更不等于交易可执行。

## 5. 正确的建模与验证顺序

1. 冻结估计对象：样本单位、正类定义、期限、基准、行动时刻与不可成交规则。
2. 生成点时面板：保留历史存续/退市证券、原始字段与修订记录；缺失填补、编码、标准化和特征筛选只在训练折拟合。
3. 先做简单基线：正类基率、正则化 Logistic、预先定义的规则；再判断 LDA、QDA、KNN 或非线性分类器是否带来增量。
4. 使用嵌套的时间验证（nested walk-forward validation）：内层选择模型、超参数、校准方法和阈值；外层只评估冻结后的流程。
5. 在自然基率样本上评价概率和阈值效用；完整记录尝试次数、失败方案及测试集复用情况。
6. 将冻结预测转成订单，建模延迟、价差、冲击、费用、借券、拒单、部分成交和持有期现金流。
7. 按年份、行业、规模、流动性、波动状态和市场分别报告；用独立日期/事件数量而非面板行数描述证据量。

## 6. A 股与美股的具体审查

| 环节 | A 股需要核对 | 美股需要核对 | 两边共同要求 |
|---|---|---|---|
| 证券池 | 上市/退市、风险警示、停复牌、板块与历史成分 | 退市证券、股类、代码变更、ADR 映射和历史指数成分 | 使用有效期明确的稳定证券标识 |
| 信息时钟 | 披露可得时间、停牌、价格限制及产品交易规则 | 常规/延长时段公告、监管文件接收时间、停牌和借券状态 | 标签从可执行入场时点开始，到完整结果期结束 |
| 标签删失 | 涨跌停或停牌使目标价不可达 | 熔断/停牌、借券不可得及公司行动影响标签 | 报告无法交易样本的比例与处理方式 |
| 经济评估 | T+1、价格限制、交易单位及费用版本 | 多场所、卖空借券、价差、路由与公司行动 | 从概率经阈值、组合、订单到成本后账户账本逐层复算 |

市场制度、费用和数据源均应按证券、账户、交易场所与生效日期版本化。跨市场移植必须重新定义样本和标签并独立验证，不能直接沿用某一市场的阈值或校准器。

## 7. 放行清单与停止条件

- [ ] 正类、决策时钟、结果期限和行动成本已冻结并能复现。
- [ ] 历史证券池、退市记录、字段版本、标签成熟和不可成交样本均已审计。
- [ ] 所有预处理在每个训练折单独拟合；同日、同公司和重叠事件没有泄漏。
- [ ] 以基线为参照报告 PR/ROC、概率校准、阈值效用和时间外分段结果。
- [ ] 类别权重/重采样后的概率已在代表自然基率的样本上校准。
- [ ] 成本后结果包含未成交、借券、限制状态、现金流和组合风险。
- [ ] 最终测试期未参与特征、模型、校准器或阈值选择；实验次数可追溯。

若历史标签无法点时复原、少数类仅来自极少数独立事件、校准在目标操作区间失效，或成本后效用为负，应停止调参并降级结论。数值收敛和样本内分数不是放行条件。

## 参考资料

> 以下文献用于追溯概率预测评分与偏差修正方法，不构成金融市场适用性或收益证据。

- Brier (1950), [概率预测的检验（Verification of Forecasts Expressed in Terms of Probability）](https://doi.org/10.1175/1520-0493(1950)078%3C0001:VOFEIT%3E2.0.CO;2).
- Firth (1993), [极大似然估计的偏差修正（Bias Reduction of Maximum Likelihood Estimates）](https://doi.org/10.1093/biomet/80.1.27).

## 英文阅读指南（English reading guide）

Read Sections 1–3 for the estimand and failure mechanisms, Section 4 for metric interpretation, Section 5 for validation design, and Sections 6–7 for market adaptation and release controls. The report is a research framework; it does not claim a live or backtested trading edge.

