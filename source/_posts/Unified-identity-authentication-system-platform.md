---
title: 统一身份认证系统平台功能设计
date: 2020-09-17 15:02:00
tags: 
  - 系统设计
  - 开发笔记
---
# 统一身份认证系统平台功能设计
## 系统设置

### 用户管理

用户属性：账户、密码、盐值、姓名、电话、角色ID、组织机构ID、数字证书（编号、路径）、密码更新时间、密码错误次数、账户有效期、是否启用、备注；
密码连续错误次数 > 5 为锁定状态；正确则置0；

### 角色管理

角色属性：角色ID、角色名称、权限项（集合）

### 权限项管理

权限项属性：权限ID、应用ID、对象名称、权限类型（菜单、按钮、API）、URL、关联权限项（多个）、备注、排序编号。

### 组织机构管理

组织机构属性：组织机构ID、名称、代码、上级ID、级别;

### 应用系统

应用系统属性：应用ID、系统名称、图标、系统令牌、加密算法；

### 安全参数

参数：密码强度、密码有效期、初次登录必须修改密码；
密码强度：低：无限制
中：长度>5; 至少2种字符组合
高：长度>7; 至少3种字符组合
密码有效期：无期限、3个月、6个月、12个月；
密码加密方式：使用SHA256哈希算法加密加密盐值+密码原文，数据库保存加密后的密文。
IP登录范围：未设置时允许所有IP登录。

### 日志管理

记录所有的接口日志 和 IP;
对非授权的接口访问视为攻击行为，并记入安全事件日志。

### 数据字典

数据字典属性：编号、字典分类、字典名称、字典值、是否启用、排序编号。

## 业务逻辑

统一权限系统接口基于OAuth2.0实现验证授权流程。该接口供内部或第三方登录访问授权。

加密算法采用RSA 2048位进行加密解密和签名，由统一权限系统创建公钥和私钥，私钥由统一权限系统持有，公钥由第三方应用持有。采用密码模式登录时，由第三方应用将密码原文用公钥进行加密，系统收到密文后采用对应的私钥进行解码，解密原文后再使用SHA256哈希算法加密加密盐值+密码原文于数据库中的密码进行比对。

登录完成后系统分配Access Token，Access Token采用[JWT](https://jwt.io/) 格式，JWT包含头部、载荷、签名三部分。JWT采用统一权限系统发布的私钥进行签名，第三方应用接收到JWT之后使用公钥进行验证有效性。

JWT载荷JSON格式如下：

| 名称  | 英文全称           | 是否必须 | 含义                                                         |
| ----- | ------------------ | -------- | ------------------------------------------------------------ |
| iss   | Issuer Identifier  | 是       | 统一权限系统URL，用于提供认证信息者的唯一标识。一般是一个https的url（不包含querystring和fragment部分）。 |
| sub   | Subject Identifier | 是       | iss提供的最终用户的标识，即用户名，在iss范围内唯一。它会被RP用来标识唯一的用户。最长为255个ASCII个字符。 |
| name  | Name               | 是       | 用户显示名称。                                               |
| aud   | Audience(s)        | 是       | 应用编号（AppId）                                            |
| exp   | Expiration time    | 是       | 过期时间，超过此时间的Access Token会作废不再被验证通过。     |
| iat   | Issued At Time     | 是       | JWT的构建的时间。                                            |
| nonce | Nonce              | 是       | 发送请求的时候提供的随机字符串，用来减缓重放攻击，也可以来关联Access Token和受信客户端本身的Session信息。 |
| role  | Role               | 否       | 角色名称                                                     |
| isa   | Is Super Admin     | 否       | 是否超级管理员                                               |

JSON内容示例：

```
{
  "sub": "SuperAdmin",
  "name": "超级管理员",
  "iss": "http://192.168.1.100",
  "aud": "TIMS-Client-1",
  "isa": true,
  "nonce": "ABCAV32hkKG",
  "role": "管理员",
  "exp": 1598851053729,
  "iat": 1598246196411
}
```

#### 前后端分离系统权限控制流程

{% asset_img 前后端分离系统权限控制流程.png %}

#### 应用程序登录权限控制流程

{% asset_img 应用程序登录权限控制流程.png %}

## 接口定义

#### 1.获取Authorization Code

**请求地址:** `/oauth2/authorize`

**请求方法:** GET

**请求参数:**

| 名称          | 必须 | 类型   | 备注                                                         |
| :------------ | :--- | :----- | :----------------------------------------------------------- |
| client_id     | 是   | long   | 申请应用时分配的应用ID                                       |
| redirect_uri  | 是   | string | 授权回调地址, 必须和申请应用是填写的一致(参数部分可不一致)   |
| response_type | 是   | string | 描述获取授权的方式， Authorization Code方式授权, **response_type=code** |
| scope         | 否   | string | 申请scope权限所需参数，可一次申请多个scope权限，用空格分隔。 |
| state         | 否   | string | 用于保持请求和回调的状态，授权请求成功后原样带回给第三方，该参数用于防止 CSRF攻击（跨站请求伪造攻击），强烈建议第三方带上该参数 |

**返回值:**

- **成功响应**

如果授权成功，授权服务器会将用户的浏览器重定向到`redirect_uri`，并带上`code`，`state`等参数，例子如下:

```
http://example.com/example?code=CODE&state=STATE
```

**返回参数说明：**

| 名称  | 必须 | 类型   | 备注                                                      |
| :---- | :--- | :----- | :-------------------------------------------------------- |
| code  | 是   | string | 用来换取`access_token`的授权码，有效期为5分钟且只能用一次 |
| state | 否   | string | 如果请求时传递参数，会回传该参数                          |

- **失败响应**

如果授权失败，授权服务器会将用户的浏览器重定向到`redirect_uri`，并带上`error`，`error_description`, `state`等参数，例子如下:

```
http://example.com/example?error=ERROR&error_description=ERROR_DESCRIPTION&state=STATE
```

**返回参数说明：**

| 名称              | 必须 | 类型   | 备注                             |
| :---------------- | :--- | :----- | :------------------------------- |
| error             | 是   | int    | OAuth定义的错误码                |
| error_description | 是   | string | 错误描述信息                     |
| state             | 否   | string | 如果请求时传递参数，会回传该参数 |

#### 2.获取Access Token

**请求地址:** `/oauth2/access_token`

**请求方法:** GET

**请求参数:**

| 名称          | 必须 | 类型   | 备注                                                       |
| :------------ | :--- | :----- | :--------------------------------------------------------- |
| client_id     | 是   | long   | 申请应用时分配的应用ID                                     |
| redirect_uri  | 是   | string | 授权回调地址, 必须和申请应用是填写的一致(参数部分可不一致) |
| client_secret | 是   | string | 申请应用时分配的系统令牌                                   |
| grant_type    | 是   | string | 固定为authorization_code                                   |
| code          | 是   | string | 第1小节中拿到的授权码                                      |

**返回值:**

- **成功响应**

如果请求成功，授权服务器会返回JSON格式的字符串：

1. access_token: 要获取的Access Token
2. token_type: token类别，固定为bearer（RFC6750 定义）
3. expires_in: Access Token的有效期，以秒为单位。
4. refresh_token: 用于刷新Access Token 的 Refresh Token,所有应用都会返回该参数（10年的有效期）
5. scope: Access Token最终的访问范围，关于权限的具体信息参考scope权限列表
6. openId: 用户统一标识，可以唯一标识一个用户.网站或应用可将此ID进行存储，便于用户下次登录时辨识其身份

```
{

  "access_token": "access token value",

  "token_type": "Bearer",

  "expires_in": 360000,

  "refresh_token": "refresh token value",

  "scope": "scope value",

  "openId":"OPENID"

}
```

- **失败响应**

如果请求失败，授权服务器会返回JSON格式的字符串：

1. error：错误码，是一个int类型的数字 请参考OAuth定义的错误码
2. error_description：一段可读的文字，用来帮助理解和解决发生的错误

```
{

  "error": "error_code",

  "error_description": "错误描述"

}
```

#### 3.Refresh Token刷新接口

**请求地址:** `/oauth2/refresh_token`

**请求方法:** GET

**请求参数:**

| 名称          | 必须 | 类型   | 备注                                                       |
| :------------ | :--- | :----- | :--------------------------------------------------------- |
| client_id     | 是   | long   | 申请应用时分配的 **App Id**                                |
| redirect_uri  | 是   | string | 授权回调地址, 必须和申请应用是填写的一致(参数部分可不一致) |
| client_secret | 是   | string | 申请应用时分配的 **App Secret**                            |
| grant_type    | 是   | string | 固定为refresh_token                                        |
| refresh_token | 是   | string | 请求授权成功时获取的刷新令牌                               |

**返回值数据:**

- **成功响应**

如果请求成功，授权服务器会返回JSON格式的字符串:

1. access_token: 要获取的Access Token
2. expires_in: Access Token的有效期，以秒为单位, 请参考Access Token生命周期
3. refresh_token: 用于刷新Access Token 的 Refresh Token,所有应用都会返回该参数（10年的有效期）
4. scope: Access Token最终的访问范围，关于权限的具体信息参考scope权限列表
5. openId: 用户统一标识，可以唯一标识一个用户.网站或应用可将此ID进行存储，便于用户下次登录时辨识其身份

```
{

  "access_token": "access token value",

  "expires_in": 360000,

  "refresh_token": "refresh token value",

  "scope": "scope value",

  "openId":"2.0XXXXXXXXX"

}
```

- **失败响应**

如果请求失败，授权服务器会返回JSON格式的字符串：

1. error：错误码，是一个int类型的数字 请参考OAuth定义的错误码
2. error_description：一段人类可读的文字，用来帮助理解和解决发生的错误

```
{

  "error": "error_code",

  "error_description": "错误描述"

}
```

#### 4.密码模式登录

**请求地址:** `/oauth2/signin`

**请求方法:** GET

**请求参数:**

| 名称          | 必须 | 类型   | 备注                      |
| :------------ | :--- | :----- | :------------------------ |
| client_id     | 是   | long   | 申请应用时分配的应用ID    |
| client_secret | 是   | string | 申请应用时分配的系统令牌  |
| grant_type    | 是   | string | 固定为password            |
| username      | 是   | string | 用户名                    |
| password      | 是   | string | 密码，密码采用RSA公钥加密 |

**返回值:**

- **成功响应**

如果请求成功，授权服务器会返回JSON格式的字符串：

1. access_token: 要获取的Access Token
2. expires_in: Access Token的有效期，以秒为单位。
3. refresh_token: 用于刷新Access Token 的 Refresh Token,所有应用都会返回该参数（10年的有效期）
4. scope: Access Token最终的访问范围，关于权限的具体信息参考scope权限列表
5. openId: 用户统一标识，可以唯一标识一个用户.网站或应用可将此ID进行存储，便于用户下次登录时辨识其身份

```
{

  "access_token": "access token value",

  "expires_in": 360000,

  "refresh_token": "refresh token value",

  "scope": "scope value",

  "openId":"OPENID"

}
```

- **失败响应**

如果请求失败，授权服务器会返回JSON格式的字符串：

1. error：错误码，是一个int类型的数字 请参考OAuth定义的错误码
2. error_description：一段可读的文字，用来帮助理解和解决发生的错误

```
{

  "error": "error_code",

  "error_description": "错误描述"

}
```

#### 5. 获取用户信息

**请求地址:** `/oauth2/account_profile`

**请求方法:** GET

**请求参数:**

| 名称     | 必须 | 类型   | 备注                                   |
| :------- | :--- | :----- | :------------------------------------- |
| clientId | 是   | long   | 申请应用时分配的APP ID                 |
| token    | 是   | string | 用户授权得到的访问令牌（Access Token） |

**返回值:**

- **成功**

```
{

  "result": "ok",

  "description": "成功",

  "data": {

            "name": "用户编号",

            "userName": "用户姓名",
            
            "email": "电子邮箱",

            "role": "角色名称，多个用英文逗号分隔"

           },

  "code": 0

}
```

- **失败**

```
{

   "result": "error",

   "description": "错误描述",

   "code": "错误码"

}
```

#### 6. 获取用户权限

**请求地址:** `/oauth2/get_permission`

**请求方法:** GET

**请求参数:**

| 名称     | 必须 | 类型   | 备注                                   |
| :------- | :--- | :----- | :------------------------------------- |
| clientId | 是   | long   | 申请应用时分配的APP ID                 |
| token    | 是   | string | 用户授权得到的访问令牌（Access Token） |

**返回值:**

- **成功**

```
{

  "result": "ok",

  "description": "成功",

  "data": [{
            "id": "权限编号",

            "name": "用户编号",

            "parentId": "父权限编号",
            
            "url": "权限地址",
            
            "icon": "图标",

            "renderForm": "呈现形式(0:菜单 1:按钮)",

            "openMode": "打开方式(0:内部,1:弹窗)"
           }
          ]

  "code": 0

}
```

- **失败**

```
{

   "result": "error",

   "description": "错误描述",

   "code": "错误码"

}
```

#### 7. 获取数据字典

**请求地址:** `/oauth2/get_dict`

**请求方法:** GET

**请求参数:**

| 名称       | 必须 | 类型   | 备注                                   |
| :--------- | :--- | :----- | :------------------------------------- |
| clientId   | 是   | long   | 申请应用时分配的APP ID                 |
| token      | 是   | string | 用户授权得到的访问令牌（Access Token） |
| categoryId | 是   | string | 字典分类                               |

**返回值:**

- **成功**

```
{

  "result": "ok",

  "description": "成功",

  "data": [{
            "key": "字典Key",

            "value": "字典Value"      
           }
          ]

  "code": 0

}
```

- **失败**

```
{

   "result": "error",

   "description": "错误描述",

   "code": "错误码"

}
```

#### 8. 获取组织机构

**请求地址:** `/oauth2/get_org`

**请求方法:** GET

**请求参数:**

| 名称  | 必须 | 类型   | 备注                                   |
| :---- | :--- | :----- | :------------------------------------- |
| token | 是   | string | 用户授权得到的访问令牌（Access Token） |

**返回值:**

- **成功**

```
{

  "result": "ok",

  "description": "成功",

  "data": [{
            "id": "组织机构编号",

            "name": "组织机构名称",

            "parentId": "组织机构父编号",

            "remark": "备注"
           }
          ]

  "code": 0

}
```

- **失败**

```
{

   "result": "error",

   "description": "错误描述",

   "code": "错误码"

}
```

#### 9. 权限校验

**请求地址:** `/oauth2/verify_permission`

**请求方法:** GET

**请求参数:**

| 名称         | 必须 | 类型   | 备注                                   |
| :----------- | :--- | :----- | :------------------------------------- |
| token        | 是   | string | 用户授权得到的访问令牌（Access Token） |
| permissionId | 是   | string | 权限编号                               |

**返回值:**

- **成功**

```
{

  "result": "ok",

  "description": "成功",
  
  "code": 0

}
```

- **失败**

```
{

   "result": "error",

   "description": "错误描述",

   "code": "错误码"

}
```

## 数据库设计

{% asset_img 数据库设计.png %}