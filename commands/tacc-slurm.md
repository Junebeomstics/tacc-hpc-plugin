---
description: Generate a Slurm batch script or idev command for a TACC job
---

The user wants to run a job on TACC via Slurm.

Using the `tacc-hpc-guide` skill knowledge:

1. Ask/confirm: which system, CPU or GPU, node count, walltime, allocation name, and what command to run.
   If the user states an **experiment purpose** (e.g. "debug", "single-GPU training", "large
   multi-GPU foundation model", "tractography sweep", "high-memory"), use the skill's
   "Queue rules & recommendation" table to recommend the best queue (and note the SU cost /
   cheapest option). Prefer a `*-dev` queue for quick tests; flag Vista `gh` as best $/GPU for
   large DL. Note the rules are dated 2026-07-05 and tell them to run `qlimits` for live limits.
2. For quick/interactive work, give the right `idev -p <queue> -N <n> -n <n> -t <hh:mm:ss>` line.
3. For batch work, produce a complete `sbatch` script with `#SBATCH` directives (-J, -o, -p, -N, -n, -t, -A), the needed `module load`, environment activation, and the run command.
4. Include the submit/monitor cycle: `sbatch`, `squeue -u <user>`, `scancel`.
5. Remind them to load `tacc-apptainer` if running containers and to keep I/O on `$SCRATCH`.
6. Differentiate per system (see the skill's Portability section): pick the correct queue
   name for `-p` (e.g. Lonestar6 `gpu-a100`, Vista `gh`, Stampede3 `pvc`/`skx`, Frontera
   `rtx`/`normal`), and flag the two hard breakpoints — **Vista is Arm (aarch64)** so x86
   wheels/containers won't run, and **Stampede3 GPUs are Intel (PVC)/oneAPI, not CUDA**, so
   `--nv`/CUDA builds don't apply there.

Use any details supplied in `$ARGUMENTS` to fill the script; otherwise use clearly-marked placeholders.

$ARGUMENTS
