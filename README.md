# 用户登录登出系统

一个简单的用户登录登出系统，使用Flask和SQLite数据库。

## 功能特性

- 用户注册和登录
- 用户登出
- 登录历史记录
- 用户会话管理
- 美观的Bootstrap界面

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
- user_id: 用户ID（外键）
- login_time: 登录时间
- logout_time: 登出时间
- ip_address: IP地址

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

- Flask: Web框架
- SQLAlchemy: ORM数据库操作
- SQLite: 数据库
- Bootstrap: 前端UI框架
- Werkzeug: 密码哈希处理 