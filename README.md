# tacc-hpc — Claude Code plugin

A Claude Code plugin that helps you work on **TACC (Texas Advanced Computing Center)**
supercomputers: choosing a system, logging in, requesting allocations, setting up
Python/conda and Apptainer/Singularity environments, and submitting Slurm jobs.
Tuned for reproducible neuroimaging / AI research workflows (e.g. brainlife.io,
tractography, deep-learning benchmarks).

## What's inside

| Component | Purpose |
|-----------|---------|
| `skills/tacc-hpc-guide` | Auto-triggered knowledge skill: system selection table, login (SSH+MFA), filesystems, Lmod, conda, Apptainer, Slurm, pitfalls |
| `skills/tacc-claude-code-setup` | Auto-triggered knowledge skill: install & run Claude Code on a TACC login node (Node.js via nvm, npm prefix fix, allow-scripts caveat, SSH auth) |
| `/tacc-login` | Walk through SSH + MFA login and allocation prerequisites |
| `/tacc-env` | Set up a `$SCRATCH` venv/conda or Apptainer container environment |
| `/tacc-slurm` | Generate an `idev` command or a complete `sbatch` batch script, and recommend the right queue for your experiment |

## Systems covered

Compute: **Frontera**, **Vista** (Arm / Grace Hopper, AI), **Stampede3**, **Lonestar6** (A100), **Jetstream2** (cloud), Chameleon, Cyclone.
Storage: **Stockyard** (`$WORK`), **Corral**, **Ranch**.

## Install

Add the marketplace/repo, then install the plugin:

```
/plugin marketplace add Junebeomstics/tacc-hpc-plugin
/plugin install tacc-hpc
```

Or clone into your Claude Code plugins directory and enable it.

## Usage

Just ask naturally — the skill auto-activates on TACC topics:

- "How do I log into Lonestar6?"
- "Set up a PyTorch conda env on Stampede3"
- "Write an sbatch script for a 4-hour A100 job on Lonestar6"
- "Which queue should I use for large multi-GPU foundation-model training?"

Or use the slash commands: `/tacc-login ls6`, `/tacc-env vista pytorch`, `/tacc-slurm`.

## Sources

- https://tacc.utexas.edu/systems/all/
- https://docs.tacc.utexas.edu/
- https://containers-at-tacc.readthedocs.io/

Specs and queue limits change with hardware upgrades — always confirm against the
current per-system User Guide before production runs.

## License

MIT
