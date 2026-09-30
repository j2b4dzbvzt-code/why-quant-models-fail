# 金融排序与推荐模型为何失效（Why My Ranking and Recommender Model Failed）

> **英文摘要（English summary）：** This report distinguishes pointwise, pairwise, and listwise ranking from recommender systems built on exposure and interaction data. It explains query-group, candidate-pool, label, and exposure bias; evaluates ranking metrics separately from portfolio outcomes; and provides A-share/U.S. controls for time-aware validation and execution.

> **案例等级**：方法报告。排序分数、推荐分数和未来收益并非同一个量；本文不提供任何市场实证收益结论。

## 1. 失败现场：NDCG/IC 提高，Top-K 组合却变差

排序模型要在一个查询组内比较对象；股票研究中的查询组通常是同一决策日的可投资证券截面。推荐系统则预测用户对物品的交互/曝光/偏好。把两者都称为“模型给股票打分”会掩盖估计对象差异。

- **Pointwise** 对每个证券独立拟合收益/类别损失，容易优化行级误差而忽略当日排序。
- **Pairwise** 优化同组证券对的顺序，证券对数量可能呈二次增长，同一日期内高度相关。
- **Listwise** 直接优化整列排序指标的近似，目标依赖候选池和标签分布。
- **协同过滤/矩阵分解** 从历史交互学习主体—资产关系；没有收益或真实曝光标签时，重建分数不是 alpha。

## 2. 排序失败：查询组、标签和候选池错配

若训练时把所有日期混成一个排序组，模型可能学到资产身份或跨日期基率，却没有解决“今天谁排在前面”。若按股票行随机切分，同一日期/同一事件几乎原样进入训练和测试。若以未来收益分位分标签，分位阈值必须在同一日期的可投资集合内按预定规则形成，不能用最终成分名单或未来可得信息。

即使组内排名正确，模型也可能只挑到高 beta、行业、规模、流动性或事件风险暴露。NDCG、Spearman、Rank IC、Top-K precision 都要说明标签、基准和组合构造；不同日期股票数量不同，组加权方式会显著改变总体指标。

## 3. 推荐系统失败：未曝光不等于负反馈

金融“推荐”常把用户/基金/公司/行业与股票建成交互矩阵。缺失格子可能是从未展示、无数据、不可投资或真实不感兴趣，不可一概设为零。基金持仓披露滞后、新闻覆盖、分析师覆盖和客户点击存在选择性观测；模型容易重现曝光机制、热门资产和覆盖偏差。

时间泄漏常来自先构造全期交互矩阵再切分、以未来交易行为识别资产相似性、用当前映射连接历史 ticker，或把最终修订持仓当作当期可见。矩阵稀疏和冷启动会使随机留出表现看似稳定、留出公司/资产就崩溃。

## 4. 评估必须逐层连接

1. **组内预测/排序**：每个日期报告 NDCG@K、Precision@K、Rank IC/相关指标及有效证券数，分行业、规模和流动性报告。
2. **组合暴露**：检查行业、国家/市场、beta、规模、动量、价值和流动性集中度；与简单 Top-K、等权、行业中性和因子基线比较。
3. **统计不确定性**：以日期/市场状态区块评估差异；成对股票数量不能当独立样本数。
4. **经济结果**：锁定 rebalance 时间、仓位、卖出/持有规则、成本和容量，形成订单/成交账本；报告净收益、换手、回撤与风险。
5. **推荐结果**：若目标为曝光后的交互，要做时间外和按主体/物品冷启动留出；考虑曝光概率/反事实估计的假设，并明确目标不是收益预测。

特征重要性或 embedding 最近邻能解释模型内部相似度，不证明共同经济机制或因果关系。Permutation 也要在有效查询组内按依赖结构做，不能随意打乱证券行。

## 5. A 股与美股的专门适配

**A 股**：每个排序组应以当日点时可投资集合定义；停牌、价格限制、上市阶段、风险警示和 T+1 等约束影响 Top-K 的真实组成。历史行业、板块、指数成分和证券代码映射要保留生效区间。若卖出腿无法卖空或买入腿无法成交，理论长短排名不能直接报告为可实现 long-short。

**美股**：候选池保留退市、ticker/share-class/ADR 历史映射；财报发布跨常规时段要把决策与成交时点分开。Bottom-K 策略需逐日验证 locate/borrow/recall 与费用；多场所、延长时段和流动性差异应进入订单仿真。

**跨市场**：排序的标签尺度、货币、交易日、股票数量和行业映射需先规范。美股假日与 A 股时区错位，按日期标签直接拼成一个 query 会改变查询定义。先独立训练/评估，再检验共享模型的迁移价值。

## 6. 排序/推荐的完整研究流程（pipeline）

1. 定义主体、物品、query group、时间点、可投资集合和排序标签；把“交互推荐”与“收益排序”分开。
2. 建立点时历史证券/主体映射和曝光/持仓/报价数据；缺失保留原因，不默认负类。
3. 以日期为 outer fold，切分后再生成 group/pair/list 样本；按标签信息区间 purge。
4. 先做点式线性基准、简单截面因子排序、行业中性 Top-K；再比较 pairwise/listwise 或矩阵分解。
5. 内层选择损失、K、正则、负采样、曝光校正和特征；阈值/组合优化不得看 outer test。
6. 按组报告排序质量和稳定性，检查候选池变化、冷启动与特征贡献。
7. 在成本、换手、风险和市场约束下形成组合结果；分市场报告，并保留未成交订单。
8. 生产监控候选覆盖、重复身份映射、分数分布、排序集中度、订单拒绝和持仓暴露。

## 7. 何时停止

若提升指标只来自候选池选择、推荐只复现历史曝光、Top-K 收益依赖不可交易尾部、排名跨时间不稳定或净收益低于朴素基准，不能以“个股相关性高”继续放行。推荐/排序的好用性应按其真正任务判断，而不是把每个高分都命名为投资信号。

## 参考资料

> 以下文献用于追溯成对排序与排序指标优化方法；搜索排序论文不直接证明金融横截面收益有效。

- Burges, Ragno & Le (2006), [非光滑代价函数下的排序学习（Learning to Rank with Nonsmooth Cost Functions）](https://proceedings.neurips.cc/paper/2006/hash/af44c4c56f385c43f2529f9b1b018f6a-Abstract.html).
- Li, Wu & Burges (2007), [McRank：用多分类与梯度提升进行排序学习（Learning to Rank Using Multiple Classification and Gradient Boosting）](https://proceedings.neurips.cc/paper/2007/hash/b86e8d03fe992d1b0e19656875ee557c-Abstract.html).


## 英文阅读指南（English reading guide）

Sections 1–3 distinguish ranking from recommendation and explain group, label, and exposure failures. Sections 4–7 connect ranking metrics to portfolios, market-specific constraints, an end-to-end workflow, and stop conditions.
