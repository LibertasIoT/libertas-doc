# Libertas Data Validation Rules Specification

**Version:** 1.11

This document is normative for the `ValidationRules` schema attribute,
including client evaluation and presentation of each rule's supplied message.

## 1. Purpose

Libertas Data Validation Rules define a small, statically analyzable language for expressing semantic validity constraints over schema-compliant data.

The input to validation is:

1. a schema \(\Sigma\),
2. one concrete or partially entered `data` instance, and
3. optional `referencedData`: the reference context supplied to evaluation,
   containing its ordered reference objects, or `null` when no context is available.

A rule is evaluated as `Eval(rule, data, referencedData)`. The schema supplies
the rule and type/path interpretation. Under the
[shared reference context](schema.md#context-and-syntax), ordinary curly
references read the [current interaction reference context](protocols.md#interaction-reference-context).
`referencedData` carries that lookup environment without changing this
evaluation signature. External references follow its source order and the
[shared first-non-null lookup rule](schema.md#availability-null-and-interaction-ownership).
It does not imply a fetch, I/O, stored history, or a merged reference object.

The rule language does not define GUI presentation, behavior, state transitions, mutation, or effects.

<!-- rule: validation.purity.expression class=hard enforced=no tasks=validation-rules -->
A rule is a pure expression of `data` and `referencedData`. During editing, any field
MAY temporarily be undefined (unanswered), including a `Nullable` field whose
[Nullable choice](presentation.md#nullable-choices) has no answer yet.
Undefined is local editor state, never encodable and never an answer; `null`
in a rule always means a present null, such as a Nullable's chosen null
answer. <!-- rule: validation.unanswered.aborts class=hard enforced=no tasks=validation-rules --> **A rule that reads an unanswered value is aborted:** the client does
not report its result and reports that value as a missing answer instead, in
incremental and final validation alike. During editing the missing answer is
the field's unanswered indicator, never a failure that rejects an incremental
change; at final validation it blocks submission like any unanswered field. A rule's result therefore never
depends on an unanswered value, and no rule distinguishes null from undefined. <!-- rule: validation.null.fallback-not-answer-test class=judgment enforced=no tasks=validation-rules -->
Within a rule, `null` serves as a fallback value, chiefly through
[`|`](#73-first-non-null-value-fallback), not as a test of whether the person answered: whether
a field must be answered is structural, declared by a non-`Nullable` field or
a `Nullable` field without a Default. <!-- rule: validation.null.accept-not-optional class=hard enforced=no tasks=validation-rules --> Mandatory-field validation and
`ValidationRules` are complementary, independent requirements. A rule MAY
accept a `null`, but that success does not make the field optional or
suppress the final mandatory-field check. <!-- rule: validation.final.requirements class=hard enforced=no tasks=validation-rules -->
Final submission separately requires structural schema validity, every field
answered (a non-`Nullable` field present, a `Nullable` field null or present),
and all validation rules to pass. <!-- rule: validation.bytes.mime-empty-null class=hard enforced=no tasks=validation-rules --> A zero-length
[MIME-aware Bytes](schema.md#mime-aware-bytes) value is a present, encodable
Bytes field but has undefined media meaning and evaluates as null. Its
non-`Nullable` declaration and size bounds do not require media selection;
an App requiring a file declares a rule such as
`[] != null : "IMAGE_REQUIRED";`. <!-- rule: validation.bytes.lmb1-non-null class=hard enforced=no tasks=validation-rules --> A defined LMB1 value evaluates as non-null,
even when its content is empty. Ordinary Bytes keep their existing valuation.

<!-- rule: validation.placement.contexts class=hard enforced=no tasks=validation-rules -->
ValidationRules uses the [shared schema reference contract](schema.md#availability-null-and-interaction-ownership).
Rules may occur in editable Request/SubscriptionRequest data, inherited types,
list items, and hidden/read-only fields. External reference reads require neither
CopyFrom nor request-side copies of server metadata.

<!-- rule: validation.reference.first-composition class=hard enforced=no tasks=validation-rules -->
Authors MUST account for first-time request composition with no reference context:
every curly reference evaluates to `null` in that state.
ValidationRules must handle that absence explicitly, for example with a null
guard or a typed `|` fallback. The client still evaluates the rule normally;
absence does not automatically make the rule pass. <!-- rule: validation.reference.no-fetch class=hard enforced=no tasks=validation-rules --> Neither validation nor
request composition fetches server data to resolve these references.

<!-- rule: validation.valid.all-true class=hard enforced=no tasks=validation-rules -->
For the supplied `referencedData`, the data instance is valid iff every
validation rule evaluates to `true`.

\[
Valid_\Sigma(data,referencedData) \iff \bigwedge_{rule\in R} Bool(Eval(rule,data,referencedData))
\]

---

## 2. Data Model

A schema MAY contain:

- primitive values
  - boolean
  - integer
  - floating-point number
  - string
  - DateOnly calendar date
  - DateTime instant
  - other schema-defined primitive types
- structs
- arrays
- nullable values
- union types

Example:

```c
struct A {
    int value;
}

struct B {
    string name;
}

struct Root {
    A | B item;
    float? limit;
    Entry[] entries;
}
```

<!-- rule: validation.editor.total-on-incomplete-null class=hard enforced=no tasks=client-runtime -->
Structural schema compliance is outside the scope of the validation-rule
language. An evaluator MUST nevertheless tolerate the transient `null` values
used by client editors and MUST remain total for that incomplete state.

<!-- rule: validation.types.semantic class=hard enforced=no tasks=validation-rules -->
Every successful intermediate evaluation carries an internal semantic type;
operators MUST NOT infer a new type merely from the scalar's storage encoding.
In particular, DateOnly is a calendar type even though its schema and wire
encoding is the integer `yyyyMMdd`, and DateTime is an instant type even though
its schema and wire encoding is integral Unix epoch seconds. Literals, paths,
fallback results, operator results, and function results all retain or produce
an explicit internal type. `null` has its own type. <!-- rule: validation.types.date-literal-only class=hard enforced=no tasks=validation-rules --> A
[DateOnly literal](#141-dateonly-literals) has type DateOnly; the language has
no DateTime literal.

<!-- rule: validation.types.bitflags-host class=hard enforced=no tasks=validation-rules -->
A [Bitflags](schema.md#43-value-constraint-and-presentation-attributes) host,
an integer Number whose `Bitflags` attribute names a flag catalog, is an
ordinary integer in rules. Its catalog's members are schema constants that
appear only as the right operand of the [membership test `has`](#142-bitflags-membership).

---

## 3. Rule Set

Rules are stored in one string-valued schema attribute:

```json
{ "Name": "ValidationRules", "Value": "[min] <= [max] : \"INVALID_RANGE\";" }
```

<!-- rule: validation.declare.owners class=hard enforced=no tasks=validation-rules -->
`ValidationRules` MAY be declared on:

- a schema function, where the evaluation root is a synthetic Table containing
  all function arguments under their schema names;
- any schema data field; or
- any schema data type, including a named primitive, Table, List, Union, or
  other named Libertas type.

<!-- rule: validation.declare.cumulative class=hard enforced=no tasks=validation-rules -->
A field/type rule is rooted at each concrete value of the annotated node. A
named-type rule and a field-local rule are cumulative when a field references
that named type. Inherited field rules are cumulative as well. <!-- rule: validation.paths.root class=hard enforced=no tasks=validation-rules --> The empty data
path `[]` denotes the annotated value itself. Function rules address arguments
by name, such as `[minimum]` and `[options/timeout]`.

<!-- rule: validation.declare.spelling class=hard enforced=no tasks=validation-rules -->
The TypeScript source spelling is
`@LibertasValidationRules("...")`. The Rust source spelling is
`#[libertas_validation_rules("...")]`. <!-- rule: validation.declare.rust-helper class=hard enforced=no tasks=validation-rules --> A Rust named type may use the
standalone attribute; fields and variants use the `LibertasExport` helper
attribute. Parsers emit the raw string and source location. Libertas Studio's
shared schema validator owns grammar and schema-decidable semantic validation.

<!-- rule: validation.syntax.pair class=hard enforced=yes tasks=validation-rules title="Every Boolean expression is followed by one colon, a quoted StringResources key and a semicolon; a bare expression is invalid" -->
A validation rule set is a sequence of expression/message pairs. Every Boolean
expression is followed by one `:`, a quoted package `StringResources` key, and
the terminating `;`:

```text
[min] <= [max] : "INVALID_RANGE";
```

<!-- rule: validation.syntax.message-key class=hard enforced=yes tasks=validation-rules title="Each message key is declared in the package StringResources with a non-empty default-locale value" -->
The key is metadata for that complete expression and does not participate in
Boolean evaluation. It MUST be declared by the package `StringResources`
catalog and have a non-empty value in the default-locale document. Studio
enforces both requirements. A bare `expression;` is invalid.

A rule set MAY begin with this case-sensitive phase directive:

```text
FinalOnly=true;
```

or explicitly:

```text
FinalOnly=false;
```

<!-- rule: validation.finalonly.default class=hard enforced=no tasks=validation-rules -->
The default is `false`. <!-- rule: validation.finalonly.directive class=hard enforced=yes tasks=validation-rules title="FinalOnly, when present, is the single leading directive, written once, with the lowercase value true or false" --> The directive, when present, MUST be the first item,
MUST occur only once, and accepts only the lowercase literals `true` and
`false`. It is metadata inside the `ValidationRules` string; it is not an
individual schema attribute.

<!-- rule: validation.finalonly.incremental-tolerance class=judgment enforced=no tasks=validation-rules -->
An omitted directive or `FinalOnly=false` places the rule set in both
incremental editor validation and final validation. Because an incremental
failure restores the previous value, such a rule set MUST tolerate incomplete
editing; an unanswered value never reaches a rule. <!-- rule: validation.finalonly.true class=hard enforced=no tasks=validation-rules --> `FinalOnly=true` explicitly
omits the entire rule set from incremental validation and evaluates it only
during final validation. This removes the need for that rule set to tolerate an
initial incomplete editor state. It does not alter `null` expression semantics, change `Nullable`, or
replace final static schema checks. <!-- rule: validation.finalonly.per-declaration class=hard enforced=no tasks=validation-rules --> When field, inherited, and named-type rule
sets accumulate, each declaration retains its own `FinalOnly` value.

### Authoring guidance

<!-- rule: validation.authoring.structural-first class=judgment enforced=no tasks=validation-rules -->
Use the schema's precise type and ordinary structural attributes for
nullability, numeric ranges, sizes, and uniqueness. `ValidationRules` express
semantic relationships that those declarations cannot express; they do not
restate or replace structural constraints.

<!-- rule: validation.authoring.required-is-structural class=judgment enforced=no tasks=validation-rules -->
When the person must answer a field, declare it structurally: a non-`Nullable`
field, or a `Nullable` field without a Default, whose
[Nullable choice](presentation.md#nullable-choices) starts unanswered. Do not
restate that requirement as a `!= null` rule; an unanswered value never
reaches a rule.

### Where a check belongs

<!-- rule: validation.placement.save-path-authoritative class=judgment enforced=no tasks=validation-rules -->
The App's save path is the authoritative validator. It checks every Request it
receives in full, whether or not a client evaluated any rule. Client rules are
a convenience for the person editing; they are never a security or correctness
boundary, and never the only copy of a check.

| Check | Where it belongs |
| --- | --- |
| <!-- rule: validation.placement.structural-checks class=judgment enforced=no tasks=validation-rules --> Type, nullability, numeric range, size, uniqueness, String format | Schema declarations and `Pattern`. The client enforces them while editing, and the SDK's Avro decoder enforces each static bound (a number's `min`, `max` and `step` included), and that every DateOnly value is a real Gregorian date, when the App decodes a Request, so App code and tests never re-check them. Dynamic `RangeMin`/`RangeMax` limits stay save-path checks. |
| <!-- rule: validation.placement.local-feedback class=judgment enforced=no tasks=validation-rules --> Simple, local relationship where immediate feedback helps the person (one field, or fields on the same page) | Optionally a rule set with the directive omitted (incremental and final), mirroring a save-path check. |
| <!-- rule: validation.placement.business-logic class=judgment enforced=no tasks=validation-rules --> Business logic: cross-page or cross-record consistency, legal or eligibility rules, anything needing App state | The save path only. Reject with an [Error response](protocols.md#protocol-roles) and name the field with [`ErrorFieldPath`](protocols.md#error-field-path). |

<!-- rule: validation.placement.final-only-discouraged class=judgment enforced=no tasks=validation-rules -->
Final-only rule sets (an explicit `FinalOnly=true;`) are
discouraged. They run at submission, the same moment as the save-path check, so
they add no earlier feedback; they only add a second copy of the logic that can
drift from the save path or fail silently. Put submission-time checks in the
save path instead. Final-only rule sets remain valid and are still evaluated.
Omitting the directive means the rule set is also evaluated incrementally;
final-only must be written explicitly.

Example:

```c
[min] <= [max] : "INVALID_RANGE";

[value] >= [min] &&
[value] <= [max] : "VALUE_OUT_OF_RANGE";

[end] > [start] : "END_BEFORE_START";
```

All rules are combined using logical AND.

Therefore:

```c
r1 : "ERROR_1";
r2 : "ERROR_2";
r3 : "ERROR_3";
```

is semantically equivalent to:

```c
r1 && r2 && r3 : "COMBINED_ERROR";
```

A data instance is valid only when every rule evaluates to `true`. <!-- rule: validation.messages.split-for-presentation class=judgment enforced=no tasks=validation-rules --> The two
forms above are validity-equivalent but intentionally not
presentation-equivalent: the first form can display the key belonging to the selected
failed rule, while the combined expression owns one message key.

### Debugging rules

<!-- rule: validation.debug.trace class=hard enforced=no tasks=validation-rules -->
Studio offers an opt-in developer trace for each applicable rule. It records
the expression text, message key, and source file and one-based line when
known (otherwise null), and a `passed`, `failed`, or `undecided` outcome.
`undecided` means an evaluated read encountered an unanswered editor value;
it is not a rule failure. Null or missing hosts and inactive Union payloads
remain ordinary absence under the path rules, not unanswered values.

<!-- rule: validation.debug.only-evaluated-branches class=hard enforced=no tasks=client-runtime -->
The trace records evaluated paths as typed values, unanswered reads, absent
values, or ordered List fan-out, and records evaluated operator and function
results, including `|`, `is`, and `has`. Numeric results retain exact rational
values; date results retain their semantic type. Skipped branches and cursor
iterations are never evaluated for debugging. Developer evaluation may inspect
all applicable rules independently in declaration order while returning the
same first violation as ordinary evaluation. Without a requested trace,
ordinary evaluation retains its existing work and first-failure behavior.

<!-- rule: validation.debug.no-values-in-ui class=hard enforced=no tasks=client-runtime -->
The control view's developer-only **Why** disclosure shows this trace under
a failing or undecided rule. Expressions and entered values never appear in
normal user presentation. Debugging changes neither stored data nor client
protocols, and a trace does not replace structural validation.

Use [`evaluate_validation_rules`](deployment.md#mcp-deployment) for saved
App source and supplied data. For CI, `test-rules <fixtures...>` is a mode of
the same installed Studio schema compiler launcher configured for MCP:
`node /path/to/launcher.cjs test-rules fixtures/*.json`. It uses the same
compilation and evaluation. <!-- rule: validation.debug.fixtures class=hard enforced=no tasks=validation-rules --> Each JSON fixture supplies `project`, optional
`entry` and `phase`, `type`, `value` in schema JSON value form, and `expected`,
an object mapping message keys to `passed`, `failed`, or `undecided`.
Relative project paths resolve from the fixture's directory. The runner reports
every fixture and exits zero only when compilation and all expectations pass;
invalid fixtures and mismatches exit nonzero. <!-- rule: validation.debug.expectations class=hard enforced=no tasks=validation-rules --> Every evaluated rule key requires
an expectation, and unknown expected keys fail. For a key evaluated at several
value occurrences, its outcome is failed if any occurrence fails, otherwise
undecided if any is undecided, otherwise passed. A rule expected to fail is a
successful test when it fails as expected. Standard output is one JSON report
containing per-fixture results, diagnostics, traces and a summary.

## 3.1 Dynamic numeric ranges

<!-- rule: validation.range.purpose class=hard enforced=no tasks=validation-rules -->
`RangeMin` and `RangeMax` reuse the Data Validation expression language to
derive live inclusive constraints for a Number or DateTime value. <!-- rule: validation.range.spelling class=hard enforced=no tasks=validation-rules --> Their source spellings
are `@LibertasRangeMin("expression")` / `@LibertasRangeMax("expression")` in
TypeScript and `#[libertas_range_min("expression")]` /
`#[libertas_range_max("expression")]` in Rust. The language-neutral schema
stores the decoded expression string under `RangeMin` or `RangeMax`.

<!-- rule: validation.range.number-annotation class=hard enforced=no tasks=validation-rules -->
The numeric annotation also accepts a quoted expression in `min` or `max`:
`@LibertasNumber(type=i64, min=0, max="[^/maximum]")` or
`#[libertas_number(min=0, max="[^/maximum]")]`. Unquoted numeric literals
and Rust `$CONST` values continue to emit `NumberMin`/`NumberMax`; quoted
expressions emit `RangeMin`/`RangeMax`, exactly like the standalone range
annotations. Rust source constants inside expression text use the existing
constant substitution. `step` remains a fixed numeric value (or Rust source
constant). No additional schema attributes are introduced.

<!-- rule: validation.range.one-expression class=hard enforced=yes tasks=validation-rules title="A RangeMin or RangeMax holds exactly one value expression, with no message pair or FinalOnly directive" -->
Each attribute contains exactly one value expression. It MUST NOT contain a
validation message pair (`: "KEY";`), multiple expressions, or a `FinalOnly`
directive. <!-- rule: validation.range.bound-type class=hard enforced=yes tasks=validation-rules title="A range expression produces a bound of the host's semantic type, never a Boolean predicate" --> The expression MUST produce a bound compatible with the annotated
node's semantic type, not a Boolean validation predicate. Studio MUST validate
its grammar, paths, calls, operand types, and result type against that node.  A
[result type known to be incompatible](#31-dynamic-numeric-ranges) MUST be rejected during schema validation;
unknown reference types remain subject to runtime validation. Nullability does
not make a known incompatible non-null result type valid.

<!-- rule: validation.range.context class=hard enforced=no tasks=validation-rules -->
Evaluation uses the same field-local context as `ValidationRules`: `[]` is the
annotated Number or DateTime and square paths traverse from it. The shared
[current-data path contract](schema.md#current-data-references-in-validation-expressions)
permits a leading parent prefix: `[^/minimum]` reads the current row's
`minimum`, and `[^^/limits/maximum]` reads a bound two levels above the field. <!-- rule: validation.range.live-binding class=hard enforced=no tasks=client-runtime -->
Limits read current values whenever the normal editor refreshes and during
final validation; rows bind independently, including after insertion, removal,
reordering, or replacement. <!-- rule: validation.range.reference-context class=hard enforced=no tasks=validation-rules --> Curly paths
inspect the [current interaction reference context](protocols.md#interaction-reference-context),
including in editable requests. The result type must match the control's semantic type:

| Annotated control | Required non-null result |
| --- | --- |
| Ordinary Number, including TimeInterval | Finite Number; integer and fractional bounds are permitted. |
| DateOnly | DateOnly calendar date. |
| DateTime | DateTime instant. |


[A numeric literal or an ordinary Number path](#31-dynamic-numeric-ranges) is not a valid bound for a date
control, even when its digits match that date's wire representation. A fixed
DateOnly bound is a [DateOnly literal](#141-dateonly-literals), such as
`d"1990-01-01"`. A
DateOnly result becomes its valid `yyyyMMdd` boundary value and a DateTime
result becomes integral epoch seconds only at the range boundary. Expressions
retain their semantic types.

<!-- rule: validation.range.null-no-bound class=hard enforced=no tasks=validation-rules -->
`null` means that no dynamic bound is currently available, including when the
first request is composed before any reference context exists. It adds no constraint. <!-- rule: validation.range.runtime-type-error class=hard enforced=no tasks=validation-rules -->
A non-null Boolean, String, array, collection, object, mismatched semantic
type, or non-finite numeric result is an error. When an expression's result
cannot be proved compatible statically, clients MUST check it at runtime and reject
validation on an incompatible result. They MUST NOT silently remove the bound,
erase its semantic type, or coerce the result to a Number.


For example, [`@LibertasRangeMax('[] <= 100')`](#31-dynamic-numeric-ranges) is rejected because its result is
Boolean. Use `@LibertasRangeMax('100')` for a numeric maximum, or put the
Boolean predicate in `ValidationRules` with its required message key. <!-- rule: validation.range.expressions class=hard enforced=no tasks=validation-rules --> Paths
and arithmetic expressions that preserve the required type are supported; bounds are not restricted to literal constants. For example, a
DateTime path plus an integer number of seconds remains a DateTime bound;
subtracting two DateTime paths yields a Number and cannot bound DateTime.

<!-- rule: validation.range.inclusive class=hard enforced=no tasks=validation-rules -->
An active `RangeMin` is an inclusive lower bound and an active `RangeMax` is an
inclusive upper bound. <!-- rule: validation.range.intersection class=hard enforced=no tasks=validation-rules --> They intersect the native numeric type limits and any
static `NumberMin` / `NumberMax` constraints. Clients MUST apply that same
intersection to final validation and to numeric, DateOnly, DateTime, and TimeInterval
GUI controls. A changing dependency value MUST update the live GUI constraint.
DateOnly bounds use the `yyyyMMdd` wire representation at the schema boundary,
but Data Validation evaluates the value as a real Gregorian calendar date. <!-- rule: validation.range.datetime-backing class=hard enforced=no tasks=validation-rules -->
DateTime remains a distinct UI/wire and expression type but is
constraint-backed as an unsigned integral Unix timestamp. Static Number constraints use
that wire representation; dynamic Range expressions require a DateTime result.
Their intersection applies to both its local date/time chooser and final UTC
epoch-seconds validation. <!-- rule: validation.range.step-unchanged class=hard enforced=no tasks=validation-rules -->
`RangeMin` and `RangeMax` do not change the `NumberStep` origin or lattice. <!-- rule: validation.range.unsatisfiable class=hard enforced=no tasks=validation-rules --> If
the active effective minimum exceeds the active effective maximum, the range
is unsatisfiable; a client MUST NOT swap the two bounds or otherwise invent a
valid interval.

<!-- rule: validation.range.absent-reference-null class=hard enforced=no tasks=validation-rules -->
A numeric field, including an editable request field, can use limits from the
current reference context. When its reference objects do not provide them, these expressions return
null and add no dynamic bound; ordinary numeric constraints still apply:

```typescript
/** Measured value
 * A numeric result constrained by the available server-provided limits.
 * @LibertasNumber(type=f64)
 * @LibertasRangeMin('{$.limits.minimum}')
 * @LibertasRangeMax('{$.limits.maximum}')
 */
value: number;
```

<!-- rule: validation.range.no-fetch class=hard enforced=no tasks=validation-rules -->
These curly references can directly constrain editable request fields. The
client reads the already received response; it does not fetch server state.
The server remains responsible for enforcing its current authoritative limits.
Authors bound a field only by values the request or its received response
already carries; the service still enforces its current limit on save.

---

# 4. Path Access

## 4.1 Syntax

Current-data references use the square-bracket expression form of
[Libertas Schema Path](schema.md#current-data-references-in-validation-expressions).
Its canonical specification defines field/index syntax, owner roots, cursors,
array fan-out, result order, and null/collection conversion.

## 4.2 Path Resolution

Apply the [Libertas Schema Path resolution rules](schema.md#current-data-references-in-validation-expressions)
to the actual current data instance. The worked examples below illustrate those
rules in validation expressions; they do not define another path language.

## 4.3 Struct Access

Given:

```c
struct User {
    string name;
}
```

and concrete data:

```text
user.name = "John"
```

then:

```c
[user/name]
```

evaluates to:

```text
"John"
```

---

## 4.4 Null Values

<!-- rule: validation.null.equality class=hard enforced=no tasks=validation-rules -->
Every field uses ordinary equality operators for its current editor value,
including a required field that has not received input yet.

```c
[email] == null
[email] != null
```

<!-- rule: validation.null.no-special-syntax class=hard enforced=no tasks=validation-rules -->
No special nullable-access syntax exists.

<!-- rule: validation.null.non-nullable-comparison class=hard enforced=no tasks=validation-rules -->
The comparison is valid even when the schema field is not `Nullable`. Client
editors use `null` as the initial state of many required fields, and rules are
allowed to describe that state before final structural validation.

For:

```c
[address] == null
```

the deeper path:

```c
[address/city]
```

also evaluates to:

```text
null
```

## 4.5 Reference paths

Curly expressions such as `{$.preferred.path,$.fallback.path}` use the
[shared path contract](schema.md#availability-null-and-interaction-ownership) for their syntax, lookup context, and
availability. During static rule checking their result is schema-unknown and
nullable. The first-composition guard is illustrated in §20.8.

---

# 5. Union Types

A schema MAY define a union:

```c
A | B | C
```

Every concrete union value has one active runtime type.

Example:

```c
struct A {
    int x;
}

struct B {
    string y;
}

A | B item;
```

---

## 5.1 `is` Operator

<!-- rule: validation.is.operator class=hard enforced=no tasks=validation-rules -->
The `is` operator tests the selected Union variant or the selected Enumeration
member of a value.

```c
[item] is A       // Union: the selected variant is A
[item] is B
[class] is Five   // Enumeration: the selected member is Five
```

<!-- rule: validation.is.union-runtime class=hard enforced=no tasks=validation-rules -->
Its result is boolean. On a Union it tests the actual runtime variant. <!-- rule: validation.is.enumeration class=hard enforced=no tasks=validation-rules --> When the
operand is an Enumeration, `Name` is a member name and the test is true exactly
when the stored value selects that member, resolved as the Enumeration itself
resolves a stored value (a member's `Fixed` value first, otherwise its index).
A null, unset, or unresolvable Enumeration value gives false. A numeric
comparison such as `[class] == 0` is unaffected.

<!-- rule: validation.is.static-names class=hard enforced=yes tasks=validation-rules title="Studio rejects an `is` name that no member of a known Enumeration or no variant of a known Union can match" -->
Studio rejects a `Name` that cannot match when the operand's type is known: on
an Enumeration, a name that is not one of its members; on a Union, a name that
no variant can ever match. `[s/distribution] is Patronage` therefore also
works as a `collect` filter.

Formally:

\[
x\ is\ T =
\begin{cases}
true & \text{if }x\text{ has runtime type }T\\
false & \text{otherwise}
\end{cases}
\]

<!-- rule: validation.is.null-false class=hard enforced=no tasks=validation-rules -->
If the left operand is `null`:

```c
null is T
```

evaluates to:

```text
false
```

Negation uses the standard `!` operator:

```c
!([item] is A)
```

---

## 5.2 Path Access Through Unions

Libertas Schema Path follows the [active Union payload](schema.md#current-data-references-in-validation-expressions).
For `item = A { x: 10 }`, `[item/x]` returns 10 and `[item/y]` returns null when
y belongs only to another variant. This uses the same absence rules as missing
fields and indices; no special Union-access syntax is introduced.

---

# 6. Array Access

## 6.1 Explicit Index

The [Libertas Schema Path index form](schema.md#current-data-references-in-validation-expressions) selects array elements, for example:

```c
[items/0]
[items/0/name]
```

If the index does not exist, the path resolves to `null`.

---

## 6.2 Cursor traversal

<!-- rule: validation.cursor.element-node class=hard enforced=no tasks=validation-rules -->
`all`, `any`, `none`, and `collect` bind a cursor to each concrete element of
the arrays selected by their source path. A cursor is the element node itself,
not its array index:

```c
all(item in [items],
    [item/value] >= 0
)
```

`[item]` denotes the bound node and `[item/value]` starts path resolution from
that node. A cursor always retains the concrete absolute path used to reach its
node, so the example's second element is rendered diagnostically as
`[items/1/value]`.

A source path MAY cross other arrays before reaching its terminal arrays. When
a named segment remains and the current node is an array, the same segment is
resolved from every element. The terminal arrays are concatenated in path and
element order, then their elements are bound in that order:

```c
all(item in [groups/items],
    [item/value] >= 0
)
```

Here `[groups/items]` selects every `items` array under every element of
`groups`. No index variable is needed, and there is no ambiguous index owner.


[The former index-binding form](#16-static-rule-validation), such as
`all(i in [items], [items/i/value] >= 0)`, is invalid. There is no compatibility
interpretation for it.

---

# 7. Boolean Semantics

## 7.1 Boolean Context

Boolean values evaluate normally:

```text
true  -> true
false -> false
```

<!-- rule: validation.boolean.null-false class=hard enforced=no tasks=validation-rules -->
`null` is implicitly converted to boolean `false`:

\[
Bool(null)=false
\]

Therefore:

```c
[enabled]
```

is valid when `[enabled]` has type:

```text
bool
```

or:

```text
bool?
```

If `[enabled]` is `null`, it evaluates as `false`.

---

## 7.2 No General Truthiness

<!-- rule: validation.boolean.no-truthiness class=hard enforced=yes tasks=validation-rules title="Non-null values other than Booleans never convert to Boolean; use an explicit comparison" -->
Non-null values other than booleans MUST NOT be implicitly converted to boolean.

The following are not valid boolean expressions merely because their values are non-null:

```c
[count]
[name]
[object]
[array]
```

<!-- rule: validation.truthiness.explicit-comparison class=hard enforced=no tasks=validation-rules -->
Explicit comparison MUST be used:

```c
[count] != null
[name] != null
[object] != null
[array] != null
```

Thus the only boolean coercion defined by this specification is:

```text
bool  -> itself
null  -> false
```

---

## 7.3 First-non-null value fallback

Single `|` is a value-preserving, left-to-right null-fallback operator:

```c
expr1 | expr2 | expr3
```

<!-- rule: validation.fallback.first-non-null class=hard enforced=no tasks=validation-rules -->
The evaluator returns the first operand whose value is not `null`, without
changing that value's type. It evaluates no later operand after finding a
winner. `false`, numeric zero, an empty string, an empty array or collection,
and an object are all non-null and therefore win. The result is `null` only
when every operand evaluates to `null`.

```c
([preferredLimit] | [legacyLimit] | 100) >= 0
[preferredEnabled] | [legacyEnabled] | false
```


The operator may produce any supported value type inside an expression, but
[the complete rule expression](#16-static-rule-validation) must still be Boolean. There is no additional
truthiness conversion: a numeric, string, array, collection, or object result
must be used with an explicit comparison or another type-appropriate operator.

<!-- rule: validation.fallback.static-types class=hard enforced=no tasks=validation-rules -->
The result's possible static types are the left operand's non-null types plus
the right operand's possible types only when the left operand may be null. A
chain is left-associative and applies that rule repeatedly.

---

# 8. Logical Operators

The language provides C-style logical operators:

```c
!
&&
||
```

Precedence, from highest to lowest:

```text
!
&&
||
```

<!-- rule: validation.logical.short-circuit class=hard enforced=no tasks=validation-rules -->
`&&` and `||` MUST use short-circuit evaluation.

Example:

```c
[address] != null &&
[address/city] != null
```

If `[address]` is `null`, the second expression need not be evaluated.

Likewise:

```c
[item] is A &&
[item/x] > 0
```

---

# 9. Equality Operators

The language provides:

```c
==
!=
```

<!-- rule: validation.equality.null class=hard enforced=no tasks=validation-rules -->
`null` is an ordinary value for equality comparison.

```c
[x] == null
[x] != null
```

The following identities apply:

```text
null == null  -> true
null != null  -> false
value == null -> false
value != null -> true
```

for every non-null `value`.

<!-- rule: validation.equality.semantic-types class=hard enforced=yes tasks=validation-rules title="Equality uses semantic types: Numbers compare with Numbers, DateOnly only with DateOnly, DateTime only with DateTime" -->
Non-null equality uses internal semantic types. Integer and floating-point
Numbers are compatible with each other. DateOnly equals only DateOnly and
DateTime equals only DateTime; neither equals a Number merely because its wire
value is numeric.

---

# 10. Relational Operators

The language provides:

```c
<
<=
>
>=
```

Examples:

```c
[age] >= 0
[min] <= [max]
[value] >= [min] && [value] <= [max]
```

<!-- rule: validation.relational.compatible class=hard enforced=yes tasks=validation-rules title="Relational operands are compatible: Numbers with Numbers, a DateOnly or DateTime with the same type, Strings with Strings" -->
Operands MUST be compatible according to the schema's type system.

<!-- rule: validation.dates.same-semantic-type class=hard enforced=no tasks=validation-rules -->
Ordinary integers and floating-point Numbers are mutually comparable.
DateOnly is comparable only with DateOnly, and DateTime only with DateTime;
their storage encodings do not make them comparable with a Number. Strings are
comparable only with strings.

<!-- rule: validation.relational.null-false class=hard enforced=no tasks=validation-rules -->
A relational comparison involving `null` evaluates to `false`.

For example:

```c
[item/x] > 0
```

evaluates to `false` if `[item/x]` resolves to `null`.

---

# 11. Arithmetic Operators

The core language provides:

```c
+
-
*
/
%
```

and unary:

```c
+
-
```

Example:

```c
[end] - [start] <= [maximum_duration]
```

<!-- rule: validation.arithmetic.exact class=hard enforced=no tasks=validation-rules -->
Numeric literals and decoded scalar decimal text are evaluated as exact rational
values. Addition, subtraction, multiplication, division, remainder, equality,
and ordering preserve the low digits of i64 Money and do not use binary64
intermediates. Division does not round a repeating fraction; remainder uses a
quotient truncated toward zero. Each reduced numerator and denominator is
limited to 256 decimal digits. Arithmetic exceeding that limit produces null,
as does division by zero. This evaluation representation does not change the
schema's integer/floating semantic types or any wire encoding. Ordinary numeric
UI controls may still use floating-point coordinates; Money bounds and final
checks retain exact values throughout.

<!-- rule: validation.arithmetic.result-types class=hard enforced=no tasks=validation-rules -->
Ordinary integer and floating-point operands use numeric arithmetic. Integer
results remain internally typed as integers except that `/` produces a Number;
an operation involving a floating-point Number produces a Number.

<!-- rule: validation.arithmetic.date-overloads class=hard enforced=no tasks=validation-rules -->
DateOnly and DateTime have these additional, exhaustive arithmetic overloads:

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


[The integer operand](#11-arithmetic-operators) MUST be internally typed as an integer; an integral value
whose declared or derived type is floating point is not sufficient. <!-- rule: validation.arithmetic.gregorian class=hard enforced=no tasks=validation-rules --> DateOnly
arithmetic MUST use Gregorian calendar rules: a DateOnly path carrying the wire
value `20240228` plus `1` evaluates to DateOnly `20240229`, and plus `2`
evaluates to DateOnly `20240301`. It MUST NOT add to the numeric `yyyyMMdd`
encoding.  [DateOnly/DateTime mixed](#16-static-rule-validation)
arithmetic, integer-minus-date, unary `+`/`-` on a date, and `*`, `/`, or `%`
on a date are invalid. Calendar-month and year shifts and the extraction of a
date's year, month, and day are the functions of section 11.1, not operators.

<!-- rule: validation.arithmetic.null class=hard enforced=no tasks=validation-rules -->
If an arithmetic operand evaluates to `null`, the arithmetic result is `null`.

Therefore:

```c
[a] + [b]
```

evaluates to `null` if either operand is `null`.

An undefined arithmetic operation, such as division by zero, also evaluates to `null`.

This keeps expression evaluation total for schema-compliant data.

## 11.1 Calendar-date functions

```c
year(date)
month(date)
day(date)
add_months(date, months)
add_years(date, years)
```

<!-- rule: validation.calendar.dateonly-argument class=hard enforced=yes tasks=validation-rules title="`year`, `month`, `day`, `add_months` and `add_years` take a DateOnly; a Number, String, collection or DateTime is a static error" -->
These functions read and shift the Gregorian calendar fields of a DateOnly
value. `date` MUST have semantic type DateOnly: a DateOnly path, a
[DateOnly literal](#141-dateonly-literals), or a fallback, function, or
arithmetic result of that type. A Number, including one whose digits spell a
`yyyyMMdd` date, a String, a collection, and a DateTime are static errors
(section 16). DateTime is excluded because an instant has no calendar date
without a time zone, and rules have no time-zone input.

<!-- rule: validation.calendar.fields class=hard enforced=no tasks=validation-rules -->
- `year(d)` returns the calendar year (1000 through 9999), `month(d)` the month
  (1 through 12), and `day(d)` the day of the month (1 through 31), each as an
  integer Number. For example, `year(d"2024-02-29") == 2024`,
  `month(d"2024-02-29") == 2`, and `day(d"2024-02-29") == 29`. <!-- rule: validation.calendar.add-months class=hard enforced=no tasks=validation-rules -->
- `add_months(d, n)` returns the DateOnly `n` calendar months after `d`, or
  before it when `n` is negative. The year and month advance by `n` months. The
  day of the month is kept when the target month has that day; otherwise it
  becomes the target month's last day:

  ```c
  add_months(d"2024-01-31", 1)  == d"2024-02-29"
  add_months(d"2023-01-31", 1)  == d"2023-02-28"
  add_months(d"2024-03-31", -1) == d"2024-02-29"
  add_months(d"2024-05-31", 1)  == d"2024-06-30"
  add_months(d"2024-01-15", 24) == d"2026-01-15"
  ```

- `add_years(d, n)` is `add_months(d, 12 * n)`, so
  `add_years(d"2024-02-29", 1) == d"2025-02-28"` and
  `add_years(d"2024-02-29", 4) == d"2028-02-29"`.

<!-- rule: validation.calendar.integer-count class=hard enforced=yes tasks=validation-rules title="The `add_months` or `add_years` count is integer-typed; a floating-point Number is a static error" -->
The count `n` MUST be internally typed as an integer, as for the day
arithmetic of section 11; a floating-point Number is a static error even when
its value is integral. <!-- rule: validation.calendar.no-compose class=judgment enforced=no tasks=validation-rules --> Because of the day clamp, shifts do not compose:
`add_months(add_months(d"2024-01-31", 1), 1)` is `d"2024-03-29"`, while
`add_months(d"2024-01-31", 2)` is `d"2024-03-31"`. A shift by days remains the
`+` and `-` arithmetic of section 11.

<!-- rule: validation.calendar.null-range class=hard enforced=no tasks=validation-rules -->
If any argument is `null`, the result is `null`. A shift whose result falls
outside the DateOnly range of years 1000 through 9999 also evaluates to `null`,
as does day arithmetic beyond that range. <!-- rule: validation.calendar.pure class=hard enforced=no tasks=validation-rules --> The functions are pure (section 17):
they read only their arguments, and the language has no current-date value.

<!-- rule: validation.calendar.month-difference class=judgment enforced=no tasks=validation-rules -->
The language has no month-difference function. A whole-month difference has
competing conventions at month ends; compare a shifted date instead. For
example, "at least `n` months from `a` to `b`" is `add_months([a], n) <= [b]`,
and the last day of a period of `n` months beginning on `[start]` is
`add_months([start], n) - 1`.

---

# 12. Collection Operations

<!-- rule: validation.collection.array-sources class=hard enforced=yes tasks=validation-rules title="Traversal sources are data paths that resolve only to arrays" -->
Traversal sources in this section MUST be data paths that resolve only to
arrays. <!-- rule: validation.collection.concatenation class=hard enforced=no tasks=validation-rules --> Arrays selected through multiple parent-array elements are concatenated
in deterministic path and element order. A selected `null` or missing array
contributes no elements.

## 12.1 `all`

```c
all(node in [array_path], expression)
```

<!-- rule: validation.collection.all class=hard enforced=no tasks=validation-rules -->
Returns `true` iff the expression evaluates to boolean `true` for every array element.

For an empty array:

```text
all(...) -> true
```

Example:

```c
all(item in [items],
    [item/price] >= 0
)
```

---

## 12.2 `any`

```c
any(node in [array_path], expression)
```

<!-- rule: validation.collection.any class=hard enforced=no tasks=validation-rules -->
Returns `true` iff the expression evaluates to boolean `true` for at least one array element.

For an empty array:

```text
any(...) -> false
```

---

## 12.3 `none`

```c
none(node in [array_path], expression)
```

<!-- rule: validation.collection.none class=hard enforced=no tasks=validation-rules -->
Returns `true` iff the expression evaluates to `false` for every array element.

For an empty array:

```text
none(...) -> true
```

---

## 12.4 `collect`

```c
collect(node in [array_path], predicate, projection_expression)
```

<!-- rule: validation.collect.predicate-first class=hard enforced=no tasks=validation-rules -->
`collect` returns an ordered collection. Its source remains a current-data
path selecting arrays, including arrays reached through parent-array fan-out.
For each concrete cursor, it evaluates `predicate` first. Only Boolean `true`
causes the projection to be evaluated, once, in that cursor's scope. False or
null predicates contribute nothing and do not read the projection.

The projection accepts any expression in this language:

<!-- rule: validation.collect.direct-path class=hard enforced=no tasks=validation-rules -->
- A direct square data path retains path-selection behavior: append every
  selected node in path order, flattening arrays crossed on the path. A
  missing field selects no node; a selected null field contributes one null.
  A path ending at a List contributes that List as one value, without
  flattening its elements. <!-- rule: validation.collect.computed-collection class=hard enforced=no tasks=validation-rules -->
- Another expression producing a collection, including nested `collect`,
  concatenates its items in order at this one boundary. An empty collection
  contributes no item. <!-- rule: validation.collect.computed-value class=hard enforced=no tasks=validation-rules -->
- Another expression producing a scalar, null, object, or List contributes
  exactly one value, retaining its semantic type and identity. A computed
  null is one item, even when it came from a missing path. Objects and Lists
  are not recursively flattened or coerced.

Thus `collect(row in [rows], true, [row/maybe])` skips missing `maybe` fields,
whereas `collect(row in [rows], true, [row/maybe] | null)` includes a null item
for each missing field. Existing null and arithmetic rules still apply. <!-- rule: validation.collect.order-bag class=hard enforced=no tasks=validation-rules -->
The result preserves source order, projection order, and duplicates; it is a
collection (bag), not a mathematical set. Empty sources and predicates that
never evaluate to true produce empty collections.

Filtering before projection permits Union-specific paths:

```c
collect(node in [nodes],
    [node] is Active,
    [node/device]
)
```

Only an `Active` concrete Union variant is projected. Other variants need no
`device` field and contribute nothing. <!-- rule: validation.collect.type-guards class=hard enforced=no tasks=validation-rules --> Studio narrows projection validation
using positive type tests on the cursor or a nested cursor path. The tests may
be conjuncts of `&&`; unrelated conjuncts do not remove their guarantees.
For example, `[row/document] is Payment && [row/enabled]` permits numeric
projection paths through the selected `Payment` payload even when another
variant declares an object field with the same name. <!-- rule: validation.collect.guard-specificity class=hard enforced=yes tasks=validation-rules title="A deeper path uses the most specific compatible guard; repeated guards intersect and contradictory guards are rejected" --> A deeper path uses the
most specific compatible guard, and repeated guards on one path intersect.
Contradictory guards with no compatible declared branch are rejected.

<!-- rule: validation.collect.guard-scope class=hard enforced=no tasks=validation-rules -->
These facts apply only to that `collect` projection and its nested scopes;
they do not change the schema or escape to later expressions. `||`, negation
and fallback do not establish positive type facts. A guard on another cursor
or path cannot narrow this one, and array fan-out remains invalid as an `is`
operand. Null and absent payloads retain their existing evaluation behavior.

Computed numeric projections can be summed without storing derived totals:

```c
sum(collect(row in [rows], [row/enabled],
    ([row/amount] | 0) * [row/count])) <= [maximum] : "TOTAL_ABOVE_MAXIMUM";
```

Nested projections can filter by an outer cursor before aggregating:

```c
sum(collect(group in [groups], true,
    sum(collect(item in [group/items], [item/year] == [year],
        [item/amount])))) <= [maximum] : "YEAR_TOTAL_ABOVE_MAXIMUM";
```

<!-- rule: validation.cursor.lexical class=hard enforced=yes tasks=validation-rules title="Cursors are lexical: inner traversals may read outer cursors but cannot reuse a name in scope or leak it" -->
Cursors remain lexical node bindings. Inner traversals may read outer cursors;
they cannot reuse a name already in scope or leak a name outside their
predicate/projection. Nested calls and quoted commas use the ordinary
expression grammar and [string escape rules](#escape-and-quote-contract); this feature adds
no quoting or escape layer. Predicate, Boolean, and first-non-null
short-circuit behavior, validation-phase behavior, and null behavior remain
unchanged.

<!-- rule: validation.collect.client-support class=judgment enforced=no tasks=validation-rules -->
Computed projections require clients implementing this contract. Older clients
support only direct data-path projections and must be updated before an App
using computed projections is distributed to them. Schema acceptance alone
does not update an installed client.

## 12.5 `is_unique_set`

```c
is_unique_set(collection)
```

<!-- rule: validation.unique.semantics class=hard enforced=no tasks=validation-rules -->
Returns `true` exactly when no two collection elements compare equal under the
ValidationRules `==` semantics. Empty and single-element collections are
unique. Two `null` elements are duplicates.

<!-- rule: validation.unique.fanout-collection class=hard enforced=no tasks=validation-rules -->
A fan-out path argument such as `[rows/key]` is always the collection of the
nodes it selects, whether that is zero, one or many, including zero when a List
on the path is `null`; it is never reduced to `null` or a single value for this
function.

<!-- rule: validation.unique.not-all class=judgment enforced=no tasks=validation-rules -->
`all` is not overloaded for this purpose. Its predicate is evaluated
independently for one bound cursor and returns only a Boolean; it has neither
collection state nor another-element binding with which to detect duplicates.

The canonical Union-filtered uniqueness rule is:

```c
is_unique_set(
    collect(node in [nodes],
        [node] is Active,
        [node/device]
    )
) : "ACTIVE_DEVICE_MUST_BE_UNIQUE";
```

A key of several values uses the cursor form:

```c
is_unique_set(cursor in [array_path], key_1, key_2, ...)
```

<!-- rule: validation.unique.key-tuple class=hard enforced=no tasks=validation-rules -->
The cursor binds to each element of the source arrays exactly as for `all`
(section 6.2), and the key expressions are evaluated for that element in
order. The element's key is the tuple of their values. The result is `true`
exactly when no two elements have keys whose corresponding components all
compare equal under `==`. A `null` component therefore equals only `null`:
two elements that agree on every other component and are both `null` in one
component are duplicates, while a `null` and a non-null component differ. <!-- rule: validation.unique.empty class=hard enforced=no tasks=validation-rules --> An
empty source, a single element, and a missing or `null` source array are
unique (section 12.7). <!-- rule: validation.unique.key-required class=hard enforced=yes tasks=validation-rules title="The cursor form of `is_unique_set` needs at least one key" --> At least one key is required. <!-- rule: validation.unique.key-single-value class=hard enforced=yes tasks=validation-rules title="Each `is_unique_set` key produces one value per element, not a collection" --> Each key MUST produce one
value per element, not a collection; it is typically a path from the cursor
and MAY be any other expression, such as `year([row/date])`.

```c
is_unique_set(row in [carrybacks], [row/credit], [row/year]) : "CARRYBACK_DUPLICATE";
```

<!-- rule: validation.unique.bound-keys class=judgment enforced=no tasks=validation-rules -->
The keys are bound to their element rather than written as separate paths:
independently resolved paths form separate collections, and a field absent
from one element's active Union payload would shift the later values, pairing
components that belong to different elements.

## 12.6 `count`

```c
count([array])
```

<!-- rule: validation.count.semantics class=hard enforced=no tasks=validation-rules -->
returns the number of elements in an array or a `collect` result.

Example:

```c
count([items]) >= 1
```

<!-- rule: validation.count.fanout class=hard enforced=no tasks=validation-rules -->
A fan-out path argument such as `[rows/key]` is always the collection of the nodes it selects, so it counts 0, 1 or more (0 when a List on the path is `null`); otherwise, if its argument resolves to `null`, `count()` evaluates to `null`.

---

## 12.7 Missing or Null Traversal Arrays

<!-- rule: validation.collection.missing-array class=hard enforced=no tasks=validation-rules -->
A missing or `null` terminal array contributes zero elements to a traversal.
Consequently, the usual empty-collection identities apply: `all` and `none`
return `true`, `any` returns `false`, and `collect` returns an empty collection.
Use an explicit path null test when array presence is itself required.

## 12.8 `sum`

```c
sum(collection)
```


[returns the sum of the Numbers its argument selects](#128-sum). The argument is a
current-data path that fans out across one or more arrays, such as
`[rows/amount]`, a `collect` result, or a path to an array of Numbers. <!-- rule: validation.sum.numeric-leaves class=hard enforced=yes tasks=validation-rules title="Every leaf `sum` selects or computes is an ordinary integer or floating-point Number, including Money" --> Every
leaf it can select or compute MUST be an ordinary integer or floating-point Number,
including Money. A DateOnly, DateTime, String, Boolean, object, or array leaf,
a path that selects one scalar without fan-out, and a direct reference path are static
errors (section 16).  [Studio retains the semantic item type](#128-sum) of computed
collections, even when an item has no schema node: numeric expressions and
date differences can be summed; dates shifted by integers remain dates and
cannot. An all-null computed projection sums to integer zero. An unresolved
reference type does not establish ordinary numeric leaves.

<!-- rule: validation.sum.null-leaves class=hard enforced=no tasks=validation-rules -->
`null` leaves are skipped. When nothing is selected, including an empty or
missing array or a source whose leaves are all `null`, the result is the
integer `0`; a single selected value is its own sum. The sum is therefore a
Number for every data instance, and a rule such as
`[principal] <= sum([contributions/amount])` needs no null guard for the
collection. <!-- rule: validation.sum.exact class=hard enforced=no tasks=validation-rules --> Addition is the exact arithmetic of section 11, so the result is
independent of order; it is `null` only when it exceeds that section's exact
numeric limit. The result is an integer when every summed leaf is
integer-typed, and a Number otherwise.

A `collect` predicate sums a filtered subset, here the deposits whose Boolean
`later` field is true:

```c
sum(collect(deposit in [deposits], [deposit/later], [deposit/amount])) <=
sum([deposits/netAmount]) : "LATER_DEPOSITS_ABOVE_NET";
```

---

# 13. String Operations

The core specification defines:

```c
length(string)
```

<!-- rule: validation.string.length-bytes class=hard enforced=no tasks=validation-rules -->
which returns the number of bytes in the string's UTF-8 encoding. Every platform
must use this same byte count, matching
[String `SizeMin`/`SizeMax`](schema.md#43-value-constraint-and-presentation-attributes).
The text is measured as supplied, without Unicode normalization. For example,
`length("A")` is 1, `length("é")` is 2, `length("中")` is 3,
`length("😀")` is 4, and `length("e\u0301")` is 3.
Native-character constraints (`MinLengthAbs`/`MaxLengthAbs`) remain a separate
Unicode-scalar constraint.

<!-- rule: validation.string.no-character-count class=hard enforced=no tasks=validation-rules -->
`length` counts bytes, not characters, by design. A byte bound is the same on
every platform and matches what storage, wire messages and native buffers
actually hold, so a value accepted on one client can never exceed a limit on
another. A character count differs by platform and Unicode version. The language
has no character-counting function today. If a rule ever needs one, it will be a
separate, explicitly named function; `length` and String Size will keep counting
UTF-8 bytes.

Example:

```c
[name] != null &&
length([name]) > 0
```

<!-- rule: validation.string.length-null class=hard enforced=no tasks=validation-rules -->
If the input is `null`:

```text
length(null) -> null
```

The full-string `matches` predicate is defined in §13.2 below.

## 13.1 String resource references

```c
resource([status])
```

<!-- rule: validation.resource.key class=hard enforced=no tasks=validation-rules -->
takes a FormattedText value and returns its string resource **key** as a
String, or null when the value is null. The key identifies the resource
template, not its localized text, so comparing it gives the same result in
every locale. Arguments do not change the key. <!-- rule: validation.resource.formattedtext-argument class=hard enforced=yes tasks=validation-rules title="`resource()` requires a FormattedText argument" --> Studio rejects `resource(...)`
on a value that is not FormattedText, <!-- rule: validation.resource.declared-key class=hard enforced=yes tasks=validation-rules title="A string literal compared with a `resource()` result is a key the package StringResources declares" --> and a string literal compared with a
`resource(...)` result that is not a key the package's `StringResources`
declares.

```c
resource([status]) == "STATUS_NOT_STARTED" || resource([status]) == "STATUS_NONE"
```

---


## 13.2 Full-string patterns

<!-- rule: validation.matches.full-string class=hard enforced=no tasks=validation-rules -->
`matches(value, pattern)` tests the entire String against the portable regex
language below. It returns Boolean for two Strings, without trimming,
normalization, substring search, flags, captures, replacement or coercion.
An empty pattern matches only empty text.

<!-- rule: validation.matches.arguments class=hard enforced=yes tasks=validation-rules title="`matches()` takes exactly two String or null arguments; Number, date, Boolean, collection and FormattedText arguments are rejected" -->
Exactly two arguments, each a String expression or null; current/reference
paths and source constants use existing expression contexts. Studio rejects
Number, dates, Boolean, collections and FormattedText arguments. There is no
coercion, translation lookup or general truthiness.

<!-- rule: validation.matches.null class=hard enforced=no tasks=validation-rules -->
With two Strings, return Boolean. Empty text matches only a pattern accepting
empty text; an empty pattern accepts only empty text. A null argument returns
null under existing missing-data/deferral rules. Optional fields can use
`[] == null || matches([], "[0-9]+")`. Complete final rules still require true.

<!-- rule: validation.matches.static-compile class=hard enforced=no tasks=validation-rules -->
Studio compiles every statically known pattern, even in an unreachable branch.
Malformed syntax, unsupported constructs and compilation limits produce a
schema error at the narrowest source location.

<!-- rule: validation.matches.ordered-evaluation class=hard enforced=no tasks=client-runtime -->
Runtime evaluation is ordered: evaluate arguments left-to-right and propagate
argument errors; reject non-null wrong types or ill-formed scalar Strings;
return null if either remaining argument is null; otherwise compile and match.
An invalid dynamic pattern with a null value therefore returns null before
compilation, while a statically known bad pattern is always rejected by Studio.
With two non-null Strings, compilation precedes any empty-text shortcut. <!-- rule: validation.matches.evaluation-errors class=hard enforced=no tasks=validation-rules -->
A dynamic pattern compiles when evaluated. Invalid patterns, wrong runtime
types and compilation failures are **evaluation errors**, not false or null.
An evaluated error makes the enclosing rule fail, including under `!`, `|`
or `||`. Short circuiting still avoids evaluating an unused branch. Thus
`!matches(value, badPattern)` cannot pass. This explicit error distinction
must be preserved through the evaluator; the existing missing-value sentinel
alone is insufficient. No expression-level exception-catching API is added.

<!-- rule: validation.matches.text-free class=hard enforced=yes tasks=validation-rules title="Text-free presentation conditions reject `matches()`" -->
Text-free presentation conditions reject matches.  [Its Boolean result](#31-dynamic-numeric-ranges) cannot
serve as RangeMin/RangeMax. Business validation never enters Display Template.

### Pattern attribute and lifecycle

<!-- rule: validation.pattern.placement class=hard enforced=yes tasks=validation-rules title="Pattern goes once on an effective String node, never on packages, functions, containers, Enumerations, numbers, dates, Bytes or FormattedText" -->
Pattern has one String value per physical node. It is valid on an effective
String field, String list element or named type resolving to String, including
nullable String. It is invalid on packages, functions, Endpoints, containers,
Enumerations, Number/date nodes and Bytes/FormattedText.  [Annotate a String](#pattern-attribute-and-lifecycle)
payload rather than its containing Union variant. It does not recurse.

<!-- rule: validation.pattern.effective.authoring class=hard enforced=no tasks=validation-rules -->
Declare a field-local Pattern to replace its type Pattern; [ValidationRules remain cumulative](#pattern-attribute-and-lifecycle).

<!-- /rule -->

<!-- rule: validation.pattern.effective class=hard enforced=no tasks=client-runtime -->
Use the existing effective-attribute resolver: the field's explicit Pattern
wins over its referenced type's Pattern. Existing cumulative ValidationRules
remain cumulative. <!-- rule: validation.pattern.studio-checks class=hard enforced=yes tasks=validation-rules title="Studio validates Pattern placement, multiplicity and grammar, and that non-null Default and Fixed values match" --> Studio validates placement, multiplicity, regex grammar
and materialized effective Default/Fixed values. Non-null defaults/fixed
values must match; allowed null remains governed by Nullable/mandatory rules.

<!-- rule: validation.pattern.client-lifecycle class=hard enforced=no tasks=client-runtime -->
Clients run Pattern through ordinary intrinsic value validation alongside
Size, on committed edits and final submission. Temporary incomplete typing
may remain editable but cannot be submitted. Import, copy, calculation,
restore, Hidden and ReadOnly do not waive validation or erase bad values.
Null adds no pattern mismatch; ordinary requiredness still applies.

<!-- rule: validation.pattern.message class=hard enforced=no tasks=validation-rules -->
A mismatch uses a standard localized message, such as “Enter a value in the
required format.” The field description explains that format. There is no
second message-key attribute; use matches in ValidationRules for a custom
message. Invalid patterns are configuration/rule errors, not typing errors.

Pattern stays semantic schema and affects the ordinary revision digest.
String wire/storage encoding is unchanged. The Hub preserves the attribute;
there is no new message field, editor, history or persistent regex cache. <!-- rule: validation.pattern.server-validates class=judgment enforced=no tasks=validation-rules -->
Server Apps still validate untrusted requests independently.

### Portable regex language

<!-- rule: validation.regex.scalars class=hard enforced=no tasks=validation-rules -->
One regular-language grammar over Unicode scalars, case sensitive and locale
independent. No normalization: composed and decomposed accents differ.

| Construct | Meaning |
| --- | --- |
|  [Literal scalar](#portable-regex-language) | That scalar, unless syntax punctuation below |
| Concatenation; `A\|B` | Concatenation; union of alternatives, including empty alternatives |
| `(A)`, `(?:A)` | Grouping; no exposed captures or references |
| `[abc]`, `[^abc]`, `[a-z]` | Set, scalar complement, inclusive scalar range |
| `?`, `*`, `+` | Zero/one, zero/more, one/more of the preceding atom/group |
| `{m}`, `{m,}`, `{m,n}` | Nonnegative decimal repetition counts; m<=n |
| `.` | One scalar except LF, CR, U+2028, U+2029 |
| `^`, `$` | Absolute start/end; no “before final newline” meaning |
| `\d`, `\D` | ASCII [0-9], its scalar complement |
| `\w`, `\W` | ASCII [A-Za-z0-9_], its scalar complement |
| `\s`, `\S` | ASCII space, tab, LF, CR, form feed, vertical tab; complement |
| `\n \r \t \f \v` | Those control scalars |
| `\xHH \uHHHH \u{H...H}` | Exactly 2/4/1–6 hex digits; valid scalar <=10FFFF, not D800–DFFF |
| `\\ \. \^ \$ \| \? \* \+ \( \) \[ \] \{ \} \-` | Escaped literal punctuation |

The complete literal-punctuation escape set, without Markdown table escaping:

```text
\\ \. \^ \$ \| \? \* \+ \( \) \[ \] \{ \} \-
```

<!-- rule: validation.regex.classes class=hard enforced=yes tasks=validation-rules title="In a class only a leading `^` negates and `-` is literal first or last; `[` and `]` are escaped; empty, nested and reversed classes are rejected" -->
Within a class, only leading unescaped `^` negates. Unescaped `-` is literal
first/last, otherwise a range separator between single-scalar endpoints.
Literal `[` and `]` must be escaped. Other syntax operators are literals in a
class. Predefined sets can be class members, never range endpoints. Reject
empty/nested/unterminated classes and reversed ranges. `&&` inside a class
means literal ampersands, never host-engine intersection.

<!-- rule: validation.regex.punctuation class=hard enforced=yes tasks=validation-rules title="Outside classes the listed punctuation is syntax; unmatched closers, misplaced quantifiers and malformed counts are invalid" -->
Outside classes, `. ^ $ \ | ? * + ( ) [ ] { }` are syntax punctuation;
unmatched closing punctuation is invalid. Quotes, apostrophes, slash, hyphen,
hash, semicolon, colon, @ and spaces are ordinary regex literals. Literal
control scalars are allowed and match themselves. A quantifier requires a
complete preceding atom/group; quantifying anchors, repeated quantifiers,
signed counts, count whitespace and malformed braces are invalid.

<!-- rule: validation.regex.escapes class=hard enforced=yes tasks=validation-rules title="Unknown escapes, surrogate-pair escapes, flags, backreferences, lookaround and similar host-engine features are errors" -->
A scalar escape produces one literal atom even when that scalar is syntax
punctuation; it is not lexed again. Unpaired surrogates in a decoded pattern
or evaluated value are errors rather than partial scalar matches.
Unknown regex escapes are errors, including `\q \0 \1 \b \p \/`. Write
quotes/slash directly and use the stated control/scalar escapes.
`\uD83D\uDE00` is invalid regex scalar notation; use literal emoji or
`\u{1F600}`. No flags, embedded modes, named groups, backreferences,
lookaround, property categories, lazy/possessive quantifiers, atomic groups
or class set operations. Slash pairs are ordinary text, not removed delimiters.

<!-- rule: validation.regex.hex-escapes class=hard enforced=yes tasks=validation-rules title="Hex escapes take exactly 2 or 4 digits, or 1–6 braced digits; incomplete, out-of-range and surrogate escapes are errors" -->
Hex digits are ASCII 0–9/A–F/a–f. Fixed-width `\xHH`/`\uHHHH` consume
exactly their stated digits; following scalars begin the next atom. Braced
Unicode escapes require 1–6 digits and a closing brace, without signs or
whitespace. Incomplete, out-of-range and surrogate escapes, or a trailing
unpaired backslash, are errors. These escape rules are identical inside and
outside classes; a decoded `-`, `]` or `^` is a literal class member, never
reinterpreted as class structure.

<!-- rule: validation.regex.non-backtracking class=hard enforced=no tasks=client-runtime -->
Require non-backtracking regular-language matching, such as Thompson NFA,
including for nested/ambiguous repetition. <!-- rule: validation.regex.limits class=hard enforced=yes tasks=validation-rules title="Patterns are limited to 4096 scalars, 64 group nesting, repeat counts up to 10000 and 65536 compiled states" --> Fixed compilation limits:
4096 pattern scalars, 64 group nesting, repeat endpoints <=10000 and at most 65536
compiled states. Over-limit patterns are errors, never truncated. Match work
is bounded by input scalars times states; ordinary String/schema/runtime limits
still bound input storage. No timing-dependent accepting fallback.

Use one compiler/matcher per client for both interfaces. Native engines can
be reused only if an adapter proves the grammar, scalar/anchor behavior and
non-backtracking guarantee. Calling JS RegExp, Android Pattern and
NSRegularExpression directly does not establish those properties.


### Escape and quote contract

<!-- rule: validation.escape.decode-once class=hard enforced=no tasks=validation-rules -->
Decode each layer once. The compiler receives **regex text**, not source code
or a JSON-encoded value. Transport JSON escaping is representation only.

At each cooked-literal or quoted-annotation boundary, a quote closes only
when preceded by an **even** number of consecutive backslashes (including
zero). An odd number escapes it. Process escapes left-to-right; checking only
the immediately preceding character is insufficient. Raw Rust literals instead
end at their exact quote-plus-hashes delimiter. After native source decoding,
there is no additional HTML entity/URL decoding, interpolation, Unicode or
whitespace normalization, or second unescape.

#### Source attribute layer

<!-- rule: validation.escape.rust-pattern class=hard enforced=no tasks=validation-rules -->
For shared semantic rejection, follow the [Pattern placement and grammar requirements](#pattern-attribute-and-lifecycle).
**Rust:** one positional native String literal, cooked or raw, or `$CONST`
resolving to a native String through existing source-constant machinery.
Cooked literals use native Rust escapes; raw `r"..."`, `r#"..."#` etc do not
decode backslashes. Increase delimiter hashes when content includes the
current closing quote/hash sequence.

<!-- rule: validation.source.raw-delimiter-choice class=hard enforced=no tasks=validation-rules -->
Native Rust source preprocessing and cooked-string line continuations retain
their native meaning; they are not regex escapes. For exact LF/CR data, prefer
explicit cooked escapes or regex `\n`/`\r` in a raw literal. If the content
cannot fit a legal raw delimiter, use a cooked literal instead. Bytes and
character literals are not accepted as String arguments.

<!-- rule: validation.escape.typescript-pattern class=hard enforced=no tasks=validation-rules -->
For shared semantic rejection, follow the [Pattern placement and grammar requirements](#pattern-attribute-and-lifecycle).
**TypeScript Pattern in JSDoc:** one positional double-quoted **JSON String
literal**, or `$CONST` resolving a native String constant. Decode JSON once,
without trim. Accept exactly JSON escapes `\" \\ \/ \b \f \n \r \t \uHHHH`;
surrogate pairs form one scalar. <!-- rule: validation.escape.unpaired-surrogate class=hard enforced=yes tasks=validation-rules title="An unpaired surrogate in decoded regex text is a Studio semantic error" --> An unpaired surrogate in decoded regex text
is a shared Studio semantic error; native syntax decoding does not replace it. Reject
single/backtick quotes, unknown escapes, raw controls, incomplete escapes,
extra/named arguments and trailing non-whitespace before `)`. Outside
whitespace is permitted; inside spaces are significant.

<!-- rule: validation.escape.json-whitespace class=hard enforced=no tasks=validation-rules -->
JSON whitespace here means only space, tab, LF and CR. Raw U+0000–U+001F
inside the JSON literal is invalid; use an escape. Literal Unicode scalars
above that range, including U+2028/U+2029, are JSON content, not delimiters.
Native source/comment syntax must also permit the spelling. No implicit
line continuation or JSDoc line-decoration removal occurs inside a literal;
use escaped controls or a constant for multiline patterns.

This is the grammar for the Pattern JSDoc annotation, not an assertion that
JSDoc is a JavaScript expression. Native constants decode in their native language
before resolution. <!-- rule: validation.escape.const-argument class=hard enforced=no tasks=validation-rules -->
For shared semantic rejection, follow the [Pattern placement and grammar requirements](#pattern-attribute-and-lifecycle). `$CONST` occupies the whole argument; no interpolation
occurs inside quoted patterns, where `$` remains a regex anchor.

<!-- rule: validation.escape.jsdoc-delimiters class=hard enforced=no tasks=validation-rules -->
Scan delimiters while respecting quoted strings and escapes: a regex `)`,
`@` or brace inside a String cannot terminate the annotation. JSDoc `*/` still
ends its comment; represent a literal star/slash sequence using JSON `*\/`
or a native constant. HTML entities have no decoding role. JSON `\b` is U+0008,
not a regex boundary; JSON has no `\u{...}` escape, so double that backslash.

<!-- rule: validation.escape.annotation-discovery class=hard enforced=no tasks=client-runtime -->
This applies to annotation discovery as well as argument extraction:
`@LibertasPattern("@LibertasHidden()")` declares only Pattern. Its contents
must never create Hidden or another annotation. A marker is recognized only
outside an already recognized annotation's quoted argument. Separate adjacent
annotations resume normal scanning after the closing `)`.

#### ValidationRules literal layer

<!-- rule: validation.literal.rule-escapes class=hard enforced=no tasks=validation-rules -->
The existing rule lexer permits single and double quotes; the matching
unescaped quote closes. A backslash followed by n/r/t emits LF/CR/tab; followed
by **any other character**, it emits that character and drops the backslash.
Thus `\\` emits one backslash, `\"` quote and `\'` apostrophe. Missing closing
quote or dangling escape is a syntax error. These identity escapes remain
part of the rule literal grammar.

<!-- rule: validation.source.enclosing-string-valid class=hard enforced=no tasks=validation-rules -->
In particular, rule `\b \f \v \0` decode to ordinary b/f/v/0, and a
backslash followed by an actual line break preserves that line break rather
than joining lines. Raw controls are accepted by the existing rule lexer;
their enclosing source representation must still be valid. To request regex
form-feed/vertical-tab escapes, transmit `\\f`/`\\v`. For NUL or backspace,
transmit regex `\\x00`/`\\x08` or an actual scalar; regex `\0`/`\b` are
unsupported regardless of their meaning in a host regex engine.

<!-- rule: validation.literal.double-backslash class=hard enforced=no tasks=validation-rules -->
Double every backslash intended for regex: rule `"\\d+"` transmits regex
`\d+`, whereas `"\d+"` transmits `d+`. Similarly `"\\q"` transmits an invalid
regex escape, while `"\q"` transmits ordinary q. Rule `\u`/`\x` are identity
escapes: write `"\\u{1F600}"` to transmit that regex Unicode escape.

<!-- rule: validation.literal.outer-envelope class=hard enforced=no tasks=validation-rules -->
An outer Rust ValidationRules literal decodes first; prefer raw Rust.
Existing TypeScript LibertasValidationRules JSDoc preserves raw quoted
annotation content, unlike the new JSON-decoded Pattern. Use a single-quoted
outer envelope and double-quoted rule strings:

```rust
#[libertas_validation_rules(r#"matches([], "\\d+") : "DIGITS";"#)]
pub digits: String,
```

```typescript
/** @LibertasValidationRules('matches([], "\\d+") : "DIGITS";') */
digits: string;
```

<!-- rule: validation.source.ts-annotation-escaping class=hard enforced=no tasks=typescript-app -->
For the old TypeScript annotation, an apostrophe inside a double-quoted rule
String must also survive the single-quoted annotation envelope. For example:

```typescript
/** @LibertasValidationRules('matches([], "[\"\']+") : "QUOTES";') */
```

<!-- rule: validation.source.annotation-versus-program-string class=hard enforced=no tasks=typescript-app -->
The outer annotation preserves both backslash/quote sequences; the rule lexer
then decodes them to regex `["']+`. Ordinary TypeScript program constants
instead use native TypeScript string decoding. Do not confuse a JSDoc
annotation envelope with a program String literal.

<!-- rule: validation.source.no-extra-unescape class=hard enforced=no tasks=client-runtime -->
The earlier Rust and TypeScript digit examples transmit the same regex.
Do not add another source-unescape pass to the
existing TypeScript ValidationRules annotation as an incidental change.

<!-- rule: validation.literal.mechanical-encoder class=hard enforced=no tasks=validation-rules -->
For a mechanical rule-literal encoder, choose double quotes and escape every
backslash as `\\`, both quote kinds as `\"` and `\'`, and LF/CR/tab as
`\n`/`\r`/`\t`. Other scalars remain unchanged. Escaping the apostrophe
is redundant for the rule's double-quote delimiter but makes the same text
safe inside the legacy TypeScript single-quoted envelope. Preserve that rule
text verbatim in the envelope; do not apply JSON or JavaScript string encoding
to it. JSDoc's physical `*/` restriction still applies to the whole comment,
including the legacy envelope. A regex slash can instead be spelled `\x2F`
(double its backslash in the rule). Native constants use their existing
annotation-specific resolution; they do not acquire interpolation semantics.

### Regex layer and exact-value examples

<!-- rule: validation.regex.no-redecode class=hard enforced=no tasks=client-runtime -->
The regex compiler never re-decodes JSON, source strings, rule literals,
constants or HTML. A String obtained from a path already contains decoded text.

| Regex text intended | Rust Pattern argument | TypeScript Pattern argument | Rule pattern literal |
| --- | --- | --- | --- |
| `\d+` | `r"\d+"` or `"\\d+"` | `"\\d+"` | `"\\d+"` |
| `\\`, matches one backslash | `r"\\"` or `"\\\\"` | `"\\\\"` | `"\\\\"` |
| `\.`, matches dot | `r"\."` | `"\\."` | `"\\."` |
| `["']+`, matches quotes | `r#"["']+"#` | `"[\"']+"` | `"[\"']+"` |
| `\u{1F600}` | `r"\u{1F600}"` | `"\\u{1F600}"` | `"\\u{1F600}"` |
| `\n`, matches LF | `r"\n"` | `"\\n"` | `"\\n"` |
| Actual LF scalar | `"\n"` | `"\n"` | `"\n"` |
| Space-a-space | `r" a "` | `" a "` | `" a "` |

<!-- rule: validation.serialization.decode-once class=hard enforced=no tasks=client-runtime -->
Regex `\d+` serializes as `{"Name":"Pattern","Value":"\\d+"}`; the decoded
Value contains one backslash, d, plus. Regex matching a literal backslash
serializes as `{"Name":"Pattern","Value":"\\\\"}`; decoded Value contains two
backslashes. Do not decode either serialized spelling again.

The two Unicode spellings distinguish the layers: TypeScript Pattern
`"\uD83D\uDE00"` decodes to literal emoji and is valid; Pattern
`"\\uD83D\\uDE00"` decodes to two regex surrogate escapes and is invalid.
JSON `"\b"` decodes to a literal backspace scalar and is valid regex text;
JSON `"\\b"` decodes to the unsupported regex escape and fails. Likewise,
JSON `"\/"` supplies literal slash; JSON `"\\/"` supplies invalid regex `\/`.
No syntax error is repaired or downgraded to a mismatch.

A decoded pattern consisting of one final backslash is malformed regex even
when its source literal is valid. To match one backslash, supply two decoded
backslashes, as above. Escaping a quote at a source boundary does not make it
a regex escape: decoded quotes are ordinary scalars. Unmatched `)`/`{`/`}`
can safely survive source delimiters but still need regex escaping or class
placement to be literal regex content.

<!-- rule: validation.regex.quotes class=hard enforced=no tasks=validation-rules -->
Quotes/apostrophes are not regex metacharacters. Escape them only for their
source/rule delimiter or choose a raw delimiter. A backslash before a closing
quote is interpreted by the enclosing literal first; literal final backslash
requires the complete doubling above. Regex `; : @ ) { }` inside a quoted
argument cannot end a call, rule or annotation.



# 14. Expression Grammar

<!-- rule: validation.grammar.ebnf class=hard enforced=yes tasks=validation-rules title="Rule sets follow the normative grammar; `length`, `resource`, `year`, `month` and `day` take one argument, `add_months`, `add_years` and `matches` two" -->
The normative grammar is:

The `current_data_schema_path` and `reference_object_schema_path` productions
are defined once in [Libertas Schema Path](schema.md#current-data-references-in-validation-expressions).

```ebnf
rule_set
    = [ "FinalOnly" "=" boolean_literal ";" ]
      rule_pair
      { rule_pair } ;

rule_pair
    = expression ":" string ";" ;

boolean_literal
    = "true" | "false" ;

expression
    = logical_or ;

logical_or
    = logical_and
      { "||" logical_and } ;

logical_and
    = equality
      { "&&" equality } ;

equality
    = relational
      { ("==" | "!=") relational } ;

relational
    = type_test
      { ("<" | "<=" | ">" | ">=") type_test } ;

type_test
    = fallback
      [ "is" type_name | "has" bitflag_member ] ;

fallback
    = additive
      { "|" additive } ;

additive
    = multiplicative
      { ("+" | "-") multiplicative } ;

multiplicative
    = unary
      { ("*" | "/" | "%") unary } ;

unary
    = ("!" | "+" | "-") unary
    | primary ;

primary
    = literal
    | path
    | reference_path
    | array_operation
    | function
    | "(" expression ")" ;

path
    = current_data_schema_path ;

reference_path
    = reference_object_schema_path ;

array_operation
    = "all"  "(" identifier "in" path "," expression ")"
    | "any"  "(" identifier "in" path "," expression ")"
    | "none" "(" identifier "in" path "," expression ")"
    | "collect" "(" identifier "in" path "," expression "," expression ")"
    | "count" "(" expression ")"
    | "sum" "(" expression ")"
    | "is_unique_set" "(" expression ")"
    | "is_unique_set" "(" identifier "in" path "," expression
      { "," expression } ")" ;

function
    = "length" "(" expression ")"
    | "resource" "(" expression ")"
    | ( "year" | "month" | "day" ) "(" expression ")"
    | ( "add_months" | "add_years" | "matches" ) "(" expression "," expression ")" ;

literal
    = integer
    | floating_point
    | string
    | date_literal
    | "true"
    | "false"
    | "null" ;

date_literal
    = "d" '"' digit digit digit digit "-" digit digit "-" digit digit '"' ;

type_name  (* a Union variant or type name, or an Enumeration member name *)
    = identifier ;

bitflag_member
    = identifier "." identifier ;
```

## 14.1 DateOnly literals


[`d"YYYY-MM-DD"`](#141-dateonly-literals) writes a constant calendar date: the letter `d` immediately
followed by a double-quoted ISO 8601 calendar date with a four-digit year and
two-digit month and day. The token contains no whitespace or escape. The `d`
prefix keeps it one token: a bare `2026-01-01` is the subtraction
`2026 - 1 - 1`, and a plain string literal remains a String.


[The literal has semantic type DateOnly](#16-static-rule-validation) and the value of a DateOnly path whose
wire value is `YYYYMMDD`. Equality, ordering, the date arithmetic of section 11,
`|`, and a DateOnly `RangeMin` or `RangeMax` bound treat the two alike; for
example, `d"2024-02-28" + 1 == d"2024-02-29"` and
`d"2024-03-01" - d"2024-02-28" == 2`. Mixing it with a Number or a DateTime is
invalid exactly as for a DateOnly path, so `d"2026-01-01" == 20260101` is
rejected. <!-- rule: validation.date-literal.valid-date class=hard enforced=yes tasks=validation-rules title="A DateOnly literal names a valid Gregorian date with a year from 1000 through 9999" --> The date MUST be a valid Gregorian calendar date with a year from
1000 through 9999, the DateOnly wire range; `d"2026-02-30"` and
`d"2026-13-01"` are static errors (section 16). <!-- rule: validation.date-literal.constant class=hard enforced=no tasks=validation-rules --> A literal is a constant; the
language has no current-date value (section 17).

<!-- rule: validation.date-literal.no-datetime class=hard enforced=no tasks=validation-rules -->
There is no DateTime literal. A written date and time names a different
instant in each time zone, and rules have no time-zone input; instants come
from DateTime paths and DateTime arithmetic.

## 14.2 Bitflags membership

`has` tests whether one flag of a
[Bitflags](schema.md#43-value-constraint-and-presentation-attributes) host is
set:

```c
[facts] has PersonFacts.ReleasedToYou
```

<!-- rule: validation.bitflags.host-operand class=hard enforced=yes tasks=validation-rules title="The left operand of `has` is a current-data path to a Bitflags host, without array fan-out" -->
The left operand is a current-data path to a Bitflags host, without array
fan-out. <!-- rule: validation.bitflags.member-operand class=hard enforced=yes tasks=validation-rules title="The right operand of `has` is one word `Catalog.Member`: the host's own catalog type name and one of its nonzero members as declared" --> The right operand `Catalog.Member` is written as one word, with no
whitespace around the `.`. `Catalog` is the type name of the catalog that the
host's `Bitflags` attribute names, without the `developer-package::` prefix of
an external catalog, and `Member` is the published name of one of its nonzero
members, exactly as the catalog declares it: a Rust `bitflags` constant keeps
its upper-snake name (`PersonFacts.RELEASED_TO_YOU`), and the examples here use
the neutral spelling of their illustrative catalogs. The
qualifier is required rather than inferred from the host, so the rule reads on
its own and a rule naming a catalog the host does not use is rejected instead of
testing a same-named member of another catalog.

<!-- rule: validation.bitflags.semantics class=hard enforced=no tasks=validation-rules -->
For a non-null host value `h` and the member's `Fixed` value `m`, the result is
the Boolean `(h & m) != 0`: true when that bit is set, whatever other bits are
set. `m` is a schema constant, so evaluation stays pure (section 17). A `null`
host yields `null`, which is `false` in Boolean context (section 7.1); the
integer itself is never converted to a Boolean. Bitflags hosts are
non-negative: a negative host value is outside the catalog and the test yields
`null`, as for a null host. Where absence matters, combine
the test with an explicit null test or a fallback:

```c
[facts] != null && !([facts] has PersonFacts.Student)
([facts] has PersonFacts.ReleasedToYou) | false
```

<!-- rule: validation.bitflags.one-member class=hard enforced=yes tasks=validation-rules title="`has` takes one nonzero member; test several flags with `&&` or `||` and the empty selection with `== 0`" -->
`has` takes one member. Test several flags with `&&` or `||`, and the empty
selection with `[facts] == 0`; the zero member, such as `None`, is not a flag
and is rejected.  Like `is`, [one operand takes at most one `has`](#14-expression-grammar), and negation
needs parentheses: `!([facts] has PersonFacts.Student)`.

---

# 15. Operator Precedence

<!-- rule: validation.precedence.table class=hard enforced=no tasks=validation-rules -->
From highest precedence to lowest:

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

<!-- rule: validation.precedence.has-fallback class=hard enforced=no tasks=validation-rules -->
`has` shares the `is` level: both test one value against a schema name and
produce a Boolean, so arithmetic and `|` bind first, while comparisons,
equality, and logical operators take the result. A fallback for a `has` result
therefore needs parentheses, as in `([facts] has PersonFacts.Student) | false`.

<!-- rule: validation.precedence.function-primary class=hard enforced=no tasks=validation-rules -->
A function call or collection operation is a primary expression: its
parenthesized arguments bind first, so `year([start]) + 1` adds one to the year
and `add_months([start], 24) - 1` is the day before the shifted date.

<!-- rule: validation.precedence.parentheses class=judgment enforced=no tasks=validation-rules -->
Parentheses SHOULD be used when they improve clarity.

Example:

```c
([item] is A && [item/x] > 0) ||
([item] is B && [item/y] != null)
```

---

# 16. Static Rule Validation

Before accepting a schema, Libertas Studio MUST statically verify that:

<!-- rule: validation.static.compatible-operators class=hard enforced=yes tasks=validation-rules title="Studio verifies that operators other than the first-non-null operator apply to compatible types" -->
1. operators other than `|` are applied to compatible types;
2. `|` preserves possible operand types and admits its right operand as a
   result only when its left operand may be null; <!-- rule: validation.static.boolean-results class=hard enforced=yes tasks=validation-rules title="Every complete rule and every `all`, `any`, `none` and `collect` predicate produces a Boolean or possibly null" -->
3. every complete rule and every predicate produces `bool` or possibly `null`;
4. arithmetic operators apply only to the ordinary numeric and date overloads
   in section 11, preserving the internal result type; <!-- rule: validation.static.is-type class=hard enforced=yes tasks=validation-rules title="An `is` test names a schema-defined type" -->
5. `is T` references a schema-defined type; <!-- rule: validation.static.cursor-use class=hard enforced=yes tasks=validation-rules title="A cursor variable is only the first segment of a data path, within its lexical traversal" -->
6. cursor variables are used only as the first segment of a data path and only
   within their lexical traversal scope;
7. every traversal source path resolves only to arrays, including when it
   selects several arrays by crossing parent arrays; <!-- rule: validation.static.resolvable-path class=hard enforced=yes tasks=validation-rules title="Every path is resolvable through at least one schema-valid structural route" -->
8. a path is potentially resolvable according to at least one schema-valid structural route;
9. every `collect` predicate is boolean and its projection is a valid expression
   in the bound cursor scope, retaining semantic item types and text provenance;
   positive cursor or nested cursor-path type tests, alone or within `&&`,
   narrow projection validation to compatible declared Union branches under
   section12.4; OR, negation and fallback do not establish such facts;
10. `is_unique_set` and `count` receive a collection or array, and the cursor
   form of `is_unique_set` has a traversal source and keys that each produce
   one value per element (section 12.5);
11. every rule's string is a declared package `StringResources` key with a
   non-empty default-locale message;
12. `FinalOnly`, when present, is the single leading directive and its value is
   exactly `true` or `false`;
13. `resource(...)` receives a FormattedText value, and a string literal
   compared with its result is a key that the package's `StringResources`
   declares (section 13.1);
14. every DateOnly literal has the form of section 14.1 and names a valid
   Gregorian calendar date in the DateOnly range;
15. the left operand of `has` is a current-data path, without array fan-out,
   that resolves only to Bitflags hosts, and its `Catalog.Member` names the
   catalog of every such host and one of that catalog's nonzero members
   (section 14.2);
16. `year`, `month`, `day`, `add_months`, and `add_years` receive a DateOnly
   first argument, and `add_months` and `add_years` an integer count
   (section 11.1); and
17. `sum` receives a fan-out path, a `collect` result, or an array whose every
   possible leaf is an ordinary Number (section 12.8).

<!-- rule: validation.static.path-roots class=hard enforced=no tasks=validation-rules -->
Square paths are validated against the attribute evaluation root. Reference
paths are syntax-checked but their value type remains unknown and nullable,
because the referenced response may use a different compatible schema.

<!-- rule: validation.static.nullable-warning class=hard enforced=no tasks=validation-rules -->
Studio warns when an applicable rule reads a nullable host or path prefix
without explicit absence handling. This includes a named-type rule at a
nullable use and a selected nullable Union payload. Under sections 1, 3 and
4.4, cumulative rules still evaluate with a present null host; descendant
reads yield null and a relational comparison then returns false (section 10). <!-- rule: validation.static.absence-not-pass class=hard enforced=no tasks=validation-rules -->
Absence never makes a rule pass automatically. For an optional record, write
`[] == null || ...` when the rule should accept its absence.

The diagnostic is a warning, not a schema error: rejecting absence may be
intentional. It reports the declaration's source file and one-based line when
known, otherwise null. It recognizes explicit null tests, short-circuit null
or type guards, and direct path fallbacks. It is conservative, not a proof of
validity or a replacement for running supplied fixtures; more complex
equivalent expressions may still warn.

<!-- rule: validation.static.union-paths class=hard enforced=no tasks=validation-rules -->
A path does not need to apply to every union alternative.

For example, given:

```c
A { int x; }
B { string y; }

A | B item;
```

both:

```c
[item/x]
[item/y]
```

are valid rule paths.

Their applicability is determined from the concrete runtime value.

<!-- rule: validation.paths.impossible-path-review class=judgment enforced=no tasks=client-runtime -->
A path that cannot exist under any value permitted by the schema SHOULD be rejected as a rule-definition error.

---

# 17. Evaluation Purity

<!-- rule: validation.purity.must class=hard enforced=no tasks=validation-rules -->
Validation expressions MUST be pure.

<!-- rule: validation.purity.forbidden-side-effects class=hard enforced=no tasks=validation-rules -->
A rule MUST NOT:

- mutate data;
- create data;
- delete data;
- perform I/O;
- access unprovided ambient state;
- read the current time from outside the supplied inputs;
- generate nondeterministic values;
- invoke arbitrary executable code.

<!-- rule: validation.purity.reference-data class=hard enforced=no tasks=validation-rules -->
Reading the supplied `referencedData` is permitted and does not access
unprovided ambient state. Evaluation does not fetch a response or consult
interaction history.

For fixed schema interpretation, `rule`, `data`, and `referencedData`:

\[
Eval(rule,data,referencedData)
\]

<!-- rule: validation.purity.deterministic-results class=hard enforced=no tasks=validation-rules -->
MUST always produce the same result.

---

# 18. Validation Result

<!-- rule: validation.result.same-language class=hard enforced=no tasks=validation-rules -->
The same expression language and evaluation semantics apply when a client
performs incremental validation during editing and when it performs final
validation. There is no separate relaxed or final expression dialect.
Incremental validation reports the rule result for the current, possibly
incomplete datum after omitting declarations whose leading directive is
`FinalOnly=true`. Final validation evaluates every declaration, including
those final-only rule sets, and combines the result with all applicable static
schema checks.

For schema \(\Sigma\), concrete or partially entered `data`, supplied
`referencedData`, and rule set:

\[
R=\{rule_1,rule_2,\dots,rule_n\}
\]

define:

\[
Result_i(data,referencedData) = Bool(Eval(rule_i,data,referencedData))
\]

For that same `referencedData`, the data is valid iff:

\[
\boxed{
Valid(data,referencedData)=
\bigwedge_{i=1}^{n}Result_i(data,referencedData)
}
\]

Consequently:

```text
every rule true  -> VALID
any rule false   -> INVALID
```

<!-- rule: validation.result.top-level-null class=hard enforced=no tasks=validation-rules -->
Because `null` becomes `false` in boolean context:

```text
top-level null -> INVALID
```

<!-- rule: validation.result.first-failure.authoring class=hard enforced=no tasks=validation-rules -->
Order rule/message pairs deliberately: [the first failing rule wins](#18-validation-result), with ordinary short-circuiting.

<!-- /rule -->

<!-- rule: validation.result.message class=hard enforced=no tasks=client-runtime -->
For `INVALID`, a Libertas client MUST display the failing rule pair's localized
`StringResources` message. <!-- rule: validation.result.first-failure class=hard enforced=no tasks=client-runtime --> Evaluate applicable cumulative rule sets in their
supplied order and pairs within each set in declaration order; stop at the
first false rule. Skip final-only sets during incremental evaluation. Reporting
the failure MUST preserve normal short-circuit behavior and MUST NOT evaluate
skipped branches, later rules, or later quantifier elements to construct an
explanation.

<!-- rule: validation.result.message-owns-explanation class=hard enforced=no tasks=client-runtime -->
The supplied message owns the explanation. Clients do not construct, rank, or
render structural failure certificates. If a trusted installed document is
unexpectedly missing the message or supplies only whitespace, report a generic
validation failure; do not expose expressions or entered values as a fallback. <!-- rule: validation.result.structural-diagnostic class=hard enforced=no tasks=client-runtime -->
A separate structural diagnostic may still reject a required field that remains
`null` at submission time.

<!-- rule: validation.result.final-valid class=hard enforced=no tasks=client-runtime -->
Let `StaticValid` include mandatory/nullability, type, range, size, uniqueness,
and every other ordinary schema constraint. These checks receive the same
`referencedData` where applicable, including dynamic bounds. Final task data
is accepted iff:

\[
\boxed{
FinalValid_\Sigma(data,referencedData)=StaticValid_\Sigma(data,referencedData)\land Valid_\Sigma(data,referencedData)
}
\]

<!-- rule: validation.result.final-failure-ui class=judgment enforced=no tasks=client-runtime -->
On final validation failure, keep the editor and its data. When the validator
identifies a failing field or a rule-owning group, show an error icon and the
explanation at that row, expand collapsed ancestors, and scroll the first
failure into view. A promoted header uses its owning row; a hidden field uses
its nearest visible owner. Do not replace the field’s normal control with an
inline editor: Boolean choices, enumerations, Unions and other pickers retain
their normal interactions. Function-wide rules without a field owner remain
message-only; do not infer field blame from an expression.

<!-- rule: validation.error-marker.occurrence-lifetime class=hard enforced=no tasks=client-runtime -->
The marker is transient client state, not encoded data. After an accepted edit,
recheck the marked field and clear its marker when its validation passes or
when that field is removed. Do not transfer the marker to a replacement List
row or Union branch. A later Submit may reveal the next failure. These local
markers do not change backend Error responses, which retain the protocol’s
message presentation.

<!-- rule: validation.incremental.every-change class=hard enforced=no tasks=client-runtime -->
A client Task Editor MUST evaluate every rule set whose effective `FinalOnly` value is `false` (including an omitted directive) after every
proposed node change. One synchronous UI action MAY be treated as one atomic
change when it necessarily performs several model mutations, such as selecting
a Union variant and materializing its fields. Node changes include scalar and
byte replacement, nulling or materialization, Union selection, and List
insertion, removal, replacement, or reordering.

<!-- rule: validation.incremental.snapshot-restore class=hard enforced=no tasks=client-runtime -->
The client MUST snapshot the complete editable root set before the change,
because a field-local change can invalidate an ancestor, named-type, or
function rule. If incremental validation fails, the client MUST restore that
snapshot before presenting the violation as a client-native error toast. The
invalid value MUST NOT remain in task data. Declarations whose effective
`FinalOnly` value is `true` are not evaluated and cannot reject an incremental change. <!-- rule: validation.incremental.no-static-checks class=hard enforced=no tasks=client-runtime --> Incremental validation
does not run mandatory/nullability or other static final checks.

<!-- rule: validation.incremental.introduced-violation class=hard enforced=no tasks=client-runtime -->
This rollback applies to the edits a user makes, on every client. An edit is
rolled back when it introduces a violation: when the first failing rule and
its owning node are the same before and after the edit, for example because a
calculation result left that rule violated, the edit is accepted, and final
validation still blocks submission. <!-- rule: validation.incremental.programmatic-assignment class=hard enforced=no tasks=client-runtime --> A
programmatic assignment is not a proposed change and is never rolled back by an
incremental rule: a calculation result, a link return, a constrained-choice
reset, and a singleton selection stay in the editor, and final validation
governs them. <!-- /rule --> A document import validates its complete candidate under
[its own rule](llm-document-transformation.md#client-workflow).

<!-- rule: validation.final.static-and-rules class=hard enforced=no tasks=client-runtime -->
The client MUST evaluate all rule sets as part of final validation; a successful
incremental or final rule result cannot override `StaticValid=false`.

<!-- rule: validation.output-only.excluded class=hard enforced=no tasks=validation-rules -->
An [OutputOnly](protocols.md#output-only-request-fields) request node is outside
client validation: the client checks neither its static constraints nor the rule
sets hosted on it or inside it, incrementally or finally. Rule sets hosted
elsewhere that read it evaluate as usual.

---

# 19. Constraint Completeness

The schema defines the structurally legal domain:

\[
D_\Sigma
\]

For fixed supplied `referencedData`, the rules define the semantically valid
subset:

\[
V_{\Sigma,R,referencedData}
=
\{
data\in D_\Sigma
\mid
Valid(data,referencedData)
\}
\]

Therefore:

\[
\boxed{
V_{\Sigma,R,referencedData}
=
\{
data\in D_\Sigma
\mid
\forall rule\in R,\ Bool(Eval(rule,data,referencedData))=true
\}
}
\]

If the Libertas schema and its validation rules are taken as the normative definition of the data model, then:

```text
Schema
+
Validation Rules
=
Semantic Type
```

or formally:

\[
\boxed{
SemanticType(\Sigma,R,referencedData)
=
\{data\in D_\Sigma\mid Valid(data,referencedData)\}
}
\]

<!-- rule: validation.reference.changes-result class=hard enforced=no tasks=validation-rules -->
Changing `referencedData` may change this subset without changing the schema,
rule, or current data. For example, the rule
`{$.maximum} == null || [] <= {$.maximum} : "ABOVE_MAXIMUM";` accepts `50`
when the supplied maximum is `100`, rejects it when the maximum is `40`, and
accepts it when there is no reference context because of its explicit null guard.

For a fixed schema, rule set, and `referencedData`, every schema-compliant
concrete data instance therefore has a deterministic validation result:

\[
\boxed{
Valid_{\Sigma,R,referencedData}: D_\Sigma
\rightarrow
\{VALID,INVALID\}
}
\]

---

# 20. Examples

## 20.1 Numeric Range

Schema:

```c
struct Range {
    float min;
    float max;
    float value;
}
```

Rules:

```c
[min] <= [max] : "INVALID_RANGE";

[value] >= [min] &&
[value] <= [max] : "VALUE_OUT_OF_RANGE";
```

---

## 20.2 Nullable Limit

```c
[limit] == null ||
[limit] >= 0 : "NEGATIVE_LIMIT";
```

---

## 20.3 Union

Schema:

```c
A {
    int x;
}

B {
    string name;
}

A | B item;
```

Rules:

```c
([item] is A && [item/x] >= 0) ||
([item] is B && [item/name] != null) : "INVALID_ITEM";
```

---

## 20.4 Union Projection Without Explicit Type Test

```c
[item/value] == null ||
[item/value] >= 0 : "NEGATIVE_VALUE";
```

This means:

> If `value` exists in the concrete union value, it must be non-negative.

---

## 20.5 Array

```c
count([items]) > 0 : "ITEM_REQUIRED";

all(item in [items],
    [item/name] != null &&
    [item/value] >= 0
) : "INVALID_ITEM";
```

---

## 20.6 Array of Union Values

```c
all(item in [items],
    ([item] is A &&
     [item/x] >= 0)
    ||
    ([item] is B &&
     [item/name] != null)
) : "INVALID_ITEM";
```

## 20.7 Named type, field, and function ownership

```typescript
/// @LibertasStringResources(ValidationMessages)
const ValidationMessages = {
    NONNEGATIVE: "Enter zero or more.",
    PERCENTAGE_MAXIMUM: "Enter a percentage no greater than 100.",
    INVALID_RANGE: "The lower limit cannot exceed the upper limit."
} as const;

/** @LibertasValidationRules('[] == null || [] >= 0 : "NONNEGATIVE";') */
export type NonNegative = number;

export interface Range {
    /** @LibertasValidationRules('[] == null || [] <= 100 : "PERCENTAGE_MAXIMUM";') */
    percentage: NonNegative;
}

/** @LibertasValidationRules('[minimum] == null || [maximum] == null || [minimum] <= [maximum] : "INVALID_RANGE";') */
export declare function configure(minimum: number, maximum: number, range: Range): void;
```

<!-- rule: validation.example.cumulative-ownership class=hard enforced=no tasks=validation-rules -->
The `percentage` value must satisfy both the named-type and field-local rules.
The function rule sees minimum and maximum in one root. The explicit null guards
are valid even though the source types are not nullable, because the editor may not
have received user input yet. If one of those arguments remains `null` at final
submission, its validation rule may pass while the independent mandatory-field
check still rejects the task data.

## 20.8 Reference-context lookup

```c
[] == null ||
{$.limits.maximum,$.legacy.maximum} == null ||
[] <= {$.limits.maximum,$.legacy.maximum} : "ABOVE_PRIOR_MAXIMUM";
```

This field-level rule uses the first non-null maximum found in the current
reference context under the shared source-then-path ordering. During first-time
composition with no reference context, every curly reference
is `null`. The explicit null guard makes this rule pass in that state without
evaluating the comparison against an absent maximum. Once a reference object
supplies a maximum, the same rule enforces it. A typed `|` fallback can instead
supply an application-selected limit when the referenced value is absent.

## 20.9 Calendar-date constants

`issued` and `arrival` are DateOnly fields:

```c
[issued] == null || [issued] >= d"1990-01-01" : "ISSUED_BEFORE_1990";

[arrival] == null ||
[arrival] >= d"2026-01-01" && [arrival] <= d"2026-12-31" : "ARRIVAL_OUTSIDE_TAX_YEAR";
```

The same constant can bound the date chooser itself, as in
`@LibertasRangeMin('d"1990-01-01"')`.

## 20.10 Bitflags membership

`facts` is an integer whose `Bitflags` attribute names the catalog
`PersonFacts`, and `dependentSupport` is a Number:

```c
[facts] == null ||
!([facts] has PersonFacts.ReleasedToYou) ||
[dependentSupport] != null : "RELEASED_CLAIM_NEEDS_SUPPORT";
```

<!-- rule: validation.bitflags.no-bit-arithmetic class=judgment enforced=no tasks=validation-rules -->
When the person's facts include `ReleasedToYou`, the support amount is
required, regardless of which other flags are set. Without the operator the
same rule needs the bit position, as in `[facts] % 2 == 1`.

## 20.11 Calendar years and month periods

`firstYear` is an integer year and `deathDate` a DateOnly field. An inherited
Roth account's first year is not after the year of death:

```c
[firstYear] == null || [deathDate] == null ||
[firstYear] <= year([deathDate]) : "ROTH_FIRST_YEAR_AFTER_DEATH";
```

`periodStart` and `placedInService` are DateOnly fields and `periodMonths` is
the integer 24 or 60. The measuring period's last day,
`add_months([periodStart], [periodMonths]) - 1`, falls in the placed-in-service
year:

```c
[periodStart] == null || [periodMonths] == null || [placedInService] == null ||
year(add_months([periodStart], [periodMonths]) - 1) == year([placedInService])
    : "MEASURING_PERIOD_OUTSIDE_SERVICE_YEAR";
```

A 24-month period beginning `d"2024-01-01"` ends `d"2025-12-31"`; one beginning
`d"2024-03-15"` ends `d"2026-03-14"`.

## 20.12 Totals and composite keys

Each element of `contributions` has a Number `amount`, and each element of
`carrybacks` a `credit` and an integer `year`:

```c
[correctedPrincipal] == null ||
[correctedPrincipal] <= sum([contributions/amount]) : "PRINCIPAL_ABOVE_CONTRIBUTIONS";

is_unique_set(row in [carrybacks], [row/credit], [row/year]) : "CARRYBACK_DUPLICATE";
```

With no contributions the sum is `0`, so any positive principal fails. Two
carryback rows may share a credit or a year, but not both.

---

# 21. Core Design Principles

The Libertas Data Validation Rules language is based on the following principles:

1. **Validation operates on concrete data and incomplete client editor state.**
2. **Schema compliance and semantic validity are separate.**
3. **Current data uses `[a/b/c]`; external lookup uses explicit `{...}` paths.**
4. **A path resolves to `null`, one concrete value, or an ordered collection when array fan-out selects multiple nodes.**
5. **Union types require no special path-access mechanism.**
6. **`is` tests the selected Union variant or Enumeration member.**
7. **Null values use ordinary `== null` and `!= null`, including on required fields during editing.**
8. **A rule that accepts temporary `null` never overrides final mandatory-field validation.**
9. **`FinalOnly` defaults to `false`; it is a rule-set directive, not a schema attribute, and only explicit `true` makes that declaration final-only.**
10. **`null` implicitly becomes `false` in boolean context.**
11. **No other general truthiness conversion exists.**
12. **Single `|` returns the first non-null operand unchanged and short-circuits every later operand.**
13. **Rules are pure and deterministic for fixed schema interpretation, rule, data, and supplied referencedData.** <!-- rule: validation.principles.cursor-traversal class=hard enforced=no tasks=validation-rules -->
14. **Arrays use cursor traversal and collection operators rather than general-purpose loops.**
15. **For fixed referencedData, a valid rule set defines a total VALID/INVALID decision over every schema-compliant concrete data instance.**

The schema and pure rules define the model; each evaluation additionally
supplies `referencedData`:

\[
\boxed{
\text{Libertas Semantic Data Type}
=
\text{Schema}
+
\text{Pure Validation Rules}
}
\]

## Dynamic property expressions

<!-- rule: dynamic.expression.grammar class=hard enforced=yes tasks=data-model title="Dynamic reuses typed expression tokens with Name = expression; assignments" -->
The schema's [Dynamic attribute](schema.md#dynamic-schema-properties) uses
`PropertyName = expression;` assignments. Reuse this language's tokenizer,
operators, functions, semantic types, paths, and string literals. Each expression
has a Boolean result, permitting null as unavailable; a known incompatible result
type is a Studio error even beside an unresolved reference. There is no message
key, ValidationRules pair suffix, or FinalOnly directive. Property definitions
cannot be used as expression variables or called as functions.

The parser recognizes delimiters through tokens and expression nesting. A
semicolon or equals sign inside a string is data, and `==` is a comparison.
Inner single- and double-quoted strings decode `\n`, `\r`, and `\t`;
a backslash before another character yields that character. Native annotation
extraction precedes these rules; follow the [source quoting rules](annotations.md#dynamic).

<!-- rule: dynamic.expression.failure.authoring class=hard enforced=no tasks=endpoint-protocol -->
Account for [unsettled calculation inputs](#dynamic-property-expressions) withholding Dynamic permission until the expression resolves.

<!-- /rule -->

<!-- rule: dynamic.expression.failure class=hard enforced=no tasks=client-runtime title="Dynamic evaluation permits only true and cannot turn unavailable calculation data into permission" -->
Only an actual Boolean true grants permission. Null, unresolved values, wrong
runtime types, and evaluation failures grant nothing. Evaluation failures use
ordinary client diagnostics. Existing left-to-right short circuiting still
applies; an unevaluated operand creates no runtime read. A read of a calculation
output whose producer is unsettled withholds permission for the entire
expression, including through negation or fallback. This does not change the
stored output value or the ordinary semantics of ValidationRules.
