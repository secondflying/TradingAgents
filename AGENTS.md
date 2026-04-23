# TradingAgents 代理系统文档

## 1. 概述

TradingAgents 采用多代理架构，模拟真实世界交易公司的运作模式。系统包含 11 个专业代理，协同完成从市场分析到交易决策的完整流程。

### 1.1 代理架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TradingAgents 代理系统                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐        │
│  │  Market Analyst │    │ Social Analyst  │    │  News Analyst   │        │
│  │   市场分析师     │    │  社交媒体分析师  │    │   新闻分析师    │        │
│  └────────┬────────┘    └────────┬────────┘    └────────┬────────┘        │
│           │                       │                       │                 │
│  ┌────────┴────────┐    ┌────────┴────────┐             │                 │
│  │ Fundamentals   │    │  Analyst Team    │             │                 │
│  │   Analyst      │────│   (4个分析师)     │─────────────┘                 │
│  │   基本面分析师  │    └────────┬────────┘                               │
│  └────────┬────────┘             │                                          │
│           │                     ▼                                          │
│           │         ┌───────────────────────────────────┐                  │
│           │         │      Researcher Team              │                  │
│           │         │   Bull Researcher (多头)           │                  │
│           │         │◄────────────────────────────►     │                  │
│           │         │   Bear Researcher (空头)          │                  │
│           │         └───────────────┬───────────────────┘                  │
│           │                       │                                          │
│           │                       ▼                                          │
│           │         ┌───────────────────────────────────┐                  │
│           │         │     Research Manager              │                  │
│           │         │     研究经理（裁判）               │                  │
│           │         └───────────────┬───────────────────┘                  │
│           │                       │                                          │
│           │                       ▼                                          │
│           │         ┌───────────────────────────────────┐                  │
│           │         │        Trader                      │                  │
│           │         │       交易员                      │                  │
│           │         └───────────────┬───────────────────┘                  │
│           │                       │                                          │
│           │                       ▼                                          │
│           │         ┌───────────────────────────────────┐                  │
│           │         │   Risk Management Team            │                  │
│           │         │  Aggressive Analyst (激进)        │                  │
│           │         │◄────────────────────────────►     │                  │
│           │         │  Neutral Analyst (中性)            │◄─────────────────┤
│           │         │◄────────────────────────────►     │                 │
│           │         │  Conservative Analyst (保守)      │                  │
│           │         └───────────────┬───────────────────┘                  │
│           │                       │                                          │
│           │                       ▼                                          │
│           │         ┌───────────────────────────────────┐                  │
│           │         │    Portfolio Manager              │                  │
│           │         │    组合经理（最终裁判）           │                  │
│           │         └───────────────────────────────────┘                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. 代理详解

### 2.1 分析师团队 (Analyst Team)

分析师团队负责收集和解析各类市场数据，为后续决策提供基础信息。

#### 2.1.1 Market Analyst（市场分析师）

**文件位置**：[tradingagents/agents/analysts/market_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/market_analyst.py)

**职责**：
- 分析金融市场的技术指标
- 识别价格走势和交易模式
- 评估市场趋势和动量

**可用工具**：
| 工具名称 | 功能描述 |
|---------|---------|
| `get_stock_data` | 获取 OHLCV 股票价格数据 |
| `get_indicators` | 获取技术分析指标 |

**技术指标列表**：

| 类别 | 指标 | 描述 |
|-----|------|-----|
| 移动平均线 | `close_50_sma` | 50日简单移动平均，中期趋势指标 |
| 移动平均线 | `close_200_sma` | 200日简单移动平均，长期趋势基准 |
| 移动平均线 | `close_10_ema` | 10日指数移动平均，短期快速响应 |
| MACD | `macd` | MACD线，动量指标 |
| MACD | `macds` | MACD信号线 |
| MACD | `macdh` | MACD柱状图 |
| 动量 | `rsi` | 相对强弱指数，衡量超买超卖 |
| 波动性 | `boll` |布林带中轨 |
| 波动性 | `boll_ub` | 布林带上轨 |
| 波动性 | `boll_lb` | 布林带下轨 |
| 波动性 | `atr` | 平均真实波幅 |
| 成交量 | `vwma` | 成交量加权平均价 |

**输出状态**：`market_report`

**使用模型**：quick_thinking_llm

---

#### 2.1.2 Social Media Analyst（社交媒体分析师）

**文件位置**：[tradingagents/agents/analysts/social_media_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/social_media_analyst.py)

**职责**：
- 分析社交媒体和公众情绪
- 使用情绪评分算法评估短期市场情绪
- 识别社交媒体上的趋势和话题

**可用工具**：
| 工具名称 | 功能描述 |
|---------|---------|
| `get_news` | 获取新闻数据 |

**输出状态**：`sentiment_report`

**使用模型**：quick_thinking_llm

---

#### 2.1.3 News Analyst（新闻分析师）

**文件位置**：[tradingagents/agents/analysts/news_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/news_analyst.py)

**职责**：
- 监控和分析全球新闻
- 解读宏观经济指标
- 评估新闻事件对市场的影响

**可用工具**：
| 工具名称 | 功能描述 |
|---------|---------|
| `get_news` | 获取公司特定或目标新闻搜索 |
| `get_global_news` | 获取更广泛的宏观经济新闻 |

**输出状态**：`news_report`

**使用模型**：quick_thinking_llm

---

#### 2.1.4 Fundamentals Analyst（基本面分析师）

**文件位置**：[tradingagents/agents/analysts/fundamentals_analyst.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/analysts/fundamentals_analyst.py)

**职责**：
- 评估公司财务和业绩指标
- 分析财务报表
- 识别内在价值和潜在风险信号

**可用工具**：
| 工具名称 | 功能描述 |
|---------|---------|
| `get_fundamentals` | 获取公司综合基本面分析 |
| `get_balance_sheet` | 获取资产负债表 |
| `get_cashflow` | 获取现金流量表 |
| `get_income_statement` | 获取损益表 |

**输出状态**：`fundamentals_report`

**使用模型**：quick_thinking_llm

---

### 2.2 研究员团队 (Researcher Team)

研究员团队负责对分析师的报告进行辩论，平衡潜在收益与固有风险。

#### 2.2.1 Bull Researcher（多头研究员）

**文件位置**：[tradingagents/agents/researchers/bull_researcher.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/researchers/bull_researcher.py)

**职责**：
- 构建支持投资的有力证据基础
- 强调增长潜力、竞争优势和积极市场指标
- 反驳空头论点

**关键分析维度**：

| 维度 | 描述 |
|-----|------|
| 增长潜力 | 强调市场机会、收入预测和可扩展性 |
| 竞争优势 | 强调独特产品、品牌优势或市场主导地位 |
| 积极指标 | 使用财务健康、行业趋势、近期正面新闻作为证据 |
| 反向论证 | 批判性分析空头论点，用数据和推理反驳 |

**输入数据**：
- `market_report`：市场研究报告
- `sentiment_report`：社交媒体情绪报告
- `news_report`：全球新闻报告
- `fundamentals_report`：基本面报告
- `memory`：历史相似情况的记忆

**输出状态**：`investment_debate_state.bull_history`

**使用模型**：quick_thinking_llm

---

#### 2.2.2 Bear Researcher（空头研究员）

**文件位置**：[tradingagents/agents/researchers/bear_researcher.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/researchers/bear_researcher.py)

**职责**：
- 提出反对投资的理由
- 强调风险、挑战和负面指标
- 反驳多头论点

**关键分析维度**：

| 维度 | 描述 |
|-----|------|
| 风险和挑战 | 强调市场饱和、财务不稳定或宏观经济威胁 |
| 竞争劣势 | 强调市场地位弱化、创新下降或竞争对手威胁 |
| 负面指标 | 使用财务数据、市场趋势或近期负面新闻作为证据 |
| 多头反驳 | 批判性分析多头论点，揭示弱点或过度乐观假设 |

**输入数据**：
- `market_report`：市场研究报告
- `sentiment_report`：社交媒体情绪报告
- `news_report`：全球新闻报告
- `fundamentals_report`：基本面报告
- `memory`：历史相似情况的记忆

**输出状态**：`investment_debate_state.bear_history`

**使用模型**：quick_thinking_llm

---

### 2.3 Research Manager（研究经理）

**文件位置**：[tradingagents/agents/managers/research_manager.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/managers/research_manager.py)

**职责**：
- 评判多头和空头研究员的辩论
- 做出明确的投资决策（买入/卖出/持有）
- 制定详细的投资计划

**决策输出**：

| 决策 | 含义 |
|-----|------|
| Buy | 强烈信念进入或增加仓位 |
| Hold | 维持当前仓位（仅在有充分理由时） |
| Sell | 退出仓位或避免入场 |

**投资计划内容**：
1. **推荐决策**：基于最有力论据的明确立场
2. **理由**：解释为何这些论据导致该结论
3. **战略行动**：实施建议的具体步骤

**输入数据**：
- 多头和空头研究员的辩论历史
- 所有分析师报告
- 历史决策记忆

**输出状态**：
- `investment_debate_state.judge_decision`
- `investment_plan`

**使用模型**：deep_thinking_llm

---

### 2.4 Trader（交易员）

**文件位置**：[tradingagents/agents/trader/trader.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/trader/trader.py)

**职责**：
- 根据分析师和研究经理的分析制定交易计划
- 确定交易的时机和规模
- 基于市场洞察做出明智的交易决策

**交易决策输出格式**：
```
FINAL TRANSACTION PROPOSAL: **BUY/HOLD/SELL**
```

**决策考虑因素**：
- 技术市场趋势
- 宏观经济指标
- 社交媒体情绪
- 基本面分析
- 历史相似情况的经验教训

**输入数据**：
- `investment_plan`：研究经理制定的投资计划
- 所有分析师报告
- 历史交易记忆

**输出状态**：`trader_investment_plan`

**使用模型**：quick_thinking_llm

---

### 2.5 风险管理团队 (Risk Management Team)

风险管理团队评估交易员决策的风险，平衡回报与风险。

#### 2.5.1 Aggressive Analyst（激进分析师）

**文件位置**：[tradingagents/agents/risk_mgmt/aggressive_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/aggressive_debator.py)

**职责**：
- 积极倡导高回报、高风险的机会
- 强调大胆策略和竞争优势
- 反驳保守和中性分析师的观点

**核心观点**：
- 高风险往往伴随高回报
- 保守策略可能错失关键机会
- 创新和进取是超越市场规范的关键

**与其他分析师的互动**：
- 主动应对保守和中性分析师的观点
- 用数据驱动的反驳说服他人
- 突出保守假设可能过于谨慎的地方

**使用模型**：quick_thinking_llm

---

#### 2.5.2 Conservative Analyst（保守分析师）

**文件位置**：[tradingagents/agents/risk_mgmt/conservative_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/conservative_debator.py)

**职责**：
- 保护资产，最小化波动
- 确保稳定可靠的增长
- 优先考虑稳定性和安全性

**核心观点**：
- 风险最小化是长期成功的关键
- 过度乐观可能带来不必要的损失
- 谨慎策略在长期内更可持续

**与其他分析师的互动**：
- 质疑激进和中性分析师的乐观情绪
- 强调潜在的下行风险
- 指出他人可能忽略的威胁

**使用模型**：quick_thinking_llm

---

#### 2.5.3 Neutral Analyst（中性分析师）

**文件位置**：[tradingagents/agents/risk_mgmt/neutral_debator.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/risk_mgmt/neutral_debator.py)

**职责**：
- 提供平衡的视角
- 同时评估潜在利益和风险
- 寻求稳健的投资决策

**核心观点**：
- 平衡的方法往往带来最可靠的结果
- 既要考虑增长潜力，又要考虑下行保护
- 多元化策略和风险对冲很重要

**与其他分析师的互动**：
- 挑战激进分析师的过度乐观
- 挑战保守分析师的过度谨慎
- 倡导中间立场

**使用模型**：quick_thinking_llm

---

### 2.6 Portfolio Manager（组合经理）

**文件位置**：[tradingagents/agents/managers/portfolio_manager.py](file:///d:/zhaofei/TradingAgents/tradingagents/agents/managers/portfolio_manager.py)

**职责**：
- 综合风险分析师的辩论
- 做出最终交易决策
- 制定执行策略

**五级评级系统**：

| 评级 | 含义 | 描述 |
|-----|------|-----|
| **Buy** | 强烈买入 | 强烈信念进入或增加仓位 |
| **Overweight** | 增持 | 前景良好，逐步增加敞口 |
| **Hold** | 持有 | 维持当前仓位，无需操作 |
| **Underweight** | 减持 | 减少敞口，部分获利了结 |
| **Sell** | 卖出 | 退出仓位或避免入场 |

**输出格式**：
1. **评级**：五级评级之一
2. **执行摘要**：涵盖入场策略、仓位规模、关键风险级别和时间范围的简明行动计划
3. **投资论点**：基于分析师辩论和历史反思的详细推理

**输入数据**：
- 研究经理的投资计划
- 交易员的交易提案
- 风险分析师的辩论历史
- 历史决策记忆

**输出状态**：
- `risk_debate_state.judge_decision`
- `final_trade_decision`

**使用模型**：deep_thinking_llm

---

## 3. 代理状态

### 3.1 AgentState（主状态）

```python
class AgentState(MessagesState):
    # 基本信息
    company_of_interest: str      # 目标公司
    trade_date: str                # 交易日期
    sender: str                    # 发送消息的代理
    
    # 分析师报告
    market_report: str             # 市场分析师报告
    sentiment_report: str           # 社交媒体分析师报告
    news_report: str              # 新闻分析师报告
    fundamentals_report: str       # 基本面分析师报告
    
    # 投资辩论
    investment_debate_state: InvestDebateState  # 投资辩论状态
    investment_plan: str           # 研究经理的投资计划
    
    # 交易
    trader_investment_plan: str    # 交易员的交易提案
    
    # 风险辩论
    risk_debate_state: RiskDebateState          # 风险辩论状态
    final_trade_decision: str      # 最终交易决策
```

### 3.2 InvestDebateState（投资辩论状态）

```python
class InvestDebateState(TypedDict):
    bull_history: str      # 多头研究员辩论历史
    bear_history: str      # 空头研究员辩论历史
    history: str           # 完整对话历史
    current_response: str   # 最新回复
    judge_decision: str     # 研究经理裁判决策
    count: int             # 辩论轮次计数
```

### 3.3 RiskDebateState（风险辩论状态）

```python
class RiskDebateState(TypedDict):
    aggressive_history: str      # 激进分析师历史
    conservative_history: str   # 保守分析师历史
    neutral_history: str        # 中性分析师历史
    history: str                # 完整对话历史
    latest_speaker: str          # 最后发言者
    
    current_aggressive_response: str   # 激进分析师最新回复
    current_conservative_response: str # 保守分析师最新回复
    current_neutral_response: str      # 中性分析师最新回复
    
    judge_decision: str          # 组合经理裁判决策
    count: int                   # 辩论轮次计数
```

---

## 4. 工具系统

### 4.1 工具分类

| 类别 | 工具 | 描述 |
|-----|------|-----|
| **核心股票数据** | `get_stock_data` | OHLCV 股票价格数据 |
| **技术指标** | `get_indicators` | 技术分析指标 |
| **基本面数据** | `get_fundamentals` | 公司综合基本面 |
| **基本面数据** | `get_balance_sheet` | 资产负债表 |
| **基本面数据** | `get_cashflow` | 现金流量表 |
| **基本面数据** | `get_income_statement` | 损益表 |
| **新闻数据** | `get_news` | 公司新闻 |
| **新闻数据** | `get_global_news` | 全球宏观经济新闻 |
| **新闻数据** | `get_insider_transactions` | 内部人交易 |

### 4.2 数据源配置

系统支持多个数据源，可在配置中切换：

```python
config["data_vendors"] = {
    "core_stock_apis": "yfinance",        # 或 "alpha_vantage"
    "technical_indicators": "yfinance",   # 或 "alpha_vantage"
    "fundamental_data": "yfinance",       # 或 "alpha_vantage"
    "news_data": "yfinance",              # 或 "alpha_vantage"
}
```

---

## 5. 辩论机制

### 5.1 投资辩论流程

```
多头研究员 ←→ 空头研究员
     ↑              ↑
     └──── 研究经理 ──┘
            ↓
         交易员
```

**辩论轮数控制**：`max_debate_rounds`（默认 1）

**结束条件**：`count >= 2 * max_debate_rounds`

### 5.2 风险辩论流程

```
激进分析师 ←→ 中性分析师 ←→ 保守分析师
     ↑              ↑              ↑
     └────────── 组合经理 ──────────┘
                    ↓
               最终决策
```

**辩论轮数控制**：`max_risk_discuss_rounds`（默认 1）

**结束条件**：`count >= 3 * max_risk_discuss_rounds`

---

## 6. 记忆系统

### 6.1 FinancialSituationMemory

每个代理都有自己的记忆系统，用于存储和检索历史决策经验。

**记忆类型**：

| 记忆名称 | 所有者 | 用途 |
|---------|-------|-----|
| `bull_memory` | 多头研究员 | 存储多头决策的经验教训 |
| `bear_memory` | 空头研究员 | 存储空头决策的经验教训 |
| `trader_memory` | 交易员 | 存储交易决策的经验教训 |
| `invest_judge_memory` | 研究经理 | 存储投资判断的经验教训 |
| `portfolio_manager_memory` | 组合经理 | 存储组合管理的经验教训 |

### 6.2 反思机制

```python
# 基于交易结果反思
ta.reflect_and_remember(returns_losses)

# 系统会更新各代理的记忆
ta.reflect_bull_researcher(curr_state, returns_losses, bull_memory)
ta.reflect_bear_researcher(curr_state, returns_losses, bear_memory)
ta.reflect_trader(curr_state, returns_losses, trader_memory)
ta.reflect_invest_judge(curr_state, returns_losses, invest_judge_memory)
ta.reflect_portfolio_manager(curr_state, returns_losses, portfolio_manager_memory)
```

---

## 7. 模型配置

### 7.1 双模型策略

系统使用两种不同的模型：

| 模型类型 | 用途 | 示例 |
|---------|-----|------|
| `deep_thinking_llm` | 复杂推理任务 | 研究经理、组合经理 |
| `quick_thinking_llm` | 快速分析任务 | 分析师、交易员 |

### 7.2 模型选择建议

**深度思考任务**（使用更强大的模型）：
- Research Manager
- Portfolio Manager

**快速思考任务**（可使用轻量级模型）：
- Market Analyst
- Social Media Analyst
- News Analyst
- Fundamentals Analyst
- Bull Researcher
- Bear Researcher
- Trader
- Aggressive Analyst
- Conservative Analyst
- Neutral Analyst

---

## 8. 代理创建工厂函数

所有代理通过工厂函数创建，签名统一：

```python
def create_{agent_name}(llm, memory=None) -> Callable:
    def {agent_name}_node(state) -> dict:
        # 代理逻辑
        return {output_state: result}
    return {agent_name}_node
```

### 8.1 创建函数列表

| 代理 | 工厂函数 | 内存参数 |
|-----|---------|---------|
| Market Analyst | `create_market_analyst(llm)` | 无 |
| Social Analyst | `create_social_media_analyst(llm)` | 无 |
| News Analyst | `create_news_analyst(llm)` | 无 |
| Fundamentals Analyst | `create_fundamentals_analyst(llm)` | 无 |
| Bull Researcher | `create_bull_researcher(llm, memory)` | ✓ |
| Bear Researcher | `create_bear_researcher(llm, memory)` | ✓ |
| Research Manager | `create_research_manager(llm, memory)` | ✓ |
| Trader | `create_trader(llm, memory)` | ✓ |
| Aggressive Analyst | `create_aggressive_debator(llm)` | 无 |
| Conservative Analyst | `create_conservative_debator(llm)` | 无 |
| Neutral Analyst | `create_neutral_debator(llm)` | 无 |
| Portfolio Manager | `create_portfolio_manager(llm, memory)` | ✓ |

---

## 9. 工作流程总结

### 9.1 完整流程

```
1. 并行分析（分析师团队）
   ├── Market Analyst → market_report
   ├── Social Analyst → sentiment_report
   ├── News Analyst → news_report
   └── Fundamentals Analyst → fundamentals_report
   
2. 投资辩论（研究员团队）
   Bull Researcher ↔ Bear Researcher → Research Manager → investment_plan
   
3. 交易决策
   Trader → trader_investment_plan
   
4. 风险辩论（风险管理团队）
   Aggressive Analyst ↔ Conservative Analyst ↔ Neutral Analyst
   → Portfolio Manager → final_trade_decision
```

### 9.2 决策输出

最终交易决策包含：
- **评级**：Buy / Overweight / Hold / Underweight / Sell
- **执行摘要**：入场策略、仓位规模、风险级别、时间范围
- **投资论点**：详细推理和证据

---

## 10. 扩展指南

### 10.1 添加新代理

1. 在对应目录创建代理文件
2. 实现工厂函数 `create_{agent_name}(llm, memory)`
3. 在 [setup.py](file:///d:/zhaofei/TradingAgents/tradingagents/graph/setup.py) 中注册节点
4. 定义输出状态字段

### 10.2 修改辩论轮数

```python
config = DEFAULT_CONFIG.copy()
config["max_debate_rounds"] = 2        # 投资辩论轮数
config["max_risk_discuss_rounds"] = 2  # 风险辩论轮数
```

### 10.3 自定义分析师选择

```python
# 只使用部分分析师
ta = TradingAgentsGraph(
    selected_analysts=["market", "news"]  # 只启用市场和新闻分析师
)
```

---

**文档版本**：v0.2.3  
**最后更新**：2026-04-22  
**维护者**：Tauric Research Team
