# Patches Applied to Upstream Matcher

This is a patched fork of aim-uofa/Matcher (https://github.com/aim-uofa/Matcher), created for the CS 4782 Final Project (Spring 2026, Cornell University).

Three modifications were applied to make the codebase work on current Colab Pro (PyTorch 2.x, NumPy >=1.24):

## Patch 1 - NumPy 1.24+ Compatibility

Files affected: Matcher.py, Matcher_SemanticSAM.py  
Change: np.int -> int (NumPy 1.24 removed the deprecated alias)  
Rationale: Without this patch, the code crashes with AttributeError on import.

## Patch 2 - Ablation Hook for Bidirectional Matching

File: Matcher.py (around line 174)  
Change: Added env-var gate. When DISABLE_REVERSE=1 is set in the environment, all forward-matched points are retained (bypassing the cycle-consistency filter). Realizes paper Table 4b (forward-only ablation).  
Rationale: Cleanest possible ablation - main run and ablation differ in one environment variable, no other hyperparameters touched.

## Patch 3 - Buffered Logging (notebook-level)

Where: In the Colab notebooks, all run commands are prefixed with stdbuf -oL python -u ... instead of bare python.  
Rationale: Drive-mounted output files do not get flushed by default; without buffering control, logs are lost on session timeout.

## Reproducing the Patches

To apply these patches yourself starting from the upstream repo:

    git clone https://github.com/aim-uofa/Matcher.git
    cd Matcher
    cp -r path/to/cs4782-matcher/code/matcher/* matcher/

The rest of the upstream Matcher repository (configs/, datasets/, dinov2/, segment_anything/, etc.) is used unchanged.
