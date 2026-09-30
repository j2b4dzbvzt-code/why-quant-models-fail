# 金融非线性回归应用为何失效（Why My Nonlinear Regression Application Failed）

> **英文摘要（English summary）：** Nonlinear regression can represent curved relationships, but its flexibility introduces non-convex optimization, parameter-identifiability, data-support, and extrapolation risks. This report explains nonlinear least squares, Gauss–Newton and Levenberg–Marquardt optimization, quote-quality controls for option surfaces, and validation that separates interpolation from extrapolation. It includes A-share and U.S. market checks. No empirical pricing or trading result is claimed.

**案例等级**：方法报告与复核框架。本文不声称任何期权曲面或收益策略已通过真实市场验证。

## 1. 失败现场：拟合收敛，曲面却不可信

非线性回归（nonlinear regression）常用于期权隐含波动率曲面、期限结构、风险响应和饱和型财务关系。训练误差下降或优化器报告“成功”，只说明在某个参数化、起点和停止条件下找到一个数值解；它不保证全局最优、参数可识别、曲面处处受数据支持，或外推具有经济意义。

对报价曲面尤其要把三件事分开：报价拟合（与观测价差的贴合）、无套利约束（模型价格不制造可实现的静态套利）和交易收益（真实订单成交后的盈亏）。其中一项通过不能代替另外两项。

## 2. 非线性最小二乘及优化机制

给定 \(y_i=f(x_i,\theta)+\epsilon_i\)，非线性最小二乘（nonlinear least squares, NLS）求解：

\[
\hat\theta=\arg\min_\theta Q(\theta),\qquad
Q(\theta)=\frac12\sum_{i=1}^{n}w_i\,[y_i-f(x_i,\theta)]^2.
\]

令残差 \(r_i(\theta)=y_i-f(x_i,\theta)\)，雅可比矩阵（Jacobian）为 \(J_{ij}=\partial f(x_i,\theta)/\partial\theta_j\)。Gauss–Newton 在当前参数处线性化模型，近似求解：

\[
(J^\top WJ)\Delta\theta=J^\top Wr,\qquad
\theta_{\text{new}}=\theta+\Delta\theta.
\]

当线性化局部有效且 \(J^\top WJ\) 条件良好时，更新可以高效；远离解、参数弱识别或矩阵病态时，步长可能不稳定。Levenberg–Marquardt 通过加入阻尼项 \(\lambda I\) 在 Gauss–Newton 与梯度下降之间调节，但不因此获得全局最优保证。结果可能依赖初始化、参数尺度、权重、边界与停止容差。

线性回归的闭式凸问题性质不能直接套到这里。非线性模型即使所有残差有限、梯度很小，也可能在局部极小、鞍点或平坦参数谷中。

## 3. 金融数据中的专属失效路径

### 3.1 输入空间覆盖不均

非线性模型在报价密集区可以插值得很好，在稀疏的期限、深度虚值区域或压力区间却主要依赖函数形式。把所有观测随机拆分，训练和测试都会落在同一密集区域，报告出的误差只测到插值，没有测到外推。

先绘制各输入维度和联合区域的样本支持；对整个期限段、行权价带、流动性层或波动状态留出区域，单独报告误差和不确定性。

### 3.2 报价错误被稀疏区域放大

坏 tick、过时 bid/ask、交叉报价、极低成交量和异步标的价格会扭曲拟合。异常报价若恰好位于稀疏区域，其影响可能远大于同等残差在密集区域的影响。稳健损失（robust loss）可降低部分极端残差影响，却不能辨认错误报价、修复错配合约或决定经济权重。

中间价只是 bid 与 ask 的统计摘要，不等于可成交价；评估拟合时应同时保留原始买卖报价、成交、报价时间、价差与流动性状态。

### 3.3 参数不可识别与过度参数化

若两个参数组合产生近似相同的曲面，参数在统计上就难以区分。检查雅可比/信息矩阵条件数、参数相关性、置信区间、profile likelihood，以及不同窗口/起点的参数漂移。参数估计不稳定时，单点拟合误差很小仍可能对应巨大的风险对冲差异。

扩大自由度能降低样本误差，却常提高方差和外推敏感性。选择参数化时要依据数据覆盖、用途和稳定性，而不是仅比较全样本 SSE。

### 3.4 目标函数与经济含义错配

等权最小二乘让不同价差、成交活跃度、期限和敏感度的报价拥有相同平方误差权重；按价差加权、按 vega 加权或在价格空间拟合会得到不同目标。必须预先解释每种权重服务于报价拟合、风险估值还是对冲用途。不得在看到测试误差后换权重，只为改善结果。

### 3.5 套利约束遗漏或写错

估计隐含波动率曲面再转成期权价格时，形状平滑不等于满足单调性、凸性等无静态套利条件。约束应在一致的合约、贴现、分红和标的口径下定义；离散网格上未发现违规，也不能证明网格之间或动态对冲过程正确。

## 4. 诊断与修复顺序

1. 验证合约映射、标的价格、报价时间戳、到期、行权价、乘数、公司行动和利率/分红曲线。
2. 预先冻结清洗规则：交叉/锁定报价、过期报价、异常价差、零成交和无效合约如何处理；保留原始记录与剔除原因。
3. 从低自由度、可解释的参数化开始；按市场和用途比较模型族，不以单一总误差选型。
4. 对参数做合理缩放和边界约束，使用多个分散起点；保存目标值、梯度、收敛状态、迭代次数和参数。
5. 画按到期、行权价、moneyness、流动性与市场状态拆分的残差；检查雅可比条件数、参数 profile 和对输入扰动的敏感度。
6. 把验证拆成随机局部留点的插值评估与整块留域的外推评估，按实际决策方向选择留出边界。
7. 单独检查无套利约束、定价误差、对冲误差和成本后交易结果，不将它们合成一个均方误差结论。

## 5. A 股与美股期权面板

| 项目 | A 股 | 美股 | 共同控制 |
|---|---|---|---|
| 合约和交易 | 不同 ETF/指数期权的合约、行权、到期和历史规则不同 | 股票、ETF、指数期权的行权/交割、分红与提前行权特征不同 | 用合约级主键与生效期保存规则版本 |
| 报价质量 | 流动性、停牌、价格边界和做市报价覆盖需按产品检查 | 多场所、NBBO、延长时段、周度合约和事件跳空需区分 | 保留原始 bid/ask、成交、时间、价差和报价状态 |
| 标的与公司行为 | ETF/指数成分和合约调整会改变映射 | 拆并股、特殊分红、并购、ADR 与股类映射影响合约 | 点时匹配标的、合约乘数、分红与贴现曲线 |
| 外推区域 | 远月/深虚值/稀疏行权价区域支持有限 | 短期限、事件期和远端行权价覆盖不均 | 按期限、moneyness、流动性和状态分块验证 |

中间价、理论价、可成交价应分别呈现。静态无套利筛查、曲面误差和动态对冲绩效是不同证据层。

## 6. 交付报告与停用条件

最低报告应包含模型用途、输入支持区域、报价筛选、目标函数与权重、参数化、起点集合、优化诊断、可识别性检查、插值/外推分块误差、套利筛查、版本信息和停止规则。

出现以下情况时不应把曲面用于决策：参数跨合理起点/窗口剧烈漂移；预测依赖无报价支撑的外推；报价错误处理改变结论；无套利违规或风险对冲对微小输入极敏感；只在随机拆分有效而留域验证失败。应收缩模型自由度、限制适用区域或回退到有支持的基线。

## 参考资料

> 以下文献用于追溯非线性最小二乘的经典数值算法，不代表算法能消除可识别性或外推问题。

- Marquardt (1963), [非线性参数最小二乘估计算法（An Algorithm for Least-Squares Estimation of Nonlinear Parameters）](https://doi.org/10.1137/0111030).

## 英文阅读指南（English reading guide）

Sections 2–3 explain the estimator and its failure modes; Sections 4–6 provide diagnostics, A-share/U.S. checks, and release rules. The article distinguishes quote fit, no-arbitrage checks, hedge quality, and executable trading outcomes.

