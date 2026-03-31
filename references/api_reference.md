# API Reference Index

这份索引文件只负责导航，不再承载全部细节。
读取 skill 时，优先按任务类型进入对应 reference，避免一次加载所有 API 契约。

校验日期：2026-03-30

## 环境入口

- 生产环境默认：`https://api.zingup.club/biz`
- 测试环境：`https://test-api.groupoo.net/biz`
- 本地开发：`http://localhost:8080/biz`
- skill 若未显式传 `--base-url` 或 `--env`，默认走生产环境

## 阅读路径

- 认证、token、请求头、环境切换：
  读 [auth_reference.md](auth_reference.md)
- 组织列表、详情、创建、修改、成员、加入组织：
  读 [org_reference.md](org_reference.md)
- 发帖、帖子分享链接、帖子读取限制：
  读 [content_reference.md](content_reference.md)
- 活动草稿、发布、搜索、报名、活动分享链接：
  读 [activity_reference.md](activity_reference.md)
- 能力优先级和后续待补清单：
  读 [capability_inventory.md](capability_inventory.md)

## 能力映射总览

| 能力 ID | 参考文档 |
| --- | --- |
| `auth.guest.generate` | [auth_reference.md](auth_reference.md) |
| `auth.agent.third_login` | [auth_reference.md](auth_reference.md) |
| `auth.refresh` | [auth_reference.md](auth_reference.md) |
| `user.profile.get` | [auth_reference.md](auth_reference.md) |
| `user.profile.update` | [auth_reference.md](auth_reference.md) |
| `web.config.get` | [org_reference.md](org_reference.md) |
| `org.list` | [org_reference.md](org_reference.md) |
| `org.detail` | [org_reference.md](org_reference.md) |
| `org.detail.manage` | [org_reference.md](org_reference.md) |
| `org.create` | [org_reference.md](org_reference.md) |
| `org.update` | [org_reference.md](org_reference.md) |
| `org.member.list` | [org_reference.md](org_reference.md) |
| `org.member.page` | [org_reference.md](org_reference.md) |
| `org.join` | [org_reference.md](org_reference.md) |
| `content.post.create` | [content_reference.md](content_reference.md) |
| `activity.save` | [activity_reference.md](activity_reference.md) |
| `activity.publish` | [activity_reference.md](activity_reference.md) |
| `activity.cancel` | [activity_reference.md](activity_reference.md) |
| `activity.delete` | [activity_reference.md](activity_reference.md) |
| `activity.detail` | [activity_reference.md](activity_reference.md) |
| `activity.search` | [activity_reference.md](activity_reference.md) |
| `activity.org.list` | [activity_reference.md](activity_reference.md) |
| `activity.user.sign.list` | [activity_reference.md](activity_reference.md) |
| `activity.sign.list` | [activity_reference.md](activity_reference.md) |
| `activity.signup` | [activity_reference.md](activity_reference.md) |

## 约定

- 不要默认整篇通读全部 references。
- 优先先看 `scripts/org_skill_cli.py --help`，再补读对应 reference。
- 文档里引用的返回结构，以当前已核实的服务端代码和实测结果为准。
