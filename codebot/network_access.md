# Network access

Load this card when an App function's task process opens TCP or UDP
connections, directly or through a dependency, or when a task adds or changes
`AccessHost`. Its rules are generated from the
[App network access contract](../reference/network-access.md), which owns
them. The [annotation inventory](schema_annotations.md) gives the source
spelling.

## Order of work

1. Declare `AccessHost` on every function whose process needs the network
   (`net.default-denied`, `net.function-attribute`).
2. List each hostname or reserved broad grant with its reason
   (`net.declare.*`, `net.reserved.*`).
3. Check what a grant permits at runtime (`net.runtime.*`), then compile with
   Studio and fix every reported rule.

<!-- generated:codebot-rules task=network-access -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### App network access

Owner: [network-access.md#app-network-access](../reference/network-access.md#app-network-access)

- `net.default-denied`: App task processes have no local or external TCP/UDP access by default. The Hub permits IP networking only when the App function carries `AccessHost`.
- `net.function-attribute`: This is a function attribute because each function is a distinct task-process identity and may need a different network boundary.

### Declaration

Owner: [network-access.md#declaration](../reference/network-access.md#declaration)

- `net.reserved.hostname-precedence`: A concrete declared hostname is permitted even when it resolves into the other address class; explicit hostname access takes precedence over a broad local/external exclusion.

### Runtime enforcement and DNS changes

Owner: [network-access.md#runtime-enforcement-and-dns-changes](../reference/network-access.md#runtime-enforcement-and-dns-changes)

- `net.runtime.tcp-udp-every-port`: A host grant permits both TCP and UDP on every port. The policy covers the whole task process tree, including native networking calls in every supported runtime. It does not depend on using a particular SDK API. Without `AccessHost`, local and external IP networking remain denied.
- `net.runtime.dns-refresh`: The Hub resolves concrete declared names to IPv4 and IPv6 addresses and keeps the task's hostname view read-only. It periodically resolves names while the process runs. Newly returned addresses become accessible; previous answers remain accessible until that process ends. A temporary lookup failure preserves the previously resolved set. Other addresses remain denied except for an explicitly declared broad local/external grant.
- `net.runtime.fresh-process`: The accumulated addresses are discarded when the process ends. A publication or configuration reload changing `AccessHost` starts a fresh task process, so removed declarations and historical answers cannot survive that change. A reload with unchanged declarations may reuse the process and its accumulated addresses.
- `net.runtime.address-based` (judgment): This is address-based enforcement: a hostname grant is not a restriction to one application protocol, port, or service sharing that address.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Declaration

Owner: [network-access.md#declaration](../reference/network-access.md#declaration)

- `net.declare.reason-key`: Every AccessHost entry is an exact package StringResources key with nonempty localized text explaining the access
- `net.declare.unique-hosts`: AccessHost hostnames are unique without regard to DNS case
- `net.declare.hostname-only`: An AccessHost hostname has no scheme, port, path, wildcard, IP literal or whitespace
- `net.reserved.used-only-by-access-host`: Reserved network-access StringResources keys require a function AccessHost use and cannot serve as unrelated text

<!-- /generated -->
