# 网络验证系统 API 文档

## 概述

网络验证系统提供用户注册、卡密验证、卡密续费等功能，用于控制软件或服务的访问权限。

### 基础信息

- 基础URL: `https://api.yourdomain.com/v1`
- 数据格式: JSON
- 认证方式: API Key (请求头中添加 `X-API-Key: your_api_key`)

## 错误码说明

| 错误码 | 描述 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权访问 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |
| 1001 | 卡密无效 |
| 1002 | 卡密已过期 |
| 1003 | 卡密已被使用 |
| 1004 | 用户已存在 |
| 1005 | 用户不存在 |

## API 接口详情

### 1. 用户注册

注册新用户账号

- **URL**: `/auth/register`
- **方法**: `POST`
- **请求体**:
  ```json
  {
    "username": "string",      // 用户名，唯一
    "password": "string",      // 密码，建议加密传输
    "email": "string",         // 邮箱，用于找回密码
    "hardware_id": "string"    // 硬件标识，可选
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "message": "注册成功",
    "data": {
      "user_id": "string",     // 用户ID
      "username": "string",    // 用户名
      "expire_time": null,     // 有效期，未激活卡密时为null
      "created_at": "timestamp" // 创建时间
    }
  }
  ```
- **失败响应** (400):
  ```json
  {
    "success": false,
    "message": "用户名已存在",
    "code": 1004
  }
  ```

### 2. 用户登录

用户账号登录

- **URL**: `/auth/login`
- **方法**: `POST`
- **请求体**:
  ```json
  {
    "username": "string",
    "password": "string",
    "hardware_id": "string"    // 硬件标识，可选
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "message": "登录成功",
    "data": {
      "user_id": "string",
      "username": "string",
      "token": "string",       // 访问令牌
      "expire_time": "timestamp", // 账号有效期
      "hardware_id": "string"  // 绑定的硬件标识
    }
  }
  ```

### 3. 验证卡密

使用卡密激活或续费账号

- **URL**: `/card/verify`
- **方法**: `POST`
- **请求头**: `Authorization: Bearer {token}`
- **请求体**:
  ```json
  {
    "card_key": "string"       // 卡密
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "message": "卡密验证成功",
    "data": {
      "card_type": "string",   // 卡密类型
      "duration": 30,          // 有效期天数
      "previous_expire": "timestamp", // 原有效期
      "new_expire": "timestamp",     // 新有效期
      "activated_at": "timestamp"    // 激活时间
    }
  }
  ```
- **失败响应** (400):
  ```json
  {
    "success": false,
    "message": "卡密无效",
    "code": 1001
  }
  ```

### 4. 卡密续费查询

查询卡密可续费的时长信息

- **URL**: `/card/info`
- **方法**: `POST`
- **请求体**:
  ```json
  {
    "card_key": "string"       // 卡密
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "data": {
      "card_type": "string",   // 卡密类型
      "duration": 30,          // 有效期天数
      "is_valid": true,        // 卡密是否有效
      "expire_date": "timestamp" // 卡密自身有效期
    }
  }
  ```

### 5. 生成卡密

管理员接口，生成新卡密

- **URL**: `/admin/card/generate`
- **方法**: `POST`
- **请求头**: `Authorization: Bearer {admin_token}`
- **请求体**:
  ```json
  {
    "card_type": "string",     // 卡密类型
    "duration": 30,            // 有效期天数
    "quantity": 10,            // 生成数量
    "expire_date": "timestamp" // 卡密自身有效期，可选
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "message": "卡密生成成功",
    "data": {
      "cards": [
        {
          "card_key": "string",
          "card_type": "string",
          "duration": 30,
          "expire_date": "timestamp"
        }
      ],
      "total": 10
    }
  }
  ```

### 6. 查询用户信息

查询当前登录用户信息

- **URL**: `/user/info`
- **方法**: `GET`
- **请求头**: `Authorization: Bearer {token}`
- **成功响应** (200):
  ```json
  {
    "success": true,
    "data": {
      "user_id": "string",
      "username": "string",
      "email": "string",
      "expire_time": "timestamp",
      "hardware_id": "string",
      "created_at": "timestamp",
      "last_login": "timestamp"
    }
  }
  ```

### 7. 硬件绑定

绑定用户到特定硬件

- **URL**: `/user/bind-hardware`
- **方法**: `POST`
- **请求头**: `Authorization: Bearer {token}`
- **请求体**:
  ```json
  {
    "hardware_id": "string"
  }
  ```
- **成功响应** (200):
  ```json
  {
    "success": true,
    "message": "硬件绑定成功"
  }
  ```

## 示例流程

1. 用户注册: `POST /auth/register`
2. 用户登录: `POST /auth/login` 获取token
3. 验证卡密(激活/续费): `POST /card/verify` 使用获取的token
4. 查询用户信息确认有效期: `GET /user/info`

## 注意事项

1. 所有敏感操作需要验证用户身份
2. 密码传输建议使用HTTPS加密
3. 重要操作建议添加IP限制或二次验证
4. 卡密生成和管理接口仅限管理员使用
5. 建议定期更换API密钥和访问令牌
