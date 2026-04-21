# wasm-bioc-repo

WebAssembly binary repository for Bioconductor packages used in the
**Foundations of Omics Study Design** workshop practicals.

Builds `edgeR`, `limma`, `DESeq2`, and their dependencies as
WebAssembly binaries deployable via GitHub Pages for use with
[webR](https://docs.r-wasm.org/webr/latest/).

---

## Setup (one time, ~5 minutes)

### 1. Create the GitHub repository

- Go to [github.com/new](https://github.com/new)
- Name it `wasm-bioc-repo` (or anything you like)
- Set it to **Public** (required for free GitHub Pages)
- **Do not** initialise with a README — clone and push these files instead

### 2. Enable GitHub Pages

- Go to your repo → **Settings** → **Pages**
- Under **Source**, select **GitHub Actions**
- Click Save

### 3. Push these files

```bash
git clone https://github.com/YOUR-USERNAME/wasm-bioc-repo
cd wasm-bioc-repo
# copy these files in, then:
git add .
git commit -m "Initial setup"
git push
```

The GitHub Actions workflow starts automatically on push.
Watch progress under the **Actions** tab — the build takes
**10–30 minutes** depending on which packages compile cleanly.

### 4. Check the build result

- Green tick = all packages built successfully
- Red cross = one or more packages failed to compile

If it fails, click into the failing step to see which package
caused the error. Common causes and fixes are in the
[Troubleshooting](#troubleshooting) section below.

---

## Using the repo in your webR practical

Once the GitHub Pages site is live, install packages in webR with:

```r
webr::install("edgeR", repos = c(
  "https://YOUR-USERNAME.github.io/wasm-bioc-repo/",
  "https://repo.r-wasm.org/"
))
library(edgeR)
```

Replace `YOUR-USERNAME` with your actual GitHub username.

The URL pattern is always:
`https://USERNAME.github.io/REPO-NAME/`

---

## What is in the `packages` file

```
edgeR              # core DE package
limma              # linear models / voom
DESeq2             # NB GLM DE package
Biobase            # Bioconductor base classes
BiocGenerics       # generic functions
S4Vectors          # S4 vector classes
IRanges            # integer range operations
GenomicRanges      # genomic coordinate classes
GenomeInfoDb       # chromosome name handling
SummarizedExperiment  # the container DESeq2 uses
DelayedArray       # lazy array evaluation
MatrixGenerics     # matrix summary generics
BiocParallel       # parallel computation
```

These are listed explicitly rather than relying on automatic
dependency resolution, because some deep Bioconductor dependencies
have known WebAssembly build issues and listing them explicitly
lets the build system handle them individually.

---

## Troubleshooting

**Build fails on `GenomeInfoDb`**
This package previously required `RCurl` which has no WebAssembly
build. The dependency was removed in Bioconductor 3.19+. If this
fails, the Bioconductor release being used in the build may be older
than that. Open an issue in `r-wasm/actions` on GitHub.

**Build fails on `DESeq2`**
DESeq2 depends on `SummarizedExperiment` which depends on `GenomeInfoDb`.
If `GenomeInfoDb` fails, `DESeq2` will too. Try removing `DESeq2` from
the `packages` file first — if `edgeR` and `limma` build cleanly,
you can still run the workshop with those two packages.

**`edgeR` or `limma` fail**
These are relatively self-contained C++ packages. Failures here
usually indicate a temporary issue with the build environment.
Try re-running the workflow manually from the Actions tab
(the `workflow_dispatch` trigger is already configured).

**Page not found after build succeeds**
GitHub Pages can take 1–2 minutes to propagate after the
action completes. Wait and refresh.

---

## Updating packages

To rebuild with newer package versions, either push a new commit
or trigger the workflow manually:
- Go to **Actions** tab
- Select **Build and deploy wasm R package repository**
- Click **Run workflow**

---

## Links

- [webR documentation](https://docs.r-wasm.org/webr/latest/)
- [r-wasm/actions](https://github.com/r-wasm/actions)
- [rwasm package](https://r-wasm.github.io/rwasm/)
- [Bioconductor webR repo (experimental)](https://webr.bioconductor.org/)
