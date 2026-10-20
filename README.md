# kelip-paper-reading

**简体中文** | [English](README.en.md)

让 AI Agent 带你精读一篇论文。它不给你一段摘要了事，而是像一个已经读懂这篇的人，一站一站带你看。

丢给它一个 arXiv 链接，它先把 **PDF、LaTeX 源码、每张图（自动裁好的 PNG）、官方代码** 拉到本地，
然后按六站讲，讲到哪张图就把那张图发给你，每站讲完问你要不要往下走：

```
① 动机与 Insights → ② 架构 → ③ 训练（损失函数）→ ④ 数据（一条样本长什么样）→ ⑤ 输入输出信息流 → ⑥ 拿一个例子把信息流走一遍（对照代码）
                                                                                （可选 ⑦ 实验和这篇现在的地位）
```

边讲边把每一站和你问过的问题记进论文目录里的 `精读笔记.md`。

## 六站，每站讲完你能说出什么

| 站 | 讲完你能说出 |
|---|---|
| ① 动机与 Insights | 具体问题、旧方法为什么卡住、作者的关键观察如何导向设计，以及依据和适用边界 |
| ② 架构 | 总览图上每个框是什么、数据从哪进从哪出；模型一次看几帧、输出几步 |
| ③ 训练 | 损失函数在逼模型学什么、分几个阶段、每阶段动了哪些参数 |
| ④ 数据 | 一条训练样本长什么样：字段、形状、单位、一行真实数值，而不只是数据集名字 |
| ⑤ 信息流 | 从输入一路跟到输出，每一步张量多大，哪些只算一次、哪些要循环 |
| ⑥ 例子 | 用一个具体任务，对着代码把整条链按时间顺序走一遍 |

## 几条规矩（都来自真实读论文时的反馈）

- **图要发出来。** 只说「见图 3」，你是看不到的。讲之前它会先自己看一遍图，再告诉你「这张图是什么、怎么读」。
- **公式写成不用渲染也能读的样子。** Unicode 公式 + 符号表 + 代一组小数字算一遍。
- **没听懂就换讲法，而不是把原话说得更长**：真实数字 → 缩到玩具尺寸一步步走 → 先讲「最朴素的做法为什么不行」→ 日常比喻 + 一张对照表。
- **先基本情况，再设计细节。** 谁做的、输入什么输出什么、开源没有；去掉论文的推销腔。
- **区分来源。** 论文写的、代码里看到的、它自己推测的，分开标；代码和论文对不上的地方会指出来。
- **代码只读不跑。** 克隆官方仓库是为了对照，不替你装依赖、不跑脚本。

## 安装

**Claude Code**（装成 skill，说「读一下这篇论文」就会自动用上）：

```bash
git clone https://github.com/skJack/kelip-paper-reading.git ~/.claude/skills/kelip-paper-reading
pip install pymupdf
```

**其他 AI Agent**（Cursor / Codex / Copilot / 自己写的 agent）：clone 到项目里，让 agent 读 `SKILL.md` 照着做。
仓库根目录有一份 `AGENTS.md`，会自动读它的 agent 不用额外配置。

依赖只有 Python 3 和 [PyMuPDF](https://pymupdf.readthedocs.io/)。要能访问 arxiv.org 和 github.com；Hugging Face 连不上不影响。

## 用法

```
读一下这篇 https://arxiv.org/abs/2410.24164
带我精读 π0，放到 ~/papers 下面
直接讲数据：一条训练样本到底长什么样？
拿一个具体例子，对着代码把推理走一遍
```

第一条回复是「基本情况 + 第 ① 站」，之后每站结尾它会问要不要继续。随时可以打断提问、跳站、
让它「翻译一下引言」（原文和译文对照），或者「整理成文档」（精读笔记已经在了）。

## 每篇论文会得到一个这样的目录

```
papers/pi0-vision-language-action/
├── paper.pdf         原文
├── source/           arXiv 的 LaTeX 源码
├── paper.tex         源码拍平成一个文件（\input 展开、去注释和参考文献），公式和表格以它为准
├── paper.txt         PDF 逐页文本
├── outline.md        章节、公式在 paper.tex 里的行号，按需跳读
├── meta.json         标题、作者、日期、摘要、代码和项目页链接
├── figures/          每张图和表一个 PNG，index.md 写 caption 和正文在哪引用，_contact.png 一眼看完全部
├── code/             官方代码（浅克隆）
└── 精读笔记.md        每一站的要点、用到的图、你问过的问题
```

在 [research-workflow](https://github.com/skJack/research-workflow) 结构的项目里（当前目录有 `related_work/`），
论文自动放进 `related_work/<slug>/`，代码放进 `复现/<repo>/code`。论文目录里本来就有你自己整理的 `figures/` 时，
自动裁的图放到 `figures-auto/`，不碰原来的。

## 让它更懂你：profile.md

```bash
cp profile.example.md profile.md
```

写上你的方向、已经熟的东西（讲到时一句带过）、不熟需要铺垫的、喜欢拿什么场景举例。
不写也能用：第一次走到第 ⑥ 站时它会问你想用什么场景举例，并替你记下来。`profile.md` 已经在 `.gitignore` 里，不会被提交。

## 只用脚本

```bash
python3 scripts/fetch_paper.py 2410.24164 --dest ~/papers/pi0             # 拉全部材料
python3 scripts/fetch_paper.py ~/papers/pi0 --code https://github.com/Physical-Intelligence/openpi   # 论文里没写代码地址时手动补
python3 scripts/extract_figures.py ~/papers/pi0                              # 重新裁所有图表
python3 scripts/crop.py ~/papers/pi0/figures/fig-03.png --grid               # 带刻度看图，找要放大的那块
python3 scripts/crop.py ~/papers/pi0/paper.pdf --page 4 0.08 0.06 0.92 0.3 -o eq.png   # 从 PDF 页面裁一块（公式、表）
```

输入可以是 arXiv ID / 链接（abs、pdf、alphaxiv、Hugging Face Papers 都行）、PDF 直链、本地 PDF（改过名的 arXiv PDF
能从第一页水印认出 ID，照样拉源码），或者已经下好的论文目录（只补缺的）。

## 已知限制

- **裁图**靠 caption 定位加版式规则：单栏、双栏、IEEE、NeurIPS / ICLR、期刊等常见模板都测过，图基本都能裁对；
  表格比图差一些。裁坏的在 `_contact.png` 里一眼能看出来，用 `crop.py` 重裁就行，agent 也会在讲到之前先检查。
- **LaTeX 源码**只有 arXiv 有；其他来源的 PDF 只能读抽出来的文本，公式和表格会粗糙一些。
- **找代码**看的是摘要、arXiv 备注、PDF 首页链接、Hugging Face Papers 和项目页；论文里压根没写的
  （比如 π0 → openpi），要 agent 去搜，再用 `--code` 补上。

## 文件

| 路径 | 是什么 |
|---|---|
| `SKILL.md` | 给 agent 的带读流程：六站各讲什么、图和公式怎么处理、没听懂时怎么换讲法 |
| `AGENTS.md` | 一页指路，给会自动读它的 agent |
| `references/examples.md` | 每一站讲成什么样的样例 |
| `references/notes-template.md` | 精读笔记模板 |
| `profile.example.md` | 用户画像模板（复制成 `profile.md`） |
| `scripts/fetch_paper.py` | 下载 PDF / LaTeX / 元数据，拍平源码，生成大纲，找官方代码并浅克隆，调用裁图 |
| `scripts/extract_figures.py` | 按 caption 从 PDF 裁出每张图和表，写 `index.md` 和总览拼图 |
| `scripts/crop.py` | 放大图的局部、从 PDF 页面裁公式或重裁坏图 |

## 相关

- [kelip-slide](https://github.com/skJack/kelip-slide)：把读懂的论文做成讲解用的 HTML 幻灯片
- [research-workflow](https://github.com/skJack/research-workflow)：用 Agent 协作做科研的项目结构（文献、复现、实验）

## License

MIT
