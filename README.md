# Reproducing Matcher: Training-Free One-Shot Segmentation

CS 4782 Advanced Deep Learning - Final Project, Spring 2026

Authors: Jiyao Xi, Bingling Liu, Jingzei Dai (Cornell University)

## 1. Introduction

This repository reproduces Matcher (Liu et al., ICLR 2024), a training-free framework for one-shot semantic segmentation that composes two frozen vision foundation models - DINOv2 (patch-level features) and SAM (class-agnostic segmentation) - via three deterministic matching stages.

The paper claims state-of-the-art performance on FSS-1000, COCO-20i, and LVIS-92i without updating any model parameter. We reproduce its FSS-1000 result and the critical bidirectional-matching ablation, and contribute a distributional analysis the paper underdiscusses.

Paper: Liu, Zhu, Li, et al. *Matcher: Segment Anything with One-Shot Using All-Purpose Feature Matching.* ICLR 2024. [arXiv:2305.13310](https://arxiv.org/abs/2305.13310)

## 2. Chosen Result

We reproduce two specific numbers from the paper:

| Setting | Paper | Ours | Reference |
|---|---|---|---|
| Matcher full (with bidirectional matching) on FSS-1000 | 87.0 mIoU | **87.33 mIoU** | Paper Table 1 |
| w/o reverse matching (forward-only ablation) | 81.1 mIoU | **80.77 mIoU** | Paper Table 4b |
| Bidirectional matching contribution (delta) | +5.9 | **+6.56** | Derived |

Both reproduction numbers are within 0.33 mIoU of the paper, well inside their reported per-fold variance.

Beyond the paper, our distributional analysis reveals:

- Per-class standard deviation: 8.06 -> 19.80 (2.46x increase) when reverse matching is removed
- Worst-class IoU drop: 64.9 -> 26.1 (38.8 absolute IoU loss for the catastrophically failing class)
- 5 of 20 classes are *better* without bidirectional matching (it is not Pareto-improving)

These findings reframe bidirectional matching as a tail-risk reducer / cycle-consistency outlier filter, not a uniform feature improvement.

## 3. GitHub Contents

    cs4782-matcher/
    |-- README.md                  (this file)
    |-- LICENSE                    (MIT)
    |-- .gitignore
    |-- code/
    |   |-- matcher/               (patched Matcher Python module)
    |   |   |-- Matcher.py         (contains DISABLE_REVERSE env-var gate)
    |   |   |-- Matcher_SemanticSAM.py
    |   |   |-- PATCHES.md         (documents all modifications)
    |   |   `-- ...
    |   |-- notebooks/             (Colab notebooks for reproduction)
    |   |   |-- 01_run_main.ipynb
    |   |   |-- 02_run_ablation.ipynb
    |   |   `-- 03_analyze_results.ipynb
    |   `-- requirements.txt
    |-- data/
    |   `-- README.md              (FSS-1000 download instructions)
    |-- results/
    |   |-- logs/                  (raw stdout from main and ablation runs)
    |   |-- tables/                (per-class CSV, summary statistics)
    |   `-- plots/                 (mIoU comparison, per-class bar chart)
    |-- poster/
    |   `-- matcher_poster.pdf
    `-- report/
        `-- group_matcher_2page_report.pdf

## 4. Re-implementation Details

- Image encoder: DINOv2 ViT-L/14 (frozen)
- Segmenter: SAM ViT-H (frozen)
- Trainable parameters: 0
- Dataset: FSS-1000 fold 0 (20 test classes)
- Episodes: 120 (reference, target) pairs per class, 2,400 total
- Hardware: NVIDIA L4 GPU (Google Colab Pro)
- Hyperparameters: all fixed to authors' FSS-1000 defaults (alpha=0.8, beta=0.2, lambda=1.0, K=8 clusters, top-10 mask merging, coverage threshold 0.3, input 518x518). No tuning performed.

Patches to upstream Matcher (see `code/matcher/PATCHES.md`):

1. `np.int` -> `int` for NumPy 1.24+ compatibility
2. Environment-variable ablation hook (`DISABLE_REVERSE=1`) to bypass cycle-consistency filter
3. Notebook-level buffered logging (`stdbuf -oL python -u`) to handle Drive sync delays

## 5. Reproduction Steps

### Prerequisites

- Google Colab Pro account (or any environment with NVIDIA L4-class GPU, ~16 GB VRAM)
- Google Drive with ~10 GB free
- ~10 GPU-hours of compute total

### Setup

**1. Clone original Matcher and apply our patches:**

    git clone https://github.com/aim-uofa/Matcher.git
    cd Matcher
    cp -r /path/to/cs4782-matcher/code/matcher/* matcher/

**2. Download model checkpoints:**

- DINOv2 ViT-L/14: https://dl.fbaipublicfiles.com/dinov2/dinov2_vitl14/dinov2_vitl14_pretrain.pth (1.2 GB)
- SAM ViT-H: https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth (2.4 GB)

**3. Download FSS-1000:** see `data/README.md`

**4. Install dependencies:**

    pip install -r code/requirements.txt

### Running

5. Open `code/notebooks/01_run_main.ipynb` in Colab. Update path constants in the first cell. Run all cells. (~3.5 GPU-hours)

6. Open `code/notebooks/02_run_ablation.ipynb`. Same setup but with `DISABLE_REVERSE=1`. Run all cells. (~3.5 GPU-hours)

7. Open `code/notebooks/03_analyze_results.ipynb` to parse logs, compute per-class statistics, and regenerate plots.

## 6. Results / Insights

The reproduction faithfully matches the paper on aggregate metrics (delta < 0.33 mIoU on both Table 1 and Table 4b). Our substantive finding is that the aggregate mean obscures what bidirectional matching actually does:

- It is a tail-risk reducer, not a location shifter. Removing it nearly triples per-class SD; the mean shift is driven by extreme left-tail mass, not a uniform downward translation.
- It is not Pareto-improving. 5 of 20 classes are slightly better without it - a bias-variance trade-off the paper does not surface.
- Reframing: Matcher succeeds because cycle-consistency outlier filtering identifies and removes biased mass on the feature-similarity assumption. The two foundation models are not enough by themselves; the filter is what makes them compose at all on visually-ambiguous classes.

See `results/plots/` for visual evidence and `results/tables/` for per-class numbers.

## 7. Conclusion

The faithful reproduction validates the paper's headline claim. Our contribution is the distributional view: the paper reports only mean mIoU, which obscures a 2.46x increase in per-class variance when the cycle-consistency filter is removed. We recommend that future work on training-free foundation-model compositions report per-class distributions, not just aggregate means.

## 8. References

1. Liu, Y., Zhu, M., Li, H., Chen, H., Wang, X., Shen, C. *Matcher: Segment Anything with One-Shot Using All-Purpose Feature Matching.* ICLR 2024. [arXiv:2305.13310](https://arxiv.org/abs/2305.13310)
2. Oquab, M., et al. *DINOv2: Learning Robust Visual Features without Supervision.* arXiv:2304.07193, 2023.
3. Kirillov, A., et al. *Segment Anything.* arXiv:2304.02643, 2023.
4. Li, X., et al. *FSS-1000: A 1000-Class Dataset for Few-Shot Segmentation.* CVPR 2020.

## 9. Acknowledgements

This project was completed as part of CS 4782 Advanced Deep Learning at Cornell University (Spring 2026). We thank the course instructors and TAs for guidance, and the original Matcher authors for releasing their code publicly.
