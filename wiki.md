# TradingAgents 项目技术文档

## 1. 项目概述

TradingAgents 是一个基于多代理（Multi-Agent）架构的大型语言模型（LLM）金融交易框架。该框架模拟真实世界交易公司的运作模式，通过部署专业化的 LLM 驱动代理来协作评估市场条件并制定交易决策。这些代理包括基本面分析师、情绪专家、技术分析师、交易员和风险管理团队，它们通过动态讨论来识别最优策略。

### 1.1 核心特性

- **多代理协作系统**：采用专业化分工的代理架构，每个代理专注于特定分析领域
- **多 LLM 提供商支持**：支持 OpenAI GPT、Anthropic Claude、Google Gemini、xAI Grok 等多种语言模型
- **模块化设计**：基于 LangGraph 构建，确保灵活性和可扩展性
- **多数据源集成**：支持 Alpha Vantage 和 Yahoo Finance 等多种金融数据提供商
- **辩论机制**：代理之间通过结构化辩论平衡潜在收益与固有风险

### 1.2 版本信息

- 当前版本：v0.2.3
- Python 版本要求：>=3.10
- 主要依赖：LangGraph >= 0.4.8、LangChain 系列库

## 2. 项目架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        TradingAgents                             │
├─────────────────────────────────────────────────────────────────┤
│  CLI 层 (cli/)                                                   │
│  ├── main.py          - 命令行入口                               │
│  ├── config.py         - CLI 配置管理                            │
│  ├── models.py         - 数据模型定义                            │
│  ├── utils.py          - 工具函数                                │
│  ├── stats_handler.py  - 统计回调处理                            │
│  └── announcements.py  - 公告功能                                │
├─────────────────────────────────────────────────────────────────┤
│  Graph 层 (tradingagents/graph/)                                │
│  ├── trading_graph.py  - 核心图结构主类                          │
│  ├── setup.py           - 图构建与配置                            │
│  ├── propagation.py     - 状态传播处理                           │
│  ├── conditional_logic.py - 条件路由逻辑                         │
│  ├── reflection.py      - 反思与记忆机制                         │
│  └── signal_processing.py - 信号处理                             │
├─────────────────────────────────────────────────────────────────┤
│  Agents 层 (tradingagents/agents/)                              │
│  ├── analysts/          - 分析师团队                               │
│  ├── researchers/      - 研究员团队                               │
│  ├── trader/           - 交易员代理                              │
│  ├── risk_mgmt/        - 风险管理团队                            │
│  ├── managers/         - 组合管理器                              │
│  └── utils/            - 代理工具函数                             │
├─────────────────────────────────────────────────────────────────┤
│  Dataflows 层 (tradingagents/dataflows/)                        │
│  ├── interface.py      - 数据接口抽象层                          │
│  ├── y_finance.py       - Yahoo Finance 数据源                    │
│  ├── alpha_vantage.py   - Alpha Vantage 数据源                    │
│  └── config.py          - 数据源配置                              │
├─────────────────────────────────────────────────────────────────┤
│  LLM Clients 层 (tradingagents/llm_clients/)                    │
│  ├── factory.py         - 客户端工厂                             │
│  ├── openai_client.py  - OpenAI 客户端                           │
│  ├── anthropic_client.py - Anthropic 客户端                      │
│  ├── google_client.py   - Google 客户端                          │
│  └── azure_client.py    - Azure OpenAI 客户端                    │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 工作流程

TradingAgents 的工作流程分为以下几个阶段：

1. **分析师阶段**：四个专业分析师并行分析市场数据
2. **投资辩论阶段**：多头和空头研究员进行辩论
3. **交易决策阶段**：交易员根据分析制定交易计划
4. **风险管理阶段**：三种风险分析师评估交易风险
5. **最终决策阶段**：组合管理器做出最终交易决定

## 3. 核心模块详解

### 3.1 Graph 模块（工作流引擎）

#### 3.1.1 TradingAgentsGraph 类

**文件位置**：[tradingagents/graph/trading_graph.py](file:///d:/zhaofei/TradingAgents/tradingagents/graph/trading_graph.py)

**类职责**：核心编排类，负责初始化和管理整个多代理交易框架。

**主要方法**：

- `__init__(selected_analysts, debug, config, callbacks)`：初始化交易代理图
  - `selected_analysts`：选择的分析师类型列表，可选值包括 "market"、"social"、"news"、"fundamentals"
  - `debug`：是否启用调试模式
  - `config`：配置字典，默认为 DEFAULT_CONFIG
  - `callbacks`：回调处理器列表

- `propagate(company_name, trade_date)`：执行交易代理图的前向传播
  - 返回：(final_state, processed_signal)
  - 内部调用流程：初始化状态 → 执行图 → 记录状态 → 处理信号

- `reflect_and_remember(returns_losses)`：基于回报反思决策并更新记忆
  - 涉及的代理：多头研究员、空头研究员、交易员、投资裁判、组合管理器

- `process_signal(full_signal)`：处理信号以提取核心决策

**初始化流程**：

```python
1. 设置配置和目录
2. 创建 LLM 客户端（deep_thinking_llm 和 quick_thinking_llm）
3. 初始化记忆系统（5 个记忆实例）
4. 创建工具节点
5. 初始化图组件（条件逻辑、图设置、传播器、反射器、信号处理器）
6. 构建并编译工作流图
```

#### 3.1.2 GraphSetup 类

**文件位置**：[tradingagents/graph/setup.py](file:///d:/zhaofei/TradingAgents/tradingagents/graph/setup.py)

**类职责**：处理代理图的设置和配置，负责构建工作流拓扑结构。

**主要方法**：

- `setup_graph(selected_analysts)`：构建并编译代理工作流图
  - 动态添加分析师节点（基于选择）
  - 添加研究员节点（多头、空头）
  - 添加交易员和风险管理节点
  - 定义节点之间的边和条件路由

**图节点结构**：

```
START 
  → Market Analyst → tools_market → Msg Clear Market
  → Social Analyst → tools_social → Msg Clear Social  
  → News Analyst → tools_news → Msg Clear News
  → Fundamentals Analyst → tools_fundamentals → Msg Clear Fundamentals
  → Bull Researcher ↔ Bear Researcher (辩论循环)
  → Research Manager
  → Trader
  → Aggressive Analyst ↔ Conservative Analyst ↔ Neutral Analyst (风险辩论)
  → Portfolio Manager
  → END
```

#### 3.1.3 Propagator 类

**文件位置**：[tradingagents/graph/propagation.py](file:///d:/zhaofei/TradingAgents/tradingagents/graph/propagation.py)

**类职责**：处理状态初始化和图传播。

**主要方法**：

- `create_initial_state(company_name, trade_date)`：创建初始状态字典
  - 初始化消息历史
  - 设置投资辩论状态（InvestDebateState）
  - 设置风险辩论状态（RiskDebateState）
  - 初始化各分析报告为空字符串

- `get_graph_args(callbacks)`：获取图调用参数
  - 设置递归限制（默认为 100）
  - 可选添加回调处理器

#### 3.1.4 ConditionalLogic 类

**文件位置**：[tradingagents/graph/conditional_logic.py](file:///d:/zhaofei/TradingAgents/tradingagents/graph/conditional_logic.py)

**类职责**：确定图的执行流程路由。

**条件判断方法**：

- `should_continue_market(state)`：判断市场分析是否继续
- `should_continue_social(state)`：判断社交媒体分析是否继续
- `should_continue_news(state)`：判断新闻分析是否继续
- `should_continue_fundamentals(state)`：判断基本面分析是否继续

这些方法检查最后一条消息是否包含工具调用，决定是否继续执行工具或清除消息。

- `should_continue_debate(state)`：判断投资辩论是否继续
  - 当辩论轮数达到 2 × max_debate_rounds 时结束
  - 否则在多方和空方之间轮转

- `should_continue_risk_analysis(state)`：判断风险分析是否继续
  - 当风险辩论轮数达到 3 × max_risk_discuss_rounds 时结束
  - 否则在三中风险分析师之间轮转

### 3.2 Agents 模块（代理实现）

#### 3.2.1 分析师团队（Analysts）

**文件位置**：[tradingagents/agents/analysts/](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/)

##### Market Analyst（市场分析师）

**文件**：[market_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/market_analyst.py)

**职责**：分析金融市场，使用技术指标评估价格走势和交易模式。

**可用工具**：
- `get_stock_data`：获取 OHLCV 股票价格数据
- `get_indicators`：获取技术指标

**分析指标类别**：
- 移动平均线：close_50_sma、close_200_sma、close_10_ema
- MACD 相关：macd、macds、macdh
- 动量指标：rsi
- 波动性指标：boll、boll_ub、boll_lb、atr
- 成交量指标：vwma

##### Social Media Analyst（社交媒体分析师）

**文件**：[social_media_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/social_media_analyst.py)

**职责**：分析社交媒体和公众情绪，使用情绪评分算法评估短期市场情绪。

**可用工具**：
- `get_news`：获取新闻数据

##### News Analyst（新闻分析师）

**文件**：[news_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/news_analyst.py)

**职责**：监控全球新闻和宏观经济指标，解读事件对市场条件的影响。

**可用工具**：
- `get_news`：获取新闻数据
- `get_global_news`：获取全球新闻
- `get_insider_transactions`：获取内部人交易信息

##### Fundamentals Analyst（基本面分析师）

**文件**：[fundamentals_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/fundamentals_analyst.py)

**职责**：评估公司财务和业绩指标，识别内在价值和潜在风险信号。

**可用工具**：
- `get_fundamentals`：获取基本面数据
- `get_balance_sheet`：获取资产负债表
- `get_cashflow`：获取现金流量表
- `get_income_statement`：获取损益表

#### 3.2.2 研究员团队（Researchers）

**文件位置**：[tradingagents/agents/researchers/](file:///d:/zhaofei/TradingAgents/tradingagents/agents/researchers/)

##### Bull Researcher（多头研究员）

**文件**：[bull_researcher.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/researchers/bull_researcher.py)

**职责**：构建支持投资的有力证据基础，强调增长潜力、竞争优势和积极市场指标。

**关键职责**：
- 突出增长潜力：强调市场机会、收入预测和可扩展性
- 竞争优势：强调独特产品、品牌优势或市场主导地位
- 积极指标：使用财务健康状况、行业趋势和近期正面新闻作为证据
- 反向论证：批判性分析空头论点，用具体数据和合理推理反驳

##### Bear Researcher（空头研究员）

**文件**：[bear_researcher.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/researchers/bear_researcher.py)

**职责**：提出反对投资的理由，强调风险、挑战和负面指标。

**关键职责**：
- 风险和挑战：强调市场饱和、财务不稳定或宏观经济威胁
- 竞争劣势：强调市场地位弱化、创新下降或竞争对手威胁
- 负面指标：使用财务数据、市场趋势或近期负面新闻作为证据
- 多头反驳：批判性分析多头论点，揭示弱点或过度乐观假设

#### 3.2.3 交易员代理（Trader）

**文件**：[tradingagents/agents/trader/trader.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/trader/trader.py)

**职责**：根据分析师和研究员的分析报告，制定具体的交易决策。

**决策流程**：
1. 接收来自分析师团队的研究报告
2. 整合研究经理的投资计划
3. 考虑历史经验和教训
4. 输出交易建议（买入/持有/卖出）

**输出格式**：必须以 "FINAL TRANSACTION PROPOSAL: **BUY/HOLD/SELL**" 结尾

#### 3.2.4 风险管理团队（Risk Management）

**文件位置**：[tradingagents/agents/risk_mgmt/](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/)

##### Aggressive Analyst（激进分析师）

**文件**：[aggressive_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/aggressive_debator.py)

**职责**：评估高回报潜力，推动积极的投资策略，关注增长机会。

##### Conservative Analyst（保守分析师）

**文件**：[conservative_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/conservative_debator.py)

**职责**：保护资产，最小化波动，确保稳定可靠增长，优先考虑稳定性和安全性。

##### Neutral Analyst（中性分析师）

**文件**：[neutral_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/neutral_debator.py)

**职责**：平衡评估风险和回报，寻求稳健的投资决策。

#### 3.2.5 组合管理器（Portfolio Manager）

**文件**：[tradingagents/agents/managers/portfolio_manager.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/managers/portfolio_manager.py)

**职责**：综合风险分析师的辩论，做出最终交易决策。

**评级量表**：
- **Buy**：强烈信念进入或增加仓位
- **Overweight**：前景良好，逐步增加敞口
- **Hold**：维持当前仓位，无需操作
- **Underweight**：减少敞口，部分获利了结
- **Sell**：退出仓位或避免入场

### 3.3 数据流模块（Dataflows）

**文件位置**：[tradingagents/dataflows/](file:///d:/zhaofei/TradingAgents/tradingagents/dataflows/)

#### 3.3.1 接口抽象层

**文件**：[interface.py](file:///d:/zhaofei/TradingAgents/tradingagents/dataflows/interface.py)

**核心概念**：

- **工具类别映射**（TOOLS_CATEGORIES）：
  - core_stock_apis：OHLCV 股票价格数据
  - technical_indicators：技术分析指标
  - fundamental_data：公司基本面数据
  - news_data：新闻和内部人数据

- **供应商列表**（VENDOR_LIST）：
  - yfinance：Yahoo Finance
  - alpha_vantage：Alpha Vantage

- **供应商方法映射**（VENDOR_METHODS）：
  - 将工具名称映射到特定供应商的实现

**关键函数**：

- `route_to_vendor(tool_name, *args, **kwargs)`：根据配置将工具调用路由到指定的供应商实现
- `get_config()`：获取当前配置
- `set_config(config)`：设置配置

#### 3.3.2 Yahoo Finance 数据源

**文件**：[y_finance.py](file:///d:/zhaofei/TradingAgents/tradingagents/dataflows/y_finance.py)

**主要函数**：
- `get_YFin_data_online(symbol, start_date, end_date)`：获取股票价格数据
- `get_stock_stats_indicators_window(symbol, indicator_names, start_date, end_date)`：获取技术指标
- `get_yfinance_fundamentals(symbol)`：获取基本面数据
- `get_yfinance_balance_sheet(symbol)`：获取资产负债表
- `get_yfinance_cashflow(symbol)`：获取现金流量表
- `get_yfinance_income_statement(symbol)`：获取损益表
- `get_yfinance_insider_transactions(symbol)`：获取内部人交易

#### 3.3.3 Alpha Vantage 数据源

**文件**：[alpha_vantage.py](file:///d:/zhaofei/TradingAgents/tradingagents/dataflows/alpha_vantage.py)

**主要函数**：
- `get_alpha_vantage_stock(symbol, function, output_size)`：获取股票数据
- `get_alpha_vantage_indicator(symbol, indicator, interval, series_type)`：获取技术指标
- `get_alpha_vantage_fundamentals(symbol, statement_type)`：获取财务报表
- `get_alpha_vantage_news(symbol)`：获取新闻数据

#### 3.3.4 工具函数

**文件**：[stockstats_utils.py](file:///d:/zhaofei/TradingAgents/tradingagents/dataflows/stockstats_utils.py)

**主要函数**：
- `calculate_technical_indicators(df, indicators)`：计算技术指标
- `get_indicator_config(indicator)`：获取指标配置

### 3.4 LLM 客户端模块

**文件位置**：[tradingagents/llm_clients/](file:///d:/zhaofei/TradingAgents/tradingagents/llm_clients/)

#### 3.4.1 客户端工厂

**文件**：[factory.py](file:///d:/zhaofei/TradingAgents/tradingagents/llm_clients/factory.py)

**关键函数**：`create_llm_client(provider, model, base_url, **kwargs)`

**支持的提供商**：
- OpenAI 兼容：openai、xai、deepseek、qwen、glm、ollama、openrouter、minimax
- Anthropic：anthropic
- Google：google
- Azure：azure

#### 3.4.2 OpenAI 客户端

**文件**：[openai_client.py](file:///d:/zhaofei/TradingAgents/tradingagents/llm_clients/openai_client.py)

**类**：OpenAIClient

**特点**：
- 使用 Responses API（原生 OpenAI 模型）
- 支持 reasoning_effort 参数
- 第三方提供商使用标准 Chat Completions

**提供商配置**（_PROVIDER_CONFIG）：
- xai：api.x.ai/v1
- deepseek：api.deepseek.com
- qwen：dashscope-intl.aliyuncs.com
- glm：api.z.ai
- openrouter：openrouter.ai/api/v1
- ollama：localhost:11434/v1
- minimax：api.minimaxi.com/v1

#### 3.4.3 Anthropic 客户端

**文件**：[anthropic_client.py](file:///d:/zhaofei/TradingAgents/tradingagents/llm_clients/anthropic_client.py)

**类**：AnthropicClient

**特点**：
- 支持 effort 参数控制思考努力程度
- 支持 extended thinking 和 tool use

#### 3.4.4 Google 客户端

**文件**：[google_client.py](file:///d:/zhaofei/TradingAgents/tradingagents/llm_clients/google_client.py)

**类**：GoogleClient

**特点**：支持 thinking_level 参数

### 3.5 代理工具模块

**文件位置**：[tradingagents/agents/utils/](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/)

#### 3.5.1 核心工具

**文件**：[core_stock_tools.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/core_stock_tools.py)

**工具函数**：

- `@tool get_stock_data(symbol, start_date, end_date)`：获取股票价格数据
- `@tool get_indicators(symbol, indicator_names, start_date, end_date)`：获取技术指标

#### 3.5.2 技术指标工具

**文件**：[technical_indicators_tools.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/technical_indicators_tools.py)

**工具函数**：
- `@tool get_indicators(...)`：计算技术指标
  - 指标名称：macd、rsi、boll、atr、vwma 等

#### 3.5.3 基本面数据工具

**文件**：[fundamental_data_tools.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/fundamental_data_tools.py)

**工具函数**：
- `@tool get_fundamentals(symbol)`：获取基本面概览
- `@tool get_balance_sheet(symbol)`：获取资产负债表
- `@tool get_cashflow(symbol)`：获取现金流量表
- `@tool get_income_statement(symbol)`：获取损益表

#### 3.5.4 新闻数据工具

**文件**：[news_data_tools.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/news_data_tools.py)

**工具函数**：
- `@tool get_news(symbol, ...)`：获取新闻数据
- `@tool get_global_news(...)`：获取全球新闻
- `@tool get_insider_transactions(symbol)`：获取内部人交易

#### 3.5.5 代理状态定义

**文件**：[agent_states.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/agent_states.py)

**状态类**：

- `AgentState(MessagesState)`：主状态类
  - company_of_interest：目标公司
  - trade_date：交易日期
  - market_report：市场报告
  - sentiment_report：情绪报告
  - news_report：新闻报告
  - fundamentals_report：基本面报告
  - investment_debate_state：投资辩论状态
  - risk_debate_state：风险辩论状态
  - final_trade_decision：最终交易决策

- `InvestDebateState`：投资辩论状态
  - bull_history：多头历史
  - bear_history：空头历史
  - judge_decision：裁判决策

- `RiskDebateState`：风险辩论状态
  - aggressive_history：激进分析师历史
  - conservative_history：保守分析师历史
  - neutral_history：中性分析师历史
  - judge_decision：裁判决策

#### 3.5.6 记忆系统

**文件**：[memory.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/utils/memory.py)

**类**：FinancialSituationMemory

**功能**：
- 存储代理的过去决策和反思
- 基于相似情况检索记忆
- 用于改进未来决策

**关键方法**：
- `get_memories(situation, n_matches)`：检索相似情况的记忆
- 支持 Bull Memory、Bear Memory、Trader Memory 等

### 3.6 CLI 模块

**文件位置**：[cli/](file:///d:/zhaofei/TradingAgents/cli/)

#### 3.6.1 CLI 主程序

**文件**：[main.py](file:///d:/zhaofei/TradingAgents/cli/main.py)

**功能**：
- 提供交互式命令行界面
- 实时显示分析进度
- 支持用户选择股票、分析日期、LLM 提供商等

**核心类**：
- `MessageBuffer`：消息缓冲区，跟踪代理状态和报告
- `app`：Typer CLI 应用实例

**命令**：
- `tradingagents`：启动交互式 CLI

#### 3.6.2 配置管理

**文件**：[config.py](file:///d:/zhaofei/TradingAgents/cli/config.py)

**功能**：
- 管理 CLI 配置选项
- 解析用户输入
- 与 TradingAgentsGraph 配置集成

## 4. 配置说明

### 4.1 默认配置

**文件**：[tradingagents/default_config.py](file:///d:/zhaofei/TradingAgents/tradingagents/default_config.py)

**配置项详解**：

```python
{
    # 目录配置
    "project_dir": "...",              # 项目根目录
    "results_dir": "...",             # 结果输出目录
    "data_cache_dir": "...",          # 数据缓存目录

    # LLM 设置
    "llm_provider": "openai",        # LLM 提供商
    "deep_think_llm": "gpt-5.4",       # 深度思考模型
    "quick_think_llm": "gpt-5.4-mini", # 快速思考模型
    "backend_url": "...",             # 后端 API 地址

    # 提供商特定配置
    "google_thinking_level": None,     # Google 思考级别
    "openai_reasoning_effort": None,   # OpenAI 推理努力
    "anthropic_effort": None,          # Anthropic 努力程度

    # 输出语言
    "output_language": "English",      # 输出语言

    # 辩论设置
    "max_debate_rounds": 1,            # 投资辩论轮数
    "max_risk_discuss_rounds": 1,      # 风险讨论轮数
    "max_recur_limit": 100,           # 最大递归限制

    # 数据供应商配置
    "data_vendors": {
        "core_stock_apis": "yfinance",
        "technical_indicators": "yfinance",
        "fundamental_data": "yfinance",
        "news_data": "yfinance",
    },

    # 工具级供应商配置（覆盖类别级）
    "tool_vendors": {},
}
```

### 4.2 环境变量

必需的环境变量（根据使用的 LLM 提供商）：

```bash
# LLM API Keys
OPENAI_API_KEY=...           # OpenAI (GPT)
GOOGLE_API_KEY=...           # Google (Gemini)
ANTHROPIC_API_KEY=...        # Anthropic (Claude)
XAI_API_KEY=...              # xAI (Grok)
DEEPSEEK_API_KEY=...         # DeepSeek
DASHSCOPE_API_KEY=...        # Qwen (Alibaba)
ZHIPU_API_KEY=...            # GLM (Zhipu)
OPENROUTER_API_KEY=...       # OpenRouter
MINIMAX_API_KEY=...          # MiniMax

# 数据 API
ALPHA_VANTAGE_API_KEY=...    # Alpha Vantage

# 自定义目录（可选）
TRADINGAGENTS_RESULTS_DIR=... # 结果目录
TRADINGAGENTS_CACHE_DIR=...   # 缓存目录
```

### 4.3 企业配置

对于 Azure OpenAI 或 AWS Bedrock 等企业提供商：
1. 复制 `.env.enterprise.example` 到 `.env.enterprise`
2. 填写企业凭证

## 5. 使用指南

### 5.1 安装

```bash
# 克隆项目
git clone https://github.com/TauricResearch/TradingAgents.git
cd TradingAgents

# 创建虚拟环境
conda create -n tradingagents python=3.13
conda activate tradingagents

# 安装依赖
pip install .
```

### 5.2 Docker 部署

```bash
# 复制环境变量文件
cp .env.example .env
# 编辑 .env 添加 API keys

# 运行 Docker
docker compose run --rm tradingagents

# 使用 Ollama 本地模型
docker compose --profile ollama run --rm tradingagents-ollama
```

### 5.3 CLI 使用

```bash
# 启动交互式 CLI
tradingagents

# 或直接运行
python -m cli.main
```

CLI 界面支持：
- 选择股票代码
- 选择分析日期
- 选择 LLM 提供商
- 选择研究深度
- 实时查看分析进度

### 5.4 Python API 使用

#### 基本用法

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

# 创建实例
ta = TradingAgentsGraph(debug=True, config=DEFAULT_CONFIG.copy())

# 执行分析
_, decision = ta.propagate("NVDA", "2026-01-15")
print(decision)

# 基于回报反思
ta.reflect_and_remember(1000)
```

#### 自定义配置

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

config = DEFAULT_CONFIG.copy()
config["deep_think_llm"] = "gpt-5.4-mini"
config["quick_think_llm"] = "gpt-5.4-mini"
config["max_debate_rounds"] = 2

# 配置数据供应商
config["data_vendors"] = {
    "core_stock_apis": "yfinance",
    "technical_indicators": "yfinance",
    "fundamental_data": "yfinance",
    "news_data": "yfinance",
}

ta = TradingAgentsGraph(debug=True, config=config)
_, decision = ta.propagate("AAPL", "2026-01-15")
```

#### 选择特定分析师

```python
# 只使用市场分析师和新闻分析师
ta = TradingAgentsGraph(
    selected_analysts=["market", "news"],
    debug=True,
    config=DEFAULT_CONFIG.copy()
)

_, decision = ta.propagate("TSM", "2026-01-15")
```

#### 使用回调处理

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG
from cli.stats_handler import StatsCallbackHandler

stats_handler = StatsCallbackHandler()

ta = TradingAgentsGraph(
    debug=True,
    config=DEFAULT_CONFIG.copy(),
    callbacks=[stats_handler]
)

_, decision = ta.propagate("MSFT", "2026-01-15")
```

## 6. 项目结构详解

### 6.1 目录树

```
TradingAgents/
├── cli/                          # 命令行界面
│   ├── __init__.py
│   ├── main.py                   # CLI 入口
│   ├── config.py                 # 配置管理
│   ├── models.py                 # 数据模型
│   ├── utils.py                  # 工具函数
│   ├── stats_handler.py          # 统计回调
│   ├── announcements.py         # 公告功能
│   └── static/
│       └── welcome.txt
│
├── tradingagents/                # 核心包
│   ├── __init__.py
│   ├── default_config.py         # 默认配置
│   │
│   ├── agents/                    # 代理模块
│   │   ├── __init__.py
│   │   ├── analysts/              # 分析师
│   │   │   ├── market_analyst.py
│   │   │   ├── social_media_analyst.py
│   │   │   ├── news_analyst.py
│   │   │   └── fundamentals_analyst.py
│   │   ├── researchers/           # 研究员
│   │   │   ├── bull_researcher.py
│   │   │   └── bear_researcher.py
│   │   ├── trader/                # 交易员
│   │   │   └── trader.py
│   │   ├── risk_mgmt/             # 风险管理
│   │   │   ├── aggressive_debator.py
│   │   │   ├── conservative_debator.py
│   │   │   └── neutral_debator.py
│   │   ├── managers/              # 组合管理
│   │   │   ├── research_manager.py
│   │   │   └── portfolio_manager.py
│   │   └── utils/                 # 代理工具
│   │       ├── agent_states.py
│   │       ├── agent_utils.py
│   │       ├── core_stock_tools.py
│   │       ├── fundamental_data_tools.py
│   │       ├── news_data_tools.py
│   │       ├── technical_indicators_tools.py
│   │       └── memory.py
│   │
│   ├── dataflows/                 # 数据流
│   │   ├── __init__.py
│   │   ├── interface.py           # 数据接口抽象
│   │   ├── config.py               # 数据配置
│   │   ├── y_finance.py            # Yahoo Finance
│   │   ├── yfinance_news.py
│   │   ├── alpha_vantage.py        # Alpha Vantage
│   │   ├── alpha_vantage_common.py
│   │   ├── alpha_vantage_fundamentals.py
│   │   ├── alpha_vantage_indicator.py
│   │   ├── alpha_vantage_news.py
│   │   ├── alpha_vantage_stock.py
│   │   ├── stockstats_utils.py
│   │   └── utils.py
│   │
│   ├── graph/                     # 工作流图
│   │   ├── __init__.py
│   │   ├── trading_graph.py       # 核心图类
│   │   ├── setup.py                # 图构建
│   │   ├── propagation.py          # 状态传播
│   │   ├── conditional_logic.py    # 条件路由
│   │   ├── reflection.py           # 反思机制
│   │   └── signal_processing.py    # 信号处理
│   │
│   └── llm_clients/               # LLM 客户端
│       ├── __init__.py
│       ├── factory.py             # 客户端工厂
│       ├── base_client.py          # 基类
│       ├── openai_client.py        # OpenAI
│       ├── anthropic_client.py     # Anthropic
│       ├── google_client.py        # Google
│       ├── azure_client.py         # Azure
│       ├── model_catalog.py        # 模型目录
│       ├── validators.py           # 验证器
│       └── TODO.md
│
├── tests/                          # 测试
│   ├── test_google_api_key.py
│   ├── test_model_validation.py
│   └── test_ticker_symbol_handling.py
│
├── assets/                         # 资源文件
│   ├── schema.png                  # 架构图
│   ├── analyst.png                 # 分析师图
│   ├── researcher.png              # 研究员图
│   ├── trader.png                  # 交易员图
│   ├── risk.png                    # 风险管理图
│   ├── TauricResearch.png
│   ├── cli/
│   │   ├── cli_init.png
│   │   ├── cli_news.png
│   │   ├── cli_technical.png
│   │   └── cli_transaction.png
│   └── wechat.png
│
├── main.py                         # 示例入口
├── test.py                         # 测试脚本
├── requirements.txt                # 依赖列表
├── pyproject.toml                  # 项目配置
├── Dockerfile                      # Docker 配置
├── docker-compose.yml              # Docker Compose
├── .env.example                    # 环境变量示例
├── .env.enterprise.example         # 企业配置示例
├── .gitignore
├── README.md
├── LICENSE
└── wiki.md                         # 本文档
```

### 6.2 依赖关系图

```
cli.main
  └── tradingagents.graph.trading_graph
        ├── tradingagents.agents.* (所有代理)
        ├── tradingagents.dataflows.* (数据流)
        ├── tradingagents.llm_clients (LLM 客户端)
        ├── tradingagents.default_config
        └── langgraph

tradingagents.agents.*
  ├── tradingagents.dataflows.interface
  ├── tradingagents.llm_clients
  └── langchain_core.prompts

tradingagents.dataflows.interface
  ├── tradingagents.dataflows.y_finance
  ├── tradingagents.dataflows.alpha_vantage
  └── tradingagents.dataflows.config

tradingagents.llm_clients.factory
  ├── tradingagents.llm_clients.openai_client
  ├── tradingagents.llm_clients.anthropic_client
  ├── tradingagents.llm_clients.google_client
  └── tradingagents.llm_clients.azure_client
```

## 7. 扩展指南

### 7.1 添加新的 LLM 提供商

1. 在 `llm_clients/` 目录创建新的客户端文件
2. 实现 `BaseLLMClient` 接口
3. 在 `factory.py` 中注册新提供商

```python
# 示例：添加新提供商
from .base_client import BaseLLMClient

class NewProviderClient(BaseLLMClient):
    def __init__(self, model: str, base_url: Optional[str] = None, **kwargs):
        super().__init__(model, base_url, **kwargs)
    
    def get_llm(self) -> Any:
        # 返回配置的 LLM 实例
        pass
    
    def validate_model(self) -> bool:
        pass
```

### 7.2 添加新的数据供应商

1. 在 `dataflows/` 目录创建新的数据源文件
2. 实现所需的数据获取函数
3. 在 `interface.py` 中注册新供应商

### 7.3 创建自定义分析师

参考现有的分析师实现创建新的分析师代理：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

def create_custom_analyst(llm):
    def custom_analyst_node(state):
        # 实现分析逻辑
        # 使用工具获取数据
        # 生成分析报告
        return {
            "messages": [result],
            "custom_report": report,
        }
    return custom_analyst_node
```

## 8. 最佳实践

### 8.1 配置建议

- **模型选择**：深度思考任务（如投资辩论）使用更强大的模型，快速分析任务使用轻量级模型
- **辩论轮数**：增加 `max_debate_rounds` 和 `max_risk_discuss_rounds` 以获得更深入的分析（但会增加成本和时间）
- **数据供应商**：yfinance 免费但可能有频率限制；Alpha Vantage 提供更稳定的服务但需要 API key

### 8.2 性能优化

- **缓存**：数据默认缓存到 `~/.tradingagents/cache/`
- **回调**：使用 `StatsCallbackHandler` 跟踪 LLM 和工具调用统计
- **调试**：非必要情况下关闭 `debug=True` 以提高性能

### 8.3 风险管理

- 本框架仅用于研究目的
- 交易表现受多种因素影响，包括模型选择、温度参数、交易周期、数据质量等
- 不作为金融、投资或交易建议

## 9. API 参考

### 9.1 TradingAgentsGraph

```python
class TradingAgentsGraph:
    def __init__(
        self,
        selected_analysts: List[str] = ["market", "social", "news", "fundamentals"],
        debug: bool = False,
        config: Dict[str, Any] = None,
        callbacks: Optional[List] = None,
    )
    
    def propagate(self, company_name: str, trade_date: str) -> Tuple[Dict, str]:
        """执行交易分析"""
    
    def reflect_and_remember(self, returns_losses: float) -> None:
        """基于回报反思决策"""
    
    def process_signal(self, full_signal: str) -> str:
        """处理信号提取核心决策"""
```

### 9.2 关键枚举

**分析师类型**（AnalystType）：
- MARKET = "market"
- SOCIAL = "social"
- NEWS = "news"
- FUNDAMENTALS = "fundamentals"

**交易评级**（Rating Scale）：
- BUY：强烈信念进入或增加仓位
- OVERWEIGHT：前景良好，逐步增加敞口
- HOLD：维持当前仓位
- UNDERWEIGHT：减少敞口
- SELL：退出仓位

## 10. 故障排除

### 常见问题

1. **API 密钥错误**
   - 检查环境变量是否正确设置
   - 确保 `.env` 文件存在且格式正确

2. **数据获取失败**
   - yfinance 可能受网络限制影响
   - Alpha Vantage 免费版有频率限制（5 calls/min）

3. **内存不足**
   - 减少辩论轮数
   - 选择较少的分析师

4. **模型不支持**
   - 检查 `model_catalog.py` 确认支持的模型
   - 使用 `validate_model()` 方法验证

## 11. 贡献指南

欢迎提交 Pull Request 和 Issue。请确保：

1. 代码遵循现有的代码风格
2. 添加适当的测试
3. 更新相关文档
4. 检查 lint 和 typecheck

## 12. 许可证

本项目采用 MIT 许可证。详见 LICENSE 文件。

## 13. 联系方式

- GitHub Issues: https://github.com/TauricResearch/TradingAgents/issues
- Discord: https://discord.com/invite/hk9PGKShPK
- X (Twitter): @TauricResearch

## 14. 引用

如果您在研究中使用了 TradingAgents，请引用：

```bibtex
@misc{TradingAgents,
  title={TradingAgents: Multi-Agents LLM Financial Trading Framework},
  author={Tauric Research},
  year={2024},
  eprint={2412.20138},
  archivePrefix={arXiv},
  primaryClass={cs.AI}
}
```

---

**文档版本**：v0.2.3  
**最后更新**：2026-04-22  
**维护者**：Tauric Research Team
