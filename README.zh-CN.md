<div align="center">

# OhMyCaptcha

**面向 flow2api 与类似集成场景的可自托管 YesCaptcha 风格验证码服务**

[English README](README.md) · [在线文档](https://shenhao-stu.github.io/ohmycaptcha/) · [Render 部署指南](https://shenhao-stu.github.io/ohmycaptcha/zh/deployment/render/) · [Hugging Face Spaces 指南](https://shenhao-stu.github.io/ohmycaptcha/zh/deployment/huggingface/)

<p>
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10%2B-blue.svg">
  <img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-0.119-009688.svg">
  <img alt="Playwright" src="https://img.shields.io/badge/Playwright-Chromium-2EAD33.svg">
  <img alt="Docs" src="https://img.shields.io/badge/docs-MkDocs%20Material-526CFE.svg">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-black.svg">
</p>

![OhMyCaptcha Hero](docs/assets/ohmycaptcha-hero.svg)

</div>

OhMyCaptcha 是一个高质量、可自托管的验证码解决服务，为本仓库已实现的任务类型提供 **YesCaptcha 风格异步 API**。它面向 **flow2api**、内部路由层，以及其他依赖 `createTask` / `getTaskResult` 语义的系统。

---

## 为什么是 OhMyCaptcha

托管式打码平台很方便，但很多自托管工作流会更在意以下能力：

- **部署自主权** —— 服务运行在你自己的环境里
- **模型路由灵活性** —— 可接入托管或自托管的 OpenAI-compatible 多模态后端
- **求解流程透明** —— 浏览器执行路径可理解、可验证，而不是黑盒
- **集成兼容性** —— 保持 YesCaptcha 风格 API，降低现有客户端接入成本

> 更准确地说，OhMyCaptcha 是一个**针对本仓库已实现任务类型的、自托管 YesCaptcha-compatible 服务**，而不是对所有商业打码平台能力的完整对标。

---

## 你能获得什么

- **YesCaptcha 风格 API 接口**
  - `POST /createTask`
  - `POST /getTaskResult`
  - `POST /getBalance`
  - `GET /api/v1/health`
- **reCAPTCHA v3 任务支持**
  - `RecaptchaV3TaskProxyless`
  - `RecaptchaV3TaskProxylessM1`
  - `RecaptchaV3TaskProxylessM1S7`
  - `RecaptchaV3TaskProxylessM1S9`
- **图片验证码识别支持**
  - `ImageToTextTask`
- **FastAPI + Playwright 架构**，用于浏览器求解流程
- **OpenAI-compatible 多模态集成**，用于图片验证码识别
- **面向 Render 与 Hugging Face Spaces 的部署路径**
- **双语 GitHub Pages 文档站**
- **可本地复用的 skills**，适用于 Claude Code、OpenCode 与类似 agent 环境

---

## 架构一览

```text
Client / flow2api
        │
        ▼
  POST /createTask
        │
        ▼
 In-memory TaskManager
        │
   ┌────┴────┐
   ▼         ▼
Playwright   OpenAI-compatible
reCAPTCHA    multimodal model
solver       backend
   │         │
   └────┬────┘
        ▼
 POST /getTaskResult
```

### 核心实现

- **FastAPI** 提供 HTTP API
- **Playwright + Chromium** 生成 reCAPTCHA v3 token
- **OpenAI-compatible 多模态接口** 分析图片验证码
- **内存型异步任务管理器** 负责后台执行

---

## 支持的任务类型

### reCAPTCHA v3

服务启动时会注册以下任务类型：

- `RecaptchaV3TaskProxyless`
- `RecaptchaV3TaskProxylessM1`
- `RecaptchaV3TaskProxylessM1S7`
- `RecaptchaV3TaskProxylessM1S9`

当前这四种任务类型在本仓库中共用同一套浏览器求解路径。

### 图片验证码识别

- `ImageToTextTask`

`ImageToTextTask` 采用受 Argus 启发的多模态识别提示词设计，会先将图片归一化到 1440×900 坐标空间，再把结构化识别结果序列化到 `solution.text` 中返回。

---

## 快速开始

### 1. 安装依赖

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
playwright install --with-deps chromium
```

### 2. 配置环境变量

```bash
export CLIENT_KEY="your-client-key"
export CAPTCHA_BASE_URL="https://your-openai-compatible-endpoint/v1"
export CAPTCHA_API_KEY="your-api-key"
export CAPTCHA_MODEL="gpt-5.4"
export CAPTCHA_MULTIMODAL_MODEL="qwen3.5-2b"
export BROWSER_HEADLESS="true"
export BROWSER_TIMEOUT="30"
```

### 3. 启动服务

```bash
python main.py
```

### 4. 进行健康检查

```bash
curl http://localhost:8000/
curl http://localhost:8000/api/v1/health
```

---

## API 示例

### 创建 reCAPTCHA v3 任务

```bash
curl -X POST http://localhost:8000/createTask \
  -H "Content-Type: application/json" \
  -d '{
    "clientKey": "your-client-key",
    "task": {
      "type": "RecaptchaV3TaskProxyless",
      "websiteURL": "https://antcpt.com/score_detector/",
      "websiteKey": "6LcR_okUAAAAAPYrPe-HK_0RULO1aZM15ENyM-Mf",
      "pageAction": "homepage"
    }
  }'
```

### 轮询任务结果

```bash
curl -X POST http://localhost:8000/getTaskResult \
  -H "Content-Type: application/json" \
  -d '{
    "clientKey": "your-client-key",
    "taskId": "uuid-from-createTask"
  }'
```

### 创建图片验证码任务

```bash
curl -X POST http://localhost:8000/createTask \
  -H "Content-Type: application/json" \
  -d '{
    "clientKey": "your-client-key",
    "task": {
      "type": "ImageToTextTask",
      "body": "<base64-encoded-image>"
    }
  }'
```

---

## 配置项

| 变量 | 说明 | 默认值 |
| --- | --- | --- |
| `CLIENT_KEY` | 客户端认证密钥，对应 `clientKey` | 未设置 |
| `CAPTCHA_BASE_URL` | OpenAI-compatible API 基地址 | `https://your-openai-compatible-endpoint/v1` |
| `CAPTCHA_API_KEY` | 模型接口密钥 | 未设置 |
| `CAPTCHA_MODEL` | 强文本模型名称 | `gpt-5.4` |
| `CAPTCHA_MULTIMODAL_MODEL` | 多模态模型名称 | `qwen3.5-2b` |
| `CAPTCHA_RETRIES` | 模型或浏览器失败重试次数 | `3` |
| `CAPTCHA_TIMEOUT` | 模型请求超时（秒） | `30` |
| `BROWSER_HEADLESS` | Chromium 是否无头运行 | `true` |
| `BROWSER_TIMEOUT` | 页面加载超时（秒） | `30` |
| `SERVER_HOST` | 监听地址 | `0.0.0.0` |
| `SERVER_PORT` | 监听端口 | `8000` |

---

## 部署

完整部署文档见文档站：

- [Render 部署](https://shenhao-stu.github.io/ohmycaptcha/zh/deployment/render/)
- [Hugging Face Spaces 部署](https://shenhao-stu.github.io/ohmycaptcha/zh/deployment/huggingface/)
- [完整文档](https://shenhao-stu.github.io/ohmycaptcha/)

---

## Skills

本仓库在 `skills/` 目录下附带可复用的本地 skills：

- `skills/ohmycaptcha/` — 用于部署、验证、集成和运维 OhMyCaptcha
- `skills/ohmycaptcha-image/` — 用于生成 README 与文档所需的公开安全图片

适用于 Claude Code、OpenCode、OpenClaw 风格工作流，以及类似的 agent 环境。

---

## 验收状态

本仓库已经在本地环境下，针对以下公开 reCAPTCHA v3 检测目标完成了验收：

- `https://antcpt.com/score_detector/`
- site key：`6LcR_okUAAAAAPYrPe-HK_0RULO1aZM15ENyM-Mf`

当前已验证的结果是：

- 服务可以正常启动
- 可以成功创建 detector 任务
- 可以轮询到 `ready`
- 可以返回非空的 `solution.gRecaptchaResponse`

本实现**不宣称**可以保证特定 score，也**不宣称**与商业打码平台所有功能完全一致。

---

## 限制说明

- 任务状态保存在**内存中**，并会在 TTL 到期后清理
- `minScore` 为兼容性字段，当前 solver **不会**依据它做分数控制
- `ImageToTextTask` 当前把结构化结果序列化后放在 `solution.text` 中返回
- 实际稳定性取决于浏览器环境、目标站行为、IP 信誉以及模型或供应商质量

---

## 开发

```bash
pytest tests/
npx pyright
python -m mkdocs build --strict
```

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=shenhao-stu/ohmycaptcha&type=Date)](https://www.star-history.com/#shenhao-stu/ohmycaptcha&Date)

## License

本仓库采用 [MIT License](LICENSE)。
