# kelip-paper-reading

[简体中文](README.md) | **English**

Let an AI agent walk you through a paper. It won't stop at a summary. It reads the paper first, the way
someone who already understands it would, and then takes you through it one stop at a time.

Give it an arXiv link. It pulls the **PDF, the LaTeX source, every figure (auto-cropped to PNG), and the official code**
to your machine, then explains the paper in six stops. Whenever it talks about a figure, it shows you that figure.
After each stop it asks whether you want to move on:

```
① Motivation & Insights → ② Architecture → ③ Training (loss) → ④ Data (what one sample looks like) → ⑤ Input/output flow → ⑥ Walk one example through (against the code)
                                                                                          (optional ⑦ Results, and where the paper stands today)
```

As it goes, it writes each stop and every question you asked into a notes file in the paper folder (`精读笔记.md`, "reading notes").

## Six stops: what you can say after each one

| Stop | After this stop you can tell someone |
|---|---|
| ① Motivation & Insights | The concrete problem, why earlier approaches fail, how the authors' key observations lead to the design, and what evidence and conditions support those insights |
| ② Architecture | What every box in the overview figure is, where data goes in and comes out; how many frames the model sees and how many steps it outputs |
| ③ Training | What the loss pushes the model to learn, how many stages there are, and which parameters each stage updates |
| ④ Data | What one training sample actually looks like: fields, shapes, units, one row of real values. More than the dataset's name |
| ⑤ Information flow | How to follow an input all the way to the output, tensor shape at every step, what runs once and what loops |
| ⑥ Example | One concrete task, walked through in time order against the code |

## House rules (each one came from feedback while reading real papers)

- **Show the figure.** Saying "see Figure 3" doesn't help: you can't see it. The agent looks at the figure itself first,
  then tells you what the figure is and how to read it.
- **Write formulas so they read without a renderer.** Unicode formula + a symbol table + one set of small numbers plugged in.
- **If you didn't get it, explain it differently instead of saying the same thing longer**: real numbers → shrink to toy size
  and step through → first show why the naive approach fails → an everyday analogy plus a mapping table back to the paper.
- **Basics first, design details second.** Who made it, what goes in and what comes out, what's open-sourced. Skip the paper's sales pitch.
- **Keep sources apart.** What the paper says, what the code shows, and what the agent is guessing are labelled separately;
  places where the code disagrees with the paper get pointed out.
- **Code is read-only.** The official repo is cloned to compare against, not to install or run.

## Install

**Claude Code** (installed as a skill, it kicks in when you say "read this paper with me"):

```bash
git clone https://github.com/skJack/kelip-paper-reading.git ~/.claude/skills/kelip-paper-reading
pip install pymupdf
```

**Other AI agents** (Cursor / Codex / Copilot / your own agent): clone it into your project and have the agent read `SKILL.md`.
There's an `AGENTS.md` at the repo root, so agents that pick that file up automatically need no extra setup.

The only dependencies are Python 3 and [PyMuPDF](https://pymupdf.readthedocs.io/). You need access to arxiv.org and github.com;
Hugging Face being unreachable is fine.

`SKILL.md` is written in Chinese. Any modern LLM agent reads it fine and will still talk to you in your own language.

## Usage

```
Read this paper with me: https://arxiv.org/abs/2410.24164
Walk me through π0 and put it under ~/papers
Skip to the data: what does one training sample actually look like?
Take one concrete example and walk the inference through the code
```

The first reply is "basics + stop ①". Every stop ends by asking whether to continue. You can interrupt with questions any time,
jump to another stop, ask it to "translate the introduction" (original and translation side by side), or ask it to
"turn this into a document" (the reading notes are already there).

## What you get for each paper

```
papers/pi0-vision-language-action/
├── paper.pdf         the paper
├── source/           LaTeX source from arXiv
├── paper.tex         the source flattened into one file (\input expanded, comments and references removed); trusted for formulas and tables
├── paper.txt         PDF text, page by page
├── outline.md        line numbers of every section and equation in paper.tex, so the agent can jump around
├── meta.json         title, authors, dates, abstract, code and project-page links
├── figures/          one PNG per figure and table; index.md has captions and where each is cited; _contact.png shows them all at once
├── code/             the official code (shallow clone)
└── 精读笔记.md        reading notes: the gist of each stop, the figures used, the questions you asked
```

In a project laid out like [research-workflow](https://github.com/skJack/research-workflow) (the current directory has `related_work/`),
papers go to `related_work/<slug>/` and code goes to `复现/<repo>/code`. If the paper folder already has a `figures/`
you curated yourself, the auto-cropped figures go to `figures-auto/` and yours are left alone.

## Make it fit you: profile.md

```bash
cp profile.example.md profile.md
```

Write down your field, what you already know (it'll mention those in one line), what needs groundwork first, and what kind of
scenarios you like for examples. It works without one too: the first time it reaches stop ⑥ it asks what scenario to use and
writes the answer down for you. `profile.md` is in `.gitignore`, so it never gets committed.

## Use the scripts on their own

```bash
python3 scripts/fetch_paper.py 2410.24164 --dest ~/papers/pi0             # pull everything
python3 scripts/fetch_paper.py ~/papers/pi0 --code https://github.com/Physical-Intelligence/openpi   # add the code repo when the paper doesn't link it
python3 scripts/extract_figures.py ~/papers/pi0                              # re-crop all figures and tables
python3 scripts/crop.py ~/papers/pi0/figures/fig-03.png --grid               # overlay a grid to locate the part you want to zoom into
python3 scripts/crop.py ~/papers/pi0/paper.pdf --page 4 0.08 0.06 0.92 0.3 -o eq.png   # crop a region of a PDF page (equation, table)
```

Input can be an arXiv ID or link (abs, pdf, alphaxiv and Hugging Face Papers links all work), a direct PDF link, a local PDF
(a renamed arXiv PDF is recognised by the arXiv stamp on its first page, so the source still gets pulled), or a paper folder
you already have (only what's missing gets filled in). The scripts print their progress in Chinese.

## Known limitations

- **Figure cropping** locates captions and applies layout rules. It's been tested on common templates (single column,
  two column, IEEE, NeurIPS / ICLR, journals) and gets figures right nearly all the time; tables are less reliable.
  A bad crop is obvious in `_contact.png`; re-crop it with `crop.py`. The agent also checks before it uses a figure.
- **LaTeX source** only exists for arXiv papers. For PDFs from elsewhere only the extracted text is available, so formulas and tables are rougher.
- **Finding the code** checks the abstract, the arXiv comment, links on the PDF's first page, Hugging Face Papers and the project page.
  When the paper never mentions its code (e.g. π0 → openpi), the agent has to search and add it with `--code`.

## Files

| Path | What it is |
|---|---|
| `SKILL.md` | The reading protocol for the agent: what each stop covers, how to handle figures and formulas, how to re-explain when something didn't land |
| `AGENTS.md` | One-page pointer for agents that read it automatically |
| `references/examples.md` | Examples of what each stop looks like when it's done well |
| `references/notes-template.md` | Template for the reading notes |
| `profile.example.md` | Template for your profile (copy to `profile.md`) |
| `scripts/fetch_paper.py` | Downloads PDF / LaTeX / metadata, flattens the source, builds the outline, finds and shallow-clones the official code, runs figure extraction |
| `scripts/extract_figures.py` | Crops every figure and table out of the PDF by caption, writes `index.md` and a contact sheet |
| `scripts/crop.py` | Zooms into part of a figure, crops equations from a PDF page, or re-crops a bad figure |

## Related

- [kelip-slide](https://github.com/skJack/kelip-slide): turn a paper you understand into an HTML slide deck for explaining it
- [research-workflow](https://github.com/skJack/research-workflow): a project layout for doing research with agents (literature, reproduction, experiments)

## License

MIT
