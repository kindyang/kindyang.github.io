# Day1 下午 · Tutorial 1：用 generate-chart 生成单张图

> 配套课件：《Day1 下午 · 数据清洗·可视化·获取》可视化章节（S14–S27）
> 目标：理解 generate-chart 的原理 → 认识它支持的图表 → 学会"针对数据选图 + 一句话出单图"
> 时长建议：20–30 分钟
> 数据：`数据集\财务舞弊单年份\附件2.csv`（4,943 行 × 363 列，标签 FLAG）

---

## 0. 本教程你能带走什么

| 你会得到 | 对应课件 |
|---|---|
| 知道 generate-chart 用什么工具画图、内部怎么工作 | S26 |
| 认识 skill 支持的 **13 种图表**，每种"什么时候用" | S16–S24 |
| 学会"选图 + 生成单图"的标准提示词写法 | S16 选图速查表 |
| 亲手看到：财务舞弊单年份数据 → 5 张真实的单图 | S27 演示 |

---

## 1. generate-chart 的原理：它用什么画图

### 1.1 一句话原理

generate-chart 的底层画图工具是 **Plotly**（Python 交互式图表库），配 `pandas`（读数据）、`numpy`（数值分析）。

```
数据文件(CSV/Excel)
      │ pandas 读取
      ▼
分析数据特征（数值列/分类列/日期列/相关性）
      │
      ▼
选择图表类型（你指定 or AI 自动推荐）
      │ plotly 绘制
      ▼
输出一张交互式 HTML 图（悬停/缩放/平移）
```

**关键点**：skill 本身只是"分析器 + 绘图脚本"；真正把数据变成图的是 **Plotly**。所以：
- 你想生成什么类型的图，由你指定 `--chart` 参数（或提示词里说清楚）；
- 图好不好看、交互、悬停信息，都由 Plotly 负责。

### 1.2 skill 装了哪些文件（安装时落在哪里）

```
generate-chart/
├── SKILL.md                      ★ 技能说明书
├── scripts/
│   ├── generate_chart.py         ★ 原版：自动分析 → 出整板仪表板
│   └── generate_single_chart.py  ★ 本教程用：按指定类型出单张图
└── references/
    ├── chart-types.md            ★ 每种图的适用场景
    └── data-format.md            ★ 输入数据格式
```

> 本教程专注 **`generate_single_chart.py`**——每次只出一张图，不拼仪表板。要哪个图、哪一列，全由你说了算。

---

## 2. 它能画哪些图（13 种，一张表记完）

| # | 图表类型 | 学名 | 课件对照 | 什么时候用（选图判断） |
|---|---|---|---|---|
| 1 | `histogram` | 直方图 | S19 | 看单个数值变量的分布形状（偏态/集中/离群） |
| 2 | `value_counts_bar` | 值计数柱状图 | S17 | 看分类变量/标签的取值分布（如 FLAG 0/1 各多少） |
| 3 | `box_single` | 单变量箱线图 | S20 | 看单个数值变量的五数概括 + 离群点 |
| 4 | `box_grouped` | 分组箱线图 | S20 | **对比**不同类别下的数值分布（如舞弊 vs 正常） |
| 5 | `violin` | 小提琴图 | S25(Orange) | 比箱线更细：分布形状 + 密度 |
| 6 | `grouped_bar` | 分组柱状图 | S18 | 不同类别下某个数值的**汇总对比**（求和/均值） |
| 7 | `pie` | 饼图 | S16 | 看占比构成（分类不多时，≤8 类） |
| 8 | `scatter` | 散点图 | S22 | 看两个数值变量的关系（正/负/无相关） |
| 9 | `line` | 折线图 | S16 | 时间/顺序趋势 |
| 10 | `area` | 面积图 | — | 趋势 + 累积量的直观展示 |
| 11 | `heatmap` | 热力图 | S23 | 相关性矩阵 / 缺失情况，一眼看大表 |
| 12 | `parallel_coordinates` | 平行坐标图 | S25(Orange) | 多变量高维比较 |
| 13 | `sunburst` | 旭日图 | S25(Orange) | 两三个分类变量的层级构成 |

> ⚠️ 这 13 种就是原版仪表板内部支持的全部类型——**单图模式把它们全部开放给你单独调用**。

---

## 3. 选图判断：先问问题，再选图（课件 S16 的 Agent 版）

课件 S16 给了口诀：**比较 → 柱状图；构成 → 堆叠图；分布 → 直方图；关系 → 散点图**。

对应到 generate-chart，就是"根据你想回答的问题，选 `--chart` 参数"：

| 你想回答的问题 | 选图 | 对应财务舞弊数据示例 |
|---|---|---|
| "舞弊样本有多少？有多不平衡？" | `value_counts_bar` | FLAG 取值分布 |
| "资产负债率分布长什么样？右偏吗？" | `histogram` | 资产负债率直方图 |
| "舞弊公司和正常公司的 ROE 差多少？" | `box_grouped` / `violin` | ROE 按 FLAG 分组 |
| "营收增速和 ROE 有没有关系？" | `scatter` | 营收增速 vs ROE |
| "这些财务指标谁和谁强相关？" | `heatmap` | 关键指标相关性热力图 |
| "行业构成怎么样？" | `pie` / `grouped_bar` | 行业占比/均值 |
| "三层分类怎么分布？" | `sunburst` | 行业→是否ST→是否舞弊 |

> 教学节奏建议：先让学生**口述问题**，再问"该选哪种图"，最后才看提示词——把课件 S16 练成肌肉记忆。

---

## 4. 实战：对财务舞弊单年份数据出 5 张单图

### 4.1 准备

- 数据：`数据集\财务舞弊单年份\附件2.csv`（utf-8-sig，4,943 × 363）
- 常用列：`FLAG`（标签 0/1）、`ASSET_LIAB_RATIO`（资产负债率）、`ROE`、`REVENUE_YOY`（营收增速）、`ROA`、`CURRENT_RATIO` 等

> 不知道有哪些列？先跑一句：`让 AI 列出附件2.csv 的所有列名`，或打开上午生成的 DATA.md 查字段说明。

### 4.2 五张图 + 提示词（直接复制可用）

**① 标签分布（value_counts_bar）——先看"不平衡"**

```text
读 财务舞弊单年份/附件2.csv，画一张 FLAG 列的取值分布柱状图，
每个取值标出数量，保存为 单图1_FLAG分布.html
```

> 对应命令：`generate_single_chart.py --chart value_counts_bar --col FLAG`

![FLAG 分布](./Day1下午_Tutorial_数据可视化/单图1_FLAG分布_截图.png)

**读出什么**：FLAG=0 约 4,914 条，FLAG=1 仅 28 条（0.57%）——**类别极度不平衡**，直接引出 Day2 的"准确率陷阱"。

---

**② 资产负债率直方图（histogram）——看分布形态**

```text
读 附件2.csv，画资产负债率(ASSET_LIAB_RATIO)的直方图，50 个区间，
标题写"资产负债率分布"，保存为 单图2.html
```

> 对应命令：`generate_single_chart.py --chart histogram --col ASSET_LIAB_RATIO --bins 50`

![资产负债率直方图](./Day1下午_Tutorial_数据可视化/单图2_资产负债率直方图_截图.png)

**读出什么**：分布是否右偏/有尖峰/有离群——对应课件 S19"先看分布再建模"、S21"右偏要 log 变换"。

---

**③ 分组箱线图（box_grouped）——对比两组分布**

```text
读 附件2.csv，画 ROE 按 FLAG 分组的分组箱线图（0 和 1 两组对比），
保存为 单图3.html
```

> 对应命令：`generate_single_chart.py --chart box_grouped --cat FLAG --col ROE`

![ROE 分组箱线图](./Day1下午_Tutorial_数据可视化/单图3_ROE分组箱线图_截图.png)

**读出什么**：舞弊组的 ROE 中位数、离群分布和正常组差多少——课件 S20 箱线图五数 + S22"用颜色装第三个变量"。

---

**④ 散点图（scatter）——看两变量关系**

```text
读 附件2.csv，画营收增速(REVENUE_YOY) 和 ROE 的散点图，
保存为 单图4.html
```

> 对应命令：`generate_single_chart.py --chart scatter --x REVENUE_YOY --y ROE`

![散点图](./Day1下午_Tutorial_数据可视化/单图4_散点图_截图.png)

**读出什么**：两个指标是正相关、负相关还是无相关——课件 S22"散点图看两变量关系，用颜色装第三变量（这里可考虑按 FLAG 着色）"。

---

**⑤ 相关性热力图（heatmap）——一眼看整张表的关系网**

```text
读 附件2.csv，选 6 个财务指标（资产负债率、流动比率、速动比率、ROE、ROA、毛利率），
画它们的相关系数热力图，每个格标出数值，保存为 单图5.html
```

> 对应命令：`generate_single_chart.py --chart heatmap --cols ASSET_LIAB_RATIO CURRENT_RATIO QUICK_RATIO ROE ROA GROSS_MARGIN`

![热力图](./Day1下午_Tutorial_数据可视化/单图5_热力图_截图.png)

**读出什么**：哪些指标 |r|>0.9 高度共线（如流动比率 vs 速动比率）——这就是 Day2 特征工程要处理的共线性，也呼应上午 DATA.md 的相关性分析。

---

## 5. 怎么控制"只出这一张"：提示词 vs 命令

**方式 A：提示词（推荐课堂演示，AI 帮你跑）**

```text
用 generate-chart 的【单图模式】读 附件2.csv，
画【<图表名>】——列用 <列名>，保存为 <文件名>.html
```

AI 收到后会自动调用 `generate_single_chart.py`，只需把图表类型、列名说清楚。

**方式 B：直接命令行（给想自己动手的学生）**

```bash
python generate_single_chart.py \
  --input 附件2.csv \
  --chart box_grouped \
  --cat FLAG --col ROE \
  --output 单图3.html
```

### 参数速查

| 参数 | 作用 | 用在哪些图 |
|---|---|---|
| `--chart` | 图表类型（13 种，见 §2） | 全部 |
| `--col` | 数值/关注列 | histogram / value_counts_bar / box_single / grouped_bar / pie / box_grouped / violin / sunburst |
| `--cat` | 分类列 | grouped_bar / pie / box_grouped / violin / sunburst |
| `--x` / `--y` | 两轴列 | line / area / scatter |
| `--cols` | 多个列（空格分隔） | heatmap / parallel_coordinates / sunburst 层级 |
| `--bins` | 直方图区间数（默认 30） | histogram |
| `--title` | 自定义标题 | 全部 |
| `--output` | 输出文件名 | 全部 |

---

## 6. 常见问题

| 问题 | 处理 |
|---|---|
| 提示词说"画图"但 AI 出了整板仪表板 | 明确加"用单图模式，只要一张"；或直接给 `--chart` 参数 |
| 提示词里列名写错了 | 先让 AI `列出列名`，再粘贴准确列名（大小写敏感） |
| 图太密/看不清 | 直方图加 `--bins` 调区间数；热力图减少 `--cols` 数量 |
| 想给散点图按 FLAG 着色 | 提示词补充："用 FLAG 作为颜色分组"（AI 会用不同颜色区分两类） |
| 中文乱码 | 数据文件保持 utf-8-sig；读文件显式写明编码（课件 S69） |

---

## 附：一句话总结

**generate-chart 用 Plotly 画图，单图模式让你"要哪张出哪张"。** 先问自己"我要回答什么问题"（比较/构成/分布/关系），再选 `--chart` 类型——选对了图，数据才会说话。