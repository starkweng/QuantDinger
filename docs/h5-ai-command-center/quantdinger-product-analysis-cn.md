# QuantDinger 产品调研与 H5 AI 交易系统适配分析

## 结论

QuantDinger 值得作为 H5 AI 交易系统的产品骨架参考。它已经把一个量化系统拆成了清晰闭环：AI 资产分析、指标市场、指标 IDE、策略与实盘、交易机器人、券商账户、Agent Token。我们不应该只抄界面，而要抄它的产品分层和风险边界。

但 H5 系统不能被改造成一个“指标拜物教”的买卖点工具。H5 的核心是：市场状态识别、多周期结构、价量时空、均线成本/抵扣价推演、Keltner/ATR 波动约束、评分准入、资金管理、交易模式门控，以及“学会不下单”的纪律系统。

## QuantDinger 当前产品骨架

| 模块 | QuantDinger 现状 | 对 H5 的可抄价值 |
|---|---|---|
| AI Asset Analysis | 多市场选择、资产 watchlist、AI 分析入口、历史分析 | 改为 Research Mission Center，支持多 agent 分工、价格路径推演、60 分钟情景预案 |
| Indicator Market | 指标市场、我的指标、审核流程 | 改为 Strategy Registry，H5 是一个 strategy package，不是唯一策略 |
| Indicator IDE | Python 指标代码、AI 生成、代码质量检查、图表、回测 | 可直接学习“研究代码 -> 图表验证 -> 回测 -> 策略化”的工作台结构 |
| Strategy & Live | Live Overview、Strategy Management、策略列表、实盘监控 | 改为 Execution Command Center：SIGNAL_ONLY / SEMI_AUTO / AUTO_API 三模式 |
| Trading Bot | AI Smart Create、Grid、Martingale、Trend Following、DCA | 保留 bot 类型入口，但 H5 的默认价值是“先不下单”，bot 只在风险核批准后进入 |
| Live Broker Accounts | 加密交易所、Alpaca、IBKR、MT5 接入 | 对应 crypto + TradFi 统一账户层，覆盖币安股票、贵金属、原油、外汇 |
| Agent Tokens | 外部 AI agent token、scope、paper-only、审计 | 非常关键：作为多 agent research、回测、执行的权限边界 |

## 后端能力观察

QuantDinger 后端已经有适合复用的能力：

- `experiment/regime.py`：市场状态识别
- `experiment/scoring.py`：策略多因子评分
- `experiment/evolution.py`：参数变体生成
- `experiment/runner.py`：状态识别、批量回测、评分、最佳策略输出
- `agent_v1`：面向外部 agent 的稳定 API 层
- `BacktestService`：指标脚本回测
- `StrategyService`：策略创建、更新、运行状态管理
- `live_trading/*`：多交易所执行适配
- `safe_exec.py`：策略代码沙盒和安全检查

这说明我们不需要从空白开始。第一阶段应该把 QuantDinger 当作“自托管量化工作台”，在其上加 H5 的决策语义、风控核和多 agent 研究流程。

## H5 不应被抄丢的核心

H5 交易系统不是只有原则，也不是单个指标策略。它至少包括：

1. 市场状态：趋势、震荡、极端、转折、等待。
2. 多周期中轴：根据品种波动和交易时长动态选择，如日内可用 5m/15m。
3. 结构证据：2B、均线密集、均线发散、Keltner 轨道、ATR 波动、成交量/成交额异动。
4. 推演系统：不是预测价格，而是推演 A/B/C/D 多路径，提前准备不同操作。
5. 评分准入：分数达标后只代表允许进入下一步，不代表必须下单。
6. 资金管理：先知道最多亏多少，再讨论可能赚多少。
7. 模式门控：SIGNAL_ONLY、SEMI_AUTO、AUTO_API 必须由前端显式下命令。
8. 风控纪律：亏损上限、日内停止、连续错误冷却、kill switch、人工复核。
9. 不交易能力：没有高盈亏比和结构证据时，最佳动作就是不下单。

## 我们应该怎么抄

| 抄法 | 内容 | 处理方式 |
|---|---|---|
| 直接抄产品骨架 | 左侧导航、研究页、IDE、策略页、bot 页、账户页、agent token 页 | 用作 H5 Command Center 的第一版信息架构 |
| 深度学习后改造 | Experiment pipeline、Agent Gateway、paper-only、audit log、safe execution | 变成 H5 Risk Kernel 与 Research Agent API |
| 不直接照搬 | Martingale、AI 一键创建交易机器人 | 作为高风险模板保留，但默认禁止自动真钱执行 |
| 必须新增 | “不下单”状态、价格路径推演、H5 四元素评分、资金管理状态机 | 做成 H5 的核心差异化 |

## H5 版本的信息架构建议

1. Command Center  
   总览当前市场状态、机会质量、风险预算、今日是否允许交易。

2. Research Agents  
   多 agent 分工：Price Action、Volume/VSA、Wyckoff、Macro/TradFi、Risk、Devil's Advocate。

3. H5 Strategy Package  
   H5 Excel 规则、结构证据、指标解释、评分阈值、Pine 回测版本、Python 策略版本。

4. Scenario Planner  
   未来 15/30/60 分钟多路径推演：A 突破、B 回踩、C 假突破、D 无效震荡。

5. Backtest Lab  
   TradingView Pine、Python 回测、walk-forward、OOS、参数稳定性、过拟合惩罚。

6. Risk Kernel  
   单笔最大亏损、日亏损、连续亏损、回撤、仓位、盈亏比、交易暂停。

7. Execution Desk  
   SIGNAL_ONLY、SEMI_AUTO、AUTO_API。任何 API 下单必须被 Risk Kernel 授权。

8. Broker & Data  
   Binance、Binance tokenized securities、Alpaca、IBKR、MT5、贵金属、原油、外汇。

9. Audit & Learning  
   每一次分析、放弃、下单、止损、风控拒绝，都进入复盘系统。

## 第一版原型目标

第一版原型不追求完整功能，而要明确产品灵魂：

- AI 不是替人冲动下单，而是提高信息处理、推演、纪律执行和复盘效率。
- 策略不是信仰，只是候选方案。
- 赚钱是硬道理，但我们只能确定亏损上限，所以风控是系统核心。
- 会不下单，是交易系统的高级能力。

对应原型文件：`h5-ai-command-center-prototype.html`。

## 下一步开发建议

1. 把 H5 Excel 规则整理成 `StrategySpec`。
2. 将 H5 的评分、风控、执行模式写入后端 schema。
3. 用 QuantDinger `experiment` 模块接入 H5 Pine/Python 回测结果。
4. 在前端先做 Command Center + Scenario Planner + Risk Kernel 三页。
5. 所有 live execution 先 paper-only，再 shadow trading，再人工授权 semi-auto，最后才允许 AUTO_API。
