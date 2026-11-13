# Understand the platform

Concept explanations connect the steps in the guides to the behavior you see.
Use them when you understand what to click but want to know why the workflow
works that way.

[Three paradigm innovations](paradigm.md) gives the overview: comments
become every text, one Union defines all interaction, and one Union defines
all stored data.

Start with [Hub architecture: resources and interaction](hub.md) for the
platform's execution and coordination core, and how human users, AI and
applications participate.

[Logically complete declarations](logically-complete-declarations.md) explains
how the platform covers client UI states and transitions without App-specific
GUI scripts, and which guarantees follow from that contract.

[From declaration to page](declaration-to-page.md) follows one protocol from
its source declaration through its Avro bytes to the pages a client draws from
the schema, with an interactive step-through.

[Data is everything](data-is-everything.md) shows one declaration and three
arrangement rules laid out at every window width by the platform's width rules,
with an interactive page.

[AI-native by design](ai-native.md) shows what AI can do with an App written
only for people: act for the person, import photos and PDFs, and navigate by
one word.

[Coming from the web](from-the-web.md) maps HTML, CSS and JavaScript to
their Libertas counterparts, for developers who know how to build web pages.

## Apps and tasks

An App package is reusable software and its public types. A public function
can be configured as a task. Several tasks can use the same function with
different settings unless the function has a singleton restriction.

## Source, schema, and visible text

Source defines the App. Studio extracts the schema used to validate and
present values, and localized documents supply human-readable text. Changing
a translation should not change the schema's structure or runtime behavior.

## Actions and ongoing information

A task can expose operations and report data. A response belongs to an
interaction; a subscription provides ongoing information. Keeping these roles
clear helps explain why a confirmation can leave current data visible and why
live data may change without a new button press.

## Shared contracts

Two Apps can cooperate only when their types and protocols remain compatible.
Public identities, variant order, and supported capabilities matter across
versions. The [shared reference](../reference/index.md) is the common home for
those exact rules; [human guides](../guides/index.md) show how to use them.

[Documentation home](../index.md)
