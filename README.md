# LocalAI Android

一个完全离线运行的 AI 聊天应用，使用 llama.cpp 在设备本地进行 GGUF 模型推理。

![Android](https://img.shields.io/badge/Android-8%2B-green)
![Kotlin](https://img.shields.io/badge/Kotlin-1.9-purple)
![Jetpack Compose](https://img.shields.io/badge/Jetpack%20Compose-BOM%202024-blue)

---

## 功能特点

- **完全离线运行** - 所有 AI 推理在设备本地完成，无需联网
- **llama.cpp 推理引擎** - 支持 GGUF 格式模型，GPU/NPU 加速
- **Agent 工具调用** - 内置计算器、搜索、笔记、正则提取等工具
- **API 模式** - 支持 OpenAI 兼容 API（GPT-4o、DeepSeek 等）
- **流式输出** - 实时显示 AI 回答
- **Material Design 3** - 现代 Android UI 设计

---

## 系统要求

| 项目 | 要求 |
|------|------|
| Android 版本 | 8.0+ (API 26) |
| CPU 架构 | arm64-v8a |
| 内存 | 6GB+ 推荐 |
| 存储空间 | 2GB+ (含模型) |

---

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/nmn999999999/LocalAI-Android.git
cd LocalAI-Android
```

### 2. 编译 Native 库

需要先安装：
- Android SDK
- Android NDK 26.3.11579264
- CMake

```bash
# 设置 Android SDK 路径（如果未在默认位置）
export ANDROID_NDK=$HOME/Library/Android/sdk/ndk/26.3.11579264

# 编译 llama.cpp
./build_native.sh
```

编译完成后会在 `app/src/main/jniLibs/arm64-v8a/` 生成：
- `liblocalai.so` - 主库
- `libc++_shared.so` - C++ 运行时

### 3. 构建 APK

```bash
./gradlew assembleDebug
```

APK 输出路径：`app/build/outputs/apk/debug/app-debug.apk`

### 4. 安装到手机

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

---

## 项目结构

```
LocalAI-Android/
├── app/src/main/
│   ├── java/com/localai/app/
│   │   ├── MainActivity.kt          # 应用入口
│   │   ├── agent/
│   │   │   └── AgentService.kt      # Agent 循环服务
│   │   ├── data/
│   │   │   ├── AIModelInfo.kt       # 模型元数据
│   │   │   ├── AgentTool.kt         # 工具定义
│   │   │   ├── ChatMessage.kt       # 聊天消息模型
│   │   │   ├── ModelSettings.kt     # 设置模型
│   │   │   └── SearchService.kt     # 搜索服务
│   │   ├── llm/
│   │   │   ├── LLMService.kt        # LLM 服务门面
│   │   │   └── LlamaEngine.kt       # llama.cpp JNI 封装
│   │   ├── store/
│   │   │   ├── ChatStore.kt         # 对话存储
│   │   │   ├── ModelManager.kt      # 模型管理
│   │   │   └── SettingsStorage.kt   # 设置存储
│   │   └── ui/
│   │       ├── MainScreen.kt        # 主界面
│   │       ├── chat/ChatScreen.kt   # 聊天界面
│   │       ├── model/ModelScreen.kt # 模型管理界面
│   │       ├── settings/SettingsScreen.kt
│   │       └── theme/Theme.kt       # 主题配置
│   ├── cpp/
│   │   ├── JniBridge.cpp            # JNI 桥接层
│   │   ├── LlamaBridge.cpp          # llama.cpp C++ 封装
│   │   └── LlamaBridge.h            # C 头文件
│   ├── jniLibs/arm64-v8a/           # 预编译 .so 库
│   └── res/                         # 资源文件
├── build_native.sh                  # llama.cpp 编译脚本
└── gradle/                          # Gradle 配置
```

---

## 技术栈

| 层级 | 技术 |
|------|------|
| **UI** | Kotlin + Jetpack Compose + Material Design 3 |
| **架构** | MVVM + StateFlow |
| **推理引擎** | llama.cpp (C++20) |
| **模型格式** | GGUF (量化模型) |
| **桥接层** | JNI (Java Native Interface) |
| **异步** | Kotlin Coroutines + Flow |

---

## 如何修改

### 1. 添加新工具

在 `app/src/main/java/com/localai/app/data/AgentTool.kt` 中添加：

```kotlin
// 在 BuiltInTools.allTools 列表中添加
AgentToolDefinition(
    id: "my_tool",
    name: "my_tool",
    description: "我的自定义工具",
    parameters: [
        "input": .init(type: "string", description: "输入参数", enumValues: nil)
    ]
)
```

然后在 `execute()` 函数中添加实现：

```kotlin
case "my_tool":    return executeMyTool(arguments: arguments)
```

### 2. 修改模型列表

在 `app/src/main/java/com/localai/app/data/AIModelInfo.kt` 中修改：

```kotlin
static let catalog: [AIModelInfo] = [
    AIModelInfo(
        id: "my-model",
        name: "我的模型",
        repo: "user/repo-GGUF",
        fileName: "model-Q4_K_M.gguf",
        sizeDescription: "~2 GB",
        description: "模型描述",
        templateType: .chatML,
        supportsMultimodal: false,
        supportsToolCalling: true
    ),
    // ... 其他模型
]
```

### 3. 修改界面主题

在 `app/src/main/java/com/localai/app/ui/theme/Theme.kt` 中修改颜色：

```kotlin
private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),        // 主色调
    onPrimary = Color.White,
    primaryContainer = Color(0xFFEADDFF),
    // ... 其他颜色
)
```

### 4. 修改默认设置

在 `app/src/main/java/com/localai/app/data/ModelSettings.kt` 中修改：

```kotlin
data class ModelSettings(
    var temperature: Double = 0.7,      // 温度
    var maxTokens: Int = 2048,          // 最大 token
    var contextLength: Int = 4096,      // 上下文长度
    var systemPrompt: String = "你是一个有帮助的AI助手。请用中文回答。",
    // ...
)
```

### 5. 添加新的 API 端点

在 `ModelSettings.kt` 中添加预设：

```kotlin
// 在 API 端点 TextField 中添加预设选项
val presets = listOf(
    "https://api.openai.com/v1/chat/completions",
    "https://api.deepseek.com/v1/chat/completions",
    "http://localhost:11434/v1/chat/completions",  // Ollama
)
```

---

## 内置工具

| 工具 | 功能 | 参数 |
|------|------|------|
| `calculator` | 数学计算 | `expression`: 表达式 |
| `current_time` | 获取当前时间 | 无 |
| `web_search` | 网页搜索 | `query`: 关键词 |
| `note` | 笔记管理 | `op`, `name`, `content` |
| `clipboard` | 剪贴板操作 | `op`, `text` |
| `hash_text` | 文本哈希 | `text`, `algorithm` |
| `regex_extract` | 正则提取 | `text`, `pattern` |
| `text_summary` | 文本摘要 | `text`, `max_length` |

---

## 支持的模型

从 [HuggingFace](https://huggingface.co/) 下载 GGUF 格式模型：

| 模型 | 大小 | 特点 |
|------|------|------|
| Qwen3 0.6B | ~0.5 GB | 轻量快速 |
| Qwen3 1.7B | ~1.2 GB | 平衡性能 |
| Phi-4-mini 3.8B | ~2.0 GB | 工具调用强 |
| Llama 3.2 3B | ~2.0 GB | 稳定可靠 |
| Gemma 3 4B | ~2.8 GB | 多模态支持 |

推荐量化格式：`Q4_K_M` (平衡大小和质量)

---

## 常见问题

### Q: 编译时报错找不到 NDK

```bash
export ANDROID_NDK=$HOME/Library/Android/sdk/ndk/26.3.11579264
```

### Q: 模型加载失败

1. 确认模型是 GGUF 格式
2. 检查文件是否完整下载
3. 尝试更小的量化版本 (Q4_K_M)

### Q: 推理速度慢

1. 使用真机而非模拟器
2. 选择更小的模型 (0.6B-3B)
3. 在设置中调整 GPU 层数

### Q: 内存不足

1. 使用 Q4 量化模型
2. 减小上下文长度 (contextLength)
3. 关闭其他后台应用

---

## 相关资源

- [llama.cpp](https://github.com/ggerganov/llama.cpp) - 推理引擎
- [GGUF 格式](https://huggingface.co/docs/hub/en/gguf) - 模型格式
- [HuggingFace](https://huggingface.co/) - 模型下载
- [Android NDK](https://developer.android.com/ndk) - NDK 下载

---

## 许可证

MIT License

```
MIT License

Copyright (c) 2024

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
