# 代码自动审查 Agent

基于 LLM 的多 Agent 协作代码审查工具，自动扫描 Python 项目并生成 HTML 审查报告。

## 架构

```
code_review_agent/
├── main.py                     # 入口：扫描文件 → 多 Agent 审查 → 生成报告
├── utils.py                    # 工具函数：文件扫描、HTML 报告保存
├── requirements.txt
├── agents/
│   ├── base_agent.py           # 基类：封装 LLM 调用（OpenAI SDK 兼容方式）
│   ├── performance_agent.py    # 性能审查 Agent
│   ├── security_agent.py       # 安全审查 Agent
│   └── style_agent.py          # 代码风格审查 Agent
└── reports/                    # 输出：审查报告 HTML
```

## 工作流程

1. `main.py` 扫描当前目录下所有 `.py` 文件
2. 每个文件分别交由 **3 个 Agent** 审查：
   - **PerformanceAgent** — 性能瓶颈、低效算法
   - **SecurityAgent** — 安全漏洞、注入风险
   - **StyleAgent** — 可读性、命名规范
3. 各 Agent 通过 DeepSeek API 返回结构化的 JSON 结果
4. 汇总生成 `reports/report.html`

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 配置 API Key

编辑 `agents/base_agent.py`，替换 `api_key` 为你的 DeepSeek API Key：

```python
self.client = OpenAI(
    api_key="你的API_KEY",
    base_url="https://api.deepseek.com",
)
```

默认使用 `deepseek-v4-pro` 模型，可在 `BaseAgent.__init__` 中修改 `model` 参数。

### 3. 运行

```bash
python main.py
```

审查报告将生成至 `reports/report.html`，浏览器打开即可查看。

## 报告格式示例

```json
{
  "file.py": {
    "PerformanceAgent": {"issues": [{"line": 15, "description": "循环内重复查询"}]},
    "SecurityAgent":  {"issues": [{"line": 8, "description": "SQL 注入风险"}]},
    "StyleAgent":     {"issues": [{"line": 23, "description": "函数命名不规范"}]}
  }
}
```

## 依赖

- Python >= 3.9
- openai >= 1.0.0（兼容任何 OpenAI 接口规范的 LLM 服务）
