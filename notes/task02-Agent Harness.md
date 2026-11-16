# task02 Agent Harness
>  从安装到运行，完成第一个能搜索网络、撰写报告的研究助手。

## 对应文档：
EP.01 从 Agent Framework 到 Agent Harness — Deep Agents 的诞生逻辑
https://datawhalechina.github.io/deepagents-in-action/chapters/ch01-agent-harness/

EP.02 快速上手 — 5 分钟构建你的第一个 Deep Agent
https://datawhalechina.github.io/deepagents-in-action/chapters/ch02-quickstart/


##  实现步骤：

```bash
# uv（安装）
uv pip install "deepagents>=0.7,<0.8" langchain-openai travily-python

配置 API Key
export SILICONFLOW_API_KEY="sk-xxx"
export MODEL_NAME="deepseek-xxx" 

# 创建相关代码并进行测试
cd "research_deepagent所在目录"
nano task2-search-tools.py  # 这里使用 附录 相关实现代码2 中的测试代码
# 运行测试
uv run --env-file .env python  task2-search-tools.py
```

## 测试结果：
相关实现代码2运行测试结果如下
```bash
开始执行 Agent...

[工具调用] 正在搜索：雾霾天气一般是由什么造成的

[工具调用] 正在搜索：雾霾成因 污染物来源 PM2.5 气象条件

========== Agent 最终回答 ==========

## 雾霾天气一般是由什么造成的

雾霾是**特定气象条件与人类活动相互作用**的结果，本质上是"污染物排放（内因）+ 不利扩散的气象条件（外因）"共同导致的。通俗地说：**雾是自然的水汽凝结现象，霾则是人为污染颗粒物导致的空气浑浊**，两者常混合出现，所以统称"雾霾天气"。

### 一、根本原因：人为排放的污染物（内因）

霾的核心物质是空气中悬浮的细颗粒物，尤其是 **PM2.5**（直径≤2.5微米的颗粒物），它是造成雾霾的"元凶"。这些颗粒物主要来自：

1. **燃煤**——冬季取暖、电厂、燃煤锅炉等排放的二氧化硫、烟尘；
2. **机动车尾气**——氮氧化物、碳氢化合物、黑碳等；
3. **工业生产废气**——各种工业污染源；
4. **扬尘**——建筑工地、道路交通产生的粉尘；
5. **生物质燃烧**——秸秆焚烧、生物质燃料等。

此外，二氧化硫、氮氧化物、挥发性有机物等气态污染物排入大气后，还会发生化学反应，**二次转化**生成硫酸盐、硝酸盐等颗粒物，进一步加重霾。

### 二、必要条件：不利于扩散的气象条件（外因）

有了污染物，还必须"憋"得住，才会形成持续的雾霾。关键气象因素有三个：

1. **静风现象增多**——城市高楼阻挡、摩擦使风速减弱，污染物不易被吹散；
2. **逆温现象**——高空比低空温度还高的"逆温层"像锅盖一样罩在城市上空，抑制空气垂直对流，污染物难以向高空扩散，被压在地面附近；
3. **湿度较高**——水汽使颗粒物吸湿增长，能见度进一步下降。

### 小结

概括而言：**高密度人口的经济社会活动排放大量细颗粒物（PM2.5），一旦排放超过大气自净能力，颗粒物持续积聚；此时若遭遇静风、逆温等静稳天气，污染物扩散不畅，就容易形成大范围雾霾**。用专家的话说，污染是"内因、可控"，气象是"外因、不可控"，而本地高强度排放才是重污染天气频发的根本原因。

**来源：**
- 中国天气网《雾霾天气形成的原因是什么》（tianqi.com）
- 百度百科/百度知道《雾是什么？霾是什么？雾霾又是什么？》
- 清华大学新闻网《环保部邀请权威专家解答公众疑虑：雾霾来袭 原因何在》（tsinghua.edu.cn）
- 中国科学院兰州文献情报中心科普文章（llas.cas.cn）
- 天奇生活《雾霾形成的主要原因是什么》（tianqijun.com）

========== 测试完成 ==========

```

## 简要内容理解

1. uv 是什么？与 pip、venv 有什么区别？

    uv 是一个高性能的 Python 包及项目管理工具，集成了依赖安装、虚拟环境管理等功能；pip 主要负责安装 Python 包，venv 负责创建虚拟环境。uv 可以替代部分 pip 和 venv 的工作，已有的 pip + venv 环境无需强制迁移。
2. ChatOpenAI、Model、Tool、Agent 分别是什么？

    - ChatOpenAI：用于创建连接大模型 API 的客户端对象，并非创建或训练大模型。
    - Model（模型）：负责理解问题、推理、生成回答及提出工具调用请求。
    - Tool（工具）：执行具体操作的函数，如搜索网络、查询天气、计算公式。
    - Agent（智能体）：将模型、工具及执行流程组合起来，由模型决策，框架负责调度和执行工具，从而完成任务。
3. 天气工具 get_weather() 是真实查询还是模拟？

    get_weather(city: str) -> str 是一个模拟天气工具，它不调用真实天气 API，而是直接返回预设字符串。虽然天气数据是模拟的，但 Agent 可以真实地调用该 Python 函数，验证模型与工具之间的调用流程。
4. create_deep_agent() 和 agent.invoke() 分别有什么作用？

    create_deep_agent() 用于创建 Agent，核心配置包括模型（model）、工具列表（tools）和系统提示词（system_prompt）；agent.invoke() 用于执行 Agent 任务，输入通常为包含 messages 的字典，返回包含对话消息等状态信息的字典。模型可根据任务决定是否调用工具，并非每次都必须调用。
5. messages 的标准格式是什么？如何获取 Agent 的结果？

    messages 通常是一个消息列表，每条消息包含角色（role）和内容（content），常见角色有 system、user、assistant、tool。在该示例中，result["messages"][-1].content 用于获取最后一条消息的内容，通常就是 Agent 的最终回答。
6. 工具函数中的 topic 和 Literal 有什么作用？

    topic 是搜索类型参数，用于指定通用搜索（general）、新闻搜索（news）或金融搜索（finance）；Literal["general", "news", "finance"] 是类型标注，表示参数预期只能取这三个值，Agent 框架可以据此生成工具参数约束，但 Python 本身不会自动强制校验。
7. Docstring 与普通 Python 注释有什么区别？

    普通注释（#）主要用于解释代码，Docstring（"""..."""）是可以被程序读取的文档字符串。在 Agent 开发中，框架可以读取工具函数的 Docstring，将其作为工具描述提供给大模型，帮助模型理解工具的用途及调用场景。
8. tools=[get_weather] 为什么不写成 tools=[get_weather()]？

    get_weather 表示函数对象本身，传入 Agent 后可以在需要时被调用；get_weather() 表示立即执行函数并取得返回值。注册 Agent 工具时，需要传递函数对象，而不是函数的执行结果。
9. ast 和 operator 分别是什么？

    ast 和 operator 都是 Python 标准库。ast 用于将字符串形式的 Python 表达式解析为抽象语法树（AST），识别其中的数字、运算符及运算结构；operator 提供加减乘除等运算函数，如 operator.add(2, 3) 等价于 2 + 3。
10. calculate() 函数如何实现数学公式计算？

    calculate() 首先使用 ast.parse() 将数学表达式字符串解析为语法树，再通过递归函数 evaluate() 遍历节点，根据节点类型选择对应的 operator 运算函数，最终得到计算结果。这是真实的本地计算，不是模拟计算，也不需要大模型参与具体运算。
11. 为什么使用 ast 而不是直接使用 eval()？

    eval() 可以执行任意符合要求的 Python 表达式，处理不可信输入时存在代码执行风险；通过 ast 解析表达式并仅允许指定的数字和运算节点，可以限制执行范围，安全性更高，但仍需配合输入长度、复杂度和资源限制。

核心总结： Agent 的基本工作模式是 用户提出任务 → 大模型分析并决定是否调用工具 → Agent 框架执行工具 → 工具返回结果 → 大模型生成最终回答。其中，大模型主要负责决策和推理，工具负责实际操作，Agent 框架负责组织和调度整个执行过程。

---

## 附录
### 相关实现代码1
天气调用 + 计算
```python
import os
from langchain_openai import ChatOpenAI
from deepagents import create_deep_agent

# 通过API接入模型（兼容 OpenAI 接口）
model = ChatOpenAI(
    # 用 MODEL_NAME 和 SILICONFLOW_API_KEY 环境变量覆盖
    model=os.environ.get("MODEL_NAME"),
    api_key=os.environ["SILICONFLOW_API_KEY"],
    base_url="http://xxx/v1",
)

# 这是一个用于获取天气的工具，这个工具返回的数据信息目前只是写死的并没有实际调用天气数据，但是这个工具是Agent可以实际调用的
def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

# 这是一个计算工具，由Agent调用
def calculate(expression: str) -> float:
    """Evaluate a math expression and return the result.

    Args:
        expression: A math expression, e.g. "1 + 2 * 3".
    """
    # 只解析数字、括号和四则运算，不执行模型生成的 Python 代码
    import ast
    import operator

    if len(expression) > 200:
        raise ValueError("表达式过长")
    tree = ast.parse(expression, mode="eval")
    if sum(1 for _ in ast.walk(tree)) > 64:
        raise ValueError("表达式过于复杂")
    operations = {ast.Add: operator.add, ast.Sub: operator.sub,
                  ast.Mult: operator.mul, ast.Div: operator.truediv}

    def evaluate(node):
        if isinstance(node, ast.Constant) and type(node.value) in (int, float):
            return float(node.value)
        if isinstance(node, ast.BinOp) and type(node.op) in operations:
            return operations[type(node.op)](evaluate(node.left), evaluate(node.right))
        if isinstance(node, ast.UnaryOp) and isinstance(node.op, (ast.UAdd, ast.USub)):
            value = evaluate(node.operand)
            return value if isinstance(node.op, ast.UAdd) else -value
        raise ValueError("只支持数字、括号和 + - * / 运算")

    return evaluate(tree.body)

# 这是一个汇率转换工具，同样由Agent调用
def convert_currency(amount: float, from_currency: str, to_currency: str = "CNY") -> dict:
    """Convert an amount from one currency to another.

    Args:
        amount: The amount to convert.
        from_currency: The source currency code, e.g. "USD".
        to_currency: The target currency code, defaults to "CNY".
    """
    # 这里用固定汇率做演示；真实场景可接入汇率 API
    rates = {"USD": 7.2, "CNY": 1.0, "EUR": 7.8}
    cny = amount * rates[from_currency]
    return {"amount": round(cny / rates[to_currency], 2), "currency": to_currency}

# 创建一个专门获取天气的Agent工具，用来获取天气信息
agent = create_deep_agent(
    model=model,
    tools=[get_weather],
    system_prompt="You are a helpful assistant.",
)

# 返回获取的天气信息结果并打印
result = agent.invoke(
    {"messages": [{"role": "user", "content": "北京今天天气怎么样？"}]}
)

print(result["messages"][-1].content)

# 创建一个专门进行汇率计算的Agent工具
agent = create_deep_agent(
    model=model,
    tools=[calculate, convert_currency],
    system_prompt="你是一个计算助手，能帮用户做数学运算和货币换算。",
)

# 返回计算结果并打印
result = agent.invoke(
    {"messages": [{"role": "user", "content": "帮我把 100 美元换算成人民币，再用它乘以 1.08 的通胀系数。"}]}
)
print(result["messages"][-1].content)
```

### 相关实现代码2
一个搜索工具`task2-search-tools.py`
```python
import os
from typing import Literal

from tavily import TavilyClient
from langchain_openai import ChatOpenAI
from deepagents import create_deep_agent
from langchain.agents.middleware import TodoListMiddleware


# 1. 创建 Tavily 搜索客户端
tavily_client = TavilyClient(
    api_key=os.environ["TAVILY_API_KEY"]
)


# 2. 定义搜索工具
def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
) -> dict:
    """Run a web search for the given query.

    Args:
        query: The search query string.
        max_results: Maximum number of results to return.
        topic: The topic category for the search.
        include_raw_content: Whether to include raw page content.
    """
    print(f"\n[工具调用] 正在搜索：{query}")

    return tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )

# 3. 创建大模型连接对象
model = ChatOpenAI(
    model=os.environ["AGENTSEEK_MODEL"],
    api_key=os.environ["OPENAI_API_KEY"],
    base_url=os.environ["OPENAI_API_BASE"],
    temperature=0,
)


# 4. 定义系统提示词
research_instructions = """
你是一位专业的研究员。
你的工作是进行深入研究，然后撰写研究报告。
你可以使用 internet_search 工具搜索互联网。
回答时请尽可能提供来源。
"""


# 5. 创建 Agent
agent = create_deep_agent(
    model=model,
    tools=[internet_search],
    system_prompt=research_instructions,
    middleware=[TodoListMiddleware()],
)


# 6. 执行 Agent
print("开始执行 Agent...")

result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",
                "content": "请搜索互联网，简要介绍 雾霾天气一般是由什么造成 。"
            }
        ]
    },
    config={"recursion_limit": 30},
)


# 7. 打印最终结果
print("\n========== Agent 最终回答 ==========\n")
print(result["messages"][-1].content)

print("\n========== 测试完成 ==========")


```

































---

End.