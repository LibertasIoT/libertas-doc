# App network access

<!-- rule: net.default-denied class=hard enforced=no tasks=network-access -->
App task processes have no local or external TCP/UDP access by default. The
Hub permits IP networking only when the App function carries `AccessHost`. <!-- rule: net.function-attribute class=hard enforced=no tasks=network-access title="AccessHost is a function attribute: each function is its own task-process identity" -->
This is a function attribute because each function is a distinct task-process
identity and may need a different network boundary.

## Declaration

The attribute value is one comma-delimited list with no whitespace:

```text
AccessHost=api.weather.example,telemetry.example
```

TypeScript uses
`@LibertasAccessHost("api.weather.example,telemetry.example")`; Rust uses
`#[libertas_access_host("api.weather.example,telemetry.example")]`.

<!-- rule: net.declare.reason-key class=hard enforced=yes tasks=network-access title="Every AccessHost entry is an exact package StringResources key with nonempty localized text explaining the access" -->
Every entry must be an exact key in the package-owned `StringResources`
catalog. Its non-empty localized text explains why the function needs that
access. <!-- rule: net.declare.unique-hosts class=hard enforced=yes tasks=network-access title="AccessHost hostnames are unique without regard to DNS case" --> Hostnames are unique without regard to DNS case. <!-- rule: net.declare.hostname-only class=hard enforced=yes tasks=network-access title="An AccessHost hostname has no scheme, port, path, wildcard, IP literal or whitespace" --> A normal entry is a
DNS hostname, not a URL: schemes, ports, paths, wildcards, IP literals, and
whitespace are invalid.

<!-- rule: net.approval.raw-host-key class=hard enforced=no tasks=client-runtime -->
Control clients present `AccessHost` entries in the same explicit task-access
approval view as function `Permissions`. For every host entry, the UI always
shows the raw key verbatim—for example, `api.weather.example`—and separately
shows its localized `StringResources` value as the reason. Localized text never
replaces or hides the host identity because the user must be able to verify the
actual network name being approved. <!-- /rule --> Permission identities and their localized
reasons are shown in the same way: raw permission key as summary, followed by
that App's explanation. The explanation is App-owned because different Apps can
request the same permission for different reasons.

The two reserved entries are:

- `{network_access_local}` — allow all local addresses;
- `{network_access_external}` — allow all non-local addresses.

They are reserved string-resource names and still must be declared in
`StringResources` with [localized explanations](#declaration). <!-- rule: net.reserved.used-only-by-access-host class=hard enforced=yes tasks=network-access title="Reserved network-access StringResources keys require a function AccessHost use and cannot serve as unrelated text" --> Studio rejects either reserved
key when no function uses it in `AccessHost`; it cannot be reused as unrelated
application text. Declaring both permits local and external networking. <!-- rule: net.reserved.hostname-precedence class=hard enforced=no tasks=network-access --> A
concrete declared hostname is permitted even when it resolves into the other
address class; explicit hostname access takes precedence over a broad
local/external exclusion.

For this policy, local addresses comprise loopback, link-local, multicast,
limited broadcast, and the active local-interface subnets/prefixes. External
means addresses outside that set. The Hub's authenticated Unix-domain control
channel is not IP networking and is unaffected.

## Runtime enforcement and DNS changes

<!-- rule: net.runtime.tcp-udp-every-port class=hard enforced=no tasks=network-access -->
A host grant permits both TCP and UDP on every port. The policy covers the
whole task process tree, including native networking calls in every supported
runtime. It does not depend on using a particular SDK API. Without
`AccessHost`, local and external IP networking remain denied.

<!-- rule: net.runtime.dns-refresh class=hard enforced=no tasks=network-access -->
The Hub resolves concrete declared names to IPv4 and IPv6 addresses and keeps
the task's hostname view read-only. It periodically resolves names while the
process runs. Newly returned addresses become accessible; previous answers
remain accessible until that process ends. A temporary lookup failure preserves
the previously resolved set. Other addresses remain denied except for an
explicitly declared broad local/external grant.

<!-- rule: net.runtime.fresh-process class=hard enforced=no tasks=network-access -->
The accumulated addresses are discarded when the process ends. A publication
or configuration reload changing `AccessHost` starts a fresh task process, so
removed declarations and historical answers cannot survive that change. A
reload with unchanged declarations may reuse the process and its accumulated
addresses. <!-- rule: net.runtime.address-based class=judgment enforced=no tasks=network-access --> This is address-based enforcement: a hostname grant is not a
restriction to one application protocol, port, or service sharing that address.
