# Logically complete declarations

**Logically Complete. Script-Free Interaction.**

A Libertas App declares the data and interactions its users work with. The
client supplies the complete UI behavior for that declared interaction:
conditional fields and choices, calculated values, editing permissions,
validation feedback, submission, waiting, results, errors and navigation.
App authors do not write GUI scripts to connect those states.

This is **client UI state completeness**. Given a valid declaration within the
supported platform contract, every reachable UI state and applicable event
has defined client behavior. The service is assumed to implement its declared
operations correctly; business-logic correctness is outside this claim.

## What logical completeness means here

A complete UI gives the person a defined experience throughout an interaction.
It covers the intermediate states as well as the finished screen: an
unanswered question, a changed choice, an invalid draft, a pending request,
a returned result, an error and leaving the interaction.

For each supported state, the contract determines:

- **What is presented:** the current values, choices, descriptions and status.
- **What the person can do:** the available edits, actions and navigation.
- **What happens next:** how an accepted input, reply, failure or exit changes
  the UI, and which unavailable or stale events have no effect.

An unanswered field is therefore a defined UI state, even though it is not a
valid submitted value. A disabled action is also defined behavior. Completeness
does not mean every action is available at every moment; it means the client
has no missing case that an App author must fill with a GUI script.

The declaration works together with the shared client rules. Types describe
values and choices; attributes describe constraints and interaction roles;
localized documents supply meaning and text; the Display Template controls
arrangement. The client owns the interaction lifecycle. The
[App model](../reference/app-contract.md#application-model) defines these
responsibilities, and the
[smallest logically complete design principle](../reference/app-contract.md#required-design-principle)
requires authors to use them without duplicating them.

## The UI guarantees

For valid declarations and a conforming client, the following are contract
guarantees, rather than capabilities an App author might choose to implement.
The linked references own their exact conditions.

| UI state or event | Guaranteed client behavior |
| --- | --- |
| A declared choice is presented | The client derives the eligible choices and the selected branch's fields from the declaration. [Union selectors](../reference/presentation.md#promotion-flattening-and-union-selectors) and [Nullable choices](../reference/presentation.md#nullable-choices) define their presentation. |
| Input changes a calculated value or a condition on an operation | The client applies the declared calculation dependencies and reevaluates affected operation conditions, including their pending and unavailable states. See [conditional GUI behavior](#conditional-gui-behavior). |
| A required answer is missing or input is invalid | The editor represents the unfinished input; final validation blocks submission until its requirements pass. See [validation](../reference/validation-rules.md). |
| A person submits a request requiring confirmation | The shared confirmation procedure offers Send and Cancel; Cancel sends nothing. See [request confirmation](../reference/protocols.md#request-confirmation). |
| A submitted request awaits its reply | The owning surface blocks conflicting edits and duplicate transitions. The owning reply or a terminal failure settles the wait. See [operation ownership](../reference/protocols.md#operation-ownership). |
| The service returns a result or another editor | The client applies the declared reply semantics. A delivered workflow editor awaits submission; a final workflow response is displayed. See [workflow interactions](../reference/protocols.md#workflow-interactions). |
| A correlated Error reply arrives or a reply is refused | The shared error procedure preserves the interaction for correction and retry. See [protocol errors](../reference/protocols.md#protocol-roles) and [reply refusal](../reference/protocols.md#entering-and-continuing-a-workflow). |
| The person cancels or leaves | An idle composer's Cancel returns to its underlying screen without sending. A pending operation follows the endpoint-exit rule. See [composer lifetime](../reference/endpoint-presentation.md#protocol-transaction-stack) and [exit while pending](../reference/protocols.md#exit-while-pending). |
| A stale reply or unrelated update arrives | It cannot take over a replacement editor or perform another operation's navigation. See [operation ownership](../reference/protocols.md#operation-ownership). |

These are semantic guarantees, not a promise of identical pixels on every
screen. Each client presents the interaction using its supported controls and
layout. An implementation that violates a defined transition has a client bug;
the App does not acquire responsibility for supplying a replacement handler.

## Conditional GUI behavior

Conditional interaction is part of the completeness guarantee. Its structure,
changing values and permitted operations are all expressed through declarations
that the shared client interprets.

### Union: statically declared conditional structure

Consider a notification preference with three alternatives: **Email**, carrying
an address; **Text message**, carrying a phone number; and **Do not send**,
carrying no destination. This is a conceptual example of a Union.

The client handles an initially unselected choice, selecting any eligible
branch, editing its payload, changing the choice and validating the resulting
request. With Email selected, it presents the address editor; with Do not send
selected, there is no destination to enter. A required unanswered choice or
invalid address prevents submission according to its declarations.

The author supplies the choice, its fields, constraints and explanations.
The platform supplies the UI states and transitions. Nesting that choice in a
record or list uses the same shared mechanisms. There is no separate
application-specific handler to remember for each branch.

The conditional structure is static: the Union declares the alternatives and
their payloads in advance. The selected branch can change at runtime, and the
client presents the corresponding fields. Thus conditional GUI structure
already follows from the data type itself.

### CalculationRequest: data-dependent values and choices

[CalculationRequest](../reference/protocols.md#field-calculation-requests)
declares which calculations depend on an input field. `CalculationRoot` binds
each calculation to its occurrence, and `CopyTo` maps its results back into
the editor. The client owns scheduling, result application, dependent updates
and stale-reply rejection.

This supports recalculated totals, changing bounds and
[calculated allowed choices](../reference/protocols.md#calculated-allowed-sets).
A calculation can also assign a Union value: its selected branch and fields
then change through the same presentation rules as a direct edit. These are
conditional UI transitions without App-specific change handlers.

The pending states are defined too. Ordinary input editing continues while
calculations run, subject to its editing permissions; Submit is disabled while
calculations are pending or deferred. Outputs keep their previous values until
a successful result assigns replacements. The
[calculation lifecycle](../reference/protocols.md#changes-resets-and-deferred-calculation)
owns the complete rules for errors, retries and submission.

### Dynamic: data-dependent operations

[Dynamic](../reference/schema.md#dynamic-schema-properties) declares conditions
for `Editable`, `Addable` and `Removable`. A field can become editable only
under a condition; a List can permit new members while protecting existing
ones; individual members can have different removal conditions. Expressions
read local values or interaction reference data, including values supplied by
calculations. `Dynamic` evaluates those conditions locally; it does not itself
invoke a calculation.

For example, changing an order's quantity can calculate its total and whether
delivery instructions are editable. `Dynamic` reads that result to control
the instructions field. The client reevaluates the condition after applying
the result. While the relevant calculation is invalidated, pending or failed,
the dependent operation is unavailable; unrelated operations retain their own
conditions. See [dynamic permissions and calculations](../reference/protocols.md#dynamic-permissions-and-calculations).

Only Boolean true grants a declared operation, subject to the other schema
restrictions. The client rechecks the condition when committing a change,
including after a dialog or import. Losing permission preserves the committed
value; it does not erase data or suspend validation. Those defined outcomes
cover the transitions between available and unavailable controls, with no
additional GUI script.

## Why this goes beyond declarative screen syntax

Declaring a button and attaching an application-specific handler still leaves
that application responsible for what the handler does to the UI. A Libertas
declaration describes the operation and its values, while the platform already
owns the supported editing and interaction behavior.

Scripted interfaces can also use sound types and consistent state models.
React's [state-design guidance](https://react.dev/learn/choosing-the-state-structure),
for example, recommends avoiding contradictory and duplicated state. Libertas
makes the shared interaction model a platform contract, so each App does not
have to construct and maintain its own UI state machinery.

| Responsibility | With application-specific GUI scripts | With Libertas |
| --- | --- | --- |
| Choices and input | Connect controls, selected branches and their values. | Declare the data; the client supplies the supported controls and editing states. |
| Conditional GUI behavior | Coordinate branch changes, recalculation, dependent choices and operation availability. | Declare Union branches, calculation dependencies and Dynamic conditions; the client supplies their combined behavior. |
| Submission and waiting | Coordinate validation, duplicate clicks and pending state. | Use the shared validation and operation lifecycle. |
| Results, errors and navigation | Connect each response to the appropriate UI transition. | Declare interaction roles; the client applies their defined behavior to the reply. |
| Another client | Implement and maintain the corresponding interface behavior. | A conforming client interprets the same interaction contract. |

The [declaration-to-page example](declaration-to-page.md) follows one
interaction through its declaration, generated editor, AI tool and typed
message. The [layout example](data-is-everything.md) shows one description
adapted to different window widths.

## What becomes guaranteed, and what remains a benefit

**Complete handling of supported UI states is the platform's responsibility.**
It is not an optional per-App implementation. The same is true of the specified
validation feedback, operation ownership and navigation behavior. This removes
whole categories of App-specific UI code and the possibility of omitting one
of those handlers in an individual App.

**People, AI and other Apps share the declared operation.** A person edits
generated controls, an AI calls an eligible schema-described tool, and another
App sends a typed Request. The [AI tool contract](../reference/protocols.md#ai-tool-calls)
defines access, confirmation and the author's controls over tool exposure.
The Hub enforces [request access](../reference/protocols.md#request-access)
for both native client requests and AI invocations.
The common description is guaranteed by the architecture; an AI's ability to
understand a person's intent is a separate matter.

**Meaning and presentation have separate owners.** The
[Display Template](../reference/presentation.md#display-template) arranges values
without changing their meaning. [Localized documents](../reference/translation.md)
supply translated labels and explanations without changing the schema.
Translations still have to be supplied, but no GUI script is needed to attach
each translation to a hand-built screen.

**Shared improvements benefit compatible Apps.** A correction to a common
control or interaction rule applies wherever that mechanism is used.
Accessibility improvements and support for additional devices can likewise be
implemented in clients and reused across Apps. A future modality, such as a
full voice interface, still needs a capable client implementation.

Lower maintenance costs, fewer defects and easier auditing are consequences
this architecture supports, rather than universal numerical guarantees. Their
size depends on the App and the client implementation. The concrete saving is
the removal of repeated application-specific UI machinery.

## How to validate the completeness claim

The relevant test is the client's coverage of UI states and events. Checking
that an Enumeration lists its members is only one part. A completeness review
also follows unfinished edits, branch changes, calculation dependencies,
changing operation conditions, validation failures, pending operations,
replies, errors, cancellation and stale events, including their
combinations in nested structures and different presentation contexts.

Studio's [schema validation](../reference/schema.md#9-static-and-runtime-validation)
checks that declarations meet the supported rules, including type, role and
navigation-target requirements. Shared client tests then check the defined
runtime transitions. A passing schema check alone is not an exhaustive proof
of a client implementation; the guarantee comes from the UI contract, and
implementation conformance is what those tests verify.

The scope is the platform's supported types, interactions and presentation
mechanisms. If an interaction within that scope lacks a defined UI state or
transition, it is a platform completeness defect. If a new interaction needs
an additional mechanism, it is a
[platform capability to add](../reference/app-contract.md#required-design-principle),
rather than a reason for every App to build its own GUI state machinery.

[Concepts](index.md) · [Start here](../principles.md)
