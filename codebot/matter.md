# Writing a Matter App

Load this card when a task uses physical or virtual Matter devices: a
`LibertasDevice` or `LibertasVirtualDevice` field and its descriptor, Matter
requests, reports or callbacks, device subscriptions, virtual-device responses,
or the handling of a Matter status. Its rules are generated from the
[device descriptor encodings](../reference/matter/descriptors.md) and the
[Matter interaction/status contract](../reference/matter/status.md), which own
them. Use the installed public SDK for exact function and tuple shapes; the
[annotation inventory](schema_annotations.md) gives each descriptor
annotation's Rust and TypeScript spelling, and the Rust manual's
[Matter SDK dependency](../reference/languages/rust.md#matter-sdk-dependency)
covers a new Rust Matter App.

`LibertasDevice` selects a physical device; `LibertasVirtualDevice` identifies a
task-owned Matter server. The descriptor's byte encoding, load and access
meaning, and matching belong to Studio's editors and the clients that decode
them.

## Order of work

1. Declare the device fields and their Studio-generated descriptors
   (`matter.descriptor.*`).
2. Use the typed SDK and register callbacks before requests
   (`matter.interaction.*`).
3. Submit each task's complete subscription list (`matter.subscription.*`).
4. Handle statuses (`matter.status.*`), including status-only completion.
5. Plan for command and attribute-write supersession (`matter.cmd.*`,
   `matter.write.*`).
6. For a virtual device, answer through the SDK (`matter.virtual.*`).
7. Compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=matter -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Matter device type attribute encodings

Owner: [descriptors.md#matter-device-type-attribute-encodings](../reference/matter/descriptors.md#matter-device-type-attribute-encodings)

- `matter.descriptor.purpose`: The descriptor is metadata used by the Hub and clients to filter a physical device or construct a task-owned virtual device.
- `matter.descriptor.payload-not-runtime-id`: This encoding does **not** describe the field's runtime object ID; that value is separate and is never part of the attribute payload.

### Schema mapping

Owner: [descriptors.md#schema-mapping](../reference/matter/descriptors.md#schema-mapping)

- `matter.descriptor.use-editor` (judgment): Studio's Device Type Editor and Virtual Device Type Editor generate these strings. Application code should normally use those editors instead of assembling binary descriptors manually.

### Excluded: runtime field value

Owner: [descriptors.md#excluded-runtime-field-value](../reference/matter/descriptors.md#excluded-runtime-field-value)

- `matter.descriptor.no-runtime-id`: The `DeviceType` and `VirtualDevice` attributes contain only descriptor metadata. They never contain the selected or allocated Libertas object ID carried by the field at runtime. Runtime value serialization is therefore outside the attribute encoding specified by this document.

### Libertas Matter client status contract

Owner: [status.md#libertas-matter-client-status-contract](../reference/matter/status.md#libertas-matter-client-status-contract)

- `matter.status.u32`: Libertas exposes Matter interaction results to TypeScript and Rust applications as one unsigned 32-bit value. This is an application-facing Libertas contract; it is not the two-field `StatusIB` encoding used on the Matter wire.
- `matter.status.bits.interaction`:

  | Bits | Meaning |
  | --- | --- |
  | 0-7 | Matter Interaction Model status |

- `matter.status.bits.cluster`:

  | Bits | Meaning |
  | --- | --- |
  | 8-15 | Optional Matter cluster status; zero when absent |

- `matter.status.bits.hub`:

  | Bits | Meaning |
  | --- | --- |
  | 16-31 | Libertas Hub status |

- `matter.status.zero-success`: Zero is success.
- `matter.status.test-whole-value` (judgment): A client that cannot recover differently for individual errors should test the complete value for zero rather than decoding its fields.

### App interaction invariants

Owner: [status.md#app-interaction-invariants](../reference/matter/status.md#app-interaction-invariants)

- `matter.interaction.typed-sdk` (judgment): Use the installed public SDK's typed Matter requests, reports, and response shapes.
- `matter.interaction.callbacks-and-transactions`: Register the device callback before issuing requests, preserve each transaction ID, and branch on the documented action and transaction semantics before decoding a callback.
- `matter.interaction.report-not-response`: A report or the ignored transaction ID is not automatically a response to the latest request.
- `matter.interaction.status-only`: An interaction may complete with a terminal error status even when no attribute, event, or command payload is returned; status-only completion is still a result and must not be discarded while waiting for payload data.
- `matter.virtual.respond-to-request`: A virtual device returns each response through the SDK with the exact peer and transaction ID from the request.
- `matter.virtual.standard-status-only`: It may return only standard Matter status bits 0-15; it never manufactures a Libertas Hub status.

### Task subscription replacement

Owner: [status.md#task-subscription-replacement](../reference/matter/status.md#task-subscription-replacement)

- `matter.subscription.task-list-replacement`: Matter device subscriptions belong to the application-layer integration. Each subscription update is sent in one shot for one App task: collect its complete desired list of device subscriptions and submit that list together. A later accepted list completely replaces that task's previous list, including removal of omitted devices and paths. Tasks may replace their list repeatedly; separate per-device submissions do not accumulate subscriptions. An empty list clears that task's device subscriptions.

### Hub statuses

Owner: [status.md#hub-statuses](../reference/matter/status.md#hub-statuses)

- `matter.status.superseded`:

  | Hub code | Encoded client value | Name | Meaning |
  | --- | --- | --- | --- |
  | `0x0001` | `0x00010000` | Superseded | A queued command was replaced, or a retained attribute write differs from the value finally written. |

- `matter.status.superseded-constant` (judgment): `Superseded` is available as `LibertasMatterHubStatus.Superseded` in the TypeScript SDK and `libertas_matter::frame::STATUS_SUPERSEDED` in the Rust SDK. Applications use that constant rather than copying the numeric value.
- `matter.status.hub-bits-origin`: Only the Hub generates bits 16-31, and only Libertas Matter client APIs carry them.
- `matter.virtual-device.standard-status`: A virtual Matter device is a Matter server and must return a standard Matter status using bits 0-15 only.

### Command supersession

Owner: [status.md#command-supersession](../reference/matter/status.md#command-supersession)

- `matter.cmd.newest-per-source.authoring`: Use the installed SDK completion contract: [only the newest retained request per source is completed](../reference/matter/status.md#command-supersession), and writes report status per attribute.
- `matter.cmd.report-independent`: Attribute reports follow ordinary Matter subscription behavior independently; command completion does not establish report-delivery ordering.
- `matter.cmd.reissue` (judgment): Applications can handle the rejection and decide whether to issue a new command.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Schema mapping

Owner: [descriptors.md#schema-mapping](../reference/matter/descriptors.md#schema-mapping)

- `matter.descriptor.host`: DeviceType goes only on an effective Device and VirtualDevice only on an effective VirtualDevice; a List inherits neither
- `matter.descriptor.nonempty-and-placement`: A device descriptor attribute needs a non-empty value and cannot be package-level

<!-- /generated -->
