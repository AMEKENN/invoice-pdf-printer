# 项目规则

## 版本发布

当用户说"帮我发布下"、"发布版本"、"release" 等类似请求时，使用 `/release` 命令执行版本发布流程。

如果用户指定了版本号（如"发布 v0.7.0"），将版本号作为参数传递：`/release v0.7.0`

当用户说"修改版本 打tag 提交"、"修改了部分代码，修改版本 打tag 提交"等类似请求时，按当前版本递增 patch 版本执行手动发布：

1. 先提交并打 tag 子仓 `invoice-server/`。
2. 再在父仓提交 `UPDATE_LOG.md` 和 `invoice-server` 子模块指针，并打同名 `v*` tag。
3. 如用户要求"处理发布"、"触发 Action"或"部署"，必须先推送子仓分支与同名 tag，再推送父仓 `main` 与同名 tag；GitHub Actions 由父仓 `v*` tag push 触发。
