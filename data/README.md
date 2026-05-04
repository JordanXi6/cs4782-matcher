# Data - FSS-1000

This project uses **FSS-1000** (Few-Shot Segmentation 1000), released by Li et al., CVPR 2020.

## Download

Original repository: https://github.com/HKUSTCV/FSS-1000

The dataset contains 1,000 object classes, each with 10 images and 10 corresponding binary segmentation masks.

## Expected Directory Structure

After download, organize as:

```
data/fss1000/
  train/  (520 classes)
  val/    (240 classes)
  test/   (240 classes; we use fold 0 = first 20 classes)
```

Each class folder contains 10 images and 10 corresponding masks.

## Fold Splits

The 240 test classes are split into 12 folds of 20 classes each. We evaluate on **fold 0** only due to compute constraints. The fold assignment is the canonical one used in the Matcher codebase (see code/matcher/data/fss.py).

## License

FSS-1000 is released for non-commercial research use only. See the original repository for full license terms.
