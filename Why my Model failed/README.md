# Why My Model Failed：量化模型失效诊断系列

本系列沿着同一条研究链定位失败原因：

**点时数据 → 可证伪目标 → 模型假设 → 时间外验证 → 交易/投资组合映射 → 生产监督**

快速入口：[量化研究审查清单](../量化研究审查清单.md) · [仓库首页](../README.md)

系列报告统一采用中文正文，并在开头提供英文摘要与阅读指南；关键术语按“中文（English）”标注。原有分类、线性回归、信息准则和重采样研究流程（pipeline）已融合在对应 Why My 报告中。

每篇专题均列出可追溯的方法参考资料，并说明它们不等于对金融市场适用性或收益的证明。索引也提供[按失败症状快速定位](#按失败症状快速定位)，适合先从模型表现、阈值、稳定性或执行问题切入。

核心问题不是“哪个算法最强”，而是：模型在哪一层失去可信度，证据如何定位，修复后是否仍有样本外、成本后、可执行的增量价值。模型分数、拟合优度、显著性、聚类图或模拟器奖励，都不能单独证明一项策略可以交易。

## 英文索引（English index）

These postmortems examine where a quantitative model loses validity—from point-in-time data and target construction through validation, portfolio translation, execution, and monitoring. Every report has a Chinese main text and an English summary/reading guide. The reports are methodological reviews, not claims of verified investment performance.

**Regression and model selection:** [Linear regression](./Why%20My%20Financial%20Linear%20Regression%20Model%20Failed.md) · [Financial classification](./Why%20My%20Financial%20Classification%20Model%20Failed.md) · [Information criteria](./Why%20My%20Information%20Criteria%20Application%20Failed.md) · [Regularization](./Why%20My%20Regularization%20Method%20Failed.md) · [Nonlinear regression](./Why%20My%20Nonlinear%20Regression%20Application%20Failed.md) · [SVM](./Why%20My%20SVM%20Application%20Failed.md)

**Trees, deep learning, and time series:** [Decision trees and random forests](./Why%20My%20Decision%20Tree%20Application%20Failed.md) · [Gradient boosting](./Why%20My%20Gradient%20Boosting%20Model%20Failed.md) · [Deep learning](./Why%20My%20Deep%20Learning%20Model%20Failed.md) · [Time series and volatility](./Why%20My%20Financial%20Time-Series%20and%20Volatility%20Model%20Failed.md) · [Unsupervised learning](./Why%20My%20Unsupervised%20Learning%20Application%20Failed.md) · [Anomaly detection](./Why%20My%20Financial%20Anomaly%20Detection%20Model%20Failed.md)

**Ranking, causal analysis, portfolios, and decisions:** [Ranking and recommenders](./Why%20My%20Ranking%20and%20Recommender%20Model%20Failed.md) · [Causal inference](./Why%20My%20Causal%20Inference%20Model%20Failed.md) · [Portfolio optimization](./Why%20My%20Portfolio%20Optimization%20Model%20Failed.md) · [Reinforcement learning](./Why%20My%20Reinforcement%20Learning%20Trading%20Model%20Failed.md) · [Resampling and validation](./Why%20My%20Resampling%20Failed.md)

For a general research audit, start with the [checklist](../量化研究审查清单.md). The repository homepage has a short [English overview](../README.md).

## 按失败症状快速定位

先找“最先失真的环节”，再读模型专题。表中的症状是排查入口，不是单凭一个指标就能成立的诊断结论。

| 你看到的症状 | 优先排查 | 建议阅读 |
|---|---|---|
| 随机切分成绩很好，换成后续年份就明显衰退 | 标签是否跨越切分边界、证券池是否点时、全样本预处理与重复调参 | [重采样与验证](./Why%20My%20Resampling%20Failed.md) · 对应模型报告 |
| AUC/IC 上升，成本后组合收益反而下降 | 概率/排名怎样转成阈值、仓位和订单；换手、冲击与未成交如何入账 | [分类模型](./Why%20My%20Financial%20Classification%20Model%20Failed.md) · [排序模型](./Why%20My%20Ranking%20and%20Recommender%20Model%20Failed.md) · [组合优化](./Why%20My%20Portfolio%20Optimization%20Model%20Failed.md) |
| 系数、特征权重或组合权重随窗口大幅跳变 | 共线性、惩罚强度、估计误差、协方差条件数及时间窗敏感性 | [正则化](./Why%20My%20Regularization%20Method%20Failed.md) · [组合优化](./Why%20My%20Portfolio%20Optimization%20Model%20Failed.md) |
| 树模型分数不错，但重要特征、切分点或跨期表现不稳定 | 叶节点有效样本、相关特征替代、类别/行业代理、搜索轮数与时间外切片 | [决策树与随机森林](./Why%20My%20Decision%20Tree%20Application%20Failed.md) · [梯度提升](./Why%20My%20Gradient%20Boosting%20Model%20Failed.md) |
| 训练拟合曲线平滑，稀疏区域或极端行情预测不可信 | 参数可识别性、边界约束、外推行为与局部数据覆盖 | [非线性回归](./Why%20My%20Nonlinear%20Regression%20Application%20Failed.md) · [深度学习](./Why%20My%20Deep%20Learning%20Model%20Failed.md) |
| SVM 准确率不错，但概率或交易阈值失灵 | 间隔分数和概率是否混为一谈、核宽度/特征尺度、校准集与最终测试是否隔离 | [支持向量机](./Why%20My%20SVM%20Application%20Failed.md) · [分类模型](./Why%20My%20Financial%20Classification%20Model%20Failed.md) |
| PCA 主成分、聚类标签或市场状态换个窗口就变化 | 尺度、协方差估计、维数与样本比、聚类稳定性和状态解释 | [无监督学习](./Why%20My%20Unsupervised%20Learning%20Application%20Failed.md) · [金融时序与波动率](./Why%20My%20Financial%20Time-Series%20and%20Volatility%20Model%20Failed.md) |
| 市场切换时异常告警突然泛滥 | 正常状态基线是否漂移、阈值是否点时估计、告警是否只是波动率变化 | [异常检测](./Why%20My%20Financial%20Anomaly%20Detection%20Model%20Failed.md) · [金融时序与波动率](./Why%20My%20Financial%20Time-Series%20and%20Volatility%20Model%20Failed.md) |
| 模拟器/回测里的策略奖励很高，真实订单执行却失败 | 行为数据覆盖、反事实有效性、撮合与冲击模型、账户状态转移 | [强化学习](./Why%20My%20Reinforcement%20Learning%20Trading%20Model%20Failed.md) · [金融数据原理](../金融数据整理/原理.md) |
| 因果效应随事件窗口、对照组或控制变量变化而翻转 | 处理时点、前趋势、处理组污染、溢出与识别假设是否可信 | [因果推断](./Why%20My%20Causal%20Inference%20Model%20Failed.md) · [重采样与验证](./Why%20My%20Resampling%20Failed.md) |
| AIC/BIC 选出的模型无法预测下一段时期 | 比较样本/似然是否一致、依赖结构、参数搜索是否泄漏；准则目标是否与预测任务一致 | [信息准则](./Why%20My%20Information%20Criteria%20Application%20Failed.md) · [重采样与验证](./Why%20My%20Resampling%20Failed.md) |

若症状跨越多个层次，从[统一复核框架](#统一复核框架)定位最早失真的证据，并用根目录[五阶段研究流程](../README.md#五阶段研究流程)检查上下游；不要从增加模型复杂度开始排错。

## 阅读路线

1. 先按“统一复核框架”检查研究对象、时间戳、证券池、标签、验证和交易账本。
2. 再按目标变量选择专题：收益/风险预测、分类、横截面排序、异常告警、因果估计、配置或执行是不同的问题。
3. 阅读所选文章中的模型假设、失效机制、端到端研究流程（pipeline）、市场差异和放行条件。
4. 回到上游数据和下游账本核实：修复只提高训练指标、不改善时间外结果，不算模型修复。

## 系列文章

### 回归、分类与统计选择

| 模型或研究问题 | Why My 报告 | 覆盖范围 |
|---|---|---|
| 线性回归与稳健推断 | [Why My Financial Linear Regression Model Failed](./Why%20My%20Financial%20Linear%20Regression%20Model%20Failed.md) | OLS、识别假设、异方差/自相关、HAC 与聚类推断、金融时间验证 |
| 逻辑回归（logistic regression）、线性判别分析（LDA）、二次判别分析（QDA）、K 近邻（KNN） | [Why My Financial Classification Model Failed](./Why%20My%20Financial%20Classification%20Model%20Failed.md) | 分类目标、概率、类不平衡、校准、阈值与分类研究流程（pipeline） |
| AIC、BIC、AICc 与模型选择 | [Why My Information Criteria Application Failed](./Why%20My%20Information%20Criteria%20Application%20Failed.md) | 似然、参数计数、比较条件、时间外预测与信息准则研究流程（pipeline） |
| Ridge、Lasso、Elastic Net | [Why My Regularization Method Failed](./Why%20My%20Regularization%20Method%20Failed.md) | 尺度、惩罚、相关因子、选择稳定性与嵌套验证 |
| 非线性回归 | [Why My Nonlinear Regression Application Failed](./Why%20My%20Nonlinear%20Regression%20Application%20Failed.md) | 可识别性、非凸优化、外推、约束和曲面稳定性 |
| 支持向量机 | [Why My SVM Application Failed](./Why%20My%20SVM%20Application%20Failed.md) | 间隔、核、尺度、类别权重、概率与时间窗 |

### 树、集成、深度学习与时序

| 模型或研究问题 | Why My 报告 | 覆盖范围 |
|---|---|---|
| 决策树、随机森林 | [Why My Decision Tree Application Failed](./Why%20My%20Decision%20Tree%20Application%20Failed.md) | 贪心切分、叶节点样本、自助聚合（bagging）/袋外评估（OOB）、重要性偏差与树模型研究流程（pipeline） |
| 梯度提升树 | [Why My Gradient Boosting Model Failed](./Why%20My%20Gradient%20Boosting%20Model%20Failed.md) | GBDT、XGBoost、LightGBM、CatBoost、早停与搜索偏差 |
| 神经网络与深度学习 | [Why My Deep Learning Model Failed](./Why%20My%20Deep%20Learning%20Model%20Failed.md) | MLP、CNN、RNN、GRU/LSTM、Transformer、预训练和因果时间窗 |
| AR/ARIMA、VAR、状态空间、GARCH | [Why My Financial Time-Series and Volatility Model Failed](./Why%20My%20Financial%20Time-Series%20and%20Volatility%20Model%20Failed.md) | 平稳性、残差、结构断点、波动率目标与滚动预测 |
| PCA、聚类与潜在状态 | [Why My Unsupervised Learning Application Failed](./Why%20My%20Unsupervised%20Learning%20Application%20Failed.md) | 高维距离、因子漂移、簇稳定性、状态解释与重采样研究流程（pipeline） |
| 异常检测 | [Why My Financial Anomaly Detection Model Failed](./Why%20My%20Financial%20Anomaly%20Detection%20Model%20Failed.md) | 稳健距离、PCA、Isolation Forest、LOF、One-Class SVM、自编码器、阈值与处置 |

### 排序、因果、组合与交易决策

| 模型或研究问题 | Why My 报告 | 覆盖范围 |
|---|---|---|
| 横截面排序与推荐 | [Why My Ranking and Recommender Model Failed](./Why%20My%20Ranking%20and%20Recommender%20Model%20Failed.md) | Pointwise/Pairwise/Listwise、暴露偏差、Top-K 与组合映射 |
| 因果推断 | [Why My Causal Inference Model Failed](./Why%20My%20Causal%20Inference%20Model%20Failed.md) | DID、事件研究、IV、合成控制、断点、处理异质性和识别边界 |
| 投资组合优化 | [Why My Portfolio Optimization Model Failed](./Why%20My%20Portfolio%20Optimization%20Model%20Failed.md) | 均值/协方差误差、最小方差、风险平价、稳健优化、约束、成本和容量 |
| 强化学习与订单执行 | [Why My Reinforcement Learning Trading Model Failed](./Why%20My%20Reinforcement%20Learning%20Trading%20Model%20Failed.md) | MDP/POMDP、离线策略评估、数据覆盖、模拟器、成本与安全约束 |

### 验证方法

- [Why My Resampling Failed](./Why%20My%20Resampling%20Failed.md)：区分预测误差、参数不确定性、策略路径风险与抽样单位；文章内含金融时间序列的重采样/CV 研究流程（pipeline）。
- 重采样并非所有项目的默认答案。按研究目标选择滚动/扩展窗、purging/embargo、区块重采样或事件/证券群组重采样；随机拆分高度相关的行通常不构成未来模拟。
- AIC/BIC 等样本内信息准则不能替代时间外预测；原有模型选择研究流程（pipeline）已融入信息准则诊断。
- 分类研究流程（pipeline）与 OLS/HAC 推导也已整合到对应 Why My 报告；本目录不再单独维护流程文档。

## 统一复核框架

| 层次 | 复核问题 | 常见逻辑错误 |
|---|---|---|
| 研究目标 | 预测/估计/排序/告警/决策的对象、期限和基准是否明确？ | 把相关性、预测能力、因果效应和可交易收益混作同一结论 |
| 信息集 | 特征在决策时是否已经可得？证券池与字段版本能否还原？ | 当前成分回填历史、公告期末误作公开时间、全样本预处理 |
| 标签与样本 | 标签成熟、重叠、删失、幸存者与不可交易状态如何处理？ | 未来收益窗口跨边界；只留成功/存续样本；错误删除尾部事件 |
| 验证设计 | 切分是否尊重日期、证券、事件和标签信息区间？ | 随机拆行；同一事件跨训练/测试；反复复用最终测试集 |
| 模型假设 | 损失、似然、距离、状态、协方差或因果识别条件是否成立？ | 收敛/显著即视为正确；用复杂算法掩盖不可识别或结构断点 |
| 选择与不确定性 | 是否披露尝试数量、种子/窗口敏感性及依赖结构？ | 只报告冠军参数；把相关日频行当独立样本 |
| 决策转换 | 预测如何转仓位、风险预算、订单和账户现金流？ | 用 AUC/IC/重构误差/异常分数直接宣称策略有效 |
| 上线监督 | 漂移、拒单、限额、延迟、账本差异触发什么操作？ | 没有停用条件或人工核验；异常分数自动触发不可逆动作 |

## A 股和美股的共同边界与差异

每篇市场适配都必须至少检查四层：

1. **点时数据**：公司行为、证券身份、上市/退市、分类、公告可得时间、时区与数据修订。
2. **可交易状态**：交易时段、集合/连续竞价、停复牌、价格边界、订单类型、卖空/借券和结算约束。
3. **成本与容量**：费用版本、价差、冲击、成交率、参与率、排队/未成交和融资成本。
4. **结果口径**：预测指标、统计不确定性、可成交收益和真实账户账本分开展示。

A 股普通股票研究常需建模 T+1、涨跌幅约束和停复牌等状态；美股研究常需区分常规与延长时段、多交易场所、卖空借券和不同公司行动。它们都依证券类型、账户、场所和生效日期而变化，不能把文中概括当成永久、无例外的交易参数。

跨市场迁移必须重新核对制度、数据供应商时间戳、标的池、费用和动作可行域。美股中得到的模型排序或阈值，不能直接视为 A 股参数；A 股回测也不能假设美股式连续成交和借券条件。

### 跨市场审计清单

| 维度 | 需要保留的时点信息 | 典型误判 |
|---|---|---|
| 证券身份 | 代码/主键有效期、上市/退市、历史成分、股类、ADR 映射和分类版本 | 只用今天仍存续的证券回填历史，造成幸存者偏差 |
| 披露与修订 | 事件发生、公开、系统接收、解析、修订和数据生效时间 | 把财报期末、公告时间和当时可交易时间视为同一时刻 |
| 交易日历 | 时区、夏令时、节假日、集合/连续竞价、常规/延长时段 | 将同一自然日的日线当作同一信息窗口 |
| 可交易状态 | 停复牌、价格边界、订单类型、成交/排队、借券和结算状态 | 目标权重或触及价格被误当成已成交 |
| 公司行为 | 分红、拆并股、送转、并购、退市和 ADR 比率变化 | 复权价格序列被直接当作账户现金流或成交价 |
| 费用与资金 | 佣金/规费、税费、价差、冲击、融资、借券和货币转换版本 | 用一项静态平均费用覆盖所有标的、时段和账户 |
| 横截面依赖 | 同日市场冲击、行业/指数暴露、共同事件和跨市场假日 | 把股票-日期行数当成独立样本量 |
| 结果报告 | 分市场、制度期、行业、规模/流动性切片及成本后账本 | 总体平均掩盖一个市场或一个状态下的失效 |

正式比较跨市场结果前，应先独立通过每个市场的时间外检验，再决定是否汇总。任何费率、交易约束和市场规则都要有来源、版本、生效区间和账户适用范围。

## 专家排错顺序与停止条件

1. **冻结失败现场**：保存原始模型、数据、参数、表现和失败时期，先说明失败是数据错误、预测退化、风险超限、成交偏差还是生产事故。
2. **核对估计对象**：写明目标变量、预测/持有期限、决策时点、信息集、交易基准和成功门槛。
3. **审计点时数据和标签**：重放证券池、字段版本、公司行为、标签成熟时间及重叠信息区间。
4. **复查切分与全部尝试**：按日期/证券/事件依赖设计验证，记录特征、窗口、模型、阈值、费用、随机种子和失败方案。
5. **检查模型专属假设**：系数/残差、切分/叶子、核尺度、概率校准、状态稳定性、因果识别、协方差条件或离线动作支持分别复核。
6. **连接决策和账本**：把预测/分数转换为仓位、订单、成交、现金和风险，分开报告模型指标与成本后账户结果。
7. **建立生产证据**：固定数据与模型版本、监控标签成熟队列、输入新鲜度、订单/账本差异，并定义暂停和回退动作。

遇到下列情形应停止继续调参，改为收缩主张或回退基线：历史证券池/字段版本不可重建；优势只存在于单一尖锐参数或无法成交标的；标签与决策时钟不相容；所有增量被合理成本/容量范围吞没；最终测试集已经反复参与选择；账户和风险账本不能复算。

## 证据等级与使用边界

- **方法说明**：推导、算法假设和诊断流程可复核，但不代表已在某市场测出收益。
- **示例结果**：文章中用于展示计算的假设数值不是实证结论；除非同时给出数据来源、区间、版本、代码和可复算账本，不应引用为收益证据。
- **时间外证据**：必须锁定决策时点、点时数据、证券池、成本和未成交规则，并披露策略/模型尝试过程。
- **生产证据**：还需影子运行、账户对账、风险闸门、版本治理与可回滚机制。

本系列用于研究设计、模型审查和回测排错，不提供投资建议、行情数据、券商接口或已验证的实盘策略。费率、制度与接口参数需按目标证券、市场、场所和日期核验。

## 与仓库其他专题的关系

本系列是模型层的补充，不能取代仓库根目录中的数据、信号、策略、回测和生产专题。建议先按[仓库主 README 的五阶段研究流程](../README.md#五阶段研究流程)明确上下游，再用此处的模型报告定位具体假设和失效环节。
