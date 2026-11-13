# Declaring a 3D chart

Load this card when a task adds or changes a three-dimensional chart: a
`Chart3D` mark or composition, `Chart3DChannel`, `Chart3DScale`,
`Chart3DGuide` or `Chart3DView`. Load the [chart card](charts.md) first: its
shared rules on output placement, snapshots, typed text, interaction and
fallback apply to 3D charts too. This card holds only the 3D rules, generated
from the [3D chart specification](../reference/charts/3d.md), the
[3D chart summary](../reference/presentation.md#3d-charts) and the
[3D chart annotation rows](../reference/annotations.md#three-dimensional-chart-annotations),
which own them.

## Order of work

1. Keep the 3D namespace separate from the 2D one and put the chart on output
   data (`chart3d.data.*`, `chart3d.boundary.*`, `chart3d.snapshot.*`,
   `annot.chart3d.*`).
2. Choose the primitive by geometry, never by family name (`chart3d.family.*`,
   `chart3d.mark.*`, `chart3d.bar.*`, `chart3d.line.*`, `chart3d.stem.*`), and
   for surfaces and wireframes supply the topology (`chart3d.uv.*`,
   `chart3d.topology.*`).
3. Bind direct scalar row fields to channels (`chart3d.channel.*`) and
   declare scales and guides only to override or share them
   (`chart3d.scale.*`, `chart3d.guide.*`).
4. Set the initial camera with one `Chart3DView` per physical scene, without
   inventing gesture modes (`chart3d.view.*`, `annot.chart3d-view.*`,
   `chart3d.no-gesture-modes`).
5. Keep visible text typed, respect the limits, and leave to the App whatever
   it can compute (`chart3d.l10n.*`, `chart3d.limits.*`,
   `chart3d.exclusion.*`).
6. Compile with the Studio MCP `compile_schema` tool and fix every reported
   rule.

<!-- generated:codebot-rules task=charts-3d -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### 1. Purpose and compatibility boundary

Owner: [3d.md#1-purpose-and-compatibility-boundary](../reference/charts/3d.md#1-purpose-and-compatibility-boundary)

- `chart3d.app-owns-calculation`: A Libertas protocol response or subscription-data payload may carry a server-prepared three-dimensional chart. The App remains responsible for every domain calculation, including sampling functions, calculating contours, projecting data, grouping series, and selecting topology. The client resolves scales, constructs the declared primitives, lays out guides, presents an accessible camera, and preserves the ordinary typed-data view.
- `chart3d.boundary.scale-id-namespace`: a `Chart3DScale.id` never resolves to a `ChartScale.id`, even when their spelling is identical;

### 3.1 Unit charts

Owner: [3d.md#31-unit-charts](../reference/charts/3d.md#31-unit-charts)

- `chart3d.data.hidden-row-allowed`: A row field may carry `Hidden=true`. It is chart data, not presentation: the client keeps it in the retained row value, so a custom chart request can CopyFrom it.

### 3.3 Protocol Union wrappers

Owner: [3d.md#33-protocol-union-wrappers](../reference/charts/3d.md#33-protocol-union-wrappers)

- `chart3d.data.title-from-variant`: The selected `Response` or `SubscriptionData` variant's localized name and description supply the overall chart title and description, even when the chart payload is a child field. Child charts within a composition retain their own localized names and descriptions.

### 3.4 Complete snapshots

Owner: [3d.md#34-complete-snapshots](../reference/charts/3d.md#34-complete-snapshots)

- `chart3d.snapshot.replace`: Every response, and every complete subscription snapshot, replaces the prior chart snapshot. A subscription report may instead extend or update the working value through the shared mechanisms described in the [2D snapshot rule](../reference/charts/2d.md#32-complete-snapshots). Clients do not otherwise join, patch, or merge rows, grids, series, or topology. An empty list is a valid empty chart when its schema is valid. A data-changing interaction sends an ordinary typed protocol request and receives another complete snapshot. Snapshot replacement does not imply presentation-camera reset; Section 8.2 defines camera continuity.

### 4.1 Primitive marks

Owner: [3d.md#41-primitive-marks](../reference/charts/3d.md#41-primitive-marks)

- `chart3d.mark.structural-roles`: `tooltip`, `detail`, `key`, `row`, and `column` are available to every mark. `order` is meaningful for `line` and for deterministic independent-mark paint and hit-test ties. Surface and wireframe topology is owned by `x,y` or `u,v`, not by list order; an `order` binding does not change mesh connectivity.

### Point and line

Owner: [3d.md#point-and-line](../reference/charts/3d.md#point-and-line)

- `chart3d.line.paths`: `point` and `line` use the mapped location `(x,y,z)`. A null or non-finite required coordinate suppresses a point and breaks a line path. Line rows are partitioned first by facet and then by the composite raw `detail` identity. List order is the default vertex order; an `order` binding sorts ascending within each path with source order retained for ties.
- `chart3d.point.axis-kinds`: Each of x, y, and z may independently be continuous, categorical, or an explicitly discrete numeric/temporal position. A categorical point axis uses a point scale unless explicitly banded; the renderer never coerces its localized label back into geometry.
- `chart3d.point.shape-tokens`: The first four are camera-facing symbols; the final four are plot-space 3D primitives. `circle` is the default. An automatic shape scale assigns categories in canonical raw-domain order, cycles through this list, and repeats from `circle` only after exhausting it. An identity shape must be one of these tokens; it is never a platform class, path, model, image, or URL. One client cannot substitute a native-only shape or change this assignment order.
- `chart3d.point.size`: `size` is a finite non-negative relative magnitude. Zero suppresses a point or stem-tip marker without removing its row from tooltips or accessibility. An automatic size scale preserves authored zero and chooses a responsive positive output range for positive values from the plot dimensions, available display area, and accessibility settings. The schema imposes no fixed projected-radius range. Numeric identity size is likewise finite and non-negative. For a camera-facing symbol it controls projected size; for a volumetric symbol it controls plot-space diameter and therefore participates in perspective. The resolved shape determines that interpretation consistently on every client. This permits both screen-stable Bubble 3D symbols and volume-calibrated solid symbols without making either model mandatory.

### Bar

Owner: [3d.md#bar](../reference/charts/3d.md#bar)

- `chart3d.bar.legacy-baseline`: The value axis may be x, y, or z. When its secondary is present, both endpoints are explicit and either order is valid. Otherwise the client uses the resolved scale base plane: zero when zero is representable and included in the resolved domain, or the lower resolved domain boundary otherwise. The sole continuous axis identifies the value axis, so this form may point along x, y, or z without another orientation property.
- `chart3d.bar.sparse-column`: Each base axis may be continuous or discrete. A band base starts with its full domain-slot bandwidth. An explicit point-scale base, or a continuous base without a secondary, derives a centered interval from adjacent distinct mapped positions in the same facet/detail group. Its available width is the smallest positive mapped gap to either neighbor, or the sole gap at an endpoint. An isolated position uses one tenth of the mapped axis length. Scale `padding` is then applied to that band, point, or inferred continuous interval. A continuous base with a matching secondary instead uses the authored interval, and padding has no effect on it. `z2` is the authored baseline. Without `z2`, the baseline is the same resolved scale base plane described above. Positive log domains, all-positive or all-negative linear domains, and temporal domains remain valid without `z2`; they use their lower resolved boundary.
- `chart3d.bar.size-shape`: `size` on a bar is a finite non-negative per-row multiplier applied to both base extents after explicit/inferred extent and padding resolution; zero produces a degenerate bar without removing its source row, and the multiplier never changes the value endpoint or baseline. Automatic shape scales use the portable bar-shape order `box,cylinder,pyramid,sphere`, cycling only after all four. Identity bar shapes use the same four-token catalog; the primitive is aligned between the baseline and value endpoint and scaled to the two resolved base extents. The default is `box`.

### 4.2 Appearance applicability

Owner: [3d.md#42-appearance-applicability](../reference/charts/3d.md#42-appearance-applicability)

- `chart3d.appearance.color-default`: `color` supplies the mark-default paint: point fill and boundary, line, wireframe, and stem stroke, or bar and surface fill. Explicit `fill` or `stroke` overrides that component. Stroke metadata alone never creates a boundary when neither `stroke` nor an applicable `color` resolves one.
- `chart3d.appearance.stem-marker`: For `stem`, `size` and `shape` describe a marker at the tip, while `fill` and `fillOpacity` describe that marker's interior. The stem segment remains the stroke component. A renderer with no native impulse-marker primitive constructs the same result from a segment and point marker; that lowering is not visible in schema or interaction semantics.

### 5. `Chart3DChannel`

Owner: [3d.md#5-chart3dchannel](../reference/charts/3d.md#5-chart3dchannel)

- `chart3d.channel.canonical-order`: The value is a comma-separated set of roles. Studio deduplicates and emits them in this order:

  ```text
  x,x2,y,y2,z,z2,u,v,
  color,fill,stroke,opacity,fillOpacity,strokeOpacity,
  size,shape,strokeWidth,strokeDash,
  tooltip,detail,order,key,row,column
  ```

  The vocabulary is:

  | Group | Roles | Meaning |
  | --- | --- | --- |
  | Position | `x,x2,y,y2,z,z2` | Three primary coordinates and paired endpoints. |
  | Surface topology | `u,v` | Parameter-grid adjacency; not visual scales. |
  | Paint | `color,fill,stroke` | Default paint or explicit component paint. |
  | Paint strength | `opacity,fillOpacity,strokeOpacity` | Overall or component alpha. |
  | Symbol/stroke | `size,shape,strokeWidth,strokeDash` | Point/bar geometry or path appearance. |
  | Inspection | `tooltip` | Values included in the typed inspector. |
  | Structure | `detail,order,key` | Series grouping, path/paint order, and stable row identity. |
  | Facet | `row,column` | Small-multiple partitioning. |

### 5.3 Identity, order, and missing values

Owner: [3d.md#53-identity-order-and-missing-values](../reference/charts/3d.md#53-identity-order-and-missing-values)

- `chart3d.identity.raw-values`: Grouping, uniqueness, category domains, series identity, and key identity use canonical raw typed values, never localized labels. Floating negative zero is canonicalized to positive zero for identity. Non-finite values cannot become keys or topology indices.
- `chart3d.identity.unique-keys`: A non-null `key` is unique within its unit, facet, and detail tuple. Duplicate keys invalidate 3D presentation. Without a key, a source row index is identity only for the current complete snapshot.
- `chart3d.identity.null-values`: Null or non-finite required point/bar/stem coordinates suppress that row. They break line paths. For surface and wireframe they remove the vertex and every incident face or edge. Null optional appearance uses the theme-native default. Null tooltips are omitted.

### 6. `Chart3DScale`

Owner: [3d.md#6-chart3dscale](../reference/charts/3d.md#6-chart3dscale)

- `chart3d.scale.extends-chart-scale`: `Chart3DScale` starts with the `ChartScale` property and type vocabulary, adds portable logarithm-base control, automatic-domain growth, and 3D bar-base `padding`, and participates only in a `Chart3D` tree:

  ```text
  property-order = id,type,logBase,min,max,mid,zero,reverse,
                   growMin,growMax,padding

  type = linear | log | symlog | sqrt | time | utc |
         ordinal | band | point | identity
  ```

- `chart3d.scale.infer-number`: ordinary Number and TimeOnly positions use `linear`;
- `chart3d.scale.infer-temporal`: DateOnly and DateTime positions use `time`;
- `chart3d.scale.infer-categorical`: categorical point/line/point-cloud positions use `point`;
- `chart3d.scale.infer-bar-base`: discrete bar bases use `band`, continuous sparse-column bases keep their continuous position scale;
- `chart3d.scale.infer-paint`: categorical paint and shape use `ordinal`; and
- `chart3d.scale.infer-numeric-appearance`: numeric paint, opacity, size, and stroke width use `linear`.

### 6.1 Shared scales

Owner: [3d.md#61-shared-scales](../reference/charts/3d.md#61-shared-scales)

- `chart3d.shared-scale.no-concat`: A same-spelled `ChartScale.id` is unrelated. A scale never crosses a concat pane. Compatible scales in one flattened layer may share one guide.

### 7.1 Placement

Owner: [3d.md#71-placement](../reference/charts/3d.md#71-placement)

- `chart3d.guide.value-candidates`: `value` for `x,y,z` creates candidates at the target coordinate of eligible rows.
- `chart3d.guide.detail-none`: `none` with `target=detail` suppresses an otherwise requested detail guide.

### 7.2 Components and inference

Owner: [3d.md#72-components-and-inference](../reference/charts/3d.md#72-components-and-inference)

- `chart3d.guide.inference`: Without an explicit declaration, every non-identity primary binding has `source=scale` with defaults. Identity appearance has `source=none`. One or more `detail` bindings infer one composite `source=value` series legend with default components. An explicit detail guide configures that legend or suppresses it with `source=none`; it never creates separate legends for individual fields of the composite detail tuple.

### 7.3 Position-guide cadence and thinning

Owner: [3d.md#73-position-guide-cadence-and-thinning](../reference/charts/3d.md#73-position-guide-cadence-and-thinning)

- `chart3d.cadence.tickstep-tickcount`: `tickStep` is an authored semantic cadence and therefore bypasses the automatic ladder. It generates only values aligned to that interval in the native convention defined above; it does not synthesize a non-aligned domain endpoint. `tickCount` cannot accompany `tickStep`. Without `tickStep`, `tickCount` only lowers the automatic tier's viewport-derived candidate capacity; it never asks for exactly that many values, invents evenly spaced samples, or replaces the required nondegenerate terminal endpoint pair. Thus `tickCount=1` does not turn a nondegenerate automatic guide into a one-label summary: if its essential terminal candidates cannot fit, ordinary guide fallback applies.

### 7.5 Layer ownership

Owner: [3d.md#75-layer-ownership](../reference/charts/3d.md#75-layer-ownership)

- `chart3d.guide.no-text`: Guide text comes from existing typed/localized sources. No `Chart3DGuide` property contains user-visible text.

### 8.2 Camera semantics

Owner: [3d.md#82-camera-semantics](../reference/charts/3d.md#82-camera-semantics)

- `chart3d.view.coordinates`: The data coordinate system is right-handed with positive z as up. After scale mapping, x, y, and z span a fitted plot box whose world dimensions have the normalized `aspectX:aspectY:aspectZ` ratio. The default remains a cube. Target coordinates use this normalized post-scale space: each axis cube runs from 0 to 1, the default `0.5,0.5,0.5` is its center, and finite targets outside the box are permitted.
- `chart3d.view.zoom-fit.authoring`: Choose `zoom=1` to fit the plot box and perspective `fov` for its vertical field of view under the [camera contract](../reference/charts/3d.md#82-camera-semantics).

### 9. Surface and wireframe topology

Owner: [3d.md#9-surface-and-wireframe-topology](../reference/charts/3d.md#9-surface-and-wireframe-topology)

- `chart3d.topology.groups`: Rows are partitioned first by facet and then by the composite raw `detail` identity. Each group independently defines one grid.

### 9.1 Rectangular height grid

Owner: [3d.md#91-rectangular-height-grid](../reference/charts/3d.md#91-rectangular-height-grid)

- `chart3d.grid.unique-pairs`: every present ordered x/y pair has at most one source row;
- `chart3d.grid.holes`: a missing pair is an explicit hole, while a duplicate pair invalidates the complete 3D chart; and
- `chart3d.grid.non-uniform`: x and y spacing may be non-uniform.
- `chart3d.grid.null-z`: This form represents an ordinary height map `z=f(x,y)` without copying x/y into separate index fields. A row with a null or non-finite z retains its unique grid slot but contributes no finite vertex, so its incident cells form a hole under Section 9.3.

### 9.2 Parameter grid

Owner: [3d.md#92-parameter-grid](../reference/charts/3d.md#92-parameter-grid)

- `chart3d.uv.numbers`: both are required finite ordinary Numbers (see [scalar compatibility](../reference/charts/3d.md#52-scalar-compatibility) and [surface requirements](../reference/charts/3d.md#41-primitive-marks));
- `chart3d.uv.domains`: each distinct raw domain is sorted by canonical numeric ordering and may be negative, fractional, non-uniform, or sparse;
- `chart3d.uv.unique`: `(u,v)` is unique within the facet/detail group;
- `chart3d.uv.positions`: x, y, and z remain arbitrary continuous positions and need not form a height function; and
- `chart3d.uv.holes`: a missing `(u,v)` pair creates a hole rather than causing neighbor inference.
- `chart3d.uv.app-computes`: This form represents non-separable, curvilinear, folded, or parameterized surfaces. The App computes the sample coordinates; `u,v` describe only adjacency.

### 10. Supported families

Owner: [3d.md#10-supported-families](../reference/charts/3d.md#10-supported-families)

- `chart3d.family.recipes` (judgment): The supported family names are recipes over the closed primitive set, not additional tokens:
- `chart3d.family.contours` (judgment):

  | Family | V1 representation |
  | --- | --- |
  | Contours on surface | A `layer` with `surface` followed by server-emitted `line` children; contour identity/level uses typed `detail` and tooltip fields |

- `chart3d.family.waterfall` (judgment):

  | Family | V1 representation |
  | --- | --- |
  | Waterfall/spectrum 3D | `line` with one `detail` identity per trace and an `order` field along each spectrum |

- `chart3d.family.math-surface` (judgment):

  | Family | V1 representation |
  | --- | --- |
  | Arbitrary mathematical surface | App code evaluates and samples the function, then emits `surface`; parametric functions use `u,v` |

- `chart3d.family.no-client-calculation`: There are intentionally no `bubble`, `contour`, `pointCloud`, `waterfall`, or formula-expression tokens. Contours are not calculated by the client. An App that changes a formula or sampling density returns another complete typed snapshot.

### 11. Localization and visible text

Owner: [3d.md#11-localization-and-visible-text](../reference/charts/3d.md#11-localization-and-visible-text)

- `chart3d.l10n.no-wording`: No `Chart3D*` property contains user-visible wording.

### 12.5 Visible and accessible controls

Owner: [3d.md#125-visible-and-accessible-controls](../reference/charts/3d.md#125-visible-and-accessible-controls)

- `chart3d.a11y.not-sole-carrier` (judgment): Depth, perspective, lighting, or color can never be the only carrier of domain meaning.

### 13. Security and resources

Owner: [3d.md#13-security-and-resources](../reference/charts/3d.md#13-security-and-resources)

- `chart3d.limits.no-schema-caps.authoring`: Author Chart3D without additional schema-specific count caps; [ordinary transport and client resource budgets still apply](../reference/charts/3d.md#13-security-and-resources).
- `chart3d.limits.no-executable`: The contract admits no executable expressions, arbitrary field paths, HTML, URLs, remote images, shader text, material programs, platform objects, model assets, client-side joins, client topology inference, or arbitrary JSON option blobs.

### 16. Deliberate exclusions

Owner: [3d.md#16-deliberate-exclusions](../reference/charts/3d.md#16-deliberate-exclusions)

- `chart3d.exclusion.compute-in-app` (judgment): If an App can compute a feature and return explicit typed rows and existing 3D primitives, it does so. A future schema extension is warranted only for a portable presentation or interaction capability that cannot be expressed safely that way.

### 3D charts

Owner: [presentation.md#3d-charts](../reference/presentation.md#3d-charts)

- `chart3d.no-gesture-modes` (judgment): Do not invent a camera or gesture `mode`, encode touch mappings in response data, or add sensitivity or control-label fields. `orbit`, `pan`, and `zoomEnabled` only gate independent client capabilities. The preview is gesture-neutral; **Explore 3D chart** opens full-screen Chart Explorer, whose fixed client vocabulary maps one-finger drag to Rotate, two-finger translation to Move, and pinch/spread to Zoom. Tap-only and accessibility actions expose Rotate, Move, Zoom, and Reset separately, without a Rotate/Move selector.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### 1. Purpose and compatibility boundary

Owner: [3d.md#1-purpose-and-compatibility-boundary](../reference/charts/3d.md#1-purpose-and-compatibility-boundary)

- `chart3d.data.no-mixed-node`: A node never carries both a `Chart3D*` and a `Chart*` attribute

### 2. Source and generated syntax

Owner: [3d.md#2-source-and-generated-syntax](../reference/charts/3d.md#2-source-and-generated-syntax)

- `chart3d.syntax.canonical-once`: Each node carries at most one of each Chart3D attribute; the nearest replaces the whole value
- `chart3d.syntax.kind-not-type`: Chart3DScale source uses kind; a source type property is rejected

### 3.1 Unit charts

Owner: [3d.md#31-unit-charts](../reference/charts/3d.md#31-unit-charts)

- `chart3d.data.unit-shape`: A 3D unit chart is a List of row Tables whose direct scalar fields carry the channels
- `chart3d.data.hidden-row-field`: A Hidden row field carries no `Chart3DChannel`, `Chart3DScale` or `Chart3DGuide`
- `chart3d.data.carrier`: A carrier Table holds exactly one non-hidden List field; every other field is Hidden

### 3.2 Compositions

Owner: [3d.md#32-compositions](../reference/charts/3d.md#32-compositions)

- `chart3d.compose.children`: Every visible 3D composition child carries Chart3D, at least one; others are Hidden

### 3.3 Protocol Union wrappers

Owner: [3d.md#33-protocol-union-wrappers](../reference/charts/3d.md#33-protocol-union-wrappers)

- `chart3d.data.union-wrapper`: A wrapping Union variant holds exactly one chart field; every sibling carries Hidden

### 4. `Chart3D`

Owner: [3d.md#4-chart3d](../reference/charts/3d.md#4-chart3d)

- `chart3d.mark.one-token`: Chart3D holds exactly one mark or composition token

### 4.1 Primitive marks

Owner: [3d.md#41-primitive-marks](../reference/charts/3d.md#41-primitive-marks)

- `chart3d.mark.xyz-required`: Every primitive 3D mark requires one x, one y and one z binding
- `chart3d.mark.point`: `point` forbids secondary and topology channels
- `chart3d.mark.line`: `line` forbids secondary and topology channels
- `chart3d.mark.surface`: `surface` and `wireframe` take paired `u,v` or none, and no secondary endpoints

### Point and line

Owner: [3d.md#point-and-line](../reference/charts/3d.md#point-and-line)

- `chart3d.line.unordered-needs-order`: A 3D `line` on an Unordered List must declare `order`

### Bar

Owner: [3d.md#bar](../reference/charts/3d.md#bar)

- `chart3d.bar.forms`: A 3D bar is an explicit cuboid, a continuous-z sparse column, or the two-band shorthand
- `chart3d.bar.cuboid`: An explicit cuboid uses six continuous positions whose secondaries match their primaries

### Stem

Owner: [3d.md#stem](../reference/charts/3d.md#stem)

- `chart3d.stem.geometry`: `stem` takes only `z2` as a secondary, no topology, and a continuous `z`

### 4.2 Appearance applicability

Owner: [3d.md#42-appearance-applicability](../reference/charts/3d.md#42-appearance-applicability)

- `chart3d.appearance.closed`: Each 3D mark accepts only the appearance roles in its table row

### 4.3 Compositions

Owner: [3d.md#43-compositions](../reference/charts/3d.md#43-compositions)

- `chart3d.compose.layer-children`: A 3D layer holds units or layers, never a direct concat

### 5.1 Multiplicity

Owner: [3d.md#51-multiplicity](../reference/charts/3d.md#51-multiplicity)

- `chart3d.channel.singletons`: Only `tooltip` and `detail` may repeat across fields of one 3D unit
- `chart3d.channel.multi-role`: A field with `Chart3DScale` has exactly one scale-bearing role, and never both `u` and `v`
- `chart3d.channel.secondary-topology`: Secondaries need their primary and own no scale; `u` and `v` own no scale or guide

### 5.2 Scalar compatibility

Owner: [3d.md#52-scalar-compatibility](../reference/charts/3d.md#52-scalar-compatibility)

- `chart3d.channel.scalar-compat`: Each 3D role accepts only the scalar types in the compatibility table
- `chart3d.channel.surface-continuous`: `surface` and `wireframe` need continuous `x`, `y` and `z` positions
- `chart3d.channel.unit-format`: Paired endpoints and shared 3D scales use the same PhysicalUnit and Format

### 6. `Chart3DScale`

Owner: [3d.md#6-chart3dscale](../reference/charts/3d.md#6-chart3dscale)

- `chart3d.scale.properties`: Chart3DScale keys are unique; `logBase` only with `log`; `padding` only on a bar base
- `chart3d.scale.growth`: `growMin` and `growMax` are non-negative and only on a continuous `x`, `y` or `z` scale
- `chart3d.scale.identity`: `identity` scales apply only to literal appearance values, never positions, facets or topology
- `chart3d.scale.domains`: 3D scale bounds, `zero`, log and sqrt restrictions match `ChartScale`

### 6.1 Shared scales

Owner: [3d.md#61-shared-scales](../reference/charts/3d.md#61-shared-scales)

- `chart3d.shared-scale.auto-layer`: Unnamed `x`, `y`, `z` scales in one 3D layer share automatically and must be compatible
- `chart3d.shared-scale.explicit-id`: 3D scales sharing an `id` must be compatible and must not conflict on domain or `logBase`

### 7. `Chart3DGuide`

Owner: [3d.md#7-chart3dguide](../reference/charts/3d.md#7-chart3dguide)

- `chart3d.guide.properties`: Chart3DGuide needs `target` and `source`; `tickCount` and `tickStep` are exclusive x/y/z scale options

### 7.1 Placement

Owner: [3d.md#71-placement](../reference/charts/3d.md#71-placement)

- `chart3d.guide.scale-placement`: A scale or none guide sits on the field owning its target primary role
- `chart3d.guide.label-field`: `value` and `span` guides sit on a guide-label field without scale-bearing roles
- `chart3d.guide.span`: A 3D `span` guide targets `x`, `y` or `z` and needs the matching secondary
- `chart3d.guide.detail-value`: A `detail` value guide sits on a `detail` field; one detail guide per unit
- `chart3d.guide.appearance-sources`: Appearance and facet 3D guides accept only source `scale` or `none`

### 7.2 Components and inference

Owner: [3d.md#72-components-and-inference](../reference/charts/3d.md#72-components-and-inference)

- `chart3d.guide.components`: A 3D guide declares only its kind's components; `source=none` declares none

### 7.5 Layer ownership

Owner: [3d.md#75-layer-ownership](../reference/charts/3d.md#75-layer-ownership)

- `chart3d.guide.shared-merge`: Explicit guides on one shared 3D scale group must be identical
- `chart3d.guide.one-axis-per-layer`: A 3D layer shows at most one `x`, one `y` and one `z` guide

### 8. `Chart3DView`

Owner: [3d.md#8-chart3dview](../reference/charts/3d.md#8-chart3dview)

- `chart3d.view.unique-canonical`: `Chart3DView` holds unique named properties in canonical order
- `chart3d.view.projection`: `projection` is `perspective` (default) or `orthographic`
- `chart3d.view.aspect`: `aspectX`, `aspectY`, `aspectZ` are positive ratios, default `1,1,1`
- `chart3d.view.target`: `targetX`, `targetY`, `targetZ` are finite normalized coordinates, default `0.5`
- `chart3d.view.azimuth`: `azimuth` is any finite degrees, canonicalized modulo 360, default `45`
- `chart3d.view.elevation`: `elevation` is any finite degrees, clamped by the client to [-90,90], default `30`
- `chart3d.view.zoom`: `zoom` is a positive finite scalar, default `1`
- `chart3d.view.fov`: `fov` is vertical degrees with 0 < fov < 180, default `45`
- `chart3d.view.orbit`: `orbit` is a Boolean, default `true`
- `chart3d.view.pan`: `pan` is a Boolean, default `true`
- `chart3d.view.zoom-enabled`: `zoomEnabled` is a Boolean, default `true`

### 8.1 Placement

Owner: [3d.md#81-placement](../reference/charts/3d.md#81-placement)

- `chart3d.view.placement`: A layer root owns its one `Chart3DView`; concat roots give inherited defaults
- `chart3d.view.invalid-elsewhere`: `Chart3DView` on a row field, hidden field, function or layer child is rejected

<!-- /generated -->
