# 系统设计文档 - 在线书店系统

## 1. 系统架构设计

### 1.1 前端架构

- 前端采用单页面应用（SPA）架构，使用React框架进行开发。
- 使用Redux进行状态管理，提供良好的用户体验和页面交互。
- 通过Webpack进行打包和构建，优化前端资源加载性能。

### 1.2 后端架构

- 后端采用微服务架构，拆分为用户服务、图书服务、订单服务等多个独立的服务。
- 使用Spring Boot框架搭建服务，提供RESTful API接口供前端调用。
- 使用Spring Security进行用户认证和权限管理，确保系统安全性。

### 1.3 数据库架构

- 使用MySQL数据库存储用户信息、图书信息、订单信息等数据。
- 使用Redis作为缓存数据库，提高系统读取性能和响应速度。
- 使用数据库连接池进行连接管理，确保数据库连接的高效利用。

## 2. 接口设计

### 2.1 用户服务接口

- 注册接口：POST /api/users/register
- 登录接口：POST /api/users/login
- 用户信息查询接口：GET /api/users/{userId}
- 用户信息更新接口：PUT /api/users/{userId}

### 2.2 图书服务接口

- 图书列表查询接口：GET /api/books
- 图书详情查询接口：GET /api/books/{bookId}
- 图书评论查询接口：GET /api/books/{bookId}/comments
- 图书评论新增接口：POST /api/books/{bookId}/comments

### 2.3 购物车服务接口

- 购物车列表查询接口：GET /api/cart
- 添加图书到购物车接口：POST /api/cart
- 修改购物车中图书数量接口：PUT /api/cart/{cartItemId}
- 删除购物车中图书接口：DELETE /api/cart/{cartItemId}

### 2.4 订单服务接口

- 提交订单接口：POST /api/orders
- 查询订单列表接口：GET /api/orders
- 查询订单详情接口：GET /api/orders/{orderId}
- 订单支付接口：POST /api/orders/{orderId}/pay

## 3. 数据库设计

### 3.1 用户表（users）

- id：用户ID（主键）
- username：用户名
- password：密码
- email：邮箱
- address：地址
- ...

### 3.2 图书表（books）

- id：图书ID（主键）
- title：图书标题
- author：作者
- publisher：出版社
- price：价格
- ...

### 3.3 订单表（orders）

- id：订单ID（主键）
- userId：用户ID（外键）
- status：订单状态（待支付、已支付、已完成等）
- totalAmount：订单总金额
- createTime：订单创建时间
- ...

## 4. 技术选型

- 前端技术：React、Redux、Webpack
- 后端技术：Spring Boot、Spring Security、Spring Data JPA
- 数据库：MySQL、Redis
- API文档工具：Swagger

## 5. 部署架构

- 使用Docker容器化部署，将前端、后端、数据库等组件分别部署为独立的容器。
- 使用Nginx作为反向代理服务器，负责请求转发和负载均衡。

## 6. 安全设计

- 使用HTTPS协议进行数据传输，保证数据的安全性。
- 使用Spring Security进行用户认证和权限管理，防止未授权访问和恶意攻击。
- 使用Token进行身份验证，防止跨站请求伪造（CSRF）攻击。