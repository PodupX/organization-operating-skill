---
name: organization-operating-skill
description: 连接组织平台与外部 agent 的通用 skill。用于接入和调用用户、组织、帖子、活动相关 API，完成登录鉴权、组织运营与活动闭环。当任务涉及让 agent/skill 调用组织平台 API 执行动作时使用。
---

# Organization Operating Skill

## Overview

这个 skill 负责把组织平台 API 收敛成稳定的可执行动作，重点覆盖登录鉴权、session 复用、组织运营、发帖、活动和报名闭环。
优先复用平台通用能力，把组织差异留给组织配置和提示词片段，而不是为单个组织写一套特化流程。

## Workflow

1. 先判断任务属于哪一类：
   认证 / 环境 / token：读 [references/auth_reference.md](references/auth_reference.md)。
   组织能力：读 [references/org_reference.md](references/org_reference.md)。
   帖子能力：读 [references/content_reference.md](references/content_reference.md)。
   活动能力：读 [references/activity_reference.md](references/activity_reference.md)。
   能力优先级与规划：读 [references/capability_inventory.md](references/capability_inventory.md)。
2. 不要默认加载所有 reference。
   先看 `scripts/org_skill_cli.py --help`，只补读当前任务相关的文档。
3. 把“平台通用能力”和“组织专属规则”分开。
   规则差异优先放组织配置层，不为一次性规则新增 skill 能力。

## Common Flows

- 首次创建 agent 账号：
  先 `session show`，再 `guest-generate`，然后 `agent-login`，最后用 `user-info` 验证身份。
- 已有 agent session：
  先 `session show`，再 `user-info` 验证身份。
  token 失效时优先 `refresh`，不再默认重复调用 `agent-login`。
- 组织运营闭环：
  先 `user-info` 看 `isAllowCreate`，再 `org-create` 或 `org-detail`，然后 `post-create`。
  所谓“求助帖”当前就是普通帖子，直接 `post-create` 写求助文案。
  要发活动时必须先 `activity-save` 拿草稿 `id`，再 `activity-publish --draft-id <id>`。
- 多 agent 并行：
  不同账号必须使用不同 `--session-file`，避免同环境 token 被后一个账号覆盖。
- 环境约定：
  默认生产；联调显式传 `--env test`；本地后端显式传 `--env local`。

## Core Capabilities

- 认证与环境切换
- 组织创建、查询、修改、加入、成员分页
- 帖子发布
- 活动草稿、发布、详情、搜索、报名
- 后续能力优先级见 [references/capability_inventory.md](references/capability_inventory.md)

## Integration Rules

- 保持 skill 通用，把组织差异放进组织配置和 agent 提示词片段。
- 优先通过已有 API 完成闭环，不为一次性组织规则新增代码。
- 任何接口接入前都先补齐认证方式、必填参数、返回字段、错误码和仓库定位方式。
- 任何高风险动作都保留人工兜底，包括封禁、纠纷裁定、信用惩罚和线下安全处理。
- 当平台 API 不足以表达组织规则时，先记录缺口，再决定是否新增接口。

## Resources

- [references/capability_inventory.md](references/capability_inventory.md)
  看能力清单和优先级，不看具体请求体。
- [references/api_reference.md](references/api_reference.md)
  总索引文件，只负责导航。
- [references/auth_reference.md](references/auth_reference.md)
  认证、请求头、token、环境与 session 规则。
- [references/org_reference.md](references/org_reference.md)
  组织相关 API、默认组织头像、创建与修改限制。
- [references/content_reference.md](references/content_reference.md)
  发帖接口与帖子分享链接。
- [references/activity_reference.md](references/activity_reference.md)
  活动草稿、发布、搜索、报名与分享链接。
- `scripts/`
  当前只放一个可执行 CLI：`scripts/org_skill_cli.py`
  需要命令清单时，优先运行：`python scripts/org_skill_cli.py --help`

## Runtime Defaults

- 默认接口基座是生产环境：`https://api.zingup.club/biz`
- session 不写入 skill 仓库。
- 最稳妥的做法是由调用方显式传 `--session-file`
- 如果没有显式传：
  - CLI 会在用户目录下默认使用 `~/.organization-operating-skill/sessions/` 并自动创建
  - 当前脚本实现可通过 `ORG_SKILL_STATE_DIR` 覆盖
  - 旧的 `.codex-state/...` session 仍兼容读取，用于平滑迁移
- 环境切换优先用 `--env`
  - `prod`
  - `test`
  - `local`
- 也可以显式传 `--base-url`
- 默认请求头会带：
  - `x-platform=3`
  - `x-language=ch`
  - `x-package=com.groupoo.zingup`
  - `x-timezone=<agent 当前时区偏移>`
- 英文用户可显式改成 `--language us`
- 具体 header 细节优先以 `auth_reference.md` 和 CLI 实际行为为准
- web 风格请求：
  - `web-config-get` 和 `post-create` 会自动补齐所需的 web 扩展请求头
- `x-device-id` 的策略：
  - 首次没有时自动生成
  - 一旦写入 session，后续命令默认复用
- 通用调试入口：
  - 先看 `python scripts/org_skill_cli.py --help`
  - `python scripts/org_skill_cli.py --env prod session show`
  - `python scripts/org_skill_cli.py --env test session show`
- 登录约定：
  - 首次建 agent：`guest-generate -> agent-login`
  - 后续同一 agent：优先复用 session，token 失效时走 `refresh`
- 组织查询约定：
  - 成员视角默认使用 `org-detail`
- `org-create` 不传头像时，会自动调用 `web-config-get` 选默认组织头像
- 活动发布约定：
  - `activity-save` 负责提交完整活动请求体
  - `activity-publish` 默认只需要草稿 `id`
- 多账号约定：
  - 同一环境下如需同时维护发布者和报名者，请显式区分 `--session-file`

## Context Discipline

只在需要时加载对应 reference，避免把整套 API 契约一次性塞进上下文。
