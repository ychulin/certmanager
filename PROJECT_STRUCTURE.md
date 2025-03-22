# 证书管理平台项目结构

## 目录结构说明

```
certmanager/
├── backend/                    # 后端项目目录
│   ├── app/                   # 应用主目录
│   │   ├── api/              # API 路由
│   │   │   ├── v1/          # API v1 版本
│   │   │   └── deps.py      # 依赖注入
│   │   ├── core/            # 核心配置
│   │   │   ├── config.py    # 配置管理
│   │   │   └── security.py  # 安全相关
│   │   ├── db/              # 数据库
│   │   │   ├── base.py      # 数据库基类
│   │   │   └── session.py   # 会话管理
│   │   ├── models/          # 数据模型
│   │   │   ├── user.py      # 用户模型
│   │   │   └── certificate.py # 证书模型
│   │   ├── schemas/         # Pydantic 模型
│   │   ├── services/        # 业务逻辑
│   │   │   ├── acme.py      # 证书服务
│   │   │   └── notify.py    # 通知服务
│   │   └── utils/           # 工具函数
│   ├── tests/               # 测试目录
│   ├── requirements.txt     # Python 依赖
│   └── Dockerfile          # 后端 Docker 配置
│
├── frontend/                  # 前端项目目录
│   ├── src/                 # 源代码
│   │   ├── assets/         # 静态资源
│   │   ├── components/     # Vue 组件
│   │   │   ├── common/    # 通用组件
│   │   │   └── layout/    # 布局组件
│   │   ├── views/         # 页面视图
│   │   │   ├── admin/    # 管理员页面
│   │   │   └── user/     # 用户页面
│   │   ├── store/         # Pinia 状态管理
│   │   ├── utils/         # 工具函数
│   │   └── api/           # API 请求
│   ├── public/            # 公共资源
│   ├── package.json       # npm 配置
│   ├── vite.config.ts     # Vite 配置
│   └── Dockerfile        # 前端 Docker 配置
│
├── docker/                    # Docker 相关配置
│   ├── nginx/               # Nginx 配置
│   └── redis/              # Redis 配置
│
├── scripts/                   # 部署脚本
│   ├── backup.sh           # 备份脚本
│   └── deploy.sh          # 部署脚本
│
├── .env.example              # 环境变量示例
├── docker-compose.yml        # Docker Compose 配置
├── README.md                 # 项目说明
└── .gitignore               # Git 忽略配置
```

## 目录用途说明

### 后端 (backend/)

- **app/api/**: FastAPI 路由和接口定义
  - **v1/**: API 版本 1 的所有接口
  - **deps.py**: 依赖注入，如认证、数据库会话等

- **app/core/**: 核心配置和功能
  - **config.py**: 应用配置管理
  - **security.py**: JWT、加密等安全相关功能

- **app/db/**: 数据库相关
  - **base.py**: SQLAlchemy 基类
  - **session.py**: 数据库会话管理

- **app/models/**: SQLAlchemy 模型
  - **user.py**: 用户相关模型
  - **certificate.py**: 证书相关模型

- **app/schemas/**: Pydantic 模型，用于数据验证和序列化

- **app/services/**: 业务逻辑实现
  - **acme.py**: acme.sh 相关操作
  - **notify.py**: 通知服务实现

- **app/utils/**: 通用工具函数

### 前端 (frontend/)

- **src/components/**: Vue 组件
  - **common/**: 按钮、表单等通用组件
  - **layout/**: 页面布局组件

- **src/views/**: 页面视图
  - **admin/**: 管理员相关页面
  - **user/**: 用户相关页面

- **src/store/**: Pinia 状态管理
  - 用户状态
  - 证书状态
  - 系统配置等

- **src/utils/**: 工具函数
  - API 请求封装
  - 日期格式化
  - 验证函数等

- **src/api/**: API 接口定义和请求函数

### Docker 配置 (docker/)

- **nginx/**: Nginx 配置文件
- **redis/**: Redis 配置文件

### 脚本 (scripts/)

- **backup.sh**: 数据库和证书备份脚本
- **deploy.sh**: 项目部署脚本

## 开发规范

1. 后端开发规范：
   - 使用 Python 3.9+
   - 遵循 PEP 8 编码规范
   - 使用 FastAPI 依赖注入系统
   - 所有 API 需要版本控制
   - 必须编写单元测试

2. 前端开发规范：
   - 使用 Vue 3 组合式 API
   - 使用 TypeScript
   - 使用 Element Plus UI 框架
   - 遵循 Vue 官方风格指南
   - 组件命名采用 PascalCase

3. 文档规范：
   - 所有公共函数必须有文档注释
   - API 接口必须有 OpenAPI 文档
   - 组件必须有使用说明

4. Git 规范：
   - 使用 feature 分支开发新功能
   - commit 信息需要清晰描述改动
   - 重要功能需要 code review 