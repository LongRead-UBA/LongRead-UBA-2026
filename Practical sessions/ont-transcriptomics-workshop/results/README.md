# results/

**This folder is intentionally empty.** Everything the notebook produces is written here:

```
results/minimap2/genome/         genome alignments (one sorted BAM + .bai per library)
results/minimap2/transcriptome/  transcriptome alignments, split by route
results/stringtie/               the de novo assembly: denovo.gtf, denovo.fa
results/oarfish/                 transcript quantification, split by route
results/SUPPA2/                  events, PSI values and differential-splicing comparisons
```

Nothing here is tracked by git — the run produces about **2.6 GB**. Delete the whole folder
to start from a clean slate; the notebook recreates it.
