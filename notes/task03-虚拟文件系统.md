# task02 虚拟文件系统
> Deep Agents 的 Context Engineering 核心

## 对应文档：
EP.03 虚拟文件系统 — Deep Agents 的 Context Engineering 核心
https://datawhalechina.github.io/deepagents-in-action/chapters/ch03-virtual-filesystem/

## 核心 10 条知识点
1. 虚拟文件系统：为 Agent 提供统一的文件操作接口，让 Agent 将资料、中间结果存入文件，按需读取，避免所有内容长期占用大模型上下文。
2. Backend（后端）：负责执行具体的文件读写、搜索等操作，并决定文件存储在 Agent State、本地磁盘还是持久化存储中；虚拟路径不一定是真实磁盘路径。
3. 7 个内置文件工具：`ls` 列目录、`glob` 找文件、`grep` 搜内容、`read_file` 读内容、`write_file` 创建或覆盖文件、`edit_file` 局部替换、`delete` 删除文件或目录。
4. 文件检索原则：不知道文件名先用 `ls/glob`，不知道内容位置用 `grep`，确定文件后用 `read_file`；大文件优先分片读取，避免无关 Token 消耗。
5. `grep` 三种模式：`files_with_matches` 返回匹配文件路径，`content` 返回匹配行及上下文，`count` 返回匹配数量；搜索结果可能被截断，不能默认已经找到全部内容。
6. 大结果自动卸载：当 **工具返回的结果** 长度超过阈值（教程默认约 20,000 Token）时，系统尝试将完整结果写入 Backend，并用文件引用和少量预览替代长文本。
7. 对话历史自动总结：当 **窗口上下文** 长度达到设定阈值时，系统保存历史消息并生成摘要，使模型后续主要读取摘要和近期消息，而非每次都处理全部历史。
8. 后端选择：`StateBackend` 适合临时文件，`FilesystemBackend` 适合本地磁盘，`StoreBackend` 适合跨会话共享，`CompositeBackend` 适合不同路径使用不同后端。
9. 安全边界：`FilesystemBackend(virtual_mode=True)` 只约束文件工具的路径访问；`LocalShellBackend` 可直接运行宿主机命令，不能仅靠路径配置保证安全。
10. 权限和自定义后端：`FilesystemPermission` 可以按路径控制允许、拒绝或人工审批；自定义 Backend 则用于实现新的存储和文件操作逻辑，两者职责不同。


## 不同 Backend 的代码示例
### StateBackend（临时存储）
```python
from deepagents import create_deep_agent
from deepagents.backends import StateBackend

agent = create_deep_agent(
    model=model,
    backend=StateBackend()
)

```

### FilesystemBackend（真实磁盘）
```python
from pathlib import Path
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend

workspace = Path("./agent_workspace").resolve()
workspace.mkdir(parents=True, exist_ok=True)

agent = create_deep_agent(
    model=model,
    backend=FilesystemBackend(
        root_dir=str(workspace),
        virtual_mode=True
    )
)

```

### StoreBackend（跨会话共享）
```python
from deepagents import create_deep_agent
from deepagents.backends import StoreBackend
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()

agent = create_deep_agent(
    model=model,
    backend=StoreBackend(
        namespace=lambda rt: ("my-agent",)
    ),
    store=store
)

```

### CompositeBackend（混合路由）
> 未来搭建自动化流水线时比较值得掌握的一种方式
```python
from deepagents import create_deep_agent
from deepagents.backends import (
    CompositeBackend,
    StateBackend,
    StoreBackend,
)
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()

agent = create_deep_agent(
    model=model,
    backend=CompositeBackend(
        default=StateBackend(),  # 没有匹配特殊路径时，使用 StateBackend
        routes={
            "/memory/": StoreBackend(
                namespace=lambda rt: ("my-agent",)  # 定义路径与 Backend 的对应关系
            )
        }
    ),
    store=store
)

```
| Agent 操作路径	       |   实际使用后端 |\
|`/workspace/temp.md`     | `StateBackend`|\
|`/workspace/notes.md`    | `StateBackend`|\
|`/memory/knowledge.md`   | `StoreBackend`|\
|`/memory/preferences.md` | `StoreBackend`|

实现了：临时资料放 State，长期共享资料放 Store，Agent 不需要关心底层如何存储。

























End.