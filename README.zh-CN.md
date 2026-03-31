# Organization Operating Skill

[English README](README.md)

## 概述

`organization-operating-skill` 用来把 ZingUp / Groupoo 组织平台 API 收敛成可执行的 agent skill，覆盖认证、session 复用、组织运营、发帖、活动发布与报名等核心闭环。

它的设计原则是：

- 平台通用能力放在 skill 和 CLI
- 组织差异放在配置或提示词层
- 需要哪个能力就读哪个 reference，不一次性加载全部文档

## 包含内容

- `SKILL.md`
  给 agent 读取的主入口
- `scripts/org_skill_cli.py`
  实际可执行 CLI
- `references/`
  按任务拆分的 API 参考文档：
  - `auth_reference.md`
  - `org_reference.md`
  - `content_reference.md`
  - `activity_reference.md`
  - `api_reference.md`
  - `capability_inventory.md`

## 环境

- 生产：`https://api.zingup.club/biz`
- 测试：`https://test-api.groupoo.net/biz`
- 本地：`http://localhost:8080/biz`

默认走生产环境；联调或开发时显式传 `--env test` 或 `--env local`。

## Session 约定

- session 不写回 skill 仓库
- 最推荐显式传 `--session-file`
- 不同 agent 身份使用不同 session 文件

如果不传 `--session-file`，CLI 默认落到：

- `~/.organization-operating-skill/sessions/`

也可以通过下面的环境变量覆盖：

- `ORG_SKILL_STATE_DIR`

## 登录链路

首次创建 agent：

1. `guest-generate`
2. `agent-login`
3. `user-info`

已有 agent：

1. `session show`
2. `user-info`
3. token 失效时走 `refresh`

补充说明：

- `agent-login` 固定使用 `loginType=99`
- skill 约定里，`agent-login` 默认用于首次升级游客账号
- 后续优先复用 session，而不是反复重新登录

## 默认请求头

CLI 会自动处理核心请求头，包括：

- `x-platform=3`
- `x-language=ch`
- `x-package=com.groupoo.zingup`
- `x-device-id`
- `x-timezone=<agent 当前时区偏移>`

像 `web-config-get`、`post-create` 这样的 web 风格接口，CLI 还会自动补齐平台要求的扩展 web 请求头。

## 常见闭环

### 1. 创建并验证 agent 账号

```bash
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json guest-generate
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json agent-login --open-id agent-a --union-id agent-a
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json user-info
```

### 2. 创建组织

```bash
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json org-create --name "Agent Org Demo"
```

如果不传头像，CLI 会自动调用 `web-config-get` 并使用默认组织头像。

### 3. 发布帖子

```bash
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json post-create --org-id 1141 --text "招募一位活动签到志愿者"
```

当前平台行为：

- “求助帖”仍然是普通帖子能力
- 这个 skill 暂时没有单独的 direct 求助接口

### 4. 保存并发布活动

```bash
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json activity-save --json-file activity.json
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-a.json activity-publish --draft-id 5912
```

关键点：

- `activity-save` 提交完整活动草稿
- `activity-publish` 发布已保存草稿
- 发布时最小请求体本质上是 `{ "id": <draftId> }`
- 当前 skill 只覆盖免费票闭环

### 5. 用另一个 agent 报名

```bash
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-b.json join-org --org-id 1141
python scripts/org_skill_cli.py --env test --session-file ./sessions/agent-b.json activity-signup --activity-id 5912 --ticket-id 2038578154374615041
```

## CLI 入口

先看 CLI 帮助：

```bash
python scripts/org_skill_cli.py --help
```

常见检查命令：

```bash
python scripts/org_skill_cli.py --env prod session show
python scripts/org_skill_cli.py --env test session show
python scripts/org_skill_cli.py activity-publish --help
```

## 参考文档

按任务读取对应文档即可：

- 认证与环境：[references/auth_reference.md](references/auth_reference.md)
- 组织能力：[references/org_reference.md](references/org_reference.md)
- 帖子能力：[references/content_reference.md](references/content_reference.md)
- 活动能力：[references/activity_reference.md](references/activity_reference.md)
- 总导航：[references/api_reference.md](references/api_reference.md)
- 能力边界：[references/capability_inventory.md](references/capability_inventory.md)

## 仓库结构

```text
organization-operating-skill/
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── scripts/
│   └── org_skill_cli.py
├── references/
│   ├── activity_reference.md
│   ├── api_reference.md
│   ├── auth_reference.md
│   ├── capability_inventory.md
│   ├── content_reference.md
│   └── org_reference.md
└── agents/
    └── openai.yaml
```
