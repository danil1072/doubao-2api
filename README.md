# doubao-2api - 豆包转 OpenAI API ✨

<div align="center">

![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)
![GitHub Repo stars](https://img.shields.io/github/stars/lzA6/doubao-2api?style=social)
![Docker](https://img.shields.io/badge/Docker-Ready-blue?logo=docker)
![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-green?logo=fastapi)

**English | [中文](#)**

> "我们并非在编写代码，而是在与数字世界对话。这个项目，就是我们教给计算机的一种新的方言。"

</div>

## 📖 目录

- [🌟 核心特性](#-核心特性)
- [🎯 适用场景](#-适用场景)
- [🚀 快速开始](#-快速开始)
  - [准备工作](#准备工作)
  - [第一步：克隆项目](#第一步克隆项目)
  - [第二步：配置环境](#第二步配置环境)
  - [第三步：启动服务](#第三步启动服务)
  - [第四步：验证部署](#第四步验证部署)
- [🏗️ 系统架构](#️-系统架构)
- [🔧 工作原理](#-工作原理)
- [📊 技术栈详解](#-技术栈详解)
- [🗺️ 项目规划](#️-项目规划)
- [📁 项目结构](#-项目结构)
- [❓ 常见问题](#-常见问题)
- [🤝 贡献指南](#-贡献指南)

## 🌟 核心特性

<div align="center">

| 特性 | 描述 | 状态 |
|------|------|------|
| **OpenAI 兼容性** | 完全模拟 `/v1/chat/completions` 和 `/v1/models` 接口 | ✅ 已实现 |
| **动态签名破解** | 基于 Playwright 的 `a_bogus` 签名生成 | ✅ 已实现 |
| **多账号轮询** | 支持多个豆包 Cookie 负载均衡 | ✅ 已实现 |
| **Docker 部署** | 一键容器化部署方案 | ✅ 已实现 |
| **状态化会话** | 智能对话上下文管理 | ✅ 已实现 |
| **流式响应** | 支持 SSE 流式输出 | ✅ 已实现 |

</div>

## 🎯 适用场景

- 🧑‍💻 **开发者与 AI 爱好者** - 集成免费强大的中文大模型
- 💰 **成本敏感型用户** - 寻找高质量免费 AI 服务替代品
- 📱 **第三方客户端用户** - 支持 ChatGPT-Next-Web、LobeChat 等
- 🔬 **技术探索者** - 学习 Web-API、反爬虫、浏览器自动化技术

## 🚀 快速开始

### 准备工作

1. 安装 `Docker` 和 `docker-compose`
2. 准备可用的豆包账号

### 第一步：克隆项目

```bash
git clone https://github.com/lzA6/doubao-2api.git
cd doubao-2api
```

### 第二步：配置环境

复制环境配置文件：

```bash
cp .env.example .env
```

编辑配置文件：

```bash
nano .env
```

关键配置项：

```env
# API 认证密钥（请修改为复杂字符串）
API_MASTER_KEY="sk-your-secret-key-here"

# 豆包 Cookie（必填）
DOUBAO_COOKIE_1="your_doubao_cookie_here"

# 可选：多账号支持
DOUBAO_COOKIE_2="another_cookie_here"
DOUBAO_COOKIE_3="third_cookie_here"
```

#### 🔍 获取 Cookie 教程

1. 使用 Chrome/Edge 浏览器登录 [豆包官网](https://www.doubao.com/chat/)
2. 按 `F12` 打开开发者工具 → 切换到 **Network** 标签
3. 发送任意消息 → 筛选 `completion` 请求
4. 右键复制为 cURL → 提取 Cookie 字段

### 第三步：启动服务

```bash
docker-compose up -d
```

### 第四步：验证部署

服务启动后，可通过以下方式验证：

**API 配置信息：**
- 地址：`http://your-server-ip:8088`
- 密钥：`.env` 中设置的 `API_MASTER_KEY`
- 模型：`doubao-pro-chat`

**测试请求：**
```bash
curl -X POST "http://localhost:8088/v1/chat/completions" \
  -H "Authorization: Bearer sk-your-secret-key-here" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "doubao-pro-chat",
    "messages": [{"role": "user", "content": "你好"}],
    "stream": false
  }'
```

## 🏗️ 系统架构

<div align="center">

```mermaid
graph TB
    subgraph "客户端 Client"
        A[OpenAI 兼容客户端] --> B[HTTP 请求]
    end
    
    subgraph "doubao-2api 服务层"
        B --> C{Nginx 反向代理}
        C --> D[FastAPI 应用]
        
        subgraph "核心服务 Core Services"
            D --> E[凭证管理器]
            D --> F[会话管理器]
            D --> G[签名服务]
        end
        
        subgraph "浏览器自动化"
            G --> H[Playwright]
            H --> I[无头浏览器]
            I --> J[生成 a_bogus 签名]
        end
    end
    
    subgraph "上游服务 Upstream"
        J --> K[豆包 API]
        K --> L[响应处理]
        L --> M[格式转换]
        M --> A
    end
    
    E --> H
    F --> K
    
    style A fill:#e1f5fe
    style D fill:#f3e5f5
    style G fill:#fff3e0
    style K fill:#e8f5e8
```

</div>

## 🔧 工作原理

### 核心挑战：`a_bogus` 签名机制

豆包网使用复杂的 JavaScript 签名算法生成 `a_bogus` 参数，作为 API 请求的安全验证。

### 解决方案：JS 执行即服务

| 步骤 | 技术实现 | 说明 |
|------|----------|------|
| **1. 环境准备** | Playwright + Stealth | 启动无头浏览器，加载豆包页面 |
| **2. 函数注入** | 执行 `frontierSign` | 调用豆包签名函数 |
| **3. 动态签名** | 参数传递 + JS 执行 | 生成有效的 `a_bogus` 签名 |
| **4. 请求转发** | httpx 客户端 | 携带签名调用豆包 API |
| **5. 格式转换** | OpenAI 兼容格式 | 将响应转换为标准格式 |

### 签名生成流程

```python
# 伪代码示例
async def generate_signature(params):
    # 1. 通过 Playwright 调用浏览器环境
    signature = await page.evaluate("""
        (params) => {
            return window.byted_acrawler.frontierSign(params);
        }
    """, params)
    
    # 2. 将签名附加到请求 URL
    url = f"https://www.doubao.com/api?{params}&a_bogus={signature}"
    
    # 3. 发送请求
    response = await httpx.post(url, cookies=cookies)
    return response
```

## 📊 技术栈详解

<div align="center">

| 技术组件 | 版本 | 用途 | 状态 |
|----------|------|------|------|
| **FastAPI** | 0.100+ | Web API 框架 | ✅ 稳定 |
| **Playwright** | 1.40+ | 浏览器自动化 | ✅ 稳定 |
| **Docker** | 20.10+ | 容器化部署 | ✅ 稳定 |
| **Nginx** | 1.24+ | 反向代理 | ✅ 稳定 |
| **Python** | 3.9+ | 后端语言 | ✅ 稳定 |
| **httpx** | 0.25+ | HTTP 客户端 | ✅ 稳定 |

</div>

### 组件职责说明

- **🎯 FastAPI**: 提供 OpenAI 兼容的 RESTful API 接口
- **🕷️ Playwright**: 管理浏览器实例，执行 JS 签名函数  
- **🐳 Docker**: 封装运行环境，简化部署流程
- **🔀 Nginx**: 负载均衡、请求路由、静态文件服务
- **📦 CredentialManager**: 多账号 Cookie 轮询管理
- **💬 SessionManager**: 对话会话状态维护

## 🗺️ 项目规划

### ✅ 已实现功能 (v1.0)

- [x] OpenAI API 标准兼容
- [x] 动态 `a_bogus` 签名生成
- [x] 流式/非流式响应支持
- [x] 多 Cookie 负载均衡
- [x] Docker 容器化部署
- [x] 内存会话管理
- [x] 设备指纹固化

### 🚧 当前限制

1. **资源消耗** - Playwright 浏览器实例占用较高内存
2. **启动速度** - 冷启动需要 30-60 秒初始化时间
3. **依赖浏览器** - 尚未实现纯 Python 签名算法

### 🔮 未来规划

#### 🎯 高优先级
- [ ] **纯 Python 签名实现** - 移除 Playwright 依赖
- [ ] **Cookie 自动检测** - 失效通知和自动刷新
- [ ] **性能优化** - 减少内存占用，提高并发能力

#### 📈 中优先级  
- [ ] **Web 管理界面** - 可视化配置和监控
- [ ] **Redis 支持** - 分布式会话存储
- [ ] **健康检查** - 服务状态监控和自愈

#### 💡 扩展功能
- [ ] **多模型支持** - 扩展到其他 AI 服务平台
- [ ] **插件架构** - 支持自定义签名算法
- [ ] **API 速率限制** - 防止滥用保护服务

## 📁 项目结构

```
doubao-2api/
├── 🐳 Docker 相关
│   ├── Dockerfile              # 容器构建配置
│   └── docker-compose.yml      # 服务编排配置
├── 🔧 配置文件  
│   ├── .env.example           # 环境变量模板
│   ├── nginx.conf             # Nginx 配置
│   └── requirements.txt       # Python 依赖
├── 🐍 应用代码
│   └── app/
│       ├── main.py            # FastAPI 应用入口
│       ├── core/              # 核心模块
│       │   ├── config.py      # 配置管理
│       │   └── __init__.py
│       ├── providers/         # 服务提供者
│       │   ├── base_provider.py
│       │   └── doubao_provider.py
│       ├── services/          # 业务服务
│       │   ├── credential_manager.py
│       │   ├── playwright_manager.py
│       │   └── session_manager.py
│       └── utils/             # 工具函数
│           └── sse_utils.py
└── 📚 文档
    ├── README.md              # 项目说明
    └── README_en.md           # 英文文档
```

## ❓ 常见问题

### 🔧 技术问题

**Q: 服务启动失败怎么办？**
**A:** 检查以下项目：
1. Docker 和 docker-compose 是否正常安装
2. `.env` 文件中的 Cookie 是否有效
3. 服务器网络能否访问豆包官网

**Q: 请求返回签名错误？**  
**A:** 通常是因为：
1. Cookie 过期 - 重新获取最新 Cookie
2. 浏览器实例异常 - 重启服务 `docker-compose restart`

**Q: 如何查看服务日志？**
**A:** 使用命令：
```bash
docker-compose logs -f app
```

### 💰 费用相关

**Q: 会消耗豆包付费额度吗？**
**A:** 不会，本项目仅调用豆包免费模型，与付费服务无关。

### 🔒 安全相关

**Q: 我的 Cookie 安全吗？**
**A:** 只要部署在可信服务器并设置复杂 API 密钥，您的凭证就是安全的。项目不会向第三方发送任何用户数据。

## 🤝 贡献指南

我们欢迎各种形式的贡献！

### 🐛 报告问题
- 使用 GitHub Issues 报告 bug
- 提供详细的重现步骤和环境信息

### 💡 功能建议  
- 在 Discussions 中提出新想法
- 描述使用场景和预期效果

### 🔨 代码贡献
1. Fork 本仓库
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)  
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建 Pull Request

### 📋 开发规范
- 遵循 PEP 8 Python 代码规范
- 添加适当的注释和文档
- 确保所有测试通过

---

<div align="center">

## 🌟 星星的力量

如果这个项目对您有帮助，请给它一个 ⭐️ 星星！您的支持是我们持续改进的最大动力。

**用代码对话世界，让技术触手可及。**

</div>
