### Day1
> 预准备与创建环境

准备本机环境：
这一步可有可无，实际可以在自己电脑上直接安装使用，虚拟机采用`Ubuntu 24.04.5 LTS`
预先准备，为了使环境运行干净安全，采用虚拟机的方式，所有环境配置与操作均在虚拟机中完成，远程虚拟机方式：
`ssh -J lizhengrui@192.xxx lizhengrui@192.xxx`

开始环境配置：
```bash
# Linux 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh
# 新增命令：uv需要激活
source $HOME/.local/bin/env
# 安装 AgentSeek
uv tool install --upgrade agentseek
agentseek version
agentseek --help

# 查看当前 CLI 识别的模板
# 修改命令：原始命令不可用，参考备注Endnote-Commend-1
agentseek create --list-templates
# 创建项目并从模板仓库获取最新批次模板并进入目录
agentseek create deepagents/research --no-input
cd research_deepagent

# 查看项目摘要
agentseek info
# 查看模板提供的一次性任务
agentseek task --list

# 运行模板声明的后端依赖
agentseek task sync
# 安装前端依赖
agentseek task frontend

# 复制后端和前端环境文件：
cp .env.example .env
cp frontend/.env.example frontend/.env

# 修改根目录下的.env文件
nano .env
# 环境文件中需要固定格式注意中间不能有多余的空格，Key参考备注Endnote-Key-1
AGENTSEEK_MODEL_PROVIDER=openai
OPENAI_API_BASE=http://xxx
OPENAI_API_KEY=sk-xxx
AGENTSEEK_MODEL=xxx
# 下面这个TAVILY_API_KEY需要申请，参考备注Endnote-Key-2
TAVILY_API_KEY=tvly-xxx
CTRL+O ENTER CTRL+X保存退出

# 运行就绪检查
agentseek doctor

# 预览开发进程，不启动服务
agentseek dev --dry-run

# 修改根目录下的.env文件
nano .env
# 将 false 改为 true 开启追踪
LANGSMITH_TRACING=true
# 填入你刚才在网页上生成的真实 Key
LANGSMITH_API_KEY=lsv2_pt_xxxxxxxxxxxxxxxxx
# 这个名字可以随便起，你的运行记录会被归类到这个项目名下
LANGSMITH_PROJECT=deepagents-course
# 默认 US 区域的 Endpoint，如果是亚太请改为 https://apac.api.smith.langchain.com，默认申请是US区
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
CTRL+O ENTER CTRL+X保存退出

# 启动前后端
agentseek dev
# 需要在MobaXterm新开一个窗口执行以下命令【可选，只针对远程虚拟机的情况】
ssh -J lizhengrui@192.xxx -L 5174:127.0.0.1:5174 -L 2024:127.0.0.1:2024 lizhengrui@192.xxx -N
# 然后在windows本地浏览器打开`http://127.0.0.1:5174`即可访问啦
# 输入`Research what LangGraph 1.0 added vs 0.x. Cite sources.`就可以测试啦
# 这里如果页面测试显示：OpenAIAuthenticationError("Error code: 401 - {'code': 30014, 'data': None, 'message': 'Token is invalid.'}")
# 这里是bug，处理方法参考Endnote-Commend-2

```

# Endnote
## Commend
### 1
`agentseek create --list-templates --checkout main`命令修改为`agentseek create --list-templates`命令：
根据终端里的报错 `Standalone template checkout is invalid: standalone catalog checkout ref could not be resolved` 所示，AgentSeek 已经处于对工程可复现性（Reproducibility）的严格要求，在底层封杀了直接传入 main 这种游移分支名的行为。
从第一段成功的 `agentseek create --list-templates` 输出可以看出，当前 CLI 绑定的本地锁定目录中，已经包含了课程需要的 deepagents/research 模板。
因此，可以无视教程里关于` --checkout main`的要求，直接使用极其稳定且与你当前环境严格匹配的内置快照来创建课程作业`agentseek create deepagents/research --no-input`

### 2
因为我是采用的虚拟机进行试验，采用的内网接口，所以一定要在另一个窗口中输入以下命令以进行连接：
``
如果在windows的浏览器打开`http://127.0.0.1:5174`，并输入`Research what LangGraph 1.0 added vs 0.x. Cite sources.`之后发现报错：penAIAuthenticationError("Error code: 401 - {'code': 30014, 'data': None, 'message': 'Token is invalid.'}")
1、需要先测试本地内网接口是否能正常联通使用：
```bash
curl http://192.xxx/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxx" \
  -d '{
    "model": "deepseek-xxx",
    "messages": [{"role": "user", "content": "测试一下，收到请回复"}]
  }'
```
2、需要检查配置文件是否准确
`nano .env`查看key等配置是否正确，是否有多余空格
3、如果以上两个都没有问题，查看是否设置了多余的变量，清除变量并重新运行：
```bash
# 有些时候你在终端里执行过 export OPENAI_API_KEY=...，这个全局变量的优先级会高于 .env 文件，导致程序一直用旧的错 Key 去请求。
# 在当前终端执行以下命令，强制清空可能残留的旧变量
unset OPENAI_API_KEY
unset OPENAI_API_BASE
unset AGENTSEEK_MODEL

# 强制加载 .env 并启动,执行以下组合命令，强制将当前的 .env 注入环境，并重新启动 AgentSeek
set -a
source .env
set +a

# 确保没有旧的后台进程卡死端口
pkill -f agentseek-api

# 重新启动
agentseek dev
```

## Key
### 1
AGENTSEEK_MODEL_PROVIDER=openai
OPENAI_API_BASE=http://xxx/v1
OPENAI_API_KEY=sk-xxx
AGENTSEEK_MODEL=deepseek-xxx
TAVILY_API_KEY=tvly-xxx

### 2
TAVILY_API_KEY需要申请
网址：https://app.tavily.com/home
Key：TAVILY_API_KEY=vly-xxx

### 3
```text
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_xxx
LANGSMITH_PROJECT=deepagents-xxx
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```
LANGSMITH_TRACING需要申请
网址：https://smith.langchain.com/
Key：lsv2_xxx



















End.