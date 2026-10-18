---
{"dg-publish":true,"permalink":"/03-resources/031-obsidian/obsidian-digital-garden-cloudflare-pages/","tags":["obsidian","digital-garden","cloudflare","运维"],"dg-note-properties":{"tags":["obsidian","digital-garden","cloudflare","运维"],"date":"2026-10-18"}}
---



本文记录了如何使用 **Obsidian Digital Garden (DG)** 插件，配合 **GitHub**、**Cloudflare Pages** 以及 **阿里云域名解析**，零成本、全自动地将本地 Obsidian 笔记发布为个人知识库网站。

---

## 一、 方案架构与优势

* **Obsidian DG 插件**：负责在 Obsidian 内一键将 Markdown 笔记通过 GitHub API 推送到远程仓库。
* **GitHub**：作为源码与笔记的中转存储中心。
* **Cloudflare Pages**：监听 GitHub 变更，自动构建 Eleventy (11ty) 静态站点，并提供不限流量的全球 CDN 加速。
* **阿里云 DNS**：托管主域名，通过 CNAME 记录将自定义二级域名解析至 Cloudflare Pages。

---

## 二、 详细部署步骤

### 阶段一：准备 GitHub 仓库与 Token

1. **Fork 模板仓库**
   * 访问官方 Digital Garden 模板仓库：[github.com/oleeskild/digitalgarden](https://github.com/oleeskild/digitalgarden)
   * 点击右上角的 **Fork**，将其克隆到你自己的 GitHub 账号下。

2. **生成 GitHub 访问 Token (PAT)**
   * 进入 GitHub $\rightarrow$ 右上角头像 **Settings** $\rightarrow$ 左侧最下方的 **Developer settings**。
   * 选择 **Personal access tokens** $\rightarrow$ **Tokens (classic)** $\rightarrow$ **Generate new token (classic)**。
   * **Note** 填写：`Obsidian DG`
   * **Expiration** 选择：`No expiration`（永不过期）
   * **Select scopes**：勾选整个 **`repo`** 权限。
   * 点击 **Generate token**，**复制并妥善保存生成的 Token**（离开页面后不予显示）。

---

### 阶段二：部署到 Cloudflare Pages

1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)。
2. 在左侧菜单栏点击 **Workers 和 Pages** $\rightarrow$ 选择 **Pages** 标签页 $\rightarrow$ 点击 **连接到 Git**。
3. 授权并选中刚才 Fork 的 `digitalgarden` 仓库，点击 **开始设置**。
4. **填写构建参数 (Build configuration)**：
   * **项目名称**：自定义（如 `my-garden`）
   * **生产分支**：`main`
   * **框架预设**：选择 `None` 或 `Eleventy`
   * **构建命令**：`npm run build`
   * **构建输出目录**：`dist`
5. 点击 **保存并部署**，等待 1-2 分钟构建完成，系统会分配一个 `.pages.dev` 的默认网址。

---

### 阶段三：在阿里云配置域名解析

1. **在 Cloudflare Pages 中绑定自定义域名**
   * 进入你的 Cloudflare Pages 项目 $\rightarrow$ **自定义域名** 标签页 $\rightarrow$ **设置自定义域名**。
   * 输入你的二级域名（例如：`garden.yourdomain.com`），点击继续，Cloudflare 会给出一条 CNAME 目标地址（如 `my-garden.pages.dev`）。

2. **去阿里云“云解析 DNS”添加 CNAME 记录**
   * 登录 [阿里云控制台](https://dc.console.aliyun.com/) $\rightarrow$ **云解析 DNS** $\rightarrow$ 找到你的主域名点击 **解析设置**。
   * 点击 **添加记录**：
     * **记录类型**：`CNAME`
     * **主机记录**：填写二级域名前缀（如 `garden`）
     * **记录值**：填写 Cloudflare 提供的 Pages 网址（例如 `my-garden.pages.dev`，不带 `https://`）
   * 点击 **确认**。
   * 返回 Cloudflare Pages 点击 **激活域名**，Cloudflare 会自动验证并颁发免费 HTTPS 证书。

---

### 阶段四：配置 Obsidian DG 插件

1. 打开 Obsidian $\rightarrow$ **设置** $\rightarrow$ **第三方插件** $\rightarrow$ 搜索并安装 **Digital Garden** 并启用。
2. 在 Digital Garden 插件设置中填写：
   * **GitHub Authentication Token**：粘贴阶段一生成的 `ghp_xxxx` Token。
   * **GitHub Username**：你的 GitHub 账号名。
   * **GitHub Repo Name**：你的仓库名（默认 `digitalgarden`）。
   * **Base URL**：填写绑定的完整域名（如 `https://garden.yourdomain.com`）。
3. 点击 **Test Connection** 确保连接成功。

---

## 三、 日常写作与发布工作流

1. **标记发布**：在任何需要公开的笔记顶部添加 YAML 属性：
   ```yaml
   ---
   dg-publish: true
   ---