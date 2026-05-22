# Documentation Discovery

Drop network documentation into this folder and IRIS will read it during initialization.

## What belongs here

Anything that describes your network:

- Markdown files with device inventory, topology notes, addressing plans
- PDFs of network diagrams or runbooks
- Text exports from Confluence, Notion, or other wikis
- Ansible inventory files and playbooks
- Terraform configurations
- Containerlab or similar topology definitions
- README files from your network automation repos
- Config snapshots or backups

Organize however makes sense to you. IRIS reads recursively.

## What does not belong here

- Credentials of any kind
- Private keys
- Anything you would not commit to version control

IRIS reads these files as context. If something sensitive is in here, it enters the model's context window. Treat this folder like a public repo even if it is not.

## How IRIS uses what it finds

During initialization, IRIS reads the contents of this folder and extracts what it can about:

- Devices and their roles
- Topology and connectivity
- Addressing and autonomous systems
- Naming conventions
- Operational procedures or quirks

IRIS treats documentation as reference material, not authoritative truth. If live discovery (via MCP or scripts) contradicts what documentation says, IRIS flags the conflict rather than silently picking one.

If documentation is the only discovery source available, IRIS uses it but notes in the environment notes that findings are documentation-derived and may not reflect current state.

## Keeping documentation fresh

Documentation goes stale. IRIS does not refresh your docs for you — it just reads what is here when asked to initialize or refresh. Update this folder when your network changes, or pair it with live discovery sources that stay current on their own.
