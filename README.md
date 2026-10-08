# Gemini API 国内能用吗？Gemini 代理接口配置与主流框架接入教程

每天都有大量的开发者在社区提问：“**Gemini API 国内能用吗？**”“**本地跑 Gemini 接口一直报超时怎么办？**”

答案是：**官方原生的 Gemini API 在国内直连是无法使用的**，会面临严格的 IP 限制和网络阻断。但是，通过**Gemini 代理**或**大模型 API 中转站**，国内开发者完全可以零门槛、低延迟地将 Gemini 接入到自己的业务代码或 AI 框架中。

**国内最推荐的 AI API 中转站平台：**

> AI API 中转站平台地址：<https://quanzil.com>

> AI API 中转站平台地址：<https://quanzil.net>

本文将详细介绍什么是 Gemini API 代理，如何获取接口权限，以及如何在 Python 纯代码、LangChain、Dify 等主流 AI 开发框架中快速配置 Gemini 模型。

---

## 为什么你需要一个 Gemini 中转站？

如果你尝试过自己去 Google AI Studio 申请 API，你通常会遇到这“三座大山”：

1. **注册与绑卡阻碍**：需要海外节点，且一旦超出极其有限的免费配额，就需要绑定海外实体信用卡才能继续使用。
2. **网络超时报错**：部署在阿里云、腾讯云等国内服务器上的程序，无法直接向 Google 接口发送 HTTP 请求，会直接报 `Connection Timeout`。
3. **接口格式不兼容**：Google 的官方 SDK（`@google/genai`）和目前行业事实上的标准（OpenAI 格式）不同。如果你想让一个项目同时支持 GPT 和 Gemini，原生的写法需要写两套完全独立的逻辑。

**Gemini API 中转站**（代理平台）完美解决了这三个问题：
- **网络直连**：提供国内可直接访问的 API 域名（Base URL）。
- **统一计费**：支持国内常见支付方式，按 Token 消耗计费，用多少充多少。
- **OpenAI 格式兼容**：中转站会在底层自动将 OpenAI 的请求格式转换为 Gemini 认识的格式。这意味着，**你可以直接用调用 GPT 的代码去调用 Gemini**。

---

## 准备工作：获取你的 Gemini 代理凭证

在使用后续的任何代码或框架前，你需要先在中转站平台获取两个核心参数：

1. **API Key**：你的专属身份令牌（通常以 `sk-` 开头）。请妥善保管，不要泄漏到公开的 GitHub 仓库。
2. **Base URL**：中转平台的请求基地址。如果是 OpenAI 兼容接口，通常长这样：`https://your-api-domain.com/v1`。

---

## 基础代码接入：Python 与 Node.js 实战

我们先来看看最基础的纯代码调用方式，这适合自己写后端服务的开发者。

### Python 接入（使用 OpenAI SDK）

因为中转站支持 OpenAI 兼容格式，请先安装 OpenAI 官方包：

```bash
pip install openai
```

然后，直接替换 `api_key` 和 `base_url`，并将 `model` 指定为 Gemini 的模型名称即可：

```python
import os
from openai import OpenAI

# 1. 配置中转站的 API 信息
client = OpenAI(
    api_key="YOUR_PROXY_API_KEY", 
    base_url="https://your-api-domain.com/v1"
)

def chat_with_gemini(prompt):
    # 2. 调用具体的 Gemini 模型，比如 3.8 Flash
    response = client.chat.completions.create(
        model="gemini-3.8-flash",
        messages=[
            {"role": "system", "content": "你是一个资深的前端开发工程师。"},
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
        max_tokens=1000
    )
    return response.choices[0].message.content

print(chat_with_gemini("请用 React 写一个简单的待办事项列表组件。"))
```

### Node.js 接入（原生 Fetch）

如果你不想引入第三方 SDK，也可以直接用原生的 `fetch` 发送请求：

```javascript
const apiKey = "YOUR_PROXY_API_KEY";
const baseUrl = "https://your-api-domain.com/v1/chat/completions";

async function askGemini() {
  const response = await fetch(baseUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${apiKey}`
    },
    body: JSON.stringify({
      model: "gemini-2.0-flash", // 切换为 Gemini 2.0
      messages: [
        { role: "user", content: "什么是大模型 API 中转？" }
      ],
      temperature: 0.5
    })
  });

  const data = await response.json();
  if (data.choices && data.choices.length > 0) {
    console.log(data.choices[0].message.content);
  } else {
    console.error("请求失败:", data);
  }
}

askGemini();
```

---

## 进阶教程：在主流 AI 框架中配置 Gemini 代理

现在的 AI 应用开发很少从零手写，大多依赖于优秀的开源框架。得益于中转平台的 OpenAI 兼容性，接入这些框架非常简单。

### 1. LangChain 接入 Gemini 中转站

在 LangChain 中，你**不需要**使用 `ChatGoogleGenerativeAI` 模块，而是直接使用 `ChatOpenAI` 模块。

```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage

# 通过环境变量注入中转站配置，LangChain 会自动读取
os.environ["OPENAI_API_KEY"] = "YOUR_PROXY_API_KEY"
os.environ["OPENAI_API_BASE"] = "https://your-api-domain.com/v1"

# 初始化模型时，强行指定为 Gemini 模型名
llm = ChatOpenAI(
    model="gemini-3.8-flash",
    temperature=0.3
)

messages = [
    SystemMessage(content="你是一个专业的英汉翻译专家。"),
    HumanMessage(content="Translate this: Using an API proxy simplifies integration.")
]

response = llm.invoke(messages)
print(response.content)
```
*优势：你的 RAG（检索增强生成）流程、Agent 逻辑完全不需要动，换模型只需改个字符串。*

---

### 2. Dify 配置 Gemini 接口

Dify 是目前最火的低代码 AI 应用编排平台。如果你使用本地部署的 Dify，配置步骤如下：

1. 登录 Dify 后台，点击右上角进入 **设置 (Settings)** -> **模型供应商 (Model Providers)**。
2. 不要选 Google 卡片！找到 **OpenAI** 或者 **OpenAI-API-compatible**（兼容 OpenAI 的自定义供应商）卡片。
3. 点击添加模型：
   - **模型类型 (Model Type)**：选择 `LLM`（文本生成）。
   - **模型名称 (Model Name)**：手动输入 `gemini-3.8-flash` 或中转站支持的模型。
   - **API 端点 (API Endpoint)**：填写 `https://your-api-domain.com/v1`。
   - **API Key**：填写中转站提供的 Key。
4. 验证并保存。之后你在搭建工作流时，就可以直接拖拽使用 Gemini 模型了。

---

### 3. FastGPT / AnythingLLM 配置

这些知识库问答工具的配置逻辑与 Dify 相似：
- 找到“大模型配置”或“添加自定义大模型”的入口。
- 将接口提供商选为 `OpenAI`。
- 修改 `Base URL` 为中转站地址。
- 填入中转站的 `API Key`。
- 在可用模型列表中，自定义添加 `gemini-3.8-flash` 或 `gemini-3.8-pro`。

---

## Gemini 核心模型参数怎么填？

中转平台通常会聚合多个大厂的模型，你在代码里的 `model` 参数必须精确匹配平台提供的模型名称，否则会报错。

常用的 Gemini 模型填写规范如下（以平台实际支持为准）：

*   `gemini-3.8-flash`：最常用的轻量模型，速度极快，价格极其便宜，适合绝大多数文本摘要、问答、语言翻译场景。
*   `gemini-2.0-flash`：新一代高性价比模型，指令遵循和多模态能力更强。
*   `gemini-3.8-pro`：重型推理模型，价格较贵，适合处理复杂的代码逻辑、数学推理和长篇大论的深度分析。

**避坑提示**：不要自己发明模型名字（比如写成 `gemini-pro-3.8` 或 `google-gemini`），一定要严格参考 API 中转站后台的【可用模型列表】文档。

---

## 常见报错处理指南

如果你在接入过程中遇到问题，可以对照以下 HTTP 状态码进行排查：

### HTTP 401 Unauthorized
*   **原因**：鉴权失败。
*   **解决办法**：检查 API Key 是否复制正确，注意不要有前导或尾随空格；如果是自己构造 Headers，务必确保加上了 `Authorization: Bearer <你的API_KEY>`。

### HTTP 404 Not Found
*   **原因**：请求路径不对，或者找不到这个模型。
*   **解决办法**：
    1. 检查 `Base URL` 是否以 `/v1` 结尾（使用 OpenAI SDK 时必须带 `/v1`）。
    2. 检查 `model` 参数填写的名称是否在中转站支持的模型列表中。

### HTTP 429 Too Many Requests
*   **原因**：请求频率过高，触发了限流（Rate Limit），或者是账户余额已耗尽。
*   **解决办法**：登录中转平台查看账户余额。如果余额充足，说明并发太高，请在代码中增加 Sleep 延迟，或者使用指数退避（Exponential Backoff）进行重试。

---

## 总结

回答文章开头的问题：**Gemini API 国内能用吗？**
能用，而且通过**Gemini API 中转站**接入，体验甚至比官方原生的方案更好。

通过 OpenAI 兼容协议，你不仅解决了国内网络直连和海外支付的阻碍，还能将 Gemini 无缝嵌入到现有的代码体系和如 LangChain、Dify 等优秀的 AI 框架中。

```
