基于对DuckyClaw-fork源代码的深入分析，我已经完成了对该AI Agent系统的架构分析。以下是关键发现：

## 一、整体架构概述

DuckyClaw是一个硬件导向的OpenClaw项目，基于TuyaOpen C SDK构建，采用设备-云融合的混合AI Agent架构。核心特点包括：

1. **跨平台部署**：支持MCU（Tuya T5AI、ESP32）、SoC（Raspberry Pi、ARM Linux）和PC（Ubuntu）
2. **统一代码库**：使用C语言编写，无需Node.js运行时
3. **设备-云融合**：通过单一TuyaOpen密钥访问Tuya云平台和设备端AI Agent
4. **统一消息层**：支持Telegram、Discord、Feishu等多种IM通道

## 二、核心架构模块

### 1. **AI Agent核心循环** (`agent/`)
- **agent_loop.c/h**：实现Claw风格的Agent循环，包含外部循环（等待用户消息）和内部工具循环（最多TOOL_LOOP_MAX次迭代）
- **context_builder.c/h**：构建系统提示、历史记录和当前内容的上下文

### 2. **AI组件系统** (`ai_components/`)
- **ai_agent/**：与Tuya AI云服务通信的核心模块
- **ai_mcp/**：MCP（Model Context Protocol）服务器实现，提供工具发现和执行能力
- **ai_skills/**：技能系统，支持自定义技能文件（.md格式）
- **ai_audio/**：音频输入输出处理，支持ASR和TTS
- **ai_mode/**：多种AI模式管理（唤醒、保持、单次、自由等）
- **ai_ui/**：用户界面显示组件
- **ai_video/**：视频处理组件
- **ai_picture/**：图片处理组件

### 3. **即时通讯模块** (`IM/`)
- **消息总线** (`bus/message_bus`)：统一的消息路由系统
- **通道支持** (`channels/`)：Telegram、Discord、Feishu机器人
- **HTTP代理** (`proxy/`)：TLS支持和云连接性
- **CLI接口** (`cli/`)：串行命令行界面

### 4. **工具系统** (`tools/`)
- **tool_cron**：CRON调度和心跳工具
- **tool_files**：文件操作工具
- **tool_exec**：远程代码执行工具（Raspberry Pi）
- **tools_register**：工具注册和分发机制

### 5. **内存管理** (`memory/`)
- **memory_manager**：管理Agent.txt、memory.txt、IoT内存等
- **session_manager**：会话管理

### 6. **网关系统** (`gateway/`)
- **WebSocket服务器**：提供外部客户端连接接口
- **Python客户端示例**：展示如何与AI Agent交互

## 三、关键API接口

### 1. **TuyaOpen AI Agent API**（云端大模型调用接口）
从`ai_agent.c`分析出的关键API：
- `tuya_ai_agent_init()`：初始化AI Agent模块
- `tuya_ai_agent_deinit()`：反初始化AI Agent模块
- `tuya_ai_text_input()`：发送文本输入到AI Agent
- `tuya_ai_file_input()`：发送文件数据到AI Agent
- `tuya_ai_image_input()`：发送图片数据到AI Agent
- `tuya_ai_agent_crt_session()`：创建新的AI会话

### 2. **MCP（Model Context Protocol）API**
从`ai_mcp_server.h`分析出的关键API：
- `ai_mcp_server_init()`：初始化MCP服务器
- `ai_mcp_server_parse_message()`：解析和处理MCP消息
- `ai_mcp_tool_register()`：注册新工具
- `ai_mcp_server_set_tool_exec_hook()`：设置工具执行后钩子

### 3. **设备控制API**
- **Tuya IoT设备控制**：通过TuyaOpen SDK的`tuya_iot_dp_obj_report()`等函数
- **本地设备控制**：通过MCP工具系统实现

### 4. **消息传递API**
- `message_bus_push_inbound()`：推送入站消息到消息总线
- `message_bus_pop_inbound()`：从消息总线弹出入站消息
- `app_im_bot_send_message()`：通过IM通道发送消息

### 5. **WebSocket网关API**
- `ws_server_start()`：启动WebSocket服务器
- `ws_server_send()`：发送文本响应到特定WebSocket客户端
- `ws_server_stop()`：停止WebSocket服务器

## 四、架构设计特点

### 1. **混合AI架构**
- **设备端Agent**：运行在本地硬件上，处理实时响应和硬件控制
- **云端Agent**：访问强大的云端大模型（GPT、Claude、Deepseek、Qwen等）
- **智能切换**：根据任务需求在设备和云端之间动态切换

### 2. **工具调用机制**
- **MCP协议**：基于JSON-RPC 2.0的工具发现和执行协议
- **工具循环**：支持最多10次工具调用迭代（TOOL_LOOP_MAX）
- **结果反馈**：工具执行结果自动反馈到下一轮AI推理

### 3. **上下文管理**
- **系统提示**：包含工具、规则、内存、技能等完整上下文
- **历史记录**：滑动窗口式对话历史管理（最多10条记录）
- **技能摘要**：动态加载技能文件并生成摘要

### 4. **硬件抽象层**
- **TuyaOpen SDK**：提供跨平台的硬件抽象
- **统一驱动**：支持传感器、显示屏、音频、摄像头等硬件
- **IoT集成**：原生支持Tuya IoT设备控制

## 五、关键配置文件

### 1. **TuyaOpen配置** (`include/tuya_app_config.h`)
- `TUYA_PRODUCT_ID`：产品ID
- `TUYA_OPENSDK_UUID`：TuyaOpen SDK UUID
- `TUYA_OPENSDK_AUTHKEY`：TuyaOpen SDK认证密钥

### 2. **IM通道配置**
- `IM_SECRET_CHANNEL_MODE`：通道模式（feishu/telegram/discord）
- 各通道的Token和ID配置

### 3. **硬件配置** (`config/`)
- 针对不同硬件平台的配置文件
- 支持Tuya T5AI、Raspberry Pi、ESP32等

## 六、总结

DuckyClaw的架构设计体现了以下核心理念：

1. **硬件优先**：专为嵌入式设备和边缘计算设计
2. **开放协议**：采用MCP等开放协议实现工具扩展
3. **云边协同**：充分利用云端大模型能力和本地硬件控制
4. **统一接口**：通过消息总线和MCP协议统一各种组件
5. **易于扩展**：模块化设计支持快速添加新功能和硬件

这套架构为开发者提供了一个强大的基础，可以在各种硬件平台上构建智能的AI Agent应用，同时保持与云端大模型的紧密集成。