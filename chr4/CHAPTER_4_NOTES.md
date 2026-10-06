# Chapter 4 速查笔记

## 模型与生成

| 名称 | 一句话解释 |
|---|---|
| `model_id` | Hugging Face Hub 上模型仓库的标识；模型可以由 Meta 等公司训练，由 Hugging Face 托管和分发。 |
| Tokenizer | 把文本切成 token 并转换成 token ID；它不负责生成 embedding。 |
| Embedding | 模型内部把 token ID 映射成向量的层，调用模型时自动执行。 |
| 模型权重 | 模型训练后确定的数值参数，包括 embedding、Attention 和 MLP 等矩阵中的数值。 |
| 4-bit 量化 | 降低权重数值的存储精度以节省内存；参数数量和向量维度不变，可能损失部分效果。 |
| `AutoModelForCausalLM` | 根据模型配置自动选择具体类，并加载用于从左到右预测下一个 token 的模型。 |
| `pipeline("text-generation")` | 把 tokenize、模型生成、token 选择和 decode 包装成文本生成流水线。 |
| `max_new_tokens` | 限制新生成的 token 数，不包含输入 token。 |
| `top_k=50` | 每生成一个 token 时，只保留概率最高的50个候选 token。 |
| `do_sample=True` | 按候选 token 的概率随机抽取；不是平均随机，也不是永远选择第一名。 |
| `temperature` | 调整概率分布的集中程度；数值较低通常使输出更稳定。 |
| `return_full_text=False` | 只返回新生成内容，避免输入 Prompt 被重复返回并干扰 Agent 解析。 |
| `HuggingFacePipeline` | 把 Transformers pipeline 适配成 LangChain 可以调用的 LLM 接口。 |

生成循环：

```text
文本 → token IDs → embedding → Transformer → 下一个 token 的概率
→ top_k / sampling 选择 token ID → 加入上下文 → 重复计算 → decode 成文本
```

## Agent 与工具

| 名称 | 一句话解释 |
|---|---|
| Tool | 一个可调用函数及其名称、描述和输入约定；描述帮助 LLM 判断何时使用它。 |
| `llm-math` | 用 LLM 把自然语言数学问题转换成可计算表达式，因此创建它时需要传入 `llm`。 |
| Template / Prompt | 向 LLM 提供用户问题、工具说明、输出格式和历史过程；它只提供指令，不执行工具。 |
| `{input}` | 原始用户问题，不包含上一轮 Observation。 |
| `{tools}` | 所有工具的名称和详细描述文本。 |
| `{tool_names}` | Agent 可以选择的工具名称列表。 |
| `{agent_scratchpad}` | 之前的 Action、Action Input 和 Observation，供下一轮决策使用。 |
| LLM | 根据 Prompt 生成原始的 Action 或 Final Answer 文本。 |
| Output Parser | 把 LLM 原始文本解析成 `AgentAction` 或 `AgentFinish` 对象。 |
| ReAct Agent | 组合 Prompt、LLM 和 Output Parser，负责返回下一步决定，不负责执行工具。 |
| `AgentExecutor` | 根据 `AgentAction` 调用真正的 Tool，把 Observation 放回历史并控制循环。 |
| `AgentAction` | 需要调用的工具名称及其输入。 |
| `AgentFinish` | 表示不再调用工具，返回最终答案。 |
| Observation | Tool 的执行结果；它由 Tool 产生，不是 LLM 自己填写。 |
| `max_iterations` | 限制 AgentExecutor 的决策/工具循环次数，防止无限循环。 |
| `handle_parsing_errors=True` | LLM 输出不符合 ReAct 格式时，将错误反馈给 Agent 尝试修正。 |

完整流程：

```text
Agent 组装 Prompt → LLM 生成原始文本 → Parser 生成 AgentAction/AgentFinish
→ AgentExecutor 执行 Tool → Tool 返回 Observation → 写入 scratchpad → 再次决策
```

## 容易混淆的点

- Tool 列表没有固定执行顺序；LLM 根据问题、工具描述和历史结果选择工具。
- LLM 产生 Action，Tool 产生 Observation，AgentExecutor 执行 Tool。
- `create_react_agent` 接收 tools 是为了让 LLM 看见工具说明；`AgentExecutor` 接收 tools 是为了持有并执行真正的工具对象。
- 新生成的 token 已经是 token ID；继续预测时把它加入上下文并进入 embedding，不需要先 decode 再 tokenizer。
- Prompt 可以要求执行顺序，但不能可靠保证；必须严格按 A → B → C 时，应由程序或工作流控制。
- 一次 ReAct iteration 通常是一次决策以及可能的一次工具调用；调用两个工具后还需要一轮让 LLM 输出 Final Answer。
- 看懂代码不等于掌握；掌握的标准是能预测流程、解释角色、指导修改并定位错误属于模型、Prompt、Agent、Executor 还是 Tool。
