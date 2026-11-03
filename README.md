# Appeasing the Machine Spirit

**Two rites of RLHF in in-context learning: predicting a solo developer's playtest notes from their todo, and then changing the todo so the notes never need writing.**

- 📄 **Preprint (PDF):** https://ghost---shadow.github.io/rlhf-in-icl/appeasing-the-machine-spirit.pdf
- ✍️ **Blog post:** https://ghost---shadow.github.io/rlhf-in-icl/

## The idea

I'm making a game alone: *The Good Vet*, an open-world vet game with soulslike fights. Every few days I play the latest build and write a list of notes. Each note names a thing, what's wrong with it, and what I want instead. The list goes back into my `todo.md`, the todo gets built, and I play again. After a month, git held sixteen of these lists, each paired with the todo it was played against.

That history is a small supervised dataset of one person's reactions. It raises two questions:

1. **Can an agent read a todo and predict the list I'll write?**
2. **If it can, can it rewrite the todo so I don't have to write the list at all?**

Nothing here is fine-tuned. Both rites use in-context learning only:
- the "weights" are one markdown file in the agent's memory folder;
- the "dataset" is git tags pairing each todo with its list;
- each epoch is a fresh agent, so the file is the only thing carried forward.

I call it **RLHF in ICL**. The human feedback is my own notes, the policy is the forecast, and the update is a rewrite of the memory file.

## The two rites

### The First Rite: train the memory

The data is fixed and the memory trains. A fresh agent reads the memory, walks the past pairs in order, and for each one:
1. forecasts the list from the todo, before opening the real list;
2. opens the list and writes down, in words, what it got wrong.

At the end of the epoch it turns the losses into patterns and rewrites the memory. One fixed scorer grades every epoch, and also counts *recall*: hits that only restate the list being scored.

```
for each epoch:
    agent = fresh agent                       # zero_grad()
    for each past (todo, list) pair, in order:
        forecast = agent.predict(memory, todo, earlier lists)    # before the list is opened
        loss     = agent.compare(forecast, list)
    memory = agent.rewrite(memory, patterns in the losses)       # do/don't rules, no past notes
    commit memory                             # checkpoint
    stop when held-out hits stop rising
```

### The Second Rite: converge the todo

The memory is frozen and the todo moves, like optimising an input against a frozen network. Each round, a fresh agent:
1. forecasts the notes I'd write if every open box were built exactly as written;
2. folds every new problem back into the todo as a fix, a warning on an existing box, a tightened box, or a decision.

It stops when a round finds almost nothing new.

```
repeat:
    agent    = fresh agent
    forecast = agent.predict(memory, todo, every past list)     # as if the todo were built
    new      = problems the todo doesn't cover yet               # the loss
    todo     = fold(todo, new)
until new is (almost) empty
```

If both rites worked perfectly, my next playtest list would come back empty, and the machine spirit would be appeased.

## What we found

All numbers use a strict rubric: a forecast line scores only if it names the thing *and* the problem.

| Run | Training pairs (564 notes) | Held out (191 notes, never opened) |
|---|---|---|
| First Rite, epoch 1 → 4, all hits | 31 → 119 | n/a |
| First Rite, epoch 1 → 4, recall removed | ~27 → ~38 (flat from epoch 2) | n/a |
| Rules memory trained on the other pairs | n/a | 4, 5, 6 over three epochs |
| No rules memory, raw earlier lists in context | 63 | 29 |
| Do/don't memory, frozen, last 3 lists in context | 32 (zero recall) | 10 |

- **The training curve lied.** By epoch 4, about 81 of 119 hits were recall. The agents had written my past notes into the memory: the labels in the weights.
- **Raw history beats distilled rules,** about five to one on lists the memory never saw. Most notes are repeats of named things, half-finished asks, or the next step of a wish, and those live in the raw lists.
- **A label-free memory** of general do/don't pairs is honest. Its biggest loss was a 12-line cap, not missing knowledge: 298 of 520 misses were rules it already had with no room to write them.
- **The Second Rite** turned a 176-note forecast into 64 fix boxes, 35 warnings and 9 decisions. Later rounds found 79, then 44 new items, roughly halving each round.

Full method, tables, the five data leaks we found, and the limitations are in the [preprint](https://ghost---shadow.github.io/rlhf-in-icl/appeasing-the-machine-spirit.pdf).

## Lessons, if you try this

- **Measure recall,** or your training curve will lie.
- **Keep the labels out of the weights.** A memory that lists past answers is a lookup table, not a predictor.
- **Give the model the raw history,** and spend the memory on how to read it.
- **Write rules as do/don't pairs,** each with its contrast, in terms that apply to a future you haven't seen.
- **Score strictly.** Half credit for "the right kind of problem" rewards vague lines.
- **Don't cap the forecast.** A predicted problem that doesn't come up costs nothing, and it may be one the human missed.
- **A fresh agent per epoch keeps it blind,** but onboarding is costly. Prepare the inputs as files, and use few, long-running agents.

## Repository layout

```
paper/
  main.tex                 the preprint
  neurips_preprint.sty     a NeurIPS-like layout, written for this repo (not the official style)
  refs.bib                 references
  figures/accuracy.png     accuracy per epoch, training and held out
blog/
  index.html               the blog post, published to gh-pages
.github/workflows/
  build.yml                CI: compile the PDF, publish the blog and the PDF
```

The developer's raw playtest lists aren't in this repository. The paper reports counts and a few short quotes.

## Building

### Locally

With a TeX distribution (TeX Live or MiKTeX):

```bash
cd paper
pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```

or with `latexmk`:

```bash
cd paper && latexmk -pdf main.tex
```

The style needs `newtx`, `microtype`, `fancyhdr`, `geometry`, `natbib`, `booktabs`, `algorithm` and `algpseudocode`. MiKTeX fetches them on demand.

### CI/CD

`.github/workflows/build.yml` runs on every push to `main`:

1. compiles `paper/main.tex` with [`xu-cheng/latex-action`](https://github.com/xu-cheng/latex-action);
2. uploads the PDF as a build artifact;
3. assembles `blog/` with the PDF and the figure, and pushes them to the `gh-pages` branch with [`peaceiris/actions-gh-pages`](https://github.com/peaceiris/actions-gh-pages).

GitHub Pages serves `gh-pages` at https://ghost---shadow.github.io/rlhf-in-icl/. Pull requests build the PDF but don't publish.

## Status

Work in progress. The real test of both rites is the next playtest, a list no model has seen. When it lands, it becomes pair seventeen, and the results here get updated.

## Citation

```bibtex
@misc{nanda2026appeasing,
  title  = {Appeasing the Machine Spirit: Two Rites of {RLHF} in In-Context Learning
            for Predicting and Preventing Playtest Notes},
  author = {Nanda, Souradeep},
  year   = {2026},
  note   = {Preprint},
  url    = {https://github.com/Ghost---Shadow/rlhf-in-icl}
}
```

## Acknowledgements

The experiments were run by Claude Opus 5.5 agents in Claude Code.

## Licence

MIT. See [`LICENSE`](LICENSE).
