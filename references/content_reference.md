# Content Reference

## 何时读取

- 要在组织里发帖子
- 要确认帖子分享链接
- 要排查帖子读取相关接口为什么和写接口表现不同

## 相关能力

| 能力 ID | API | 方法 |
| --- | --- | --- |
| `content.post.create` | `/outer/api/proxy/meta/api/v1/content/article/create` | `POST` |

## `content.post.create`

- 接口：`POST /outer/api/proxy/meta/api/v1/content/article/create`
- 鉴权：是
- 推荐请求头：
  - `Authorization`
  - `x-platform=3`
  - `x-device-id`
  - `x-device_id`
  - `x-timezone`
  - `x-version`
  - `x-buildnumber`
  - `x-brand`
  - `x-model`
  - `x-system-version`
  - `x-system_version`
- 最小请求体：
  - `orgId`
  - `article.richText`
  - `article.visibled`
  - `article.vcmUids`
  - `saasList`

最小示例：

```json
{
  "orgId": 1141,
  "article": {
    "richText": [
      { "insert": "Agent 联调帖：验证发帖能力。\n" }
    ],
    "visibled": 0,
    "vcmUids": []
  },
  "saasList": []
}
```

## 已确认行为

- 代理层会做 camelCase / snake_case 转换
- `post-create` 命令已经内置 web 发帖附加头
- 当前写接口比读接口稳定
- 当前所谓“求助帖”没有单独 API，实际就是普通帖子能力
- 测试环境给人验收时，优先返回 `sharePost` 页面链接

## 读取限制

- 服务端存在 `ContentArticleOuterControllerV1`
- 但测试环境里，通过通用请求直调帖子详情 / 列表接口时，实测可能返回：
  - `code=1144`
  - `msg=System error.`
- 这意味着当前 skill 第一阶段更适合把帖子能力定位为“写闭环优先”

## 分享链接

服务端分享基座配置：

- 生产：`https://share.zingup.club/`
- 测试 / 本地：`https://test-share.groupoo.net/`

推荐按下面的 SSR 规则理解帖子分享链接：

```text
{shareBase}ssr/sharePost/{articleId}?id={articleId}&t2=1&t8=ch
```

补充验证：

- 测试环境历史实测链接 `https://test-web.groupoo.net/ssr/sharePost/{articleId}?id={articleId}&t2=1&t8=ch` 也可打开分享页
- `sharePost` 页面标题会直接显示帖子正文摘要
- `shareArticle` / `shareDynamic` 在当前测试环境下能打开页面，但内容命中率不如 `sharePost`

配置来源：

- `application-prod.yml` 的 `biz-config.share.url=https://share.zingup.club/`
- `application-test.yml` / `application-local.yml` 的 `biz-config.share.url=https://test-share.groupoo.net/`

## CLI 常用命令

```bash
python scripts/org_skill_cli.py --env test post-create --org-id 1141 --text "招募一位活动摄影志愿者"
```

## 仓库定位

- 代理控制器：`biz-service/.../ProxyController.java`
- 帖子外层控制器：`biz-service/.../ContentArticleOuterControllerV1.java`
