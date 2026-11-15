# Coming from the web

If you have built pages with HTML, CSS and JavaScript, most of what you know
has a Libertas counterpart, but it lives somewhere else. On the web you write
the page: markup for structure, a stylesheet for appearance and script for
behavior. In Libertas you declare the data. The schema extracted from your
types and attributes is the structure, a [Display Template](../reference/presentation.md#display-template)
arranges it from a closed catalog, and the [protocol](../reference/protocols.md)
supplies the behavior. Android, iOS and Studio/Web each draw the same declaration
natively; your App contains no UI code.

The tables below map familiar web ideas to their Libertas owners. Each link
leads to the page that holds the exact rule.

This correspondence is work in progress, and new semantics may be added later.
The design principle ensures that any web behavior not yet covered can be added
under the same principle: as a declared semantic of the data that every client
evaluates, never as script in a page. What Libertas leaves out on purpose,
listed at the end, are mechanisms such as page script and free-form styles, not
the behaviors they produce.

## Structure: HTML

| On the web | In Libertas |
| --- | --- |
| A page | One Request or Response value, drawn as a page |
| `<form>` | A Request editor; Submit sends one complete Request, never a partial one |
| `<fieldset>`, `<section>`, `<div>` | A Table type, arranged by template [layout groups](../reference/presentation.md#display-template-catalog) that nest to any depth |
| `<input type=text>`, `<textarea>` | A String field; [`Multiline`](../reference/annotations.md) for several lines |
| `<input type=password>` | A String field with [`Secret`](../reference/presentation.md#secret-values): masked, with a Show control, and withheld from AI |
| `<input type=number>` with `min` and `max` | A number field with [`RangeMin` and `RangeMax`](../reference/validation-rules.md#31-dynamic-numeric-ranges), plus a [physical unit](../reference/presentation.md#physical-units) for quantities |
| `<input type=date>`, `datetime-local` | [DateOnly and DateTime](../reference/presentation.md#date-and-time), typed calendar dates and instants |
| `<select>`, radio buttons | An Enumeration, or a Union when a choice carries its own data; drawn as a picker or as [inline choices](../reference/presentation.md#inline-choices) |
| A group of checkboxes | [Bitflags](../reference/presentation.md#bitflags), with a zero label such as "None of these" as the exclusive answer |
| `<input type=checkbox>` | A [Boolean](../reference/presentation.md#boolean-answer-controls) whose texts name what true, false and null mean |
| `placeholder`, an empty field | An [unanswered field](../reference/presentation.md#unanswered-fields-and-null); a field is never filled in by accident |
| `value=` | [`Default`, `Fixed` or `CopyFrom`](../reference/schema.md#ordinary-initialization) |
| `readonly`, `disabled` | `ReadOnly`, or `Fixed`, which also locks the value |
| `type=hidden`, `display:none` | The schema's `Hidden`; no style token hides a field |
| Repeating rows | A List, with a table, grid or summary layout |
| `<label>`, `title` | The field's name and doc comment, shown under the [label rules](../reference/presentation.md#labels-and-titles) |
| `<img>`, `<video>` | [Bytes with a MIME type](../reference/presentation.md#mime-aware-bytes-presentation) and the `media` hint |
| A chart library on `<canvas>` | [Chart declarations](../reference/presentation.md#charts) on a Response or subscription payload |
| Rich read-only HTML | [FormattedText](../reference/presentation.md#string-resources-and-formatted-output), always read-only |
| `<a href>` | A [Link](../reference/protocols.md#links) to one Request of the same protocol |
| A same-page fragment link | A row's [reveal path](../reference/protocols.md#in-page-reveal-paths): it scrolls to the field and highlights it without sending anything |
| A link to another page with a fragment | A Request's [reveal path](../reference/protocols.md#request-reveal-paths), applied once to the page its reply opens |
| `autofocus`, initial scroll position | The page's [landing path](../reference/protocols.md#landing-paths), set by the service |
| `id` and selectors | A [Libertas Schema Path](../reference/schema.md#libertas-schema-path); a key step finds a List member by its key, even after reordering |

## Appearance: CSS

| On the web | In Libertas |
| --- | --- |
| A stylesheet | `display.json`, compiled into the [Display Template](../reference/presentation.md#display-template) |
| The cascade | The [template cascade](../reference/presentation.md#display-template-cascade), with rules matched by schema path |
| `@media` queries | [`when` conditions](../reference/presentation.md#display-template-conditions): window size class, orientation, pointer, theme, device form and edit or view mode |
| Flexbox and Grid | The `rows`, `form-grid`, `grid`, `split`, `card`, `table` and `summary` layouts, adapted to every width by the [width rules](../reference/presentation.md#width-based-layout) |
| Tabs and accordions | Template groups, and [folding](../reference/presentation.md#folding-structs-and-unions) |
| Colors, fonts and pixel values | [Style tokens](../reference/presentation.md#style-tokens) only: platform color roles, typography roles, a spacing scale, density, emphasis, alignment and icons |
| Hiding empty or trivial content | The `when_empty` and `when_single_choice` [behavior hints](../reference/presentation.md#behavior-hints); presentation only, values unchanged |

## Behavior: JavaScript and the DOM

| On the web | In Libertas |
| --- | --- |
| `onsubmit` and `fetch` | Submit sends a Request; the service replies with a Response |
| Client-side validation | [ValidationRules](../reference/validation-rules.md); the service's save path stays authoritative |
| Recomputing fields on `onchange` | [Field calculations](../reference/protocols.md#field-calculation-requests) with `CopyTo` |
| Script that copies values between fields | [`CopyFrom` and `CopyTo`](../reference/schema.md#shared-copyfrom-and-copyto-rules), declared rather than scripted |
| Redirects and single-page routing | `NextRequest`, `EditRequest` and `WorkflowRequest` in the [workflow contract](../reference/protocols.md#workflow-interactions) |
| A wizard's Back button | `PrevRequest`; the client keeps no history of earlier requests |
| WebSocket updates | Subscriptions, delivering ongoing data |
| An error beside a field, then `focus()` | An Error with an [error field path](../reference/protocols.md#error-field-path) |
| Draft autosave in `localStorage` | [CacheIdentity](../reference/annotations.md#cacheidentity), which recovers drafts locally and never sends them |
| Buttons and menus | The Actions surface and [named action controls](../reference/presentation.md#named-action-controls) |
| `aria-*` attributes | Derived by every client from labels and roles |
| Autofill and search | [AI-native features](ai-native.md): import from photos and PDFs, and one-word navigation |

## What Libertas leaves out on purpose

- Free-form styles, pixel values, literal colors and fonts. Every client draws
  the same closed catalog well.
- Script in the page. Behavior is declared, so every client behaves alike.
- Step-by-step screens inside one page. A sequence of steps is a
  [workflow](../reference/protocols.md#workflow-interactions), where the
  service validates and branches each step.
- Hiding by style. What a person must not reach stays out of the data.

[Understand the platform](index.md)
