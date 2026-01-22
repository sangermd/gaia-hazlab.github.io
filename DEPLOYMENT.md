# Deployment Guide (pixi-first)

This guide explains how to build and deploy the GAIA HazLab website. The repository's CI uses `pixi` to create a reproducible environment and build the site; follow the `pixi` workflow locally when possible. A conda fallback is provided.

## Quick summary

- Preferred local workflow: use `pixi` tasks defined in `pixi.toml`.
- CI: `.github/workflows/deploy.yml` runs `pixi run build-ci` and deploys the `website` folder to GitHub Pages.
- Output: the Jupyter Book HTML is placed in `website/book` (this directory is what gets deployed).

## Prerequisites

- Install `pixi` per https://pixi.sh/dev/installation/ (recommended).
- Recommended Python: 3.11+ (CI/pixi lock pins newer versions; see `pixi.lock`).

## Pixi workflow (recommended)

After installing `pixi`, from the repository root run:

```bash
# Serve locally (preview site):
pixi run serve-all

# Build the book and copy HTML into website/book (local build):
pixi run build-all

# Run the CI-equivalent build locally (same commands as GitHub Actions):
pixi run build-ci

# Run link and spell checks (optional):
pixi run linkcheck
pixi run spellcheck
```

Notes:
- `build-all` and `build-ci` perform `jupyter book build --html`, then move `_build/html` to `website/book` and copy images.
- The `BASE_URL` environment variable is set in `pixi.toml` (and in CI) to ` /book` so asset URLs resolve correctly on GitHub Pages; keep this unless you change the deployment path.

## Conda fallback (no pixi)

If you prefer conda/mamba without pixi, create an environment and build manually:

```bash
conda create -n gaia-hazlab python=3.11 -c conda-forge jupyter-book sphinx-design codespell nodejs -y
conda activate gaia-hazlab

# Build the book
jupyter-book build book

# Move build to website/book to match CI behavior
mv book/_build/html website/book
mkdir -p website/book/img
cp -R book/img/. website/book/img/

# Preview the full website
python -m http.server 8000 --directory website
```

## Fixing/Generating `book/_toc.yml` (Table of Contents)

If the book fails to build because a ToC is missing, generate a minimal `book/_toc.yml`:

Preferred (if `jupyter-book` CLI supports it):

```bash
jupyter-book init --write-toc book
# or, depending on Jupyter Book CLI installed:
jupyter book init --write-toc book
```

Minimal `book/_toc.yml` example you can create and commit manually:

```yaml
format: jb-book
root: intro
chapters:
  - file: chapters/project-organization
  - file: chapters/datahub
  - file: chapters/modelhub
  - file: chapters/hazevalhub
  - file: chapters/research-software
  - file: chapters/soil-memory
  - file: chapters/convective-thunderstorms
  - file: chapters/wa-2025-river-floods-sediment-transport
  - file: chapters/wa-2025-stehekin
  - file: chapters/wa-2001-2031-nisqually-earthquake
  - file: chapters/resources
```

Commit `book/_toc.yml` and push before running the CI build.

## What CI does (overview)

The GitHub Actions workflow `.github/workflows/deploy.yml`:

1. Sets `BASE_URL=/book` (so assets reference `/book/...`).
2. Uses `prefix-dev/setup-pixi` and runs `pixi run build-ci`.
3. Optionally runs spellcheck and linkcheck.
4. Uploads `website/` as the Pages artifact and deploys using `actions/deploy-pages`.

If you need to reproduce CI locally, use `pixi run build-ci` so the environment and commands match.

## Troubleshooting

- Missing `_toc.yml` or incorrect ToC: create `book/_toc.yml` (see above), rebuild, and commit.
- Asset 404s (CSS/JS missing): ensure `BASE_URL` is set to `/book` when building (pixi tasks do this). If deploying to a root path, set `BASE_URL` to `''` and rebuild.
- Sphinx extension errors (e.g., `sphinx_design`): install required packages (`jupyter-book`, `sphinx-design`) in the same environment used to build (pixi ensures reproducibility).
- Link or spellcheck failures: run `pixi run linkcheck` and `pixi run spellcheck` locally; both are allowed to fail in CI but help catch issues early.
- Build still fails in Actions: open the Actions run, inspect logs for failing step output; try reproducing the failing command locally with `pixi run build-ci`.

## Deployment (merge to `main`)

1. Merge your PR to `main`.
2. CI will run and, on success, publish the content of `website/` to GitHub Pages.
3. Access the site at `https://gaia-hazlab.github.io` (or your custom domain if configured).

## Additional notes

- The repo includes `pixi.lock` which pins a reproducible environment (CI uses this). If you need exact versions, use pixi or consult `pixi.lock`.
- If you want me to commit a polished `book/_toc.yml` tailored to any reordering, tell me the order and I will update it.
