# CLIProxyAPI

> 反代 XAI，实现 Token 自由与无限制内容访问

<p align="center">
  <img src="./_resources/a8acc6fa55f794a8dd89180ef7e4a07e.png" alt="CLIProxyAPI Banner" width="800"/>
</p>

---

## 目录

- [简介](#简介)
- [项目地址](#项目地址)
- [服务器选购](#服务器选购)
- [部署教程](#部署教程)
- [配置说明](#配置说明)
- [更新项目](#更新项目)
- [常见问题](#常见问题)

---

## 简介

本教程介绍如何将 **CLIProxyAPI** 通过 Docker 部署到服务器，实现 24 小时调用自由，并支持分享给朋友、同事使用。

---

## 项目地址

- GitHub: [https://github.com/router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI)

---

## 服务器选购

> **注意：** 部署服务器时，**不要选择国内地区**，建议选择 Linux 版本，使用 Docker 上手更快。

- **推荐配置：** 腾讯云（新加坡 / 硅谷 / 东京 / 首尔)
- **价格：** 199 元/年
- **配置：** 2核 4G 内存，30M 带宽，60GB SSD，1.5T 月流量
- **系统：** Ubuntu 24
- **购买地址：** [https://curl.qcloud.com/oyWDLkRJ](https://curl.qcloud.com/oyWDLkRJ)

<p align="center">
  <img src="./_resources/3832b2ca64da72076cb50108c41078c0.png" alt="腾讯云服务器推荐" width="600"/>
</p>

---

## 部署教程

### 1. 创建项目目录

在 Ubuntu 24 系统中创建一个文件夹来存储项目文件：

```bash
mkdir CLIProxyAPI
cd CLIProxyAPI
```

<p align="center">
  <img src="./_resources/b7a83e43803546161a0241090577810e.png" alt="创建项目目录" width="600"/>
</p>

### 2. 创建配置文件

创建配置文件，路径为 `/home/ubuntu/CLIProxyAPI/config.yaml`：

```bash
touch config.yaml
```

<p align="center">
  <img src="./_resources/9cb839e88df45c68c6d481d4deb7dbcd.png" alt="创建配置文件" width="600"/>
</p>

### 3. 编辑配置文件

使用 `nano` 编辑配置文件：

```bash
nano config.yaml
```

**填入以下内容：**

```yaml
# Server host/interface to bind to. Default is empty ("") to bind all interfaces (IPv4 + IPv6).
# Use "127.0.0.1" or "localhost" to restrict access to local machine only.
host: ""

# Server port
port: 8317

# TLS settings for HTTPS. When enabled, the server listens with the provided certificate and key.
tls:
  enable: false
  cert: ""
  key: ""

# Management API settings
remote-management:
  # Whether to allow remote (non-localhost) management access.
  # When false, only localhost can access management endpoints (a key is still required).
  allow-remote: false

  # Management key. If a plaintext value is provided here, it will be hashed on startup.
  # All management requests (even from localhost) require this key.
  # Leave empty to disable the Management API entirely (404 for all /v0/management routes).
  secret-key: ""

  # Disable the bundled management control panel asset download and HTTP route when true.
  disable-control-panel: false

  # Disable automatic periodic background updates of the management panel from GitHub (default: false).
  # When enabled, the panel is only downloaded on first access if missing, and never auto-updated afterward.
  # disable-auto-update-panel: false

  # GitHub repository for the management control panel. Accepts a repository URL or releases API URL.
  panel-github-repository: "https://github.com/router-for-me/Cli-Proxy-API-Management-Center"

# Authentication directory (supports ~ for home directory)
auth-dir: "~/.cli-proxy-api"

# API keys for authentication
api-keys:
  - "your-api-key-1"
  - "your-api-key-2"
  - "your-api-key-3"

# Enable debug logging
debug: false

# Enable pprof HTTP debug server (host:port). Keep it bound to localhost for safety.
pprof:
  enable: false
  addr: "127.0.0.1:8316"

# When true, disable high-overhead HTTP middleware features to reduce per-request memory usage under high concurrency.
commercial-mode: false

# When true, write application logs to rotating files instead of stdout
logging-to-file: false

# Maximum total size (MB) of log files under the logs directory. When exceeded, the oldest log
# files are deleted until within the limit. Set to 0 to disable.
logs-max-total-size-mb: 0

# Maximum number of error log files retained when request logging is disabled.
# When exceeded, the oldest error log files are deleted. Default is 10. Set to 0 to disable cleanup.
error-logs-max-files: 10

# When false, disable in-memory usage statistics aggregation
usage-statistics-enabled: false

# Proxy URL. Supports socks5/http/https protocols. Example: socks5://user:pass@192.168.1.1:1080/
# Per-entry proxy-url also supports "direct" or "none" to bypass both the global proxy-url and environment proxies explicitly.
proxy-url: ""

# When true, unprefixed model requests only use credentials without a prefix (except when prefix == model name).
force-model-prefix: false

# When true, forward filtered upstream response headers to downstream clients.
# Default is false (disabled).
passthrough-headers: false

# Number of times to retry a request. Retries will occur if the HTTP response code is 403, 408, 500, 502, 503, or 504.
request-retry: 3

# Maximum number of different credentials to try for one failed request.
# Set to 0 to keep legacy behavior (try all available credentials).
max-retry-credentials: 0

# Maximum wait time in seconds for a cooled-down credential before triggering a retry.
max-retry-interval: 30

# When true, disable auth/model cooldown scheduling globally (prevents blackout windows after failure states).
disable-cooling: false

# Core auth auto-refresh worker pool size (OAuth/file-based auth token refresh).
# When > 0, overrides the default worker count (16).
# auth-auto-refresh-workers: 16

# Quota exceeded behavior
quota-exceeded:
  switch-project: true # Whether to automatically switch to another project when a quota is exceeded
  switch-preview-model: true # Whether to automatically switch to a preview model when a quota is exceeded
  antigravity-credits: true # Whether to retry Antigravity quota_exhausted 429s once with enabledCreditTypes=["GOOGLE_ONE_AI"]

# Routing strategy for selecting credentials when multiple match.
routing:
  strategy: "round-robin" # round-robin (default), fill-first
  # Enable universal session-sticky routing for all clients.
  # Session IDs are extracted from: X-Session-ID header, Idempotency-Key,
  # metadata.user_id, conversation_id, or first few messages hash.
  # Automatic failover is always enabled when bound auth becomes unavailable.
  session-affinity: false # default: false
  # How long session-to-auth bindings are retained. Default: 1h
  session-affinity-ttl: "1h"

# When true, enable authentication for the WebSocket API (/v1/ws).
ws-auth: false

# When true, enable Gemini CLI internal endpoints (/v1internal:*).
# Default is false for safety.
enable-gemini-cli-endpoint: false

# When > 0, emit blank lines every N seconds for non-streaming responses to prevent idle timeouts.
nonstream-keepalive-interval: 0
# Streaming behavior (SSE keep-alives + safe bootstrap retries).
# streaming:
#   keepalive-seconds: 15   # Default: 0 (disabled). <= 0 disables keep-alives.
#   bootstrap-retries: 1    # Default: 0 (disabled). Retries before first byte is sent.

# Signature cache validation for thinking blocks (Antigravity/Claude).
# When true (default), cached signatures are preferred and validated.
# When false, client signatures are used directly after normalization (bypass mode for testing).
# antigravity-signature-cache-enabled: true

# Bypass mode signature validation strictness (only applies when signature cache is disabled).
# When true, validates full Claude protobuf tree (Field 2 -> Field 1 structure).
# When false (default), only checks R/E prefix + base64 + first byte 0x12.
# antigravity-signature-bypass-strict: false

# Gemini API keys
# gemini-api-key:
#   - api-key: "AIzaSy...01"
#     prefix: "test" # optional: require calls like "test/gemini-3-pro-preview" to target this credential
#     base-url: "https://generativelanguage.googleapis.com"
#     headers:
#       X-Custom-Header: "custom-value"
#     proxy-url: "socks5://proxy.example.com:1080"
#     # proxy-url: "direct" # optional: explicit direct connect for this credential
#     models:
#       - name: "gemini-2.5-flash" # upstream model name
#         alias: "gemini-flash"    # client alias mapped to the upstream model
#     excluded-models:
#       - "gemini-2.5-pro"     # exclude specific models from this provider (exact match)
#       - "gemini-2.5-*"       # wildcard matching prefix (e.g. gemini-2.5-flash, gemini-2.5-pro)
#       - "*-preview"          # wildcard matching suffix (e.g. gemini-3-pro-preview)
#       - "*flash*"            # wildcard matching substring (e.g. gemini-2.5-flash-lite)
#   - api-key: "AIzaSy...02"

# Codex API keys
# codex-api-key:
#   - api-key: "sk-atSM..."
#     prefix: "test" # optional: require calls like "test/gpt-5-codex" to target this credential
#     base-url: "https://www.example.com" # use the custom codex API endpoint
#     headers:
#       X-Custom-Header: "custom-value"
#     proxy-url: "socks5://proxy.example.com:1080" # optional: per-key proxy override
#     # proxy-url: "direct" # optional: explicit direct connect for this credential
#     models:
#       - name: "gpt-5-codex"   # upstream model name
#         alias: "codex-latest" # client alias mapped to the upstream model
#     excluded-models:
#       - "gpt-5.1"         # exclude specific models (exact match)
#       - "gpt-5-*"         # wildcard matching prefix (e.g. gpt-5-medium, gpt-5-codex)
#       - "*-mini"          # wildcard matching suffix (e.g. gpt-5-codex-mini)
#       - "*codex*"         # wildcard matching substring (e.g. gpt-5-codex-low)

# Claude API keys
# claude-api-key:
#   - api-key: "sk-atSM..." # use the official claude API key, no need to set the base url
#   - api-key: "sk-atSM..."
#     prefix: "test" # optional: require calls like "test/claude-sonnet-latest" to target this credential
#     base-url: "https://www.example.com" # use the custom claude API endpoint
#     headers:
#       X-Custom-Header: "custom-value"
#     proxy-url: "socks5://proxy.example.com:1080" # optional: per-key proxy override
#     # proxy-url: "direct" # optional: explicit direct connect for this credential
#     models:
#       - name: "claude-3-5-sonnet-20241022" # upstream model name
#         alias: "claude-sonnet-latest"      # client alias mapped to the upstream model
#     excluded-models:
#       - "claude-opus-4-5-20251101" # exclude specific models (exact match)
#       - "claude-3-*"               # wildcard matching prefix (e.g. claude-3-7-sonnet-20250219)
#       - "*-thinking"               # wildcard matching suffix (e.g. claude-opus-4-5-thinking)
#       - "*haiku*"                  # wildcard matching substring (e.g. claude-3-5-haiku-20241022)
#     cloak:                         # optional: request cloaking for non-Claude-Code clients
#       mode: "auto"                 # "auto" (default): cloak only when client is not Claude Code
#                                    # "always": always apply cloaking
#                                    # "never": never apply cloaking
#       strict-mode: false           # false (default): prepend Claude Code prompt to user system messages
#                                    # true: strip all user system messages, keep only Claude Code prompt
#       sensitive-words:             # optional: words to obfuscate with zero-width characters
#         - "API"
#         - "proxy"
#       cache-user-id: true          # optional: default is false; set true to reuse cached user_id per API key instead of generating a random one each request
#     experimental-cch-signing: false # optional: default is false; when true, sign the final /v1/messages body using the current Claude Code cch algorithm
#                                     # keep this disabled unless you explicitly need the behavior, so upstream seed changes fall back to legacy proxy behavior

# Default headers for Claude API requests. Update when Claude Code releases new versions.
# In legacy mode, user-agent/package-version/runtime-version/timeout are used as fallbacks
# when the client omits them, while OS/arch remain runtime-derived. When
# stabilize-device-profile is enabled, OS/arch stay pinned to the baseline values below,
# while user-agent/package-version/runtime-version seed a software fingerprint that can
# still upgrade to newer official Claude client versions.
# claude-header-defaults:
#   user-agent: "claude-cli/2.1.44 (external, sdk-cli)"
#   package-version: "0.74.0"
#   runtime-version: "v24.3.0"
#   os: "MacOS"
#   arch: "arm64"
#   timeout: "600"
#   stabilize-device-profile: false  # optional, default false; set true to enable per-auth/API-key fingerprint pinning

# Default headers for Codex OAuth model requests.
# These are used only for file-backed/OAuth Codex requests when the client
# does not send the header. `user-agent` applies to HTTP and websocket requests;
# `beta-features` only applies to websocket requests. They do not apply to codex-api-key entries.
# codex-header-defaults:
#   user-agent: "codex_cli_rs/0.114.0 (Mac OS 14.2.0; x86_64) vscode/1.111.0"
#   beta-features: "multi_agent"

# OpenAI compatibility providers
# openai-compatibility:
#   - name: "openrouter" # The name of the provider; it will be used in the user agent and other places.
#     prefix: "test" # optional: require calls like "test/kimi-k2" to target this provider's credentials
#     base-url: "https://openrouter.ai/api/v1" # The base URL of the provider.
#     headers:
#       X-Custom-Header: "custom-value"
#     api-key-entries:
#       - api-key: "sk-or-v1-...b780"
#         proxy-url: "socks5://proxy.example.com:1080" # optional: per-key proxy override
#         # proxy-url: "direct" # optional: explicit direct connect for this credential
#       - api-key: "sk-or-v1-...b781" # without proxy-url
#     models: # The models supported by the provider.
#       - name: "moonshotai/kimi-k2:free" # The actual model name.
#         alias: "kimi-k2"               # The alias used in the API.
#         thinking:                      # optional: omit to default to levels ["low","medium","high"]
#           levels: ["low", "medium", "high"]
#       # You may repeat the same alias to build an internal model pool.
#       # The client still sees only one alias in the model list.
#       # Requests to that alias will round-robin across the upstream names below,
#       # and if the chosen upstream fails before producing output, the request will
#       # continue with the next upstream model in the same alias pool.
#       - name: "deepseek-v3.1"
#         alias: "claude-opus-4.66"
#       - name: "glm-5"
#         alias: "claude-opus-4.66"
#       - name: "kimi-k2.5"
#         alias: "claude-opus-4.66"

# Vertex API keys (Vertex-compatible endpoints, base-url is optional)
# vertex-api-key:
#   - api-key: "vk-123..."                        # x-goog-api-key header
#     prefix: "test"                              # optional: require calls like "test/vertex-pro" to target this credential
#     base-url: "https://example.com/api"         # optional, e.g. https://zenmux.ai/api; falls back to Google Vertex when omitted
#     proxy-url: "socks5://proxy.example.com:1080" # optional per-key proxy override
#     # proxy-url: "direct" # optional: explicit direct connect for this credential
#     headers:
#       X-Custom-Header: "custom-value"
#     models:                                     # optional: map aliases to upstream model names
#       - name: "gemini-2.5-flash"                # upstream model name
#         alias: "vertex-flash"                   # client-visible alias
#       - name: "gemini-2.5-pro"
#         alias: "vertex-pro"
#     excluded-models:                            # optional: models to exclude from listing
#       - "imagen-3.0-generate-002"
#       - "imagen-*"

# Amp Integration
# ampcode:
#   # Configure upstream URL for Amp CLI OAuth and management features
#   upstream-url: "https://ampcode.com"
#   # Optional: Override API key for Amp upstream (otherwise uses env or file)
#   upstream-api-key: ""
#   # Per-client upstream API key mapping
#   # Maps client API keys (from top-level api-keys) to different Amp upstream API keys.
#   # Useful when different clients need to use different Amp accounts/quotas.
#   # If a client key isn't mapped, falls back to upstream-api-key (default behavior).
#   upstream-api-keys:
#     - upstream-api-key: "amp_key_for_team_a"    # Upstream key to use for these clients
#       api-keys:                                 # Client keys that use this upstream key
#         - "your-api-key-1"
#         - "your-api-key-2"
#     - upstream-api-key: "amp_key_for_team_b"
#       api-keys:
#         - "your-api-key-3"
#   # Restrict Amp management routes (/api/auth, /api/user, etc.) to localhost only (default: false)
#   restrict-management-to-localhost: false
#   # Force model mappings to run before checking local API keys (default: false)
#   force-model-mappings: false
#   # Amp Model Mappings
#   # Route unavailable Amp models to alternative models available in your local proxy.
#   # Useful when Amp CLI requests models you don't have access to (e.g., Claude Opus 4.5)
#   # but you have a similar model available (e.g., Claude Sonnet 4).
#   model-mappings:
#     - from: "claude-opus-4-5-20251101"          # Model requested by Amp CLI
#       to: "gemini-claude-opus-4-5-thinking"     # Route to this available model instead
#     - from: "claude-sonnet-4-5-20250929"
#       to: "gemini-claude-sonnet-4-5-thinking"
#     - from: "claude-haiku-4-5-20251001"
#       to: "gemini-2.5-flash"

# Global OAuth model name aliases (per channel)
# These aliases rename model IDs for both model listing and request routing.
# Supported channels: gemini-cli, vertex, aistudio, antigravity, claude, codex, kimi.
# NOTE: Aliases do not apply to gemini-api-key, codex-api-key, claude-api-key, openai-compatibility, vertex-api-key, or ampcode.
# NOTE: Because aliases affect the merged /v1 model list and merged request routing, overlapping
# client-visible names can become ambiguous across providers. /api/provider/{provider}/... helps
# you select the protocol surface, but inference backend selection can still follow the resolved
# model/alias. For strict backend pinning, use unique aliases/prefixes or avoid overlapping names.
# You can repeat the same name with different aliases to expose multiple client model names.
# oauth-model-alias:
#   gemini-cli:
#     - name: "gemini-2.5-pro"          # original model name under this channel
#       alias: "g2.5p"                  # client-visible alias
#       fork: true                      # when true, keep original and also add the alias as an extra model (default: false)
#   vertex:
#     - name: "gemini-2.5-pro"
#       alias: "g2.5p"
#   aistudio:
#     - name: "gemini-2.5-pro"
#       alias: "g2.5p"
#   antigravity:
#     - name: "gemini-3-pro-high"
#       alias: "gemini-3-pro-preview"
#   claude:
#     - name: "claude-sonnet-4-5-20250929"
#       alias: "cs4.5"
#   codex:
#     - name: "gpt-5"
#       alias: "g5"
#   kimi:
#     - name: "kimi-k2.5"
#       alias: "k2.5"

# OAuth provider excluded models
# oauth-excluded-models:
#   gemini-cli:
#     - "gemini-2.5-pro"     # exclude specific models (exact match)
#     - "gemini-2.5-*"       # wildcard matching prefix (e.g. gemini-2.5-flash, gemini-2.5-pro)
#     - "*-preview"          # wildcard matching suffix (e.g. gemini-3-pro-preview)
#     - "*flash*"            # wildcard matching substring (e.g. gemini-2.5-flash-lite)
#   vertex:
#     - "gemini-3-pro-preview"
#   aistudio:
#     - "gemini-3-pro-preview"
#   antigravity:
#     - "gemini-3-pro-preview"
#   claude:
#     - "claude-3-5-haiku-20241022"
#   codex:
#     - "gpt-5-codex-mini"
#   kimi:
#     - "kimi-k2-thinking"

# Optional payload configuration
# payload:
#   default: # Default rules only set parameters when they are missing in the payload.
#     - models:
#         - name: "gemini-2.5-pro" # Supports wildcards (e.g., "gemini-*")
#           protocol: "gemini" # restricts the rule to a specific protocol, options: openai, gemini, claude, codex, antigravity
#       params: # JSON path (gjson/sjson syntax) -> value
#         "generationConfig.thinkingConfig.thinkingBudget": 32768
#   default-raw: # Default raw rules set parameters using raw JSON when missing (must be valid JSON).
#     - models:
#         - name: "gemini-2.5-pro" # Supports wildcards (e.g., "gemini-*")
#           protocol: "gemini" # restricts the rule to a specific protocol, options: openai, gemini, claude, codex, antigravity
#       params: # JSON path (gjson/sjson syntax) -> raw JSON value (strings are used as-is, must be valid JSON)
#         "generationConfig.responseJsonSchema": "{\"type\":\"object\",\"properties\":{\"answer\":{\"type\":\"string\"}}}"
#   override: # Override rules always set parameters, overwriting any existing values.
#     - models:
#         - name: "gpt-*" # Supports wildcards (e.g., "gpt-*")
#           protocol: "codex" # restricts the rule to a specific protocol, options: openai, gemini, claude, codex, antigravity
#       params: # JSON path (gjson/sjson syntax) -> value
#         "reasoning.effort": "high"
#   override-raw: # Override raw rules always set parameters using raw JSON (must be valid JSON).
#     - models:
#         - name: "gpt-*" # Supports wildcards (e.g., "gpt-*")
#           protocol: "codex" # restricts the rule to a specific protocol, options: openai, gemini, claude, codex, antigravity
#       params: # JSON path (gjson/sjson syntax) -> raw JSON value (strings are used as-is, must be valid JSON)
#         "response_format": "{\"type\":\"json_schema\",\"json_schema\":{\"name\":\"answer\",\"schema\":{\"type\":\"object\"}}}"
#   filter: # Filter rules remove specified parameters from the payload.
#     - models:
#         - name: "gemini-2.5-pro" # Supports wildcards (e.g., "gemini-*")
#           protocol: "gemini" # restricts the rule to a specific protocol, options: openai, gemini, claude, codex, antigravity
#       params: # JSON paths (gjson/sjson syntax) to remove from the payload
#         - "generationConfig.thinkingConfig.thinkingBudget"
#         - "generationConfig.responseJsonSchema"
```

<p align="center">
  <img src="./_resources/e744529f4526f27778ee5651cdf56d21.png" alt="配置文件示例" width="600"/>
</p>

> ⚠️ **安全提示：**
> - 将 `allow-remote: false` 修改为 `true`
> - `secret-key: ""` 填入强密码（例如：`123456`）
> - **服务器部署时密码一定要设置复杂！！！**

<p align="center">
  <img src="./_resources/08c6fc7da01aaac00fa97b0883eb25ad.png" alt="配置 allow-remote 和 secret-key" width="600"/>
</p>

编辑完成后，按 `Ctrl + O` 保存，回车确认文件名，再按 `Ctrl + X` 退出编辑器。

### 4. 安装 Docker

运行以下命令安装 Docker：

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

<p align="center">
  <img src="./_resources/55120241b7dc413b00c5c83149e27cfa.png" alt="安装 Docker" width="600"/>
</p>

### 5. 下载并部署项目

拉取 Docker 镜像并运行：

```bash
# 拉取最新镜像
sudo docker pull eceasy/cli-proxy-api:latest

# 运行容器
sudo docker run -d \
  --name cli-proxy-api \
  -p 8317:8317 \
  -v /home/ubuntu/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml \
  -v /home/ubuntu/CLIProxyAPI/auths:/root/.cli-proxy-api \
  --restart unless-stopped \
  eceasy/cli-proxy-api:latest
```

<p align="center">
  <img src="./_resources/e0910fedec8812a910394a12351503d3.png" alt="部署项目" width="600"/>
</p>

### 6. 访问管理面板

浏览器访问管理页面（**记得放通防火墙 8317 端口**）：

```
http://你的服务器IP:8317/management.html
```

- 默认密码：之前设置的 `secret-key`（如：`123456`）

<p align="center">
  <img src="./_resources/5e83c80573822213690834858469ec33.png" alt="管理面板登录" width="600"/>
</p>

### 7. OAuth 登录（以 XAI 为例）

<p align="center">
  <img src="./_resources/06ff66f10747b7142ddc50bf966d6b6b.png" alt="XAI OAuth 登录" width="600"/>
</p>

登录 XAI 账号后，将回调 URL 复制到管理面板：

<p align="center">
  <img src="./_resources/50f889a37970d63ee2996356773b7eea.png" alt="复制回调 URL" width="600"/>
</p>

点击「提交回调 URL」：

<p align="center">
  <img src="./_resources/1d9ca119d2cfc26f375928138540ba27.png" alt="提交回调 URL" width="600"/>
</p>

认证文件成功生成后：

<p align="center">
  <img src="./_resources/4a1034cf54ade2c7ab7fe994c133d51e.png" alt="认证文件" width="600"/>
</p>

### 8. 查看可用模型

在「信息中心」查看可调用模型：

<p align="center">
  <img src="./_resources/96b91997c80d12f6af8064fbc0492602.png" alt="信息中心" width="600"/>
</p>

- **API 接口：** `http://你的服务器IP:8317/v1`
- **模型：** `grok-4.3`

---

## 配置说明

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `host` | 服务器监听地址 | `""` (监听所有接口) |
| `port` | 服务端口 | `8317` |
| `secret-key` | 管理面板密码 | `123456` |
| `api-keys` | API 密钥列表 | `["your-api-key-1"]` |
| `request-retry` | 请求重试次数 | `3` |

---

## 更新项目

执行以下命令更新项目，**密钥不会丢失**，只需重新登录 OAuth：

```bash
sudo docker pull eceasy/cli-proxy-api:latest
sudo docker stop cli-proxy-api
sudo docker rm cli-proxy-api
sudo docker run -d \
  --name cli-proxy-api \
  -p 8317:8317 \
  -v /home/ubuntu/CLIProxyAPI/config.yaml:/CLIProxyAPI/config.yaml \
  -v /home/ubuntu/CLIProxyAPI/auths:/root/.cli-proxy-api \
  --restart unless-stopped \
  eceasy/cli-proxy-api:latest
```

<p align="center">
  <img src="./_resources/7c98e1d190c64c2082281dc6e898f197.png" alt="更新项目" width="600"/>
</p>

---

## 常见问题

### Q: 无法访问管理页面？
**A:** 请检查服务器防火墙是否放通 8317 端口。

### Q: OAuth 认证失败？
**A:** 请确保 `allow-remote` 已设置为 `true`，并且 `secret-key` 已正确配置。

### Q: 如何修改配置？
**A:** 编辑 `config.yaml` 文件后，重启 Docker 容器：

```bash
sudo docker restart cli-proxy-api
```

---

## 许可证

本项目基于开源协议，仅供学习交流使用。
