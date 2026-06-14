# SQL-LangChain — 基于 LangChain SQL Agent 的 Text2SQL 智能体

本项目用 **LangChain 的 SQL Agent**（`create_sql_agent` + `SQLDatabaseToolkit`）实现 Text2SQL：你用一句自然语言提出数据需求，大模型（DeepSeek-V3）作为智能体**自主规划**——先列出数据库有哪些表、查看相关表的结构、编写并校验 SQL、执行查询，最后用自然语言把结果回答给你。

相比「一次性把 Schema 塞进 Prompt 让模型直接吐 SQL」的做法，Agent 方案能**自己探查库表、按需查 Schema、对 SQL 自检纠错**，更接近真实的「自助式取数」体验。

---

## 与 SQL Copilot 的区别

| | 直接生成（SQL Copilot） | 智能体（本项目） |
| --- | --- | --- |
| 思路 | 把表结构 + 问题塞进 Prompt，模型一次性生成 SQL | 给 Agent 一个目标，它**多步规划**：列表→查 Schema→写 SQL→校验→执行→回答 |
| Schema | 需要人工准备字段说明 / DDL 喂给模型 | Agent 运行时**自己调用工具**探查库表结构 |
| 纠错 | 生成后再单独评测能否运行 | Agent 内置 `sql_db_query_checker` 自检，并可根据报错自我修正 |
| 输出 | SQL 语句 | 直接给出查询结果的自然语言回答 |

> 直接生成的版本见姊妹项目 [SQL Copilot](https://github.com/zhenyuduan98/SQL-Copilot)。

---

## 核心功能

- **自然语言一问到底**：输入「找出英雄攻击力最高的前 5 个英雄」「查询所有未支付保费的保单号和客户姓名」，Agent 自动完成从理解到执行的全过程。
- **自主探查数据库**：通过 LangChain `SQLDatabaseToolkit` 提供的工具集，Agent 自己决定查哪些表、看哪些字段。
- **SQL 自检与纠错**：执行前先用 `sql_db_query_checker` 检查常见错误，降低跑错的概率。
- **两个演示场景**：
  - `sql_life_insurance.*` —— 面向**寿险业务库** `life_insurance`（客户、保单、理赔、产品等表）。
  - `sql_agent_deepseek.*` —— 面向**多表综合库** `action`（含英雄 `heros`、订单 `orders`、客户 `customers` 等），演示描述表结构、处理不存在的表、Top-N 查询等。

---

## Agent 工作流程

`create_sql_agent` 基于 ReAct 模式驱动，单次问答内部会循环调用 `SQLDatabaseToolkit` 的工具：

```
用户提问
   │
   ▼
sql_db_list_tables   ── 列出数据库所有表
   │
   ▼
sql_db_schema        ── 查看相关表的建表语句与样例数据
   │
   ▼
sql_db_query_checker ── 校验生成的 SQL 是否有常见错误
   │
   ▼
sql_db_query         ── 执行 SQL，拿到结果
   │
   ▼
自然语言回答
```

---

## 目录结构

```
SQL-LangChain/
├── README.md
├── .gitignore
├── requirements.txt
├── sql_life_insurance.py      # 寿险库 life_insurance 的 SQL Agent（脚本版）
├── sql_life_insurance.ipynb   # 同上（Notebook 版，含运行过程与结果）
├── sql_agent_deepseek.py      # 综合库 action 的 SQL Agent（脚本版）
└── sql_agent_deepseek.ipynb   # 同上（Notebook 版，含 Agent 推理轨迹）
```

---

## 快速开始

### 1. 安装依赖

```bash
# 建议 Python 3.10+
pip install -r requirements.txt
```

主要依赖：`langchain`、`openai`、`pandas`、`pymysql`（连接 MySQL）；`requirements.txt` 另列 `qdrant_client`、`vanna`（基于 RAG 的 Text2SQL 框架，可作进一步扩展）。

### 2. 配置 API Key

模型走 DashScope 的 OpenAI 兼容接口，通过环境变量提供 Key：

```bash
# Linux / macOS
export DASHSCOPE_API_KEY="你的_API_Key"

# Windows PowerShell
$env:DASHSCOPE_API_KEY="你的_API_Key"
```

### 3. 运行

直接跑脚本，或在 Jupyter 中打开对应 notebook 逐格执行：

```bash
python sql_life_insurance.py      # 寿险库示例
python sql_agent_deepseek.py      # 综合库示例
```

把 `agent_executor.run("...")` 里的问题换成你自己的需求即可。

> 代码中的数据库连接信息为课程提供的共享教学库，可按需替换为自己的数据库。

---

## 技术栈

- **框架**：LangChain（`create_sql_agent`、`SQLDatabaseToolkit`、`SQLDatabase`）
- **大模型**：DeepSeek-V3（经 DashScope OpenAI 兼容端点调用）
- **数据库**：MySQL（`pymysql` 驱动）
- **可选扩展**：Vanna、Qdrant（向量检索 / RAG Text2SQL）
- **运行环境**：Python、Jupyter Notebook

---

