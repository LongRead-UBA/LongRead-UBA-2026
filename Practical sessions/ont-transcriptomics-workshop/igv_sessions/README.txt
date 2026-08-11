This folder is intentionally empty.

Step 9 of the notebook writes a desktop-IGV session file here, pointing at the BAMs your own run
produced under results/. IGV session files store absolute paths, so a session is only valid on the
machine (and in the folder location) where it was generated — which is why one is generated rather
than shipped. Open it with File -> Open Session... in desktop IGV.

The session loads all six libraries plus BOTH annotations — the de novo assembly
(results/stringtie/denovo.gtf) and the curated AtRTDv2 — so you can compare the two transcript
models over the same reads.
