# IRIS Automation Learning Workflow

How IRIS learns a new automation artifact and catalogs it in `automation/artifacts.md`. Run this workflow when the operator adds a new file to `automation/scripts/`, when the operator explicitly asks IRIS to learn something, or when IRIS notices a new or modified artifact at session start.

## Principles

Learning is observation. IRIS reads the artifact and describes what it finds. If behavior is unclear, ask the operator rather than guessing.

Learning is safe. Reading an artifact does not run it. Installing dependencies is the only side effect, and IRIS announces what it is about to install before doing so.

Learning is incremental. New artifacts get appended to `artifacts.md`. Modified artifacts get updated. Removed artifacts get noted but not silently deleted — the operator confirms removal.

## Workflow

### Step 1 — Identify the artifact

Read the file IRIS is being asked to learn. Determine:

- Type — Python script, Ansible playbook, bash script, API request collection, or other
- What it does — intent, scope, and the network operation it performs
- Parameters it takes
- Outputs it produces
- Whether it modifies state or is read-only

If any of these cannot be determined from reading the file, ask the operator.

### Step 2 — Identify and install dependencies

Determine what the artifact needs to run:

- Python scripts — required packages, Python version
- Ansible playbooks — required collections, Ansible version
- Bash scripts — required CLI tools
- API collections — required environment variables or credentials

List what is needed, check what is already installed, and install what is missing. Announce each install before running it. Never install silently.

If a dependency cannot be installed automatically — requires privileged access, requires a license, requires manual configuration — stop and tell the operator what needs to happen.

### Step 3 — Classify safety

Declare the artifact as one of:

- **Read-only** — queries state, does not modify anything
- **State-changing** — modifies device configuration, pushes changes, creates or deletes resources

If the classification is ambiguous, default to state-changing. IRIS errs on the side of caution.

### Step 4 — Summarize for the catalog

Produce a catalog entry with:

- Artifact name and path
- Type
- Purpose — one sentence describing what the artifact does
- Parameters
- Expected output
- Safety classification
- Dependencies required to run
- When IRIS should propose using this artifact (which kinds of tickets or tasks it matches)

### Step 5 — Confirm with the operator

Present the proposed catalog entry to the operator. Ask for corrections or additions. The operator may add context IRIS could not infer — usage patterns, known limitations, escalation notes.

### Step 6 — Append to the catalog

After operator confirmation, append the entry to `automation/artifacts.md`. Update the status block at the top with the new count and timestamp.

### Step 7 — Acknowledge

Tell the operator the artifact has been learned and summarize what IRIS can now do with it.

## Handling changes

If IRIS detects an artifact has been modified since it was last learned, offer to relearn it. Do not automatically overwrite the existing catalog entry — confirm with the operator first.

If an artifact has been removed from `scripts/` but still exists in `artifacts.md`, flag the discrepancy. Offer to remove the entry or mark it as archived.

## When the artifact is opaque

If IRIS cannot determine what an artifact does — the code is heavily obfuscated, the playbook uses unfamiliar custom modules, the script wraps an opaque binary — ask the operator to describe the artifact in plain language. Write the operator's description into the catalog entry and note that the entry is operator-supplied rather than IRIS-derived.
