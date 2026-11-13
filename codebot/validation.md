# Writing validation rules

Load this card when a task adds or changes a `ValidationRules` rule set, a
`RangeMin` or `RangeMax` bound (including a quoted `min` or `max` on the
numeric annotation) or a String `Pattern`. Its rules are generated from the
[Data Validation Rules specification](../reference/validation-rules.md), which
owns them. Paths follow the
[Libertas Schema Path](../reference/schema.md#current-data-references-in-validation-expressions)
and its [reference availability rules](../reference/schema.md#availability-null-and-interaction-ownership).
The [ValidationRules quick reference](validation_rules.md) lists the syntax and
the [annotation inventory](schema_annotations.md) gives each annotation's Rust
and TypeScript spelling.

Some owners continue a quoted rule ending in a colon with a code block, such as
the [precedence table](../reference/validation-rules.md#15-operator-precedence)
and the [grammar](../reference/validation-rules.md#14-expression-grammar);
follow the owner link for it.

## Order of work

1. Decide where each check belongs (`validation.placement.*`,
   `validation.authoring.*`) before writing any rule.
2. Choose the form: a rule set (`validation.declare.*`, `validation.syntax.*`,
   `validation.finalonly.*`), a dynamic bound (`validation.range.*`) or a
   String format (`validation.pattern.*`, `validation.regex.*`,
   `validation.escape.*`, `validation.literal.*`).
3. Write each expression with typed values and operators (`validation.types.*`,
   `validation.null.*`, `validation.unanswered.*`, `validation.fallback.*`,
   `validation.boolean.*`, `validation.equality.*`, `validation.relational.*`,
   `validation.arithmetic.*`, `validation.precedence.*`) and the functions it
   needs (`validation.is.*`, `validation.calendar.*`, `validation.collection.*`,
   `validation.cursor.*`, `validation.collect.*`, `validation.unique.*`,
   `validation.count.*`, `validation.sum.*`, `validation.string.*`,
   `validation.resource.*`, `validation.matches.*`, `validation.bitflags.*`).
4. Handle first-time composition without a reference context and every
   nullable read (`validation.reference.*`, `validation.static.*`).
5. Declare each message key in `StringResources` under the
   [source documentation card](source_documentation.md), compile with Studio
   and fix every reported rule, then test the rules with
   `evaluate_validation_rules` or `test-rules` fixtures (`validation.debug.*`).

<!-- generated:codebot-rules task=validation-rules -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined.copy-source` (judgment): **Copy source:** `CopyFrom` skips an undefined or absent value for the next alternative and stops at a valid null; with nothing available it copies nothing ([copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)). An undefined required calculation input blocks its calculation; a validation rule that reads undefined is aborted and reports a missing answer.

### 1. Purpose

Owner: [validation-rules.md#1-purpose](../reference/validation-rules.md#1-purpose)

- `validation.purity.expression`: A rule is a pure expression of `data` and `referencedData`. During editing, any field MAY temporarily be undefined (unanswered), including a `Nullable` field whose [Nullable choice](../reference/presentation.md#nullable-choices) has no answer yet. Undefined is local editor state, never encodable and never an answer; `null` in a rule always means a present null, such as a Nullable's chosen null answer.
- `validation.unanswered.aborts`: **A rule that reads an unanswered value is aborted:** the client does not report its result and reports that value as a missing answer instead, in incremental and final validation alike. During editing the missing answer is the field's unanswered indicator, never a failure that rejects an incremental change; at final validation it blocks submission like any unanswered field. A rule's result therefore never depends on an unanswered value, and no rule distinguishes null from undefined.
- `validation.null.fallback-not-answer-test` (judgment): Within a rule, `null` serves as a fallback value, chiefly through [`|`](../reference/validation-rules.md#73-first-non-null-value-fallback), not as a test of whether the person answered: whether a field must be answered is structural, declared by a non-`Nullable` field or a `Nullable` field without a Default.
- `validation.null.accept-not-optional`: Mandatory-field validation and `ValidationRules` are complementary, independent requirements. A rule MAY accept a `null`, but that success does not make the field optional or suppress the final mandatory-field check.
- `validation.final.requirements`: Final submission separately requires structural schema validity, every field answered (a non-`Nullable` field present, a `Nullable` field null or present), and all validation rules to pass.
- `validation.bytes.mime-empty-null`: A zero-length [MIME-aware Bytes](../reference/schema.md#mime-aware-bytes) value is a present, encodable Bytes field but has undefined media meaning and evaluates as null. Its non-`Nullable` declaration and size bounds do not require media selection; an App requiring a file declares a rule such as `[] != null : "IMAGE_REQUIRED";`.
- `validation.bytes.lmb1-non-null`: A defined LMB1 value evaluates as non-null, even when its content is empty. Ordinary Bytes keep their existing valuation.
- `validation.placement.contexts`: ValidationRules uses the [shared schema reference contract](../reference/schema.md#availability-null-and-interaction-ownership). Rules may occur in editable Request/SubscriptionRequest data, inherited types, list items, and hidden/read-only fields. External reference reads require neither CopyFrom nor request-side copies of server metadata.
- `validation.reference.first-composition`: Authors MUST account for first-time request composition with no reference context: every curly reference evaluates to `null` in that state. ValidationRules must handle that absence explicitly, for example with a null guard or a typed `|` fallback. The client still evaluates the rule normally; absence does not automatically make the rule pass.
- `validation.reference.no-fetch`: Neither validation nor request composition fetches server data to resolve these references.
- `validation.valid.all-true`: For the supplied `referencedData`, the data instance is valid iff every validation rule evaluates to `true`.

### 2. Data Model

Owner: [validation-rules.md#2-data-model](../reference/validation-rules.md#2-data-model)

- `validation.types.semantic`: Every successful intermediate evaluation carries an internal semantic type; operators MUST NOT infer a new type merely from the scalar's storage encoding. In particular, DateOnly is a calendar type even though its schema and wire encoding is the integer `yyyyMMdd`, and DateTime is an instant type even though its schema and wire encoding is integral Unix epoch seconds. Literals, paths, fallback results, operator results, and function results all retain or produce an explicit internal type. `null` has its own type.
- `validation.types.date-literal-only`: A [DateOnly literal](../reference/validation-rules.md#141-dateonly-literals) has type DateOnly; the language has no DateTime literal.
- `validation.types.bitflags-host`: A [Bitflags](../reference/schema.md#43-value-constraint-and-presentation-attributes) host, an integer Number whose `Bitflags` attribute names a flag catalog, is an ordinary integer in rules. Its catalog's members are schema constants that appear only as the right operand of the [membership test `has`](../reference/validation-rules.md#142-bitflags-membership).

### 3. Rule Set

Owner: [validation-rules.md#3-rule-set](../reference/validation-rules.md#3-rule-set)

- `validation.declare.owners`: `ValidationRules` MAY be declared on:
- `validation.declare.cumulative`: A field/type rule is rooted at each concrete value of the annotated node. A named-type rule and a field-local rule are cumulative when a field references that named type. Inherited field rules are cumulative as well.
- `validation.paths.root`: The empty data path `[]` denotes the annotated value itself. Function rules address arguments by name, such as `[minimum]` and `[options/timeout]`.
- `validation.declare.spelling`: The TypeScript source spelling is `@LibertasValidationRules("...")`. The Rust source spelling is `#[libertas_validation_rules("...")]`.
- `validation.declare.rust-helper`: A Rust named type may use the standalone attribute; fields and variants use the `LibertasExport` helper attribute. Parsers emit the raw string and source location. Libertas Studio's shared schema validator owns grammar and schema-decidable semantic validation.
- `validation.finalonly.default`: The default is `false`.
- `validation.finalonly.incremental-tolerance` (judgment): An omitted directive or `FinalOnly=false` places the rule set in both incremental editor validation and final validation. Because an incremental failure restores the previous value, such a rule set MUST tolerate incomplete editing; an unanswered value never reaches a rule.
- `validation.finalonly.true`: `FinalOnly=true` explicitly omits the entire rule set from incremental validation and evaluates it only during final validation. This removes the need for that rule set to tolerate an initial incomplete editor state. It does not alter `null` expression semantics, change `Nullable`, or replace final static schema checks.
- `validation.finalonly.per-declaration`: When field, inherited, and named-type rule sets accumulate, each declaration retains its own `FinalOnly` value.

### Authoring guidance

Owner: [validation-rules.md#authoring-guidance](../reference/validation-rules.md#authoring-guidance)

- `validation.authoring.structural-first` (judgment): Use the schema's precise type and ordinary structural attributes for nullability, numeric ranges, sizes, and uniqueness. `ValidationRules` express semantic relationships that those declarations cannot express; they do not restate or replace structural constraints.
- `validation.authoring.required-is-structural` (judgment): When the person must answer a field, declare it structurally: a non-`Nullable` field, or a `Nullable` field without a Default, whose [Nullable choice](../reference/presentation.md#nullable-choices) starts unanswered. Do not restate that requirement as a `!= null` rule; an unanswered value never reaches a rule.

### Where a check belongs

Owner: [validation-rules.md#where-a-check-belongs](../reference/validation-rules.md#where-a-check-belongs)

- `validation.placement.save-path-authoritative` (judgment): The App's save path is the authoritative validator. It checks every Request it receives in full, whether or not a client evaluated any rule. Client rules are a convenience for the person editing; they are never a security or correctness boundary, and never the only copy of a check.
- `validation.placement.structural-checks` (judgment):

  | Check | Where it belongs |
  | --- | --- |
  | Type, nullability, numeric range, size, uniqueness, String format | Schema declarations and `Pattern`. The client enforces them while editing, and the SDK's Avro decoder enforces each static bound (a number's `min`, `max` and `step` included), and that every DateOnly value is a real Gregorian date, when the App decodes a Request, so App code and tests never re-check them. Dynamic `RangeMin`/`RangeMax` limits stay save-path checks. |

- `validation.placement.local-feedback` (judgment):

  | Check | Where it belongs |
  | --- | --- |
  | Simple, local relationship where immediate feedback helps the person (one field, or fields on the same page) | Optionally a rule set with the directive omitted (incremental and final), mirroring a save-path check. |

- `validation.placement.business-logic` (judgment):

  | Check | Where it belongs |
  | --- | --- |
  | Business logic: cross-page or cross-record consistency, legal or eligibility rules, anything needing App state | The save path only. Reject with an [Error response](../reference/protocols.md#protocol-roles) and name the field with [`ErrorFieldPath`](../reference/protocols.md#error-field-path). |

- `validation.placement.final-only-discouraged` (judgment): Final-only rule sets (an explicit `FinalOnly=true;`) are discouraged. They run at submission, the same moment as the save-path check, so they add no earlier feedback; they only add a second copy of the logic that can drift from the save path or fail silently. Put submission-time checks in the save path instead. Final-only rule sets remain valid and are still evaluated. Omitting the directive means the rule set is also evaluated incrementally; final-only must be written explicitly.
- `validation.messages.split-for-presentation` (judgment): The two forms above are validity-equivalent but intentionally not presentation-equivalent: the first form can display the key belonging to the selected failed rule, while the combined expression owns one message key.

### Debugging rules

Owner: [validation-rules.md#debugging-rules](../reference/validation-rules.md#debugging-rules)

- `validation.debug.trace`: Studio offers an opt-in developer trace for each applicable rule. It records the expression text, message key, and source file and one-based line when known (otherwise null), and a `passed`, `failed`, or `undecided` outcome. `undecided` means an evaluated read encountered an unanswered editor value; it is not a rule failure. Null or missing hosts and inactive Union payloads remain ordinary absence under the path rules, not unanswered values.
- `validation.debug.fixtures`: Each JSON fixture supplies `project`, optional `entry` and `phase`, `type`, `value` in schema JSON value form, and `expected`, an object mapping message keys to `passed`, `failed`, or `undecided`. Relative project paths resolve from the fixture's directory. The runner reports every fixture and exits zero only when compilation and all expectations pass; invalid fixtures and mismatches exit nonzero.
- `validation.debug.expectations`: Every evaluated rule key requires an expectation, and unknown expected keys fail. For a key evaluated at several value occurrences, its outcome is failed if any occurrence fails, otherwise undecided if any is undecided, otherwise passed. A rule expected to fail is a successful test when it fails as expected. Standard output is one JSON report containing per-fixture results, diagnostics, traces and a summary.

### 3.1 Dynamic numeric ranges

Owner: [validation-rules.md#31-dynamic-numeric-ranges](../reference/validation-rules.md#31-dynamic-numeric-ranges)

- `validation.range.purpose`: `RangeMin` and `RangeMax` reuse the Data Validation expression language to derive live inclusive constraints for a Number or DateTime value.
- `validation.range.spelling`: Their source spellings are `@LibertasRangeMin("expression")` / `@LibertasRangeMax("expression")` in TypeScript and `#[libertas_range_min("expression")]` / `#[libertas_range_max("expression")]` in Rust. The language-neutral schema stores the decoded expression string under `RangeMin` or `RangeMax`.
- `validation.range.number-annotation`: The numeric annotation also accepts a quoted expression in `min` or `max`: `@LibertasNumber(type=i64, min=0, max="[^/maximum]")` or `#[libertas_number(min=0, max="[^/maximum]")]`. Unquoted numeric literals and Rust `$CONST` values continue to emit `NumberMin`/`NumberMax`; quoted expressions emit `RangeMin`/`RangeMax`, exactly like the standalone range annotations. Rust source constants inside expression text use the existing constant substitution. `step` remains a fixed numeric value (or Rust source constant). No additional schema attributes are introduced.
- `validation.range.context`: Evaluation uses the same field-local context as `ValidationRules`: `[]` is the annotated Number or DateTime and square paths traverse from it. The shared [current-data path contract](../reference/schema.md#current-data-references-in-validation-expressions) permits a leading parent prefix: `[^/minimum]` reads the current row's `minimum`, and `[^^/limits/maximum]` reads a bound two levels above the field.
- `validation.range.reference-context`: Curly paths inspect the [current interaction reference context](../reference/protocols.md#interaction-reference-context), including in editable requests. The result type must match the control's semantic type:

  | Annotated control | Required non-null result |
  | --- | --- |
  | Ordinary Number, including TimeInterval | Finite Number; integer and fractional bounds are permitted. |
  | DateOnly | DateOnly calendar date. |
  | DateTime | DateTime instant. |

- `validation.range.null-no-bound`: `null` means that no dynamic bound is currently available, including when the first request is composed before any reference context exists. It adds no constraint.
- `validation.range.runtime-type-error`: A non-null Boolean, String, array, collection, object, mismatched semantic type, or non-finite numeric result is an error. When an expression's result cannot be proved compatible statically, clients MUST check it at runtime and reject validation on an incompatible result. They MUST NOT silently remove the bound, erase its semantic type, or coerce the result to a Number.
- `validation.range.expressions`: Paths and arithmetic expressions that preserve the required type are supported; bounds are not restricted to literal constants. For example, a DateTime path plus an integer number of seconds remains a DateTime bound; subtracting two DateTime paths yields a Number and cannot bound DateTime.
- `validation.range.inclusive`: An active `RangeMin` is an inclusive lower bound and an active `RangeMax` is an inclusive upper bound.
- `validation.range.intersection`: They intersect the native numeric type limits and any static `NumberMin` / `NumberMax` constraints. Clients MUST apply that same intersection to final validation and to numeric, DateOnly, DateTime, and TimeInterval GUI controls. A changing dependency value MUST update the live GUI constraint. DateOnly bounds use the `yyyyMMdd` wire representation at the schema boundary, but Data Validation evaluates the value as a real Gregorian calendar date.
- `validation.range.datetime-backing`: DateTime remains a distinct UI/wire and expression type but is constraint-backed as an unsigned integral Unix timestamp. Static Number constraints use that wire representation; dynamic Range expressions require a DateTime result. Their intersection applies to both its local date/time chooser and final UTC epoch-seconds validation.
- `validation.range.step-unchanged`: `RangeMin` and `RangeMax` do not change the `NumberStep` origin or lattice.
- `validation.range.unsatisfiable`: If the active effective minimum exceeds the active effective maximum, the range is unsatisfiable; a client MUST NOT swap the two bounds or otherwise invent a valid interval.
- `validation.range.absent-reference-null`: A numeric field, including an editable request field, can use limits from the current reference context. When its reference objects do not provide them, these expressions return null and add no dynamic bound; ordinary numeric constraints still apply:

  ```typescript
  /** Measured value
   * A numeric result constrained by the available server-provided limits.
   * @LibertasNumber(type=f64)
   * @LibertasRangeMin('{$.limits.minimum}')
   * @LibertasRangeMax('{$.limits.maximum}')
   */
  value: number;
  ```

- `validation.range.no-fetch`: These curly references can directly constrain editable request fields. The client reads the already received response; it does not fetch server state. The server remains responsible for enforcing its current authoritative limits. Authors bound a field only by values the request or its received response already carries; the service still enforces its current limit on save.

### 4.4 Null Values

Owner: [validation-rules.md#44-null-values](../reference/validation-rules.md#44-null-values)

- `validation.null.equality`: Every field uses ordinary equality operators for its current editor value, including a required field that has not received input yet.
- `validation.null.no-special-syntax`: No special nullable-access syntax exists.
- `validation.null.non-nullable-comparison`: The comparison is valid even when the schema field is not `Nullable`. Client editors use `null` as the initial state of many required fields, and rules are allowed to describe that state before final structural validation.

### 5.1 `is` Operator

Owner: [validation-rules.md#51-is-operator](../reference/validation-rules.md#51-is-operator)

- `validation.is.operator`: The `is` operator tests the selected Union variant or the selected Enumeration member of a value.
- `validation.is.union-runtime`: Its result is boolean. On a Union it tests the actual runtime variant.
- `validation.is.enumeration`: When the operand is an Enumeration, `Name` is a member name and the test is true exactly when the stored value selects that member, resolved as the Enumeration itself resolves a stored value (a member's `Fixed` value first, otherwise its index). A null, unset, or unresolvable Enumeration value gives false. A numeric comparison such as `[class] == 0` is unaffected.
- `validation.is.null-false`: If the left operand is `null`:

  ```c
  null is T
  ```

  evaluates to:

  ```text
  false
  ```

### 6.2 Cursor traversal

Owner: [validation-rules.md#62-cursor-traversal](../reference/validation-rules.md#62-cursor-traversal)

- `validation.cursor.element-node`: `all`, `any`, `none`, and `collect` bind a cursor to each concrete element of the arrays selected by their source path. A cursor is the element node itself, not its array index:

  ```c
  all(item in [items],
      [item/value] >= 0
  )
  ```

### 7.1 Boolean Context

Owner: [validation-rules.md#71-boolean-context](../reference/validation-rules.md#71-boolean-context)

- `validation.boolean.null-false`: `null` is implicitly converted to boolean `false`:

### 7.2 No General Truthiness

Owner: [validation-rules.md#72-no-general-truthiness](../reference/validation-rules.md#72-no-general-truthiness)

- `validation.truthiness.explicit-comparison`: Explicit comparison MUST be used:

  ```c
  [count] != null
  [name] != null
  [object] != null
  [array] != null
  ```

### 7.3 First-non-null value fallback

Owner: [validation-rules.md#73-first-non-null-value-fallback](../reference/validation-rules.md#73-first-non-null-value-fallback)

- `validation.fallback.first-non-null`: The evaluator returns the first operand whose value is not `null`, without changing that value's type. It evaluates no later operand after finding a winner. `false`, numeric zero, an empty string, an empty array or collection, and an object are all non-null and therefore win. The result is `null` only when every operand evaluates to `null`.
- `validation.fallback.static-types`: The result's possible static types are the left operand's non-null types plus the right operand's possible types only when the left operand may be null. A chain is left-associative and applies that rule repeatedly.

### 8. Logical Operators

Owner: [validation-rules.md#8-logical-operators](../reference/validation-rules.md#8-logical-operators)

- `validation.logical.short-circuit`: `&&` and `||` MUST use short-circuit evaluation.

### 9. Equality Operators

Owner: [validation-rules.md#9-equality-operators](../reference/validation-rules.md#9-equality-operators)

- `validation.equality.null`: `null` is an ordinary value for equality comparison.

### 10. Relational Operators

Owner: [validation-rules.md#10-relational-operators](../reference/validation-rules.md#10-relational-operators)

- `validation.dates.same-semantic-type`: Ordinary integers and floating-point Numbers are mutually comparable. DateOnly is comparable only with DateOnly, and DateTime only with DateTime; their storage encodings do not make them comparable with a Number. Strings are comparable only with strings.
- `validation.relational.null-false`: A relational comparison involving `null` evaluates to `false`.

### 11. Arithmetic Operators

Owner: [validation-rules.md#11-arithmetic-operators](../reference/validation-rules.md#11-arithmetic-operators)

- `validation.arithmetic.exact`: Numeric literals and decoded scalar decimal text are evaluated as exact rational values. Addition, subtraction, multiplication, division, remainder, equality, and ordering preserve the low digits of i64 Money and do not use binary64 intermediates. Division does not round a repeating fraction; remainder uses a quotient truncated toward zero. Each reduced numerator and denominator is limited to 256 decimal digits. Arithmetic exceeding that limit produces null, as does division by zero. This evaluation representation does not change the schema's integer/floating semantic types or any wire encoding. Ordinary numeric UI controls may still use floating-point coordinates; Money bounds and final checks retain exact values throughout.
- `validation.arithmetic.result-types`: Ordinary integer and floating-point operands use numeric arithmetic. Integer results remain internally typed as integers except that `/` produces a Number; an operation involving a floating-point Number produces a Number.
- `validation.arithmetic.date-overloads`: DateOnly and DateTime have these additional, exhaustive arithmetic overloads:

  ```text
  DateOnly + integer   -> DateOnly, shifted by Gregorian calendar days
  integer + DateOnly   -> DateOnly, shifted by Gregorian calendar days
  DateOnly - integer   -> DateOnly, shifted by Gregorian calendar days
  DateOnly - DateOnly  -> Number, elapsed calendar days

  DateTime + integer   -> DateTime, shifted by Unix seconds
  integer + DateTime   -> DateTime, shifted by Unix seconds
  DateTime - integer   -> DateTime, shifted by Unix seconds
  DateTime - DateTime  -> Number, elapsed seconds
  ```

- `validation.arithmetic.gregorian`: DateOnly arithmetic MUST use Gregorian calendar rules: a DateOnly path carrying the wire value `20240228` plus `1` evaluates to DateOnly `20240229`, and plus `2` evaluates to DateOnly `20240301`. It MUST NOT add to the numeric `yyyyMMdd` encoding. [DateOnly/DateTime mixed](../reference/validation-rules.md#16-static-rule-validation) arithmetic, integer-minus-date, unary `+`/`-` on a date, and `*`, `/`, or `%` on a date are invalid. Calendar-month and year shifts and the extraction of a date's year, month, and day are the functions of section 11.1, not operators.
- `validation.arithmetic.null`: If an arithmetic operand evaluates to `null`, the arithmetic result is `null`.

### 11.1 Calendar-date functions

Owner: [validation-rules.md#111-calendar-date-functions](../reference/validation-rules.md#111-calendar-date-functions)

- `validation.calendar.fields`: `year(d)` returns the calendar year (1000 through 9999), `month(d)` the month (1 through 12), and `day(d)` the day of the month (1 through 31), each as an integer Number. For example, `year(d"2024-02-29") == 2024`, `month(d"2024-02-29") == 2`, and `day(d"2024-02-29") == 29`.
- `validation.calendar.add-months`: `add_months(d, n)` returns the DateOnly `n` calendar months after `d`, or before it when `n` is negative. The year and month advance by `n` months. The day of the month is kept when the target month has that day; otherwise it becomes the target month's last day:

  ```c
  add_months(d"2024-01-31", 1)  == d"2024-02-29"
  add_months(d"2023-01-31", 1)  == d"2023-02-28"
  add_months(d"2024-03-31", -1) == d"2024-02-29"
  add_months(d"2024-05-31", 1)  == d"2024-06-30"
  add_months(d"2024-01-15", 24) == d"2026-01-15"
  ```

- `validation.calendar.no-compose` (judgment): Because of the day clamp, shifts do not compose: `add_months(add_months(d"2024-01-31", 1), 1)` is `d"2024-03-29"`, while `add_months(d"2024-01-31", 2)` is `d"2024-03-31"`. A shift by days remains the `+` and `-` arithmetic of section 11.
- `validation.calendar.null-range`: If any argument is `null`, the result is `null`. A shift whose result falls outside the DateOnly range of years 1000 through 9999 also evaluates to `null`, as does day arithmetic beyond that range.
- `validation.calendar.pure`: The functions are pure (section 17): they read only their arguments, and the language has no current-date value.
- `validation.calendar.month-difference` (judgment): The language has no month-difference function. A whole-month difference has competing conventions at month ends; compare a shifted date instead. For example, "at least `n` months from `a` to `b`" is `add_months([a], n) <= [b]`, and the last day of a period of `n` months beginning on `[start]` is `add_months([start], n) - 1`.

### 12. Collection Operations

Owner: [validation-rules.md#12-collection-operations](../reference/validation-rules.md#12-collection-operations)

- `validation.collection.concatenation`: Arrays selected through multiple parent-array elements are concatenated in deterministic path and element order. A selected `null` or missing array contributes no elements.

### 12.1 `all`

Owner: [validation-rules.md#121-all](../reference/validation-rules.md#121-all)

- `validation.collection.all`: Returns `true` iff the expression evaluates to boolean `true` for every array element.

### 12.2 `any`

Owner: [validation-rules.md#122-any](../reference/validation-rules.md#122-any)

- `validation.collection.any`: Returns `true` iff the expression evaluates to boolean `true` for at least one array element.

### 12.3 `none`

Owner: [validation-rules.md#123-none](../reference/validation-rules.md#123-none)

- `validation.collection.none`: Returns `true` iff the expression evaluates to `false` for every array element.

### 12.4 `collect`

Owner: [validation-rules.md#124-collect](../reference/validation-rules.md#124-collect)

- `validation.collect.predicate-first`: `collect` returns an ordered collection. Its source remains a current-data path selecting arrays, including arrays reached through parent-array fan-out. For each concrete cursor, it evaluates `predicate` first. Only Boolean `true` causes the projection to be evaluated, once, in that cursor's scope. False or null predicates contribute nothing and do not read the projection.
- `validation.collect.direct-path`: A direct square data path retains path-selection behavior: append every selected node in path order, flattening arrays crossed on the path. A missing field selects no node; a selected null field contributes one null. A path ending at a List contributes that List as one value, without flattening its elements.
- `validation.collect.computed-collection`: Another expression producing a collection, including nested `collect`, concatenates its items in order at this one boundary. An empty collection contributes no item.
- `validation.collect.computed-value`: Another expression producing a scalar, null, object, or List contributes exactly one value, retaining its semantic type and identity. A computed null is one item, even when it came from a missing path. Objects and Lists are not recursively flattened or coerced.
- `validation.collect.order-bag`: The result preserves source order, projection order, and duplicates; it is a collection (bag), not a mathematical set. Empty sources and predicates that never evaluate to true produce empty collections.
- `validation.collect.type-guards`: Studio narrows projection validation using positive type tests on the cursor or a nested cursor path. The tests may be conjuncts of `&&`; unrelated conjuncts do not remove their guarantees. For example, `[row/document] is Payment && [row/enabled]` permits numeric projection paths through the selected `Payment` payload even when another variant declares an object field with the same name.
- `validation.collect.guard-scope`: These facts apply only to that `collect` projection and its nested scopes; they do not change the schema or escape to later expressions. `||`, negation and fallback do not establish positive type facts. A guard on another cursor or path cannot narrow this one, and array fan-out remains invalid as an `is` operand. Null and absent payloads retain their existing evaluation behavior.
- `validation.collect.client-support` (judgment): Computed projections require clients implementing this contract. Older clients support only direct data-path projections and must be updated before an App using computed projections is distributed to them. Schema acceptance alone does not update an installed client.

### 12.5 `is_unique_set`

Owner: [validation-rules.md#125-isuniqueset](../reference/validation-rules.md#125-isuniqueset)

- `validation.unique.semantics`: Returns `true` exactly when no two collection elements compare equal under the ValidationRules `==` semantics. Empty and single-element collections are unique. Two `null` elements are duplicates.
- `validation.unique.fanout-collection`: A fan-out path argument such as `[rows/key]` is always the collection of the nodes it selects, whether that is zero, one or many, including zero when a List on the path is `null`; it is never reduced to `null` or a single value for this function.
- `validation.unique.not-all` (judgment): `all` is not overloaded for this purpose. Its predicate is evaluated independently for one bound cursor and returns only a Boolean; it has neither collection state nor another-element binding with which to detect duplicates.
- `validation.unique.key-tuple`: The cursor binds to each element of the source arrays exactly as for `all` (section 6.2), and the key expressions are evaluated for that element in order. The element's key is the tuple of their values. The result is `true` exactly when no two elements have keys whose corresponding components all compare equal under `==`. A `null` component therefore equals only `null`: two elements that agree on every other component and are both `null` in one component are duplicates, while a `null` and a non-null component differ.
- `validation.unique.empty`: An empty source, a single element, and a missing or `null` source array are unique (section 12.7).
- `validation.unique.bound-keys` (judgment): The keys are bound to their element rather than written as separate paths: independently resolved paths form separate collections, and a field absent from one element's active Union payload would shift the later values, pairing components that belong to different elements.

### 12.6 `count`

Owner: [validation-rules.md#126-count](../reference/validation-rules.md#126-count)

- `validation.count.semantics`: returns the number of elements in an array or a `collect` result.
- `validation.count.fanout`: A fan-out path argument such as `[rows/key]` is always the collection of the nodes it selects, so it counts 0, 1 or more (0 when a List on the path is `null`); otherwise, if its argument resolves to `null`, `count()` evaluates to `null`.

### 12.7 Missing or Null Traversal Arrays

Owner: [validation-rules.md#127-missing-or-null-traversal-arrays](../reference/validation-rules.md#127-missing-or-null-traversal-arrays)

- `validation.collection.missing-array`: A missing or `null` terminal array contributes zero elements to a traversal. Consequently, the usual empty-collection identities apply: `all` and `none` return `true`, `any` returns `false`, and `collect` returns an empty collection. Use an explicit path null test when array presence is itself required.

### 12.8 `sum`

Owner: [validation-rules.md#128-sum](../reference/validation-rules.md#128-sum)

- `validation.sum.null-leaves`: `null` leaves are skipped. When nothing is selected, including an empty or missing array or a source whose leaves are all `null`, the result is the integer `0`; a single selected value is its own sum. The sum is therefore a Number for every data instance, and a rule such as `[principal] <= sum([contributions/amount])` needs no null guard for the collection.
- `validation.sum.exact`: Addition is the exact arithmetic of section 11, so the result is independent of order; it is `null` only when it exceeds that section's exact numeric limit. The result is an integer when every summed leaf is integer-typed, and a Number otherwise.

### 13. String Operations

Owner: [validation-rules.md#13-string-operations](../reference/validation-rules.md#13-string-operations)

- `validation.string.length-bytes`: which returns the number of bytes in the string's UTF-8 encoding. Every platform must use this same byte count, matching [String `SizeMin`/`SizeMax`](../reference/schema.md#43-value-constraint-and-presentation-attributes). The text is measured as supplied, without Unicode normalization. For example, `length("A")` is 1, `length("é")` is 2, `length("中")` is 3, `length("😀")` is 4, and `length("e\u0301")` is 3. Native-character constraints (`MinLengthAbs`/`MaxLengthAbs`) remain a separate Unicode-scalar constraint.
- `validation.string.no-character-count`: `length` counts bytes, not characters, by design. A byte bound is the same on every platform and matches what storage, wire messages and native buffers actually hold, so a value accepted on one client can never exceed a limit on another. A character count differs by platform and Unicode version. The language has no character-counting function today. If a rule ever needs one, it will be a separate, explicitly named function; `length` and String Size will keep counting UTF-8 bytes.
- `validation.string.length-null`: If the input is `null`:

  ```text
  length(null) -> null
  ```

### 13.1 String resource references

Owner: [validation-rules.md#131-string-resource-references](../reference/validation-rules.md#131-string-resource-references)

- `validation.resource.key`: takes a FormattedText value and returns its string resource **key** as a String, or null when the value is null. The key identifies the resource template, not its localized text, so comparing it gives the same result in every locale. Arguments do not change the key.

### 13.2 Full-string patterns

Owner: [validation-rules.md#132-full-string-patterns](../reference/validation-rules.md#132-full-string-patterns)

- `validation.matches.full-string`: `matches(value, pattern)` tests the entire String against the portable regex language below. It returns Boolean for two Strings, without trimming, normalization, substring search, flags, captures, replacement or coercion. An empty pattern matches only empty text.
- `validation.matches.null`: With two Strings, return Boolean. Empty text matches only a pattern accepting empty text; an empty pattern accepts only empty text. A null argument returns null under existing missing-data/deferral rules. Optional fields can use `[] == null || matches([], "[0-9]+")`. Complete final rules still require true.
- `validation.matches.static-compile`: Studio compiles every statically known pattern, even in an unreachable branch. Malformed syntax, unsupported constructs and compilation limits produce a schema error at the narrowest source location.
- `validation.matches.evaluation-errors`: A dynamic pattern compiles when evaluated. Invalid patterns, wrong runtime types and compilation failures are **evaluation errors**, not false or null. An evaluated error makes the enclosing rule fail, including under `!`, `|` or `||`. Short circuiting still avoids evaluating an unused branch. Thus `!matches(value, badPattern)` cannot pass. This explicit error distinction must be preserved through the evaluator; the existing missing-value sentinel alone is insufficient. No expression-level exception-catching API is added.

### Pattern attribute and lifecycle

Owner: [validation-rules.md#pattern-attribute-and-lifecycle](../reference/validation-rules.md#pattern-attribute-and-lifecycle)

- `validation.pattern.effective.authoring`: Declare a field-local Pattern to replace its type Pattern; [ValidationRules remain cumulative](../reference/validation-rules.md#pattern-attribute-and-lifecycle).
- `validation.pattern.message`: A mismatch uses a standard localized message, such as “Enter a value in the required format.” The field description explains that format. There is no second message-key attribute; use matches in ValidationRules for a custom message. Invalid patterns are configuration/rule errors, not typing errors.
- `validation.pattern.server-validates` (judgment): Server Apps still validate untrusted requests independently.

### Portable regex language

Owner: [validation-rules.md#portable-regex-language](../reference/validation-rules.md#portable-regex-language)

- `validation.regex.scalars`: One regular-language grammar over Unicode scalars, case sensitive and locale independent. No normalization: composed and decomposed accents differ.

### Escape and quote contract

Owner: [validation-rules.md#escape-and-quote-contract](../reference/validation-rules.md#escape-and-quote-contract)

- `validation.escape.decode-once`: Decode each layer once. The compiler receives **regex text**, not source code or a JSON-encoded value. Transport JSON escaping is representation only.

### Source attribute layer

Owner: [validation-rules.md#source-attribute-layer](../reference/validation-rules.md#source-attribute-layer)

- `validation.escape.rust-pattern`: For shared semantic rejection, follow the [Pattern placement and grammar requirements](../reference/validation-rules.md#pattern-attribute-and-lifecycle). **Rust:** one positional native String literal, cooked or raw, or `$CONST` resolving to a native String through existing source-constant machinery. Cooked literals use native Rust escapes; raw `r"..."`, `r#"..."#` etc do not decode backslashes. Increase delimiter hashes when content includes the current closing quote/hash sequence.
- `validation.source.raw-delimiter-choice`: Native Rust source preprocessing and cooked-string line continuations retain their native meaning; they are not regex escapes. For exact LF/CR data, prefer explicit cooked escapes or regex `\n`/`\r` in a raw literal. If the content cannot fit a legal raw delimiter, use a cooked literal instead. Bytes and character literals are not accepted as String arguments.
- `validation.escape.typescript-pattern`: For shared semantic rejection, follow the [Pattern placement and grammar requirements](../reference/validation-rules.md#pattern-attribute-and-lifecycle). **TypeScript Pattern in JSDoc:** one positional double-quoted **JSON String literal**, or `$CONST` resolving a native String constant. Decode JSON once, without trim. Accept exactly JSON escapes `\" \\ \/ \b \f \n \r \t \uHHHH`; surrogate pairs form one scalar.
- `validation.escape.json-whitespace`: JSON whitespace here means only space, tab, LF and CR. Raw U+0000–U+001F inside the JSON literal is invalid; use an escape. Literal Unicode scalars above that range, including U+2028/U+2029, are JSON content, not delimiters. Native source/comment syntax must also permit the spelling. No implicit line continuation or JSDoc line-decoration removal occurs inside a literal; use escaped controls or a constant for multiline patterns.
- `validation.escape.const-argument`: For shared semantic rejection, follow the [Pattern placement and grammar requirements](../reference/validation-rules.md#pattern-attribute-and-lifecycle). `$CONST` occupies the whole argument; no interpolation occurs inside quoted patterns, where `$` remains a regex anchor.
- `validation.escape.jsdoc-delimiters`: Scan delimiters while respecting quoted strings and escapes: a regex `)`, `@` or brace inside a String cannot terminate the annotation. JSDoc `*/` still ends its comment; represent a literal star/slash sequence using JSON `*\/` or a native constant. HTML entities have no decoding role. JSON `\b` is U+0008, not a regex boundary; JSON has no `\u{...}` escape, so double that backslash.

### ValidationRules literal layer

Owner: [validation-rules.md#validationrules-literal-layer](../reference/validation-rules.md#validationrules-literal-layer)

- `validation.literal.rule-escapes`: The existing rule lexer permits single and double quotes; the matching unescaped quote closes. A backslash followed by n/r/t emits LF/CR/tab; followed by **any other character**, it emits that character and drops the backslash. Thus `\\` emits one backslash, `\"` quote and `\'` apostrophe. Missing closing quote or dangling escape is a syntax error. These identity escapes remain part of the rule literal grammar.
- `validation.source.enclosing-string-valid`: In particular, rule `\b \f \v \0` decode to ordinary b/f/v/0, and a backslash followed by an actual line break preserves that line break rather than joining lines. Raw controls are accepted by the existing rule lexer; their enclosing source representation must still be valid. To request regex form-feed/vertical-tab escapes, transmit `\\f`/`\\v`. For NUL or backspace, transmit regex `\\x00`/`\\x08` or an actual scalar; regex `\0`/`\b` are unsupported regardless of their meaning in a host regex engine.
- `validation.literal.double-backslash`: Double every backslash intended for regex: rule `"\\d+"` transmits regex `\d+`, whereas `"\d+"` transmits `d+`. Similarly `"\\q"` transmits an invalid regex escape, while `"\q"` transmits ordinary q. Rule `\u`/`\x` are identity escapes: write `"\\u{1F600}"` to transmit that regex Unicode escape.
- `validation.literal.outer-envelope`: An outer Rust ValidationRules literal decodes first; prefer raw Rust. Existing TypeScript LibertasValidationRules JSDoc preserves raw quoted annotation content, unlike the new JSON-decoded Pattern. Use a single-quoted outer envelope and double-quoted rule strings:

  ```rust
  #[libertas_validation_rules(r#"matches([], "\\d+") : "DIGITS";"#)]
  pub digits: String,
  ```

  ```typescript
  /** @LibertasValidationRules('matches([], "\\d+") : "DIGITS";') */
  digits: string;
  ```

- `validation.literal.mechanical-encoder`: For a mechanical rule-literal encoder, choose double quotes and escape every backslash as `\\`, both quote kinds as `\"` and `\'`, and LF/CR/tab as `\n`/`\r`/`\t`. Other scalars remain unchanged. Escaping the apostrophe is redundant for the rule's double-quote delimiter but makes the same text safe inside the legacy TypeScript single-quoted envelope. Preserve that rule text verbatim in the envelope; do not apply JSON or JavaScript string encoding to it. JSDoc's physical `*/` restriction still applies to the whole comment, including the legacy envelope. A regex slash can instead be spelled `\x2F` (double its backslash in the rule). Native constants use their existing annotation-specific resolution; they do not acquire interpolation semantics.

### Regex layer and exact-value examples

Owner: [validation-rules.md#regex-layer-and-exact-value-examples](../reference/validation-rules.md#regex-layer-and-exact-value-examples)

- `validation.regex.quotes`: Quotes/apostrophes are not regex metacharacters. Escape them only for their source/rule delimiter or choose a raw delimiter. A backslash before a closing quote is interpreted by the enclosing literal first; literal final backslash requires the complete doubling above. Regex `; : @ ) { }` inside a quoted argument cannot end a call, rule or annotation.

### 14.1 DateOnly literals

Owner: [validation-rules.md#141-dateonly-literals](../reference/validation-rules.md#141-dateonly-literals)

- `validation.date-literal.constant`: A literal is a constant; the language has no current-date value (section 17).
- `validation.date-literal.no-datetime`: There is no DateTime literal. A written date and time names a different instant in each time zone, and rules have no time-zone input; instants come from DateTime paths and DateTime arithmetic.

### 14.2 Bitflags membership

Owner: [validation-rules.md#142-bitflags-membership](../reference/validation-rules.md#142-bitflags-membership)

- `validation.bitflags.semantics`: For a non-null host value `h` and the member's `Fixed` value `m`, the result is the Boolean `(h & m) != 0`: true when that bit is set, whatever other bits are set. `m` is a schema constant, so evaluation stays pure (section 17). A `null` host yields `null`, which is `false` in Boolean context (section 7.1); the integer itself is never converted to a Boolean. Bitflags hosts are non-negative: a negative host value is outside the catalog and the test yields `null`, as for a null host. Where absence matters, combine the test with an explicit null test or a fallback:

  ```c
  [facts] != null && !([facts] has PersonFacts.Student)
  ([facts] has PersonFacts.ReleasedToYou) | false
  ```

### 15. Operator Precedence

Owner: [validation-rules.md#15-operator-precedence](../reference/validation-rules.md#15-operator-precedence)

- `validation.precedence.table`: From highest precedence to lowest:

  ```text
  1.  ()
  2.  path access
  3.  unary ! + -
  4.  * / %
  5.  + -
  6.  |
  7.  is has
  8.  < <= > >=
  9.  == !=
  10. &&
  11. ||
  ```

- `validation.precedence.has-fallback`: `has` shares the `is` level: both test one value against a schema name and produce a Boolean, so arithmetic and `|` bind first, while comparisons, equality, and logical operators take the result. A fallback for a `has` result therefore needs parentheses, as in `([facts] has PersonFacts.Student) | false`.
- `validation.precedence.function-primary`: A function call or collection operation is a primary expression: its parenthesized arguments bind first, so `year([start]) + 1` adds one to the year and `add_months([start], 24) - 1` is the day before the shifted date.
- `validation.precedence.parentheses` (judgment): Parentheses SHOULD be used when they improve clarity.

### 16. Static Rule Validation

Owner: [validation-rules.md#16-static-rule-validation](../reference/validation-rules.md#16-static-rule-validation)

- `validation.static.path-roots`: Square paths are validated against the attribute evaluation root. Reference paths are syntax-checked but their value type remains unknown and nullable, because the referenced response may use a different compatible schema.
- `validation.static.nullable-warning`: Studio warns when an applicable rule reads a nullable host or path prefix without explicit absence handling. This includes a named-type rule at a nullable use and a selected nullable Union payload. Under sections 1, 3 and 4.4, cumulative rules still evaluate with a present null host; descendant reads yield null and a relational comparison then returns false (section 10).
- `validation.static.absence-not-pass`: Absence never makes a rule pass automatically. For an optional record, write `[] == null || ...` when the rule should accept its absence.
- `validation.static.union-paths`: A path does not need to apply to every union alternative.

### 17. Evaluation Purity

Owner: [validation-rules.md#17-evaluation-purity](../reference/validation-rules.md#17-evaluation-purity)

- `validation.purity.must`: Validation expressions MUST be pure.
- `validation.purity.forbidden-side-effects`: A rule MUST NOT:
- `validation.purity.reference-data`: Reading the supplied `referencedData` is permitted and does not access unprovided ambient state. Evaluation does not fetch a response or consult interaction history.
- `validation.purity.deterministic-results`: MUST always produce the same result.

### 18. Validation Result

Owner: [validation-rules.md#18-validation-result](../reference/validation-rules.md#18-validation-result)

- `validation.result.same-language`: The same expression language and evaluation semantics apply when a client performs incremental validation during editing and when it performs final validation. There is no separate relaxed or final expression dialect. Incremental validation reports the rule result for the current, possibly incomplete datum after omitting declarations whose leading directive is `FinalOnly=true`. Final validation evaluates every declaration, including those final-only rule sets, and combines the result with all applicable static schema checks.
- `validation.result.top-level-null`: Because `null` becomes `false` in boolean context:

  ```text
  top-level null -> INVALID
  ```

- `validation.result.first-failure.authoring`: Order rule/message pairs deliberately: [the first failing rule wins](../reference/validation-rules.md#18-validation-result), with ordinary short-circuiting.
- `validation.output-only.excluded`: An [OutputOnly](../reference/protocols.md#output-only-request-fields) request node is outside client validation: the client checks neither its static constraints nor the rule sets hosted on it or inside it, incrementally or finally. Rule sets hosted elsewhere that read it evaluate as usual.

### 19. Constraint Completeness

Owner: [validation-rules.md#19-constraint-completeness](../reference/validation-rules.md#19-constraint-completeness)

- `validation.reference.changes-result`: Changing `referencedData` may change this subset without changing the schema, rule, or current data. For example, the rule `{$.maximum} == null || [] <= {$.maximum} : "ABOVE_MAXIMUM";` accepts `50` when the supplied maximum is `100`, rejects it when the maximum is `40`, and accepts it when there is no reference context because of its explicit null guard.

### 20.7 Named type, field, and function ownership

Owner: [validation-rules.md#207-named-type-field-and-function-ownership](../reference/validation-rules.md#207-named-type-field-and-function-ownership)

- `validation.example.cumulative-ownership`: The `percentage` value must satisfy both the named-type and field-local rules. The function rule sees minimum and maximum in one root. The explicit null guards are valid even though the source types are not nullable, because the editor may not have received user input yet. If one of those arguments remains `null` at final submission, its validation rule may pass while the independent mandatory-field check still rejects the task data.

### 20.10 Bitflags membership

Owner: [validation-rules.md#2010-bitflags-membership](../reference/validation-rules.md#2010-bitflags-membership)

- `validation.bitflags.no-bit-arithmetic` (judgment): When the person's facts include `ReleasedToYou`, the support amount is required, regardless of which other flags are set. Without the operator the same rule needs the bit position, as in `[facts] % 2 == 1`.

### 21. Core Design Principles

Owner: [validation-rules.md#21-core-design-principles](../reference/validation-rules.md#21-core-design-principles)

- `validation.principles.cursor-traversal`: **Arrays use cursor traversal and collection operators rather than general-purpose loops.**

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### 3. Rule Set

Owner: [validation-rules.md#3-rule-set](../reference/validation-rules.md#3-rule-set)

- `validation.syntax.pair`: Every Boolean expression is followed by one colon, a quoted StringResources key and a semicolon; a bare expression is invalid
- `validation.syntax.message-key`: Each message key is declared in the package StringResources with a non-empty default-locale value
- `validation.finalonly.directive`: FinalOnly, when present, is the single leading directive, written once, with the lowercase value true or false

### 3.1 Dynamic numeric ranges

Owner: [validation-rules.md#31-dynamic-numeric-ranges](../reference/validation-rules.md#31-dynamic-numeric-ranges)

- `validation.range.one-expression`: A RangeMin or RangeMax holds exactly one value expression, with no message pair or FinalOnly directive
- `validation.range.bound-type`: A range expression produces a bound of the host's semantic type, never a Boolean predicate

### 5.1 `is` Operator

Owner: [validation-rules.md#51-is-operator](../reference/validation-rules.md#51-is-operator)

- `validation.is.static-names`: Studio rejects an `is` name that no member of a known Enumeration or no variant of a known Union can match

### 7.2 No General Truthiness

Owner: [validation-rules.md#72-no-general-truthiness](../reference/validation-rules.md#72-no-general-truthiness)

- `validation.boolean.no-truthiness`: Non-null values other than Booleans never convert to Boolean; use an explicit comparison

### 9. Equality Operators

Owner: [validation-rules.md#9-equality-operators](../reference/validation-rules.md#9-equality-operators)

- `validation.equality.semantic-types`: Equality uses semantic types: Numbers compare with Numbers, DateOnly only with DateOnly, DateTime only with DateTime

### 10. Relational Operators

Owner: [validation-rules.md#10-relational-operators](../reference/validation-rules.md#10-relational-operators)

- `validation.relational.compatible`: Relational operands are compatible: Numbers with Numbers, a DateOnly or DateTime with the same type, Strings with Strings

### 11.1 Calendar-date functions

Owner: [validation-rules.md#111-calendar-date-functions](../reference/validation-rules.md#111-calendar-date-functions)

- `validation.calendar.dateonly-argument`: `year`, `month`, `day`, `add_months` and `add_years` take a DateOnly; a Number, String, collection or DateTime is a static error
- `validation.calendar.integer-count`: The `add_months` or `add_years` count is integer-typed; a floating-point Number is a static error

### 12. Collection Operations

Owner: [validation-rules.md#12-collection-operations](../reference/validation-rules.md#12-collection-operations)

- `validation.collection.array-sources`: Traversal sources are data paths that resolve only to arrays

### 12.4 `collect`

Owner: [validation-rules.md#124-collect](../reference/validation-rules.md#124-collect)

- `validation.collect.guard-specificity`: A deeper path uses the most specific compatible guard; repeated guards intersect and contradictory guards are rejected
- `validation.cursor.lexical`: Cursors are lexical: inner traversals may read outer cursors but cannot reuse a name in scope or leak it

### 12.5 `is_unique_set`

Owner: [validation-rules.md#125-isuniqueset](../reference/validation-rules.md#125-isuniqueset)

- `validation.unique.key-required`: The cursor form of `is_unique_set` needs at least one key
- `validation.unique.key-single-value`: Each `is_unique_set` key produces one value per element, not a collection

### 12.8 `sum`

Owner: [validation-rules.md#128-sum](../reference/validation-rules.md#128-sum)

- `validation.sum.numeric-leaves`: Every leaf `sum` selects or computes is an ordinary integer or floating-point Number, including Money

### 13.1 String resource references

Owner: [validation-rules.md#131-string-resource-references](../reference/validation-rules.md#131-string-resource-references)

- `validation.resource.formattedtext-argument`: `resource()` requires a FormattedText argument
- `validation.resource.declared-key`: A string literal compared with a `resource()` result is a key the package StringResources declares

### 13.2 Full-string patterns

Owner: [validation-rules.md#132-full-string-patterns](../reference/validation-rules.md#132-full-string-patterns)

- `validation.matches.arguments`: `matches()` takes exactly two String or null arguments; Number, date, Boolean, collection and FormattedText arguments are rejected
- `validation.matches.text-free`: Text-free presentation conditions reject `matches()`

### Pattern attribute and lifecycle

Owner: [validation-rules.md#pattern-attribute-and-lifecycle](../reference/validation-rules.md#pattern-attribute-and-lifecycle)

- `validation.pattern.placement`: Pattern goes once on an effective String node, never on packages, functions, containers, Enumerations, numbers, dates, Bytes or FormattedText
- `validation.pattern.studio-checks`: Studio validates Pattern placement, multiplicity and grammar, and that non-null Default and Fixed values match

### Portable regex language

Owner: [validation-rules.md#portable-regex-language](../reference/validation-rules.md#portable-regex-language)

- `validation.regex.classes`: In a class only a leading `^` negates and `-` is literal first or last; `[` and `]` are escaped; empty, nested and reversed classes are rejected
- `validation.regex.punctuation`: Outside classes the listed punctuation is syntax; unmatched closers, misplaced quantifiers and malformed counts are invalid
- `validation.regex.escapes`: Unknown escapes, surrogate-pair escapes, flags, backreferences, lookaround and similar host-engine features are errors
- `validation.regex.hex-escapes`: Hex escapes take exactly 2 or 4 digits, or 1–6 braced digits; incomplete, out-of-range and surrogate escapes are errors
- `validation.regex.limits`: Patterns are limited to 4096 scalars, 64 group nesting, repeat counts up to 10000 and 65536 compiled states

### Source attribute layer

Owner: [validation-rules.md#source-attribute-layer](../reference/validation-rules.md#source-attribute-layer)

- `validation.escape.unpaired-surrogate`: An unpaired surrogate in decoded regex text is a Studio semantic error

### 14. Expression Grammar

Owner: [validation-rules.md#14-expression-grammar](../reference/validation-rules.md#14-expression-grammar)

- `validation.grammar.ebnf`: Rule sets follow the normative grammar; `length`, `resource`, `year`, `month` and `day` take one argument, `add_months`, `add_years` and `matches` two

### 14.1 DateOnly literals

Owner: [validation-rules.md#141-dateonly-literals](../reference/validation-rules.md#141-dateonly-literals)

- `validation.date-literal.valid-date`: A DateOnly literal names a valid Gregorian date with a year from 1000 through 9999

### 14.2 Bitflags membership

Owner: [validation-rules.md#142-bitflags-membership](../reference/validation-rules.md#142-bitflags-membership)

- `validation.bitflags.host-operand`: The left operand of `has` is a current-data path to a Bitflags host, without array fan-out
- `validation.bitflags.member-operand`: The right operand of `has` is one word `Catalog.Member`: the host's own catalog type name and one of its nonzero members as declared
- `validation.bitflags.one-member`: `has` takes one nonzero member; test several flags with `&&` or `||` and the empty selection with `== 0`

### 16. Static Rule Validation

Owner: [validation-rules.md#16-static-rule-validation](../reference/validation-rules.md#16-static-rule-validation)

- `validation.static.compatible-operators`: Studio verifies that operators other than the first-non-null operator apply to compatible types
- `validation.static.boolean-results`: Every complete rule and every `all`, `any`, `none` and `collect` predicate produces a Boolean or possibly null
- `validation.static.is-type`: An `is` test names a schema-defined type
- `validation.static.cursor-use`: A cursor variable is only the first segment of a data path, within its lexical traversal
- `validation.static.resolvable-path`: Every path is resolvable through at least one schema-valid structural route

<!-- /generated -->
