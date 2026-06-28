# 14. 清理旧静态描述文件

## 目标

把已经由后端动态路由接管的 `manifest.json` 和 `opensearch.xml` 静态旧文件从 fork 中移走，避免浏览器读到过时品牌信息。

## 这次改动解决什么问题

在自定义品牌和全源码部署已经落地后，旧的静态描述文件会造成三个问题：

- 浏览器可能继续读取旧 manifest 或旧搜索配置。
- 新旧品牌描述并存，排查 PWA 名称、搜索入口和缓存时容易看错对象。
- Git diff 里长期混着无用描述文件，后续拆提交和回滚会变得不清楚。

这次只删除确认已经由后端动态接口替代的旧文件：

- `static/manifest.json`
- `static/opensearch.xml`

注意：`static/image-placeholder.png` 仍被富文本图片组件引用，Leaflet marker 图片仍可能被地图组件使用，`robots.txt` 也不是这次的品牌描述迁移目标，所以都不放进本提交。

## 手工复现流程

1. 先确认当前品牌描述已经由后端动态路由接管：

   ```bash
   rg -n "@app.get\\('/manifest.json'\\)|@app.get\\('/opensearch.xml'\\)" backend/open_webui/main.py
   ```

2. 检查旧静态文件内容是否已经过时：

   ```bash
   git show HEAD:static/manifest.json
   git show HEAD:static/opensearch.xml
   ```

3. 如果后端动态接口已经负责返回品牌化内容，就从 Git 中删除旧静态描述文件：

   ```bash
   git rm static/manifest.json static/opensearch.xml
   ```

4. 不要把本地临时文件一起提交。比如调试时出现的空 `static/custom.css`、`static/loader.js`、`static/user-import.csv`，应该留在工作树外，等确认来源后再决定是否忽略或删除。

## 验证门禁

- 后端存在 `/manifest.json` 和 `/opensearch.xml` 动态路由。
- `static/image-placeholder.png`、Leaflet marker 图片和 `robots.txt` 没有被误删。
- `git diff --cached --check` 没有空白错误。
- staged 文件里没有 `.env`、token、Cookie、数据库和本机私密路径。
- 课程 HTML 可以被基础 HTML parser 读取。

## 背后的知识点

静态描述文件清理不是简单删文件，而是在整理三层边界：

- 源码入口：应用实际从哪里引用图标、manifest、搜索配置。
- 部署入口：Nginx、Docker 和品牌补丁实际向浏览器暴露什么。
- 浏览器缓存：用户看到的 favicon、PWA 名称和旧资源可能来自缓存，需要通过版本化资源和明确入口减少误判。
