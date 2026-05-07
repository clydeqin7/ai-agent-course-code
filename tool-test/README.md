# tool-test

基于 [LangChain](https://js.langchain.com/) 的 Node.js 示例项目，演示 OpenAI 兼容模型的对话、工具调用（Tool Calling）以及结合命令行创建 React 应用的完整流程。

## 项目结构

```bash
tool-test/
├── src/
│   ├── hello-langchain.mjs   # 最小示例：调用模型做自我介绍
│   ├── tool-file-read.mjs    # 单工具示例：read_file 读取文件并让模型解释
│   ├── all-tools.mjs         # 通用工具集合：读写文件、执行命令、列出目录
│   ├── mini-cursor.mjs       # 迷你版“Cursor Agent”：用多工具完成复杂任务
│   └── node-exec.mjs         # 直接在 Node 中执行创建 Vite React Todo 项目的命令
├── react-todo-app/           # 由脚本创建的 Vite + React + TS Todo 示例应用
├── package.json
├── pnpm-lock.yaml
└── README.md
```

## 依赖

- **@langchain/core** / **@langchain/openai**：LangChain JS SDK，用于调用大模型与工具调用
- **dotenv**：加载 `.env` 环境变量
- **zod**：工具参数校验

（`react-todo-app` 目录下有自己独立的 React/Vite 相关依赖，可在该目录内单独安装与运行。）

## 环境配置

在项目根目录创建 `.env` 文件（已被 `.gitignore` 忽略），配置：

```env
OPENAI_API_KEY=你的API密钥
OPENAI_BASE_URL=你的API基础URL（兼容 OpenAI 的接口）
# 可选，默认 qwen-coder-turbo
MODEL_NAME=qwen-coder-turbo
```

## 脚本说明

### 1. 基础对话：`src/hello-langchain.mjs`

最简单的 LangChain 示例，直接向模型发送「介绍下自己」，并打印回复：

```bash
node src/hello-langchain.mjs
```

### 2. 单工具调用示例：`src/tool-file-read.mjs`

- 使用 `@langchain/core/tools` 定义 **read_file** 工具，根据路径读取文件内容；
- 将工具绑定到模型，通过消息流（HumanMessage → AIMessage + tool_calls → ToolMessage）实现：  
  用户请求「读取并解释某文件」→ 模型调用 `read_file` → 根据返回内容生成解释。

运行：

```bash
node src/tool-file-read.mjs
```

默认会请求模型读取并解释 `src/tool-file-read.mjs` 自身，可在代码中修改 `HumanMessage` 内容测试其他文件。

### 3. 通用工具集合：`src/all-tools.mjs`

封装了可复用的四个工具：

- `read_file`：读取文件内容；
- `write_file`：写入文件（自动创建目录）；
- `execute_command`：在指定工作目录中执行系统命令，实时输出；
- `list_directory`：列出指定目录下的文件和文件夹。

这些工具会被 `mini-cursor.mjs` 等脚本复用。

### 4. 迷你 Cursor Agent：`src/mini-cursor.mjs`

通过 `ChatOpenAI` + `all-tools.mjs` 中的多工具，模拟一个简单的“项目管理助手”：

- 根据系统提示和用户自然语言任务，自动选择：
  - 读取/写入文件；
  - 在指定 `workingDirectory` 下执行命令（如在 `react-todo-app` 中安装依赖、启动 dev 服务器）；
  - 列出目录结构。
- 内置提示会强调：**当指定了 `workingDirectory` 时，不要在命令里再写 `cd`**，避免路径错误。

你可以修改 `mini-cursor.mjs` 中的 `case1` 文本，让 Agent 自动完成不同的工程化任务。

运行：

```bash
node src/mini-cursor.mjs
```

### 5. 一键创建 React Todo 应用：`src/node-exec.mjs`

如果只想直接在 Node 中运行命令，不经过 LangChain Agent，可以使用这个脚本：

- 内置命令：  
  `echo -e "n\nn" | pnpm create vite react-todo-app --template react-ts`
- 使用 `child_process.spawn` 在当前工作目录下执行，并继承终端输出。

运行：

```bash
node src/node-exec.mjs
```

执行成功后，会在仓库根目录下生成 `react-todo-app` 子项目。

### 6. React Todo 应用：`react-todo-app/`

`react-todo-app` 是通过 Vite+React+TypeScript 创建的前端项目，你可以在其中实现/扩展一个功能完整、样式精致的 Todo 应用。

在该目录中安装依赖并启动开发服务器：

```bash
cd react-todo-app
pnpm install
pnpm run dev
```

## 根项目安装与运行汇总

```bash
# 在仓库根目录安装依赖（推荐 pnpm）
pnpm install

# 运行各个示例
node src/hello-langchain.mjs      # 基础对话
node src/tool-file-read.mjs       # 单工具读取文件示例
node src/mini-cursor.mjs          # 多工具 Agent 示例
node src/node-exec.mjs            # 直接执行创建 React 项目的命令
```

## 许可证

ISC
