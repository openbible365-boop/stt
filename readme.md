# 语音对话应用 — Voice Chat
一个基于 Google AI (Gemini) API 的语音转文字对话应用。按住左 Shift 键录音，松开自动将语音转为文字并发送到对话中。
---
## 功能概览
| 功能 | 说明 |
|------|------|
| Google 登录 | 使用 Google 账号安全登录，用户信息保存在本地 |
| 语音录制 | 按住**左 Shift 键**开始录音，松开自动停止 |
| 语音转文字 | 使用 Gemini 2.0 Flash 模型将音频转录为文字 |
| AI 对话 | 转录的文字自动发送给 Gemini，获取 AI 回复 |
| 对话历史 | 所有对话记录保存在浏览器 localStorage 中 |
| 对话管理 | 支持新建对话、切换对话、删除不需要的对话记录 |
---
## 技术架构
```
┌─────────────────────────────────────────────┐
│                  前端 (HTML)                  │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │ Google   │  │ 语音录制  │  │ 对话界面   │ │
│  │ 登录模块  │  │ 模块     │  │ 模块      │ │
│  └────┬─────┘  └────┬─────┘  └─────┬─────┘ │
│       │              │              │       │
│  ┌────▼──────────────▼──────────────▼─────┐ │
│  │          localStorage 持久化           │ │
│  └────────────────────────────────────────┘ │
└──────────────────┬──────────────────────────┘
                   │ HTTPS API 调用
      ┌────────────▼────────────┐
      │  Google AI (Gemini API) │
      │  - 语音转文字           │
      │  - 对话生成             │
      └────────────────────────┘
```
---
## 使用的 API 和服务
### Google OAuth 2.0
- **Client ID**: `307380593339-lkskv3k92uc2qamj0gebkkfkmfjce045.apps.googleusercontent.com`
- 使用 Google Identity Services (GIS) 库
- 通过 JWT 解析获取用户信息（姓名、邮箱、头像）
### Google AI — Gemini API
- **模型**: `gemini-2.0-flash`
- **API 端点**: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- **两个用途**:
  1. **语音转文字**: 将录制的音频 (WebM/Opus) 以 base64 编码发送给 Gemini，提示模型进行转录
  2. **对话回复**: 将用户的文字消息（包含上下文历史）发送给 Gemini，获取 AI 回复
---
## 操作指南
### 1. 登录
打开页面后，点击「使用 Google 登录」按钮，在弹出的 Google 窗口中选择你的账号完成登录。
### 2. 语音录制与转文字
| 操作 | 效果 |
|------|------|
| 按住左 Shift 键 | 开始录音，状态指示器变红显示「录音中...」 |
| 松开左 Shift 键 | 停止录音，音频自动发送到 Gemini 进行转录 |
| 转录完成 | 文字自动填入输入框并发送，AI 开始回复 |
> **注意**: 首次录音时浏览器会请求麦克风权限，请点击「允许」。
### 3. 文字输入
也可以直接在输入框中打字，按 Enter 键发送消息（Shift+Enter 换行）。
### 4. 对话管理
- **新建对话**: 点击侧边栏顶部的 `+` 按钮
- **切换对话**: 在侧边栏中点击对话项
- **删除对话**: 将鼠标悬停在对话项上，点击右侧的 `✕` 按钮
- **退出登录**: 点击侧边栏底部的「退出」按钮
---
## 文件结构
```
index.html    ← 完整的单文件应用（HTML + CSS + JS）
README.md              ← 本文档
```
应用是一个**单文件 HTML**，不需要构建工具或服务器，直接在浏览器中打开即可使用。

### 方式二：本地服务器
```bash
# Python
python3 -m http.server 8080
# Node.js
npx serve .
# 然后访问 http://localhost:8080/index.html
- **Firebase Hosting**
确保在 Google Cloud Console 中将托管域名添加到 OAuth 2.0 客户端的「已授权的 JavaScript 来源」中。
---
## Google Cloud Console 配置
### OAuth 2.0 配置
1. 打开 [Google Cloud Console](https://console.cloud.google.com/)
2. 进入「API 和服务」→「凭据」
3. 找到对应的 OAuth 2.0 客户端 ID
4. 在「已授权的 JavaScript 来源」中添加你的域名，例如：
   - `http://localhost:8080`（本地开发）
   - `https://yourdomain.com`（生产环境）
### Gemini API 配置
1. 进入「API 和服务」→「已启用的 API」
2. 确保已启用 **Generative Language API**
3. API 密钥已内嵌在代码中
---
## 数据存储
所有数据存储在浏览器的 `localStorage` 中：
| 键名 | 内容 |
|------|------|
| `vc_user` | 当前登录用户信息 (JSON) |
| `vc_chats_{email}` | 该用户的所有对话记录 (JSON) |
数据结构示例：
```json
{
  "id": "1710000000000",
  "title": "对话标题（取自第一条消息）",
  "createdAt": "2026-03-16T10:00:00.000Z",
  "messages": [
    {
      "role": "user",
      "content": "你好，今天天气怎么样？",
      "time": "2026-03-16T10:00:01.000Z"
    },
    {
      "role": "assistant",
      "content": "你好！我是 AI 助手...",
      "time": "2026-03-16T10:00:03.000Z"
    }
  ]
}
```
---
## 浏览器兼容性
| 浏览器 | 支持情况 |
|--------|---------|
| Chrome 90+ | ✅ 完全支持 |
| Edge 90+ | ✅ 完全支持 |
| Firefox 85+ | ✅ 支持（需允许麦克风） |
| Safari 15+ | ⚠️ WebM 录制可能需要 polyfill |
**必需权限**: 麦克风访问权限（首次使用时浏览器会弹出提示）
---
## 安全注意事项
1. **API 密钥**: 当前 API 密钥直接嵌入前端代码中。生产环境建议通过后端代理调用 API，避免密钥泄露。
2. **数据存储**: 对话数据仅保存在浏览器本地，清除浏览器数据会丢失所有记录。
3. **Google 登录**: 使用 Google 官方 Identity Services 库，登录凭证不会被存储。
---
## 自定义修改
### 修改 AI 行为
在代码中找到 `callGemini` 函数，可以调整：
```javascript
generationConfig: {
  temperature: 0.7,      // 创造性程度 (0-1)
  maxOutputTokens: 2048  // 回复最大长度
}
```
### 修改录音提示语
在代码中搜索 `请将这段音频转录为文字`，可以修改转录的系统提示。
### 修改主题配色
在 CSS 的 `:root` 中修改 CSS 变量：
```css
:root {
  --bg: #0a0a0f;          /* 主背景 */
  --accent: #7c6cf0;      /* 主题色 */
  --accent2: #9d8ff8;     /* 高亮色 */
}
