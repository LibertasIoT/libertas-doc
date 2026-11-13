# Writing a calculation

Load this card when a task adds or changes a field calculation: a
`CalculationRequest` or `CalculationRoot`, a calculation Request and its
Response, calculation `CopyFrom` or `CopyTo` mappings, or a calculated allowed
set. Its rules are generated from the
[field calculation contract](../reference/protocols.md#field-calculation-requests)
and the [shared path contract](../reference/schema.md#libertas-schema-path),
which own them. The [annotation inventory](schema_annotations.md) gives each
annotation's Rust and TypeScript spelling.

## Order of work

1. Choose the root node of each calculation (`calc.root.*`, `calc.instance.*`).
2. Declare the calculation Request and its Response (`calc.target.*`).
3. Map the inputs from the root (`calc.inputs.*`, `calc.paths.*`, `paths.*`,
   `copy.*`) and declare participation on the triggers
   (`calc.participation.*`, `calc.invariant.*`, `calc.trigger.*`).
4. Map the outputs (`calc.copyto.*`, `calc.cycle.*`, `calc.projection.*`,
   `calc.union.*`), and for a calculated allowed set `calc.allowed-set.*`.
5. Check the design against the runtime rules (`calc.schedule.*`,
   `calc.queue.*`, `calc.submit.*`, `calc.apply.*`, `calc.removal.*`), then
   compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=calculation -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Presentation locks

Owner: [presentation.md#presentation-locks](../reference/presentation.md#presentation-locks)

- `tpl.lock.not-invariant`: **Not an invariant:** unlike `ReadOnly`, a lock never makes a [calculation input invariant](../reference/protocols.md#invariant-calculation-inputs); a locked input that triggers a calculation still declares `CalculationRequest`.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.calculation` (judgment): **Field values computed while editing:** a calculation Request that each changeable input field names in `CalculationRequest`; an [invariant input](../reference/protocols.md#invariant-calculation-inputs) needs none ([calculations](../reference/protocols.md#field-calculation-requests)).

### Calculation root

Owner: [protocols.md#calculation-root](../reference/protocols.md#calculation-root)

- `calc.root.invocation-only` (judgment): **Calculation only.** `CalculationRoot` governs only calculation invocation; a [link](../reference/protocols.md#links) to the same variant still binds from its host field or tapped node.

### Field calculation requests

Owner: [protocols.md#field-calculation-requests](../reference/protocols.md#field-calculation-requests)

- `calc.inputs.discriminators-and-fixed`: Native discriminators are transport metadata, not calculation inputs. Neither is a Fixed field: it is filled by ordinary initialization, never by copying editor data, so it needs no CopyFrom and is never a dependency.
- `calc.inputs.fixed-current-user` (judgment): A `Fixed("current-user")` User field is resolved from the authenticated session and verified by the Hub on the calculation request under the [security constraint](../reference/schema.md#fixed-current-user-security-constraint) as on any other request; every other Fixed input takes its declared value.
- `calc.trigger.nearest-root` (judgment): Every copied input is a calculation dependency. Every participating field is a **trigger**; the root need not be one. A request finds the **nearest root**: a trigger belongs to the instance rooted at its nearest self-or-ancestor occurrence that declares a root for that calculation. Roots of one calculation may nest; the nearest one wins. A change to a trigger re-runs its nearest root's instance **and** every other instance of that calculation whose bound inputs read that trigger occurrence, so an outer instance that reads a field inside an inner root through a whole List stays current.
- `calc.trigger.rootless` (judgment): A trigger with no root for its calculation at or above it re-runs every instance whose bound inputs read it; a trigger that is an ancestor of root occurrences, such as a List read as a whole, therefore reaches every root occurrence below it that reads it. A trigger never creates an instance. When one change schedules several instances of one type, they join that type's [deferred entry](../reference/protocols.md#changes-resets-and-deferred-calculation) in array order.

### Invariant calculation inputs

Owner: [protocols.md#invariant-calculation-inputs](../reference/protocols.md#invariant-calculation-inputs)

- `calc.invariant.reference-context` (judgment): a value in the interaction's [reference context](../reference/protocols.md#interaction-reference-context), whose reference objects the editor never changes. Only their replacement under that rule is an input change, as described under [changes](../reference/protocols.md#changes-resets-and-deferred-calculation).

### Field calculation requests

Owner: [protocols.md#field-calculation-requests](../reference/protocols.md#field-calculation-requests)

- `calc.invariant.never-triggers` (judgment): An invariant current-request input never triggers the calculation. Its value is read when a trigger schedules an instance and resolves from that instance's root occurrence, so a page-level hidden list is available to a per-row calculation rooted at the row. Declaring participation on an invariant field is unnecessary, and only makes it a trigger that never fires.
- `calc.inputs.reference-candidates`: Reference-object candidates follow the same [interaction context](../reference/protocols.md#interaction-reference-context): a Request variant is never a reference object, except the complete request that an EditRequest delivers, which is the reference object of its own editor, and a Response or SubscriptionData variant is a candidate only when its own NextRequest or WorkflowRequest actually offers the current request; Studio does not treat an unrelated variant that merely resolves the same path as a source.
- `calc.inputs.whole-container` (judgment): A containing Table or List may participate as a whole; changes below it are changes to that input.
- `calc.inputs.literal-indices` (judgment): Paths support named fields and zero-based literal array indices, such as `@.items[0].quantity`. Whole-list copies remain available. Indices denote positions, never persistent row identities.

### Typed ancestor paths and per-item calculations

Owner: [protocols.md#typed-ancestor-paths-and-per-item-calculations](../reference/protocols.md#typed-ancestor-paths-and-per-item-calculations)

- `calc.instance.per-root-occurrence.authoring`: Place CalculationRoot at the logical scope: [each occurrence has an independent instance](../reference/protocols.md#typed-ancestor-paths-and-per-item-calculations) whose current-request inputs and CopyTo destinations resolve from that root.
- `calc.union.inapplicable.authoring`: Use branch-specific CopyTo destinations knowing a Union-root calculation is [inapplicable when none has a destination](../reference/protocols.md#union-calculation-roots).

### Union calculation roots

Owner: [protocols.md#union-calculation-roots](../reference/protocols.md#union-calculation-roots)

- `calc.union.branch-alternatives` (judgment): **Union roots and inapplicable instances.** With a [Union root](../reference/protocols.md#calculation-root), inputs read branch fields through CopyFrom alternatives, such as `"@.Business.limit,@.Farm.limit"`. Only the selected branch resolves, so the first available value is that branch's value; no reduced Union copy is needed, and Union identity rules are unchanged.

### Typed ancestor paths and per-item calculations

Owner: [protocols.md#typed-ancestor-paths-and-per-item-calculations](../reference/protocols.md#typed-ancestor-paths-and-per-item-calculations)

- `calc.copyto.overlapping-writers` (judgment): Overlapping writers are permitted (the later applied assignment wins); a calculation instance writing its own array item and an enclosing whole-list write may both apply. Runtime checks still enforce actual values, bounds, and stale-instance ownership.

### Dynamic permissions and calculations

Owner: [protocols.md#dynamic-permissions-and-calculations](../reference/protocols.md#dynamic-permissions-and-calculations)

- `dynamic.calculation.lifecycle`: [Dynamic schema properties](../reference/schema.md#dynamic-schema-properties) read available data locally. App logic may supply their input values through ordinary rooted calculations; `Dynamic` itself names or invokes no Request. Its expression binds to its host occurrence, except List-field Removable binds to each member, even when the producing calculation has a different root. Existing input participation, output ownership, deferred execution, stale-response rejection, and submission gating apply. Dynamic Editable=false never qualifies as an invariant input.
- `dynamic.calculation.no-trigger`: Property changes never become calculation inputs or trigger requests. Calculations remain runnable while a control is unavailable. Authors provide an entry path through initial values, reference data, or an explicit local condition when a policy would otherwise depend on input that the same policy prevents entering. The App's save path remains authoritative.

### Reduced record and array copies

Owner: [protocols.md#reduced-record-and-array-copies](../reference/protocols.md#reduced-record-and-array-copies)

- `calc.projection.destination-type` (judgment): Calculation copies use the [shared field projection](../reference/schema.md#shared-copyfrom-and-copyto-rules). The destination type of calculation CopyFrom specifies the projection; no extra attribute is needed. A calculation input type selects the fields relevant to the calculation. Unrelated fields can be omitted even when complete or immutable, including large descriptive name strings that do not contribute to the result. Omitting these values reduces serialized payload size and avoids repeatedly transmitting them. A required field remains included regardless of its size or immutability.
- `calc.projection.construction.authoring`: Choose reduced input types deliberately: [only projected fields participate and block calculation](../reference/protocols.md#reduced-record-and-array-copies), while arrays retain every element in order.
- `calc.projection.reduced-copyto` (judgment): CopyTo may apply a reduced record to an existing record, or a reduced array to an existing array, under that same field projection. An existing destination field with an unavailable value is not an extra field. Reduced arrays require the original length and order, use zero-based positional correspondence, and cannot insert, remove, or reorder elements. Nested projections follow the same rule. Validate all mapped fields, bounds, protected destinations, and retained bindings before applying any assignment. A membership change invalidates a pending whole-array result. This is a merge into existing data, not permission to create missing parents or replace a union with an incompatible branch. Full-type assignments retain their existing meaning.

### Applying calculation results

Owner: [protocols.md#applying-calculation-results](../reference/protocols.md#applying-calculation-results)

- `calc.copyto.overlap-order` (judgment): Overlapping destinations are otherwise permitted: distinct calculations, a link return and a calculation, or assignments within one response may write the same or nested destinations. Assignments within one response apply in declaration order; distinct transactions apply in the order their replies are applied under the [ordering guarantee](../reference/protocols.md#request-and-response-ordering), so the later assignment wins. The platform guarantees that order and nothing more: the settled value of a destination that several calculations write follows the order of the edits that triggered them. Designing calculation requests whose overlapping results agree is the App developer's responsibility.
- `calc.copyto.readonly-writable` (judgment): ReadOnly prevents user edits, not calculation assignments.
- `calc.copyto.app-owns-defaults` (judgment): Applications own meaningful default values and whether an editable destination is appropriate for an automatically assigned result.

### Changes, resets, and deferred calculation

Owner: [protocols.md#changes-resets-and-deferred-calculation](../reference/protocols.md#changes-resets-and-deferred-calculation)

- `calc.schedule.triggers` (judgment): Editor initialization alone does not trigger calculations. EditRequest values are supplied by the service. Changes to participating inputs, including results assigned by another calculation or changed reference context inputs, trigger their calculations.
- `calc.submit.app-error` (judgment): An App Error response does not block Submit: the save path then rejects the request and marks the field through ErrorFieldPath.
- `calc.app-owns-business-rules` (judgment): Applications remain responsible for backend business rules and meaningful defaults; Studio proves structural declarations, while clients validate actual runtime values.

### Calculated allowed sets

Owner: [protocols.md#calculated-allowed-sets](../reference/protocols.md#calculated-allowed-sets)

- `calc.allowed-set.copyto-source`: A successful [field calculation](../reference/protocols.md#field-calculation-requests) Response field may `CopyTo` a ConstrainedBy source, or data containing it, under the ordinary CopyTo rules. A row's own source (a parent-relative ConstrainedBy path) is written by a calculation rooted at that row, through its `@`-relative CopyTo, so each row's allowed set follows that row's inputs. This lets the service change which choices a page offers while the page stays open, from what has been entered so far.
- `calc.allowed-set.refilter`: When a client applies a result that changes a source, it re-filters every chooser reading that source. A current selection inside the new set, and its payload, are untouched. A current selection outside the new set is reset to unanswered, discarding a removed Union branch's payload with it. The field's `Default` applies only when the new set contains it; a Default outside the set is not applied, and a Nullable field is reset to unanswered, never to null, because null would turn a removed answer into a complete one. The [singleton rule](../reference/protocols.md#backend-constrained-choices) then applies to the unanswered field. That reset is an input change for calculations reading the field and propagates through the dependency graph like any other result. The reset, any singleton selection, and the assignment are one atomic editor change.

### Single-action confirmation and input presentation

Owner: [protocols.md#single-action-confirmation-and-input-presentation](../reference/protocols.md#single-action-confirmation-and-input-presentation)

- `calc.request-confirmation.ignored`: A [calculation request](../reference/protocols.md#field-calculation-requests) ignores the marker: when the client sends a marked variant as a calculation, it shows no dialog and asks nobody. The same variant chosen by a person, forwarded at landing, or submitted still requires confirmation.

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `calc.inputs.alternatives-order`: Calculation CopyFrom instead names each alternative's source and tries its alternatives in declaration order under the [context table](../reference/schema.md#context-and-syntax).
- `copy.selection.single-source`: Selection is independent for each annotated receiving node. A selected composite value is projected from that one source; do not merge reference objects or fill its absent members by searching another object.
- `calc.inputs.unavailable-blocks`: A required calculation mapping with no available input instead blocks that calculation. The operation supplies the context; it does not redefine path matching or field projection.
- `copy.paths.explicit-qualifiers`: An explicit path may address a node at a different nesting depth or with a different Native Name. Type or Protocol Variant qualifiers identify the permitted schema context; a compatible type alone does not identify the intended value. Within a copied Table, corresponding members retain exact, case-sensitive Native Name matching. Selecting a differently located Table does not implicitly rename its members; such a difference requires an explicit mapping of the relevant node where permitted, or a payload with corresponding member names.
- `copy.projection.native-names`: Both CopyFrom and CopyTo use the same reduced-field projection rules. Match record fields by exact, case-sensitive Native Name and recurse through records and array elements. Source-only fields absent from the destination type MUST be ignored without a copy error; they are not assignments and do not participate in destination compatibility, overlap, or protection checks. Matching fields retain the applicable type, nullability, semantic scalar, and Union identity requirements. This does not make a required destination field optional or permit an incompatible matching field.
- `copy.projection.null-and-validity`: A valid null assigns null to a nullable destination; it does not mean unchanged. A received CopyTo payload must remain valid under its own schema. CopyFrom may project an unfinished editor under the destination's completeness rules.
- `copy.naming.shared-roles` (judgment): **Design rationale: structural matching by field name.** Because copied records match members by Native Name and ignore source-only fields, a Rust or TypeScript App composes values the way a scripting language does: any record with a member of the expected name and a compatible type can supply or receive it, whatever its declaring type. A calculation input can list `@.Business.assets,@.Farm.assets,@.Rental.assets`, and each branch's rows project into the input's element type because every branch names that member `assets`. A field name is therefore part of an App's interface, not only a label. Name fields that play the same role across types deliberately and identically, and fields with different meanings differently; one calculation, projection or alternative list then serves every type that shares the role, with no adapter types.
- `copy.initialization.unaffected-nodes`: CopyFrom and CopyTo override initialization only at the nodes they actually assign. On a newly constructed destination, every unaffected node retains its ordinary Fixed, Default, localized default, nullable, and type initialization. This includes destination-only fields within a copied Table or array element.

### Ordinary initialization

Owner: [schema.md#ordinary-initialization](../reference/schema.md#ordinary-initialization)

- `copy.initialization.existing-destination`: On an existing destination, unaffected nodes keep their current values; copying another field does not reset them.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.
- `init.undefined.copy-source` (judgment): **Copy source:** `CopyFrom` skips an undefined or absent value for the next alternative and stops at a valid null; with nothing available it copies nothing ([copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)). An undefined required calculation input blocks its calculation; a validation rule that reads undefined is aborted and reports a missing answer.
- `init.undefined.copy-destination` (judgment): **Copy destination:** a successful `CopyTo` answers the destination or replaces its answer, the later assignment winning; it never makes a node undefined, assigns null only to a Nullable, and never overwrites `Fixed` or `CacheIdentity` data ([applying results](../reference/protocols.md#applying-calculation-results)).
- `init.undefined.constrained-by` (judgment): **[`ConstrainedBy`](../reference/protocols.md#backend-constrained-choices):** an undefined non-nullable field with a one-entry allowed set takes that entry, as a Union `Default` selects its variant; an empty set leaves a Nullable field only null; a constructed value outside the set stays a field error. When a result changes a [calculated set](../reference/protocols.md#calculated-allowed-sets), a selection outside it resets to undefined, never null; its `Default` reapplies only if the new set contains it, and the singleton rule follows.

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `copy.fixed.member-precedence`: Within a copied Table or array, a copied member takes precedence over a Fixed initializer, but still has to satisfy the Fixed constraint and every other applicable constraint. Copying does not waive destination protections such as the calculation CopyTo prohibition on overwriting Fixed fields.
- `copy.apply.construct-or-merge`: The operation defines how the projected values are applied. CopyFrom constructs its destination; CopyTo merges into an existing destination. Source selection, destination uniqueness, array correspondence, and operation ownership retain their respective contracts.

### Typed ancestors and bound occurrences

Owner: [schema.md#typed-ancestors-and-bound-occurrences](../reference/schema.md#typed-ancestors-and-bound-occurrences)

- `calc.paths.parent-steps`: Calculation paths are relative to the [calculation root](../reference/protocols.md#calculation-root) occurrence: `@` selects the root occurrence, `^` its parent, `^^` its grandparent, and further parent steps, each followed by ordinary field/index segments. Calculation CopyFrom also accepts two selector forms:
- `calc.paths.binds-from-root`: A calculation binds self-or-ancestor from its root occurrence, so a typed selector naming the root's own type selects the root itself.
- `calc.paths.current-request-only`: Calculation `@`, `^` and typed selectors read only the current request, never a reference object; a calculation reads the interaction reference context only through `&` paths, under [calculation requests](../reference/protocols.md#field-calculation-requests). Calculation bindings do not permit CalculationRequest on protocol variants. The two logical contexts do not require separate stored copies.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Field calculation requests

Owner: [protocols.md#field-calculation-requests](../reference/protocols.md#field-calculation-requests)

- `calc.request.declaration`: CalculationRequest is field-only and lists distinct, case-sensitive calculation Request variant names

### Calculation root

Owner: [protocols.md#calculation-root](../reference/protocols.md#calculation-root)

- `calc.root.declaration`: CalculationRoot goes on the root node and lists distinct, case-sensitive calculation Request names of each containing request's protocol
- `calc.root.mandatory`: Every calculation Request has a CalculationRoot, and a CalculationRoot names only calculation Requests
- `calc.root.active-requires-root`: Each current request with a field participating in a calculation contains at least one root occurrence of it
- `calc.root.union-one-occurrence`: A calculation's root goes on a Union field or on its branch payload, never on both

### Field calculation requests

Owner: [protocols.md#field-calculation-requests](../reference/protocols.md#field-calculation-requests)

- `calc.target.ordinary-request`: A calculation targets another ordinary Request with a Table payload in the same protocol and NextResponse naming correlated responses
- `calc.target.reply-plain-or-error`: A calculation NextResponse names no variant with NextRequest or WorkflowRequest and no Request-only variant; Error is admitted
- `calc.inputs.copyfrom`: Every direct calculation input has CopyFrom; root-relative paths read the request, & paths read reference context, first available alternative wins
- `calc.participation.both-directions`: Each trigger is read by its nearest root's instance, or by some instance when no root is at or above it

### Invariant calculation inputs

Owner: [protocols.md#invariant-calculation-inputs](../reference/protocols.md#invariant-calculation-inputs)

- `calc.invariant.protected-value`: An invariant value is Hidden, Fixed or ReadOnly (itself or an enclosing Table or Union) and written by no CopyTo
- `calc.invariant.copyto-destinations`: A CopyTo destination, including a calculated allowed set and its constrained fields, is never invariant

### Field calculation requests

Owner: [protocols.md#field-calculation-requests](../reference/protocols.md#field-calculation-requests)

- `calc.inputs.alternatives-resolve`: Every required input resolves in each instance, and every declared alternative resolves somewhere

### Union calculation roots

Owner: [protocols.md#union-calculation-roots](../reference/protocols.md#union-calculation-roots)

- `calc.union.one-destination-per-output`: Wherever a calculation instance is applicable, every reply output has exactly one destination

### Applying calculation results

Owner: [protocols.md#applying-calculation-results](../reference/protocols.md#applying-calculation-results)

- `calc.copyto.every-field`: Every direct successful Response payload field has CopyTo; discriminators and Error responses are exempt
- `calc.copyto.type-checks`: Studio checks mapping types, nullability, numeric semantics and money currency
- `calc.cycle.no-self-trigger`: Calculations cannot read fields they write, directly or through other calculations; composite reads include membership and projected descendants
- `calc.copyto.protected-destinations`: Fixed and CacheIdentity destinations, and data containing them, cannot be overwritten

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `copy.fixed.no-copyfrom`: A field cannot combine Fixed and CopyFrom
- `copy.copyto.exactly-one-match`: Exactly one CopyTo destination matches; several matches are rejected before any assignment
- `copy.types.compatible`: CopyFrom and CopyTo require compatible receiving types and preserved operation ownership; runtime checks actual values, bindings and unresolved ambiguity

### Context and syntax

Owner: [schema.md#context-and-syntax](../reference/schema.md#context-and-syntax)

- `calc.paths.syntax`: Calculation CopyFrom takes `@`, `^`, `^<Type>` and `&` alternatives; CopyTo takes only `@`; `$` is rejected

### Typed ancestors and bound occurrences

Owner: [schema.md#typed-ancestors-and-bound-occurrences](../reference/schema.md#typed-ancestors-and-bound-occurrences)

- `paths.typed.strict`: Strict `^<Variant.field.item>` names every schema level from the active variant
- `paths.typed.relaxed`: Relaxed `^<Type>` selects the nearest enclosing occurrence of the resolved named type

<!-- /generated -->
