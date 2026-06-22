# 文献摘录墙 · 部署与使用指南

## 文件结构

```
literature-wall/             ← 你的 GitHub 仓库
├── index.html               ← 网页本体
├── data.json                ← 所有卡片的元数据（一开始是空数组）
├── images/                  ← 截图文件夹（一开始空着，导出时会自动建好）
└── README.md                ← 本文档（可选）
```

---

## 一、第一次部署（30 分钟左右）

### 1. 注册 GitHub 账号

如果还没有就去 https://github.com 注册一个。用户名建议简短，因为之后的网址会用到（比如 `caiyindong.github.io`）。

### 2. 新建一个公开仓库

- 右上角 `+` → `New repository`
- 仓库名随意，比如 `literature-wall`
- 选 `Public`（GitHub Pages 免费版要求公开）
- 勾选 `Add a README file`
- 点 `Create repository`

### 3. 上传初始文件

进入新仓库 → 点 `Add file` → `Upload files` → 把 `index.html` 和 `data.json` 拖进去 → 提交。

（`images/` 文件夹现在还不需要，等你导出第一个数据包时会自动有。）

### 4. 开启 GitHub Pages

- 仓库页面顶部 → `Settings`
- 左侧菜单 → `Pages`
- `Source` 选 `Deploy from a branch`
- `Branch` 选 `main`，文件夹 `/ (root)`，点 `Save`
- 等 1 分钟左右，页面上方会出现绿色提示：你的网址是 `https://你的用户名.github.io/literature-wall/`

把这个网址收藏起来——任何设备的浏览器打开它就能看你的摘录墙。

---

## 二、（推荐）配置 GitHub 一键发布

设好之后，网页右上角会出现"🚀 一键发布"按钮，点一下就把所有草稿（图片 + 元数据）直接 commit 到仓库，不用再下载 zip。

### 1. 生成 Personal Access Token (PAT)

打开 https://github.com/settings/personal-access-tokens → `Generate new token` → `Fine-grained personal access token`。

填写：
- **Token name**：随便起，比如 `literature-wall-publish`
- **Expiration**：建议 90 天（到期重新生成一个即可）
- **Repository access**：选 `Only select repositories`，然后选你新建的那个仓库（`literature-wall`）
- **Repository permissions** → 找到 `Contents`，选 `Read and write`

点 `Generate token`，复制出来的字符串（以 `github_pat_` 开头），**马上复制保存**，离开页面就再也看不到了。

### 2. 在网页里填好

打开你的网页 → 点右上角 ⚙ 按钮 → 填四项：
- GitHub 用户名（比如 `caiyindong`）
- 仓库名（`literature-wall`）
- 分支（默认 `main`）
- 粘贴刚才那个 token

点"测试连接"，看到 `✓ 连接成功，有写入权限` 就保存。

之后你只要点"🚀 一键发布"，等几秒进度条跑完，所有草稿就上 GitHub 了。

### 关于 token 安全

- Token 只存在你当前这个浏览器的 localStorage 里
- 换电脑要重新填一遍
- 如果电脑可能被别人用，**不要填这里**，继续用下面的"导出 zip"方案
- Token 写错或过期，"一键发布"会失败并提示，不会造成静默错误
- 在设置弹窗里有"清除配置"按钮可以随时删掉

---

## 三、日常使用流程

### 添加新卡片（在你的主电脑上）

1. 打开你的网页（不管是 GitHub Pages 上的还是本地的 `index.html`）
2. 点右上角 `+ 新建`，或者随便哪里 `Ctrl+V` 粘贴一张截图（会自动打开新建表单）
3. 填写来源、页码、笔记、跳转链接、标签
4. 保存 → 这张卡片暂时进入"本地草稿"（浏览器 localStorage），顶部会有红色提示条

可以一次性加好几十张，攒一批再发布。

### 发布到仓库（让所有设备能看到）

**模式 A：一键发布（推荐，需要先按第二节配好 PAT）**

1. 点右上角 `🚀 一键发布`
2. 等进度条跑完（图片越多越慢，但通常 10-30 秒）
3. 等约 30-60 秒，GitHub Pages 重新部署完，其他设备刷新就能看到

**模式 B：导出 zip（备用，不用 PAT）**

1. 点右上角 `📥 导出 zip`
2. 浏览器下载一个 zip，解压
3. 把里面的 `data.json` 拖到 GitHub 仓库（网页直接拖拽就能上传，会覆盖原来的）
4. 把里面的 `images/` 文件夹里的所有图片，上传到仓库的 `images/` 文件夹里
5. 提交（Commit changes）
6. 等约 30-60 秒，GitHub Pages 会自动重新部署
7. 刷新网页 → 看到所有卡片都是"正式"状态（不再是草稿）

### 在其他设备查看

直接打开网址即可。其他设备上不要新建卡片（因为它们的 localStorage 是独立的，加了也只在那台设备可见）。

---

## 四、关于 Zotero 跳转链接

每个 Zotero 条目都有一个全球唯一的链接，格式是：

```
https://www.zotero.org/users/你的userID/items/8位itemKey
```

获取方法：

1. **找你的 userID**：登录 https://www.zotero.org，进入设置 → `Feeds/API` 页面，最上面那串数字就是。
2. **找 itemKey**：在 Zotero 桌面端，右键任意条目 → "查看在线"，浏览器打开的网址末尾 8 位就是。

把它填到摘录卡片的"跳转链接"栏，效果是：
- 装了 Zotero 桌面端的电脑点击 → 直接打开本地 Zotero 那篇文献
- 手机/平板点击 → 在浏览器打开 Zotero 网页版（前提是你 Zotero 开了云同步）

不想用 Zotero 也行，链接栏可以填 DOI、本地 PDF 路径（`file:///...`，但只在本机有效）、或者任意 URL。

---

## 五、推荐的截图工作流

在 Zotero 6/7 自带的 PDF 阅读器里：

1. 工具栏选**区域标注**工具（一个矩形图标）
2. 在 PDF 上画一个框，框住你要的图/表/段落
3. 标注会显示在右侧栏，右键 → "复制图片"
4. 切回你的摘录墙网页，`Ctrl+V` 粘贴
5. 填好元数据 → 保存

好处是图片的页码、来源都还在你脑子里，顺手就填了。

---

## 六、常见问题

**Q: 一键发布失败了怎么办？**

弹窗会显示具体错误信息。常见原因：
- **401 Unauthorized**：token 错了或者过期了，去设置里重新填一个
- **404 Not Found**：用户名或仓库名拼写错了
- **403 / 没有写入权限**：生成 token 时没勾 Contents: Read and write
- **网络错误**：可能是 GitHub API 在你网络下不通，挂代理试试
- **422 / 部分图片已存在**：上次发布到一半失败留下的痕迹，再点一次"一键发布"会自动重试，不会重复

如果反复失败，用"📥 导出 zip"作为备用方案，永远管用。

**Q: 一键发布之后，刷新页面看到的还是旧版本？**

GitHub Pages 有 CDN 缓存，新内容部署完到对外可见，通常要 30-60 秒。等一下再刷。

**Q: 我换了一台电脑/换了浏览器，发布按钮没出现？**

正常。token 是按浏览器存的，新设备需要去 ⚙ 设置里重新填。或者继续用导出 zip 的方式。

**Q: 我能从手机上添加卡片吗？**

可以新建，但只会存在手机浏览器的 localStorage，不会同步到仓库。建议手机只用来查看，添加卡片在电脑上做。

**Q: 卡片不小心删了能恢复吗？**

- 已发布的卡片：去仓库的 git 历史里找旧的 `data.json`，复制那条记录回来
- 本地草稿：localStorage 没有版本控制，删了就没了。所以养成习惯：积累一定数量就导出发布。

**Q: GitHub 在国内访问慢/不稳定怎么办？**

几个选择：
1. **Cloudflare Pages** —— 部署流程跟 GitHub Pages 几乎一样，但国内访问更稳。在 Cloudflare 注册账号，连接 GitHub 仓库即可。
2. **Vercel** —— 同样很方便，国内可访问。
3. **腾讯云静态网站托管** —— 国内速度最快，但要实名认证。

部署方式不影响本网页的工作方式，data.json + images/ 的结构通用。

**Q: 想批量整理已有的截图怎么办？**

直接编辑 `data.json` 文件最快。每条记录的字段：

```json
{
  "id": "唯一字符串，建议 'p001'、'p002' 这样",
  "image": "images/p001.jpg",
  "text": "原文摘录文字（可选）",
  "note": "你的笔记（可选）",
  "source": "Zhang et al., 2023",
  "page": "12",
  "link": "https://www.zotero.org/users/.../items/...",
  "tags": ["CAV", "bilevel"],
  "created": 1718438400000
}
```

`created` 是毫秒级时间戳，可以在浏览器 console 里 `Date.now()` 拿到，或者用 `new Date('2024-01-15').getTime()` 转换。

把图片放进 `images/` 文件夹，文件名跟 JSON 里的 `image` 字段对应即可。

**Q: 想加新功能（比如按年份排序、按作者分组）怎么办？**

`index.html` 是单文件，可以直接改。所有逻辑在文件末尾的 `<script>` 块里，结构很清晰，可以让我或其他 AI 帮你改。
