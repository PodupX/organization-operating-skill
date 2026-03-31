# Org Reference

## 何时读取

- 要接组织列表、详情、创建、修改、成员、加入组织
- 要获取默认组织头像
- 要判断组织接口的最小请求体和权限限制

## 相关能力

| 能力 ID | API | 方法 |
| --- | --- | --- |
| `web.config.get` | `/outer/api/nl/v1/web/config/get` | `GET` |
| `org.list` | `/outer/api/v1/common/user/org/page` | `GET` |
| `org.detail` | `/outer/api/v1/common/org/info/detail` | `GET` |
| `org.detail.manage` | `/outer/api/v1/common/org/info/basic` | `GET` |
| `org.create` | `/outer/api/v1/common/org/create` | `POST` |
| `org.update` | `/outer/api/v1/common/org/update` | `PUT` |
| `org.member.list` | `/outer/api/v1/common/org/member/list` | `GET` |
| `org.member.page` | `/outer/api/v1/common/org/member/page` | `GET` |
| `org.join` | `/outer/api/v1/common/voiceroom/member/joinOrg` | `POST` |

## `web.config.get`

- 接口：`GET /outer/api/nl/v1/web/config/get`
- 推荐 web 风格请求头：
  - `x-platform=3`
  - `x-device-id`
  - `x-device_id`
  - `x-version`
  - `x-buildnumber`
  - `x-brand`
  - `x-model`
  - `x-system-version`
  - `x-system_version`
- 关键返回：
  - `data.groupAvatarList[]`
- 当前实测：
  - 测试环境返回 6 张默认组织头像
  - `groupAvatarList[].url` 可直接作为 `org.create.avatar`

## `org.create`

- 接口：`POST /outer/api/v1/common/org/create`
- 鉴权：是
- 最小请求体：
  - `name`
  - `avatar`
- 可选字段：
  - `info`
  - `tagId`
  - `background`
  - `location`
  - `country`
  - `city`
  - `region`
  - `ruleInfo`
  - `settings`
  - `avatarNew`
  - `backgroundNew`
- 已确认限制：
  - 名称不能为空
  - 名称重复会抛 `CODE_10225`
  - `avatar` 为空会抛 `CODE_10240`
  - 存在创建资格和创建成本校验
- 实操建议：
  - 创建前先查 `user-info`
  - 若返回 `isAllowCreate=0`，当前账号通常不适合作为稳定创建组织账号
- skill 行为：
  - `org-create` 不传头像时，会自动先调 `web-config-get` 选第一张默认头像

最小示例：

```json
{
  "name": "Agent Org Demo",
  "avatar": "https://na-rs2.podupx.com/avatar/org/orgh1.png",
  "info": "Agent 自动创建的演示组织。"
}
```

## `org.update`

- 接口：`PUT /outer/api/v1/common/org/update`
- 鉴权：是
- 必填字段：
  - `id`
- 可更新字段：
  - `info`
  - `background`
  - `backgroundNew`
  - `avatar`
  - `avatarNew`
  - `country`
  - `city`
  - `region`
  - `location`
  - `ruleInfo`
  - `tagId`
  - `settings`
  - `diplomat`
- 已确认限制：
  - 不能改 `name`
  - 组织不存在时抛 `CODE_10201`
  - `checked=0` 时抛 `CODE_90297`
  - 必须 owner 或具备 `R_M_O`

## 读取类接口

### `org.list`

- 接口：`GET /outer/api/v1/common/user/org/page`
- 作用：查我的或他人的组织分页列表

### `org.detail`

- 接口：`GET /outer/api/v1/common/org/info/detail`
- 作用：成员视角主详情接口
- 推荐优先使用这个接口，而不是 `org.detail.manage`

### `org.detail.manage`

- 接口：`GET /outer/api/v1/common/org/info/basic`
- 作用：管理态补充信息
- 普通成员实测可能命中权限限制

### `org.member.list`

- 接口：`GET /outer/api/v1/common/org/member/list`
- 作用：非分页成员列表，偏 IM / 群同步

### `org.member.page`

- 接口：`GET /outer/api/v1/common/org/member/page`
- Query：
  - `orgId`
  - `page`
  - `count`
  - `mtype`
  - `enablePost`
  - `keyword`
  - `lids`

## `org.join`

- 接口：`POST /outer/api/v1/common/voiceroom/member/joinOrg`
- 最小请求体：
  - `orgId`
  - `roomId` 可传空字符串

## CLI 常用命令

```bash
python scripts/org_skill_cli.py --env test web-config-get
python scripts/org_skill_cli.py --env test org-list --my 1
python scripts/org_skill_cli.py --env test org-detail --org-id 1141
python scripts/org_skill_cli.py --env test org-create --name "Agent Org Demo"
python scripts/org_skill_cli.py --env test org-update --org-id 1141 --info "新的组织简介"
python scripts/org_skill_cli.py --env test org-member-page --org-id 1141 --count 10
```

## 仓库定位

- 组织控制器：`biz-service/.../OrgInfoOuterControllerV1.java`
- 组织 DTO：`biz-service/.../OrgDto.java`
- 组织服务：`biz-service/.../OrgInfoService.java`
- web 配置：`biz-service/.../WebConfigController.java`
