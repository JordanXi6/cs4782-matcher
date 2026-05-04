# FSS-1000 One-Shot Segmentation - Reproduction Results

## Setup
- **Benchmark**: FSS-1000, fold 0 (20 test classes, 120 episodes per class, 2400 total)
- **Hardware**: NVIDIA L4 GPU (Google Colab Pro)

## Results

| Setting                                 | mIoU (Ours) | mIoU (Paper) | FB-IoU (Ours) |
|------------------------------------------|-------------|--------------|----------------|
| Matcher (full, bidirectional matching)   | **87.33**     | 87.0         | 91.73          |
| Matcher w/o bidirectional (forward-only) | **80.77**     | 81.1         | 86.21          |
| **Bidirectional matching contribution**  | **+6.56** | **+5.9**     | -              |

*Note: Paper's 87.0 is the mean across all FSS-1000 folds. We evaluated only fold 0 due to compute constraints; per-fold variance in the paper is ~1 mIoU.*
