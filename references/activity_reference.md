# Activity Reference

## 何时读取

- 要保存 / 发布 / 下架 / 删除活动
- 要做免费票活动闭环
- 要查询活动详情、搜索、报名、报名列表
- 要生成活动分享链接

## 相关能力

| 能力 ID | API | 方法 |
| --- | --- | --- |
| `activity.save` | `/outer/api/v1/common/activity/save` | `PUT` |
| `activity.publish` | `/outer/api/v1/common/activity/publish` | `POST` |
| `activity.cancel` | `/outer/api/v1/common/activity/cancel` | `PUT` |
| `activity.delete` | `/outer/api/v1/common/activity/delete` | `DELETE` |
| `activity.detail` | `/outer/api/v1/common/activity/detail` | `GET` |
| `activity.search` | `/outer/api/v1/common/activity/search` | `GET` |
| `activity.org.list` | `/outer/api/v1/common/activity/org/ac` | `GET` |
| `activity.user.sign.list` | `/outer/api/v1/common/activity/user/sign/list` | `GET` |
| `activity.sign.list` | `/outer/api/v1/common/activity/sign/signList` | `GET` |
| `activity.signup` | `/outer/api/v1/common/activity/orders/signup` | `POST` |

## `activity.save`

- 接口：`PUT /outer/api/v1/common/activity/save`
- 鉴权：是
- 必填头：
  - `Authorization`
  - `x-language`
  - `x-timezone`
- 当前最小关键字段：
  - `title`
  - `orgId`
  - `beginTime`
  - `endTime`
  - `venue`
  - `openSign`
  - `richText`
  - `tickets`
- 已确认限制：
  - `orgId` 不能为空
  - `title` 不能为空
  - `tickets` 至少一张
  - 至少一张票 `hiddenTicket=0`
- 关键返回：
  - `data.id` 可作为后续发布用的草稿 `id`
  - `data.tickets[]` 里可拿报名所需票 `id`

## `activity.publish`

- 接口：`POST /outer/api/v1/common/activity/publish`
- 最小请求体：
  - `{ "id": <draftId> }`
- 已确认限制：
  - `endTime >= 当前时间`
  - `beginTime <= endTime`
  - `publish` 会先按 `dto.id` 读取已保存草稿，再做发布校验
  - skill 当前只允许免费票

发布链路请按下面顺序理解：

1. `activity.save` 提交完整活动草稿
2. 从返回里拿 `data.id`
3. `activity.publish` 传草稿 `id`

## 免费票最小示例

```json
{
  "title": "Agent Activity Demo",
  "orgId": 1141,
  "quantity": 30,
  "quantityLimit": 1,
  "beginTime": 1774780022676,
  "endTime": 1774790822676,
  "published": 0,
  "venue": 2,
  "openSignInfo": 0,
  "openSign": 1,
  "customerServices": [],
  "richText": [
    { "insert": "Agent 联调活动：验证免费票活动能力。\n" }
  ],
  "tickets": [
    {
      "title": "Free Ticket",
      "description": "Agent 验证免费票",
      "openTicketLimit": 1,
      "ticketLimit": 30,
      "openSalesStart": 0,
      "openSalesEnd": 0,
      "hiddenTicket": 0,
      "status": 1,
      "paid": 0,
      "amount": 0,
      "currency": 1
    }
  ],
  "multipleTicket": 0
}
```

## 其他接口

### `activity.cancel`

- 最小请求体：`{ "id": <activityId> }`

### `activity.delete`

- 最小请求体：`{ "id": <activityId> }`

### `activity.detail`

- Query：`id`

### `activity.search`

- Query：
  - `keyword`
  - `page`
  - `pageSize`
  - `nextId`

### `activity.org.list`

- Query：
  - `orgId`
  - `tab`
  - `location`
  - `keyword`
  - `page`
  - `pageSize`
  - `nextId`

### `activity.user.sign.list`

- 查询当前用户已报名活动

### `activity.sign.list`

- 查询活动报名列表
- 当前账号实测可能因为权限限制失败

### `activity.signup`

- 最小请求体至少要带：
  - `id`
  - `tickets[].id`
  - `tickets[].quantity`
- 实操建议：
  - 报名前先 `join-org`，用独立 session 代表报名 agent

## 分享链接

服务端分享基座配置：

- 生产：`https://share.zingup.club/`
- 测试 / 本地：`https://test-share.groupoo.net/`

推荐按下面的 SSR 规则理解活动分享链接：

```text
{shareBase}ssr/shareEvent/{activityId}?id={activityId}&t2=2&t8=ch
```

补充验证：

- 测试环境历史实测链接 `https://test-web.groupoo.net/ssr/shareEvent/{activityId}?id={activityId}&t2=2&t8=ch` 可打开页面
- 服务端邮件和报名逻辑也都在拼这个 `shareEvent` 规则

代码证据位置：

- `biz-service/.../ActivityEmailService.java`
- `biz-service/.../ActivitySignUpService.java`
- `application-prod.yml` / `application-test.yml` / `application-local.yml` 里的 `biz-config.share.url`

## CLI 常用命令

```bash
python scripts/org_skill_cli.py --env test activity-save --json-file activity.json
python scripts/org_skill_cli.py --env test activity-publish --draft-id 5912
python scripts/org_skill_cli.py --env test activity-detail --activity-id 5911
python scripts/org_skill_cli.py --env test activity-search --keyword meetup
```

## 仓库定位

- 活动控制器：`biz-service/.../ActivityOuterController.java`
- 活动服务：`biz-service/.../ActivityInfoService.java`
- 报名服务：`biz-service/.../ActivitySignUpService.java`
- 票务逻辑：`biz-service/.../ActivityTicketLogic.java`
