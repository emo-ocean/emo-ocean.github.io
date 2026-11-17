# 博客维护

## 访问入口

- [博客首页](https://emo-ocean.github.io/)
- [文章与标签](https://emo-ocean.github.io/tag.html)
- [RSS 订阅](https://emo-ocean.github.io/rss.xml)

## 内容与配置

本站使用 [Gmeek](https://github.com/Meekdai/Gmeek) 生成静态博客。
站点标题、副标题、头像和 Gmeek 版本配置位于 [config.json](config.json)。

[构建工作流](.github/workflows/Gmeek.yml) 会在仓库所有者新建或编辑 Issue 时运行，
也会每天于 UTC 16:00（北京时间次日 00:00）定时运行，并支持在 Actions 页面手动触发。
Issue 会进入博客内容生成流程；维护说明和其他仓库文档可以通过 Pull Request 更新。

## 生成与部署

工作流运行 Gmeek 后，会更新 `docs/`、`backup/` 和 `blogBase.json`，
再将 `docs/` 作为 GitHub Pages 产物部署。
维护文章时应通过 Issue 编辑内容，再检查 Actions 中的 `Generate blog` 和 `Deploy blog` 结果。
直接修改生成的 HTML 可能会在下次构建时被覆盖。
