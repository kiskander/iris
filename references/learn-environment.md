# IRIS Discovery Workflow

How IRIS learns about an environment and keeps its understanding current. This workflow runs automatically the first time IRIS is invoked in a new deployment. It also runs when the operator explicitly asks IRIS to discover, or when IRIS notices that what it sees in the live environment meaningfully contradicts what is in the environment notes.

## Principles

Discovery is observation. Everything written to `environment/artifacts.md` must come from observed sources — connected MCPs, operator-supplied documentation, or executable discovery scripts. If a piece of information cannot be confirmed from any source, leave it out or note it as unknown.

Discovery is safe. It runs read-only queries against MCPs, reads files from the documentation folder, and executes scripts from the scripts folder. No configuration changes. No ticket modifications. No pushes of any kind.

Discovery is diff-aware. When environment notes already exist, discovery compares what it finds now against what it already knows. It surfaces what is new, what has changed, and what has disappeared. It does not overwrite blindly.

Discovery is verifiable. Before committing any updates to the environment notes, surface what was learned to the operator so they can confirm or correct.

## Discovery Sources

IRIS looks for three kinds of input during discovery. Use whatever is available — you do not need all three.

1. **Connected MCPs** — defined in `.mcp.json`. Live queryable access to ticketing, device management, source of truth, and observability tools.
2. **Documentation** — files in `environment/discovery/docs/`. Markdown, text, PDFs, network-as-code artifacts.
3. **Executable scripts** — files in `environment/discovery/scripts/`. Python, bash, or other executables that query the network and print structured output.

Details on each folder's conventions live in the README inside that folder.

## When Discovery Runs

Discovery runs in three situations:

**First run** — environment notes contain only the placeholder. IRIS discovers everything, writes the notes, then continues with the operator's original request. Automatic. No explicit command required.

**Explicit discover** — operator runs `/iris discover` or tells IRIS to rediscover the network. IRIS runs full discovery and performs a diff against existing notes.

**Drift detection** — during normal workflow execution, IRIS observes something that meaningfully contradicts what environment notes describe. IRIS flags the discrepancy and asks the operator whether to run discovery. IRIS does not refresh autonomously.

## Workflow

### Step 1 — Inventory the discovery sources

Before anything else, check what discovery input is available:

- **MCPs** — for each connected MCP, run a minimal health query to confirm it is reachable. Record which categories are available: ticketing, device management, source of truth, observability.
- **Documentation** — check `environment/discovery/docs/` for any files beyond the README. If files exist, note what is there.
- **Scripts** — check `environment/discovery/scripts/` for any executable files beyond the README. If scripts exist, note them.

Report what was found and what is missing. Missing sources are not a blocker — IRIS can still operate with whatever subset is available — but the operator should know what IRIS can and cannot see.

If all three sources are empty, stop and tell the operator IRIS has nothing to discover from. Ask them to connect an MCP, add documentation, or drop a discovery script before proceeding.

### Step 2 — Discover the device inventory

Pull device inventory from whatever sources are available:

- **From source-of-truth MCP** (Netbox or similar) — list of devices IRIS is expected to work with, including hostname, role, platform, management address.
- **From device management MCP** (CML, Meraki, Catalyst Center) — enumerate what the tool can see live.
- **From documentation** — extract device names and roles from markdown, inventory files, Ansible hosts files, Terraform configs.
- **From discovery scripts** — execute any scripts that produce device inventory output. Parse the JSON or text they return.

Merge findings across sources. If sources conflict — different hostname for the same device, different role assignment — flag the conflict rather than picking a winner silently.

### Step 3 — Discover the topology

For each reachable device or documented device, capture:

- Neighbor relationships — CDP, LLDP, or equivalent (from MCP or script)
- Documented topology — diagrams, link tables, architecture notes (from docs)
- Routing protocols in use and neighbor sessions

Build a minimal topology map. Enough to know what connects to what and what role each device plays. Exhaustive detail is not required.

### Step 4 — Discover the addressing and AS plan

Capture the autonomous systems and key prefixes:

- Loopbacks
- Link subnets
- Significant prefixes — customer-facing, transit, internal

Source these from source-of-truth tools, documentation, or script output. Skip exhaustive enumeration. Focus on architectural-level addressing.

### Step 5 — Discover the ticketing context

If a ticketing MCP is connected, identify:

- Which queue or assignment group handles network operations
- Ticket categories IRIS is expected to handle
- Workflow conventions — status transitions, required fields, closure criteria

If no ticketing MCP is connected, skip this step and note in the environment notes that IRIS cannot read tickets autonomously in this deployment.

### Step 6 — Diff against existing notes

If `environment/artifacts.md` already contains populated content (not the placeholder), perform a diff between what was just discovered and what is currently documented. Categorize the findings:

- **Unchanged** — already in the notes, still accurate. Do not touch.
- **New** — discovered now, not in the notes. Candidate for addition.
- **Changed** — in the notes but with different values now (new IP, different role, different platform). Candidate for update.
- **Missing** — in the notes but not found during discovery. Candidate for removal or marking as unreachable.

If this is first-run discovery, treat everything as new.

### Step 7 — Summarize and confirm

Present the diff to the operator as a clear summary:

- What is new since last discovery
- What has changed
- What appears to be missing
- What is unchanged (brief mention, not exhaustive)
- Tools and sources consulted
- Any conflicts between sources

Ask the operator to confirm, correct, or add context. For first-run discovery, also ask for items that could not be observed — change windows, escalation contacts, known quirks.

The operator can accept all changes, reject specific changes, or add operator-supplied context that discovery could not find on its own.

### Step 8 — Update the notes

After operator confirmation, update `environment/artifacts.md` with the confirmed changes only. Append what is new. Update what has changed. Note what is missing rather than silently deleting it — missing items may be temporarily unreachable, not actually gone.

Include:

- A timestamp for when this discovery completed
- An `Initialized: Yes` status field
- A `Last discovery` field with the current timestamp
- A brief note about what changed during this run (or "first discovery" for initial runs)
- A list of which discovery sources contributed to this run

### Step 9 — Acknowledge and move on

Tell the operator discovery is complete and summarize what changed. If the operator's original request was to work a ticket, proceed with that request now. If the request was `/iris discover`, wait for the next instruction.

## When nothing is available

If no MCPs are connected, no documentation exists, and no scripts are present, IRIS cannot perform discovery. In this case:

- Tell the operator plainly that discovery cannot proceed without at least one source
- Offer three options:
  - Connect an MCP and retry
  - Drop documentation or network-as-code files into `environment/discovery/docs/` and retry
  - Drop a discovery script into `environment/discovery/scripts/` and retry
- As a last resort, offer to write `environment/artifacts.md` from operator-supplied description only. Flag in the file that the contents are operator-described and not independently verified.
