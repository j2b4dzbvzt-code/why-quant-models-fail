# 金融深度学习模型为何失效（Why My Deep Learning Model Failed）

> **英文摘要（English summary）：** This report diagnoses MLP, CNN/TCN, recurrent networks, Transformers, and pretrained text models in financial research. It focuses on effective sample size, overlapping windows, temporal leakage, architecture-specific failure modes, calibration, seed sensitivity, and deployment controls. Numerical training success does not establish out-of-time or cost-adjusted value.

> **案例等级**：方法诊断模板，不含实证结果。本文覆盖 MLP、CNN、RNN/GRU/LSTM、Transformer 和预训练文本模型；这些网络结构回答不同问题，不能笼统称为一个“深度模型”。

## 1. 失败现场：网络收敛，市场外推却不存在

深度网络能学习非线性表示，但市场数据的有效信息量远小于行数。一个日频面板有上百万股票日记录，并不代表拥有上百万独立样本：同一日期共振、证券反复出现、滑动窗口重叠、事件标签共享未来区间。若数据时钟、标签和验证设计错误，网络容量只会更充分地记忆错误。

常见失败表象：训练损失持续下降而前向表现波动；不同种子排名完全变；某个批次大小或归一化方式决定结果；窗口一平移模型就失效；验证表现依赖于随机切分；文本模型“预知”历史事件结局；离线预测无法映射到稳定净收益。

## 2. 时间窗与目标定义的特有风险

序列输入 \(X_t=[x_{t-L+1},\dots,x_t]\) 必须只包含决策时刻可获得的数据，目标 \(y_t\) 需从决策后可执行时间开始。很多细小实现错误会造成未来信息：包括在窗口尾部拼进收盘后统计量、用未来窗口均值归一化、padding mask 错位、目标列偏移一根 bar、把尚未成熟标签用于训练、训练切分后再构造重叠滑窗。

应显式保存每个样本的 `decision_at`、`feature_available_at`、`label_start_at`、`label_end_at`、`label_available_at`。若一个事件产生多条序列窗，切分要按事件/日期边界完成后再生成窗口，或按完整信息区间 purge，不能先生成高度重叠窗口再随机分行。

## 3. 不同架构有不同故障机制

| 架构 | 适用结构 | 专属风险 |
|---|---|---|
| MLP | 表格横截面/固定长度特征 | 过度参数化、输入尺度、时间与行业身份记忆；通常应先和线性/树基线比较 |
| CNN / TCN | 局部时序模式、可控因果卷积 | padding/卷积感受野跨越未来、平滑使不同事件变得相似 |
| RNN / LSTM / GRU | 有序序列状态 | 截断反传长度、状态重置、隐藏状态跨样本污染、长序列遗忘 |
| Transformer | 长程依赖、多变量/文本序列 | 因果 mask、注意力复杂度、训练语料时间穿越、位置与频率错配 |
| 自监督/预训练模型 | 大语料表示迁移 | 语料/权重已学到测试期新闻和公司结局，历史回测并非历史可部署模型 |

Dropout、weight decay、early stopping 和归一化不是无条件稳健开关。BatchNorm 可能让未来批次统计穿过边界；LayerNorm 通常按样本计算也不能修正特征构造本身的泄漏。训练时 `model.train()` 与推理 `model.eval()` 影响 dropout/BatchNorm，关闭梯度则是另一个设置；两类状态都要检查。

## 4. 损失、概率与金融效用

MSE 的最优预测是条件均值，Logloss 对数概率得分，排序损失优化相对次序；它们都不是自动的投资效用函数。网络对稀有极端标签的平均损失较小，也可能错过投资者最关心的尾部事件。类别采样和 class weight 会改变训练先验；用网络输出概率直接乘仓位可能使风险远超预算。

应把拟合目标、概率校准、政策阈值、组合权重和账户收益分层处理。校准器和温度参数也须在独立时间窗确定。若关注尾部风险，单独评估分位数、极端事件召回和压力情景，不能只展示平均 RMSE。

## 5. 诊断与时间验证

1. 先验证样本构造：随机抽样打印输入窗口、可得时间、目标区间与原始市场记录。
2. 做端到端时间置换/未来扰动检查：改变未来价格不应影响已形成的过去特征、预测和仓位。
3. 建立日期整组的外层前向切分；所有 scaler、词表、缺失处理和特征选择在训练折拟合。
4. 内层选择架构、层数、宽度、学习率、序列长度、batch、dropout、权重衰减和早停；不得用外层测试挑种子。
5. 记录多随机种子，但种子不是独立样本。报告均值/离散度、按时间块的不确定性、不同市场状态和未成熟标签队列。
6. 进行架构消融和基线比较：线性/岭回归、树模型、浅 MLP、序列网络逐步增加复杂度。复杂模型必须显示同窗外层增量。
7. 检查残差/分数稳定、概率校准、条件切片、批次大小、CPU/GPU 精度和 train/eval 一致性。
8. 将锁定预测送入真实交易规则和账户账本，分开报告成本前预测、成本后收益、风险、换手和容量。

## 6. A 股与美股的序列/文本差异

**A 股**：按交易所和产品时钟定义日内序列；停牌、价格限制、午休和夜间公告会造成非均匀有效步长。财报和公告保留披露、供应商接收、解析和更正时间；不能仅按报告期或网页日期建立序列。历史板块/证券状态和退市样本必须进入训练覆盖审计。

**美股**：把常规交易时段、盘前盘后、隔夜跳空、财报发布时间和多场所数据区分开。Share class、ADR、ticker 变化、拆股和退市影响实体连续性。预训练文本模型需证明权重、词典、embedding 或训练语料在测试期没有使用后续新闻；只限制输入新闻日期不能清除权重中的未来知识。

**共同要求**：不同市场的交易日不完全重合。跨市输入要按真正可用时间对齐，并记录币种、日历、价格精度和交易状态。只在一市场训练的表示迁移到另一市场是独立假设，需留出市场/时间做测试，而不是把市场编码送入模型后宣称泛化。

## 7. 深度模型训练与部署研究流程（pipeline）

1. 写出模型可见的信息集、样本窗、决策时点、目标成熟时点和网络输入/输出契约。
2. 用无泄漏检查与小型手算例验证时间窗、mask、label shift、单位和数据归一化。
3. 先冻结前向验证方案，再拟合训练折预处理、模型和早停；标签有重叠时执行 purging/embargo。
4. 对架构/参数设置有限搜索预算，登记失败版本；选中的模型在每个外层窗口独立评价。
5. 以种子、时间区块、行业/流动性、市场/regime 作稳健性切片，估计有效样本量而非报告原始窗口数。
6. 保存模型权重、特征顺序、词表/分词器、scaler、训练截止时间、库/驱动/精度版本和校准器。
7. 做离线重放、同输入重复推理、影子和小规模受限验证；监控数据漂移、批次依赖、延迟、订单和账本。

## 8. 放弃复杂网络的条件

若简单基线在未触碰的前向期同样好、深网增量低于其估计误差、不同窗口/市场/种子结论矛盾，或收益来自事后训练语料和不可执行成交，就没有理由保留复杂网络。更大的参数量和计算成本必须由稳定、净额、可复现的增量价值支付。

## 参考资料

> 以下文献用于追溯代表性架构的原始方法，不构成金融市场适用性或收益证据。

- Hochreiter & Schmidhuber (1997), [长短期记忆网络（Long Short-Term Memory）](https://doi.org/10.1162/neco.1997.9.8.1735).
- Vaswani et al. (2017), [注意力就是一切（Attention Is All You Need）](https://proceedings.neurips.cc/paper/2017/hash/3f5ee243547dee91fbd053c1c4a845aa-Abstract.html).


## 英文阅读指南（English reading guide）

Sections 1–4 explain temporal leakage, architecture-specific risks, and objective mismatch. Sections 5–8 cover diagnostics, market adaptation, training/deployment controls, and conditions for preferring a simpler model.
