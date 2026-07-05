---
description: Generate a Slurm batch script or idev command for a TACC job
---

The user wants to run a job on TACC via Slurm.

Using the `tacc-hpc-guide` skill knowledge:

1. Ask/confirm: which system, CPU or GPU, node count, walltime, allocation name, and what command to run.
2. For quick/interactive work, give the right `idev -p <queue> -N <n> -n <n> -t <hh:mm:ss>` line.
3. For batch work, produce a complete `sbatch` script with `#SBATCH` directives (-J, -o, -p, -N, -n, -t, -A), the needed `module load`, environment activation, and the run command.
4. Include the submit/monitor cycle: `sbatch`, `squeue -u <user>`, `scancel`.
5. Remind them to load `tacc-apptainer` if running containers and to keep I/O on `$SCRATCH`.

Use any details supplied in `$ARGUMENTS` to fill the script; otherwise use clearly-marked placeholders.

$ARGUMENTS
