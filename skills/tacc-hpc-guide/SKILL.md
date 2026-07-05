---
name: tacc-hpc-guide
description: >-
  Guide for working on TACC (Texas Advanced Computing Center) supercomputers.
  Use when the user asks how to choose a TACC system, log in, request an
  allocation, set up a Python/conda environment, run Apptainer/Singularity
  containers, choose the right Slurm queue for an experiment, or submit Slurm
  jobs on Frontera, Vista, Stampede3, Lonestar6, or Jetstream2. Also triggers on
  "TACC", "슈퍼컴퓨터", "supercomputer allocation", "which queue", "queue recommendation",
  "idev", "sbatch", "module load", "$SCRATCH", and reproducible-neuroscience /
  brainlife.io HPC questions.
---

# TACC HPC Guide

Actionable reference for using Texas Advanced Computing Center (TACC) systems.
Source: https://tacc.utexas.edu/systems/all/ and https://docs.tacc.utexas.edu/.
Verify exact specs/queues against each system's User Guide before production runs —
hardware and queue limits change with upgrades.

## 1. Pick the right system

| Workload | Recommended system | Why |
|----------|-------------------|-----|
| Large-scale DL training / foundation models | **Vista** (NVIDIA GH200 + H100) | AI-focused, Grace Hopper. **Arm (aarch64)** — needs Arm builds or NGC containers |
| GPU ML + simulation mix (UT/partner access) | **Lonestar6** (AMD EPYC Milan + A100 40GB) | Fast TxRAS access for UT-Austin/partners |
| Massive CPU parallelism (preprocessing, tractography sweeps) | **Stampede3** / **Frontera** | 140k+ / large core counts, Omni-Path / HDR InfiniBand |
| Reproducible, shareable, on-demand VMs | **Jetstream2** (OpenStack cloud) | Cloud-native workflows |
| CS / networking testbed | **Chameleon** | Reconfigurable OpenStack |
| Large dataset hosting / collaboration | **Corral** | Data management resource |
| Long-term archive | **Ranch** | Tape-based archiving |

`$WORK` is the **Stockyard** global filesystem shared across systems.
Decommissioned (do NOT target): Stampede2, Maverick2, Longhorn, Catapult, Fabric.

## 2. Access & allocation

1. Request an account: https://accounts.tacc.utexas.edu/register
   - PIs (faculty/research staff/postdocs) request allocations; students must be **added by a PI**.
2. Request an allocation:
   - **TxRAS** — UT Austin / UT System / TACC partner institutions.
   - **ACCESS** — all other US academic researchers.
   - **Frontera** — separate NSF PRAC guidelines.
3. **MFA is mandatory**: pair a token in the TACC User Portal (Account Profile → Multi-Factor Authentication → Manage).

## 3. Log in (SSH + MFA)

```bash
ssh myuser@frontera.tacc.utexas.edu   # Frontera
ssh myuser@ls6.tacc.utexas.edu        # Lonestar6
ssh myuser@stampede3.tacc.utexas.edu  # Stampede3
ssh myuser@vista.tacc.utexas.edu      # Vista
# → password → 6-digit MFA token
```
You land on a **login node**. Do NOT run heavy compute there (builds/short tests only) —
real work goes to compute nodes via `idev` or `sbatch`.

## 4. Filesystems (critical)

| Var | Use | Notes |
|-----|-----|-------|
| `$HOME` | source, configs | small, backed up |
| `$WORK` | project data/software (Stockyard) | large, **no backup**, cross-system |
| `$SCRATCH` | job I/O, environments | largest, **purged periodically**, fastest |

Install conda/venv envs and training data in **`$SCRATCH`** (or `$WORK`), never `$HOME`.
Move important results off `$SCRATCH` before purge (to `$WORK`/`$HOME`/Ranch).

## 5. Environment — Lmod modules

```bash
module avail            # what's available
module spider python    # search a package + versions/deps
module load gcc python3 # load
module purge            # clean slate
module save mymods      # save / restore mymods
```

## 6. Python / conda

```bash
# venv (lightweight, recommended) — run on a compute node
module load python3
python3 -m venv $SCRATCH/envs/myproj
source $SCRATCH/envs/myproj/bin/activate
pip install torch nibabel dipy numpy

# or personal Miniconda in $SCRATCH
cd $SCRATCH && wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p $SCRATCH/miniconda3
source $SCRATCH/miniconda3/etc/profile.d/conda.sh
conda create -n neuro python=3.11 -y && conda activate neuro
```
**Vista is Arm (aarch64)**: x86 wheels/conda pkgs won't work — use aarch64 builds or NVIDIA containers.

## 7. Containers — Apptainer (formerly Singularity)

Available on **compute nodes only**. The reproducibility backbone (runs Docker images without root).

```bash
idev -m 40                    # interactive compute session
module load tacc-apptainer
apptainer pull docker://godlovedc/lolcow   # Docker → .sif
apptainer shell lolcow_latest.sif
apptainer exec --nv image.sif python train.py   # --nv exposes GPUs
```
brainlife.io Apps and benchmarks (e.g. TractSeg) ship as Docker images — pull with
`apptainer pull docker://...` for identical results across laptop and cluster.

## 8. Slurm jobs

Interactive (`idev`):
```bash
idev -p gpu-a100-dev -N 1 -n 1 -t 1:00:00   # Lonestar6 A100 dev queue, 1 node, 1h
```

Batch (`sbatch`):
```bash
#!/bin/bash
#SBATCH -J myjob
#SBATCH -o myjob.o%j
#SBATCH -p gpu-a100        # partition/queue (CPU: normal, etc.)
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 04:00:00
#SBATCH -A MyAllocation    # allocation/project name

module load tacc-apptainer
source $SCRATCH/envs/myproj/bin/activate
python train.py
```
```bash
sbatch job.slurm          # submit
squeue -u myuser          # status (or showq)
scancel <jobid>           # cancel
# find your running node: squeue -u myuser → NODELIST → ssh <node>
```
Remote viz: `sbatch /share/doc/slurm/job.vnc` (or `job.dcv`).
Parallelize many serial tasks: `module load pylauncher`.

## 9. Portability — what carries across systems vs. what must change

The command **interface** is TACC-wide; the **values** are per-system. When generating
a script, always fill the per-system fields and check the two hard breakpoints below.

**Portable as-is (TACC conventions, same everywhere):**
`ssh user@host` + MFA · `$HOME`/`$WORK`/`$SCRATCH` (note: `$WORK`/Stockyard is shared
across systems) · Lmod (`module load/spider/avail/purge`) · `idev`/`sbatch`/`squeue`/`scancel`
· `module load tacc-apptainer` · the conda/venv-on-`$SCRATCH` pattern.

**Must be set per system:**

| Field | Frontera | Stampede3 | Lonestar6 | Vista |
|-------|----------|-----------|-----------|-------|
| SSH host | `frontera` | `stampede3` | `ls6` | `vista` |
| CPU arch | x86 (Intel) | x86 (Intel) | x86 (AMD) | **Arm aarch64** |
| GPU / toolchain | NVIDIA RTX / CUDA | **Intel Max (PVC) / oneAPI** | NVIDIA A100 / CUDA | NVIDIA H100 / CUDA |
| Example queue `-p` | `normal`, `rtx` | `skx`, `icx`, `pvc` | `normal`, `gpu-a100` | `gh`, `gg` |
| `-A <allocation>` | project-specific | project-specific | project-specific | project-specific |

**Two hard breakpoints — an x86+NVIDIA script does NOT transfer:**
1. **Vista is Arm (aarch64).** x86 pip wheels, conda packages, and `.sif` container images
   won't run. Use aarch64 builds or NVIDIA NGC/ARM containers.
2. **Stampede3 GPU vendor depends on the queue.** `pvc` = Intel Max (oneAPI/SYCL; CUDA and
   `apptainer --nv` do NOT apply); `h100` = NVIDIA (CUDA/`--nv` apply). (Lonestar6, Vista,
   Frontera are all CUDA.)

Rule of thumb: **x86 + NVIDIA systems (Frontera ↔ Lonestar6)** reuse environments and
containers well; crossing into **Vista (Arm)** or **Stampede3 GPU (Intel)** requires
rebuilding the environment.

**Outside TACC:** `idev`, `tacc-apptainer`, `$SCRATCH` setup, and MFA are TACC-specific.
Generic Slurm (`sbatch`/`squeue`), Lmod, and container concepts port to most HPC clusters;
the specifics do not.

## 10. Queue rules & recommendation (as of 2026-07-05)

> Queues, limits, and charge rates **change without notice** — run `qlimits` on the target
> system for live values. Below is the 2026-07-05 snapshot from TACC docs. Billing unit:
> 1 SU = one node-hour; **15-min (0.25h) minimum charge**; no node sharing (whole node billed).

**Lonestar6** — `development`(8N/2h/1SU), `normal`(64N/48h/1SU), `large`(256N/1SU),
`gpu-a100`(8N/3SU), `gpu-a100-dev`(2N/2h/3SU), `gpu-a100-small`(1N/1.5SU), `gpu-h100`(1N/6SU), `vm-small`(0.143SU).
**Frontera** — `development`(40N/2h/1SU), `normal`(3–512N/1SU), `large`(2048N/1SU),
`flex`(128N/**0.8SU**, pre-emptible), `rtx`(16N GPU/3SU), `rtx-dev`(2N/2h/3SU), `nvdimm`(4N/2SU).
**Stampede3** — CPU: `skx`(256N/1SU), `skx-dev`(16N/2h/1SU), `icx`(32N/1.5SU), `spr`(HBM/2SU), `nvdimm`(4SU);
GPU: `pvc`(**Intel** Max/3SU), `h100`(**NVIDIA**/4SU).
**Vista** (Arm) — `gg`(Grace-Grace CPU/32N/0.33SU), `gh`(Grace-Hopper H100/64N/**1SU**), `gh-dev`(8N/2h/1SU).

**Recommend a queue by the user's experiment purpose:**

| Purpose | Recommend | Why |
|---------|-----------|-----|
| Quick debug / interactive test | any `*-dev` queue (`development`, `skx-dev`, `gpu-a100-dev`, `rtx-dev`, `gh-dev`) + `idev` | 2h cap, short wait |
| Single-GPU DL train/infer (small) | LS6 `gpu-a100-small` (1.5 SU, cheapest 1-GPU) or `gpu-a100` | cost-efficient |
| Large multi-GPU DL / foundation model | **Vista `gh`** (up to 64 GPUs, 1 SU) > Stampede3 `h100` > LS6 `gpu-a100` | GPU density, best $/GPU on Vista |
| Large CPU parallel (tractography sweep, batch preprocessing) | Stampede3 `skx`/`icx`, Frontera `normal`/`large`, LS6 `normal`/`large` | hundreds of CPU nodes |
| Many small jobs / cost-saving | Frontera `flex` (0.8 SU, pre-emptible) + `pylauncher` | lowest charge |
| High-memory (large volumes/graphs) | Frontera `nvdimm`, Stampede3 `nvdimm`/`spr`(HBM) | high mem/node |
| Intel GPU / oneAPI experiments | Stampede3 `pvc` | SYCL, not CUDA |

Cost order (per node-hour): `gpu-h100`(6) > S3 `h100`(4) > `gpu-a100`/`pvc`(3) > `nvdimm`(2) >
Vista `gh`(1)/CPU `normal`(1) > `flex`(0.8) > Vista `gg`(0.33). **Vista `gh` runs H100 at 1 SU —
best value for large DL (needs Arm builds).** Always confirm the user's `-A` allocation covers the system.

## 11. Common pitfalls

- Running compute on login nodes → account suspension.
- Putting envs on `$HOME` → quota errors mid-job.
- Assuming x86 on Vista → binary incompatibility.
- Leaving results on `$SCRATCH` → purged/lost.
- Forgetting `-A <allocation>` on a multi-allocation account.

## References

- All Systems: https://tacc.utexas.edu/systems/all/
- Docs: https://docs.tacc.utexas.edu/
- Containers@TACC: https://containers-at-tacc.readthedocs.io/
- Getting Started: https://tacc.utexas.edu/use-tacc/getting-started/
