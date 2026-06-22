# CLAUDE.md

这份文件是给后续在 Claude Code 里维护本项目的 AI 助手看的。请先读完再动手。

---

## 1. 项目本质

这是一个**个人文献片段回忆墙**：把读论文时截下的图、表、关键段落，连同元数据（来源、页码、笔记、跳转链接）展示成 Pinterest 式的瀑布流，任何设备打开浏览器都能看，点击图片可以跳回原文献。

**它是什么**：一个纯静态网页 + 一份 JSON 数据 + 一个图片文件夹。
**它不是什么**：不是 Web App、不是 CMS、不是协作工具、没有后端、没有数据库。

维护这个项目时，**优先保持简单**，不要引入框架、构建工具、Node 依赖、后端服务。

---

## 2. Claude Code 实操：从零到上线

这一节是为"现在打开 Claude Code，想把这个项目从零跑起来"的场景写的。按 A → G 顺序执行，整体大约 30-40 分钟。

**符号约定**：
- 🤖 = Claude Code 可以直接用 bash 工具执行的命令
- 👤 = 用户需要自己在浏览器或图形界面里操作的步骤

Claude Code 在带用户走流程时，要在每一阶段开头明确说"接下来我会执行 X / 接下来需要你去做 Y"，避免用户不清楚谁该做什么。

---

### 阶段 A：前置检查

🤖 检查环境：
```bash
git --version
gh --version  || echo "gh 未安装"
python3 --version || echo "python3 未安装"
```

判断标准：
- `git`：**必需**。没有的话用户自己装（macOS 自带；Windows 装 Git for Windows；Linux 用包管理器）。
- `gh`（GitHub CLI）：**强烈推荐**，能让阶段 C/D 一行命令完成。没装的话提示：
  - macOS: `brew install gh`
  - Windows: `winget install --id GitHub.cli`
  - Linux: 见 https://github.com/cli/cli/blob/trunk/docs/install_linux.md
  - 装完执行 `gh auth login`，按提示在浏览器授权
- `python3`：**可选**，用来跑本地预览。没有也可以用 `npx serve` 或别的静态服务器。

---

### 阶段 B：确认四个文件齐全

🤖 ：
```bash
ls -la
```

期望看到这四个文件：
- `index.html` （约 50KB，核心网页）
- `data.json` （`[]` 一个空数组）
- `README.md`
- `CLAUDE.md`（本文件）

如果有缺失：用户应该回到 Claude 对话里把全套文件下载到当前目录。`index.html` 是最关键的，没有它整个项目就空了。

---

### 阶段 C：本地预览验证

启动一个静态文件服务器（**不能直接双击打开 index.html**，因为 `fetch('./data.json')` 在 `file://` 协议下被浏览器拦截）：

🤖 ：
```bash
python3 -m http.server 8000
```
（或 `npx serve -p 8000 .`）

👤 在浏览器打开 http://localhost:8000

期望看到一片空白的"文献摘录墙"，顶部 badge 显示 "0 张卡片"。

让用户做一个最小验证：直接在网页里 Ctrl+V 粘贴任意图片 → 填一两个字段 → 保存。卡片应该出现，左侧橘色虚线（草稿），顶部红色提示条出现。这一步通过，说明前端逻辑没问题。

确认完 🤖 关掉服务器（`Ctrl+C` 或 `kill %1`）。如果用户保存了测试卡片想保留，让他们去阶段 F 再发布。

---

### 阶段 D：连接 GitHub 并推送

让用户先告诉你两个值：
- **GitHub 用户名**（如 `caiyindong`）
- **仓库名**（如 `literature-wall`）

🤖 初始化本地仓库：
```bash
git init -b main
git add .
git commit -m "initial commit: literature wall scaffold"
```

#### 路径 1：装了 gh CLI 且 `gh auth status` 显示已登录

🤖 一条命令搞定建仓库 + 关联 remote + push：
```bash
gh repo create <用户名>/<仓库名> --public --source=. --remote=origin --push
```

完成后：
```bash
gh repo view --web
```
👤 在打开的网页上确认文件已经上去了（应该能看到 index.html、data.json 等）。

#### 路径 2：没有 gh，走纯 git

👤 去 https://github.com/new 建仓库：
- Repository name：填刚才约定的仓库名
- Public（**必须公开**，GitHub Pages 免费版要求）
- **不要**勾任何初始化选项（README/.gitignore/license 一个都别勾）
- Create repository

🤖 关联 remote 并 push：
```bash
git remote add origin https://github.com/<用户名>/<仓库名>.git
git push -u origin main
```

第一次 push 时 GitHub 会让用户登录。**密码不能用账户密码**，必须用 Personal Access Token（PAT）当密码。如果用户还没有 PAT，先跳到阶段 F 生成，回来用那个 token 完成 push。

---

### 阶段 E：开启 GitHub Pages

👤 在仓库网页：
1. Settings → Pages（左侧菜单）
2. Source 选 **Deploy from a branch**
3. Branch 选 **main**，文件夹选 **/ (root)**
4. 点 **Save**

等 30-60 秒，刷新这个 Pages 设置页，顶部会出现绿色横幅：

> ✓ Your site is live at https://<用户名>.github.io/<仓库名>/

让用户把这个 URL 收藏。这就是日后任意设备访问的入口。

🤖 自动验证（可选）：
```bash
sleep 60 && curl -sI https://<用户名>.github.io/<仓库名>/ | head -1
```
看到 `HTTP/2 200` 就部署成功了。如果是 404，等一会儿再试，Pages 第一次部署有时候要 1-2 分钟。

---

### 阶段 F：生成 PAT 并配置"一键发布"

这是为了让用户在网页里点 🚀 一键发布按钮能直接 commit 到仓库。

👤 在浏览器：

1. 打开 https://github.com/settings/personal-access-tokens
2. 点 **Generate new token** → **Generate new token (Fine-grained)**
3. 字段填写：
   - **Token name**：`literature-wall-publish`
   - **Expiration**：建议 90 days（到期重新生成一个即可）
   - **Repository access**：选 **Only select repositories**，下面下拉框选刚建的那个仓库
   - **Repository permissions** 展开：找到 **Contents**，下拉选 **Read and write**（一定要 write）
4. 滚到底，点 **Generate token**
5. 出现的 token（开头 `github_pat_`）**立刻复制保存**——离开页面后就再也看不到

👤 回到刚才那个 Pages URL 打开的网页：
1. 点右上角 ⚙ 按钮
2. 填 GitHub 用户名、仓库名、分支（默认 main）
3. 粘贴刚才那个 token
4. 点"测试连接"
5. 看到绿色 `✓ 连接成功，有写入权限` → 点保存

如果测试失败，根据错误信息排查：
- `401 Unauthorized`：token 复制错了或过期
- `403 Forbidden`：token 没勾 Contents: Read and write
- `404 Not Found`：用户名或仓库名拼写错了
- 网络错误：检查网络是否能访问 api.github.com

---

### 阶段 G：端到端验证

👤 在 Pages URL 的网页上：
1. 点右上角 + 新建
2. 粘贴一张测试图 + 填几个字段
3. 保存（出现橘色虚线草稿 + 红色顶部提示条）
4. 点右上角 🚀 一键发布
5. 进度条跑完，看到 toast `✓ 已发布 1 张到 GitHub`
6. 等 30-60 秒（GitHub Pages CDN 部署）
7. 刷新页面
8. 那张卡片橘色虚线应该消失了（变成"已发布"）

🤖 验证仓库已收到：
```bash
git pull
ls images/
cat data.json | python3 -m json.tool | head -20
```

应该看到一个新的图片文件和 JSON 里多了一条记录。

走到这里整个系统就跑通了。

---

### 后续日常运维

用户的常规动作只有两个：
1. **加卡片**：网页 Ctrl+V 截图 → 填表 → 保存（自动进 localStorage 草稿）
2. **攒一批发布**：点 🚀 → 等部署 → 完成

跨设备约定：
- **主设备（电脑）**：填好 PAT，负责所有"加 + 发布"
- **移动设备（手机/平板）**：只查看，不要在上面加卡片（草稿是按浏览器隔离的，不会同步）

#### 换电脑或浏览器后的恢复流程

PAT 是按浏览器存的，新设备没有。两个选择：
- 在新设备的 ⚙ 设置里重新填一次 PAT（同一个 PAT 复制粘贴即可，不用重新生成）
- 或者继续用导出 zip 方式手动 push

#### 用 Claude Code 在本地批量改动 data.json 或图片

如果用户想批量整理（比如改文件名、按主题分组、批量加标签），Claude Code 可以直接在本地：
```bash
git pull origin main
# ... 修改 data.json 或 images/ ...
git add . && git commit -m "..." && git push
```
推完同样等 30-60 秒，GitHub Pages 重新部署。

注意：如果用户当前在浏览器里有本地草稿（localStorage 里），先让他们点 🚀 把草稿发布出去再 git pull，不然会有合并冲突。

#### PAT 过期续期

90 天到期前，GitHub 会给用户发邮件提示。续期流程：
1. 回到 https://github.com/settings/personal-access-tokens 找到那个 token
2. 点 Regenerate token，确认延期
3. 复制新 token，回到网页 ⚙ 里替换 token 字段
4. 测试连接确认

---

## 3. 文件结构

```
.
├── index.html          # 单文件应用，所有 UI、CSS、JS 都在这里
├── data.json           # 已发布卡片的元数据（项目的"真理之源"）
├── images/             # 已发布的截图文件
│   ├── zhang2023-fig3.jpg
│   ├── li2024-tab2.png
│   └── ...
├── README.md           # 给人看的部署和使用说明
└── CLAUDE.md           # 本文件
```

部署在 GitHub Pages（或 Cloudflare Pages / Vercel）。

---

## 4. 数据架构（关键概念）

这个项目有一个核心架构，理解它之后其他都顺：

**两个数据来源，但只有一个真理之源。**

| 来源 | 角色 | 在哪 | 是否跨设备 |
|------|------|------|------------|
| `data.json` | **权威源**，已发布的卡片 | 仓库根目录 | 是 |
| `localStorage['litCards']` | **草稿区**，未同步的新卡片 | 浏览器本地 | 否 |

**网页启动时的合并逻辑**（`loadCardsFromSources` 函数）：

1. 尝试 `fetch('./data.json')`，拿到 `serverCards`
2. 读 `localStorage`，拿到所有本地卡片
3. **如果 localStorage 里某条卡片的 id 已经出现在 data.json 里，就把它从 localStorage 删掉**（认为已发布成功）
4. 剩下的本地卡片成为 `localDrafts`
5. 渲染时把 `serverCards` 和 `localDrafts` 拼起来显示

**用户的"发布"流程**有两条路径：

**路径 A — 一键发布**（用户在网页 ⚙ 设置里配好 GitHub PAT 之后）：
1. 用户点"🚀 一键发布"
2. 网页用 GitHub Contents API 把所有 base64 图片 PUT 到 `images/` 文件夹
3. 然后 PUT 更新后的 `data.json`
4. 上传完成后，本地直接更新 `serverCards` 内存状态、清空 localStorage 草稿、重渲染
5. GitHub Pages CDN 在 30-60 秒后部署完毕，其他设备能看到

**路径 B — 导出 zip**（不需要 PAT，永远可用的备用方案）：
1. 用户点"📥 导出 zip" → 生成 zip（含 `data.json` 和 `images/`）
2. 用户解压、覆盖到仓库、git push
3. 下次打开网页，data.json 里有了这些卡片 → 草稿自动消失

**这套逻辑的不变量**（不要破坏）：

- 服务器版本优先于本地草稿
- 只有"本地草稿"卡片可以编辑/删除；已发布的没有编辑按钮（强制走 git 流程，避免改了不发布造成的鬼影）
- file:// 协议下 fetch 会失败，要静默 fallback 到纯 localStorage 模式

---

## 5. 卡片数据 Schema

```typescript
type Card = {
  id: string;          // 唯一字符串，不能含空格 / \ ? # &
  image?: string;      // 相对路径 'images/xxx.jpg'，或 base64 dataURL（仅 localStorage）
  text?: string;       // 原文摘录文字
  note?: string;       // 个人笔记
  source?: string;     // 文献署名，如 "Zhang et al., 2023"
  page?: string;       // 页码，如 "12" 或 "P.12-15"
  link?: string;       // 跳转链接，推荐 Zotero URL
  tags?: string[];     // 标签数组
  created: number;     // 毫秒时间戳，Date.now()
}
```

**特别注意 `image` 字段的两种形态**：

- 在 `data.json` 里：**永远是相对路径**（如 `"images/zhang2023-fig3.jpg"`）
- 在 localStorage 里：**可能是 base64 dataURL**（用户刚粘贴截图时的形态）
- 导出 zip 的逻辑（`exportPackage` 函数）会自动把 base64 拆成文件、改成路径

如果你写代码处理 `image` 字段，记得判断 `card.image.startsWith('data:')` 来区分。

---

## 6. 命名约定

### 卡片 ID

- **网页自动生成的草稿**：`d{timestamp}`，如 `d1718438400000`（`d` 前缀表示 draft）
- **手动批量录入时推荐格式**：`{firstauthor}{year}-{type}-{index}`
  - `zhang2023-fig3`
  - `li2024-tab2`
  - `wang2022-eq14`
- ID 一旦发布到 data.json 就不要改（会破坏图片关联，且其他设备的浏览器历史可能引用旧 id）

### 图片文件名

- 推荐：和 ID 同名，加扩展名。例如卡片 id 是 `zhang2023-fig3`，图片就是 `images/zhang2023-fig3.jpg`
- 扩展名小写（`.jpg` 不是 `.JPG`，`jpeg` 统一改成 `jpg`）
- 导出 zip 时网页会自动按这个规则命名

### 标签

- 全小写或全中文，二选一保持一致
- 多词标签用连字符，不要空格：`mixed-traffic` 而非 `mixed traffic`

---

## 7. AI 助手在做修改时遵守的规则

### 必须遵守

- **不要把 localStorage 换成别的存储**（IndexedDB、cookies、SQLite Wasm 等）。保持简单。
- **不要破坏 file:// 兼容性**。`fetch('./data.json')` 失败必须能优雅降级到纯 localStorage 模式。
- **修改 CSS 时使用 `:root` 里定义好的 CSS 变量**，不要硬编码颜色。如果确实需要新颜色，加到 `:root` 里。
- **不要给 published 卡片加编辑/删除按钮**。这是有意的——强制走 git 流程才能改已发布内容。
- **不要把项目改成需要 npm install / 构建步骤的形态**。单文件原则。
- **不要把 JSZip CDN 换成本地文件或别的库**。CDN URL 已固定，依赖跨设备可达。
- **绝对不要把用户的 PAT 写进日志、console.log、或往用户没明确发起的请求里塞**。PAT 存在 `localStorage['litWallConfig']` 里，只在调用 GitHub API 时通过 Authorization header 用。任何记录到 DOM、console、外部服务的代码都是泄漏。
- **不要替换 PAT 方案为 OAuth 后端**。OAuth 需要服务器、回调路由、token 存储，与本项目"零运维"原则冲突。PAT + Contents API 是有意为之的方案。

### 修改 HTML 时

- 启动顺序是 `loadCardsFromSources()` → `renderCards()`，不要反过来调
- `getMergedCards()` 是唯一的"我现在要显示哪些卡片"入口，不要在外面重新拼数组
- `escHtml` 和 `escAttr` 是模板字符串里的转义函数，往 `cardHTML` 里拼用户数据时务必走它们

### 文案语言

- 用户工作语言是中文，所有 UI 文案用中文
- 关键术语保持一致：**草稿** / **已发布** / **导出** / **同步** / **跳转原文**

---

## 8. 常见维护任务速查

### 用户说"我有 30 张已经截好的图，能不能批量加进去"

1. 收集所有图片，重命名为 `{某个有意义的 id}.jpg`，放进 `images/`
2. 让用户提供每张图的元数据（来源、页码、笔记、链接）
3. 编辑 `data.json`，按 schema 加 30 条记录
4. `git add . && git commit && git push`

不要让用户一张张通过网页 UI 加——批量编辑 JSON 快得多。

### 用户说"想按项目分组"

加一个可选字段 `project: string` 到 schema，然后在 toolbar 加一个项目筛选器。CSS 变量已经有了，按现有 `.filter-btn` 样式加即可。

### 用户说"卡片太多，加载慢"

- 第一步：检查是不是 data.json 里的图片还是 base64（如果是，说明早期版本遗留，需要拆出来）
- 第二步：给 `<img>` 加 `loading="lazy"` 属性
- 第三步：仍然慢的话，考虑虚拟滚动，但通常 < 1000 张卡片不用

### 用户说"我想给每张卡片加创建者/我自己的评分"

加字段到 schema，加输入控件到 modal，加显示元素到 `cardHTML`。注意：data.json 是数组，新字段只对新卡片生效，老卡片对应字段是 `undefined`，渲染时要处理。

### 用户说"导出按钮不工作"

- 看浏览器 Console
- 99% 是 JSZip CDN 没加载（网络问题）
- 备选 CDN：`https://unpkg.com/jszip@3.10.1/dist/jszip.min.js`

---

## 9. Zotero 集成约定

用户主要用 **Zotero** 管理文献，跳转链接字段统一这个格式：

```
https://www.zotero.org/users/{userID}/items/{itemKey}
```

- userID 在 Zotero 设置 → Feeds/API 里能找到
- itemKey 在 Zotero 桌面端右键条目 → 查看在线，URL 末尾 8 位

这个链接的好处：装了 Zotero 桌面端的设备点击直接打开本地 Zotero；没装的设备打开 Zotero 网页版。

**未来可能的进阶**（用户问起再做）：通过 Zotero Web API 自动拉取所有 area annotations，免去手动复制图片。需要：
- 用户提供 Zotero API key（写在某处配置）
- 调用 `https://api.zotero.org/users/{userID}/items?itemType=annotation`
- 区域标注的图片需要单独 fetch 二进制
- 这会让网页变重，需要慎重权衡

---

## 10. 调试速查

| 症状 | 可能原因 | 验证方法 |
|------|---------|---------|
| 卡片标题显示"载入中…"不变 | data.json 加载失败 | F12 → Network 看 data.json 请求状态 |
| 我新加的卡片刷新后没了 | 没导出/没 push/没等 GitHub Pages 重新部署 | 检查仓库 data.json 内容；等 30-60 秒 |
| 草稿一直不消失 | data.json 里 id 不匹配 | 比对 data.json 里的 id 和 localStorage 里的 id |
| 图片裂图 | images/ 里没有这个文件 / 路径写错 | F12 → Network 看 404 的图片名 |
| 导出 zip 是空的 | 没有卡片或 JSZip 没加载 | 看 Console |
| 手机上看到的卡片比电脑少 | 本地草稿是设备隔离的，手机的草稿在手机自己的 localStorage | 在电脑上完成发布流程 |
| 一键发布按钮不出现 | 没配置 PAT 或没草稿 | 去 ⚙ 设置看 token 是否填了；看 localDrafts.length |
| 一键发布报 401 | PAT 错或过期 | 重新生成 fine-grained PAT |
| 一键发布报 403 | PAT 没有 Contents: write 权限 | 重新生成时勾上 |
| 一键发布报 422 | 文件已存在但缺 sha | `putFile` 函数应该自动处理；如果没处理说明上游 API 返回的状态码变了 |

调试时优先开 F12 Console 和 Network。

---

## 11. 已经被否决的方向

记录一下用户已经讨论过、明确不要的方向，避免再次提议：

- ❌ **后端服务（Firebase、Supabase、自建 API 等）**：保持零运维成本
- ❌ **OAuth 授权流程**：需要服务器中转，与本项目静态部署原则冲突
- ❌ **复杂构建（Vue/React/Vite）**：单文件优于工程化
- ❌ **本地 SQLite / IndexedDB**：localStorage 已经够用
- ❌ **要求 npm install 的工具链**：跨设备无障碍是核心目标
- ❌ **多用户/协作功能**：这是个人工具
- ✅ **PAT + GitHub Contents API 直接发布**：已实现，是项目的一部分，保留

---

## 12. 当用户的需求超出本项目范畴

如果用户开始要求：实时协作、复杂搜索（向量检索）、PDF 全文索引、AI 自动打标签等——这些都已经不在"个人文献回忆墙"的范畴内了，应该礼貌提醒：

> 这个需求可能更适合换成 Zotero 自身的插件，或者搭一个有后端的应用。我们这个项目刻意保持简单，加这个功能会破坏它的可维护性。要不要先评估一下其他方案？

不要为了帮忙就把项目搞复杂。
