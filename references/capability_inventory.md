# Capability Inventory

这份文档记录 organization-operating-skill 当前已经接入的能力，以及后续仍值得补充的缺口。
目标不是一次做全，而是优先把平台通用动作和组织个性化规则分开。

## 分层原则

- `skill` 负责动作能力：调用 API、拿结果、做最小校验。
- `agent` 负责判断与表达：选择何时调用、如何引导用户、何时升级人工。
- 组织配置负责规则差异：组织目标、可发起的互助类型、匹配字段、活动节奏、信誉规则。

## 当前已接入能力

当前已经具备可执行闭环，不需要再把下面这些能力当成待办：

- 认证闭环：
  `auth.guest.generate`、`auth.agent.third_login`、`auth.refresh`
- 用户资料：
  `user.profile.get`、`user.profile.update`
- 组织闭环：
  `web.config.get`、`org.list`、`org.detail`、`org.detail.manage`、`org.create`、`org.update`、`org.member.list`、`org.member.page`、`org.join`
- 内容闭环：
  `content.post.create`
- 活动闭环：
  `activity.save`、`activity.publish`、`activity.cancel`、`activity.delete`、`activity.detail`、`activity.search`、`activity.org.list`、`activity.user.sign.list`、`activity.sign.list`、`activity.signup`

当前运营闭环主要通过帖子发布和活动能力承接。
当前所谓“求助”能力也是通过 `content.post.create` 发普通帖子来承接，还没有独立的求助发布接口。

## 后续待补能力

下面这些仍然值得补，但已经不阻塞当前 skill 的最小闭环：

| 优先级 | 能力 ID | 能力名称 | 作用 | 典型输入 | 典型输出 | 是否建议补 API |
| --- | --- | --- | --- | --- | --- | --- |
| P0 | interaction.comment.create / list | 互动与点评 | 支撑互评、追问、补充说明 | 目标内容 ID、评论内容 | 评论记录、时间、作者 | 是 |
| P1 | help.offer.create / list | 发布可提供帮助 | 支撑“我能帮什么”这条主链路 | 组织 ID、帮助类型、说明 | 帮助供给记录 | 建议 |
| P1 | match.recommend | 匹配推荐 | 让 agent 主动推荐可响应的人或任务 | 组织 ID、请求 ID、规则字段 | 候选成员 / 候选任务 | 建议 |
| P1 | interaction.feedback.create | 感谢与反馈 | 为信誉系统提供正向数据 | 目标 ID、评分、感谢语 | 反馈结果 | 建议 |
| P2 | reputation.get | 信誉 / 积分查询 | 排序、推荐、激励 | 用户 ID、组织 ID | 积分、信誉、徽章 | 可后补 |
| P2 | summary.org.weekly | 摘要与周报 | 让 agent 输出组织总结和活跃信息 | 组织 ID、时间范围 | 汇总数据、案例 | 可后补 |
| P2 | moderation.report | 风险上报 | 处理骚扰、失约、线下安全等风险 | 目标 ID、原因、证据 | 举报结果 | 可后补 |

## 组织配置，不应该直接做成 skill 能力

下面这些内容优先放进组织配置，而不是先写死到 skill 里：

- 组织名称与一句话定义
- 适合加入的人群
- 允许的求助类型
- 允许的帮助类型
- 匹配字段，例如城市、风格、档期、线上线下
- 固定活动节奏
- 成功互助标准
- 风险规则和禁用行为
- 文案语气和欢迎词

## 你后续提供 API 时，我更希望按这个顺序补

1. 评论 / 点评
2. 发布 / 查询可提供帮助
3. 匹配推荐
4. 感谢与反馈
5. 信誉、周报、风控

## 每个能力需要你补哪些信息

每补一个新增能力，请尽量同时告诉我：

- 对应的 API 是什么
- 在项目中怎么找到它
- 注册 / 登录 / token 链路是否涉及它
- 请求方法、路径、鉴权方式
- 必填参数、可选参数
- 返回字段里哪些最关键
- 常见错误码或失败场景
- 有没有现成的前端 / 服务端调用示例
