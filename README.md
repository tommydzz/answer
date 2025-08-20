## 后端结构重构说明

本次重构仅针对后端代码，目标是提升可维护性、可扩展性与清晰度，同时保持对现有模板与接口端点的完全兼容（`url_for('login')`、`url_for('register')` 等无需修改）。

### 重构前问题

- 所有配置、模型、路由均集中在单一文件 `app.py` 中，职责耦合高。
- 扩展（SQLAlchemy）与应用紧耦合，不便于测试与复用。

### 重构后结构

```
questions-001/
  app.py                # 应用工厂与入口（create_app + 运行）
  extensions.py         # 第三方扩展延迟初始化（db）
  models.py             # 数据模型（User, LoginLog）
  routes.py             # 路由注册（register_routes(app)）
  requirements.txt
  templates/
    base.html
    dashboard.html
    login.html
    profile.html
    register.html
```

### 关键改动

- 应用工厂

  - 新增 `create_app()`：集中初始化配置、扩展与路由。
  - 入口仍为 `app.py`，直接运行即可启动。

- 扩展初始化解耦

  - 新增 `extensions.py`，使用延迟绑定：
    - `db = SQLAlchemy()` 在 `create_app()` 中执行 `db.init_app(app)`。

- 模型拆分

  - 新增 `models.py`，迁移原先的 `User` 与 `LoginLog` 模型；字段与关系保持不变。

- 路由拆分
  - 新增 `routes.py`，以 `register_routes(app)` 的方式集中注册：`/`、`/login`、`/logout`、`/register`、`/profile`。
  - 端点名称保持不变：`index`、`login`、`logout`、`register`、`profile`。
  - 模板中的 `url_for('login')`、`url_for('register')`、`url_for('index')` 等无需修改。

### 与原逻辑保持一致的点

- 用户注册、登录、登出、个人资料页面逻辑未变。
- 登录成功后更新 `last_login`，会话内记录 `login_log_id`，登出时回写 `logout_time`。
- 首次启动时会自动 `db.create_all()` 生成 `sqlite:///users.db`。

### 运行方式

1. 安装依赖

```bash
pip install -r requirements.txt
```

2. 启动服务（默认端口 5001）

```bash
python app.py
```

访问：`http://127.0.0.1:5001`

### 配置说明（可选）

- 当前示例将配置硬编码在 `create_app()` 中：
  - `SECRET_KEY = 'your-secret-key-here'`
  - `SQLALCHEMY_DATABASE_URI = 'sqlite:///users.db'`
- 生产环境建议改为读取环境变量或独立配置文件。

### 常见问题

- 端点 404：请确认 `app.py` 中已调用 `register_routes(app)`。
- 数据库未创建：首次启动自动创建；也可手动删除 `users.db` 重新生成。
- 模板引用报错：检查模板中 `url_for` 是否仍为 `login`、`register`、`index`、`logout`、`profile`。

### 变更摘要

- 单一文件应用 → 工厂模式 + 模块化拆分。
- 扩展延迟初始化，提升测试与扩展能力。
- 路由与模型解耦，目录职责更清晰。

# 用户登录登出系统

一个简单的用户登录登出系统，使用 Flask 和 SQLite 数据库。

## 功能特性

- 用户注册和登录
- 用户登出
- 登录历史记录
- 用户会话管理
- 美观的 Bootstrap 界面

## 数据库结构

### 用户表 (User)

- id: 主键
- username: 用户名（唯一）
- email: 邮箱（唯一）
- password_hash: 密码哈希
- created_at: 创建时间
- last_login: 最后登录时间

### 登录记录表 (LoginLog)

- id: 主键
- user_id: 用户 ID（外键）
- login_time: 登录时间
- logout_time: 登出时间
- ip_address: IP 地址

## 安装和运行

1. 安装依赖：

```bash
pip install -r requirements.txt
```

2. 运行应用：

```bash
python app.py
```

3. 访问应用：
   打开浏览器访问 http://localhost:5001

## 使用说明

1. 首先注册一个新用户
2. 使用注册的用户名和密码登录
3. 在仪表板查看用户信息
4. 在个人资料页面查看登录历史
5. 点击登出按钮退出登录

## 技术栈

- Flask: Web 框架
- SQLAlchemy: ORM 数据库操作
- SQLite: 数据库
- Bootstrap: 前端 UI 框架
- Werkzeug: 密码哈希处理
