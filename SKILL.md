---
name: ur-api
description: "联犀 SaaS + IoT 平台 API 统一入口。涵盖设备管理、产品管理、项目管理、用户管理、系统管理、租户管理、AI 管理、设备数据分析、设备调试、场景联动、物模型、协议脚本。triggers: 联犀, UnitedRhino, SaaS, IoT, 设备管理, 产品管理, 用户管理, 项目管理, 系统管理, 租户管理, AI管理, 场景联动, 物模型, 协议脚本, 设备调试, 数据分析, 能耗分析"
metadata:
  hermes:
    tags: [api, saas, iot, device, product, user, project, system, tenant, ai, analytics]
  openclaw:
    requires:
      bins:
        - ur
---

# ur-api — 联犀 SaaS + IoT 平台 API 工具

## 概述

联犀（UnitedRhino）是一个 SaaS + IoT 平台，提供设备管理、产品定义、项目管理、用户权限、租户管理、AI 智能体等能力。

**统一入口**：`ur` CLI（单二进制，通过 `--app` 切换应用上下文）

| 应用 | `--app` 值 | AppID | 默认 TenantCode | 覆盖域 |
|------|-----------|-------|----------------|--------|
| 物联网 | `iot` | 200 | `platform` | 设备、产品、项目、OTA、协议 |
| 平台管理 | `platform-manage` | 100 | `platform` | 租户、用户、应用、授权 |
| 组织管理 | `org-manage` | 300 | 配置文件 | 组织用户、角色、AI 智能体 |
| 能源管理 | `org-energy` | 1000 | 配置文件 | 能耗分析、电力集抄、预付费 |
| 控制台 | `console` | 77 | `platform` | 个人信息、访问令牌 |

所有接口均为 POST 方法，请求格式 `{code, msg, data}`。

## 安装 CLI

```bash
# Linux x86_64
VERSION="v0.3.3"
curl -L "https://github.com/unitedrhino/cli/releases/download/${VERSION}/ur-cli-${VERSION}-Linux-x86_64.tar.gz" | tar -xz -C ~/.local/bin --strip-components=1

# macOS ARM
curl -L "https://github.com/unitedrhino/cli/releases/download/${VERSION}/ur-cli-${VERSION}-macOS-arm64.tar.gz" | tar -xz -C ~/.local/bin --strip-components=1

# Windows (PowerShell)
# Invoke-WebRequest -Uri "https://github.com/unitedrhino/cli/releases/download/${VERSION}/ur-cli-${VERSION}-Windows-x86_64.zip" -OutFile "ur.zip"; Expand-Archive "ur.zip" -DestinationPath "$env:USERPROFILE\.local\bin"
```

## 认证（Device Flow）

**AI Agent 自动执行，无需用户输入密码：**

```bash
# 1. 获取授权 URL
ur login --no-wait --json
```

输出：
```json
{
  "status": "authorization_required",
  "verification_url": "https://saas.unitedrhino.com/#/user/settings?tab=access-tokens&setup=ABC123&redirect=openclaw",
  "setup_code": "ABC123",
  "expires_in": 600,
  "next_command": "ur login --setup-code ABC123"
}
```

AI 向用户发送 `verification_url`，用户浏览器点击「完成 CLI 绑定」后，AI 执行：

```bash
# 2. 完成授权
ur login --setup-code ABC123 --json
```

输出：
```json
{
  "event": "authorization_complete",
  "tenant_code": "t1",
  "access_key": "ak_xxxx",
  "access_secret": "sk_xxxx",
  "user_id": "123"
}
```

## 角色权限

调用 API 前**先确认角色**，错误角色会导致 403：

```bash
ur check
```

| 角色 | 权限范围 | 典型场景 |
|------|---------|----------|
| **平台管理员** | 跨租户操作、租户 CRUD | 创建租户、管理租户配置 |
| **租户管理员** | 本租户内 CRUD、用户管理 | 创建设备、管理用户 |
| **普通用户** | 个人信息、设备分享 | 登录、修改密码 |

## 子 Skill 索引

根据用户意图加载对应子 Skill：

| 用户意图 | 子 Skill | 最低角色 |
|---------|---------|---------|
| 设备列表、设备控制、属性上报、网关、OTA | `ur-device` | admin |
| 产品定义、物模型、协议脚本、OTA 固件 | `ur-product` | admin |
| 项目、区域、分组、数据权限 | `ur-project` | admin |
| 登录、用户信息、角色、部门、字典、通知 | `ur-user` | admin / user |
| 文件上传、应用管理、Hook、授权码 | `ur-system` | admin / platform |
| 创建租户、租户列表、租户配置 | `ur-tenant` | platform |
| AI Agent、告警规则、场景联动 | `ur-ai` | admin |
| 属性历史、趋势分析、能耗统计、报表 | `ur-device-analytics` | admin |
| 设备日志、实时调试、Mock 数据 | `ur-device-debug` | admin |
| 场景联动规则生成与校验 | `scene-linkage` | admin |
| 物模型生成与校验 | `thing-model` | admin |
| 协议脚本模板与校验 | `protocol-script` | admin |

## 高频端点速查

| 操作 | 端点 | 最小请求体 |
|------|------|-----------|
| 当前用户信息 | `POST /api/v1/system/user/self/get-one` | `{}` |
| 设备列表 | `POST /api/v1/things/device/info/get-list` | `{"page":{"page":1,"size":10}}` |
| 控制设备属性 | `POST /api/v1/things/device/interact/property-control-send` | `{"productID":"x","deviceName":"x","data":{"Key":值}}` |
| 设备最新属性 | `POST /api/v1/things/device/msg/property-latest/get-list` | `{"productID":"x","deviceName":"x"}` |
| 产品列表 | `POST /api/v1/things/product/info/get-list` | `{"page":{"page":1,"size":10}}` |
| 项目列表 | `POST /api/v1/things/project/crud/get-list` | `{"page":{"page":1,"size":10}}` |
| 租户列表 | `POST /api/v1/system/tenant/info/get-list` | `{"page":{"page":1,"size":10}}` |
| 告警规则列表 | `POST /api/v1/things/alarm/info/get-list` | `{"page":{"page":1,"size":10}}` |
| 场景列表 | `POST /api/v1/things/scene/info/get-list` | `{"page":{"page":1,"size":10}}` |
| 文件上传 | `POST /api/v1/system/common/upload-file` | multipart |

## CLI 用法速查

```bash
# 切换应用上下文
ur --app iot api /api/v1/things/device/info/get-list --body '{"page":{"page":1,"size":10}}'

# 调试模式
ur api /api/v1/things/device/info/get-list --body '{}' --debug

# 字段筛选
ur api ... --fields code,data.total,data.list

# 查看 schema
ur schema /api/v1/things/device/
```

## 404 回退策略（强制）

**绝对禁止臆造 API 路径。** 若调用返回 404：

1. 调用 `ur schema <已知前缀>` 查看真实接口列表
2. 查阅对应子 Skill 的端点列表
3. **严禁凭直觉尝试相似路径**

## 认证方式

- **Device Auth**（推荐）：`ur login` → 浏览器授权 → 自动保存 AK/SK
- **AccessKey/JWT**：直接配置 AK + SK → HS256 JWT → `Authorization: Bearer`
- **环境变量**：`UR_BASE_URL`、`UR_APP_ID`、`UR_ACCESS_KEY`、`UR_ACCESS_SECRET`

JWT 中 `userID` 必须为**字符串格式**（`"userID": "12345"`），否则 401。

## 参考

- CLI 仓库：https://github.com/unitedrhino/cli
- Skill 仓库：https://github.com/unitedrhino/skills
