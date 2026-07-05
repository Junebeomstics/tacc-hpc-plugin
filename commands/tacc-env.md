---
description: Set up a Python/conda or Apptainer container environment on TACC
---

The user wants to set up a research environment on a TACC system.

Using the `tacc-hpc-guide` skill knowledge, help them:

1. Choose `$SCRATCH` (or `$WORK`) for the environment — never `$HOME`.
2. Pick venv vs. personal Miniconda, and give exact `module load` + create/activate commands.
3. For containers: `idev` → `module load tacc-apptainer` → `apptainer pull docker://...`, with `--nv` for GPUs.
4. Warn about Vista's Arm (aarch64) architecture if the target is Vista — x86 wheels/conda won't work; suggest aarch64 builds or NGC containers.
5. Suggest a reproducible container approach for neuroimaging/brainlife.io workflows when relevant.

Tailor to any system/framework named in `$ARGUMENTS`.

$ARGUMENTS
