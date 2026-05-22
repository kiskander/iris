# Pre-Change Checks

Common pre-condition patterns organized by change type. Load this file when SKILL.md references it during step 3 of the workflow.

This list is a starting point, not an exhaustive reference. Apply judgment. If the change you are working on does not fit neatly into one of these categories, fall back to first principles: what does this change depend on being true, and how do you verify each dependency before acting?

## Reporting a Blocker

When a pre-condition fails and you need to stop, report in this shape:

**Blocker:** one sentence.

**Evidence:**
- device: command → result
- device: command → result

**Why proceeding is unsafe:** one or two sentences.

**Recommendation:** what to do next.

Do not enumerate every related observation. If something is worth flagging but not blocking, hold it for the operator's follow-up question. If they ask "what else did you notice," answer then.

## Routing and Advertisement Changes

Changes that add, modify, or remove routing information.

**Read before you check:**

Pull these configuration sections from the affected device so your checks run with full context:

- The routing protocol configuration relevant to the change (`router bgp`, `router ospf`, `router isis`, static routes)
- Any route-maps referenced by the neighbor, interface, or redistribution statement involved
- Prefix-lists, community-lists, or AS-path access-lists referenced by those route-maps
- The specific address-family block if the change is address-family-scoped
- Redistribution statements if the change touches cross-protocol route injection

**Before advertising a prefix into a protocol:**

- Confirm the prefix is present in the routing table on the advertising device
- Confirm the prefix is reachable — not just present as a static or null route
- Check whether the prefix is already being advertised by another device, which could create routing inconsistency
- If advertising into BGP, confirm the network statement or redistribution source matches the actual prefix in RIB

**Before modifying a route:**

- Confirm the current route is what the ticket assumes it to be
- Identify all devices that learn this route and how
- Check for any dependent static routes, policies, or ACLs referencing this prefix

**Before removing a route or network statement:**

- Confirm nothing else depends on the route being present
- Check downstream devices to see what they would lose visibility to
- Look for policy, ACL, or redistribution references to the prefix

## Peering and Session Changes

Changes that affect BGP, OSPF, IS-IS, or other neighbor relationships.

**Read before you check:**

Pull these configuration sections before running the checks below:

- The full neighbor configuration including peer group membership
- Any inbound and outbound route-maps or filter-lists applied to the neighbor
- Authentication configuration for the session
- Timers, hold-time, and keepalive settings if the change could flap the session
- Any BFD or fast-failover configuration tied to this session

**Before modifying a neighbor:**

- Confirm the current session is in the state the ticket assumes
- Confirm reachability to the new peer IP before changing configuration
- Check authentication parameters — mismatched passwords are silent failures
- Identify what prefixes are currently exchanged over this session

**Before removing a neighbor:**

- Identify what prefixes come in via this neighbor
- Check whether any of those prefixes are unique to this session or available via alternate paths
- Confirm redundancy before tearing down the only path to a destination

**Before changing AS numbers or router IDs:**

- Understand the blast radius — this touches every session on the device
- Plan for session flaps and temporary loss of routes
- Confirm the change window accounts for convergence time

## Interface and Link Changes

Changes that affect physical or logical interfaces.

**Read before you check:**

Pull these configuration sections before running the checks below:

- The full interface configuration including description, IP, MTU, and encapsulation
- Any subinterfaces, bridge-groups, or VLAN assignments tied to the parent interface
- Routing protocol configuration referencing this interface (network statements, OSPF area assignment, passive-interface declarations)
- Any ACLs, QoS policies, or service policies applied to the interface
- Any channel-group or port-channel membership

**Before modifying an interface:**

- Confirm the interface is in the expected state — up/up, administratively down, role
- Identify what runs over the interface — routing protocols, circuits, VLANs, trunks
- Check for any dependency on the interface remaining in its current configuration

**Before shutting down an interface:**

- Identify what traffic flows through it
- Confirm redundancy exists for anything depending on the link
- Check whether shutting down this interface will drop any neighbor sessions

**Before removing an interface configuration:**

- Confirm no routing neighbors are currently established over the interface
- Confirm no VLANs, subinterfaces, or bridge groups depend on the parent config

## Policy and ACL Changes

Changes to filtering, route-maps, prefix-lists, or access control lists.

**Read before you check:**

Pull these configuration sections before running the checks below:

- The full current definition of the policy, ACL, route-map, or prefix-list being modified
- Every place the policy is applied — interfaces, neighbors, redistribution points, class-maps, other policies
- Any policies that reference the one being changed (nested policy chains)
- Any counters or hit-count information if the platform supports it, to understand whether the policy is actively matching traffic

**Before modifying a policy:**

- Understand what the policy currently does and where it is applied
- Identify all attachments — interfaces, neighbors, redistribution points
- Predict the impact on traffic or routing behavior and confirm the ticket intends that impact

**Before adding a deny or block:**

- Confirm you understand what is currently being permitted that you are about to block
- Check whether any legitimate traffic or control plane sessions depend on the behavior you are changing

**Before removing a policy:**

- Confirm whatever the policy was protecting or controlling is still protected another way, or the protection is no longer needed

## Configuration Cleanup and Decommission

Changes that remove configuration believed to be unused.

**Read before you check:**

Pull these configuration sections before running the checks below:

- The full configuration of the object being removed
- A search across the device configuration for any reference to the object by name
- The running configuration on related devices if the decommission could affect them
- Any documentation or tickets that reference the object, if accessible through connected tools

**Before removing configuration:**

- Confirm the configuration is actually unused — no neighbors established, no traffic matching, no references from elsewhere
- Search the rest of the device for any reference to the object being removed — names in ACLs, groups, policies
- Check related devices for references — a decommission often has dependencies on devices the ticket does not mention

**Before decommissioning a circuit or device:**

- Map every service, route, or dependency that currently uses the target
- Check redundancy for each dependency
- Confirm nothing silent — monitoring probes, backup paths, lab gear — relies on the target being available

## General Principles

When the change type does not fit a category above, reason from first principles:

- What does this change depend on being true?
- What will this change affect that is not explicitly mentioned?
- What verifies that the dependencies are actually met?
- If the dependency check fails, what happens if I proceed anyway?

If the answer to that last question is "something breaks silently" — stop and flag it. That is the black hole pattern. Loud failures are recoverable. Silent failures are not.

Before declaring something unreachable, broken, or failing, verify the failure is what you think it is. A symptom can have multiple causes — a failed connectivity test could mean a policy is dropping traffic, a route is missing, or the destination does not exist. Check the originating side and the destination side before concluding which link in the chain is broken.
