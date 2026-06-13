# TechZone 电子产品商城

## 项目结构

```
electronics-store/
├── public/                 # 前端静态文件（宝塔网站根目录）
│   ├── index.html          # 前台用户端
│   └── admin.html          # 后台管理端
├── api/                    # Node.js API 后端
│   ├── server.js           # 主服务文件
│   ├── package.json        # 依赖配置
│   └── data/
│       └── db.json         # 数据库文件
└── README.md
```

## 宝塔面板部署指南

### 第一步：创建网站

1. 宝塔面板 → 网站 → 添加站点
2. 域名填写你的域名（如 `shop.example.com`）
3. 根目录选择（如 `/www/wwwroot/shop`）
4. PHP版本选"纯静态"

### 第二步：上传前端文件

将 `public/` 目录下的文件上传到网站根目录：
- `/www/wwwroot/shop/index.html`
- `/www/wwwroot/shop/admin.html`

### 第三步：部署 Node.js API

1. 宝塔面板 → 软件商店 → 搜索安装 **Node.js版本管理器**
2. 安装 Node.js（推荐 v18+）
3. 上传 `api/` 目录到服务器，如 `/www/wwwroot/api`
4. 在服务器终端执行：

```bash
cd /www/wwwroot/api
npm install
# 创建管理员账号
node -e "
const fs=require('fs'),bcrypt=require('bcryptjs'),{v4}=require('uuid');
const db=JSON.parse(fs.readFileSync('data/db.json'));
db.users[0].password=bcrypt.hashSync('admin123',10);
db.users[0].createdAt=Date.now();
fs.writeFileSync('data/db.json',JSON.stringify(db,null,2));
console.log('管理员初始化完成');
"
# 启动服务
node server.js
```

### 第四步：使用 PM2 守护进程

```bash
pm2 start server.js --name techzone-api
pm2 save
pm2 startup
```

### 第五步：配置宝塔反向代理

1. 宝塔面板 → 网站 → 你的站点 → 设置 → 反向代理
2. 添加反向代理：
   - 代理名称：`api`
   - 目标URL：`http://127.0.0.1:3000`
   - 发送域名：`$host`

### 第六步：配置伪静态（可选）

在网站设置 → 伪静态中添加：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

## 默认管理员账号

- 账号：`admin`
- 密码：`admin123`

登录后台：`https://你的域名/admin.html`

## API 接口文档

### 认证接口

| 方法 | 路径 | 说明 | 鉴权 |
|------|------|------|------|
| POST | /api/auth/register | 用户注册 | 无 |
| POST | /api/auth/login | 用户登录 | 无 |
| GET | /api/auth/profile | 获取个人信息 | 需要 |
| PUT | /api/auth/profile | 更新个人信息 | 需要 |

### 商品接口

| 方法 | 路径 | 说明 | 鉴权 |
|------|------|------|------|
| GET | /api/products | 获取商品列表 | 无 |
| GET | /api/products/:id | 获取商品详情 | 无 |
| POST | /api/products | 添加商品 | 管理员 |
| PUT | /api/products/:id | 更新商品 | 管理员 |
| DELETE | /api/products/:id | 删除商品 | 管理员 |

### 订单接口

| 方法 | 路径 | 说明 | 鉴权 |
|------|------|------|------|
| GET | /api/orders | 获取订单列表 | 需要 |
| POST | /api/orders | 创建订单 | 需要 |
| PUT | /api/orders/:id/status | 更新订单状态 | 需要 |

### 管理接口

| 方法 | 路径 | 说明 | 鉴权 |
|------|------|------|------|
| GET | /api/admin/stats | 获取统计数据 | 管理员 |
| GET | /api/admin/users | 获取用户列表 | 管理员 |
| GET | /api/admin/orders | 获取所有订单 | 管理员 |
| POST | /api/admin/init | 初始化管理员 | 无 |

### 请求示例

```bash
# 登录
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"phone":"admin","password":"admin123"}'

# 获取商品
curl http://localhost:3000/api/products

# 创建订单（需要token）
curl -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"items":[{"id":"p1","qty":1}]}'
```

## 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| PORT | 3000 | API 端口 |
| JWT_SECRET | techzone_secret_key_2026 | JWT 密钥（建议修改） |
