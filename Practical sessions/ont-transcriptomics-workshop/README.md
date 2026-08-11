# ONT Transcriptomics Workshop — alternative splicing from nanopore cDNA

Measuring **alternative splicing without a polished reference transcriptome**, end to end, in about
90 minutes. Oxford Nanopore full-length cDNA reads (kit SQK-PCB109) from *Arabidopsis thaliana*,
nucleus vs. cytoplasm, chromosome 3.

The session assembles a transcriptome **from the reads themselves**, quantifies splicing against it,
then runs the *identical* analysis against the curated AtRTDv2 annotation and puts the two side by
side — so you can say concretely what a reference buys you, and what it does not.

```
minimap2 → StringTie (-L) → gffread → oarfish → SUPPA2 → IGV
```

---

## Quick start

```bash
# 1) Create the Python environment (once, ~3 min)
conda env create -f environment.yml          # or: mamba env create -f environment.yml
conda activate ont-rna-3.1
python -m ipykernel install --user --name ont-rna-3.1 --display-name "ONT RNA 3.1"

# 2) Launch Jupyter
jupyter lab notebook/3.1_Transcriptomics_splicing.ipynb
```

Pick the **ONT RNA 3.1** kernel and run the cells top to bottom.

> **Select that kernel — it is not optional.** The notebook launches SUPPA2 with its own interpreter
> (`sys.executable`). Under a different kernel, every SUPPA step fails on `import sklearn` before it
> ever looks at your data.

**The first cell downloads the inputs.** This repository holds only the notebook — the reads,
references and command-line tools are far too big for git, so the first code cell fetches and
unpacks them (about **610 MB, once**) from Google Drive:

| Fetched into | Contents | Download |
|---|---|---|
| `data/` | 6 FASTQ libraries, chr3-only (1.5 GB unpacked) | 579 MB |
| `references/` | `TAIR10_chr3.fna` + `.fai`, AtRTDv2 `.fa` + `.gtf` | 11 MB |
| `tools/` | minimap2, samtools, stringtie, gffread, oarfish, bgzip, tabix + SUPPA2 | 19 MB |

Anything already on disk is skipped, so re-running that cell is free. It also writes the
`WORKSHOP_ROOT` marker that every later cell uses to find the folder.

> **No network, or a slow room?** Download the three archives by hand, drop `data.zip`,
> `references.zip` and `tools.zip` next to `notebook/`, and the first cell unpacks those instead of
> fetching anything. Archives it downloaded itself are deleted after unpacking; ones you staged are
> left alone.

After the environment, total compute is about **5 minutes on 8 threads** — and that covers *both*
annotation routes.

## Requirements

* **Linux x86-64** — the bundled binaries are Linux builds. They need only glibc ≥ 2.16, so every
  supported Ubuntu release runs them as-is; the shared libraries they need ship alongside them in
  `tools/lib`, so they work whether or not the conda environment is active.
* **conda or mamba**, for the Python stack in `environment.yml` (pandas, numpy, scipy,
  scikit-learn, statsmodels, matplotlib, seaborn — SUPPA2 imports the middle two at start-up).
* **~5 GB free disk** — 1.6 GB of inputs plus ~2.6 GB the run produces.
* **Desktop IGV** for the last section. It comes with the environment; launch it from an activated
  shell (`conda activate ont-rna-3.1 && igv`) so it picks up the environment's Java 21 — the system
  Java 17 fails with `Unsupported major.minor version 65.0`.

## What's in here

| Path | Contents |
|------|----------|
| `notebook/3.1_Transcriptomics_splicing.ipynb` | **The workshop** — 39 cells, outputs from a full run included so you can read the expected results before running anything. The last section is a skippable annex. |
| `environment.yml` | The Python environment (`ont-rna-3.1`). |
| `reference_outputs/` | Pre-computed SUPPA2 results from an earlier, salmon-based 4-condition run — for comparison only; nothing reads them. |
| `results/` | Empty. Everything the notebook writes lands here (~2.6 GB), split by route. |
| `igv_sessions/` | Empty. Step 9 generates a desktop-IGV session here, pointing at *your* BAMs. |
| `data/`, `references/`, `tools/` | Not in git — created by the first cell. |

## What the notebook does

1. Inspect the reads, then align all six libraries to the **genome** with `minimap2 -ax splice`.
   This one alignment feeds the assembler *and* the genome browser at the end.
2. Check alignment quality with `samtools flagstat`.
3. **Assemble a transcriptome de novo** with StringTie in long-read mode (`-L`), merge the six
   per-sample assemblies, and extract transcript sequences with `gffread`.
4. **Score that assembly** against AtRTDv2 — splice-junction precision and sensitivity, and exact
   intron-chain recovery.
5. Quantify abundance with **oarfish** against **both** transcript sets.
6. Run **SUPPA2** twice per route: isoform level (`-f ioi`) and local events (`-f ioe`).
7. **Compare the two routes** on event counts and on differential intron retention.
8. **Look at the reads in IGV** at a de novo-called retained intron, with both the assembled and the
   curated model on screen.

Nucleus vs. cytoplasm is a deliberately strong contrast: splicing happens in the nucleus and only
fully processed mRNA is exported, so **intron-retaining molecules should pile up in the nuclear
fraction**. Because we already know which way the answer should come out, it tests the *method*, not
the biology — if the de novo route recovers the same result as the reference route, the de novo
route works.

## Notes

* **`MSTRG.*` identifiers are not stable.** They are assigned at assembly time, so re-running can
  renumber them; the coordinates do not move. Any gene id quoted in the prose may differ from what
  your own run produces.
* **The reads are restricted to chromosome 3**, which is what keeps the whole pipeline inside a
  single session.
* Uses **oarfish** rather than `salmon quant --ont`: long-read support was dropped from salmon
  upstream in favour of oarfish.
