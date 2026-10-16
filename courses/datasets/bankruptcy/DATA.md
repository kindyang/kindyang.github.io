# 公司破产预测数据集 — DATA.md

## 1. 数据来源 / 时间范围
- 来源：台湾《台湾经济杂志》，1999–2009 年上市公司财务数据。
- 破产标签依据台湾证券交易所业务规定定义。
- 提供者：Deron Liang、Chih-Fong Tsai（国立中央大学管理学院）。
- 关联论文：Liang et al. (2016), *European Journal of Operational Research* 252(2):561–572。
- 文件：`bankruptcy_data.csv`，形状 **6819 行 × 96 列**（1 标签 + 95 财务比率）。

## 2. 标签含义与分布（`Bankrupt?`，第 1 列）
| 取值 | 含义 | 样本数 | 占比 |
|---|---|---|---|
| 0 | 未破产 | 6599 | 96.77% |
| 1 | 破产 | 220 | 3.23% |

**类别极度不平衡**（约 1:30），需用分层抽样、重采样或 class_weight 处理。

## 3. 字段中文含义分组（95 个特征，X1–X95）
- **盈利能力 (X1–X10, 86–90)**：税前/税后 ROA、毛利率、营业利润率、净利率、持续利率、净收入/总资产、股东权益净收入。
- **费用与现金流率 (X11–X13, 75–83)**：营业/研发费用率、现金流量率、销售现金流、负债/权益/资产现金流、CFO to Assets。
- **资本结构/偿债 (X14–X18, 36–44, 85, 91–95)**：有息债务率、每股净值、税率、资产负债率、权益/资产、借款依赖、财务杠杆 DFL、利息覆盖率、负债资产标志。
- **成长率 (X24–X31)**：销售毛利、营业利润、税后/常规/持续净利、总资产、净值、资产回报成长。
- **周转与效率 (X45–X53, 71–74)**：总资产/应收/存货/固定资产/净值周转、平均收款天数、人均收入/营业利润/固定资产。
- **流动性 (X32–X35, 54–67, 84)**：现金再投资、流动比、速动比、营运资本占比、现金/流动资产对负债、流动负债结构。
- **其他 (X68–X70, 87–89, 93)**：留存收益/总资产、总收入/总费用、费用/资产、资产与 GNP 比、No-credit interval。

> 完整逐字段释义见 `说明书.txt`（X1–X95 一一对应）。

## 4. 质量陷阱
- **极度类别不平衡**：正类仅 3.23%，准确率会失真，须看 AUC/召回/F1。
- **常量列**：`one if net income was negative for the last two year zero otherwise` 唯一值 ≤1，建模前删除。
- **无缺失**：0 缺失单元、0 缺失列，无需插补。
- **dtype**：93 列 float64、3 列 int64（含标签与两个 0/1 标志列）。
- **列名含前导空格**：原始表头有空格（如 `' Bankrupt?'`），须 strip 后使用，本仓库脚本已处理。
- **财务比率离群值**：多数为比率，长尾明显，建议按需 winsorize/标准化；对线性模型建议检查共线性（多个 ROA 变体高度相关）。
- **可能的数据泄漏面**：多个比率由同一报表口径派生（ROA A/B/C、负债率与净值/资产互为补数），注意特征冗余。

## 5. 与标签相关性 |abs| Top 10（Pearson，负号=越破产越低）
| 排名 | 字段 | 相关系数 |
|---|---|---|
| 1 | net income to total assets | -0.3155 |
| 2 | ROA(A) before interest and % after tax | -0.2829 |
| 3 | ROA(B) ... depreciation after tax | -0.2731 |
| 4 | ROA(C) ... depreciation before interest | -0.2608 |
| 5 | debt ratio % | +0.2502 |
| 6 | net worth/assets | -0.2502 |
| 7 | Persistent EPS in the Last Four Seasons | -0.2196 |
| 8 | Retained Earnings/Total assets | -0.2178 |
| 9 | net profit before tax/paid-in capital | -0.2079 |
| 10 | Per Share Net profit before tax (yuan) | -0.2014 |

**解读**：盈利能力（净收入/总资产、ROA 系列）与留存收益是最强单变量信号，负债率与破产正相关（与净值/资产严格互补）。

## 6. 复现
```bash
"C://ProgramData//anaconda3//python.exe" explore_bankrupt.py
```
