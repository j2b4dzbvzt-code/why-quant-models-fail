# 金融线性回归模型为何失效（Why My Financial Linear Regression Model Failed）

> **英文摘要（English summary）：** This report distinguishes prediction, association, and causal estimation in financial OLS. It covers point-in-time samples, identification assumptions, heteroskedasticity/autocorrelation-consistent inference, time-based validation, and the gap between regression scores and executable portfolios. HAC standard errors address certain inference problems; they do not repair leakage, misspecification, or biased estimates.

## 金融线性回归为什么会失败：OLS、HAC 与中美股票实务

> **定位**：本篇是线性回归失效诊断的中文主稿，涵盖问题定义、点时样本、OLS 推导、识别假设、稳健推断、时间验证、交易映射和中美股市差异。它不把“算出了系数”当成模型成功，也不把 HAC 标准误当成修复模型偏差的工具。
>
> **案例说明**：文中的情境和代码是教学模板；没有附带真实行情、数据版本、代码运行报告或实证结果。示例中的窗口与滞后阶数不是市场通用参数。

## 目录

1. [先定义 OLS 在估计什么](#1-先定义-ols-在估计什么)
2. [OLS 推导与它保证的边界](#2-ols-推导与它保证的边界)
3. [金融数据让 OLS 失效的路径](#3-金融数据让-ols-失效的路径)
4. [推断：HC、HAC 和聚类标准误](#4-推断hc-hac-和聚类标准误)
5. [预测验证与模型比较](#5-预测验证与模型比较)
6. [诊断流程与修复选择](#6-诊断流程与修复选择)
7. [Python 最小实现](#7-python-最小实现)
8. [A 股与美股的专门检查](#8-a-股与美股的专门检查)
9. [从回归分数到可交易组合](#9-从回归分数到可交易组合)
10. [放行清单与常见误区](#10-放行清单与常见误区)

## 1. 先定义 OLS 在估计什么

最一般的线性回归写作：

\[
y_i = x_i^\top\beta + \varepsilon_i,\qquad i=1,\ldots,n.
\]

“线性”指模型对参数 \(\beta\) 线性，不要求原始变量只能以直线进入模型。平方项、交互项、样条基函数和预先定义的状态交互都可以在线性参数框架内估计。反过来，OLS 也不会自动把相关性变成因果关系。

先固定估计对象：

| 任务 | 因变量与样本单位 | OLS 可回答什么 | 不能自动回答什么 |
|---|---|---|---|
| 因子暴露 | 资产超额收益对因子收益 | 给定因子定义和窗口下的线性暴露估计 | 因子有因果作用、暴露永久不变 |
| 收益预测 | 决策时点之后、持有期明确的收益 | 条件均值预测基线 | 预测误差小就能盈利 |
| 横截面定价 | 同一决策日多个证券的未来收益 | 当日截面关系或排序信号 | 把所有股票日当独立样本 |
| 风险模型 | 收益对市场/行业/风格因子 | 条件风险暴露估计 | 误差协方差正确或极端风险可控 |
| 事件研究 | 事件后异常收益对处理变量 | 特定识别假设下的条件差异 | 仅凭回归就识别了因果效果 |

对于收益预测，明确 \(t\) 时刻信息集 \(\mathcal F_t\)、预测期 \(h\)、收益口径、下单可用时点和标签成熟时点。若用未来 20 日收益作为 \(y_t\)，训练记录必须等到第 20 日结果可用之后才能进入训练；相邻标签共享未来收益区间时，也不能按行随机拆分。

## 2. OLS 推导与它保证的边界

矩阵形式为 \(y=X\beta+\varepsilon\)，其中截距若需要应显式包含在 \(X\) 中。最小化残差平方和：

\[
S(\beta)=(y-X\beta)^\top(y-X\beta).
\]

求导并令梯度为零，得到正规方程 \(X^\top X\hat\beta=X^\top y\)。若设计矩阵满列秩：

\[
\hat\beta_{OLS}=(X^\top X)^{-1}X^\top y.
\]

如果 \(E[\varepsilon\mid X]=0\)，则 \(E[\hat\beta\mid X]=\beta\)。这个条件期望为零（外生性）是无偏性的关键。严重共线性会令估计不稳定；完全共线会使正规方程不可唯一求解。正规方程的数值解存在，不表示变量具有经济解释，也不表示预测能跨时期复现。

Gauss–Markov 定理的经典结论需要线性设定、外生性、满秩，以及条件同方差、无误差自相关等条件；在这些条件下，OLS 是线性无偏估计量中方差最小者。误差正态不是 OLS 无偏或 Gauss–Markov 结论的必要条件；正态性常用于小样本经典 t/F 推断的精确分布结果。金融残差非正态时，仍需处理尾部与不确定性，但不能把所有结论混成一个“OLS 假设”。

## 3. 金融数据让 OLS 失效的路径

| 失效路径 | 为什么会出错 | 典型表象 | 能区分原因的检查 |
|---|---|---|---|
| 前视与幸存者偏差 | 未来修订值、最终成分股、退市遗漏改变样本 | IS 极好，前向期骤降 | 重放历史数据版本、历史证券池、标签与决策时钟 |
| 估计对象错配 | 用收盘后已知变量解释收盘前决策，或把价格水平回归当收益预测 | 系数“显著”但不可执行 | 按真实公开/接收/交易时点重建特征 |
| 内生性/遗漏变量 | \(E[\varepsilon\mid X]\ne0\)，系数收敛到错误对象 | 控制变量一加一减，系数翻转 | 经济机制、先后关系、替代变量、工具变量有效性论证 |
| 多重共线性 | \(X^\top X\) 病态，小样本中的方向难区分 | 系数巨大、符号不稳、标准误高 | 条件数、相关结构、逐期系数和重采样稳定性 |
| 异方差/相关误差 | 经典协方差公式错，效率和推断受影响 | t 值偏乐观，波动聚集 | 残差图、ACF、ARCH 检查，稳健协方差敏感性 |
| 非平稳和结构突变 | 一个常数 \(\beta\) 混合不同机制 | 全样本拟合看似一般，分段符号相反 | 滚动估计、断点/状态切片、前向窗口比较 |
| 极端点和数据错误 | 平方损失放大大残差；单位错、拆股未调尤甚 | 少数点决定整个回归 | 原始记录核验、杠杆/影响诊断、含/不含事件敏感性 |
| 规格搜索和多重试验 | 反复选变量/窗口后仍用同一数据报告显著性 | p 值偏小、赢家脆弱 | 完整试验台账、嵌套选择、锁定外层时期 |

杠杆值、学生化残差和 Cook 距离可以定位有影响的点；它们只是调查提示，不是删点规则。收益极端值可能正是策略需要承受的风险。Winsorize、稳健损失或删除事件会改变估计问题，必须说明经济理由并报告敏感性结果。

时间序列中的滚动窗口共享过去观测，不等于自动发生泄漏。真正要问的是测试预测是否使用了决策时点不可得的信息、训练标签是否跨入测试结果期，以及验证方案是否模拟了拟部署的预测过程。单纯把“特征窗口重叠”称为泄漏会误诊；相邻未来收益标签重叠、全样本预处理和随机时间切分则需要严格检查。

## 4. 推断：HC、HAC 和聚类标准误

OLS 系数和系数协方差是两件事。异方差稳健或 HAC 协方差通常改变标准误、置信区间和检验，不改变同一设计矩阵下的 \(\hat\beta_{OLS}\)。它不会消除遗漏变量偏差、前视偏差、结构断点或错误标签。

通用 sandwich 形式为：

\[
\widehat{\mathrm{Var}}(\hat\beta)=
(X^\top X)^{-1}\,\widehat S\,(X^\top X)^{-1}.
\]

在独立观测但异方差时，HC 类估计构造 \(\widehat S\)；小样本高杠杆情形可比较 HC2/HC3。按时间相关的序列，Newey–West HAC 在给定滞后和核权重下估计长期协方差：

\[
\widehat S=\widehat\Gamma_0+
\sum_{\ell=1}^{L}w_\ell(\widehat\Gamma_\ell+\widehat\Gamma_\ell^\top),
\quad
\widehat\Gamma_\ell=\sum_{t=\ell+1}^{n}x_t\hat\varepsilon_t\hat\varepsilon_{t-\ell}x_{t-\ell}^\top.
\]

这里的 \(L\) 与核权重属于推断设定，应按采样频率、持有期重叠和依赖衰减做敏感性分析。它不是“选择一个滞后就保证稳健”。短序列、大滞后、强结构变化会让渐近近似不可靠。

面板中同一证券内误差持续相关时，常需按证券聚类；共同日期冲击还可能需要日期维聚类或适用的双向聚类方法。少数聚类时常规聚类渐近标准误也不可靠，要报告聚类数并考虑小样本修正或适合设计的 wild cluster bootstrap。聚类标准误只改变不确定性估计，不会自动修复证券选择偏差或内生性。

## 5. 预测验证与模型比较

预测目标必须与部署一致。以日期排序，用扩展窗或滚动窗前向拟合；面板横截面研究应按共同决策日期整组切分，而非把同一天的股票随机分给训练和测试。对未来区间标签，训练集标签的可用时间必须早于测试决策时点；按标签信息区间 purge，必要时加 embargo。若模型选择和阈值调节消耗了验证集，该验证集就不再是最终测试集。

评价要分层报告：

1. **拟合和残差**：训练窗 R²、残差均值/尺度、ACF、异方差、杠杆和异常输入；训练 R²只用于诊断，不作泛化证据。
2. **时间外预测**：每期 RMSE/MAE、基准预测比较、方向/排序指标、稳定区间；样本外 R² 可以为负，表示不如指定基准。
3. **参数稳定性**：窗口、制度阶段、行业/规模/流动性切片；报告系数及区间变化，避免只展示全样本平均。
4. **交易结果**：若信号用于投资，重建下单和成交，再扣佣金、税费、价差、冲击、借券和融资；同一份测试数据不能同时挑阈值和宣称最终绩效。

日度 Sharpe、t 值和普通 IID 置信区间不能直接沿用在重叠持有期或强序列相关结果上。按“每一行股票样本”计算标准误也会把共同市场冲击伪装成大量独立证据。

## 6. 诊断流程与修复选择

按下面顺序排查，避免先换模型掩盖上游错误：

1. **复现目标**：冻结预测对象、期限、时点、单位和基准；确认收益和公司行为的账本口径。
2. **复查数据**：核对历史证券池、退市、公告初值/修订、停牌、公司行动、时区和可交易性。
3. **重做切分**：严格前向测试；训练标签成熟后才能训练；所有标准化、筛选、变换只在训练折拟合。
4. **读残差和设计矩阵**：看自相关、波动聚集、条件数、影响点、分段系数和状态稳定性。
5. **只修相应问题**：HAC用于误差相关推断；Ridge用于降低共线系数方差但增加偏差；交互/样条改变函数形式；IV/DID等要求独立的识别假设；结构断点可选择重设窗口，但缩窗会增加估计方差。
6. **重新做嵌套验证**：在内层定规格，外层留出未来时期；锁定后才看最终结果，并登记全部尝试。
7. **将预测接入实际策略账本**：风险限额、成本、交易限制和容量均需进入结果。若只在理想成交下有效，结论应是执行假设失效。

### 结果表建议

| 版本 | 数据/证券池 | 训练结束日 | 验证/测试期 | 规格与变换 | 标准误方法 | OOS 预测指标 | 成本后组合结果 | 失败/放行原因 |
|---|---|---|---|---|---|---|---|---|

## 7. Python 最小实现

以下仅示范训练窗拟合及 HAC 推断的接口。必须先在外部构造正确的点时切分；代码不会替你完成 PIT 数据治理、purging 或市场成交模拟。

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm

# X_train / y_train 只包含训练截止时已可用的样本。
# X_test 必须晚于训练期，且训练标签在对应决策时点前已成熟。
# 两个特征表须属于同一单序列、按时间递增，使用相同列和顺序，且不含常数特征或截距列。
if not isinstance(X_train.index, pd.DatetimeIndex) or not isinstance(X_test.index, pd.DatetimeIndex):
    raise TypeError("single-series train/test indexes must be DatetimeIndex")
if (not X_train.index.is_unique or not X_train.index.is_monotonic_increasing
        or not X_test.index.is_unique or not X_test.index.is_monotonic_increasing):
    raise ValueError("train/test timestamps must be unique and increasing")
if X_train.index.tz != X_test.index.tz:
    raise ValueError("train and test timestamps must use the same timezone")
if len(X_train) == 0 or len(X_test) == 0 or X_test.index[0] <= X_train.index[-1]:
    raise ValueError("test data must be non-empty and strictly later than training data")
if not isinstance(y_train, pd.Series) or not y_train.index.equals(X_train.index):
    raise ValueError("y_train must be a Series aligned to X_train")
if not X_train.columns.equals(X_test.columns):
    raise ValueError("X_train and X_test must have identical feature columns")
if not X_train.columns.is_unique:
    raise ValueError("feature column names must be unique")
if (not np.isfinite(X_train.to_numpy(dtype=float)).all()
        or not np.isfinite(X_test.to_numpy(dtype=float)).all()
        or not np.isfinite(y_train.to_numpy(dtype=float)).all()):
    raise ValueError("features and training targets must be finite and complete")
if X_train.nunique(dropna=False).le(1).any():
    raise ValueError("remove constant features before adding the intercept")
X_train_c = sm.add_constant(X_train, has_constant="add")
X_test_c = sm.add_constant(X_test, has_constant="add")
if not X_train_c.columns.equals(X_test_c.columns):
    raise ValueError("training and test design matrices are not aligned")
if np.linalg.matrix_rank(X_train_c.to_numpy(dtype=float)) < X_train_c.shape[1]:
    raise ValueError("training design matrix is rank deficient")
if len(y_train) <= X_train_c.shape[1]:
    raise ValueError("training sample must exceed the number of design-matrix columns")
if (not isinstance(hac_lag, int) or isinstance(hac_lag, bool)
        or hac_lag < 0 or hac_lag >= len(y_train)):
    raise ValueError("hac_lag must be a non-negative integer smaller than the training sample")

ols = sm.OLS(y_train, X_train_c, missing="raise").fit()
hac = ols.get_robustcov_results(
    cov_type="HAC",
    maxlags=hac_lag,  # 根据频率/重叠持有期预先选择，并做敏感性分析
)
prediction = ols.predict(X_test_c)
if not np.isfinite(np.asarray(prediction, dtype=float)).all():
    raise ValueError("predictions must remain finite")
```

`prediction` 仅是预测值；`hac` 是推断对象，二者不能混为一谈。面板的簇结构不同于单序列 HAC；使用聚类接口前应按实际公司和日期依赖设计相应协方差估计，并记录软件版本。本节代码已用合成时间序列核验语法与基本拟合/预测路径；它没有用真实行情、点时数据或交易账本验证，也不是生产实现。

## 8. A 股与美股的专门检查

| 环节 | A 股 | 美股 | 两边共同要求 |
|---|---|---|---|
| 证券池 | 保存历史上市/退市、风险警示、停复牌及板块身份；不要只用当前成分股 | 保存退市证券、不同 share class、ADR 映射和历史成分 | 证券身份用稳定主键和有效期，不只靠代码/ ticker |
| 基本面 | 区分报告期、法定披露时点、实际接收时间和更正/重述版本 | 区分 filing 接收时间、报告期、修订申报及数据商加工延迟 | 只用决策时点已可用版本，不能回填最终财报值 |
| 行情与成交 | 停牌、涨跌停、上市初期限制、T+1 和规则变化影响可成交性与收益路径 | 多交易场所、盘前盘后、LULD/停牌、订单路由和借券影响成交 | 交易规则按市场、品种、日期版本化；触价不等于成交 |
| 收益与公司行动 | 拆并股、现金分红、送转、配股、退市结算需进入账户账本 | 拆股、股息、并购、退市、ADR 比率变化需进入账本 | 复权序列用于特定统计用途，真实组合收益从现金/持仓重放 |
| 推断与共同冲击 | 行业、指数成分和交易制度可造成同日/板块相关性 | 行业、ETF/指数和宏观公告造成共同冲击 | 面板标准误反映样本聚类，不能把股票行数当独立样本数 |

A 股和美股的交易日、公告时钟、价格限制、做空可用性、费用及流动性差别不能通过一个“市场虚拟变量”概括。用统一研究框架，但为市场、交易所、产品、账户和生效日期保存各自的数据与成交配置。规则数值需要在研究时核验，不在模型里写死。

## 9. 从回归分数到可交易组合

预测 \(\hat\mu_{i,t}\) 必须说明单位、期限和置信度。横截面排序常比单点收益幅度稳定，但排序指标也不是净收益。组合构建仍需独立考虑协方差、行业/风格集中度、毛净敞口、换手、交易成本和可成交量。

对长短仓，加入做空可用性、借券费和召回；对只做多策略，目标权重受现金、持仓、交易单位和不可交易状态约束。信号形成后再计算可交易目标，在实际可执行时点提交订单。不能用未来成交量、未来停牌状态或收盘后的信息来决定当时的仓位。

至少比较：不交易、历史均值/截面均值、简单线性基准、正则化基准和原策略；对每个版本报告成本前预测改进与成本后组合增量。若加了复杂规格却没有稳定的时间外增量，就保留简单基准。

## 10. 放行清单与常见误区

- [ ] 估计对象、单位、频率、信息时点、标签成熟时点和比较基准冻结。
- [ ] 历史证券池、退市、修订、公司行为和市场规则可重放。
- [ ] 同日面板整组切分，标签重叠经过 purge；调参与最终测试隔离。
- [ ] 外生性论证与预测边界明确；多重共线、影响点、结构断点和残差相关有诊断。
- [ ] 标准误方法匹配序列/面板依赖，并注明其不能修复识别偏差。
- [ ] 报告多个时间段与切片的预测和组合证据，包括成本、成交和容量。
- [ ] 所有尝试、失败版本、阈值和最终锁定时间可审计。

**必须纠正的误区：**异方差不会自动让 OLS 系数有偏，但会让经典标准误和效率结论失效；HAC 不会让模型预测更好；滚动特征共享过去数据本身不等于泄漏；Cook 距离阈值只是经验提示；训练集高 \(R^2\)、系数显著和收敛成功都不是部署证据。

更广的研究顺序见[项目 README](../README.md)；金融横截面验证与标签边界另见[信号提取原理](../提取可复用信号/原理.md)和[回测可信度实务](../检验回测可信度/实务.md)。

## 参考资料

> 以下文献用于追溯异方差稳健与 HAC 推断方法；稳健标准误不修复错设模型、内生性或预测失效。

- White (1980), [异方差稳健协方差矩阵估计（A Heteroskedasticity-Consistent Covariance Matrix Estimator）](https://doi.org/10.2307/1912934).
- Newey & West (1987), [异方差与自相关稳健协方差矩阵估计（A Simple, Positive Semi-Definite Heteroskedasticity and Autocorrelation Consistent Covariance Matrix）](https://doi.org/10.2307/1913610).


## 英文阅读指南（English reading guide）

Sections 1–4 cover the estimand, OLS assumptions, financial failure paths, and robust inference. Sections 5–10 address time validation, implementation, A-share/U.S. checks, portfolio translation, and release criteria.
