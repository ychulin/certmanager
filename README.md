

---

## 产品需求文档（PRD）：轻量化证书自动管理工具

### 1. 产品概述

#### 1.1 背景与目标
- **背景**：用户需要管理大量域名的 SSL/TLS 证书，手动管理效率低下。基于 `acme.sh` 开发一个轻量化工具，实现证书的自动更新和发放。
- **目标**：提供带前端界面的证书管理工具，支持多种登录方式（用户名密码、短信验证码、微信扫码），区分管理员和用户权限，验证域名可控性（DNS TXT 记录和 Web 根目录文件），自动定期更新证书并发布到指定服务器，提供实时状态更新和多方式通知。
- **设计原则**：
  - 轻量化：减少外部依赖，使用简单技术栈。
  - 模块化：功能清晰，易维护和扩展。
  - 易用性：通过前端界面简化操作，支持移动端适配。

#### 1.2 核心功能
- **前端界面**：Web 界面，支持多种登录方式，适配 PC 和移动端，管理员和用户按权限编辑配置。
- **权限管理**：区分管理员和用户权限，管理员配置全局设置，用户管理自己的域名和服务器。
- **登录方式**：支持用户名密码、短信验证码、微信扫码登录。
- **域名验证**：通过 `acme.sh` 验证域名，支持 DNS TXT 记录和 Web 根目录文件两种方式。
- **证书续期**：自动定期更新证书，提供实时状态更新。
- **证书发布**：自动将证书发布到用户指定服务器。
- **存储管理**：存储证书和元数据，支持导出。
- **日志记录**：记录操作日志，用户仅查看自己的日志。
- **通知机制**：支持多种通知方式（邮件、短信、Webhook），续期失败时每天通知，连续 7 天。

#### 1.3 使用场景

- **场景 1**：用户通过用户名密码、短信验证码或微信扫码在 PC 或移动端登录。
- **场景 2**：管理员配置全局续期计划和支持的 DNS 提供商。
- **场景 3**：用户添加域名，选择验证方式（DNS TXT 或 Web 根目录文件），配置信息，系统验证后生成证书并发布。
- **场景 4**：系统定期检查证书有效期，自动续期并发布，实时更新状态，失败时通知用户。

---

### 2. 功能需求

#### 2.1 权限管理模块

- **功能描述**：区分管理员和用户权限，控制配置和操作范围。
- **权限划分**：
  - **管理员**：
    - 配置全局设置：支持的 DNS 提供商、续期计划、通知方式。
    - 管理用户：添加/删除用户，分配角色（管理员/普通用户）。
  - **用户**：
    - 配置自己的域名、验证方式（DNS TXT 或 Web 根目录）、DNS 提供商信息、Web 服务器信息、目标服务器信息。
    - 查看自己的证书状态和日志。
    - 下载自己的证书文件。
- **权限细化**：当前版本用户权限统一，预留未来扩展接口（如域名只读权限）。

#### 2.2 登录模块
- **功能描述**：支持用户名密码、短信验证码、微信扫码登录，适配移动端。
- **需求细节**：
  - **登录方式 1：用户名密码**：
    - 用户输入用户名和密码完成滑块验证码后点击登录。
    - 后端验证密码（bcrypt 加密存储），返回 JWT token。
  - **登录方式 2：短信验证码**：
    - 用户输入手机号，完成滑块验证码后点击“发送验证码”。
    - 后端通过阿里云短信服务发送 6 位验证码（有效期 5 分钟）。
    - 用户输入验证码，后端验证，返回 JWT token。
    - 验证码存储：Redis（键：手机号，值：验证码，TTL 5 分钟）。
    - 发送频率限制：每分钟 1 次，每小时 5 次，每天 10 次。
  - **登录方式 3：微信扫码登录**：
    - 前端展示微信二维码（微信开放平台 API 生成）。
    - 用户扫码授权，后端获取 openid。
    - 已绑定用户直接登录，未绑定则提示绑定或注册，返回 JWT token。
    - 移动端优化：若安装微信客户端，直接跳转扫码。
- **前端界面**：
  - 登录页面：选项卡切换（用户名密码、短信验证码、微信扫码）。
    - 用户名密码：输入框 +“登录”按钮。
    - 短信验证码：手机号输入 +“发送验证码”+ 验证码输入 +“登录”。
    - 微信扫码：展示二维码，扫码后自动登录。
  - 移动端适配：选项卡垂直排列，增大输入框和按钮点击区域（最小 44px × 44px）。
- **安全措施**：
  - 登录失败 5 次后锁定 10 分钟。
  - 短信验证码需通过滑块验证码验证。
  - 微信扫码登录使用 HTTPS 回调。

#### 2.3 配置管理模块（前端界面）
- **功能描述**：通过 Web 界面实现配置操作，支持 PC 和移动端。
- **前端框架**：Vue.js（Vue 3 + Vite）+ Element Plus。
- **前端界面设计**：
  - **登录页面**（已在上文描述）。
  - **管理员页面**：
    - **全局配置**：
      - DNS 提供商管理：添加/删除支持的提供商（Cloudflare、阿里云等）。
      - 续期计划：设置检查频率（默认 0-6 点随机）、提前续期天数（默认 7 天）、分批处理窗口（默认每小时 100 个域名）。
      - 通知设置：配置邮件（SMTP）、短信（阿里云）、Webhook（URL + 密钥）。
    - **用户管理**：
      - 用户列表：展示用户名、手机号、微信绑定状态、角色、创建时间。
      - 添加/删除用户：设置角色（管理员/普通用户）。
  - **用户页面**：
    - **域名管理**：
      - 域名列表：展示域名、验证方式、DNS 提供商/Web 服务器、状态。
      - 添加域名（向导式）：
        - 输入域名。
        - 选择验证方式：DNS TXT 或 Web 根目录。
        - DNS TXT：选择提供商，输入 API 密钥，提供工具提示。
        - Web 根目录：输入地址、端口、路径、SSH 信息，提供帮助链接。
      - 删除域名。
    - **服务器管理**：
      - 服务器列表：展示地址、登录方式、存储目录。
      - 添加服务器：输入地址（IP/域名）、SSH 登录（用户名+密码/私钥）、存储目录。
      - 删除服务器。
    - **证书状态**：
      - 证书列表：展示域名、有效期、状态、最后续期时间、发布状态。
      - 下载证书：ZIP 格式（`domain.com.cer`、`domain.com.key`、`domain.com.chain.cer`）。
      - 实时更新：WebSocket 推送状态。
    - **移动端适配**：列表转为卡片式，支持左右滑动切换。
  - **日志查看**：
    - 日志列表：展示时间、域名、操作类型、结果。
    - 过滤：按域名或时间范围。
    - 用户限制：仅查看自己的日志。
    - 移动端适配：支持滑动切换页面。
- **多语言支持**：仅支持中文。

#### 2.4 域名验证模块
- **功能描述**：通过 `acme.sh` 验证域名，支持 DNS TXT 和 Web 根目录两种方式。
- **需求细节**：
  - **DNS TXT 记录**：
    - 用户选择提供商，输入 API 密钥。
    - 后端调用 `acme.sh --issue --dns <provider>`。
  - **Web 根目录文件**：
    - 用户输入地址、端口、路径、SSH 信息。
    - 后端通过 SSH 创建 `.well-known/acme-challenge/`，调用 `acme.sh --issue --webroot`。
    - 验证后删除临时文件。
  - **错误处理**：
    - 失败时重试 3 次（间隔 5 分钟），记录详细错误（“DNS 未生效”、“SSH 不可达”等）。
    - 多次失败后通知用户（弹窗 + 日志）。
  - **成功后**：生成证书，存储元数据，触发发布。

#### 2.5 证书续期模块
- **功能描述**：定期检查证书有效期，自动续期，实时更新状态。
- **需求细节**：
  - 检查频率：管理员配置（默认 0-6 点随机），支持分批处理。
  - 续期条件：有效期 < 7 天（可配置）。
  - 根据验证方式续期。
  - 成功后：更新存储，触发发布，推送状态（WebSocket）。
  - **错误处理**：
    - 失败时重试 3 次（指数退避：1 分钟、5 分钟、15 分钟）。
    - 多次失败后通知用户。
  - **实时更新**：WebSocket 推送状态（`pending`、`renewing`、`success`、`failed`）。

#### 2.6 证书发布模块
- **功能描述**：将证书发布到用户指定服务器。
- **需求细节**：
  - 用户配置服务器信息（地址、SSH 登录、存储目录）。
  - 证书生成/续期后，后端通过 SSH 上传文件，设置权限 600。
  - 发布状态通过 WebSocket 推送（`publishing`、`success`、`failed`）。
  - **错误处理**：
    - 失败时重试 3 次（指数退避：1 分钟、5 分钟、15 分钟），前置检查服务器连通性。
    - 多次失败后通知用户。
  - **手动重试**：前端提供“重试”按钮。

#### 2.7 证书存储模块
- **功能描述**：存储证书和元数据，支持导出。
- **需求细节**：
  - 证书文件：存储于 `~/.acme.sh/`。
  - 元数据：SQLite 存储。
  - 数据库表结构：
    ```sql
    CREATE TABLE certificates (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      user_id INTEGER,
      domain TEXT NOT NULL,
      validation_type TEXT NOT NULL,
      dns_provider TEXT,
      web_server_address TEXT,
      web_server_port INTEGER,
      web_root_path TEXT,
      web_ssh_username TEXT,
      web_ssh_auth_type TEXT,
      web_ssh_password TEXT,
      web_ssh_private_key TEXT,
      cert_path TEXT NOT NULL,
      expiry_date TEXT NOT NULL,
      status TEXT NOT NULL,
      last_renewed TEXT,
      publish_status TEXT DEFAULT 'pending',
      last_published TEXT
    );
    CREATE TABLE users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      username TEXT NOT NULL,
      password TEXT NOT NULL,
      phone_number TEXT,
      wechat_openid TEXT,
      role TEXT NOT NULL,
      notification_preferences TEXT -- JSON 格式存储通知偏好
    );
    CREATE TABLE servers (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      user_id INTEGER,
      server_address TEXT NOT NULL,
      ssh_username TEXT NOT NULL,
      ssh_auth_type TEXT NOT NULL,
      ssh_password TEXT,
      ssh_private_key TEXT,
      cert_directory TEXT NOT NULL
    );
    ```
  - **证书导出**：
    - 下载 ZIP 文件，包含 `domain.com.cer`、`domain.com.key`、`domain.com.chain.cer`。
    - 支持选择导出格式（ZIP 或 PEM）。

#### 2.8 日志记录模块
- **功能描述**：记录操作日志，用户仅查看自己的日志。
- **需求细节**：
  - 记录内容：时间、用户、域名、操作类型（验证、续期、发布）、结果、错误信息。
  - 存储：SQLite。
  - 轮转：按 10MB 分割，最多保留 10 个文件，超出删除最早文件，支持压缩（gzip）。
  - 过滤：用户通过 `user_id` 查看自己的日志。

#### 2.9 通知机制模块
- **功能描述**：续期或发布失败时通知用户，支持多种方式。
- **需求细节**：
  - **通知方式**：
    - 前端弹窗（默认，WebSocket）。
    - 邮件：管理员配置 SMTP。
    - 短信：阿里云服务。
    - Webhook：管理员配置 URL + 密钥。
  - **用户偏好**：用户选择启用的通知方式，存储于 `users.notification_preferences`。
  - **通知逻辑**：
    - 第一次失败开始，每天通知一次，连续 7 天。
    - 问题解决后停止，新失败重新计数。
    - 同一问题每天最多通知一次。

---

### 3. 非功能需求

#### 3.1 性能
- 支持 1000 个域名，分批处理（默认每批 100 个）。
- 前端响应时间 < 2 秒，移动端首屏加载 < 3 秒。

#### 3.2 安全性
- 密码使用 bcrypt 加密。
- DNS API 密钥、SSH 密码/私钥使用 AES 加密。
- 验证码存储于 Redis（TTL 5 分钟）。
- 微信扫码登录使用 HTTPS。
- 证书文件权限 600。
- 前端-后端通信使用 HTTPS，WebSocket 使用 WSS。

#### 3.3 可维护性
- 模块化设计，前端后端解耦。
- 提供详细日志和错误信息。

#### 3.4 兼容性
- 后端：Linux（Ubuntu）。
- 前端：Chrome、Firefox、Edge；移动端支持 Chrome（Android）、Safari（iOS）、UC 浏览器。

#### 3.5 移动端适配
- 响应式布局：适配 320px - 1920px。
- 小屏幕优化：导航折叠为汉堡菜单，表格转为卡片式。
- 交互优化：增大点击区域（44px × 44px），支持手势滑动。

---

### 4. 技术实现方案

#### 4.1 技术选型
- **后端**：
  - 框架：FastAPI（Python）。
  - 数据库：SQLite（轻量化，预留 MySQL 扩展）。
  - 缓存：Redis（验证码存储）。
  - 核心工具：`acme.sh`。
  - 定时任务：Linux `cron`。
  - 通知：smtplib（邮件）、阿里云 SMS SDK（短信）、HTTP POST（Webhook）。
  - WebSocket：FastAPI + `websockets`。
  - SSH：`paramiko`。
  - 微信登录：`weixin-python`。
- **前端**：
  - 框架：Vue.js（Vue 3 + Vite）+ Element Plus + `vueuse`（移动端支持）。
  - CSS：CSS Media Queries 或 Tailwind CSS。
  - WebSocket：`WebSocket` API。
- **日志**：SQLite 存储。

#### 4.2 模块划分
- **前端模块**：
  - 登录模块：支持三种登录方式。
  - 管理员模块：全局配置、用户管理。
  - 用户模块：域名管理、服务器管理、证书状态、下载。
  - 日志模块：展示和过滤。
  - WebSocket 模块：实时状态更新。
- **后端模块**：
  - API 模块：RESTful API。
  - 登录模块：处理登录逻辑。
  - 配置模块：管理配置。
  - 验证模块：调用 `acme.sh`。
  - 续期模块：定时续期。
  - 发布模块：SSH 发布。
  - 存储模块：证书和元数据管理。
  - 日志模块：记录操作。
  - 通知模块：发送通知。
  - WebSocket 模块：推送状态。

#### 4.3 数据交互
- **前端-后端交互**：
  - **RESTful API**：
    - `POST /api/login/password`：用户名密码登录。
    - `POST /api/login/sms/send`：发送短信验证码。
    - `POST /api/login/sms/verify`：验证短信登录。
    - `GET /api/login/wechat/qrcode`：获取微信二维码。
    - `POST /api/login/wechat/bind`：绑定微信。
    - `GET /api/config`：获取全局配置（管理员）。
    - `POST /api/config`：更新全局配置（管理员）。
    - `POST /api/domains`：添加域名。
    - `DELETE /api/domains/{id}`：删除域名。
    - `POST /api/servers`：添加服务器。
    - `DELETE /api/servers/{id}`：删除服务器。
    - `GET /api/certificates`：获取证书列表。
    - `GET /api/certificates/{id}/download`：下载证书。
    - `GET /api/logs`：获取日志。
  - **WebSocket**：
    - 地址：`wss://<host>/ws`。
    - 消息格式：`{"domain": "example.com", "status": "renewed", "publish_status": "success", "expiry_date": "2025-06-21"}`。
- **后端-`acme.sh`**：通过 `subprocess` 调用。
- **后端-服务器**：通过 `paramiko` SSH 访问。
- **后端-微信**：微信开放平台 API。

#### 4.4 流程设计
1. **登录**：
   - 用户选择登录方式，成功后存储 JWT token。
2. **初始化**：
   - 管理员配置全局设置。
   - 用户添加域名和服务器，后端验证并发布证书。
3. **续期**：
   - 定时任务检查证书，续期并发布，推送状态。
4. **通知**：
   - 失败时记录日志，推送弹窗，每天通知，连续 7 天。

