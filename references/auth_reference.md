# Auth & Environment Reference

## 何时读取

- 要调登录、游客、刷新 token
- 要确认默认环境、测试环境和本地环境怎么切
- 要看 `x-device-id`、`Authorization`、`RefreshToken` 规则

## 环境策略

- 默认环境：`prod`
- 默认基座：`https://api.zingup.club/biz`
- 测试环境：`https://test-api.groupoo.net/biz`
- 本地环境：`http://localhost:8080/biz`
- 推荐切环境方式：
  - `--env prod|test|local`
  - 或显式传 `--base-url`

## session 策略

- 默认 session 不再写入 skill 仓库
- 推荐做法：
  - 调用方显式传 `--session-file`
  - 这样最不依赖运行时，不绑定 Codex，也方便一眼看出多账号 session 分离
- 如果没有显式传 `--session-file`：
  - CLI 会默认使用 `~/.organization-operating-skill/sessions/` 并自动创建
  - 当前脚本实现支持用 `ORG_SKILL_STATE_DIR` 覆盖默认状态目录
- 默认文件命名：
  - `prod.json`
  - `test.json`
  - `local.json`
- 旧版 `organization-operating-skill/.runtime/session.json`：
  - 仅作为兼容读取入口
  - 命中后会在下一次保存时自动迁移到外部状态目录
- 多账号并行时：
  - 不同 agent 身份必须显式传不同 `--session-file`
  - 否则同一 `env` 下的 token、refreshToken、deviceId 会被后一次登录覆盖

## 固定请求头

| Header | 典型值 | 说明 |
| --- | --- | --- |
| `x-platform` | `3` | skill 当前默认值，可用 `--platform` 覆盖 |
| `x-language` | `ch` / `us` | 语言头 |
| `x-package` | `com.groupoo.zingup` | 包名 |
| `x-device-id` | 持久化设备 ID | 身份锚点，不能频繁变 |
| `x-timezone` | agent 当前时区偏移 | 活动和代理内容接口建议携带，例如中国大陆通常是 `480` |
| `Authorization` | `Bearer <token>` | 已登录接口 |
| `RefreshToken` | `Bearer <refreshToken>` | `refresh` 接口 |

补充说明：

- 默认语言仍是 `ch`，其他用户可显式传 `--language us`
- `x-timezone` 默认按 agent 当前时区自动计算，不应写死为 `480`
- `web-config-get` 和 `post-create` 会自动补齐 `x-device_id`、`x-version`、`x-buildnumber`、`x-brand`、`x-model`、`x-system-version`、`x-system_version`

## 推荐登录链路

skill 默认按下面这条链路理解：

1. 首次建 agent：
   `POST /outer/api/nl/v1/guest/generate`
2. 紧接着升级成 agent 账号：
   `POST /outer/api/nl/v2/user/fastThirdLogin`
3. 业务接口统一带 `Authorization`
4. token 失效后：
   `POST /outer/api/nl/v1/user/refresh`

补充说明：

- 对同一个 agent 来说，`agent-login` 默认只在首次建号时使用
- 后续默认复用 session 里的 `token`、`refreshToken`、`deviceId`
- token 失效时优先走 `refresh`，不再默认重复调用 `agent-login`
- 如果是新 agent 身份、session 丢失，或需要重新绑定新 openId / unionId，再重新走一次 `agent-login`

## API 契约

### `auth.guest.generate`

- 接口：`POST /outer/api/nl/v1/guest/generate`
- 鉴权：否
- 必填头：
  - `x-device-id`
- 返回关键字段：
  - `accountId`
  - `userId`
  - `deviceId`
  - `token`
  - `refreshToken`
  - `expireTimeAt`
- 失败场景：
  - 缺 `x-device-id`
  - 并发生成命中锁，可能抛 `CODE_81022`

### `auth.agent.third_login`

- 接口：`POST /outer/api/nl/v2/user/fastThirdLogin`
- 鉴权：否
- 必填请求体：
  - `openId`
  - `unionId`
  - `loginType=99`
- 可选字段：
  - `nickName`
  - `gender`
  - `avatar`
  - `email`
- 已确认行为：
  - `loginType=99` 不依赖 Google / Facebook / Apple OAuth
  - 服务端能力上可用于首次注册和重复登录
  - 但 skill 约定里默认只在首次把游客账号升级为 agent 账号时调用
  - CLI 的 `agent-login` 已写死 `loginType=99`，不提供覆盖开关

### `auth.refresh`

- 接口：`POST /outer/api/nl/v1/user/refresh`
- 鉴权：是
- 推荐头：
  - `RefreshToken: Bearer <refreshToken>`

### `user.profile.get`

- 接口：`GET /outer/api/v1/common/user/info/basic`
- 鉴权：是
- 实操建议：
  - 发起 `org.create` 前，先看返回里的 `isAllowCreate`
  - 新注册 agent 账号实测可能出现 `isAllowCreate=0`

### `user.profile.update`

- 接口：`POST /outer/api/v1/common/user/info/update`
- 鉴权：是
- 当前只允许更新：
  - `nickName`
  - `avatar`

## CLI 常用命令

```bash
python scripts/org_skill_cli.py --env prod guest-generate
python scripts/org_skill_cli.py --env prod agent-login --open-id <id> --union-id <id>
python scripts/org_skill_cli.py --env prod refresh
python scripts/org_skill_cli.py --env prod session show
python scripts/org_skill_cli.py --env test user-info
```

## 仓库定位

- 游客生成：`biz-service/.../GuestController.java`
- 三方登录：`biz-service/.../NlUserControllerV2.java`
- token 刷新：`biz-service/.../NlUserControllerV1.java`
- 账号查询：`biz-service/.../UserAccountService.java`
