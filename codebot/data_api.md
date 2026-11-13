# Persistent data

Load this card when a task declares a function's storage schema or reads or
writes persistent records: key and value unions, key layouts, the
transactional key-value API, and data-failure handling. It applies also to an
App with no Endpoint. Its rules are generated from the
[persistent data schemas contract](../reference/protocols.md#persistent-data-schemas),
which owns them. Exact API signatures belong to the installed public SDK and
the [Rust storage section](../reference/languages/rust.md#persistent-data-schemas);
published roots evolve under the
[append-only rule](../reference/data-type-evolution.md#append-only-evolution),
and persisted `FormattedText` follows the
[payload guidance](../reference/presentation.md#persisting-formatted-record-payloads).

## Order of work

1. Declare the storage schema on every task function (`data.schema.*`).
2. Design the key union and its fixed-width fields (`data.key.*`), putting a
   person's `User` first when the App stores per-person data
   (`data.key.per-user-prefix`).
3. Read and write through targets and transactions (`data.kv.*`,
   `data.names.*`).
4. Decide data-failure handling: the default stop, or an explicit
   `DataErrorStatus` opt-in (`data.failure.*`, `data.error-status.*`), then
   compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=data-api -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Persistence failure and the no-status API

Owner: [protocols.md#persistence-failure-and-the-no-status-api](../reference/protocols.md#persistence-failure-and-the-no-status-api)

- `data.failure.stop-task`: By default, **a Data API operation succeeds or the Hub runtime stops the affected task**. Reads can also report ordinary absence. Writes and removals retain their existing no-status signatures. Failures must never appear as success, missing records, empty collections, or default values. Termination is owned and enforced by the Hub runtime, not by an SDK or App recovery handler.
- `data.failure.third-party-enable` (judgment): Only a third party may issue `Enable` after a data failure; preserving the configured enabled state never schedules it automatically. If restarted after storage repair, the App follows ordinary startup and loads persisted data. No additional recovery logic is required for an ordinary App.

### Explicit error-status opt-in

Owner: [protocols.md#explicit-error-status-opt-in](../reference/protocols.md#explicit-error-status-opt-in)

- `data.error-status.checked-requires-marker`: The opt-in permits checked counterparts of every Data API operation: `try…` methods in TypeScript and `try_…` methods in Rust. They perform the same operation and return its status synchronously or through the async result. Existing ordinary calls retain their behavior, including Hub-owned termination on failure. Using a checked call without the marker is a task error, not an alternative way to bypass termination. A matching Hub and SDK with checked-operation support are required.
- `data.error-status.return-shapes`: Rust checked calls return `Result<T, DataError>`; missing records are `Ok(None)`. TypeScript checked calls return `{ status: DataStatus.Success, value: T }` or `{ status: errorStatus }`; a missing record is successful `undefined`. Writes and removals return a successful unit/void value. Removing an absent record is successful.
- `data.error-status.no-partial-results` (judgment): Failure never carries usable partial read results. A failure is not a durability receipt or a guarantee that retrying an operation is safe.
- `data.error-status.app-owns-consistency` (judgment): An opted-in App owns deciding how to proceed after a reported failure and must preserve its own consistency: it cannot acknowledge failed persistence as saved data or treat failure as absence. The flag does not repair storage, provide a cross-call transaction, or authorize automatic task restart.

### Storage schema declarations

Owner: [protocols.md#storage-schema-declarations](../reference/protocols.md#storage-schema-declarations)

- `data.schema.types-only`: TypesOnly packages have no task functions requiring this declaration.
- `data.key.system-identities`: System identity fields (`User`, `Device`, `VirtualDevice`, `Endpoint`, and `LanDevice`) retain their semantic types and use the signed 64-bit key codec: big-endian with the sign bit flipped, occupying eight bytes and preserving numeric order. `User` also represents group IDs. No `NumberType` attribute is needed. The same representation applies through named aliases; nullable identity fields add the ordinary one-byte presence flag.
- `data.key.durable-identities` (judgment): Negative IDs are runtime dynamic identities, not durable references; durable keys must use a stable App identity or a durable allocated system ID.
- `data.key.exact-size`:

  | Attribute | Meaning in a key |
  | --- | --- |
  | Explicit `SizeMin=N, SizeMax=N` | Exact N-byte String/Bytes payload without a length suffix. |

- `data.key.minimum-size`:

  | Attribute | Meaning in a key |
  | --- | --- |
  | `SizeMin<M`, with `SizeMax=M` | Minimum present payload length; the bounded field still occupies M + 4 bytes. |

- `data.key.nullable-slot`:

  | Attribute | Meaning in a key |
  | --- | --- |
  | `Nullable=true` | One presence byte plus the complete fixed payload slot. Absence bypasses content bounds. |

- `data.key.utf8-bytes`: Key strings and ordinary value String size constraints both count UTF-8 bytes under the [shared size contract](../reference/schema.md#43-value-constraint-and-presentation-attributes).
- `data.key.no-layout-attributes`: Do not add a discriminator, byte-order, or padding attribute: variant order, type, and the common codec determine them.
- `data.schema.append-only`: Published key and value roots follow the [append-only evolution rule](../reference/data-type-evolution.md#append-only-evolution). Existing key payload layouts, including field unions and their bounds, stay fixed.
- `data.key.byte-order`: SQLite and TiKV compare the same encoded keys in unsigned lexicographic byte order. Compare the first unequal byte; if every shared byte matches, the shorter key comes first. Key equality is encoded-byte equality. No locale, database collation, or host endianness participates. Avro is the value/message codec; the ordered key codec has its own representation.
- `data.key.prefix-boundaries`: Each union supports discriminator values 0 through 65,535. Tags decide union order before payloads. Root variant lengths can differ, but each variant and every embedded field have a statically known width. Generated key length selection is O(1). A prefix at a field boundary selects all keys with the same leading fields; the root discriminator is part of that prefix.

### Per user key design pattern

Owner: [protocols.md#per-user-key-design-pattern](../reference/protocols.md#per-user-key-design-pattern)

- `data.key.per-user-prefix` (judgment): **Per-user key design pattern.** This applies only when an App stores data that belongs to a person; an App that keeps no per-person data adds no `User` field. When it applies, make the record owner's `User` the first field of every key variant that stores a person's data, directly after the root discriminator. Then each such variant keeps one user's records in one prefix, `(variant, user)`, so the App can find or remove all of a person's data at that person's request with one prefix target per variant. Putting all of a person's records under one variant, with the record kind as a later field, makes it a single prefix. A key whose owner appears after another field scatters that person's records across the key space.

### Storage schema declarations

Owner: [protocols.md#storage-schema-declarations](../reference/protocols.md#storage-schema-declarations)

- `data.key.float-and-string-exact`: Floating-point keys preserve every IEEE bit. Negative and positive zero are different keys; NaN signs and payloads remain distinct. Ordinary floating-point equality is therefore not key equality. Strings are neither normalized nor truncated. Reject values outside declared byte bounds.
- `data.key.dateonly`: DateOnly remains a Number with its declared fixed numeric representation; ordered `yyyyMMdd` values follow calendar order. Enumeration and List are not supported key field kinds.
- `data.key.decode-strict`: Decoders require the exact selected layout length, known tags, valid Boolean and presence bytes, valid UTF-8 for String, valid payload lengths, and zero padding. Malformed encodings are errors, never absent records. Apps can retain encoded scan keys without decoding them. Rust's separate encode/decode derives and buffer interfaces are specified in the [Rust storage section](../reference/languages/rust.md#persistent-data-schemas) and [encoding buffers](../reference/languages/rust.md#encoding-buffers).
- `data.schema.example-identity` (judgment): Replace the example developer identity before publication.
- `data.schema.external-unions`: Each target may be local or an external resolved App union. For payload design, follow the [formatted record-payload guidance](../reference/presentation.md#persisting-formatted-record-payloads).
- `data.schema.versionless-union`: Keep that persistent-data Union's name versionless and invariant across App versions. Append each new persistent payload type as a new final variant of the same Union so newer App versions continue to recognize all older stored values. Do not replace it with a version-named Union or mutate an existing payload case. See the [App data-type design guidelines](../reference/data-type-evolution.md).

### Transactional key-value API

Owner: [protocols.md#transactional-key-value-api](../reference/protocols.md#transactional-key-value-api)

- `data.api.matching-runtime-abi`: This specification describes the current development API and requires matching Hub and SDK builds with storage runtime ABI 0.4.0. The API is not yet released. There is one KV API; experimental predecessor APIs and encodings have no compatibility requirement.
- `data.kv.context`: A task accesses one schema-bound `Data<K,V>` context. Rust obtains it with `libertas::data::Data::<Keys, Values>::current()` after the task entry point's schema binding. TypeScript uses `libertasData<Keys, Values>()`. `K` is the key union and `V` is the persistent-value union. The host selects storage and owns namespace isolation and protection; Apps do not select tables or backends.
- `data.kv.target-exact-keys`: **Exact keys:** a nonempty set of complete encoded keys, strictly increasing in unsigned lexicographic byte order, with no duplicates. Absent declared keys are protected too. A scan returns only members of the declared set.
- `data.kv.target-prefix`: **Prefix:** every complete key beginning with the specified bytes, including absent keys. The empty prefix selects the entire task namespace. Unbounded scan endpoints mean the edges of this prefix; explicit bounds must stay within it.
- `data.kv.serializable-domains`: Target ordering is independent of batch mutation order. Keys within an explicit set are sorted before `begin`; the host acquires protection consistently across hosts. Transactions are serializable within their declared domain: reads use a stable view plus the transaction's own writes, and the prefix includes gaps where another writer could insert. The host may protect a larger domain, so Apps must not assume independent targets execute simultaneously. A synchronous call that would wait behind another transaction owned by the same runtime is rejected; use the asynchronous API for queued admission. Do not hold one transaction while waiting for another on the same task context.
- `data.kv.transaction-owner`: Transactions have one owner and execute operations sequentially. Their mode cannot be upgraded. A failed operation invalidates its transaction. Rust drops request rollback; TypeScript provides `cancel()` and should use an explicit commit/rollback or a `finally` cleanup. Cancellation cannot undo a commit that has already taken effect. A failed or cancelled commit does not prove that retrying the transaction is safe.
- `data.kv.scan-continuation`: Scan ranges have inclusive, exclusive, or unbounded endpoints, an ascending or descending direction, and a positive limit. Returned continuation tokens are opaque and bind the context, transaction, target, range, direction, and projection. Continue using the same parameters. Standalone pages each use a new transaction; keep the scan inside an explicit transaction when all pages need one stable view. Retain encoded keys for ordering and further operations; decoding keys is optional.
- `data.kv.async`: Rust's default core remains single-threaded and supports `no_std`. With the optional `tokio` feature, attach a `libertas::tokio::Dispatcher` on the task thread, then call `dispatcher.data(&data)` and move the returned cloneable handle into coroutines. The same methods become async, and transaction handles remain exclusive. TypeScript's `data.async()` returns the Promise API. Both forms use the same eleven primitives and persistence-failure policy. Owned Avro messages cross the dispatcher boundary; storage buffers are copied or decoded before reuse. Async storage operations allow the dispatcher to continue serving other work while storage executes. Their completion has the same durability and transaction guarantees as the synchronous API.
- `data.kv.limits`: The current host limits encoded keys to 4,096 bytes, values to 4 MiB, and command frames to 16 MiB. A key set, bulk read, or page contains at most 4,096 entries. Scan pages and bulk read values have an 8 MiB payload budget; a page may end before its requested item limit. Writes have no count limit: a write batch holds any number of mutations within its frame, and a transaction permits 16 MiB of submitted mutation bytes (keys plus values) and expires after 60 seconds. Hosts bound outstanding async requests to 256 per task context. Checked calls report `ResourceLimit` when applicable; ordinary failures follow the rule above.
- `data.kv.prefix-bytes`: `Target::<DataKey>::prefix(&prefix_bytes)` instead covers all keys beginning with those bytes. Construct bytes with the key codec and retain the leading field boundary required by the layout. An empty prefix (`Target::all()`) covers the whole task key space. Target construction checks ordering and limits; the host checks that every operation stays within its declared domain.

### Data and database names

Owner: [protocols.md#data-and-database-names](../reference/protocols.md#data-and-database-names)

- `data.names.key-identity`: A record's identity is its encoded schema key. Human-readable labels can be constructed from decoded key fields and the App's normal localized resources; a translation never changes key bytes. The API has one ordered key space per task context. A key-union variant supplies the record category, and its fixed fields supply any additional identity or ordering. There are no separate single-record and indexed-store namespaces in this API.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Storage schema declarations

Owner: [protocols.md#storage-schema-declarations](../reference/protocols.md#storage-schema-declarations)

- `data.key.variant-tags`: Key union variant indexes are two-byte u16 tags: at most 65,536 variants per union
- `data.key.fixed-width`: Every key field has a fixed width; recursive key layouts are invalid
- `data.key.number-type`: A key Number needs a fixed-width NumberType
- `data.key.size-max`: A key String or Bytes needs SizeMax; the field occupies SizeMax + 4 bytes
- `data.key.static-bounds`: Key bounds are static nonnegative integers within the 32-bit encoded-size limit
- `data.key.unsupported-fields`: Unbounded fields, variable collections, machine-width integers and unsupported types are rejected in keys
- `data.key.one-nullable-layer`: A key field has one nullable layer; nested Option and Box are invalid in Rust keys
- `data.key.rust-derives`: Rust key and value types carry the expected codec derives

<!-- /generated -->
