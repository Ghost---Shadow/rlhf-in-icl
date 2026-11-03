# Appeasing the Machine Spirit

Two rites of RLHF in in-context learning: predicting a solo developer's playtest notes from their todo, and then changing the todo so the notes never need writing.

- **Preprint:** `paper/`, NeurIPS-style LaTeX. CI compiles it on every push to `main`.
- **Blog post:** `blog/`, published with the PDF to the `gh-pages` branch.

## Build locally

```bash
cd paper && latexmk -pdf main.tex
```

or, with [Tectonic](https://tectonic-typesetting.github.io/):

```bash
cd paper && tectonic main.tex
```

## CI

`.github/workflows/build.yml`:
1. compiles `paper/main.tex` with `xu-cheng/latex-action`;
2. uploads the PDF as an artifact;
3. pushes the blog and the PDF to `gh-pages`.

`paper/neurips_preprint.sty` is a small style written for this repository that follows the NeurIPS layout. It is not the official NeurIPS style file.

## Licence

MIT. See `LICENSE`.
