# tony-skills

Skills for [Claude Code](https://claude.com/claude-code) (and any agent that
reads `SKILL.md`).

| Skill | What it does |
|---|---|
| [`pishi/`](pishi/) | Russian prose for a human reader — messages to a CTO, ops instructions, tickets, PR bodies, explanations, cover letters, how-tos for parents. Infostyle method + Russian AI-tell removal + your own house rules, with a zero-dependency linter that scores a draft 0–100. |
| [`motion-video/`](motion-video/) | Motion-design videos as code — app showcases, launch films, rebrand reveals, vertical ads. HTML composition with `__seek(t)`, rendered frame by frame (Playwright + ffmpeg) with linear-light motion blur and correct BT.709 encoding; real-UI capture on a slowed page clock; a sound mixer from recorded SFX verified by measurement; contact sheets, automated checks and a critic loop. |
| [**howmanyusers**](https://github.com/tonysafonov/howmanyusers) (was `perf-lab/`) | How many users can your app hold? Load-test, profile and optimize a backend on two rented VPS boxes, Arjay-McCandless-style. Now its own repo; `perf-lab/` here is frozen for existing installs. |

## pishi — «Пиши, сокращай» для агента

Claude's Russian is grammatically clean and still reads as machine output: twice
as long as needed, opens with a preamble, cites numbers that are not in the
source, formats a chat answer as a document with headers. `pishi` fixes that
layer first, then the language layer (stop-words, bureaucratese, passive,
vagueness, AI-tells).

**Order of work** (this order, because it matches how people actually correct
machine Russian): reader / action / channel / length → facts only from the
source → first line is the point → structure = the reader's question → eight
stop-word groups → verbs and one thought per sentence → AI-tells → one linter
pass → deliver.

**One file at runtime.** Everything the model needs is in `SKILL.md` (≈ 210
lines); nothing else is read except your own `house-rules.md`. The linter is
executed, not read.

**Linter** — `pishi/scripts/check.py`, stdlib only:

```bash
python3 pishi/scripts/check.py draft.txt
python3 pishi/scripts/check.py draft.txt --source notes.md --max-words 180 --no-greeting
echo "текст" | python3 pishi/scripts/check.py -
```

Prints hits by category with line numbers and a `ЧИСТОТА: N/100` score.
`--source` flags every number in the draft that does not appear in the source
material — the «цифры непонятно откуда» detector. Exit code 1 on `--max-words`,
`--no-greeting`, `--min-score` or foreign-number failures, so it works in evals.

**House rules** — the skill reads `~/.claude/pishi/house-rules.md` before every
task if it exists. That file is where *your* repeated corrections live (quote →
rule), plus your team's formats. Template: `pishi/house-rules.example.md`.
Keep it out of public repos.

**Benchmark** — `pishi/evals/`: six tasks on fictional data with mechanical
checks. On the author's private set of real tasks (iteration 1, Opus, 5 tasks ×
with/without skill): assertion pass rate 100 % vs 77 %; mean output length −29 %;
linter scores were 96–100 in both arms — the wins are all in length, format and
facts-from-source, not in vocabulary.

## Install

```bash
git clone https://github.com/tonysafonov/tony-skills ~/tony-skills
ln -s ~/tony-skills/pishi ~/.claude/skills/pishi          # user scope: every project
# or project scope:
ln -s ~/tony-skills/pishi <repo>/.claude/skills/pishi
mkdir -p ~/.claude/pishi && cp ~/tony-skills/pishi/house-rules.example.md ~/.claude/pishi/house-rules.md
```

Restart Claude Code. The skill triggers on Russian writing requests («напиши»,
«объясни по-русски», «сократи», «сообщение для…», «тикет», «PR на русском»).

## Credits

The editing method is the information style of Максим Ильяхов and Людмила
Сарычева («Пиши, сокращай»), paraphrased; examples here are original. The
Russian AI-tell list draws on public work such as
[humanizer-ru](https://github.com/ilyautov/humanizer-ru) and the
[LLM Writing Quality by Language](https://peterkaminski.ai/research/llm-writing-quality-by-language/)
survey.

MIT.

## motion-video — motion design as code

The agent is the motion designer; the skill is its studio. The film is a program — `templates/compose.html`
exposes `__meta` and `__seek(t)` — and `scripts/render.mjs` renders every frame and encodes a clean MP4
(PNG frames, BT.709, linear-light motion blur on fast moves, never across a cut, parallel workers).
Around it: `capture.mjs` records a real app on a slowed page clock, `beats.py` finds a track's beats and drop
so the film can be cut to music, `mix.py` builds the soundtrack from a spec (recorded SFX placed on peaks,
levelled by perceived loudness, one room), and `qa/` gives contact sheets and mechanical checks.
`references/inspiration.md` collects the films that landed and the prompts behind them; the rest of
`references/` is a library of moves with good default numbers — not rules.

```bash
node motion-video/scripts/render.mjs comp.html --stills 0,1.6,3.3 -o stills/
node motion-video/scripts/render.mjs comp.html -o out.mp4
uv run --with librosa python motion-video/scripts/beats.py track.mp3 --len 15
```

Personal taste rules (short!) go in `~/.claude/motion-video/house-rules.md` — see `house-rules.example.md`.
Built from claude-motion-design, onetake (ideas only — it is non-commercial), brag, HyperFrames,
ClaudeAnimationBase and a lot of rejected renders; sources in `motion-video/references/tools.md`.

## howmanyusers (was perf-lab)

The load-testing skill moved to its own repo: **[tonysafonov/howmanyusers](https://github.com/tonysafonov/howmanyusers)**.
How many users can your app hold? Two rented boxes in one datacenter, a realistic user from your analytics, 5-minute
holds, CPU prices and profiles, a fix ladder, a chaos test, an honest report. Case studies: a Python framework from 220
to 1,400 users on the same 2-vCPU box, a production Mini App from 120 to 900, Python vs TypeScript on the same night.

`perf-lab/` stays here unchanged so existing symlinks keep working; new work happens in howmanyusers.

