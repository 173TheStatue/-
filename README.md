# 信息系统分析与设计 · Foxcore Retail 展会零售与销售追踪系统

一份 **88 页** 的系统分析与设计报告，按 RUP 四阶段（业务建模 → 系统需求 → 系统分析 → 系统设计）逐层推演，最终落到 SQL Server 的物理数据模型与 T-SQL 脚本。

案例取自 Foxcore Retail——一家采用 Carnie（快闪）模式在多场展会间游击经营的零售团队。

> **本仓库目前只有设计文档，不含系统实现代码。**

| 篇幅 | 图 | 表 | 业务用例 | 系统用例 | 数据库 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 88 页 | 42 张 UML 图 | 26 张 | 9 个（BUC01–BUC09） | 10 个（UC01–UC10，6 个附完整规约） | 7 表 / 3 视图 / 1 函数 / 1 触发器 |

所有图都由 `puml/` 下的 PlantUML 源码生成，不是截图拼贴；所有 UML 图共用一套黑白线框样式。

## 案例与改进目标

17 名销售顾问、28 个 SKU、单周末最多并发展开 3 场跨区域展会。销售在纸质便签上记账，展后由合伙人深夜把数百张褶皱便签手工转录进多个离散 Excel，单场对账平均耗时 3.5 小时，佣金核算争议率约 12%，且一线可以在不被追踪的情况下击穿最低限价成交。

报告把这条业务链拆成可度量的指标，再倒推系统能力：

| 度量指标 | 手工基线 | 目标值 |
| :--- | :---: | :---: |
| 单场展会结业 → 佣金核算确认周期（首要指标） | 3.5 h | ≤ 0.25 h |
| 佣金核算争议率 | 12% | < 0.5% |
| 跨展会 / 跨品类经营分析报表编制周期 | ≈ 16 h（2 个工作日） | < 10 min |
| 低于最低限价的违规成交 | 时有发生 | 0（事前拦截） |

## 文档结构

| 章节 | 主要内容 | 关键产物 |
| :--- | :--- | :--- |
| **第一章 业务建模** | 业务事实与假设登记、涉众权力-利益矩阵与「老大」判定、愿景指标推导模型、9 个业务用例、现状与改进流程、3 个生命周期状态机、领域类图与 Barker ERD、业务规则矩阵、业务→系统追踪矩阵 | 图 1-1 ~ 1-18 |
| **第二章 系统需求** | 系统执行者与上下文边界、功能需求优先级评定、10 个系统用例、6 份用例规约（UC04/06/07/08/09/10）、操作权限矩阵、URPS 非功能需求、外部约束 | 图 2-1 ~ 2-3 |
| **第三章 系统分析** | 四色原型建模、充血模型与 BCE 分工、分析类图与操作契约、4 张分析序列图（含价格击穿异常路径）、实体状态机与操作合法性矩阵、对象图实例验证 | 图 3-1 ~ 3-11 |
| **第四章 系统设计** | 物理部署图与组件图、设计类图、事务控制时序图、概念模型 → 关系模式映射与 3NF 无损分解证明、7 张表数据字典、T-SQL 建库脚本、佣金对账视图与排班冲突检测函数、RBAC 与审计触发器、备份恢复方案 | 图 4-1 ~ 4-5、`tools/sqlserver_ddl.sql` |

完整 PDF：**`foxcore-report.pdf`**。

## 仓库结构

```
.
├── foxcore-report.pdf      # 成品（88 页）
├── foxcore-report.tex      # 由 src/*.md 生成，不要手改
├── src/                    # 正文源文件 ← 改内容在这里
│   ├── 第一章.md
│   ├── 第二章.md
│   ├── 第三章.md
│   └── 第四章.md
├── puml/                   # 42 张图的 PlantUML 源码
│   └── _common_*.puml      # 公共皮肤：黑白色板、线宽、字体、图标精灵
├── figs/                   # 出图结果（300 dpi PNG）
├── assets/icons/           # 业务工人 / 业务实体 / 执行者图标（512 px 源图）
├── tools/
│   ├── convert.py          # Markdown → LaTeX（表格、公式、图、代码块、追踪矩阵）
│   ├── template.tex        # 撞色学术风 LaTeX 模板
│   ├── render_figs.py      # puml → SVG → 拉直边 / 换高清图标 → Edge 无头截图
│   ├── sqlserver_ddl.sql   # T-SQL 建库脚本
│   ├── radar.py            # 图 2-3 URPS 雷达图
│   ├── qc.py               # 版面质检（空白页 / 触底 / 越界 / 缺图）
│   ├── make_icons.py       # 图标精灵生成
│   └── sqlserver_fix.py    # 早期 MySQL → SQL Server 文本迁移（一次性，留档）
├── render_all.sh           # 一键出图
└── plantuml.jar            # PlantUML 1.2024.7（随仓库提供，免下载）
```

## 构建

依赖：

- **TeX Live 2026 / XeLaTeX**：`ctex`、`unicode-math`（Latin Modern Math）、`tcolorbox`、`xltabular`、`titlesec` 等
- **Python 3.11+**：`Pillow`、`matplotlib`、`pdfplumber`
- **Java 8+**：跑 `plantuml.jar`（已随仓库提供）
- **Microsoft Edge**：无头截图，把 SVG 栅格化为 PNG

```bash
# 1) 出图（figs/ 已提交，只有改了 puml/ 才需要重跑）
python tools/render_figs.py              # 全部
python tools/render_figs.py fig1-0       # 按文件名前缀

# 2) Markdown → LaTeX
python tools/convert.py

# 3) 编译（跑两遍，第二遍才出目录与交叉引用）
xelatex foxcore-report.tex
xelatex foxcore-report.tex
```

Java 或 Edge 不在默认安装位置时，用环境变量指定：`JAVA_EXE=... EDGE_EXE=...`。

## 改哪里

| 想改 | 动这个文件 |
| :--- | :--- |
| 正文文字、表格、公式 | `src/第X章.md`，改完跑 `convert.py` 再编译 |
| 图的形状与内容 | `puml/*.puml`（公共样式在 `puml/_common_*.puml`），改完跑 `render_figs.py` |
| 封面署名、副标题 | `tools/convert.py` 里的 `AuthorName` / `AuthorID` / `AuthorClass` 等字段 |
| 建库脚本 | `tools/sqlserver_ddl.sql` |

`foxcore-report.tex` 每次都由脚本整份重写，手改会丢失。

## 几个值得说明的实现细节

**1. 为什么图里的线是直的**

`skinparam linetype` 只有 `ortho`、`polyline`、`spline` 三个合法值，**没有 `straight`**——写 `straight` 时 PlantUML 既不报错也不生效。`ortho` 会把所有边强制掰成直角，标签被挤到线上；`polyline` 只是把布局引擎为绕开节点而产生的折点连成折线，反而更丑。真正可行的做法是在矢量层动手：PlantUML 导出的每条边都是一条独立的 `<path d="M…C…">`，箭头是多边形，所以把 `d` 改写成 `M…L…` 即可得到直线边，箭头不受影响。`render_figs.py` 做的就是这件事。

**2. 图标为什么曾经是糊的**

PlantUML 会把精灵重新栅格化到显示尺寸（约 67 px）再放大绘制，无论原始精灵多大都会糊。渲染时把 SVG 里每张嵌入位图按墨迹匹配换回 `assets/icons` 的 512 px 源图，圆环边缘才干净。

**3. 中文长标识符断行**

`venue_id`、`NVARCHAR(MAX)`、`SalesConsultant` 这类 token 在窄表格列里会溢出。`convert.py` 在 CamelCase 边界、下划线和左括号前插入断行点，26 张表因此没有出现 Overfull。

**4. SQL Server 而不是 MySQL**

全文按 T-SQL 书写：`IDENTITY(1,1)`、`NVARCHAR/NVARCHAR(MAX)`、`DATETIME2(0)`、`CHECK` 约束代替 `ENUM`、`SYSDATETIME()`、`AFTER UPDATE` 触发器代替 `ON UPDATE CURRENT_TIMESTAMP`、`BACKUP DATABASE` 代替 `mysqldump`，排序规则 `Chinese_PRC_CI_AS`，驱动用 `pymssql`。

## 说明

- 图 1-8、图 1-13 画的是普通活动图而非泳道图（课程要求不使用泳道）。
- 图 2-3 URPS 雷达图由 matplotlib 绘制，UML 无对应图种。
- 案例中的公司与人名来自公开教学案例；报告里的量化基线属案例给定值或已显式登记的建模假设，逐条列在正文表 1-1（业务事实与假设登记表）。
- 尚无实现代码。文档中对落地技术栈的规划是 Python 3.11 + FastAPI + pymssql + SQL Server 2019+。

## 作者

周春尧 · 8102240203 · 信管 T2401 · 指导教师：毕文杰 教授

课程设计作业，仅供学习参考，请勿直接提交。
