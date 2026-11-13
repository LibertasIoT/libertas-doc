# Data is everything

An App declares its data: fields, types and the words people read. It never
draws a screen. A few optional Display Template rules say how the data should
be arranged (two panes, a table, a form grid) but never how wide anything is.
Every client then fits that arrangement to the space it has, with the
platform's width rules. The same declaration works on a phone, a tablet and a
large display.

**[Open the interactive page](../assets/interactive/data-is-everything.html)**
to watch one window widen from phone to a 1920 dp desktop or TV and change its layout at the
widths the platform chooses. You can also drag the width yourself. The rest of
this page tells the same story without the animation.

## The declaration

One W-2 wage statement. Comments become the labels; `libertas_money` makes
three amounts money in US dollars. `TaxState` is an enum of states, and
`WageDeductionEligibility` has three answers: No amount to classify,
Ineligible — no deduction, and Eligible for the deduction.
`libertas_inline_choices` shows those answers as radio choices on the row
instead of a value that opens a chooser.

```rust
/// W-2 wage statement
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub struct W2 {
    /// Employer
    pub employer: String,
    /// Employer ID (EIN)
    pub ein: String,
    /// Wages, tips, other pay
    #[libertas_money("USD", 2)] pub wages: i64,
    /// Federal tax withheld
    #[libertas_money("USD", 2)] pub federal_withheld: i64,
    /// Social Security wages
    #[libertas_money("USD", 2)] pub ss_wages: i64,
    /// State
    pub state: TaxState,
    /// Retirement plan
    pub retirement_plan: bool,
    /// Overtime deduction
    #[libertas_inline_choices] pub overtime: WageDeductionEligibility,
}
```

The App's protocol has an `Income` response listing the W-2s in `w2s`, an
`EditW2` request editing one, and an `OpenIncome` request that loads the list.

## The adjustment

Five rules in the project's `display.json` ask for an arrangement. None names
a size, a device or a pixel. You can change them by hand, or ask an AI
assistant to change them while Studio previews them on live data.

```json
{ "select": "^<Income>", "right_pane": ["EditW2"], "refresh": "OpenIncome" },
{ "select": "^<Income.w2s>", "layout": "table",
  "columns": ["employer", "wages", "federal_withheld", "retirement_plan"] },
{ "select": "^<EditW2.w2>", "layout": "form-grid", "columns": 3 },
{ "select": "^<EditW2.w2>.employer", "style": { "span": 2 } },
{ "select": "^<EditW2.w2>.overtime", "style": { "span": 3 } }
```

- `right_pane` puts the W-2 editor beside the list when two panes fit.
- `table` shows the list as a table when its columns fit.
- `form-grid` lets the editor use up to three columns when they fit.
- `span` gives the employer row two columns and the overtime row three, so
  their long text keeps its room when the form has several columns. A span
  larger than the columns in use is reduced to them.

## What the platform decides

The window's width class caps the structure: one pane and one column below
600 dp, then two panes and two, three or four columns as the window widens.
Within that cap, the content decides:

- **Panes:** two panes when both pane minimums (360 dp each) and the 16 dp
  gutter fit, that is from 736 dp. The list takes 40% of the width after the
  gutter, but never less than 360 dp and never so much that the editor gets
  less than 360 dp.
- **Form columns:** as many 360 dp columns, with 16 dp between them, as fit
  the editor's width, never more than `columns` or the window's cap. A field
  with a `span` takes that many columns and the gutters between them.
- **Table:** a table while the measured minimum widths of its columns, with
  16 dp between them, fit the list's width; otherwise rows.
- **Labels:** each label stays beside its value when both, measured, and the
  gap fit the field's width; otherwise the value goes below the label.
- **Choices:** the overtime answers stand beside the label when the label and
  one line of answers fit, on one line below it when only the answers fit, and
  one per line otherwise.

With the sample data the interactive page shows, a long employer name
(Northwind Community Health Clinic, Inc.) and the three overtime answers, these
rules give:

| Window | Width class | Panes | List | Editor form | Not beside their labels |
| --- | --- | --- | --- | --- | --- |
| Phone, 390 dp | compact | one | rows, when shown alone | 1 column | Employer below; overtime one per line |
| Small tablet, 600 dp | medium | one (under 736 dp) | table, when shown alone | 1 column | overtime on one line below |
| Tablet, 834 dp | medium | two, 360 + 458 dp | rows: the table needs about 420 of 336 dp | 1 column | overtime one per line |
| Laptop, 1180 dp | expanded | two, 466 + 698 dp | table | 1 column | none: every value and the answers beside |
| Desktop, 1440 dp | large | two, 570 + 854 dp | table | 2 columns; employer and overtime span both | none |
| Web/TV, 1920 dp | extra-large | two, 762 + 1142 dp | table | 3 columns; employer spans 2, overtime 3 | none |

More columns make each field narrower. Without the two `span` rules, the
employer value and the overtime answers would move below their labels again at
1440 and 1920 dp; the spans keep the wide rows wide. The measured widths (the table's 420 dp, the employer
row) come from the fonts of the browser that drew them; another font can move
a threshold by a few dp. The page always applies the rules to what it measures.

The interactive page computes these decisions with the same rules Studio uses,
at normal text size, measuring its own text in your browser's font. Larger
text raises the minimums, so layouts change at wider windows. Its screens are
conceptual drawings, not product screenshots: the 12 dp padding inside each
pane and the look of each control are the drawing's own, and each client draws the
same decisions in its native style.

## Where the rules live

- [Width-based layout](../reference/presentation.md#width-based-layout), with the
  [pane threshold](../reference/presentation.md#pane-threshold),
  [form-grid columns](../reference/presentation.md#form-grid-columns) and
  [table column fit](../reference/presentation.md#table-column-fit).
- [Master-detail panes](../reference/presentation.md#master-detail-panes) and the
  [Display Template file](../reference/presentation.md#display-template-file).
- [Money](../reference/presentation.md#money) and the
  [Rust declaration syntax](../reference/languages/rust.md).

[Concepts](index.md)
