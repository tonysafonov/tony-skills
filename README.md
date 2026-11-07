# tony-skills

Skills for [Claude Code](https://claude.com/claude-code) (and any agent that
reads `SKILL.md`).

| Skill | What it does |
|---|---|
| [`pishi/`](pishi/) | Russian prose for a human reader — messages to a CTO, ops instructions, tickets, PR bodies, explanations, cover letters, how-tos for parents. Infostyle method + Russian AI-tell removal + your own house rules, with a zero-dependency linter that scores a draft 0–100. |
| [`motion-video/`](motion-video/) | Motion-design videos as code — app showcases, launch films, rebrand reveals, vertical ads. HTML composition with `__seek(t)`, rendered frame by frame (Playwright + ffmpeg) with linear-light motion blur and correct BT.709 encoding; real-UI capture on a slowed page clock; a sound mixer from recorded SFX verified by measurement; contact sheets, automated checks and a critic loop. |
| [`perf-lab/`](perf-lab/) | Load-test, profile and optimize a web backend on two rented VPS boxes, Arjay-McCandless-style ("how many users can this server hold"): calibrate against a published reference, a realistic closed-model k6 journey from your analytics, staircase + 5-minute holds, CPU price per endpoint, py-spy at saturation, a fix ladder A/B'd by commit, a chaos test, and an honest report. |

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
git clone https://github.com/AHTOOOXA/tony-skills ~/tony-skills
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

## perf-lab — how many users does your backend hold?

A performance lab in a skill: rent two boxes in one datacenter, calibrate the rig against a published reference
implementation, model a real user from your analytics, find the number of users the backend holds for five minutes,
find out why it isn't more, fix it one step at a time, and write it up so a stranger can check it.

The method is Arjay McCandless's — his YouTube "how many users can a $12 server hold" challenges and their
leaderboards ([github.com/arjaythedev](https://github.com/arjaythedev)): a target box and a load generator in one
datacenter, a closed-model user with think times, 5-minute holds judged on p95 < 500 ms, p99 < 1 s, < 1 % errors.
The experience is from load-testing [gramkit](https://github.com/AHTOOOXA/gramkit) (FastAPI + Vue/React Telegram
Mini App framework) and a production Mini App built on it. On Arjay's multicore Conduit benchmark, 2 dedicated vCPU:

| Users held 5 min | first round | re-run on the fixed core, same day as its reference |
|---|---:|---:|
| plain FastAPI reference (`examples/python`, one worker per core = 2) | 3,400 | 3,000–3,100 |
| gramkit as first shipped (1 worker) | 220 | — |
| gramkit with the fixes, 1 worker (template default) | — | 710 |
| gramkit with the fixes, 2 workers (one per core, like the reference) | 1,400 | 1,300 |

The fixes: per-process clients, a bounded DB pool, pure-ASGI middleware, real core dependencies instead of
`dependency_overrides`, off-loop log writes. Only same-day numbers compare: the unchanged reference drifted ~10 %
between the two days, so gramkit's share is the stable figure (41 % → 42–43 % of the reference).

On the production stand the realistic journey found what a synthetic benchmark never would: per-request connections
overwhelming Docker's DNS until auth failures logged users out (401), a burst of app opens OOM-killing the container,
DB transactions held open through 20-second LLM streams, aiohttp's 100-connection default — and a probe that measured
fast 401s for hours because k6's `__ENV` includes the OS environment. All of it is in `references/pitfalls.md`.

**Runtime:** `SKILL.md` (the procedure, ≈ 125 lines) + `references/` loaded on demand (rig and provider recipe,
calibration, journey, load, profiling, fixes, pitfalls, reporting). Scripts are executed, not read:

```bash
S=~/tony-skills/perf-lab
mkdir -p perf && cp -r $S/scripts/k6 perf/k6 && cp $S/templates/journey.js perf/k6/journey.js   # write your user here
BOT_TOKEN=<test bot token> python3 -I $S/scripts/tg_initdata.py 3000 > perf/k6/data/users.json   # or a JSON array of tokens your app accepts
export TARGET=root@<target> LOADER=root@<loader> CONTAINER=<app container> BASE_URL=http://<target>:8000
$S/scripts/preflight.sh                                   # RTT, steal, disk, ports open past ufw, tools
$S/scripts/load.sh staircase asis -e START=30 -e STEP=30   # estimate
$S/scripts/load.sh hold asis 300                           # verdict: 5-minute hold
$S/scripts/cpuprobe.sh endpointprobe.js 6 me -e EP='GET /users/me'   # ms CPU per request (refuses non-2xx)
$S/scripts/pyspy.sh out/me.txt endpointprobe.js 6 -e EP='GET /users/me' && python3 $S/scripts/stacks.py top out/me.txt
DEP=<db container> $S/scripts/chaos.sh before              # DB down 6 s under load: 503 good, 401 bad
DEPLOY_CMD=... $S/scripts/ab.sh out/x base:<sha> new:<sha> # same procedure per variant, one table
```

Needs ssh keys to both boxes, Docker + py-spy on the target, k6 on the loader, python3 locally. No hosts or keys in
the scripts; ordering servers is always the owner's call. Personal rules go in `~/.claude/perf-lab/house-rules.md`
(see `house-rules.example.md`).

Install like the others:

```bash
ln -s ~/tony-skills/perf-lab ~/.claude/skills/perf-lab
mkdir -p ~/.claude/perf-lab && cp ~/tony-skills/perf-lab/house-rules.example.md ~/.claude/perf-lab/house-rules.md
```

Credits: Arjay McCandless — the challenges, the scoring, and `realistic-load-test` (MIT), from which `lib.js`,
`hold.js` and `staircase.js` are adapted (his licence is next to them in `scripts/k6/`). [k6](https://k6.io),
[py-spy](https://github.com/benfred/py-spy). The boxes were from Synteq (Arjay's sponsor); any provider works.
