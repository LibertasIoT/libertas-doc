# Presentation and localization

Presentation is the App-authoring projection of the full
[presentation and localization contract](../reference/presentation.md).
Presentation metadata never changes the underlying typed wire value: the
schema owns what the App accepts, sends, computes, protects or means, and the
package's Display Template owns only how the same value is arranged and
styled on a device. The rules are generated from that contract into four task
cards; load the card for the task and read its linked owner before
implementing.

| The task changes | Load |
| --- | --- |
| A field's text, metadata, answer controls, `Foldable`, `InlineChoices`, `SuppressLabel`, `UIIncludeDescription`, media or `Link` rendering | [Presenting a field](field_presentation.md) |
| `display.json`: layouts, master-detail panes, labels and titles, page controls, promotion and flattening, conditions, hints and style | [Writing a Display Template](display_template.md), with its arrangement procedure and [display presets](display_template.md#display-presets) |
| Date and time types, `Money`, `PhysicalUnit`, `Bitflags`, `Format`, `StringResources`, `FormattedText`, translation | [Formatting values and text](formatted_output.md) |
| Chart metadata and chart-producing requests | [Declaring a chart](charts.md), and [Declaring a 3D chart](charts_3d.md) for a 3D chart |

Complete applicable metadata proactively on every authored App — useful
initial task and server names, localized text defaults, value defaults, units,
bounds, inline help and grouping — and review those choices before delivery
without waiting for a separate user request. Each annotation must serve the
App's actual behavior; configuration-only examples need no invented protocol or
persistence. Check the [annotation inventory](schema_annotations.md) before
adding source syntax, and follow the [source documentation card](source_documentation.md)
for the text every node carries. The schema-side placement and value rules
of these attributes (`attr.*`, `format.*`, `numeric.*`, `bitflags.*`) are on
the [data model card](data_model.md).

How a chosen request is presented by its input, including the
[hierarchical choice popup](../reference/protocols.md#hierarchical-single-field-requests),
is on the [actions card](actions.md); the [messages card](messages_lmf1.md)
owns LMF1 templates and the `FormattedText` tuple.
