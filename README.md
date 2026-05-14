# ur-skills — 联犀 SaaS + IoT 平台 AI Skills

联犀（UnitedRhino）SaaS + IoT 平台的 AI Skill 集合，供 AI Agent 调用平台 API。

## 安装

### 安装 CLI

```bash
# Linux x86_64
VERSION="v0.3.3"
curl -L "https://github.com/unitedrhino/cli/releases/download/${VERSION}/ur-cli-${VERSION}-Linux-x86_64.tar.gz" | tar -xz -C ~/.local/bin --strip-components=1

# macOS ARM
curl -L "https://github.com/unitedrhino/cli/releases/download/${VERSION}/ur-cli-${VERSION}-macOS-arm64.tar.gz" | tar -xz -C ~/.local/bin --strip-components=1
```

### 认证

```bash
# 获取授权 URL
ur login --no-wait --json

# 用户浏览器完成授权后，执行
ur login --setup-code <CODE> --json
```

## Skill 结构

```
SKILL.md              # 主 Skill（统一入口）
ur-device/            # 设备管理
ur-product/           # 产品管理
ur-project/           # 项目管理
ur-user/              # 用户管理
ur-system/            # 系统管理
ur-tenant/            # 租户管理
ur-ai/                # AI 管理
ur-device-analytics/  # 设备数据分析
ur-device-debug/      # 设备调试
scene-linkage/        # 场景联动
thing-model/          # 物模型
protocol-script/      # 协议脚本
```

## 快速开始

```bash
# 查看当前用户
ur api /api/v1/system/user/self/get-one

# 设备列表（iot 应用上下文）
ur --app iot api /api/v1/things/device/info/get-list --body '{"page":{"page":1,"size":10}}'

# 查看 schema
ur schema /api/v1/things/device/
```

## 仓库

- CLI: https://github.com/unitedrhino/cli
- Skills: https://github.com/unitedrhino/skills
