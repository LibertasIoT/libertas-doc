# From declaration to page

An App describes its interaction once, as ordinary types and comments in its
own language. That declaration is all its author writes. Studio extracts a
schema from it, and three kinds of consumers use the same declaration:

- **People** use a page the client draws from the schema.
- **AI** calls a tool the Hub generates from the same declaration and comments.
- **Programs**, other Apps, use the declared types through the SDK.

All three send the same Avro bytes, so the App's code does not change with the
sender. The App never reads a schema and never draws a screen: it receives and
returns its own native values.

**[Open the interactive step-through](../assets/interactive/declaration-to-page.html)**
to follow one small tax interview through seven steps, in Rust or TypeScript,
and switch between People, AI and Programs at each request.
The rest of this page tells the same story without the animation.

## The declaration

Every data interaction the App offers is declared in one place: its protocol
Union, `TaxInterviewProtocol`. Each variant is one message, either a request a
person, an AI or another App sends, or a reply the App returns. Pages, AI tools
and SDK calls all come from this one type, so there is no separate API
description, form or tool list to keep in step with it.

Comments become the words people see; attributes say which messages the client
sends (`Request`) and which the App answers (`Response`). `Started` also carries `WorkflowRequest`: when it
arrives as a reply, the client opens the named request editor at once.

```rust
/// Filing status
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub enum FilingStatus {
    /// Single
    Single,
    /// Married filing jointly
    MarriedJointly {
        /// Spouse's first name
        spouse: String,
    },
    /// Head of household
    HeadOfHousehold,
}

/// Tax interview
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub enum TaxInterviewProtocol {
    /// Start my return
    #[libertas_request]
    Start,
    /// Your interview has started
    #[libertas_response]
    #[libertas_workflow_request("SaveFilingStatus")]
    Started,
    /// Your filing status
    #[libertas_request]
    SaveFilingStatus {
        /// What is your filing status?
        status: FilingStatus,
    },
    /// Saved
    #[libertas_response]
    Saved {
        /// Message
        message: String,
    },
}
```

The TypeScript declaration is a discriminated union. Each variant's first
field, `kind`, is a member of a numeric enum, and that member's name is the
variant name that `@LibertasWorkflowRequest` refers to.

```typescript
export enum FilingStatusKind { Single, MarriedJointly, HeadOfHousehold }
/** Single */
export interface Single { kind: FilingStatusKind.Single; }
/** Married filing jointly */
export interface MarriedJointly {
  kind: FilingStatusKind.MarriedJointly;
  /** Spouse's first name */
  spouse: string;
}
/** Head of household */
export interface HeadOfHousehold { kind: FilingStatusKind.HeadOfHousehold; }
/** Filing status */
export type FilingStatus = Single | MarriedJointly | HeadOfHousehold;

export enum Step { Start, Started, SaveFilingStatus, Saved }
/** Start my return
 * @LibertasRequest */
export interface Start { kind: Step.Start; }
/** Your interview has started
 * @LibertasResponse
 * @LibertasWorkflowRequest("SaveFilingStatus") */
export interface Started { kind: Step.Started; }
/** Your filing status
 * @LibertasRequest */
export interface SaveFilingStatus {
  kind: Step.SaveFilingStatus;
  /** What is your filing status? */
  status: FilingStatus;
}
/** Saved
 * @LibertasResponse */
export interface Saved {
  kind: Step.Saved;
  /** Message */
  message: string;
}
/** Tax interview */
export type TaxInterviewProtocol = Start | Started | SaveFilingStatus | Saved;
```

## The bytes

Each message is the Avro encoding of one value: the variant index first, then
the fields in declaration order. Strings are a length followed by UTF-8 bytes;
Avro writes integers in zigzag form, so variant 1 is `02` and length 4 is `08`.
The bytes carry only values, never names, which is why the client needs the
schema to read them.

| Step | Direction | Value | Bytes (hex) |
| --- | --- | --- | --- |
| The person starts | Client → App | `Start` | `00` |
| The App answers | App → Client | `Started` | `02` |
| The person submits | Client → App | `SaveFilingStatus { status: MarriedJointly { spouse: "Alex" } }` | `04 02 08 41 6C 65 78` |
| The App replies | App → Client | `Saved { message: "Saved" }` | `06 0A 53 61 76 65 64` |

A TypeScript App exchanges the same bytes: its `kind` field is the variant
index, written once.

This is not the Web. No page, markup or script travels with a message: the
client already has the App's schema, so one byte, `Started`,
opens a full editing page, and the whole interview is 16 bytes of data.

## The pages

1. The client offers the protocol's requests. The person chooses
   **Start my return**, and the App receives a native `Start` value.
2. The App answers `Started`. Because it carries `WorkflowRequest`, the client
   does not show it as a page; it opens the **Your filing status** editor,
   drawn entirely from the `SaveFilingStatus` schema.
3. The person picks **Married filing jointly**. The schema says that choice
   carries a spouse, so the client asks for the spouse's first name under it.
4. Nothing is sent while the person edits. **Next** validates the request and
   sends it as seven bytes; the App decodes them into its own type and matches
   `MarriedJointly` with the spouse inside.
5. The App replies `Saved`, and the client shows the reply from its schema.

Each client lays out the same schema in its own native controls; the drawings
in the step-through are conceptual, not product screenshots.

## The same request from an AI

For an AI assistant, the Hub turns each Request variant into a tool. The
variant's name is the tool's name, unless another tool already uses it; its
comment is the tool's description; its fields are the tool's parameters, with
their comments as titles and descriptions. A named Union becomes a `oneOf` in
`definitions`, one object per variant, keyed by the variant's name. This is
the tool the Hub generates for `SaveFilingStatus` from the Rust declaration:

```json
{
  "name": "SaveFilingStatus",
  "parameters": {
    "type": "object",
    "properties": {
      "status": {
        "$ref": "#/definitions/FilingStatus",
        "title": "What is your filing status?",
        "description": "What is your filing status?"
      }
    },
    "required": ["status"],
    "definitions": {
      "FilingStatus": {
        "oneOf": [
          {
            "type": "object",
            "properties": {
              "Single": { "type": "null", "title": "Single", "description": "Single" }
            },
            "required": ["Single"],
            "additionalProperties": false
          },
          {
            "type": "object",
            "properties": {
              "MarriedJointly": {
                "type": "object",
                "properties": {
                  "spouse": {
                    "type": "string",
                    "title": "Spouse's first name",
                    "description": "Spouse's first name"
                  }
                },
                "required": ["spouse"],
                "title": "Married filing jointly",
                "description": "Married filing jointly"
              }
            },
            "required": ["MarriedJointly"],
            "additionalProperties": false
          },
          {
            "type": "object",
            "properties": {
              "HeadOfHousehold": {
                "type": "null",
                "title": "Head of household",
                "description": "Head of household"
              }
            },
            "required": ["HeadOfHousehold"],
            "additionalProperties": false
          }
        ],
        "title": "Filing status",
        "description": "Filing status"
      }
    }
  },
  "description": "Your filing status"
}
```

These comments are only display names, so each description repeats its
title. A second comment line, the description, would give the AI more to
read. Depending on the AI provider, the Hub wraps this definition in that
provider's tool format. `Start` becomes a tool with no parameters.

The AI calls the tool with:

```json
{ "status": { "MarriedJointly": { "spouse": "Alex" } } }
```

The Hub converts those arguments into the declared value and encodes it, so
the App receives `04 02 08 41 6C 65 78`, the same bytes as from the page. A
`null` for a field that is not Nullable, such as `"spouse": null`, cannot be
encoded, and the Hub rejects the call. The rules for tool descriptions are in
[AI tool-call descriptions](../reference/presentation.md#ai-tool-call-descriptions);
`#[libertas_llm_tool_call(false)]` or `#[libertas_exclude_ui]` leaves a Request
out of the tools.

## The same request from another program

Another App that declares a `LibertasEndpoint` field with
`#[libertas_endpoint_schema(TaxInterviewProtocol)]` uses the same types. Its
task's configuration connects that Endpoint to this App. Sending the request
is a few lines:

```rust
use libertas::libertas_endpoint_request;
use tax_interview::{FilingStatus, TaxInterviewProtocol};

let request = TaxInterviewProtocol::SaveFilingStatus {
    status: FilingStatus::MarriedJointly { spouse: "Alex".into() },
};
libertas_endpoint_request(interview, &request);
```

```typescript
import { EndpointAction } from "@libertas-ai/consts";
import { FilingStatusKind, Step } from "tax-interview";

libertasEndpointSendRequest(interview, EndpointAction.Request, {
  kind: Step.SaveFilingStatus,
  status: { kind: FilingStatusKind.MarriedJointly, spouse: "Alex" },
});
```

The SDK encodes the value with the same generated code, so these bytes are
also `04 02 08 41 6C 65 78`. The reply arrives through the program's endpoint
listener as a native `TaxInterviewProtocol` value.

## Where the rules live

- [Rust](../reference/languages/rust.md) and
  [TypeScript](../reference/languages/typescript.md#discriminated-unions-union-types)
  declaration syntax.
- [Unions](../reference/schema.md#54-unions) and their variant order.
- [Workflow interactions](../reference/protocols.md#workflow-interactions) and the
  [WorkflowRequest role](../reference/protocols.md#workflowrequest-semi-role).
- [AI tool-call descriptions](../reference/presentation.md#ai-tool-call-descriptions)
  and the [tool-call switch](../reference/annotations.md).
- [Endpoint schemas](../reference/languages/rust.md#endpoint-schema) for an App
  that uses another App's protocol.

[Concepts](index.md)
