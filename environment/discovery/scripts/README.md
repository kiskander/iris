# Script Discovery

Drop executable discovery scripts into this folder and IRIS will run them during initialization.

This is for operators who have CLI-only access to their network and already have working automation scripts — netmiko, Napalm, Nornir, paramiko, or raw SSH wrappers. Instead of rewriting that tooling as an MCP, point IRIS at the scripts you already trust.

## What belongs here

Executable scripts that query the network and output what they find:

- Python scripts using netmiko, Napalm, Nornir, or paramiko
- Bash scripts that wrap SSH or vendor CLI tools
- Anything else executable that reads network state and prints the result

IRIS runs whatever it finds here during initialization. The operator is responsible for what goes in this folder — IRIS does not write, modify, or generate scripts.

## The output contract

Scripts must print structured output to stdout so IRIS can parse the results. JSON is preferred. The expected shape:

```json
{
  "devices": [
    {
      "hostname": "core-01",
      "role": "edge",
      "platform": "ios-xe",
      "management_ip": "10.0.0.1"
    }
  ],
  "topology": [
    {
      "from": "core-01",
      "from_interface": "Ethernet0/1",
      "to": "core-02",
      "to_interface": "Ethernet0/0"
    }
  ],
  "addressing": {
    "loopbacks": [
      {"device": "core-01", "address": "1.1.1.1/32"}
    ],
    "links": [
      {"subnet": "10.1.1.0/30", "purpose": "core-01 to core-02"}
    ]
  },
  "autonomous_systems": [
    {"asn": 65001, "devices": ["core-01", "core-02"]}
  ]
}
```

Any field can be empty or omitted. Partial output is better than no output. IRIS merges what it gets from each script with findings from other discovery sources.

If a script cannot produce JSON, plain text describing what it found is acceptable as a fallback. IRIS will parse it as best it can and flag anything ambiguous.

## Credentials

Credentials come from environment variables. Never hardcode credentials in scripts. Never log credentials in script output. Never include credentials in the JSON returned to IRIS.

Common pattern:

```python
import os
username = os.environ["NET_USERNAME"]
password = os.environ["NET_PASSWORD"]
```

The operator is responsible for setting these environment variables in their shell before running IRIS. Typically in `~/.zshrc`, `~/.bashrc`, or a `.env` file sourced at session start.

If a required environment variable is missing, the script should exit with a clear error message. IRIS will surface the error to the operator rather than trying to proceed with incomplete credentials.

## What scripts should not do

These scripts are for discovery only. They must be read-only.

- No configuration changes
- No pushes
- No restarts
- No interface shutdowns
- Nothing that modifies device state

If a script in this folder makes changes, the operator put it there intentionally. IRIS will not stop it, but this is not the pattern this folder is designed for. Changes should happen during the normal ticket workflow with engineer approval — not during discovery.

## Naming conventions

Name scripts descriptively:

- `discover-devices.py`
- `pull-bgp-neighbors.sh`
- `napalm-inventory.py`

IRIS runs every executable file in this folder. Non-executable files are ignored, so keep helper modules or config files separate or make them non-executable with `chmod -x`.

## Execution order

IRIS runs scripts in alphabetical order. If order matters — for example, one script depends on output from another — prefix filenames with numbers:

- `01-discover-devices.py`
- `02-discover-topology.py`
- `03-discover-bgp.py`

Otherwise, scripts run independently and their outputs get merged.
