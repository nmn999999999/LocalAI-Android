# LocalAI Android

离线 AI 聊天应用 - Android 版本

## 功能特点

- **完全离线运行** - 所有 AI 推理在设备本地完成
- **llama.cpp 推理引擎** - 使用 GGUF 格式模型
- **Jetpack Compose UI** - 现代 Android 界面设计
- **Agent 工具调用** - 支持计算器、搜索、笔记等工具
- **API 模式** - 支持 OpenAI 兼容 API

## 系统要求

- Android 8.0+ (API 26)
- arm64-v8a 架构
- 6GB+ RAM 推荐

## 构建

### 前置条件

- Android Studio
- Android SDK
- Android NDK 26.3.11579264
- JDK 17

### 编译 Native 库

```bash
./build_native.sh
```

### 构建 APK

```bash
./gradlew assembleDebug
```

APK 输出: `app/build/outputs/apk/debug/`

## 项目结构

```
app/src/main/
├── java/com/localai/app/
│   ├── agent/          # Agent 服务
│   ├── data/           # 数据模型
│   ├── llm/            # LLM 引擎
│   ├── store/          # 存储服务
│   └── ui/             # UI 界面
├── cpp/                # JNI 桥接代码
├── jniLibs/            # 预编译 .so 库
└── res/                # 资源文件
```

## 支持的模型

从 [HuggingFace](https://huggingface.co/) 下载 GGUF 格式模型，推荐:

- Qwen3 0.6B/1.7B
- Phi-4-mini 3.8B
- Llama 3.2 3B
- Gemma 3 4B

## 许可证

MIT License
