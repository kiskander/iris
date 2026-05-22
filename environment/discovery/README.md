# Discovery

This folder is where IRIS learns about your network. During initialization, IRIS checks each source below and uses whatever is available. You do not need all three — use whatever matches the access you have.

## Sources

IRIS looks for three kinds of discovery input, in no particular order:

### 1. Connected MCPs

Defined in `.mcp.json` at the project root. If you have MCP servers for ticketing, device management, source of truth, or observability, IRIS uses them during discovery and during normal workflow execution.

This is the richest discovery source because it gives IRIS live queryable access. If you have MCPs available, connect them here first.

### 2. Documentation — `environment/discovery/docs/`

Drop network documentation into `environment/discovery/docs/`. Markdown, text files, PDFs, diagrams — anything that describes the network IRIS will be operating against. Network-as-code artifacts like Ansible playbooks, Terraform configs, or topology files also live here.

Use this when you have documented your network somewhere but don't have an MCP-style tool to query it directly. IRIS reads these files during initialization and uses them as a reference during normal workflows.

### 3. Executable scripts — `environment/discovery/scripts/`

Drop executable discovery scripts into `environment/discovery/scripts/`. Python, bash, or anything else that can run on the operator's machine. Credentials come from environment variables, never hardcoded.

Use this when all you have is CLI access to the devices and you already have working automation scripts — netmiko, Napalm, Nornir, raw SSH. IRIS runs these scripts during initialization, reads their output, and learns from what they return.

See the README files inside `docs/` and `scripts/` for the specific conventions each source expects.

## How IRIS uses what it finds

During initialization, IRIS checks all three sources and combines what it learns. The results get written to `environment/artifacts.md`.

If multiple sources describe the same thing with conflicting details, IRIS flags the conflict to the operator rather than picking a winner silently.

If no discovery sources are available, IRIS says so and asks the operator to point it at a source before proceeding.

## When you have nothing

If you have no MCPs, no documentation, and no scripts, IRIS cannot discover anything on its own. In that case the operator can describe the environment manually and IRIS will write that description into `environment/artifacts.md` as operator-supplied context. This is the least preferred path but it exists.
