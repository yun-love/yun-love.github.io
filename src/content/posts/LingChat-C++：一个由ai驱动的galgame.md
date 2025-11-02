---
title: LingChat-C++：一个由ai驱动的galgame
published: 2025-08-21
description: ''
image: ''
tags: [C++, termux, esay, AI, LingChat]
category: '教程'
draft: false 
lang: ''
---
### 在Termux中运行LingChat-C++：打造移动端AI聊天伴侣

本文将详细介绍如何在Termux环境中编译和运行LingChat-C++项目，这是一个基于C++开发的AI聊天应用，特别适配了Android终端环境。项目融合了WebSocket通信、RAG记忆系统和情感化交互界面，打造出独特的聊天体验。

---

#### **项目背景**
LingChat-C++是[LingChat](https://github.com/SlimeBoyOwO/LingChat)的二创项目，专为Termux环境优化。它通过以下核心特性提升交互体验：
- ✅ **RAG记忆系统**：实现AI的长期记忆功能(**现已废弃，基于cloudflare的新版正在路上**)
- 🎭 **情感化交互**：立绘表情与音效随对话情绪变化
- 🌙 **昼夜模式**：自动切换的日/夜主题界面
- 🔌 **WebSocket通信**：高效的前后端实时交互

---

#### **环境准备与编译**
**1. 安装工具链与依赖库**
```bash
pkg install curl libcurl clang make openssl nlohmann-json git #自行选择包管理器，如apt，apt-get等
```

**2. 克隆并准备仓库**
```bash
git clone https://github.com/yun-helpr/LingChat-C--Termux.git
mv LingChat-C--Termux chat && cd chat
```

**3. 编译项目**
```bash
make
```
完整编译日志示例：
```txt
❯ make
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/AIEngine.d -c backend/src/AIEngine.cpp -o build/obj/AIEngine.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/ConfigManager.d -c backend/src/ConfigManager.cpp -o build/obj/ConfigManager.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/HTTPClient.d -c backend/src/HTTPClient.cpp -o build/obj/HTTPClient.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/Logger.d -c backend/src/Logger.cpp -o build/obj/Logger.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/MemoryManager.d -c backend/src/MemoryManager.cpp -o build/obj/MemoryManager.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/SessionManager.d -c backend/src/SessionManager.cpp -o build/obj/SessionManager.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/WebSocketServer.d -c backend/src/WebSocketServer.cpp -o build/obj/WebSocketServer.o
aarch64-linux-android-clang++ -std=c++17 -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -MMD -MP -MF build/dep/main.d -c backend/src/main.cpp -o build/obj/main.o
cc -Ibackend/include -Wall  -O3 -DNDEBUG -I/data/data/com.termux/files/usr/include -DOPENSSL_API_3_0 -DUSE_WEBSOCKET -MMD -MP -MF build/dep/civetweb.d -c backend/src/civetweb.c -o build/obj/civetweb.o
aarch64-linux-android-clang++ -L/data/data/com.termux/files/usr/lib build/obj/AIEngine.o build/obj/ConfigManager.o build/obj/HTTPClient.o build/obj/Logger.o build/obj/MemoryManager.o build/obj/SessionManager.o build/obj/WebSocketServer.o build/obj/main.o build/obj/civetweb.o -lcurl -lssl -lcrypto -lpthread -o backend_server
```

---

#### **配置项详解**
编辑`.env`文件配置AI行为：
```txt
# ==================================================
#                      配置文件
# ==================================================

[Server]
BACKEND_PORT="1234" 
DOCUMENT_ROOT="frontend"

[API_LLM]
# 在这里填入你自己模型提供商的 API Key
DEEPSEEK_API_KEY = "your api key"
# LLM API的基础URL
API_BASE_URL = "https://api.deepseek.com/v1"


# --- Embedding模型的配置  ---
[API_EMBEDDING]
# 在这里填入您的Embedding的API Key
EMBEDDING_API_KEY = "your api key"
# 在这里填入其API的基础URL
EMBEDDING_API_URL = "https://api.siliconflow.com/v1/"
# Embedding模型的具体名称
EMBEDDING_MODEL = "Qwen/Qwen3-Embedding-0.6B"
# 该模型的向量维度
EMBEDDING_VECTOR_DIMENSION = "1024"

[Database]
# 轻量级RAG的记忆存储文件，它将自动被创建
MEMORY_DB_PATH = "memory.json"

[AI]
MODEL="deepseek-chat" # 这里填写你所调用的模型名称
TEMPERATURE=0.7 # 这里填写的数值表示模型思维的发散程度，越低发散程度越高，反之亦然。
MAX_HISTORY_TURNS="10" # 这里则数值则表示模型的记忆长度，由于目前市面上绝大多数大模型api都是无状态的，所以我们每次调用模型都需要将上文一同告诉模型，但这样太费token,所以要加以限制，所以模型只会记得包括你这句话的前十句话，但不包括RAG系统。
ENABLE_RAG = false # 这里控制RAG的开关,目前RAG系统为实验性功能，可能无法使用


[Voice]
# 在这里填入您的语音合成API的URL(已废弃)
VOICE_API_URL=""

[Character]
CHARACTER_NAME="钦灵" # 这里是ai的名称，具体如何体现请看web界面
CHARACTER_IDENTITY="可爱的狼娘" # 这里是ai的身份，具体如何体现请看web界面

[UI]
BACKGROUND_DAY_PATH="assets/bg/白天.jpeg" # 这里是白天的背景，当然你也可以换成别的
BACKGROUND_NIGHT_PATH="assets/bg/夜晚.jpg" # 这里是夜晚的背景，同样你也可以换成别的
CHARACTER_SPRITE_DIR="assets/char/ling/" # 这里是人物立绘的文件夹，具体作用可见下文的立绘情绪标签映射表
SFX_DIR="assets/sfx/" #这里是音效文件夹，具体作用可见下文的音效情绪标签映射表
# 注意，所有包含前端资源的文件夹均在frontend中

[EmotionMap] # 这里是立绘情绪标签映射表，你可以将assets/char/ling/中的文件名针对该映射表进行修改(不包括后缀)
高兴=兴奋 # 这里实际在文件夹中的文件名应当是"兴奋.png",下面一样，英文的是我懒的改了
害羞=害羞
生气=厌恶
无语=speechless
惊讶=surprised
哭泣=sad
担心=sad
情动=love
调皮=playful
认真=neutral
疑惑=neutral
尴尬=shy
紧张=sad
自信=happy
害怕=sad
厌恶=angry
羞耻=shy
旁白=兴奋 # 旁边讲话时的立绘，我不想做多人物了
default=兴奋 # 当情绪标签无法识别时默认返回的立绘

[EmotionSfxMap] # 这里是音效情绪标签映射表，工作逻辑同上，文件夹是assets/sfx/
高兴=喜爱.wav
害羞=喜爱.wav
生气=喜爱.wav

[SystemPrompt] # 这里是ai的系统提示词，你可以在项目的根目录下的prompt.txt中修改
PROMPT_FILE="prompt.txt"
    
```

> **配置说明**：
> - RAG系统需额外配置(**现已废弃**)`EMBEDDING_MODEL`和`EMBEDDING_API_URL`
> - 立绘文件需按`[情绪]=[文件名]`格式存放于`assets/char/ling/`
> - 修改`prompt.txt`可定制AI系统级行为指令

---

#### **运行与交互**
**启动服务**：
```bash
make run 
```
成功启动日志：
```txt
❯ make run
======================================
             启动后台服务
======================================
./backend_server
[信息] CivetWeb 库已初始化。
[信息] ConfigManager: 已成功加载并解析配置文件: .env
[信息] 已从 memory.json 加载 6 条记忆。
[信息] HTTPClient 已初始化 (无SSL初始化)。
[信息] HTTPClient 已初始化 (无SSL初始化)。
[信息] AIEngine 已初始化。RAG记忆系统: [已禁用]
[信息] SessionManager 已初始化，最大对话历史记录: 10 轮
[信息] WebSocketServer: WebSocketServer 已初始化。
[信息] WebSocketServer: 正在启动 WebSocket 服务器于 8765...
[信息] WebSocketServer: 正在从以下目录提供静态文件: /data/data/com.termux/files/home/chat/frontend
[信息] WebSocketServer: WebSocket 服务器启动成功。
[信息] WebSocketServer: WebSocket 端点已注册: /websocket
[信息] 服务器已成功启动。主线程进入等待模式。按 Ctrl+C 关闭。
```

**访问前端**：
浏览器打开：`http://localhost:8765/`

**关闭服务**：
```bash
Ctrl + C  
# 终端输出优雅关闭日志
^C
[信息] 收到停止信号，准备关闭服务器...
[信息] 服务器主循环已退出，正在清理...
[信息] WebSocketServer: 正在停止 WebSocket 服务器...
[信息] WebSocketServer: WebSocket 服务器已停止。
[信息] WebSocketServer: WebSocketServer 已销毁。
[信息] 已成功将 6 条记忆保存到 memory.json
[信息] 程序已优雅关闭。
```

---

#### **项目文件结构**
```txt
❯ tree
.
├── backend #后端文件夹
│   ├── include 
│   │   ├── AIEngine.hpp
│   │   ├── ConfigManager.hpp
│   │   ├── HTTPClient.hpp
│   │   ├── Logger.hpp
│   │   ├── MemoryManager.hpp
│   │   ├── SessionManager.hpp
│   │   ├── WebSocketServer.hpp
│   │   ├── civetweb.c
│   │   └── civetweb.h
│   └── src
│       ├── AIEngine.cpp
│       ├── ConfigManager.cpp
│       ├── HTTPClient.cpp
│       ├── Logger.cpp
│       ├── MemoryManager.cpp
│       ├── SessionManager.cpp
│       ├── WebSocketServer.cpp
│       ├── civetweb.c
│       ├── handle_form.inl
│       ├── http2.inl
│       ├── main.cpp
│       ├── match.inl
│       ├── md5.inl
│       ├── mod_duktape.inl
│       ├── mod_lua.inl
│       ├── mod_lua_shared.inl
│       ├── mod_mbedtls.inl
│       ├── mod_zlib.inl
│       ├── openssl_dl.inl
│       ├── response.inl
│       ├── sha1.inl
│       ├── sort.inl
│       ├── timer.inl
│       └── wolfssl_extras.inl
├── backend_server
├── build
│   ├── dep
│   │   ├── AIEngine.d
│   │   ├── ConfigManager.d
│   │   ├── HTTPClient.d
│   │   ├── Logger.d
│   │   ├── MemoryManager.d
│   │   ├── SessionManager.d
│   │   ├── WebSocketServer.d
│   │   ├── civetweb.d
│   │   └── main.d
│   └── obj
│       ├── AIEngine.o
│       ├── ConfigManager.o
│       ├── HTTPClient.o
│       ├── Logger.o
│       ├── MemoryManager.o
│       ├── SessionManager.o
│       ├── WebSocketServer.o
│       ├── civetweb.o
│       └── main.o
├── frontend 前端文件夹
│   ├── app.js
│   ├── assets 前端资源文件夹
│   │   ├── bg
│   │   │   ├── 夜晚.jpg
│   │   │   └── 白天.jpeg
│   │   ├── char
│   │   │   └── ling
│   │   │       ├── 伤心.png
│   │   │       ├── 兴奋.png
│   │   │       ├── 厌恶.png
│   │   │       ├── 头像.png
│   │   │       ├── 害怕.png
│   │   │       └── 害羞.png
│   │   └── sfx
│   │       └── 喜爱.wav
│   ├── index.html
│   └── style.css
├── makefile
├── memory.json
├── prompt.txt
├── readme.md
└── 文件结构.txt

13 directories, 69 files
```

---

#### **常见问题处理**
1. **API调用失败**：
   - 检查`.env`中的`DEEPSEEK_API_KEY`是否有效
   - 确认网络可访问`api.deepseek.com`

2. **立绘/音效不生效**：
   - 确保文件名与`EmotionMap`配置完全匹配
   - 文件需放至`frontend/assets`对应子目录

3. **RAG系统启用**：(**现已废弃,新的RAG系统正在开发**)
   - 设置`ENABLE_RAG=true`
   - 补充配置embedding模型参数

> **提示**：项目日志存储在终端输出，调试时请关注`[信息]`和`[错误]`标签。

---

通过Termux，你现在可以在Android设备上运行完整的AI聊天服务。项目融合了前沿的RAG记忆技术和情感化交互设计，代码已开源，欢迎开发者贡献和改进！遇到技术问题可查阅[原始仓库](https://github.com/yun-helpr/LingChat-C--Termux)或提交Issue。
