---
description: Show how to log into a TACC system (SSH + MFA) and set up allocations
---

The user wants help logging into a TACC (Texas Advanced Computing Center) system.

Using the `tacc-hpc-guide` skill knowledge, walk them through:

1. Confirm which system they target (Frontera / Vista / Stampede3 / Lonestar6 / Jetstream2) and pick the right SSH host.
2. Account + allocation prerequisites (TxRAS for UT/partners, ACCESS otherwise) and PI-adds-student rule.
3. MFA pairing on the TACC User Portal.
4. The exact `ssh myuser@<host>.tacc.utexas.edu` command and the password → 6-digit token flow.
5. Remind: you land on a login node — do heavy work via `idev`/`sbatch`, not there.

If the user provided a username or system in `$ARGUMENTS`, tailor the commands to it.

$ARGUMENTS
