# cell_fate



## Run celloracle container
- module load singularity/3.11.3
- singularity pull docker://kenjikamimoto126/celloracle_ubuntu:0.18.0
- singularity exec --bind /share/crsp/lab/pkaiser/ddlin/cell_fate:/data celloracle_0.18.0.sif jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser
- Jupyter Server at http://127.0.0.1:8888/tree?token=XXXXXXXXXXXXXXXXXX

## Vectors
- pseudotime field alone (developmental roadmap)
- simulation field alone (effect of TF KO or overexpression)
- both fields + an inner-product heatmap to see where they align/conflict.

### CellOracle vs CellRank — a side-by-side comparison


| Dimension                         | **CellOracle**                                                                                                                                                                             | **CellRank (v2, 2024-)**                                                                                                                                                                                                                                                          |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Primary goal**                  | *Mechanistic* fate prediction via **in-silico TF perturbation** using an inferred gene-regulatory network (GRN).                                                                           | *Statistical* fate mapping that assigns **probabilities of ending in each terminal state** by modelling the cell-state transition process.                                                                                                                                        |
| **Key data inputs**               | scRNA-seq ± scATAC-seq (multi-ome optional). Needs genome annotation & motif database to build the GRN.                                                                                    | Any single-cell modality that can yield a **transition kernel**: RNA velocity, pseudotime, metabolic-label velocity, experimental time-series, etc. ([Nature][1])                                                                                                                 |
| **Core algorithm**                | 1. Infer a *directed* GRN per cell<br>2. Simulate expression change after virtual TF KO/OE<br>3. Project ∆expression back onto your embedding to visualise predicted shifts. ([Nature][2]) | 1. Build a Markov chain of cell-to-cell transition probabilities (mixing similarity and directional info)<br>2. Identify macrostates (initial & terminal)<br>3. Compute each cell’s *absorption probability* for every terminal fate. ([Nature][3], [cellrank.readthedocs.io][4]) |
| **What the vectors mean**         | Arrows show *simulated* direction a cell would move after a TF perturbation; can be compared with a gradient field of observed pseudotime.                                                 | Arrows (optional) visualise **expected future movement** derived from the transition matrix; main output is per-cell fate probability, not the quiver plot.                                                                                                                       |
| **Outputs most CAR-T groups use** | *Perturbation flow plots*, list of TFs whose KO steers cells toward/away from exhaustion or memory fate.                                                                                   | *Fate bias matrix* (e.g., prob. of each cell becoming effector vs. memory), *lineage-restricted gene trends*, *entropy / plasticity scores*.                                                                                                                                      |
| **Strengths**                     | • Explicit mechanistic hypotheses (GRN-based)<br>• Works even when RNA velocity is unreliable (e.g., slow-cycling CAR-T memory)\*<br>• Multi-omic friendly.                                | • Scalable (millions of cells, GPU support)<br>• Flexible directional evidence (not just velocity)<br>• Quantitative, uncertainty-aware fate probabilities and gene-trend analysis.                                                                                               |
| **Limitations**                   | • GRN inference is computationally heavy; quality depends on motif priors and ATAC depth.<br>• Gives *direction after perturbation*, but not a baseline numeric fate probability.          | • Needs at least one reliable directional signal (velocity or time) — pure snapshot data without splicing/time info yields only similarity-based diffusion.<br>• No mechanistic “why”; cannot predict TF-KO effects without extra analysis.                                       |
| **Typical runtime** (100 k cells) | 2–6 h on 32 GB RAM (GRN build dominates)                                                                                                                                                   | Minutes to <1 h with GPU/CPU cluster for kernel + absorption.                                                                                                                                                                                                                     |
| **Language / ecosystem**          | Python (Scanpy-compatible), tutorial notebooks, Docker image. ([morris-lab.github.io][5])                                                                                                  | Python, Scanpy-native; v2 API unifies multiple kernels, integrates with dynamo & velocyto outputs. ([cellrank.readthedocs.io][4])                                                                                                                                                 |
| **Licence & maintenance**         | BSD-3; active dev by Morris Lab (latest 0.10, Apr 2025).                                                                                                                                   | BSD-3; Heidelberg/Bergen group, CellRank 2 released Jan 2024, frequent updates. ([Nature][1])                                                                                                                                                                                     |

\*CAR-T exhaustion programs often have weak splicing kinetics, making RNA-velocity-based tools less reliable.

---

### Which one to pick for a CAR-T project?

| If you need…                                                                                                     | Choose…          | Why                                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **To rank transcription factors whose modulation could bias manufactured CAR-T cells toward a memory-like fate** | **CellOracle**   | Only tool that couples GRN inference with *virtual* KO/OE simulations, letting you test TFs *in silico* before CRISPR screens.                                                     |
| **To quantify, for every single cell after infusion, its probability of ending up exhausted vs. persisting**     | **CellRank**     | Absorption probabilities give you numerical fate bias that you can correlate with patient outcome or manufacturing variables.                                                      |
| **Both mechanistic TF insight *and* numeric fate bias**                                                          | **Combine them** | 1) Use CellRank to define initial vs. terminal macrostates and identify branch points; 2) Run CellOracle on branch-defining TFs to predict interventions that shift probabilities. |

---

### Practical integration workflow

1. **Pre-processing**: Standard Scanpy pipeline → UMAP, clusters.
2. **Directionality source**:

   * If you have good splicing → feed velocity to **CellRank**.
   * If velocity is noisy, use time-series labeling or pseudotime as CellRank kernels.
3. **CellRank**: Compute fate probabilities & identify key driver genes.
4. **CellOracle**: Build GRN, run *simulate\_shift()* on those drivers, visualise arrows to validate whether KO/OE reinforces the desired CellRank-predicted trajectory.
5. **Wet-lab follow-up**: Prioritise TFs whose simulation arrows push cells toward high-probability persistent states.

That combination gives you **numbers to monitor in patients** *and* **mechanistic levers to turn during manufacturing** — the sweet spot for translational CAR-T research.

[1]: https://www.nature.com/articles/s41592-024-02303-9?utm_source=chatgpt.com "CellRank 2: unified fate mapping in multiview single-cell data - Nature"
[2]: https://www.nature.com/articles/s41586-022-05688-9?utm_source=chatgpt.com "Dissecting cell identity via network inference and in silico gene ..."
[3]: https://www.nature.com/articles/s41592-021-01346-6?utm_source=chatgpt.com "CellRank for directed single-cell fate mapping | Nature Methods"
[4]: https://cellrank.readthedocs.io/en/latest/about/version2.html?utm_source=chatgpt.com "Moving to CellRank 2"
[5]: https://morris-lab.github.io/CellOracle.documentation/?utm_source=chatgpt.com "Welcome to CellOracle's documentation! - GitHub Pages"
