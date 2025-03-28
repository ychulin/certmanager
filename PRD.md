# 自助式 SSL 证书托管更新平台 - 产品需求文档（PRD）

## 目录

1. [产品概述](#1-产品概述)
   - [1.1 背景与目标](#11-背景与目标)
   - [1.2 核心功能](#12-核心功能)
   - [1.3 使用场景](#13-使用场景)
2. [功能需求](#2-功能需求)
   - [2.1 用户与权限管理](#21-用户与权限管理)
   - [2.2 登录模块](#22-登录模块)
   - [2.3 域名管理与验证](#23-域名管理与验证)
   - [2.4 服务器管理](#24-服务器管理)
   - [2.5 证书申请与续期](#25-证书申请与续期)
   - [2.6 证书发布](#26-证书发布)
   - [2.7 证书存储与导出](#27-证书存储与导出)
   - [2.8 日志记录](#28-日志记录)
   - [2.9 通知机制](#29-通知机制)
3. [界面设计](#3-界面设计)
   - [3.1 登录页面](#31-登录页面)
   - [3.2 用户主界面](#32-用户主界面)
   - [3.3 管理员界面](#33-管理员界面)
4. [流程设计](#4-流程设计)
   - [4.1 用户注册与登录流程](#41-用户注册与登录流程)
   - [4.2 域名添加与验证流程](#42-域名添加与验证流程)
   - [4.3 证书续期与发布流程](#43-证书续期与发布流程)
5. [非功能性需求](#5-非功能性需求)
   - [5.1 性能](#51-性能)
   - [5.2 安全性](#52-安全性)
   - [5.3 可维护性](#53-可维护性)
   - [5.4 兼容性](#54-兼容性)
   - [5.5 移动端适配](#55-移动端适配)
6. [技术实现方案](#6-技术实现方案)
   - [6.1 技术选型](#61-技术选型)
   - [6.2 容器化部署](#62-容器化部署)
   - [6.3 数据交互](#63-数据交互)
7. [附录](#7-附录)
   - [7.1 术语解释](#71-术语解释)

---

## 1. 产品概述

### 1.1 背景与目标
- **背景**: 在中国大陆，随着 HTTPS 的普及，企业与个人用户需要管理大量域名的 SSL/TLS 证书。手动操作效率低下且易出错。本产品基于 `acme.sh`，提供一个轻量化的自助式证书托管更新平台，用户配置域名和服务器信息后，平台自动完成证书管理全流程。
- **目标**: 
  - 提供直观 Web 界面，支持 PC 和移动端，适配中国大陆用户习惯。
  - 实现证书自动申请、续期和发布，减少 80% 管理时间，提升续期成功率至 99.9%。
  - 支持中国大陆主流服务（阿里云 SMS、微信登录等），确保合规性和稳定性。
  - 通过容器化部署降低门槛，适用于中小型企业和个人用户。
- **设计原则**: 
  - **轻量化**: 核心依赖仅 `acme.sh` 和 SQLite。
  - **易用性**: 简洁界面，自助操作，符合国内习惯。
  - **稳定性**: 高可用，支持高并发。

### 1.2 核心功能
| 模块       | 功能描述                                      |
| ---------- | --------------------------------------------- |
| 用户管理   | 用户注册、登录、权限分配                      |
| 域名管理   | 添加域名、选择验证方式、验证所有权            |
| 服务器管理 | 配置目标服务器，用于证书发布                  |
| 证书续期   | 自动检查并续期，提供实时状态                  |
| 证书发布   | 自动发布证书至目标服务器                      |
| 存储与导出 | 存储证书和元数据，支持导出                    |
| 日志记录   | 记录操作日志，用户查看自身记录                |
| 通知机制   | 邮件和短信通知续期/发布结果，短信由管理员控制 |

### 1.3 使用场景
- **场景 1**: 用户通过邮箱或手机验证码注册，添加域名并配置验证方式，平台自动生成并发布证书。
- **场景 2**: 用户用微信扫码登录，配置目标服务器，平台定期续期证书，失败时根据管理员设置发送邮件或短信通知。
- **场景 3**: 管理员调整全局续期计划或为用户启用短信通知，确保平台稳定。
- **场景 4**: 用户在移动端查看证书状态，下载证书文件。

---

## 2. 功能需求

### 2.1 用户与权限管理
- **功能描述**: 区分管理员和用户权限，确保数据隔离和系统安全。
- **权限划分**:
  - **管理员**: 
    - 配置全局设置（DNS 服务商模板、续期计划、通知服务，包括邮箱和短信服务商）。
    - 管理用户（添加、删除、分配角色、设置短信通知开关）。
  - **用户**: 
    - 配置自身域名、DNS 服务商（从管理员提供的模板中选择）、目标服务器。
    - 查看自身证书状态和日志。
    - 下载证书文件。
    - 设置邮件通知偏好（短信通知由管理员控制）。
- **实现细节**: 
  - 用户可管理自己添加的域名和服务器配置信息，也可将对应配置移交给其他用户，需管理员审核、接收用户确认后生效。
  - 用户数据存储在数据库表 `users` 中，新增字段 `sms_notification_enabled` 表示是否启用短信通知。

### 2.2 登录模块
- **功能描述**: 支持用户名密码、短信验证码和微信扫码三种登录方式，提供安全身份验证。
- **需求细节**:
  - **注册**: 
    - 方式1：输入用户名、密码和邮箱地址，从邮箱接收验证码后验证生效。
    - 方式2：输入用户名、密码和手机号，若管理员为该用户启用短信通知，则获取 6 位短信验证码（有效期 5 分钟），验证后生效。
    - 后端通过管理员配置的 SMTP 服务器发送邮件验证码，通过管理员配置的短信服务商发送 SMS 验证码。
  - **登录**: 
    - **用户名 + 密码**: 
      - 输入用户名和密码，完成滑块验证码后登录。
      - 后端验证（bcrypt 加密），返回 JWT token。
    - **短信验证码**: 
      - 输入手机号，完成滑块验证码后，若管理员为该用户启用短信通知，则获取 6 位验证码（有效期 5 分钟）；否则提示“此账户未启用短信服务”。
      - 后端通过管理员配置的短信服务商发送，验证后返回 JWT token。
      - 频率限制：每分钟 1 次，每小时 5 次，每天 10 次。
    - **微信扫码**: 
      - 前端展示微信二维码（通过微信开放平台 API 生成）。
      - 用户扫码授权，后端获取 `openid`，若已绑定手机号，直接返回 JWT token；否则提示“请先绑定手机号”。
  - **密码找回**: 
    - 支持邮箱密码找回（管理员配置邮件服务商）、短信密码找回（若管理员启用短信通知）、通知管理员重置密码三种方式。
  - **安全措施**: 
    - 密码使用 bcrypt 加密存储。
    - 连续失败 5 次锁定账号 10 分钟（按用户名或手机号计数）。
    - HTTPS 通信，微信回调使用 HTTPS。

### 2.3 域名管理与验证
- **功能描述**: 用户自助添加域名并验证所有权。
- **需求细节**:
  - **添加域名**: 
    - 输入域名（如 `example.com`），选择验证方式。
    - 支持批量导入（CSV 格式）。
  - **验证方式**:
    - **DNS TXT**: 
      - 用户从管理员配置的 DNS 服务商模板中选择（如阿里云、腾讯云），输入自己的 API 密钥。
      - 后端调用 `acme.sh --issue --dns <provider>`。
    - **Web 根目录**: 
      - 输入服务器地址、端口、路径、SSH 信息（用户名 + 密码/私钥）。
      - 后端创建 `.well-known/acme-challenge/`，调用 `acme.sh --issue --webroot`。
  - **错误处理**: 
    - 失败时重试 3 次，DNS TXT 验证间隔为 30 分钟，Web 根目录验证间隔为 10 分钟，记录错误日志。
    - 提供帮助链接（如“如何获取阿里云 API 密钥”）。

### 2.4 服务器管理
- **功能描述**: 用户配置目标服务器，用于证书发布。
- **需求细节**:
  - 选择是公共集群还是自有服务器，公共集群默认由管理员配置证书发布服务器信息，用户不可自助操作。
  - **用户自助操作添加自有服务器用以发布证书**: 
    - 输入服务器地址（IP/域名）、SSH 用户名、认证方式（密码/私钥）、证书存储目录。
    - 支持“测试连接”按钮验证连通性。
  - **管理**: 
    - 查看服务器列表，支持编辑和删除。

### 2.5 证书申请与续期
- **功能描述**: 自动申请和续期证书，提供实时状态。
- **需求细节**:
  - **申请**: 
    - 域名验证成功后，自动生成证书，首次生成后将有效期保存至数据库 `certificates` 表 `expiry_date` 字段。
  - **续期**: 
    - 检查频率：每周一次，随机在周中任意一天的 0-6 点执行。
    - 触发条件：有效期 < 7 天（用户可调整）。
    - 并发控制：每批 5 个域名，5 线程并行。
    - 每次续期检查成功后，更新数据库 `expiry_date`。
  - **状态更新**: 
    - 通过邮件和 Webhook 推送（`pending`、`renewing`、`success`、`failed`）。
  - **错误处理**: 
    - 重试 3 次（间隔 1、5、15 分钟），失败后通知用户。

### 2.6 证书发布
- **功能描述**: 自动发布证书至目标服务器。
- **需求细节**:
  - **发布过程**: 
    - **公共集群**: 用户仅提供网站域名，目标服务器信息由管理员配置。
    - **自有服务器**: 用户配置服务器地址、SSH 信息、证书存储目录，需具备足够权限。
    - 通过 SSH（SFTP）上传证书文件（`.cer`、`.key`、`.chain.cer`）。
    - 上传后设置文件权限为 600。
    - 验证证书是否生效（测试访问 HTTPS）。
  - **状态更新**: 
    - WebSocket 推送（`publishing`、`success`、`failed`）。
  - **错误处理**: 
    - 检查连通性（超时 5 秒），失败重试 3 次（1、5、15 分钟）。
    - 提供手动重试按钮。

### 2.7 证书存储与导出
- **功能描述**: 存储证书文件和元数据，支持导出。
- **需求细节**:
  - **存储**: 
    - 证书文件：容器内 `/app/data/acme.sh/`（通过卷挂载持久化）。
    - 元数据：SQLite 表 `certificates`。
  - **导出**: 
    - 支持 ZIP（默认，包含 `.cer`、`.key`、`.chain.cer`）和 PEM 格式。
  - **表结构**:
    ```sql
    CREATE TABLE certificates (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      user_id INTEGER,
      domain TEXT NOT NULL,
      validation_type TEXT NOT NULL,
      dns_provider TEXT,
      web_server_address TEXT,
      cert_path TEXT NOT NULL,
      expiry_date TEXT NOT NULL,
      status TEXT NOT NULL,
      last_renewed TEXT,
      publish_status TEXT DEFAULT 'pending'
    );
    ```

### 2.8 日志记录
- **功能描述**: 记录操作日志，用户查看自身记录。
- **需求细节**:
  - **内容**: 时间、用户 ID、域名、操作类型、结果、错误信息。
  - **存储**: SQLite，10MB 轮转，保留 10 个文件，旧文件压缩（gzip）。
  - **查看**: 用户仅见自身日志，支持按域名和时间过滤。

### 2.9 通知机制
- **功能描述**: 续期或发布失败时通知用户，短信通知由管理员控制。
- **需求细节**:
  - **方式**: 
    - **邮件**: 管理员配置全局邮件服务商（如阿里云邮件推送），用户可选择是否接收。
    - **短信**: 管理员配置全局短信服务商（如腾讯云 SMS），并为每个用户设置是否启用短信通知。
    - **Webhook**: 如钉钉机器人。
  - **逻辑**: 
    - 检查用户表 `sms_notification_enabled`，若启用则发送短信通知，否则仅发送邮件（若用户启用邮件通知）。
    - 同一问题每天通知一次，最多 7 天。
  - **配置**: 
    - 管理员在“外部服务商管理”中设置邮件和短信服务商参数。
    - 管理员在“用户管理”中设置用户短信通知开关。
    - 用户在“通知设置”中配置邮件偏好。

---

## 3. 界面设计

### 3.1 登录页面
- **PC 视图**:
  ```
  +-------------------------------------+
  | [用户名+密码] [短信验证码] [微信扫码]|
  |                                     |
  | 用户名: [_________]                 |
  | 密码:   [_________]                 |
  | [滑块验证码]                        |
  |            [登录]                   |
  +-------------------------------------+
  ```
- **移动端视图**:
  ```
  +-----------------+
  | 用户名+密码     |
  | 用户名: [____]  |
  | 密码:   [____]  |
  | [滑块验证码]    |
  | [登录]          |
  |-----------------|
  | 短信验证码      |
  | 手机号: [____]  |
  | [发送验证码]    |
  | 验证码: [____]  |
  | [登录]          |
  |-----------------|
  | 微信扫码        |
  | [二维码]        |
  | 请用微信扫码    |
  +-----------------+
  ```

### 3.2 用户主界面
- **域名管理**:
  ```
  +-------------------------------------+
  | 域名管理                            |
  | [添加域名]                          |
  | +---------+-------+-------+-------+ |
  | | 域名    | 验证方式 | 状态  | 操作  | |
  | +---------+-------+-------+-------+ |
  | | ex.com  | DNS TXT | 有效  | [删除] | |
  +-------------------------------------+
  ```
- **服务器管理**:
  ```
  +-------------------------------------+
  | 服务器管理                          |
  | [添加服务器]                        |
  | +---------+-------+-------+-------+ |
  | | 地址    | 目录    | 状态  | 操作  | |
  | +---------+-------+-------+-------+ |
  | | 1.2.3.4 | /ssl    | 正常  | [删除] | |
  +-------------------------------------+
  ```
- **证书状态**:
  ```
  +-------------------------------------+
  | 证书状态                            |
  | +---------+-------+-------+-------+ |
  | | 域名    | 有效期    | 状态  | 操作  | |
  | +---------+-------+-------+-------+ |
  | | ex.com  | 2025-06-21| 有效  | [下载] | |
  +-------------------------------------+
  ```
- **通知设置**:
  ```
  +-------------------------------------+
  | 通知设置                            |
  | - 邮件通知: [开启 / 关闭]           |
  | - 短信通知: [由管理员控制]          |
  +-------------------------------------+
  ```
- **服务商配置**:
  ```
  +-------------------------------------+
  | 服务商配置                          |
  | DNS 服务商                          |
  | 服务商: [下拉菜单: 阿里云 / Cloudflare] |
  | API Key: [__________]               |
  | API Secret: [__________]            |
  | [保存]                              |
  +-------------------------------------+
  ```

### 3.3 管理员界面
- **全局配置**:
  ```
  +-------------------------------------+
  | 全局配置                            |
  | DNS 服务商模板                      |
  | [阿里云] [删除]                     |
  | [添加: ___] [确认]                  |
  | 续期计划                            |
  | 检查时间: [周中随机0-6点]           |
  | 提前天数: [7]                       |
  +-------------------------------------+
  ```
- **外部服务商管理**:
  ```
  +-------------------------------------+
  | 外部服务商管理                      |
  | 邮件服务商                          |
  | 服务商: [阿里云]                    |
  | SMTP: [smtp.mxhichina.com]          |
  | 端口: [587]                         |
  | 用户名/密码: [____] [____]          |
  | [保存]                              |
  |-------------------------------------|
  | 短信服务商                          |
  | 服务商: [腾讯云]                    |
  | API Key: [__________]               |
  | API Secret: [__________]            |
  | [保存]                              |
  +-------------------------------------+
  ```
- **用户管理**:
  ```
  +-------------------------------------+
  | 用户管理                            |
  | [添加用户]                          |
  | +---------+-------+-------+-------+-------+
  | | 用户名  | 手机号  | 角色  | 短信通知 | 操作  |
  | +---------+-------+-------+-------+-------+
  | | user1   | 138...  | 用户  | [开启]  | [删除] |
  +-------------------------------------+
  ```

---

## 4. 流程设计

### 4.1 用户注册与登录流程
```
开始
  ↓
选择注册/登录
  ↓
{选择方式}
  ↓
[用户名+密码] → 输入用户名和密码 → 滑块验证码 → 后端验证
  ↓
[短信验证码] → 输入手机号 → 滑块验证码 → 检查 sms_notification_enabled → 是 → 发送验证码 → 输入验证码 → 后端验证
  ↓                                    否 → 提示“未启用短信服务”
[微信扫码] → 扫码授权 → 获取 openid → 已绑定手机号? → 是 → 返回 token
  ↓                                    否 → 返回失败（“请先绑定手机号”）
成功? → 是 → 返回 JWT token → 进入主界面
  ↓
否 → 失败计数 +1 → ≥5次? → 是 → 锁定 10 分钟
  ↓
否 → 返回错误 → 重试
```

### 4.2 域名添加与验证流程
```
开始
  ↓
输入域名
  ↓
选择验证方式 (DNS TXT / Web 根目录)
  ↓
{DNS TXT} → 选择管理员配置的 DNS 服务商 → 输入用户自己的 API 密钥
  ↓
调用 acme.sh 验证
  ↓
成功? → 是 → 生成证书 → 存储 → 发布
  ↓
否 → 重试 (3次，DNS TXT 30分钟间隔，Web 10分钟间隔)
  ↓
仍失败? → 是 → 通知用户 → 记录日志
  ↓
否 → 生成证书 → 存储 → 发布
```

### 4.3 证书续期与发布流程
```
开始 (每周随机一天 0-6点)
  ↓
检查数据库中证书有效期 (expiry_date)
  ↓
< 7 天? → 是 → 调用 acme.sh 续期
  ↓
否 → 结束
  ↓
成功? → 是 → 更新 expiry_date → 发布 → 推送状态
  ↓
否 → 重试 (3次，1、5、15分钟)
  ↓
仍失败? → 是 → 检查 sms_notification_enabled → 是 → 发送短信和邮件 → 记录日志
  ↓                                    否 → 发送邮件（若启用）→ 记录日志
否 → 更新 expiry_date → 发布 → 推送状态
```

---

## 5. 非功能性需求

### 5.1 性能
- 支持管理 1000 个域名，每批续期 5 个域名，5 线程并行。
- 前端响应时间 < 2 秒，移动端首屏 < 3 秒。
- 证书续期平均耗时 < 1 分钟，发布 < 30 秒。

### 5.2 安全性
- 密码 bcrypt 加密，API 密钥和 SSH 凭据 AES 加密。
- 文件权限 600，HTTPS 和 WSS 通信。
- 每周自动备份数据库和证书文件。

### 5.3 可维护性
- 模块化设计，前后端解耦。
- 提供详细日志和错误信息。
- 支持 Docker 部署，配置通过 `.env` 文件。

### 5.4 兼容性
- **后端**: Linux（Ubuntu）。
- **前端**: Chrome、Firefox、Edge；移动端 Chrome（Android）、Safari（iOS）、微信内置浏览器。

### 5.5 移动端适配
- 响应式布局（320px - 1920px）。
- 小屏幕优化：导航折叠为汉堡菜单，表格转为卡片，点击区域 ≥ 44px × 44px。

---

## 6. 技术实现方案

### 6.1 技术选型
- **后端**: 
  - FastAPI（Python）：轻量、高性能。
  - `acme.sh`：证书管理核心。
  - SQLite：默认数据库（支持切换 PostgreSQL）。
  - Celery：任务队列，使用 Redis 作为消息代理。
  - `paramiko`：SSH 操作。
  - `smtplib`：邮件通知。
  - `weixin-python`：微信登录支持。
- **前端**: 
  - Vue.js（Vue 3 + Vite）+ Element Plus。
  - Tailwind CSS：响应式布局。

### 6.2 容器化部署
- **镜像**: 
  - **前端**: Nginx + Vue.js 构建结果。
  - **后端**: FastAPI + Celery + `acme.sh`。
- **配置**: 
  - 通过 `.env` 文件指定变量：
    ```env
    DB_TYPE=sqlite
    DB_PATH=/app/data/certificates.db
    SMTP_HOST=smtp.mxhichina.com
    SMTP_PORT=587
    SMTP_USER=user@example.com
    SMS_ACCESS_KEY_ID=your_key
    WECHAT_APP_ID=your_app_id
    WECHAT_APP_SECRET=your_secret
    APP_PORT=8000
    REDIS_HOST=redis
    REDIS_PORT=6379
    ```
- **Docker Compose 示例**:
  ```yaml
  version: '3.8'
  services:
    backend:
      build: ./backend
      ports:
        - "8000:8000"
      volumes:
        - cert_data:/app/data
      env_file:
        - .env
      depends_on:
        - redis
    frontend:
      build: ./frontend
      ports:
        - "80:80"
    redis:
      image: redis:latest
      ports:
        - "6379:6379"
      volumes:
        - redis_data:/data
  volumes:
    cert_data:
    redis_data:
  ```

### 6.3 数据交互
- **RESTful API**:
  - `POST /api/register`: 注册。
  - `POST /api/login/password`: 密码登录。
  - `POST /api/login/sms`: 短信登录。
  - `GET /api/login/wechat/qrcode`: 获取微信二维码。
  - `POST /api/domains`: 添加域名。
  - `GET /api/certificates`: 获取证书列表。
  - `POST /api/admin/service-templates`: 管理员配置服务商。
  - `POST /api/admin/users/{user_id}/sms-notification`: 管理员设置用户短信通知。
- **WebSocket**: `wss://<host>/ws`，推送状态更新。

---

## 7. 附录

### 7.1 术语解释
- **acme.sh**: 开源工具，用于自动申请和续期 Let’s Encrypt 证书。
- **DNS TXT**: 通过 DNS 记录验证域名所有权。
- **JWT**: JSON Web Token，用于身份验证。
- **AES**: 高级加密标准，用于敏感数据加密。
- **openid**: 微信用户唯一标识。
