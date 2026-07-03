# AuthHub 登录弹窗与用户中心设计

## 背景

发票助手当前已有早鸟 Pro 弹窗和 Pro 模型入口，但没有统一账号体系。新的目标是接入 `/Users/sh/Desktop/sidework/AuthHub` 的公开业务接口，用正式白底弹窗完成登录/注册、用户信息、会员有效期、邀请记录展示。

本设计只覆盖普通用户侧能力，不接入 AuthHub 管理接口，不新增 vue-router，不做支付、订单、复杂权益对比表。

## 范围

### 本期包含

- 登录/注册使用弹窗，不做独立页面。
- 点击右上角圆角头像/账号入口时：
  - 未登录：打开登录/注册弹窗。
  - 已登录：打开用户中心弹窗。
- 选择 Pro 模型时：
  - 未登录：打开登录/注册弹窗，登录成功后继续原 Pro 模型选择流程。
  - 已登录且会员有效：允许切换 Pro 模型。
  - 已登录但会员无效或过期：打开用户中心的会员状态 tab，提示当前未开通或已过期。
- 用户中心使用正式白底 Modal：顶部身份摘要 + 横向 tab + 内容区。
- 邀请记录是唯一重内容 tab；我的资料和会员状态保持轻量。
- AuthHub token 本地持久化，并由 AuthHub 专用请求层统一注入。

### 本期不包含

- 支付、续费、订单记录、权益对比大表。
- 手机号登录、验证码登录、第三方登录。
- 用户头像上传、昵称编辑的真实保存接口。
- vue-router。
- 后台管理接口。

## AuthHub 接口

接口基础地址默认使用：

```text
https://auth.yoloxy.com
```

前端通过环境变量配置：

- `VITE_AUTHHUB_API_URL`
- `VITE_AUTHHUB_APP_KEY`

所有 AuthHub 业务请求带：

```http
X-App-Key: <VITE_AUTHHUB_APP_KEY>
```

登录后请求额外带：

```http
Authorization: Bearer <token>
```

使用的接口：

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`
- `GET /api/membership/status`
- `GET /api/invite/my-summary`
- `GET /api/invite/my-records?page=1&limit=20`
- 可选：`POST /api/invite/use`

AuthHub 成功响应为 `{ success: true, data }`，失败响应为 `{ success: false, error: { code, message } }`。前端以 `error.code` 做少量中文映射，保留后端 `message` 作为兜底。

## 信息结构

### 登录/注册弹窗

尺寸约 420px 宽，白底、12px 圆角、轻边框、遮罩点击关闭。内部用邮箱/密码作为主流程。

登录表单：

- 邮箱
- 密码
- 登录按钮
- 切换到注册

注册表单：

- 邮箱
- 密码
- 邀请码，可选
- 注册按钮
- 切换到登录

登录或注册成功后：

- 保存 token。
- 保存 `user` 和 `membership` 到共享 auth 状态。
- 关闭登录弹窗。
- 如果来源是右上角入口，打开用户中心。
- 如果来源是 Pro 模型入口，继续检查会员状态，再决定是否允许切换 Pro。

### 用户中心弹窗

尺寸建议桌面端 720px 宽、最高 720px；移动端全屏 sheet。背景纯白，12px 圆角，0.5px/1px 边框，遮罩为半透明黑色 + 模糊，点击遮罩关闭。

布局采用：

```text
标题栏：关闭按钮 + 用户中心
身份摘要：头像占位 / 邮箱或昵称 / UID / 会员状态 / 有效期
横向 tab：我的资料 / 会员状态 / 邀请记录 / 账户设置
内容区：根据 tab 渲染；内容区独立滚动
```

不采用 240px 固定左栏。原因是当前只有邀请记录信息量大，固定左栏会压缩所有 tab 的有效宽度，让简单 tab 显得笨重。

## Tab 设计

### 我的资料

轻量信息卡：

- 邮箱
- UID，带复制按钮
- 注册时间
- 最近登录时间
- 本地累计处理发票数，若没有可靠数据则不显示

头像和昵称先做展示占位，不提供真实编辑保存。

### 会员状态

轻量会员卡：

- 当前状态：Pro / 免费 / 已过期
- 有效期：`expire_at > 0` 时显示到期日期；`expire_at = 0` 显示“暂无有效期”
- Pro 模型使用状态：可用 / 不可用
- 引导入口：去邀请记录获取奖励

不放订单记录，不放权益对比表。

### 邀请记录

重内容 tab：

- 三个指标：成功邀请数、已用次数、累计奖励天数。
- 我的邀请码：大号等宽码 + 复制按钮。
- 邀请链接：由当前页面 URL 拼接 `?invite=<code>`，复制分享。
- 邀请记录列表：默认加载第一页，每页 20 条。
- 奖励规则折叠区：默认收起，说明“每成功邀请 1 人奖励 5 天，最多 5 次”等后端规则。

接口调用策略：

- 打开邀请 tab 时优先调用 `/api/invite/my-summary`。
- 记录列表只在邀请 tab 打开后调用 `/api/invite/my-records`。
- 60 秒内重复打开邀请 tab 使用本地内存缓存，避免轮询。

### 账户设置

保持轻量：

- 退出登录。
- 清除本地缓存。
- 消息偏好开关作为本地 UI 状态，若没有后端接口则不持久化到服务端。
- 注销账号暂不实现真实动作，只展示“请联系支持”或不显示。

## 状态与数据流

新增 `src/composables/useAuth.ts`，作为全局单例 composable：

- `token`
- `user`
- `membership`
- `membershipStatus`
- `isLoggedIn`
- `isProActive`
- `authReady`
- `login(email, password)`
- `register(email, password, inviteCode?)`
- `refreshMe()`
- `refreshMembership()`
- `logout()`

本地缓存 key：

- `AUTHHUB_TOKEN`

启动时：

1. 从 localStorage 读取 token。
2. 有 token 则调用 `/api/auth/me`。
3. 成功则恢复 user/membership。
4. 401 或用户不可用则清理 token。

AuthHub 请求注入逻辑：

- 保留现有 `plugins/axios.ts` 的通用能力，不改变既有 `VITE_GLOBAL_API_URL` 请求行为。
- 新增 `src/api/authhub.ts` 作为 AuthHub 专用请求层。
- AuthHub 请求必须注入 `X-App-Key`。
- 有 token 时注入 `Authorization: Bearer <token>`。

## Pro 模型拦截

当前 `AmountRecognitionModeSelect` 组件直接切换 `ai` 并加载模型。改为在选择 Pro 选项时先发出请求事件：

```text
用户选择 ai
→ 父级 Home 检查 useAuth
→ 未登录：打开 LoginModal，记录 pendingProSelection
→ 登录成功：refreshMembership
→ 有效：通知选择器确认切换 ai
→ 无效：打开用户中心会员 tab，保持 default
```

这样 Pro 入口只负责展示选择器，登录和会员判断集中在 Home/useAuth，避免组件里混入业务弹窗状态。

## URL 行为

不使用 vue-router。本期只支持轻量 query 行为：

- 页面打开时如存在 `?invite=<code>`，注册表单的邀请码默认填入该值。
- 可选支持 `?user-center` 打开用户中心，hash 可定位 tab，例如 `?user-center#invite`。

关闭弹窗时不强制清理 URL，除非实现中发现会造成重复弹窗。

## 错误处理

- 登录失败：显示“邮箱或密码错误”。
- 注册邮箱已存在：显示“该邮箱已注册，请直接登录”。
- 邀请码无效：显示“邀请码无效或不属于当前应用”。
- token 失效：清理登录态，打开登录弹窗。
- AuthHub 配置缺失：登录弹窗显示“登录服务暂不可用”，并在控制台输出缺失的环境变量名。

## 文件结构

```text
src/api/authhub.ts
src/composables/useAuth.ts
src/views/user/
├─ LoginModal.vue
├─ UserCenterModal.vue
└─ tabs/
   ├─ ProfileTab.vue
   ├─ MembershipTab.vue
   ├─ InviteTab.vue
   └─ SettingsTab.vue
```

## 验收标准

- 未登录点击右上角账号入口，出现登录/注册弹窗。
- 已登录点击右上角账号入口，出现用户中心弹窗。
- 注册时可带邀请码，成功后保持登录态。
- 刷新页面后 token 有效时能恢复登录态。
- token 失效时自动清理并要求重新登录。
- 未登录选择 Pro 模型会先打开登录弹窗。
- 登录且会员有效时可选择 Pro 模型。
- 登录但会员无效时保持默认模型，并打开用户中心会员 tab。
- 用户中心我的资料和会员状态保持轻量。
- 邀请记录 tab 能展示邀请码、摘要指标和第一页记录。
- 构建和类型检查通过。
