# GitHub Pages 双仓库发布

源码仓库计划使用 `Ke1thLi/Ke1thLi-Blog`，本地 `main` 目前没有远程。已确认的新站发布目标为 `Ke1thLi/Ke1thLi-Blog-Pages`，预期使用 `https://ke1thli.github.io/Ke1thLi-Blog-Pages/`。现有 `Ke1thLi/ke1thli.github.io` 继续承载旧博客。迁移旧站是独立操作。

## 基线提交

1. 从项目根目录执行 `git status --short --untracked-files=all` 和 `git status --ignored --short`。检查规划、源码、静态资源、测试、锁文件与工作流，确认生成产物、环境文件、凭据和截图留在暂存区之外。
2. 逐项暂存已审阅文件，执行 `git diff --cached --check`、`git diff --cached --stat` 和 `git diff --cached`。
3. 审查通过后创建首个本地提交，例如 `git commit -m "chore: initialize Ke1thLi Blog MVP"`。先确认源码仓库实际地址，再添加 `origin`。将基线推送到源码仓库是独立操作。

## 发布设置

1. 在新发布仓库中初始化一个专用产物分支，命名为 `gh-pages`。在该仓库的 **Settings → Pages** 选择 **Deploy from a branch**，选择该分支和 `/ (root)`。发布分支只用于静态产物，工作流会移除上次发布遗留的文件。
2. 在源码仓库设置 Actions variables。 `PAGES_REPOSITORY=Ke1thLi/Ke1thLi-Blog-Pages`，`PAGES_BRANCH=gh-pages`，`PAGES_APP_CLIENT_ID` 填入 GitHub App 的 Client ID，`PAGES_COMMIT_EMAIL` 填入发布账户已验证邮箱或该账户的 GitHub noreply 邮箱。可选的 `BLOG_GITHUB_URL` 控制公开 GitHub 链接。
3. 在源码仓库创建 `pages-publish` 环境，限制默认分支，可按仓库权限启用发布审批。注册专用 GitHub App，仅授予仓库 **Contents: Read and write**，安装范围仅选 `Ke1thLi/Ke1thLi-Blog-Pages`。在该环境配置 `PAGES_APP_PRIVATE_KEY` secret。发布任务生成仅限目标仓库的短期安装令牌，任务结束时自动撤销。源码仓库自带的 `GITHUB_TOKEN` 只用于读源码。
4. 默认分支 push 和 PR 运行验证并上传 `pages-dist`。检查产物后，从源码默认分支手动运行 **Validate and publish Pages**，勾选 **publish**。工作流会验证目标、设置已确认的站点前缀、重新构建，再用普通提交更新目标产物分支。
5. 到目标仓库查看 Pages 部署记录。发布后访问首页、深层文章 URL、一个不存在的 URL、RSS、Sitemap 和搜索。子路径项目的 `robots.txt` 位于项目产物下，爬虫按站点根目录读取 robots 规则。若需根级规则，由旧站的所有者单独维护；可通过 Search Console 提交新站 Sitemap。

发布工作流只接受已确认的源码仓库、目标仓库和 `gh-pages` 分支。发布构建固定使用 `BLOG_SITE_URL=https://ke1thli.github.io` 和 `BLOG_BASE_PATH=/Ke1thLi-Blog-Pages/`。普通 Actions Artifact 仅在构建和发布任务之间传递产物。Pages 来源选择分支发布。

工作流只同步 `dist/`，包括 `404.html`、Pagefind 索引、RSS、Sitemap 和 Robots。部署时写入 `.nojekyll` 以保留 `_astro/` 资源。如果产物或目标分支包含 CNAME，工作流会中止。自定义域名尚未配置。

常规 CI 使用占位站点地址生成审查产物，HTML 带有 `noindex`。手动发布会根据已确认的目标重新构建，并生成对应的 Canonical、RSS 和 Sitemap。

## 故障定位与回滚

- 源码工作流的验证日志区分依赖、类型、内容、路由、构建、目标解析和同步错误。
- 同步成功但站点未更新时，检查目标仓库的 Pages 来源、部署记录、分支保护和发布账户权限。
- 如需回滚，在目标产物分支通过常规 revert 或重发已确认版本完成。保留提交历史。
