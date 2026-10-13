# Supabase + GitHub Pages 发布包

此包包含：
- `site/index.html`：GitHub Pages 只读网页；Supabase 是规则与二维码索引的云端数据源。
- `supabase-schema-seed.sql`：创建只读规则表、二维码索引、Supabase Storage bucket 和 59 条当前剂量规则。
- `site/supabase-config.js`：需填写 Supabase 项目 URL 和 **Publishable Key**。
- `.github/workflows/pages.yml`：推送到 GitHub `main` 或 `master` 分支后自动发布到 GitHub Pages。

云端包只包含公开只读站点；单机维护版文件仍独立保留在工作区，不会一起部署到 GitHub Pages。

## 先说明：我无法替你创建真实项目或域名

当前环境中的 Supabase 与 GitHub 工具连接器均不支持，且没有你的 Supabase/GitHub 账户授权。因此我可以交付可发布包，但不能代你创建 Supabase 项目、推送仓库或取得最终网址。实际的 `github.io` 地址要在你的 GitHub 仓库启用 Pages 并完成一次 workflow 部署后产生。

## 发布步骤

### A. 创建 Supabase 云端数据库

1. 在 Supabase 创建一个项目。
2. 打开 SQL Editor，将 `supabase-schema-seed.sql` 全文执行。脚本创建 `drug_rules`、`drug_qr` 两张表，并初始化当前 59 条规则；匿名角色只有 SELECT 权限，没有公开写入策略。
3. 在项目 API 设置中复制 Project URL 和 **Publishable Key**，编辑 `site/supabase-config.js`：
   - `url` 填项目 URL，例如 `https://项目引用.supabase.co`。
   - `publishableKey` 填 `sb_publishable_...`。
4. **绝不要**把 `sb_secret_...` 或旧版 `service_role` key 放进 `site/` 或任何前端文件。前端 Publishable Key 可见是预期行为，实际保护依赖 SQL 中的 RLS 与只读权限。

Supabase 对前端建议使用 Publishable Key，并要求对公开表启用 RLS；Secret/service_role key 会绕过 RLS，不能放入网页。参见 [Supabase API key 文档](https://supabase.com/docs/guides/getting-started/api-keys) 与[数据安全文档](https://supabase.com/docs/guides/database/secure-data)。

### B. 发布到 GitHub Pages

1. 将本包内容上传到一个 GitHub 仓库的根目录。
2. 在仓库 **Settings → Pages → Build and deployment → Source** 选择 **GitHub Actions**。
3. 确认 `site/supabase-config.js` 已填写并提交。workflow 会在 `main` 分支更新时部署 `site/`。
4. 等待 **Actions** 中的 Pages workflow 成功；随后到 **Settings → Pages** 查看实际网站网址。GitHub 官方流程要求由 Actions 上传并部署 Pages artifact，详见[GitHub Pages 文档](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)。

网址一般形如 `https://<GitHub用户名>.github.io/<仓库名>/`，但具体地址由 GitHub 仓库和 Pages 设置决定，本包不能预先代替 GitHub 生成。

## 云端二维码

当前源数据库有 0 个二维码关联。将来如需给只读网站显示二维码：在 Supabase Storage 的 `instructions` bucket 上传图片，再在 `drug_qr` 表插入对应 `drug_id`、`drug_name`、`storage_path` 和 `original_name`。SQL 只授权匿名读取 Storage 对象，不授权网页上传或删除。

## 安全与医疗提示

- 公网页面只读；规则维护、导入、编辑和图片上传都关闭。
- Supabase 前端 key 是 Publishable Key；不要使用 Secret/service_role key。
- 公开网页仅公开剂量规则和二维码索引。患者输入留在浏览器端计算；不要把患者资料写入 Supabase。
- 此平台为辅助工具；剂量须依现行说明书和临床方案核对。
