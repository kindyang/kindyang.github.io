# Day1 下午 · Tutorial 2：数据获取——用 browser-skill 爬巨潮资讯网

> 配套课件：《Day1 下午 · 数据清洗·可视化·获取》数据获取章节（S28–S72）
> 目标：装好 browser-skill → 初始化（浏览器插件）→ 用「录制轨迹」省 token → 从巨潮资讯网爬深交所公告前 5 页
> 时长建议：30–40 分钟

---

## 0. 本教程你能带走什么

| 你会得到 | 对应课件 |
|---|---|
| 知道 browser-skill 是什么、为什么比"让 AI 直接读网页"省钱 | S62–S64 |
| 亲手装好 bsk CLI + 浏览器插件，完成初始化 | S63 |
| 学会「录制轨迹 → 按轨迹重复」，理解省 token 的原理 | S63（Skill 复用思想） |
| 用精准提示词从巨潮资讯网爬取深交所公告（前 5 页） | S40–S48 / S71–S72 |

---

## 1. browser-skill 是什么 & 为什么要用它

**一句话**：让 WorkBuddy 能**操作你的真实浏览器**——打开网页、搜索、点击、填表、下载文件，且保留你已登录的会话（Cookie）。

**对比三种爬虫方式**（对应课件 S64）：

| 方式 | 适用 | 代价 |
|---|---|---|
| 直接爬 API（requests 一行） | 网站开放了数据接口 | 最省，但要先找到 API |
| 无头浏览器（Playwright/Selenium） | 能过 JS 渲染 | 环境配置麻烦 |
| **MCP + browser-skill** | 要登录、要点击、要下载 | 真实浏览器 + AI 决策 |

**本项目原则**（课件 S64）：**能走 API 就走 API，要登录就用 browser-skill。**

### 1.1 为什么「录制轨迹」能省 token（重点理解）

**场景**：让 AI 打开巨潮资讯网 → 找到搜索框 → 输入代码 → 点查询 → 点开 PDF → 下载。

- **方式 A：让 AI 自己探索网页**——它每点一步都要把整页 DOM 结构读进上下文（几十 KB～几百 KB token），一个 5 步操作可能烧掉大量 token，而且每一步都可能走偏。
- **方式 B：你（或第一次 AI）录一条轨迹**——浏览器插件把"点哪、填什么、按哪个按钮"记成一份轨迹文件；以后每次只是**播放这条轨迹**，AI 只在关键节点做判断。**省下的就是反复读网页的那部分 token。**

> 类比课件 S19 的 Skill 思想：**第一次教会它，之后调用同一套流程**——轨迹就是"操作版的 Skill"。

---

## 2. 安装与初始化

### 2.1 安装 bsk CLI（AI 的浏览器驱动）

browser-skill 的核心命令行工具叫 **bsk**（BrowserSkill）。安装方式：

```bash
# Windows / macOS / Linux 通用（官方安装脚本）
curl -fsSL https://raw.githubusercontent.com/Tencent/BrowserSkill/main/AGENT_INSTALL.md -o /tmp/bsk_install.md

# 或按官方文档用包管理器：
# Windows（PowerShell，管理员）:
#   winget install Tencent.BrowserSkill
# 或直接下载 release 的 bsk.exe 放入 PATH
```

安装后验证（本机实测路径）：

```bash
bsk --version        # 应显示 bsk 0.2.x
```

> ⚠️ 本机已装好：`C:\Users\kindy\.local\bin\bsk`（v0.2.1），且 Chrome 扩展已连接（extension v0.2.1 在线）。

### 2.2 安装浏览器插件（Chrome / Edge）

browser-skill 需要**一个配套浏览器扩展**，AI 通过它驱动真实浏览器：

1. 打开 Chrome / Edge 扩展商店，搜索 **"BrowserSkill"**（或访问官方仓库 README 里的商店链接）；
2. 点「添加扩展」，安装后浏览器右上角出现 BrowserSkill 图标；
3. 点击图标，复制扩展显示的连接码（或按提示配对）。

> 对应课件 S63 安装三步：① WorkBuddy 启用 browser-skill → ② 浏览器装配套插件 → ③ 授权连接。

### 2.3 初始化验证（一次即可）

让 AI 跑一次健康检查，确认"CLI ↔ 插件 ↔ 浏览器"链路通：

```bash
bsk doctor
```

看到各项 `ok` / `pass` 即可。也可以让 AI 执行：

```text
运行 bsk doctor，检查 browser-skill 是否连接正常，用中文汇报每项结果。
```

---

## 3. 核心能力①：直接驱动（AI 自己操作）

AI 拿到 skill 后，会自动按下面的生命周期操作浏览器（这是 skill 的标准流程，你只需在提示词里给目标）：

```text
1. bsk session start              # 开一个隔离的 Agent 窗口会话
2. bsk navigate <网址> --session <id>   # 打开网页
3. bsk observe --session <id>     # 读当前页面，拿到可操作的元素
4. bsk click|fill|select ...      # 点击/填表/选择
5. bsk screenshot --out <路径>    # 需要时截图留证
6. bsk session stop <id>          # 完成后关会话
```

**你的提示词只需要给"目标 + 约束"**，具体点哪由 AI 自己观察页面决定：

```text
用 browser-skill 打开 https://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse ，
找到"深交所"公告列表，截一张图给我看。完成后关闭会话。
```

> ⚠️ 注意：这就是"方式 A"——AI 每步都要读页面，**token 消耗高**。适合一次性、不重复的任务。

---

## 4. 核心能力②：录制轨迹，按轨迹重复（省 token 的关键）

### 4.1 什么时候用

**任务固定、要反复跑**（比如每天抓一次深交所公告）→ 先录一条轨迹，之后每次播放。

### 4.2 怎么录

让 AI 执行：

```text
启动轨迹录制：bsk record start --url "https://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse" --purpose "深交所公告列表 → 翻前5页 → 收集公告标题与PDF链接"
```

浏览器会打开**录制窗口**——**这时你（老师）亲手操作一遍**：点搜索、选板块、翻页、点开 PDF……插件把你每一步记下来，存成轨迹文件（trace.json + 页面状态）。

### 4.3 怎么重复

以后让 AI 读轨迹并按它执行：

```text
读取 trace 目录下的轨迹文件，按轨迹里的步骤在真实浏览器里重复执行，
目标是抓取深交所公告前 5 页，输出 CSV。轨迹没有覆盖到的分支再你自己决定。
```

### 4.4 为什么推荐先录（给学生的解释）

| | 让 AI 直接探索（方式 A） | 录制轨迹后重复（方式 B） |
|---|---|---|
| 每步成本 | 读整页 DOM，token 大 | 只按轨迹走，token 极小 |
| 稳定性 | 页面一变就可能走偏 | 轨迹固定，可复现 |
| 学习成本 | 每次重来 | 录一次，永久复用 |

> 对应课件 S68「从『一步步问』到『固化成流程』」：轨迹 = 操作版 Workflow。

---

## 5. 实战：从巨潮资讯网爬取深交所公告（前 5 页）

### 5.1 目标与合规边界（先讲清楚）

- **数据源**：巨潮资讯网（cninfo，证监会指定信息披露网站）
- **URL**：`https://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse`（深交所公告列表）
- **本次范围**：只爬**前 5 页**（每页 30 条 → 约 150 条公告的标题 + 公告时间 + PDF 链接）
- **合规底线**（课件 S57 / S71）：
  - ✅ 公开披露信息，可访问即可获取
  - ✅ 控制频率（每页间停顿 3–6 秒，别把网站搞崩——记住 429）
  - ✅ 标注来源（数据来源、获取时间、使用条款）
  - ❌ 不得用于商业转售；教学演示用途需在文档中声明
  - ❌ 不爬取 robots.txt 明确禁止的路径（巨潮无 robots 限制，但依然礼貌爬取）

### 5.2 提示词是怎么构建的（结合 PPT 知识）

**课件里讲过的知识点，这里全用上了**：

| 提示词里的约束 | 用到的课件知识 |
|---|---|
| "前 5 页，每页 30 条" | S72：产出物有明确范围 |
| "每页间隔 3–6 秒随机停顿" | S53：礼貌爬虫 time.sleep(random) |
| "带上浏览器的 User-Agent" | S43 / S51：伪装成浏览器，否则被识别为爬虫 |
| "若返回 429/403，降低频率重试" | S45 状态码 + S58 反爬应对 |
| "记录数据来源 + 获取时间 + 使用条款" | S72：带出处的数据表 |
| "不要爬 robots.txt 禁止的路径" | S57 / S71：合规与公开边界 |
| "公告 PDF 只取链接，先不下载全部正文" | S64：能走 API 就走 API（可先探测接口） |

### 5.3 完整提示词（直接复制可用）

**路线一：先探测 API（最省 token，推荐先试）**

巨潮资讯网其实有个公开查询接口（实测可用）：`https://www.cninfo.com.cn/new/hisAnnouncement/query`

```text
用 Python 的 requests 调巨潮资讯网的公告查询接口：
POST https://www.cninfo.com.cn/new/hisAnnouncement/query

参数（深交所公告，前 5 页，每页 30 条）：
pageNum=1..5
pageSize=30
column=szse          ← 深交所
tabName=fulltext
plate=
seDate=（今天日期，或留空）

合规约束：
1. 请求头带上浏览器 User-Agent（伪装成浏览器，见课件 S43/S51）
2. 每请求一页后随机 sleep 3~6 秒，控制频率（见 S53）
3. 若返回 429/403，退避重试，最多 3 次，别硬刚（见 S45/S58）
4. 只取 announcements 里的：secCode、secName、announcementTitle、announcementTime、adjunctUrl（PDF 链接）

输出：合并 5 页为一张 CSV 表，列名：
股票代码, 股票简称, 公告标题, 公告时间, PDF链接, 数据来源=巨潮资讯网, 获取时间=<今天>, 使用条款=仅教学演示

先给我前 5 页的 JSON 预览（第 1 页即可），确认字段后我再放行全量抓取。
```

> 这个接口实测返回 `totalAnnouncement: 342,392`，字段完整——**先走 API 是课件 S64 的"本项目原则"**。

**路线二：必须用 browser-skill（登录态/下载 PDF 时）**

```text
用 browser-skill 打开巨潮资讯网深交所公告列表：
https://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse

任务：
1. 确认列表加载完成，读取第 1 页公告（标题、日期）
2. 翻页到第 5 页为止，共收集 5 页公告的：标题、日期、详情链接
3. 打开其中 1 条公告详情，确认 PDF 原文可下载，下载到 ./数据/巨潮公告/ 下做示例
4. 不要爬第 5 页之后的内容（本次范围仅前 5 页）

合规约束：
· 每翻一页停顿 3~6 秒（随机），控制请求频率
· 若出现验证码/429，停下来告诉我，不要自动重试轰炸
· 全程只读公开披露信息，不登录、不提交表单、不下载非公开文件
· 结束后 bsk session stop，汇报：抓取条数、CSV 路径、示例 PDF 路径、遇到的阻碍
```

### 5.4 输出物长什么样

```csv
股票代码,股票简称,公告标题,公告时间,PDF链接,数据来源,获取时间,使用条款
300723,一品红,关于全资子公司获得帕拉米韦注射液注册证书的公告,2026-10-16,finalpage/2026-10-16/1225607812.PDF,巨潮资讯网,2026-10-16,仅教学演示
...
```

> 对应课件 S72「产出：一份带出处的数据表」——**八字段 + 来源/时间/条款三列**，这份表就是后面建模的直接输入。

---

## 6. 三个必然会遇到的故障（课件 S66–S69 对照）

| 故障 | 现象 | 对策 |
|---|---|---|
| 页面是空的 | JS 渲染，requests 拿不到内容 | 找背后 API（§5.3 路线一），或上 browser-skill |
| 429 / 验证码 | 频率过高触发反爬 | 停手，等 30 秒，降频重试；让 AI 把报错整段贴给你判断 |
| 编码炸了 | 中文乱码 | 读文件显式写 `encoding='utf-8'`，CSV 用 utf-8-sig |

> 报错处理口诀（课件 S66）：**把报错信息整段贴给 AI**，别只贴"它报错了"。

---

## 附：一句话总结

**browser-skill = 给 AI 一双操作真实浏览器的"手"；录制轨迹 = 把第一次的操作固化成可重复的流程，省下反复读网页的 token。** 提示词里把课件讲的"范围、频率、UA、合规、出处"写进去，AI 就不敢乱来。
