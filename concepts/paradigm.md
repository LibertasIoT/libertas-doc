# Three paradigm innovations

## 1. Comments speak to people and AI

Doc comments and attributes can serve as:

- GUI texts and controls;
- chart definitions;
- AI tool-call descriptions;
- AI form-filling prompts;
- AI navigation prompts.

The standard compiler ignores them; the Libertas compiler collects them.

```rust
/// Desired temperature
/// The temperature the user wants the selected sensor's area to reach.
#[libertas_number(min = -40, max = 100, step = 0.5)]
#[libertas_physical_unit("celsius")]
target: f64,
```

[From declaration to page](declaration-to-page.md) ·
[AI-native by design](ai-native.md)

## 2. One Union defines all interaction

That Union is the protocol: every form, link, page and live update comes from it.

A protocol, not an API:

- an API is a bare function that nothing wires into a process;
- an API runs only when its counterparty calls it; interaction must also send
  data nobody asked for, such as live updates;
- a protocol is a data loop, driven by a standard function inside the App's
  process until the App is stopped.

```rust
pub enum TemperatureProtocol {
    /// Read temperatures
    #[libertas_request]
    #[libertas_subscription_request]
    ReadStatus,
    /// Change desired temperature
    #[libertas_request]
    SetTarget { target: f64 },
    /// Temperature readings
    #[libertas_response]
    #[libertas_subscription_data]
    Status { current: f64, target: f64 },
}
```

[Protocols](../reference/protocols.md) ·
[Logically complete declarations](logically-complete-declarations.md)

## 3. One Union defines all stored data

Keys are a second Union. One place holds it all.

```rust
#[derive(LibertasKeyCodec)]
pub enum DataKey {
    /// Latest temperature
    Latest,
    /// Recording position
    Checkpoint,
}

pub enum StoredData {
    /// Saved temperature
    Sample { temperature: f64 },
    /// Saved recording position
    Checkpoint { sample_count: u64 },
}
```

[Transactional key-value API](../reference/protocols.md#transactional-key-value-api) ·
[Data type evolution](../reference/data-type-evolution.md)

## Unions do it all

A Union means "exactly one of these", and that one idea fits all three:

- a conditional GUI: choose a variant, and its own fields appear;
- a protocol: every message is one variant;
- stored data: every record is one variant.

## GUI semantics

The behavior of the generated GUI is declared, and the client evaluates it.

Where a behavior depends on the data, an expression in a typed, side-effect-free
expression language decides it: whether a value may be edited, a List member added
or removed, a value accepted, a number or date-time bound applied, or a
presentation condition met. The expression may perform simple calculations,
including arithmetic, calendar-date functions and collection operations.

The client evaluates each expression itself against the current data, without a
request to the App, so the page responds at once. Only an actual true result
grants an operation; unavailable or unsettled data grants nothing. An author may
equally place a calculation in the App through a field calculation request; the
choice is the author's.

An expression differs from general-purpose code: it always terminates, and for
the same data it gives the same result on every client. It has no
general-purpose loops, visits arrays only through cursors and collection
operators over the data supplied, and matches patterns without backtracking. It
reads only the data it is given, never the clock or a random source. Today an
expression only reads data, and its types are checked when the App is compiled.

```rust
/// Order line
#[libertas_dynamic(r#"Editable = [submitted] == false;"#)]
pub struct OrderLine {
    /// Submitted
    submitted: bool,
    /// Quantity ordered
    ordered: u32,
    /// Quantity shipped
    #[libertas_number(min = 0, max = "[^/ordered]")]
    shipped: u32,
}
```

[Dynamic properties](../reference/schema.md#dynamic-schema-properties) ·
[Validation rules](../reference/validation-rules.md) ·
[Evaluation rules](../reference/validation-rules.md#17-evaluation-purity) ·
[Dynamic ranges](../reference/validation-rules.md#31-dynamic-numeric-ranges) ·
[Template conditions](../reference/presentation.md#display-template-conditions) ·
[Field calculations](../reference/protocols.md#field-calculation-requests)

Snippets 1 to 3 are shortened from the
[temperature monitor recipe](../examples/app-recipes.md) and the
[Data protocol example](../reference/protocols.md#transactional-key-value-api);
the GUI semantics snippet is illustrative.

## CodeBot friendly

The three innovations leave a coding agent one job: write the declarations and
the logic behind them, in one language. There is no frontend, no separate API
layer and no screen code to keep in step.

- **One place per fact.** A field's screen, checks and AI prompts come from its
  one declaration, so an agent changes one place and nothing else drifts.
- **A compiler that answers.** `compile_schema` checks saved source with the
  platform's own validator and names the file and line; the agent fixes it and
  compiles again.
- **Rules an agent can find.** [CodeBot cards](../codebot/index.md) give each
  task its rules, generated from the reference.
- **QA an agent can run.** Studio's QA tools let an agent use a deployed App
  the way a person does: open pages, import documents with AI, answer, save
  and read every reply.

[Schema compilation](../reference/app-contract.md#schema-compilation-with-codex) ·
[Automated QA](../reference/app-contract.md#automated-qa-through-studio-mcp) ·
[Codex and Claude Code setup](../guides/studio.md#using-studio-mcp-with-codex-or-claude-code)

[Understand the platform](index.md)
