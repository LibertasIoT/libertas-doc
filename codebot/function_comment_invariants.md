# Function-comment template

Shape for a public function's comment. The
[source documentation card](source_documentation.md) quotes every rule from the
[developer function-comment guide](../reference/function-comments.md),
including its seven
[completeness categories](source_documentation.md#completeness-invariant), the
[actor, action and scope](source_documentation.md#actor-action-and-scope-invariant)
wording and the [attribution](source_documentation.md#developer-attribution-invariant).

## Attribution

End every codebot-generated description with exactly:

```text
Developed by Display Name (account_name).
```

Before the developer has a Plaza account:

```text
Developed by account_name.
```

## Template

```text
Concise activity name
The domain actor performs the main activity under its material condition.
The user chooses the applicable setup values and their scope.
The user can perform the applicable controls or overrides.
The domain actor automatically performs applicable behavior and says what
influences it.
The App shows, records, summarizes, or delivers applicable results and states
material prerequisites or limits.
Developed by Display Name (account_name).
```

Omit inapplicable sentences; never invent behavior. The guide's
[example](../reference/function-comments.md#example) shows a complete comment
in Rust and TypeScript.
