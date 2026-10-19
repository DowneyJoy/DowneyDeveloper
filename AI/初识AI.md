- **LLM**：Large Language Model大语言模型，简称**大模型**

- **Token**：**大模型处理文本的最基本单位**

- 用户-->Tokenizer【编码：切分、映射|解码：映射】-->大模型

- Token拆解：https://platform.openai.com/tokenizer
  - 1个Token：0.75个英文单词|1.2-2个汉字
- **Context**：上下文【**大模型每次处理任务时所接收到的信息总和**】大模型的临时记忆体
  - 对话历史、用户问题、当前输出、工具列表、System Prompt等
- **Context Window**：上下文窗口【**Context能容纳的最大Token数量**】
  - 主流模型的Context Window大小
    - GPT-5.4：105w
    - Gemini 3.1 Pro：100w
    - Claude Opus 4.6：100w
- RAG：Retrieval Augmented Generation 检索 增强 生成
  - 先从资料库里检索相关内容
  - 再基于这些内容来生成答案
- **Prompt**：提示词【**用户或系统当前给大模型下达的具体指令或问题**】
- Prompt Engineering：提示词工程
  - User Prompt：用户输入，具体任务
  - System Prompt：后台配置，人设和做事规则
    - 例如：你是一位耐心的数学老师。当学生问你数学问题时，不要直接给出答案，而是要一步一步引导学生思考，帮助他们理解接替思路。
- **Tool**：工具（函数）【**给大模型提供一套它可以调用的外部能力，让大模型能够感知和影响外部环境**】
  - 例如：天气查询工具【调用气象局接口，输入城市日期，输出天气温度】
    - 大模型：选择工具、归纳总结
    - 工具：查询天气
    - 平台：串联流程
- **MCP**：Model Context Protocol模型上下文协议【**统一了工具接入格式的标准协议**】
- **Agent**：**能自主规划和调用工具、直至解决用户问题的程序**
  - Agent产品
    - Claude Code
    - Codex
    - Gemini CLI
  - 构建模式
    - ReAct
    - Plan And Execute

![image-20260807111336726](..\Images\image-20260807111336726.png)

- **Agent Skill**：**给Agent看的说明文档**
  - 例如：cd ~/.claude/skills
    - mkdir go-out-checklist
    - cd go-out-checklist
    - vim SKILL.md