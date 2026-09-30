# 量化研究与金融模型失效诊断手册

> 一套以中文为主、面向 A 股与美股研究的量化方法手册：从历史信息重建、模型验证，到成本后账本与失效复盘。

**适合读者**：量化研究员、金融机器学习实践者、策略审查与风险团队、希望系统学习回测的开发者  
**语言（Language）**：[English overview / 英文导读](#english-overview) · 中文正文，模型专题附英文摘要  
**内容形式**：Markdown 手册 · 五阶段研究流程 · 17 篇附英文导读与方法参考资料的 Why My 模型失效报告 · 可复用审查清单与案例演练  
**项目形态**：文档知识库；当前不附真实行情数据、可运行策略包或券商接口

<a id="english-overview"></a>

## 英文导读（English overview）

This repository is a Chinese-first, bilingual-accessible handbook for quantitative research in **China A-shares and U.S. equities**. Its main content is Chinese; technical reports include English summaries and reading guides, while major terms are presented with Chinese and English names.

The material follows a five-stage research lifecycle:

1. **Financial data** — [workflow](./金融数据整理/流程.md) · [principles](./金融数据整理/原理.md): reconstruct what was knowable and tradable at each decision time.
2. **Reusable signals** — [workflow](./提取可复用信号/流程.md) · [methods](./提取可复用信号/原理.md): define targets, features, and time-aware evidence.
3. **Strategy design** — [workflow](./建立策略解释/流程.md) · [practice](./建立策略解释/实务.md): convert predictions into positions, portfolios, and risk limits.
4. **Backtest review** — [workflow](./检验回测可信度/流程.md) · [practice](./检验回测可信度/实务.md): audit leakage, selection bias, fills, costs, and account-level results.
5. **Deployment and monitoring** — [workflow](./部署与持续监督/流程.md) · [practice](./部署与持续监督/实务.md): version releases, monitor risk, and define stop/rollback actions.

Start with the [research audit checklist](./量化研究审查清单.md), then use the [Why My Model Failed index](./Why%20my%20Model%20failed/README.md) to find model-specific failure mechanisms. The five-stage links below lead to the full Chinese workflow and practice guides. The English summaries at the beginning of each model report describe its scope, assumptions, and reading route.

**Scope:** This is a documentation and research-review project. It does not include licensed market data, a maintained trading package, broker connectivity, or verified live performance. Examples without reproducible point-in-time data, code, and an execution/accounting ledger are methodological illustrations, not empirical return claims.

**Core terms / 术语对照：** 点时数据（point-in-time data）；前视偏差（look-ahead bias）；幸存者偏差（survivorship bias）；时间外验证（out-of-time validation）；成本后收益（net-of-cost return）；组合与账户账本（portfolio and account ledger）；研究流程（pipeline）；向前滚动验证（walk-forward validation）；重叠样本区间清除（purging）；隔离期（embargo）；组合清除交叉验证（combinatorial purged cross-validation, CPCV）；回测过拟合概率（probability of backtest overfitting, PBO）；膨胀修正夏普比率（deflated Sharpe ratio, DSR）。

## 先从这里开始

| 你现在要做什么 | 从这里开始 |
|---|---|
| 快速判断一份策略研究是否站得住 | [量化研究审查清单](./量化研究审查清单.md) |
| 通过一个反例练习完整复盘 | [“财报惊喜”虚假 Alpha 案例演练](./量化研究案例演练：一条财报惊喜策略如何制造虚假Alpha.md) |
| 模型指标看起来不错，实盘环节却出问题 | [按失败症状定位模型专题](./Why%20my%20Model%20failed/README.md#按失败症状快速定位) |
| 系统走完一次量化研究 | [五阶段研究流程](#五阶段研究流程) |
| 不确定模型为什么失效 | [Why My Model Failed 模型诊断索引](./Why%20my%20Model%20failed/README.md) |
| 先理解量化研究的基本问题 | [量化交易导入](./量化交易导入.md) |
| 需要机器学习选型或交易映射 | [算法与专题映射](./机器学习算法与量化专题映射.md) · [机器学习量化流程](./机器学习算法量化交易流程详细.MD) |
| 想改进文档或报告勘误 | [贡献指南](./CONTRIBUTING.md) |

## 为什么整理这套手册

很多研究不是败在算法不够复杂，而是败在模型之前或之后：历史字段当时还不可得，样本只留下今天仍存续的证券，标签跨过训练边界，随机切分让同一事件同时进入训练和测试，或者模型信号根本不能按回测价格成交。

本项目把这些环节接成一个可复核工作流，并提供可直接使用的[审查清单](./量化研究审查清单.md)和带答案的[端到端案例演练](./量化研究案例演练：一条财报惊喜策略如何制造虚假Alpha.md)：

```mermaid
flowchart LR
    A[点时数据与证券池] --> B[研究目标、特征与标签]
    B --> C[模型与时间外验证]
    C --> D[仓位、组合与风险]
    D --> E[订单、成交与成本后账本]
    E --> F[生产监控、暂停与复盘]
    F -. 失效证据回流 .-> A
```

模型输出只是中间结果：**拟合分数不是预测证据，预测指标不是投资组合，目标仓位不是成交，回测收益也不是实盘业绩。**

## 五阶段研究流程

| 阶段 | 核心问题 | 流程 | 原理或实务 | 交付物 |
|---|---|---|---|---|
| 1. 金融数据整理 | 决策时点能看到什么？证券身份、字段版本和市场状态能否重建？ | [流程](./金融数据整理/流程.md) | [原理](./金融数据整理/原理.md) | 点时数据集、时间口径、历史证券池、质量审计 |
| 2. 提取可复用信号 | 特征和标签是否对齐？信号能否在新时期复现？ | [流程](./提取可复用信号/流程.md) | [原理](./提取可复用信号/原理.md) | 研究假设、目标定义、验证方案、适用边界 |
| 3. 建立策略解释 | 信号如何转成方向、仓位、组合和风险？ | [流程](./建立策略解释/流程.md) | [实务](./建立策略解释/实务.md) | 仓位映射、组合规则、风险预算和失效条件 |
| 4. 检验回测可信度 | 历史结果是否经得起点时、成交、费用与稳健性检查？ | [流程](./检验回测可信度/流程.md) | [实务](./检验回测可信度/实务.md) | 可复算账本、偏差审计、锁定测试期、放行结论 |
| 5. 部署与持续监督 | 上线后如何监控、暂停、回滚和重新验证？ | [流程](./部署与持续监督/流程.md) | [实务](./部署与持续监督/实务.md) | 版本工件、风险闸门、告警动作和运行审计 |

### 三条阅读路线

- **初学者**：先读[量化交易导入](./量化交易导入.md)，再按上表从数据走到部署。
- **正在做信号/策略**：先完成[审查清单](./量化研究审查清单.md)，定位缺口后回到对应专题。
- **正在排查模型失败**：按模型类别进入 [Why My Model Failed 索引](./Why%20my%20Model%20failed/README.md)，先找失效机制，再读同篇的验证、修复与中美市场适配。

## Why My Model Failed：模型失效诊断系列

该系列把方法推导与失败复盘结合，覆盖线性回归、分类、信息准则、正则化、非线性回归、SVM、树与提升、深度学习、金融时序、PCA/聚类、异常检测、排序推荐、因果推断、组合优化、强化学习和重采样。

每篇围绕相同的问题链展开：**目标与样本 → 失效机制 → 模型假设 → 诊断与研究流程（pipeline） → A 股/美股差异 → 放行或停用条件**。分类、OLS/HAC、信息准则和重采样的研究流程均已融入对应报告；系列目录只维护 Why My 文章和索引。五阶段主线与 17 篇模型报告以中文为正文，并提供英文导读；专题报告还附英文摘要和阅读指南，关键术语按“中文（English）”呈现，方便非中文读者快速定位。

👉 [浏览完整模型目录](./Why%20my%20Model%20failed/README.md)

## 贯穿全项目的研究口径

### 时间与点时信息

区分事件发生、公开、供应商/系统取得、决策、下单、成交及标签成熟时间。决策只能使用当时可得的信息；财报期末不等于公开时间，复权值不等于当时可交易价格。

### 验证与证据

预处理和调参只能使用对应训练窗口。按时间、证券、事件与标签信息区间设计切分。Purged CV/CPCV 适用于特定稳健性问题，不等同于真实的未来顺序运行。保留试验记录，锁定最终测试期。

### 指标、策略与账户结果

AUC、IC、R²、显著性、风险估计、组合收益和真实成交表现是不同证据，不能互相替代。回测须说明基准、仓位、订单、未成交、费用、融资/借券及账户现金流口径。

### A 股与美股

分别处理交易时钟、证券状态、公司行为、可交易约束、费用、借券和历史证券池；跨市场迁移是需要单独验证的研究假设。规则及费率需按证券、场所、账户和生效日期核验。

## 项目范围与证据边界

这是研究方法与审查框架，不是实盘信号服务。仓库不含真实行情数据、独立 Python 包、券商接口或可复现的策略收益。文章中的示例若未同时给出数据源、样本区间、点时版本、代码与可复算账本，应视为方法说明或假设案例，不应当作实证收益。

使用本项目进行研究时，应结合目标数据供应商、市场制度、交易频率与账户约束；引用外部规则时保留来源、生效时间和适用对象。发现错误或希望补充内容，请先看[贡献指南](./CONTRIBUTING.md)。

## 项目目录

```text
README.md                         项目首页与总导航
CONTRIBUTING.md                  内容贡献与勘误规范
量化研究审查清单.md               可复用的研究审查与放行模板
量化研究案例演练*.md              端到端反例与诊断答案
量化交易导入.md                   入门说明
机器学习算法与量化专题映射.md      算法选型入口
机器学习算法量化交易流程详细.MD   模型到交易的补充手册
金融数据整理/                    数据流程与原理
提取可复用信号/                  信号流程与原理
建立策略解释/                    策略流程与实务
检验回测可信度/                  回测流程与实务
部署与持续监督/                  生产流程与实务
Why my Model failed/             模型失效诊断系列及索引
```

本仓库目前没有附带 `LICENSE` 文件。公开复用或分发前，许可范围仍需由作者明确；仓库获得星标本身不代表授予复用权限。
