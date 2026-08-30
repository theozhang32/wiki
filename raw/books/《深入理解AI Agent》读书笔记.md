> 作者：李博杰 [Github地址](https://github.com/bojieli/ai-agent-book)

# 第一章 Agent基础知识

### Agent交互的最小结构
![Agent交互的最小结构](https://bojieli.github.io/ai-agent-book/book/images/fig1-1.svg)

`Agent = LLM + 上下文 + 工具 = LLM + Harness`

### 工具设计的核心原则

通用基础能力用于组合与探索；专用工具用户约束高风险和强业务规则操作。

### 上下文组成

`上下文 = （系统提示词 + 工具定义） + （用户消息 + 模型回复 + 工具执行结果） = 静态前缀 + 轨迹`

