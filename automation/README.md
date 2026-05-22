# Automation

This folder is where you give IRIS access to your operational automation — scripts, playbooks, and anything else that executes real work on the network.

## What belongs here

Drop your automation artifacts into `scripts/`. Anything IRIS can execute:

- Python scripts (netmiko, Napalm, SDK wrappers, API calls)
- Ansible playbooks
- Bash scripts
- Executable HTTP request collections

IRIS reads what you drop in and learns what each artifact does. Once learned, the artifact becomes part of IRIS's toolbelt — usable during normal ticket workflows when the task matches.

## How IRIS learns your automation

When you add a new artifact and tell IRIS to learn it, IRIS follows the workflow in `references/learn-automation.md`:

1. Reads the artifact
2. Identifies the type and what dependencies it needs
3. Installs missing dependencies
4. Summarizes what the artifact does, what parameters it takes, what it returns
5. Appends the summary to `artifacts.md` after you confirm

IRIS does not modify your scripts. It only reads, understands, and catalogs them.

## Safety

Every artifact IRIS catalogs gets a declared safety classification — read-only or state-changing. State-changing artifacts always require explicit operator approval before IRIS runs them. Read-only artifacts can run as part of investigation without approval.

If IRIS cannot determine whether an artifact is safe, it marks it as state-changing and requires approval.

## Keeping artifacts current

If you modify an artifact after IRIS has learned it, tell IRIS to relearn it so the catalog stays accurate.
