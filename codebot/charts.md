# Declaring a chart

Load this card when a task adds or changes chart metadata on a Response or
SubscriptionData payload: a mark, channels, scales, guides, or the way a
chart-producing request appears in the endpoint shell. It holds the
two-dimensional rules and the rules every chart shares; a 3D chart also loads
the [3D chart card](charts_3d.md) after this one. Its rules are generated from
the [2D chart specification](../reference/charts/2d.md), the
[chart summary](../reference/presentation.md#charts), the
[2D chart annotation rows](../reference/annotations.md#two-dimensional-chart-annotations)
and the [endpoint presentation contract](../reference/endpoint-presentation.md),
which own them; the
[schema chart inventory](../reference/schema.md#44-chart-presentation-attributes)
summarizes them. Chart appearance hints (`scale`, `guide`, `palette`, `chart`)
are template members on the [Display Template card](display_template.md).

## Order of work

1. Put the chart on output data: a unit List of row Tables, a carrier Table,
   a composition or a Union wrapper (`chart.data.*`, `chart.compose.*`,
   `annot.chart.*`), returning a complete chart-ready snapshot computed by the
   App (`chart.snapshot.*`, `chart.exclusion.*`).
2. Choose the mark by geometry and bind direct scalar row fields to channels
   (`chart.mark.*`, `chart.channel.*`, `annot.chart-channel.*`,
   `chart.geometry.*`, `chart.order.*`, `chart.row.*`).
3. Give every field its semantic type, unit and format (`chart.type.*`,
   `chart.unit.*`, `chart.format.*`), and keep every text value typed
   (`chart.type.source-*`, `chart.l10n.*`).
4. Declare scales only to override or share them (`chart.scale.*`,
   `annot.chart-scale.*`, `chart.domain.*`, `chart.lane.*`,
   `chart.shared-scale.*`, `money.shared-scales-match`), and guides only for an
   explicit override or a value/span label (`chart.guide.*`,
   `annot.chart-guide.*`, `chart.guide-label.*`, `chart.legend.*`,
   `chart.explicit-guides`).
5. Leave appearance to the template (`chart.appearance.*`, `chart.palette.*`,
   `chart.appearance-in-template`) and keep the structured view meaningful
   (`chart.structured-view-meaningful`).
6. Design interaction, fallback and limits around the client contract
   (`chart.interaction.*`, `chart.fallback.*`, `chart.limits.*`,
   `chart.a11y.*`, `chart.attr.*`, `chart.example.*`).
7. Know how the endpoint presents chart-producing requests and their previews
   (`endpoint.presentation.*`, `endpoint.workspace.*`, `chart.preview.*`).
8. Compile with the Studio MCP `compile_schema` tool and fix every reported
   rule.

<!-- generated:codebot-rules task=charts -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### 1. Purpose

Owner: [2d.md#1-purpose](../reference/charts/2d.md#1-purpose)

- `chart.data.app-computes`: The App task remains responsible for all domain calculations—including arbitrary time transforms—and can return an arbitrarily transformed but strictly typed payload. The client is responsible only for scales, layout, primitive marks, guides, accessibility, and standard chart interaction.
- `chart.data.scope-limits`: These attributes target **broad chart-family coverage for server-prepared data**. They do not provide Vega-Lite syntax compatibility, pixel compatibility, or complete feature parity. In particular, they do not reproduce Vega's runtime language, signals, arbitrary event expressions, client-side transforms, geographic coordinate/projection/topology semantics, raster/image marks, or a general widget system. A server-projected geographic region is ordinary polygon geometry and carries no client-side geographic meaning.

### 3.1 Chart metadata belongs to returned data

Owner: [2d.md#31-chart-metadata-belongs-to-returned-data](../reference/charts/2d.md#31-chart-metadata-belongs-to-returned-data)

- `chart.data.hidden-row-allowed`: A row field may carry `Hidden=true`. It is chart data, not presentation: the client keeps it in the retained row value, so a custom chart request can CopyFrom it.
- `chart.data.carrier-context`: Hidden carrier fields may use any otherwise valid Libertas type and remain ordinary encoded response data. They do not become rows, channels, guides, titles, tooltips, accessibility content, interaction identities, or chart resource usage. Their primary purpose is request/response transaction context analogous to an HTTP cookie: the client retains the complete original response tree so a later typed `NextRequest` or `CopyFrom` operation can copy the context back without displaying it or requiring the App to duplicate it in every chart row.
- `chart.data.title-from-variant`: For a protocol chart, the selected `Response` or `SubscriptionData` variant's localized name and description supply the overall chart title and description, including when its chart payload is a child field. Unwrapping the payload does not replace that wording with the child field's documentation. Charts nested inside a composition retain their own localized names and descriptions.

### 3.2 Complete snapshots

Owner: [2d.md#32-complete-snapshots](../reference/charts/2d.md#32-complete-snapshots)

- `chart.snapshot.replace`: Every response, and every complete subscription snapshot, replaces the previous chart snapshot. A subscription report may instead update the working value through the shared SubscriptionData mechanisms: an accepted [append report](../reference/protocols.md#live-chart-use) extends the chart-bearing working array, and a [reduced SubscriptionData](../reference/protocols.md#reduced-subscriptiondata-design-pattern) report updates the stable value it copies to. The chart presents the complete resulting value. The client does not otherwise join, patch, or merge chart rows. An empty list is a valid empty chart when the schema is valid.
- `chart.snapshot.request-new-snapshot`: If a chart must react to a user choice, the client sends a typed Libertas request and the endpoint returns another complete snapshot. Filtering, aggregation, binning, sorting, stacking, window calculation, regression, projection, and sampling therefore remain ordinary App code.

### 4.1 Primitive marks

Owner: [2d.md#41-primitive-marks](../reference/charts/2d.md#41-primitive-marks)

- `chart.mark.shared-channels`: `row`, `column`, `tooltip`, `detail`, `order`, and `key` are available to every mark under their ordinary facet, grouping, ordering, and interaction rules. Cartesian offsets remain subject to the discrete parent-band rules in §5.3. Appearance applicability is closed:

  | Mark | Allowed appearance channels |
  | --- | --- |
  | `point` | `color,fill,stroke,opacity,fillOpacity,strokeOpacity,size,shape,angle,strokeWidth,strokeDash` |
  | `line` | `color,stroke,opacity,strokeOpacity,strokeWidth,strokeDash` |
  | `area`, `polygon`, `bar`, `rect`, `arc` | `color,fill,stroke,opacity,fillOpacity,strokeOpacity,strokeWidth,strokeDash` |
  | `rule` | `color,stroke,opacity,strokeOpacity,strokeWidth,strokeDash` |
  | `text` | `color,fill,opacity,fillOpacity,size,angle` |

- `chart.mark.tick-as-rule` (judgment): Tick-like glyphs use an explicit short `rule`, so their length is portable data-space geometry rather than a platform-dependent size. Point, line, rule, and text also have the polar coordinate forms defined in Section 5.3. `polygon` supplies the closed filled-path primitive; `line` remains open and `arc` remains the only primitive whose edges are circular rather than straight screen-space segments.
- `chart.mark.family-histogram` (judgment): histogram: server-computed bins rendered as `bar` with `x,x2,y`;
- `chart.mark.family-stacked` (judgment): stacked bar or area: server-computed bounds rendered with `y,y2` or `x,x2`;
- `chart.mark.family-box-plot` (judgment): box plot: layered `rect` and `rule`;
- `chart.mark.family-candlestick` (judgment): candlestick: layered `rect` and `rule`;
- `chart.mark.family-ohlc` (judgment): OHLC: layered `rule` marks;
- `chart.mark.family-error-band` (judgment): error bar or confidence band: layered `rule` or `area` and `line`;
- `chart.mark.family-map-network` (judgment): map, network, or Sankey: server-computed coordinates rendered with primitive polygons, paths, rules, areas, points, and text.

### 4.2 Compositions

Owner: [2d.md#42-compositions](../reference/charts/2d.md#42-compositions)

- `chart.compose.layer-order`:

  | Value | Meaning |
  | --- | --- |
  | `layer` | Draw every child in schema order in one plot area. Later children are visually above earlier children. |

- `chart.compose.facets-and-grids` (judgment): Faceting is expressed by `row` and `column` channels on a unit chart. A repeat operation is unnecessary: the endpoint emits the explicit chart children or facet values it wants the client to display. A grid is represented by nested `hconcat` and `vconcat` compositions, avoiding a separate column-count grammar.

### 5. `ChartChannel`

Owner: [2d.md#5-chartchannel](../reference/charts/2d.md#5-chartchannel)

- `chart.channel.nearest-complete`: The nearest `ChartChannel` declaration is complete; channel values are not merged through a reference chain.

### 5.3 Cartesian and polar geometry

Owner: [2d.md#53-cartesian-and-polar-geometry](../reference/charts/2d.md#53-cartesian-and-polar-geometry)

- `chart.geometry.endpoint-order`: `x` and `y` locate the primary point. `x2` and `y2` are explicit data-space endpoints. Endpoint order is not semantic: clients render the interval between the two mapped values, so `open > close` and reversed ranges are valid. A zero-length rectangle dimension remains meaningful and is drawn one device pixel thick, so a doji or median remains visible.
- `chart.geometry.area-baseline`: without `y2`, a vertical `area` has a quantitative y baseline of zero;
- `chart.geometry.rect-no-baseline`: `rect` never invents a baseline;
- `chart.geometry.polar-zero`: angular zero is at twelve o'clock and positive angles advance clockwise;
- `chart.geometry.theta-turns`: a linear theta domain maps its minimum to zero turns and maximum to one full turn; an identity theta value is already measured in turns;
- `chart.geometry.radius-normalized`: an identity radius is normalized from `0` at the plot center to `1` at the available plot radius; a quantitative radius scale maps its domain into the same normalized interval;
- `chart.geometry.polar-range`: mapped theta and radius values must be finite and lie in the inclusive interval `0...1`. An out-of-range mapped coordinate is invalid; it is not wrapped or clamped to the circular boundary. An invalid point, rule, or text row is suppressed; an invalid line vertex breaks its path; and an invalid polygon vertex becomes a subpath separator after scale domains are resolved. Its finite raw positions still contribute to automatic domain resolution, but it contributes no geometry, appearance domain, group-style comparison, hit target, or accessibility focus. Tooltip and accessibility output for valid geometry retain the raw source value, not its normalized coordinate;
- `chart.geometry.straight-segments`: `point`, `line`, `rule`, `text`, and `polygon` use straight screen-space segments after the conversion above. A provider samples any spiral, circular edge, geodesic, or other curve it needs. Only `arc` has intrinsically circular edges;
- `chart.geometry.arc-angular-order`: before output-range reversal, `theta2` must not precede `theta`, and one arc cannot exceed one full turn;
- `chart.geometry.arc-default-radius`: omitting both radial channels means inner radius `0` and outer radius `1`;
- `chart.geometry.arc-radial-order`: before output-range reversal, `radius` is the inner source endpoint and `radius2` is the outer source endpoint.
- `chart.geometry.arc-mapped-interval`: After both endpoints are mapped through their shared scale, painted arc geometry and hit testing use the interval between the two mapped endpoints. `reverse=true` may therefore place the mapped primary endpoint above its secondary endpoint without invalidating the row: the physical angular or radial interval is mirrored, while source roles and tooltip values remain unchanged. Both mapped endpoint intervals must remain within `0...1`; polar geometry is not wrapped or clipped across an angular boundary. A source ordering or mapped-range violation suppresses that arc row as invalid geometry; it is not repaired by reordering the source values.
- `chart.geometry.polar-closure` (judgment): For connected polar vertices, mapped theta `0` and `1` are physically coincident. Polygon closure is implicit, so authors normally use an explicit theta domain such as `0...1`, emit vertices in the half-open interval `0...1`, and do not repeat the first vertex. Repeating it is nevertheless well-defined: it remains a counted source vertex and creates a zero-length edge. An automatic theta domain still maps its finite minimum to `0` and maximum to `1`; the client does not invent a cyclic category or an unobserved angular endpoint.

### 5.4 Default order and grouping

Owner: [2d.md#54-default-order-and-grouping](../reference/charts/2d.md#54-default-order-and-grouping)

- `chart.order.grouping`: List order is the default path and semantic paint order. An `order` channel overrides it. The sole presentation-only exception is the synthetic-overlap z-order rule in [Cartesian and polar geometry](../reference/charts/2d.md#53-cartesian-and-polar-geometry), which never reverses two rows whose authored raw intervals overlap. For line, area, and polygon marks, rows are first partitioned by facet values and then by the composite `detail` key. A `color`, `stroke`, or `fill` category does not implicitly become a grouping key; the same field should also declare `detail` when separate paths or polygons are required:

  ```typescript
  enum TemperatureSeries {
    Indoor,
    Outdoor,
  }

  /** @LibertasChartChannel(color, detail, tooltip) */
  series: TemperatureSeries;
  ```

- `chart.order.polygon-subpaths`: Within one polygon facet/detail group, source order first determines subpaths. A row with a null or non-finite required position is a source separator. After scale domains are resolved, a polar row whose finite required positions map to non-finite values or outside `0...1` is a mapped separator. Either kind ends the current subpath and begins a new one at the next validly mapped vertex. Leading, trailing, and consecutive separators do not create empty geometry. If `order` is present, the client then sorts the validly mapped vertices ascending **within each resulting subpath**, retaining source order for equal values; sorting never moves a vertex across a separator. For polygon, `order` affects only vertices within a subpath. Polygon groups paint in order of their first validly mapped source row and are not reordered by one member's `order` value.
- `chart.order.polygon-evenodd`: Each contiguous run of at least three finite mapped vertices is closed by an implicit last-to-first edge. A shorter run produces no polygon geometry and a bounded diagnostic. All valid subpaths in one facet/detail group form one compound polygon. The fill rule is always `evenOdd`: a test point is filled when a ray from it crosses the compound boundary an odd number of times. Ring winding has no meaning, nested rings create holes by parity, and disjoint rings create islands. Self-intersections, coincident vertices, touching rings, and crossing subpaths are legal and use the same rule; clients neither repair them nor impose a platform-native simple-polygon restriction. This permits, among other valid data, a radar polygon whose zero values collapse several vertices at the center.
- `chart.order.polygon-edges`: Polygon edges are straight segments in mapped screen coordinates. Their implicit closing edges participate in stroke dashing, clipping, hit testing, and resource accounting. A repeated first vertex is permitted but unnecessary; it remains a source vertex and creates one zero-length edge rather than changing closure or fill semantics. A provider samples a curved boundary explicitly.
- `chart.row.key-identity`: The optional `key` channel identifies a logical row across complete snapshots. It supports stable accessibility focus and future transitions, but does not change snapshot replacement semantics.
- `chart.row.no-implicit-key`: Position, tooltip, and detail values are not implicit row keys. Any number of source rows may carry the same `x`, `y`, temporal bucket, or other scalar value; clients preserve those rows independently through projection, paint, tooltip, and accessibility lowering. This rule is type-independent: a repeated bucket may be `DateOnly`, `DateTime`, numeric, or categorical. Only an authored `key` asserts logical row identity, and its documented facet/detail scope still applies.
- `chart.order.enum-appearance-domain`: An Enumeration is a closed categorical declaration. For an automatic categorical `color`, `fill`, `stroke`, `shape`, or `strokeDash` scale, the domain contains every declared member in declaration order, including members that have no eligible row in the current snapshot. This keeps appearance slots and legends stable as sparse data appears or disappears; declared-but-absent members create no marks. The complete declared appearance domain is subject to the ordinary paint, shape, and dash category limits. Position, offset, and facet domains remain derived from eligible rows, so a declaration alone never creates an empty axis band, nested lane, or facet pane. Open categorical types likewise remain row-derived.

### 5.5 Type inference

Owner: [2d.md#55-type-inference](../reference/charts/2d.md#55-type-inference)

- `chart.type.datetime-local`:

  | Libertas type | Default chart interpretation |
  | --- | --- |
  | `DateTime` | Temporal data shown in local time unless `type=utc` is explicit. |

- `chart.type.string-row`:

  | Libertas type | Default chart interpretation |
  | --- | --- |
  | `String` | Genuine literal runtime/domain text, a nominal category whose value is itself literal data, or a literal appearance token. It is never a fixed program vocabulary or a copied object display name. |

- `chart.type.string-source-choice`: The `String` row is a semantic assertion about provenance, not permission to put untranslated program wording in a payload. Whenever a channel can expose a value as human-readable text—including position/facet labels, ordinal legends, `text`, `tooltip`, explicit value/span guides, and accessibility—the App must choose the source type as follows:
- `chart.type.source-enumeration`: a closed App-defined vocabulary uses an `Enumeration`;
- `chart.type.source-object`: an identity that is a Libertas system object remains `Device`, `VirtualDevice`, `User`, `Endpoint`, or `LanDevice`, so the ordinary client resolver supplies its display name;
- `chart.type.source-formatted`: App-authored dynamic or parameterized wording uses a formatted Bytes value and LMF1; and
- `chart.type.source-literal`: only genuine literal domain/user text, such as an externally supplied proper name or free-form annotation, remains a plain `String` and is displayed verbatim.
- `chart.type.string-runtime-token` (judgment): A small or repeated set of proper names does not become a program vocabulary, and a token does not become literal data merely because task code writes it at runtime. A plain String used only by `key`, or by an identity-scale appearance channel whose guide is necessarily absent, is not rendered as text and is outside this presentation-source rule. If that same field also carries `tooltip` or another text-emitting role, the rule applies.
- `chart.type.string-provenance-review`: The schema validator can enforce the effective scalar, channel, scale, and guide combinations, but schema JSON cannot prove whether task code obtained a String from a user/domain source or selected it from a fixed English token set. Declaring a visible plain String is therefore an author/provider conformance assertion that package review and response fixtures must audit.
- `chart.type.date-only-civil`: `DateTime` is an instant. An ordinary `Number` retains its declared quantitative semantics, including unit or duration metadata. `DateOnly=true` is the explicit exception: its integer is the decimal calendar tuple `yyyyMMdd`, not epoch time. Clients validate the Gregorian date and construct chart/calendar components directly, without local- or UTC-offset conversion.
- `chart.type.timeonly-range`: A `TimeOnly` payload is finite seconds from midnight in `0 <= value < 86400`. An out-of-range value is invalid typed input and never wraps modulo a day for geometry, grouping, labels, or tooltips.

### 5.5.1 Physical units

Owner: [2d.md#551-physical-units](../reference/charts/2d.md#551-physical-units)

- `chart.unit.identifier-meaning`: CLDR core identifiers include simple units such as `millimeter`, generated compound units such as `millimeter-per-day`, products such as `kilowatt-hour`, and powered units such as `square-meter`. These identifiers are not source field names, localized strings, symbols, legacy UCUM aliases, or App resource keys. Thus an author declares `millimeter`, while a supporting CLDR formatter may visibly produce `mm` or a localized long name. `percent` means that raw `50` is fifty percent and displays as `50%`; it never rescales `0.5` into fifty percent. A plain Number with `PhysicalUnit=second` is a measured quantity in seconds, while `TimeInterval` retains its separate duration semantics; the two attributes cannot coexist.

### 5.5.2 Scalar display formats

Owner: [2d.md#552-scalar-display-formats](../reference/charts/2d.md#552-scalar-display-formats)

- `chart.format.text-only`: A chart applies the effective `Format` whenever that field produces visible text: scale ticks, tooltips, text marks, value/span guide labels, selection overlays, and accessibility descriptions. It never parses the formatted text back into data. Domains, tick candidates, sorting, grouping, equality, focus keys, interpolation, scale transforms, and geometry always use the raw Number, `yyyyMMdd`, epoch-second, or seconds-from-midnight value. DateTime formatting uses the viewer's local timezone even when an explicitly `utc` scale uses UTC for geometry; the authored pattern controls presentation, not scale semantics.
- `chart.format.datetime-range`: A chart `DateTime` payload is an integral Unix second in the inclusive range `0..253402214399`, whose upper endpoint is 9999-12-30T23:59:59Z. This chart-specific bound deliberately reserves a full day before the end of civil year 9999 so conversion through any configured time-zone offset whose absolute value is less than 86400 seconds stays within the proleptic-Gregorian years 0001 through 9999. A nonintegral or out-of-range row value, explicit bound, unavailable time-zone offset, or conversion outside that civil range invalidates chart presentation with a bounded diagnostic; it is never clamped or delegated to a platform formatter's private range. Client-generated temporal ticks and transient DateTime viewport edges may be fractional epoch seconds; they are inert derived guide instants, never a DateTime payload, tooltip, key, or typed request value.
- `chart.type.retain-domain-identity`: An App must not flatten an `Enumeration` or Libertas object reference into a duplicate display `String` merely for a chart. Keeping the typed value lets the frontend apply the same localization and permission-aware identity resolution as every other Libertas view, without adding configuration burden to the end user.
- `chart.row.nullable-positions`: Nullable positional values create missing points. For line and area marks, a missing position breaks the path; it is not interpolated. For polygon, it separates subpaths as defined in Section 5.4. A missing optional appearance value uses the native mark default. Non-finite numbers receive the same mark-specific suppression or separator behavior and are reported through bounded diagnostics.

### 5.6 Appearance values

Owner: [2d.md#56-appearance-values](../reference/charts/2d.md#56-appearance-values)

- `chart.appearance.defaults`: Without an appearance channel, the client uses its accessible theme-native default. Categorical paint channels use a deterministic Libertas palette. Literal values use `ChartScale(type=identity)`; their implicit guide is none.
- `chart.appearance.color-default-paint`: `color` is the mark's default paint: it supplies stroke for `line` and `rule`, fill for `area`, `bar`, `rect`, `arc`, `text`, and `polygon`, and both fill and stroke for `point`. An explicit `fill` or `stroke` overrides `color` for that component. Overall `opacity` multiplies the color alpha and the applicable `fillOpacity` or `strokeOpacity`; missing opacity components default to `1`.
- `chart.appearance.explicit-stroke`: An explicit `stroke` paints the boundary of an area, bar, rect, arc, polygon, or point and the path of a line or rule. For area, bar, rect, arc, and polygon, the boundary is absent when no explicit stroke resolves; `strokeWidth`, `strokeDash`, and `strokeOpacity` alone never synthesize it. Text uses its fill paint for glyphs and has no outline in version one. Point shape, rotation, radius, fill, and boundary use the same final geometry in marks, legends, hit testing, and accessibility.
- `chart.appearance.polygon-fill`: A polygon always has a fill: an explicit `fill`, otherwise `color`, otherwise the accessible theme-native polygon fill. Its boundary stroke is absent unless an explicit `stroke` binding resolves to visible paint. `strokeWidth`, `strokeDash`, and `strokeOpacity` do not synthesize a stroke or a stroke hit region by themselves.
- `chart.appearance.polygon-group-style`: A polygon facet/detail group has one paint and stroke style, not a per-vertex gradient. For every group-level appearance binding—`color`, `fill`, `stroke`, `opacity`, `fillOpacity`, `strokeOpacity`, `strokeWidth`, and `strokeDash`—all finite, validly mapped vertices in all of that group's subpaths must carry the same typed canonical raw value, or all carry null and use the same mark default. Comparison uses raw identity, never a localized label, clamped scale output, or theme-adjusted paint. A mismatch invalidates the complete chart presentation and invokes the chart failure policy in §10.
- `chart.appearance.discrete-order`: For any discrete scale, Enumeration values use declaration order; Boolean uses `false,true`; Number and Libertas object references use raw numeric order; a discrete DateTime uses raw epoch-second order and a DateOnly value uses its validated chronological `yyyyMMdd` numeric order; String uses Unicode scalar-value order of its raw value; and formatted Bytes use canonical blob order. Resolved display names and formatted output never affect ordering. The same ordering applies to categorical axes, legends, facets, and nested bands. An **automatic categorical paint** domain is limited to ten values and maps in order to these stable sRGB base hues:

  ```text
  #4E79A7,#F28E2B,#E15759,#76B7B2,#59A14F,
  #EDC948,#B07AA1,#FF9DA7,#9C755F,#BAB0AC
  ```

- `chart.palette.identity-contrast` (judgment): An identity color is literal endpoint data and is never theme-adjusted. An author who chooses `ChartScale(type=identity)` owns its contrast on every supported theme.
- `chart.palette.ten-categories`: An endpoint needing more than ten automatic paint categories supplies a bounded identity-color field or uses facets; clients do not repeat colors and imply false equivalence. Positional bands, facet values, and textual legends are not subject to the paint limit; they remain subject to row, facet, and layout caps.
- `chart.palette.diverging`: With `mid`, it uses these blue-neutral-red stops with the neutral stop fixed at the midpoint:

  ```text
  #053061,#2166AC,#4393C3,#92C5DE,#D1E5F0,#F7F7F7,
  #FDDBC7,#F4A582,#D6604D,#B2182B,#67001F
  ```

- `chart.palette.custom-colors` (judgment): Authors needing different colors send validated identity values.
- `chart.appearance.ranges-and-limits`: Other automatic appearance ranges are also portable: point radius maps to `3..18` density-independent points, text size to `10..32`, stroke width to `1..8`, opacity to `0.15..1`, and angle to `0..360` clockwise degrees. Ordinal shape uses `circle,square,triangle,diamond,cross` in order, while ordinal stroke dash uses `solid,dash,dot,dash-dot`. Automatic shape and dash domains are therefore limited to five and four categories respectively. Identity values are used directly. Size must be finite and at least `0.5`, with no schema upper bound; stroke width is bounded to `0..32`, opacity to `0..1`, and angle to finite degrees.
- `chart.appearance.identity-color`:

  | Channel | Identity value |
  | --- | --- |
  | `color`, `fill`, `stroke` | sRGB `#RRGGBB` or `#RRGGBBAA`. |

- `chart.appearance.identity-shape`:

  | Channel | Identity value |
  | --- | --- |
  | `shape` | A supported token such as `circle`, `square`, `triangle`, `diamond`, or `cross`. |

- `chart.appearance.identity-dash`:

  | Channel | Identity value |
  | --- | --- |
  | `strokeDash` | `solid`, `dash`, `dot`, `dash-dot`, or a bounded space-separated positive-number pattern. |

- `chart.appearance.unsupported-literal`: Unsupported literal values invalidate the chart presentation, not the protocol payload. Arbitrary CSS, URLs, scripts, and platform-specific color objects are not accepted.

### 5.6.1 Display Template palette

Owner: [2d.md#561-display-template-palette](../reference/charts/2d.md#561-display-template-palette)

- `chart.palette.hint`: A [Display Template](../reference/presentation.md#behavior-hints) `palette` hint on a `Chart` or `Chart3D` root selects the discrete color list for that chart's automatic categorical and ordinal paint (`color`, `fill`, and `stroke`) and for the default colors of series that bind no paint channel:

  | Value | Discrete color list |
  | --- | --- |
  | `auto` | The specification default: the ten categorical base hues above. |
  | `categorical` | The same ten categorical base hues. |
  | `sequential` | The ten Viridis stops above, as entries `0` through `9`. |
  | `diverging` | The eleven blue-neutral-red stops above, as entries `0` through `10`. |

- `chart.palette.hint-scope`: Category `i` uses entry `i` of the selected list, the theme contrast adjustment applies to every list, and the ten-category paint limit applies for every value. The hint never changes continuous numeric paint, whose stops `mid` alone chooses, or identity colors.

### 5.7 Runtime row semantics

Owner: [2d.md#57-runtime-row-semantics](../reference/charts/2d.md#57-runtime-row-semantics)

- `chart.row.order-sort`: List order is stable source order for semantic paint order and hit-test ties. An `order` binding sorts ascending by the effective scalar's canonical ordering; equal values retain list order. The synthetic-overlap exception in [Cartesian and polar geometry](../reference/charts/2d.md#53-cartesian-and-polar-geometry) changes only screen z-order and the matching hit-test tie for otherwise-disjoint intervals; canonical order remains the owner of style fallback and every raw-overlap tie.
- `chart.row.unordered-list-order`: `Unordered` on a chart list does not change chart geometry. A line, area, or polygon without an `order` binding follows the received list order, as on any other list, and polygon separator rows split subpaths in that order. An App whose path must not depend on the emitted sequence declares `order`.
- `chart.row.null-required`: A null required position suppresses a point, bar, rect, rule, arc, or text row. For line and area, a null required position breaks the path. For polygon it is a subpath separator under Section 5.4. A non-finite required position has the same mark-specific behavior and produces a bounded diagnostic; it never reaches a native drawing API. Null optional style values use the mark default. Tooltip fields are shown in structural field order and null tooltip values are omitted.
- `chart.row.key-unique`: A non-null `key` must be unique within the tuple of chart unit, facet values, and `detail` values. A duplicate key makes the runtime chart invalid and invokes the chart failure policy in §10 rather than ambiguous interaction. Without `key`, row index is identity only for the current complete snapshot.

### 6. `ChartScale`

Owner: [2d.md#6-chartscale](../reference/charts/2d.md#6-chartscale)

- `chart.scale.optional`: `ChartScale` is optional. Native automatic scales are used when it is absent. Its `type` reaches clients through the [Display Template](../reference/presentation.md#display-template) [as described above](../reference/charts/2d.md#22-generated-language-neutral-schema-attributes). Its generated `Value` is a comma-separated sequence of `key=value` properties in this canonical order:

  ```text
  id,type,min,max,mid,zero,reverse
  ```

### 6.1 Properties

Owner: [2d.md#61-properties](../reference/charts/2d.md#61-properties)

- `chart.scale.nearest-replaces`: Because lookup uses nearest-value precedence, a local `ChartScale` replaces the complete inherited or referenced scale value; individual scale properties are never merged across declarations.
- `chart.scale.time-vs-utc`: For `DateTime`, `time` formats and groups an epoch value in the device's configured local time zone and `utc` uses UTC. For `DateOnly`, both scale tokens operate on the same timezone-free calendar components. `identity` is intended for literal appearance values and normalized polar geometry; it must not bypass bounds or type validation.
- `chart.scale.explicit-domain-clips`: Explicit bounds and automatic-domain inference remain raw data values and are validated before transformation. Values outside an explicit domain are clipped by the plot; a client does not clamp or rewrite their source values. When `lo == hi`, that one transformed value maps to the range midpoint and the scale-sourced guide exposes one raw-value tick; clients do not invent different padding domains around it.
- `chart.scale.raw-values-exposed`: Tooltips, accessibility, explicit bounds, and request values always expose raw values, never transformed coordinates. The logarithm base does not change a normalized position; natural logarithms above define the portable inverse.
- `chart.scale.discrete-reverse`: For ordinal, band, and point scales, `reverse=true` reverses the output range, not the canonical raw-identity domain. Axis/facet placement and automatic appearance ranges therefore run in the opposite direction while grouping, ordering, keys, labels, and tooltip identity remain unchanged.

### 6.2 Automatic domains

Owner: [2d.md#62-automatic-domains](../reference/charts/2d.md#62-automatic-domains)

- `chart.domain.default-number`:

  | Field and channel | Default scale |
  | --- | --- |
  | `Number` on a quantitative channel | `linear` |

- `chart.domain.default-datetime`:

  | Field and channel | Default scale |
  | --- | --- |
  | `DateTime` on a position | `time` |

- `chart.domain.default-date-only`:

  | Field and channel | Default scale |
  | --- | --- |
  | `Number` with `DateOnly` on a position | `time`, formatted as a timezone-free calendar date |

- `chart.domain.default-time-only`:

  | Field and channel | Default scale |
  | --- | --- |
  | `TimeOnly` on a position | `linear`, formatted as local clock time |

- `chart.domain.default-time-interval`:

  | Field and channel | Default scale |
  | --- | --- |
  | `Number` with `TimeInterval` | `linear`, formatted as a duration |

- `chart.domain.default-discrete-band`:

  | Field and channel | Default scale |
  | --- | --- |
  | Discrete value on `bar`, `rect`, `row`, `column`, or an offset | `band` |

- `chart.domain.default-discrete-point`:

  | Field and channel | Default scale |
  | --- | --- |
  | Discrete value on another position | `point` |

- `chart.domain.default-discrete-ordinal`:

  | Field and channel | Default scale |
  | --- | --- |
  | Discrete value on paint, shape, or stroke dash | `ordinal` |

- `chart.domain.default-numeric-paint`:

  | Field and channel | Default scale |
  | --- | --- |
  | Numeric paint | `linear` sequential, or divergent when `mid` is present |

- `chart.domain.datetime-utc-choice` (judgment): A `DateTime` author uses `type=utc` when UTC formatting is semantically required; otherwise device local time is the default. An explicit `band` or `point` instead requests a discrete, sparse DateTime axis and therefore cannot simultaneously request the `utc` scale. Endpoints should choose bucket instants whose local labels are unambiguous for their audience.

### Normative broken-time interval-lane policy

Owner: [2d.md#normative-broken-time-interval-lane-policy](../reference/charts/2d.md#normative-broken-time-interval-lane-policy)

- `chart.lane.when-to-use` (judgment): This pattern is for sparse, interval-like quantities whose visual comparison is more useful than a calendar-proportional gap. It makes occupied buckets legible without pretending that empty calendar time is measured. It is not a time series, event timeline, duration chart, or a client-side stacking feature.
- `chart.lane.provider-owns-geometry`: The provider owns every geometric decision. Its row schema uses the generic guide attribute explicitly:

  ```rust
  #[libertas_chart_channel(x)]
  #[libertas_chart_scale(id = usage_display, kind = linear)]
  display_start: f64,
  #[libertas_chart_channel(x2)]
  display_end: f64,
  #[libertas_date_only]
  #[libertas_chart_channel(tooltip)]
  #[libertas_chart_guide(
      target = x,
      source = span,
      position = bottom,
      title = none,
      grid = none
  )]
  bucket_starts_on: u32,
  #[libertas_chart_channel(tooltip)]
  amount: f32,
  ```

- `chart.lane.display-coordinates`: `display_start,display_end` must be finite numeric coordinates and every valid positive contribution must have `display_end > display_start`. They are explicitly display coordinates: neither field is an event time, and an automatic time guide must be absent. The provider supplies the real bucket date and exact quantity in tooltip fields. Those tooltip values, not the synthetic coordinate, are the factual date and amount presented to a person, copied to accessibility output, and used by details or drill-in actions. A bucket value is never implicit row identity: every same-bucket source contribution remains a distinct row unless the author provides an explicit `key` channel.
- `chart.lane.anchor-grouping`: For a response, group all valid rows by the exact real bucket anchor across all lanes. Sort those anchors ascending, start the first display anchor at zero, and give every lane in a group that same anchor. Use one response-wide quantity denominator. For each lane, allocate the ordered positive contributors from cumulative quantity boundaries, preserving contiguity and reserving a whole synthetic unit for every remaining contributor. The group extent is the largest resulting rounded lane extent. Thus rounding cannot make another lane overlap or cause its next group to relocate.
- `chart.lane.fixed-gap-parameters`: For the sprinkler V1 contract, the full-scale extent is exactly 600 synthetic seconds. Let `previous_extent` be the preceding real bucket's group extent and let `delta` be the difference between consecutive real bucket anchors. The next display anchor is `previous_anchor + previous_extent + delta` when `0 <= delta < 60`, and `previous_anchor + previous_extent + 30` when `delta >= 60`. Equal anchors are one group, not a zero-gap second group. The 30-unit collapsed gap is five percent of 600 and leaves at least four integer display units for one-unit positive-contributor floors. A later contract version may choose different response-wide fixed parameters, but it must document them as a provider contract; clients must never infer or normalize a gap locally.
- `chart.lane.provider-rounding`: The provider uses stable contributor ordering (for sprinkler: rain, irrigation, forecast rain, scheduled water), a deterministic sort by real bucket anchor, and cumulative-boundary rounding rather than independently rounded segment widths. It omits nonfinite or nonpositive inputs. If no valid positive interval can be produced for a lane, it returns the ordinary empty-lane annotation rather than a zero-width mark or fabricated value. Saturating coordinate arithmetic must preserve order where representable; a provider that cannot emit a finite, strictly positive interval omits that invalid row rather than wrapping or relocating unrelated rows.
- `chart.lane.provider-mapping`: The provider owns this precomputed visual-coordinate contract. It must use one consistent chart-wide mapping and emit a finite, nondegenerate `x,x2` interval for every positive segment; a clipped edge, rounding step, or empty contributor must not corrupt unrelated geometry. The client maps the supplied endpoints literally. It does not aggregate, normalize, stack, facet, or repair these rows. Responsive renderers may guarantee a minimum visible pixel width for a valid positive interval; that presentation aid is not a substitute for valid provider geometry and does not change the data-space mapping. The explicit span guide replaces the misleading numeric axis for synthetic coordinates. This uses the generic `ChartGuide` attribute and adds no special bucket, histogram, stack, or timeline grammar.

### 6.2.2 Domain contributors and baselines

Owner: [2d.md#622-domain-contributors-and-baselines](../reference/charts/2d.md#622-domain-contributors-and-baselines)

- `chart.domain.automatic-zero`: Automatic continuous domains cover the finite visible data. One-sided quantitative `bar` and area baselines include zero by default. Line, point, explicit area bands, rule, rect, polygon, and ranged bar marks fit their explicit data by default. `zero` overrides that choice when the scale type permits it. If exactly one explicit bound lies beyond every automatic domain contributor, or there are no contributors, the unresolved side collapses to that explicit bound. The result is the degenerate one-tick domain defined above; clients do not ignore the bound or invent platform-specific padding merely because all supplied marks are clipped.
- `chart.domain.guide-visibility`: A continuous scale-sourced guide is visible when its resolved shared scale has at least one eligible domain contributor anywhere under the selected chart root, or when the author supplied a domain anchor (`min`, `max`, paint `mid`, or `zero=true`). Thus an empty first binding can still own the guide for data contributed by a later binding with the same scale ID, and an explicitly bounded empty snapshot retains its truthful authored guide. A fully automatic scale with no contributors owns only an internal transform placeholder and shows no axis or legend. A categorical scale-sourced guide likewise requires at least one resolved category. A member of a closed Enumeration appearance domain is resolved even when the current snapshot has no row for that member; positional and facet guides remain eligible-row-driven.
- `chart.domain.shared-zero`: For a shared scale whose bindings all omit `zero`, the automatic domain is the union of every contributing binding's domain contribution, so any one-sided bar or baseline area contributes zero. An explicit shared `zero=false` disables that automatic contribution for the complete shared scale; `zero=true` forces it. This rule does not depend on composition traversal order.
- `chart.domain.number-bounds-not-domain`: `NumberMin` and `NumberMax` remain input/data validation constraints. They do not silently become chart domain bounds. Authors use `ChartScale(min=...,max=...)` when a stable visual domain is required.

### 6.3 Shared and independent scales

Owner: [2d.md#63-shared-and-independent-scales](../reference/charts/2d.md#63-shared-and-independent-scales)

- `chart.shared-scale.scope`: A scale `id` is scoped to the outermost rendered chart root. Compatible bindings with the same ID share their **domain and scale configuration**, never their physical pixel range. Each unit chart maps that shared domain into its own plot rectangle. Thus vertically concatenated price and volume charts can align in time without sharing screen coordinates.
- `chart.shared-scale.unnamed-local`: An unnamed scale is always local to one unit chart. There is no implicit cross-layer sharing based on coincidental field names. Different IDs create independent scales, which supports multi-scale layers without a separate Vega `resolve` grammar; §6.4 defines their deterministic guide layout.
- `chart.shared-scale.endpoint-domain`: domain inference includes the primary and secondary endpoint values from every binding carrying that ID;
- `chart.shared-scale.property-fill`: declarations may repeat identical `min`, `max`, `mid`, `zero`, and `reverse` properties, while one of those omitted properties is filled from another binding or inferred automatically;
- `chart.shared-scale.continuous-union`: continuous domain contributions union across all bindings, including the automatic baseline rule above;
- `chart.shared-scale.paired-inherit`: Paired endpoint fields inherit their primary field's scale and do not repeat the ID. This is how a candlestick's wick and body share time and price domains. The body still has its own pixel geometry, and an optional volume subplot uses an independent y-scale while sharing only the time ID.

### 6.4 `ChartGuide`: generic guides

Owner: [2d.md#64-chartguide-generic-guides](../reference/charts/2d.md#64-chartguide-generic-guides)

- `chart.guide.attribute`: `ChartGuide` is the fourth chart attribute. It separates guide presentation from `ChartScale` domain and transform semantics. Its `position` reaches clients through the [Display Template](../reference/presentation.md#display-template). Its generated value contains properties in this canonical order:

  ```text
  target,source,position,title,domain,ticks,labels,grid
  ```

### 6.4.1 Placement and validation

Owner: [2d.md#641-placement-and-validation](../reference/charts/2d.md#641-placement-and-validation)

- `chart.guide.auto-defaults.authoring`: Set guide position and components only to override the [inferred guide defaults](../reference/charts/2d.md#641-placement-and-validation).
- `chart.guide.kinds`: The inferred scale guide is an axis for Cartesian or polar positions, a nested categorical label tier for an offset, a facet header for `row` or `column`, and a legend for an appearance role. There are no bucket-date, radar, map, or other chart-family guide types.

### 6.4.2 Ownership, layers, facets, and composition

Owner: [2d.md#642-ownership-layers-facets-and-composition](../reference/charts/2d.md#642-ownership-layers-facets-and-composition)

- `chart.guide.label-candidates-local`: An explicit `source=value` or `source=span` guide takes candidates only from the declaring unit's rows; sharing its target scale with another layer does not join row data across units. Its output positions nevertheless use the complete resolved shared domain.
- `chart.guide.title-owner`: A resolved guide takes its title policy and localized title source only from that owner. For `source=scale` the source is the owner's target-binding field; for `source=value` or `source=span` it is the guide-label declaration field. Documents on other shared-scale contributors are ignored. Matching or differing rendered title text never changes scale sharing or chart validity.

### 6.4.5 Scale ticks, labels, and typed formatting

Owner: [2d.md#645-scale-ticks-labels-and-typed-formatting](../reference/charts/2d.md#645-scale-ticks-labels-and-typed-formatting)

- `chart.guide-label.text-sources`: Guide text never comes from a `ChartGuide` property: its grammar contains no literal or format string. On the exact owner field defined in §6.4.2, a title resolves `ChartGuideTitle`, then the localized document `Name`; `title=none` omits it. Boolean values use the client's localized true/false resources while raw `false,true` remain identity and order.
- `chart.guide-label.authored-format`: A nonempty authored `Format` on the exact guide owner is authoritative.
- `chart.guide-label.typed-resolution`: Enumeration labels use their member documents. FormattedText Bytes use their declared resource catalog and arguments; an unresolved or malformed essential formatted value falls back the chart rather than exposing its resource key or argument diagnostics. `Device`, `VirtualDevice`, `User`, `Endpoint`, and `LanDevice` use the ordinary permission-aware system object-name resolver. Resolution never changes raw identity, grouping, ordering, or scale mapping. An unavailable object uses the resolver's existing localized unavailable-name result; if that contract supplies no safe display value for an essential label, the chart falls back instead of exposing a raw ID. Fixed program tokens must be Enumeration or FormattedText. A value that identifies a Libertas system object must retain its typed object reference and use the ordinary resolver; it must not copy the resolved or source-language name into a String. A plain String is visible only when it is genuine runtime/domain/user text and is displayed literally. Generic words, overflow indicators, and accessibility phrases come exclusively from client resources. Native names, scale IDs, channel tokens, and raw object IDs are never synthetic user-visible fallbacks.

### 6.4.6 Value and span guide sources

Owner: [2d.md#646-value-and-span-guide-sources](../reference/charts/2d.md#646-value-and-span-guide-sources)

- `chart.guide-label.guide-only-rows`: Another required mark position may be null, so an App task can emit a guide-only row that produces no painted data mark. The row nevertheless keeps its ordinary path meaning: it breaks a line or area run or separates a polygon subpath when the missing role ordinarily does so. This is the explicit generic path for task-computed ticks or intervals after an arbitrary transformation; clients never have to infer the transformation from the label.
- `chart.guide-label.span-grouping.authoring`: Choose `source=span` for connected intervals with the same typed label, or `source=scale` for bounded empty axes; [value and span guides need eligible rows](../reference/charts/2d.md#646-value-and-span-guide-sources).
- `chart.guide-label.tooltip-isolation`: This is the complete generic rule for sparse or task-transformed interval labels. A tooltip field can affect a guide only when it explicitly carries `ChartGuide`; adding an unrelated tooltip can never change guide semantics.

### 6.4.7 Legend lowering and layout

Owner: [2d.md#647-legend-lowering-and-layout](../reference/charts/2d.md#647-legend-lowering-and-layout)

- `chart.legend.guides-inert`: Guides are semantically inert. They do not compete in mark hit testing, own a tooltip or source row, trigger a typed request, or become a selection. A value or span label remains guide-inert even though rows supplied it; authors use a `text` mark when the label must be interactive.
- `chart.legend.guide-resource-limits`: Guide projection is bounded before native allocation: at most `256` resolved guide instances under one chart root, `4096` semantic candidates for one guide, `16384` guide entries in total, and `32768` guide primitives in total. A title, label, tick, grid line/spoke/circle, domain stroke, or legend sample is one guide primitive. These counts are separate from the data-mark primitive budget; responsive thinning may reduce native objects but cannot make an over-limit semantic guide valid. Text retains the existing 4096-byte per-value limit.

### 6.4.8 Deterministic responsive guide overlay

Owner: [2d.md#648-deterministic-responsive-guide-overlay](../reference/charts/2d.md#648-deterministic-responsive-guide-overlay)

- `chart.preview.height.authoring`: Choose Display Template `chart.aspect` and `max_height` to override the [responsive preview defaults](../reference/charts/2d.md#preview-chart-height).

### 7. Why four attribute families are sufficient

Owner: [2d.md#7-why-four-attribute-families-are-sufficient](../reference/charts/2d.md#7-why-four-attribute-families-are-sufficient)

- `chart.attr.time-projection`: Time is not a privileged client-side transform. An App task may emit a native DateTime/DateOnly position for the ordinary automatic temporal scale, or it may project time into any bounded numeric coordinate its domain requires. In the latter case, a typed DateTime, DateOnly, Enumeration, FormattedText, or Libertas object-reference field can independently supply `source=value` or `source=span` guide labels, including on guide-only rows whose unrelated mark geometry is null. The client therefore localizes truthful typed labels without reverse-engineering calendar meaning from synthetic geometry. This ability to execute arbitrary application code before returning one typed, chart-ready snapshot is a defining Libertas boundary, not a missing client transform language.

### 8. Attribute resolution and references

Owner: [2d.md#8-attribute-resolution-and-references](../reference/charts/2d.md#8-attribute-resolution-and-references)

- `chart.attr.nearest-complete`: Chart metadata uses the shared [attribute resolution](../reference/schema.md#attribute-resolution) rule. The nearest declaration supplies the complete same-named attribute.
- `chart.channels.override-complete-set`: It does not union values from several declarations. Therefore a local override must contain the complete channel set:

  ```text
  referenced type: ChartChannel=tooltip
  use site:        ChartChannel=x,tooltip   correct
  use site:        ChartChannel=x           tooltip is intentionally replaced
  ```

- `chart.attr.names-compose`: Different attribute names compose normally. A local `ChartChannel` can use a referenced type's `ChartScale` or `ChartGuide` when no nearer same-named attribute replaces it. A local `ChartGuide` is always a complete declaration; its properties never merge with an inherited guide.
- `chart.attr.foreign-preserved`: Foreign compatible types preserve official chart attributes unless the local schema supplies a valid explicit override. Both language parsers must extract the same local declarations and references; Studio's shared validator applies the portable placement, inheritance, canonicalization, and override rules.

### 8.1 Localization and schema-document dependency

Owner: [2d.md#81-localization-and-schema-document-dependency](../reference/charts/2d.md#81-localization-and-schema-document-dependency)

- `chart.l10n.enum-documents-required`:

  | Existing localized source | Chart use | Required? |
  | --- | --- | --- |
  | Enumeration member `Name` and `Description` | Fixed category label and help | Required only when that Enumeration is visibly encoded. |

- `chart.l10n.string-resources-runtime-keys`:

  | Existing localized source | Chart use | Required? |
  | --- | --- | --- |
  | Package document `StringResources` | Render a `FormattedText` Bytes tuple with Libertas Message Format 1 (LMF1) | Used when supplied; schema validation checks the declared package keys but does not prove every runtime tuple key. |

- `chart.l10n.custom-property-not-token-map`:

  | Existing localized source | Chart use | Required? |
  | --- | --- | --- |
  | Flattened custom document property | Static App-specific chart wording | Never required by the minimum grammar and never a runtime-token map. |

- `chart.l10n.optional-custom-properties`: The minimum chart introduces no mandatory custom document keys. Existing `DisplayName`, long description, field names/descriptions, and Enumeration documents are sufficient. An App may optionally provide localized flattened properties such as `ChartGuideTitle`, `ChartTooltipName`, `ChartEmpty`, or `ChartAccessibilitySummary` when the established text is not appropriate. The corresponding ordinary name/description or a built-in localized client resource remains the fallback; these properties never affect chart validation. They are static wording attached to the documented schema node. A client must never use them as a dictionary for arbitrary plain String payload values.
- `chart.l10n.formatted-runtime-text`: Formatted runtime text has a separate, closed path. The App schema declares one package-owned `StringResources` catalog, and its package document contains the localized LMF1 templates. The emitting task already identifies that package; the tuple therefore carries no type owner or resource FQN. Schema validation checks declared keys and default-document membership but does not prove every runtime tuple key. At chart presentation time, an unresolved or malformed optional formatted value is omitted and an essential one invokes the chart failure policy in §10 for the complete chart. Resource keys, tuple bytes, and typed argument diagnostics may appear in bounded developer diagnostics or the ordinary data view's established fallback, but never as chart presentation text.
- `chart.text.formatted-bytes`: Formatted chart values never use a String containing Base64. Encoding only to decode again at render time provides no benefit, while the Bytes host creates a sharp distinction from literal text and makes authoring mistakes less likely.
- `chart.text.lmf-format-complete`: The LMF1 number or date style is the complete formatting instruction for that message argument; field-level `Format` and chart scale formatting do not override it. A locale, translated template, number style, or resolved object or Enumeration label may change text measurement and therefore label collision, guide reservation, legend flow, or chart presentation failure. It cannot change the data-derived plot geometry, domain membership, category ordering, shared scale compatibility, or interaction identity.
- `chart.text.no-runtime-template`: The payload never contains a template. The raw key/blob remains the category, grouping, ordering, and interaction identity. Two values that render to the same text therefore remain distinct.
- `chart.l10n.static-wording-source`: static App-authored chart wording comes from the resolved document or an optional custom document property;
- `chart.l10n.typed-labels`: localized Enumeration text comes from Enumeration documents;
- `chart.l10n.dynamic-formatted`: App-authored dynamic presentation uses `FormattedText`; the App package's `StringResources` catalog supplies localization;
- `chart.l10n.string-never-resource-key`: a plain String payload is always literal data, never an implicit resource key; a fixed localizable program token must instead be an Enumeration or formatted Bytes output, and a Libertas object name must instead come from its typed object reference and normal resolver.
- `chart.l10n.uniform-scope`: These rules apply uniformly to every primitive and server-projected family, including direct labels, heatmaps, bubbles, networks, and Sankey-style flows. Changing geometry does not create an exception for presentation provenance. Only `key`-only Strings and guide-less identity appearance Strings are non-textual. A String that also participates in a tooltip, label, guide, legend, facet, or accessibility distinction is user-visible and must satisfy the literal-data assertion above.
- `chart.l10n.bubble-product`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Bubble portfolio `product` | Plain String only when it is the product's proper name; use an Enumeration for a fixed product class. |

- `chart.l10n.heatmap-time-block`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Activity heatmap `time_block` | Enumeration for fixed blocks, or formatted Bytes for parameterized wording. |

- `chart.l10n.direct-label`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Direct-label `label` | Enumeration, formatted Bytes, typed Libertas object reference, or genuine literal domain/user text according to provenance. |

- `chart.l10n.network-node-connection`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Network `node` / `connection` | Typed Libertas object references when they identify system objects; otherwise Enumeration for a closed relationship vocabulary or formatted Bytes for dynamic wording. |

- `chart.l10n.sankey-stage`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Sankey `stage` | Enumeration for fixed stages, or formatted Bytes for parameterized wording. |

- `chart.l10n.identity-and-key-strings`:

  | Chart family and field | Required semantic source |
  | --- | --- |
  | Identity-scale colors and key-only identifiers | Plain String is allowed only while the field has no text-emitting role or guide. |

- `chart.l10n.enum-label-not-flattened`: An enum-backed traffic-source arc label already satisfies this rule because the client resolves its member document; it must not be flattened into a String.

### 9. Interaction model

Owner: [2d.md#9-interaction-model](../reference/charts/2d.md#9-interaction-model)

- `chart.interaction.categories-preserved`: a Cartesian physical plot preserves every authored category on positional `band` and `point` scales. It never samples, aggregates, drops, or replaces a category. Only the App task may intentionally return already-aggregated category rows, which are then ordinary authored chart data;
- `chart.interaction.data-change-new-snapshot`: a data-changing action sends a protocol request and replaces the chart with the returned snapshot.

### 10. Validation and fallback

Owner: [2d.md#10-validation-and-fallback](../reference/charts/2d.md#10-validation-and-fallback)

- `chart.text.review-literal-provenance`: Package conformance review and response-fixture tests additionally validate the provenance assertion that cannot be derived from schema JSON: every visible plain String is genuine literal domain/user text. Fixed program vocabularies use Enumeration or FormattedText, and Libertas object labels use their typed system-object reference. This audit covers every role that can emit text and explicitly excludes only key-only and guide-less identity appearance Strings.

### 11. Detailed examples

Owner: [2d.md#11-detailed-examples](../reference/charts/2d.md#11-detailed-examples)

- `chart.example.document-chart-fields` (judgment): The examples use the implemented TypeScript or Rust source syntax. The first cross-language example shows ordinary short and long documentation. Later examples omit most comments for geometric clarity, but their native field names never stand in for user-visible labels. Production packages supply localized names and descriptions through the existing document pipeline; optional labels are omitted and essential category failures follow §8.1 and the chart failure policy in §10.

### 11.7 Candlestick chart

Owner: [2d.md#117-candlestick-chart](../reference/charts/2d.md#117-candlestick-chart)

- `chart.example.layers-not-joined`: Both layers repeat all desired OHLCV tooltip fields because the client does not join rows across layers, even when their `key` values match. The key preserves conceptual identity and focus stability only. Both literal paint fields declare an identity scale and are not rendered as text. The fixed `direction` wording is an Enumeration whose localized member label is used by the tooltip. The single-pixel zero-height rule from the geometry contract keeps an `open == close` doji visible.

### 11.10 Server-projected maps, networks, and flows

Owner: [2d.md#1110-server-projected-maps-networks-and-flows](../reference/charts/2d.md#1110-server-projected-maps-networks-and-flows)

- `chart.example.server-projected-geometry` (judgment): The minimum grammar deliberately has no geographic projection, topology, force layout, contour generation, or Sankey transform. The endpoint returns projected and sampled geometry that the primitives can express:
- `chart.example.projected-region-polygon`: a filled projected region or choropleth region as a polygon detail group, using null-position separators for islands and holes and repeating one canonical raw fill value on all of its vertices;
- `chart.example.projected-text-source`: Projected geometry does not relax the presentation-source rule. The field-by-field audit in §8.1 is normative for bubble `product`, activity `time_block`, direct `label`, network `node` and `connection`, Sankey `stage`, and traffic-source values. Apps must not copy an object's current display name or emit fixed English tokens in a visible plain String. Reusing one of these row schemas in another response, such as an operations dashboard reusing the activity heatmap, does not create an exception.
- `chart.example.app-owns-projection`: This provides useful projected-map and flow/network charts without embedding a geospatial, contour, or graph runtime in every client. The App remains responsible for projection, antimeridian decisions, topology/ring extraction, force simulation, contouring, and path sampling. A client does not interpret geographic coordinates or topology merely because a polygon happens to depict a map.

### 11.11 Scatter and bubble chart

Owner: [2d.md#1111-scatter-and-bubble-chart](../reference/charts/2d.md#1111-scatter-and-bubble-chart)

- `chart.example.literal-point-radius`: If the endpoint supplies a literal point radius in density-independent points, it uses `type=identity` and must stay within the client resource limit.

### 11.13 Filled radar chart

Owner: [2d.md#1113-filled-radar-chart](../reference/charts/2d.md#1113-filled-radar-chart)

- `chart.example.radar-app-normalizes` (judgment): A radar chart is not a special mark. The App assigns every indicator an explicit turn, normalizes that indicator's value against its own documented bound, and emits one polar polygon detail group per series. The original value and indicator remain tooltip data. The client does not infer indicator maxima or evenly space categories.

### 12. Security, resource, and accessibility requirements

Owner: [2d.md#12-security-resource-and-accessibility-requirements](../reference/charts/2d.md#12-security-resource-and-accessibility-requirements)

- `chart.limits.structural`: impose the same version-one structural and metadata limits on both platforms: composition depth `8`, unit charts `64`, facets `64`, resolved guides `256`, candidates per guide `4096`, total guide entries `16384`, total guide primitives `32768`, tooltip fields per row `32`, and UTF-8 bytes in one text, guide label, or tooltip value `4096`;
- `chart.limits.no-row-ceiling`: impose no chart-specific numeric ceiling on source rows, eligible source primitives, or vertices in a source path. The enclosing ScriptValue transport and ordinary-data model already own payload admission; chart presentation must not reject the same admitted snapshot because it crosses an unrelated conservative geometry count;
- `chart.a11y.color-not-only-carrier` (judgment): require any meaning encoded through paint to also be exposed through a localized `tooltip`, `detail`, text, shape, or equivalent accessibility value, so color is not the only carrier of state;
- `chart.limits.depth-and-unit-counting`: the outer selected chart root has composition depth `0`; each composition edge adds one, so a unit reached through at most eight composition edges is permitted. Every mark-bearing List leaf is one unit chart; composition containers do not consume the `64` unit-chart budget;
- `chart.limits.facet-counting`: a facet grid contains the Cartesian product of its resolved `row` and `column` domains, with a missing dimension contributing a factor of one. The `64` limit is the sum of panes produced by faceted plot nodes under the outer chart root. Sibling units in one `layer` count one grid only when their ordered raw row and column domains are identical; otherwise each distinct grid contributes its own product.

### 14. Deliberate exclusions and future evolution

Owner: [2d.md#14-deliberate-exclusions-and-future-evolution](../reference/charts/2d.md#14-deliberate-exclusions-and-future-evolution)

- `chart.exclusion.client-transforms`: client-side transform and expression languages;
- `chart.exclusion.option-blobs`: arbitrary JSON option blobs;
- `chart.exclusion.client-joins`: client joins, lookups, aggregation, binning, or stack inference;
- `chart.exclusion.field-paths`: arbitrary field paths;
- `chart.exclusion.url-html-marks`: raw URL or HTML marks;
- `chart.exclusion.client-geo`: client-side geographic projection, topology/ring extraction, force layout, contouring, curve interpolation, or path sampling;
- `chart.exclusion.signals`: Vega signals and general event streams;
- `chart.exclusion.brush-state`: persistent crossfilter or brush state;
- `chart.exclusion.patches`: chart-specific incremental patch formats;
- `chart.exclusion.special-marks`: special marks whose geometry can be composed from primitives.
- `chart.exclusion.emit-rows-first` (judgment): New capability should first be tested against the core question: can the App server emit explicit rows and primitive layers instead? If yes, no schema extension is needed. A future attribute is justified only when it represents portable client interaction or presentation that cannot be supplied safely as typed chart-ready data.

### Charts

Owner: [presentation.md#charts](../reference/presentation.md#charts)

- `chart.appearance-in-template`: Chart appearance belongs to the [Display Template](../reference/presentation.md#display-template). Studio validates the complete chart metadata on the parser result, then compiles the scale `type` of each `ChartScale` and `Chart3DScale` into a `scale` hint and the `position` of each `ChartGuide` into a `guide` hint, and removes those properties from the final schema; an attribute left with no property is removed. Channels, scale IDs, domains, and every other chart property remain schema. A template rule may also choose a chart `palette`. Clients read the effective scale type, guide position, and palette only from the template, and a template value is validated under the same chart matrices as the source.
- `chart.explicit-guides` (judgment): Use `@LibertasChartGuide(target=..., source=...)` or `#[libertas_chart_guide(target = ..., source = ...)]` only for an explicit guide. Targets are `x`, `y`, `xOffset`, `yOffset`, `theta`, `radius`, `color`, `fill`, `stroke`, `opacity`, `fillOpacity`, `strokeOpacity`, `size`, `shape`, `angle`, `strokeWidth`, `strokeDash`, `row`, and `column`. Sources are `scale`, `value`, `span`, or `none`; `value` is limited to `x`, `y`, `theta`, or `radius`, while `span` is limited to continuous Cartesian `x/x2` or `y/y2`. The required properties are `target` and `source`. Optional properties are `position`, `title`, `domain`, `ticks`, `labels`, and `grid`; positions are `auto`, `top`, `right`, `bottom`, or `left`, and component switches are `auto` or `none`. Prefer inferred scale guides unless the App needs an explicit override.

### 3D charts

Owner: [presentation.md#3d-charts](../reference/presentation.md#3d-charts)

- `chart.structured-view-meaningful` (judgment): Always keep the ordinary structured data view meaningful. Charts are an additional presentation, not the only representation of returned data.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### 2.1 Source-language annotations

Owner: [2d.md#21-source-language-annotations](../reference/charts/2d.md#21-source-language-annotations)

- `chart.data.kind-not-type`: Source annotations use `kind`; the generated scale property is `type`

### 3.1 Chart metadata belongs to returned data

Owner: [2d.md#31-chart-metadata-belongs-to-returned-data](../reference/charts/2d.md#31-chart-metadata-belongs-to-returned-data)

- `chart.data.response-payload-only`: Chart attributes go on a response or subscription-data payload type
- `chart.data.unit-shape`: A unit chart is a List of row Tables; the List carries the mark
- `chart.data.hidden-row-field`: A Hidden row field carries no ChartChannel, ChartScale or ChartGuide
- `chart.data.carrier-table`: A carrier Table has exactly one non-hidden List field; every other field is Hidden
- `chart.compose.shape`: A composition Table holds at least one chart child; every other child is Hidden
- `chart.data.variant-wrapper`: A chart-wrapping Union variant has exactly one chart field; every other field is Hidden

### 3.3 Direct row fields only

Owner: [2d.md#33-direct-row-fields-only](../reference/charts/2d.md#33-direct-row-fields-only)

- `chart.data.direct-row-fields`: Chart channels go only on direct scalar fields of the chart row

### 4. `Chart`

Owner: [2d.md#4-chart](../reference/charts/2d.md#4-chart)

- `chart.mark.one-token`: Chart has exactly one lowercase token as its generated Value

### 4.1 Primitive marks

Owner: [2d.md#41-primitive-marks](../reference/charts/2d.md#41-primitive-marks)

- `chart.mark.appearance-closed`: Each mark accepts only the appearance channels in its row of the table

### 4.2 Compositions

Owner: [2d.md#42-compositions](../reference/charts/2d.md#42-compositions)

- `chart.compose.layer-children`: A layer child is a unit chart or a layer, never a concat
- `chart.compose.layer-one-family`: A layer is entirely Cartesian or entirely polar

### 5. `ChartChannel`

Owner: [2d.md#5-chartchannel](../reference/charts/2d.md#5-chartchannel)

- `chart.channel.one-per-node`: At most one Chart, ChartChannel, ChartScale and ChartGuide per schema node

### 5.1 Channel vocabulary

Owner: [2d.md#51-channel-vocabulary](../reference/charts/2d.md#51-channel-vocabulary)

- `chart.channel.positional-roles`: ChartChannel roles are positional vocabulary tokens, never named properties

### 5.2 Multiplicity

Owner: [2d.md#52-multiplicity](../reference/charts/2d.md#52-multiplicity)

- `chart.channel.singletons`: Every role except tooltip and detail is a singleton; detail fields form one series key
- `chart.channel.scale-bearing`: A field carries at most one scale-bearing role; duplicate the value for a second scale
- `chart.channel.scale-needs-role`: No two fields share a singleton role; ChartScale needs a scale-bearing role
- `chart.channel.secondary-endpoints`: A secondary endpoint needs its primary, shares its scale and type, and carries no ChartScale

### 5.3 Cartesian and polar geometry

Owner: [2d.md#53-cartesian-and-polar-geometry](../reference/charts/2d.md#53-cartesian-and-polar-geometry)

- `chart.geometry.offsets`: xOffset requires a discrete x band and yOffset a discrete y band; each creates a nested categorical band
- `chart.geometry.point`: A point requires exactly one primary pair: Cartesian x,y or polar theta,radius
- `chart.geometry.line`: A line requires exactly one primary pair: Cartesian x,y or polar theta,radius
- `chart.geometry.area-vertical`: A vertical area uses x,y and optional y2
- `chart.geometry.area-horizontal`: A horizontal area uses y,x,x2; an area never uses both x2 and y2
- `chart.geometry.bar-one-sided`: A one-sided bar has one discrete axis and one quantitative Number axis from zero
- `chart.geometry.bar-rect-axes`: Each bar or rect axis is one discrete band field or an explicit interval
- `chart.geometry.bar-x-interval`: bar(x,x2,y) needs continuous x and either a discrete y band or a Number histogram y
- `chart.geometry.bar-y-interval`: bar(x,y,y2) needs a discrete x band; with x2 and y2 both primaries are continuous
- `chart.geometry.no-log-baseline`: No log scale on a one-sided bar, a histogram y, or a baseline area y
- `chart.geometry.rule-cartesian`: A Cartesian rule needs x,y plus x2 and/or y2; no plot-spanning endpoint is implied
- `chart.geometry.rule-polar`: A polar rule needs theta,radius plus theta2 and/or radius2
- `chart.geometry.text`: A text mark requires exactly one of x,y,text or theta,radius,text
- `chart.geometry.polygon`: A polygon uses x,y or theta,radius with no endpoints, offsets, text, size, shape or angle
- `chart.geometry.one-family`: One mark never mixes Cartesian and polar positions; area, bar and rect are Cartesian, arc polar
- `chart.geometry.arc-theta`: An arc requires theta and theta2; the server supplies explicit angular bounds
- `chart.geometry.arc-radius-pair`: An arc uses both radius and radius2 or neither

### 5.5 Type inference

Owner: [2d.md#55-type-inference](../reference/charts/2d.md#55-type-inference)

- `chart.type.discrete-opt-in`: Discrete Number scales need an integral non-duration Number; DateTime may use band or point

### 5.5.1 Physical units

Owner: [2d.md#551-physical-units](../reference/charts/2d.md#551-physical-units)

- `chart.unit.continuous-compatible`: Unit-bearing chart fields use continuous scales; paired and shared bindings need the same unit

### 5.5.2 Scalar display formats

Owner: [2d.md#552-scalar-display-formats](../reference/charts/2d.md#552-scalar-display-formats)

- `chart.type.no-other-scalars`: Other semantic types, tables, lists, unions, nil and plain Bytes are not chart scalars
- `chart.type.compatibility-table`: Each channel role accepts only the scalar types in the compatibility table

### 5.6.1 Display Template palette

Owner: [2d.md#561-display-template-palette](../reference/charts/2d.md#561-display-template-palette)

- `chart.palette.hint-validation`: A palette rule targets a Chart or Chart3D root with one of the four tokens

### 6. `ChartScale`

Owner: [2d.md#6-chartscale](../reference/charts/2d.md#6-chartscale)

- `chart.scale.grammar-closed`: The ChartScale grammar is closed: id, type, min, max, mid, zero and reverse only
- `chart.scale.scalar-syntax`: ChartScale properties appear once; scalars are finite grammar numbers without leading zeros
- `chart.scale.scalar-units`: Scale scalars use runtime units: epoch seconds, seconds of day, duration seconds, yyyyMMdd

### 6.1 Properties

Owner: [2d.md#61-properties](../reference/charts/2d.md#61-properties)

- `chart.scale.id-identifier`: A scale id is a schema identifier
- `chart.scale.type-tokens`: Scale type is one of linear, log, symlog, sqrt, time, utc, ordinal, band, point, identity
- `chart.scale.invalid-is-error`: Repeated or unknown ChartScale keys and invalid combinations are errors
- `chart.scale.type-matrix`: The scale-type matrix is closed; only integral non-duration Numbers become categories
- `chart.scale.matrix.xy-number`: Numeric x and y use linear, log, symlog or sqrt; integral Numbers may use band or point
- `chart.scale.matrix.xy-temporal`: DateTime and DateOnly x and y use time, utc, band or point
- `chart.scale.matrix.xy-time-only`: TimeOnly x and y use linear
- `chart.scale.matrix.xy-categorical`: Categorical and FormattedText x and y use band or point
- `chart.scale.matrix.offset-facet`: Offsets and facets take discrete values on a band scale
- `chart.scale.matrix.theta-radius`: `theta` and `radius` take linear, log, symlog, sqrt or identity scales
- `chart.scale.matrix.paint-categorical`: Categorical paint uses ordinal; a plain String may use identity
- `chart.scale.band-extent`: Only scale-bearing primaries own scales; bar and rect discrete dimensions use band
- `chart.scale.time-interval`: A TimeInterval scale is linear, log, symlog or sqrt, never discrete or identity
- `chart.scale.min-max-order`: Scale min must not exceed max
- `chart.scale.mid-in-domain`: mid belongs to a continuous divergent scale and lies within explicit bounds
- `chart.scale.log-positive`: A log scale and every value the App supplies to it are strictly positive
- `chart.scale.sqrt-non-negative`: A sqrt scale and every value the App supplies to it are non-negative
- `chart.scale.zero-quantitative`: zero only on quantitative continuous non-log scales
- `chart.scale.identity-literal`: An identity scale declares no domain properties; its values are already portable literals
- `chart.scale.zero-not-temporal`: Temporal, categorical and identity scales cannot use zero
- `chart.scale.band-point-positional`: band and point only on discrete positional or facet channels; on DateTime they make sparse categories
- `chart.scale.temporal-types`: time and utc need DateTime or DateOnly; log, symlog and sqrt need an ordinary Number
- `chart.scale.bounds-continuous`: min and max need a continuous scale; mid needs a continuous quantitative paint scale
- `chart.scale.secondary-no-scale`: A secondary endpoint channel cannot declare a scale
- `chart.scale.date-only-bounds`: DateOnly min and max canonicalize to exact valid yyyyMMdd dates

### 6.2 Automatic domains

Owner: [2d.md#62-automatic-domains](../reference/charts/2d.md#62-automatic-domains)

- `chart.domain.time-only`: TimeOnly row values lie in half-open 0..86400; bounds in inclusive 0..86400

### 6.3 Shared and independent scales

Owner: [2d.md#63-shared-and-independent-scales](../reference/charts/2d.md#63-shared-and-independent-scales)

- `chart.shared-scale.compatible-bindings`: Bindings sharing a scale ID agree on channel family, scalar type, presentation class, Enumeration, Money, PhysicalUnit and Format
- `chart.shared-scale.range-class`: Point-radius size and text-font size cannot share a scale ID
- `chart.shared-scale.conflict-invalid`: Conflicting properties on a shared scale ID invalidate the chart

### 6.4 `ChartGuide`: generic guides

Owner: [2d.md#64-chartguide-generic-guides](../reference/charts/2d.md#64-chartguide-generic-guides)

- `chart.guide.grammar-closed`: The ChartGuide grammar is closed: target, source, position, title, domain, ticks, labels, grid
- `chart.guide.required-keys`: ChartGuide keys appear once; source tokens equal the generated keys

### 6.4.1 Placement and validation

Owner: [2d.md#641-placement-and-validation](../reference/charts/2d.md#641-placement-and-validation)

- `chart.guide.scale-source-placement`: source=scale or none goes on the field whose primary scale-bearing role is the target
- `chart.guide.label-field-placement`: value and span guides sit on a scale-free label field; span needs continuous x or y with x2 or y2
- `chart.guide.matrix.cartesian-axis`: Cartesian axes: x top or bottom, y left or right
- `chart.guide.matrix.polar-axis`: Polar axes take only position auto
- `chart.guide.matrix.offset-tier`: Offset tiers: xOffset top or bottom, yOffset left or right
- `chart.guide.matrix.facet-header`: Facet headers take only title and labels; row left or right, column top or bottom
- `chart.guide.matrix.value-rail`: Value label rails use their Cartesian axis side matrix
- `chart.guide.matrix.polar-value`: Polar value labels take only position auto
- `chart.guide.matrix.span-rail`: Span label rails use their Cartesian axis side matrix
- `chart.guide.matrix.suppressed`: A suppressed guide declares no position or components
- `chart.guide.outside-row-error`: A guide property outside its target row is an error
- `chart.guide.implicit-explicit`: One explicit ChartGuide per target per unit replaces the implicit guide; identity and offsets default to none

### 6.4.2 Ownership, layers, facets, and composition

Owner: [2d.md#642-ownership-layers-facets-and-composition](../reference/charts/2d.md#642-ownership-layers-facets-and-composition)

- `chart.guide.shared-ownership`: A shared scale ID in one layered plot owns one guide with at most one explicit declaration
- `chart.guide.polar-one-per-family`: A polar layer shows at most one theta and one radius guide

### Money

Owner: [presentation.md#money](../reference/presentation.md#money)

- `money.shared-scales-match`: Shared chart scales and paired interval endpoints require matching Money metadata

<!-- /generated -->
