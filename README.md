# Gemini for Claude Code: An Anthropic-Compatible Proxy

[English Readme](./README.en.md)

这是一个 Claude Code 的代理服务器，旨在无缝桥接 Anthropic Messages API 与 Google Gemini API。它能够实时地将来自客户端（如 Claude Code）的 Anthropic API 格式请求，智能转换为 Gemini 模型所需的格式，并将 Gemini 的响应再转换回 Anthropic 格式。

![Claude Code with Gemini Proxy](image.png)

该项目利用 FastAPI 构建，并通过 LiteLLM 与 Gemini 后端进行交互，确保了高效、稳健的通信。核心功能包括对流式响应、多模态（文本与图像）输入、工具使用（Tool Use）以及系统提示（System Prompt）的全面支持。此外，它还内置了强大的错误处理和恢复机制，能自动应对并修复来自 Gemini API 的不完整数据流，为开发者提供稳定、一致的 API 体验。

最终，这个项目充当了一个强大的 API 适配器，让开发者可以在不修改任何客户端代码的情况下，将为 Anthropic 生态系统构建的工具无缝对接到强大的 Gemini 模型上，极大地增强了技术选型的灵活性和互操作性。

## 主要特性

- **Claude Code 兼容性**: 让 Claude Code CLI 能直接使用 Google Gemini 模型。
- **智能模型映射**: 自动将 Claude Code 的模型请求（如 `haiku`, `sonnet` 别名）映射到您选择的 Gemini 模型。
- **LiteLLM 集成**: 利用 LiteLLM 保证与 Gemini API 交互的健壮性和灵活性。
- **增强的流式传输支持**: 具备强大的错误恢复能力，能处理来自 Gemini 的格式错误的响应块。
- **完整的工具使用支持**: 在 Claude Code 和 Gemini 之间双向转换工具使用（函数调用）的格式。
- **高级错误处理**: 为常见的 Gemini API 问题提供具体、可操作的错误信息和自动回退策略。
- **诊断端点**: 包含 `/health` 和 `/test-connection`，方便快速排查问题。

## 环境准备

- 一个 Google Gemini API 密钥。
- Python 3.8+。
- 已安装 Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)。

## 安装与配置

1.  **克隆仓库**:

    ```bash
    git clone https://github.com/kele527/claude-gemini-bridge.git # 或者你的 fork
    cd claude-gemini-bridge
    ```

2.  **创建并激活虚拟环境** (推荐):

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

3.  **安装依赖**:

    ```bash
    pip install -r requirements.txt
    ```

4.  **配置环境变量**:
    复制环境变量示例文件：

    ```bash
    cp .env.example .env
    ```

    编辑 `.env` 文件，填入您的 Gemini API 密钥，并可按需自定义模型映射和服务器设置：

    ```dotenv
    # 必需: 你的 Google AI Studio API 密钥
    GEMINI_API_KEY="your-google-ai-studio-key"

    # 可选: Claude Code 模型别名映射
    BIG_MODEL="gemini-1.5-pro-latest"    # 用于 'sonnet' 或 'opus' 请求
    SMALL_MODEL="gemini-1.5-flash-latest" # 用于 'haiku' 请求

    # 可选: 服务器设置
    HOST="0.0.0.0"
    PORT="8082"
    ```

5.  **运行服务器**:
    ```bash
    python server.py
    ```
    对于开发模式，使用以下命令以实现自动重载：
    ```bash
    uvicorn server:app --host 0.0.0.0 --port 8082 --reload
    ```

## 如何配合 Claude Code 使用

1.  **启动代理服务器**: 确保本项目（代理服务器）正在运行。

2.  **配置 Claude Code 使用代理**:
    在运行 Claude Code 时，设置 `ANTHROPIC_BASE_URL` 环境变量：

    ```bash
    ANTHROPIC_BASE_URL=http://localhost:8082 claude
    ```

3.  **使用 `CLAUDE.md` (关键步骤)**:
    为了让 Gemini 更好地理解 Claude Code 的指令，请将本仓库中的 `CLAUDE.md` 文件复制到您的项目根目录。这个文件包含了针对性的指令，能显著提升 Gemini 的表现。

## 工作原理

1.  **Claude Code 请求**: 您在 Claude Code CLI 中发出一个指令。
2.  **代理转换 (Anthropic -> Gemini)**: 代理服务器接收到 Anthropic 格式的请求，将其转换为 Gemini 兼容的格式。
3.  **LiteLLM 到 Gemini**: LiteLLM 将准备好的请求发送到 Gemini API。
4.  **代理转换 (Gemini -> Anthropic)**: 代理服务器接收 Gemini 的响应，处理可能出现的流式错误，并将其转换回 Anthropic 格式。
5.  **响应到 Claude Code**: 格式化后的响应被送回 Claude Code 客户端并显示。

## 贡献

欢迎提交问题、功能请求或代码贡献！

## 致谢

该项目深受 [claude-code-proxy by @1rgs](https://github.com/1rgs/claude-code-proxy) 的启发，并建立在其开创性的工作之上。
