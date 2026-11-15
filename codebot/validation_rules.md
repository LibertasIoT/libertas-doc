# ValidationRules quick reference

Syntax lookup for the [validation rules card](validation.md), which quotes every
rule from the [Data Validation Rules specification](../reference/validation-rules.md).
Path syntax and resolution are owned by
[Libertas Schema Path](../reference/schema.md#libertas-schema-path), whose
rules (`paths.*`) are on the [data model card](data_model.md); the
examples below apply its validation-expression forms. Debug rules with Studio's
developer trace, `evaluate_validation_rules` and CI fixtures under
[Debugging rules](../reference/validation-rules.md#debugging-rules).

## Choose a rule or a bound

Decide first [where a check belongs](validation.md#where-a-check-belongs).

| Need | Attribute value |
| --- | --- |
| Accept or reject data | `ValidationRules`: a Boolean expression followed by `: "MESSAGE_KEY";` ([rule set](validation.md#3-rule-set)). |
| Compute a minimum or maximum | `RangeMin` / `RangeMax`: one bound expression ([dynamic ranges](validation.md#31-dynamic-numeric-ranges)). |
| Require a fixed String format | `Pattern`: one portable full-string pattern ([Pattern](validation.md#pattern-attribute-and-lifecycle)). |

## Declare

| Form | TypeScript | Rust |
| --- | --- | --- |
| Rule set | `@LibertasValidationRules('expression : "MESSAGE_KEY";')` | `#[libertas_validation_rules(r#"expression : "MESSAGE_KEY";"#)]` |
| Final-only rule set (discouraged) | `@LibertasValidationRules('FinalOnly=true; expression : "KEY";')` | `#[libertas_validation_rules(r#"FinalOnly=true; expression : "KEY";"#)]` |
| Dynamic bound | `@LibertasRangeMin("expression")` / `@LibertasRangeMax("expression")` | `#[libertas_range_min("expression")]` / `#[libertas_range_max("expression")]` |

## Read values in the correct context

| Purpose | Syntax |
| --- | --- |
| Annotated value / descendant | `[]`, `[field/child]` |
| Parent's field (current row) | `[^/minimum]` |
| Function argument | `[argument]` |
| Reference-context path(s) | `{$.value}`, `{$.preferred,$.fallback}` |

## Operators and functions

| Purpose | Syntax |
| --- | --- |
| First non-null value | `[preferred] \| [fallback] \| 0` |
| Logic and comparison | `!`, `&&`, `||`, `==`, `!=`, `<`, `<=`, `>`, `>=` |
| Active Union variant or Enumeration member | `[value] is VariantName` |
| One flag set in a Bitflags field | `[facts] has PersonFacts.Student` |
| String length in UTF-8 bytes | `length([name])` |
| Complete String format | `matches([identifier], "[0-9]{9}")` |
| Fixed calendar date | `d"1990-01-01"` |
| DateOnly year, month, day (integers) | `year([issued])`, `month([issued])`, `day([issued])` |
| Shift a DateOnly by months or years | `add_months([start], 24)`, `add_years([start], 1)` |
| FormattedText resource key (locale independent) | `resource([status]) == "STATUS_NONE"` |

## Traverse collections

| Purpose | Syntax |
| --- | --- |
| List cursor | `all(item in [items], [item/value] >= 0)` |
| Paths across parent lists | `all(item in [groups/items], [item/value] >= 0)` |
| Filter and project | `collect(node in [nodes], [node] is Active, [node/device])` |
| Collection uniqueness | `is_unique_set(collection)` |
| Composite-key uniqueness | `is_unique_set(row in [rows], [row/credit], [row/year])` |
| Total of Number leaves | `sum([rows/amount])`, `sum(collect(...))` |
| Lists and collections | `all`, `any`, `none`, `collect`, `is_unique_set`, `count`, `sum` |

Combine operators under the full
[precedence table](../reference/validation-rules.md#15-operator-precedence) and
the [grammar](../reference/validation-rules.md#14-expression-grammar).
