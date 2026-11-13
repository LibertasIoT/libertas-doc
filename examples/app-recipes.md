# Libertas App recipes

These are declaration starting shapes. The Endpoint, permission and persistence
examples have empty function bodies: they do not read a sensor, serve requests,
publish subscriptions, use location, or save data. Implement those behaviors
from the linked contracts and installed SDK before delivery; a successful
typecheck or schema extraction does not make these examples working services.
The TypesOnly declarations are complete for a shared-type package. For a
runnable form-only App, use the tutorial's
[complete configuration example](../guides/developing-apps.md#complete-copyable-source).
Every declaration must follow the
[schema documentation contract](../reference/documentation.md). Keep the
project's existing dependency pins and naming.
The examples use `Example Developer (example_developer)` as placeholder Plaza
identity text. Replace both names with the current developer profile before
publishing.

## TypeScript Endpoint server

**Development-only, schema-only recipe.**
[TypeScript is under development and not release ready](../reference/languages/typescript.md#development-status).
Start with the [strict project setup](../reference/languages/typescript.md#project-configuration-baseline)
and satisfy the [matching SDK prerequisite](../reference/languages/typescript.md#source-code-is-the-ground-truth).
The manifest below requires SDK 0.2.0; confirm that it is publicly available
before installing. For an existing project retain its compatible pins.
Implement the [Endpoint runtime](../reference/protocols.md#endpoint-runtime-rules),
including sensor acquisition and correlated subscription snapshots, before
claiming the service described by these declarations.

`package.json`:

```json
{
  "name": "acme-temperature_monitor",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "build": "tsdown",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "@libertas-ai/types": "0.2.0",
    "tsdown": "^0.22.0",
    "typescript": "^6.0.0"
  }
}
```

`src/index.ts`:

```typescript
/// Temperature monitor
/// The monitor shows a sensor's current temperature and lets the user choose its desired temperature.
/// @LibertasDefaultLocale("en")

/**
 * Temperature activity
 * The user can read temperatures or change the desired temperature, and the monitor returns the resulting readings.
 */
export const enum TemperatureKind {
    /**
     * Read or follow temperatures
     * The user asks for the current and desired temperatures and can keep later readings up to date.
     */
    ReadStatus = 0,

    /**
     * Change desired temperature
     * The user asks the monitor to use a new desired temperature.
     */
    SetTarget = 1,

    /**
     * Temperature readings
     * The monitor returns the current and desired temperatures.
     */
    Status = 2,
}

/**
 * Read or follow temperatures
 * The user asks for the current and desired temperatures and can keep later readings up to date.
 * @LibertasRequest
 * @LibertasSubscriptionRequest
 * @LibertasAccessPrivilege("Read")
 * @LibertasNextResponse("Status")
 */
export interface ReadStatus {
    /**
     * Read or follow temperatures
     * The user asks for the current and desired temperatures and can keep later readings up to date.
     */
    kind: TemperatureKind.ReadStatus;
}

/**
 * Change desired temperature
 * The user changes the desired temperature used by the monitor.
 * @LibertasRequest
 * @LibertasAccessPrivilege("Write")
 * @LibertasNextResponse("Status")
 */
export interface SetTarget {
    /**
     * Change desired temperature
     * The user asks the monitor to use a new desired temperature.
     */
    kind: TemperatureKind.SetTarget;

    /**
     * Desired temperature
     * The temperature the user wants the selected sensor's area to reach.
     * @LibertasNumber(type=f64, min=-40, max=100, step=0.5)
     * @LibertasPhysicalUnit("celsius")
     */
    target: number;
}

/**
 * Temperature readings
 * The monitor shows the latest measured temperature together with the user's desired temperature.
 * @LibertasResponse
 * @LibertasSubscriptionData
 * @LibertasNextRequest("ReadStatus,SetTarget")
 */
export interface Status {
    /**
     * Temperature readings
     * The monitor returns the current and desired temperatures.
     */
    kind: TemperatureKind.Status;

    /**
     * Current temperature
     * The latest temperature measured by the selected sensor.
     * @LibertasNumber(type=f64)
     * @LibertasPhysicalUnit("celsius")
     * @LibertasFormat("0.0")
     */
    current: number;

    /**
     * Desired temperature
     * The temperature the user currently wants the selected sensor's area to reach.
     * @LibertasNumber(type=f64)
     * @LibertasPhysicalUnit("celsius")
     * @LibertasFormat("0.0")
     */
    target: number;
}

/**
 * Temperature activities
 * The user can read or change temperatures, and the monitor returns the resulting readings.
 */
export type TemperatureProtocol = ReadStatus | SetTarget | Status;

/**
 * Monitor choices
 * The sensor the user chooses for this temperature monitor.
 */
export interface Configuration {
    /**
     * Temperature sensor
     * The sensor whose temperature the monitor shows and manages.
     */
    sensor: LibertasDevice;

    /**
     * Sensor connection
     * The monitor uses this connection to read from and make changes for the selected sensor.
     * @LibertasEndpointSchema(TemperatureProtocol)
     * @LibertasEndpointServer
     * @LibertasEndpointBaseObjects("$.sensor")
     */
    endpoint: LibertasEndpoint;
}

/**
 * Start temperature monitor
 * The user chooses a sensor, views its current and desired temperatures, and can change the desired temperature. The monitor keeps the displayed readings current.
 * Developed by Example Developer (example_developer).
 * @LibertasSingleton
 */
export function startTemperatureMonitor(
    /**
     * Monitor choices
     * The user chooses the sensor and connection used by this monitor.
     */
    configuration: Configuration,
): void {
    void configuration;
}
```

Then run:

```sh
npm run typecheck
npm run build
```

Studio performs schema validation after TypeScript succeeds.

## Rust Endpoint server

**Schema-only recipe.** Follow the [Rust project setup](../reference/languages/rust.md#project-contract-and-library-requirements)
and implement the [Endpoint runtime](../reference/protocols.md#endpoint-runtime-rules),
including sensor acquisition and correlated subscription snapshots. The empty
function below exposes the declarations but provides none of that runtime work.

`Cargo.toml`:

```toml
[package]
name = "acme-temperature_monitor"
version = "0.1.0"
edition = "2024"

[dependencies]
libertas = { git = "https://github.com/LibertasIoT/libertas-rs.git", package = "libertas" }
libertas_macros = { git = "https://github.com/LibertasIoT/libertas-rs.git", package = "libertas_macros" }

[lib]
crate-type = ["rlib"]
```

`src/lib.rs`:

```rust
//! Temperature monitor
//! The monitor shows a sensor's current temperature and lets the user choose its desired temperature.
//! #[libertas_default_locale("en")]

use libertas::*;
use libertas_macros::*;

/// Temperature activities
/// The user can read or change temperatures, and the monitor returns the resulting readings.
#[derive(Clone, Debug, PartialEq, LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub enum TemperatureProtocol {
    /// Read temperatures
    /// The monitor returns the current and desired temperatures and keeps subsequent readings up to date.
    #[libertas_request]
    #[libertas_subscription_request]
    #[libertas_access_privilege("Read")]
    #[libertas_next_response("Status")]
    ReadStatus,

    /// Change desired temperature
    /// The user changes the desired temperature used by the monitor.
    #[libertas_request]
    #[libertas_access_privilege("Write")]
    #[libertas_next_response("Status")]
    SetTarget {
        /// Desired temperature
        /// The temperature the user wants the selected sensor's area to reach.
        #[libertas_number(min = -40, max = 100, step = 0.5)]
        #[libertas_physical_unit("celsius")]
        target: f64,
    },

    /// Temperature readings
    /// The monitor shows the latest measured temperature together with the user's desired temperature.
    #[libertas_response]
    #[libertas_subscription_data]
    #[libertas_next_request("ReadStatus,SetTarget")]
    Status {
        /// Current temperature
        /// The latest temperature measured by the selected sensor.
        #[libertas_physical_unit("celsius")]
        #[libertas_format("0.0")]
        current: f64,

        /// Desired temperature
        /// The temperature the user currently wants the selected sensor's area to reach.
        #[libertas_physical_unit("celsius")]
        #[libertas_format("0.0")]
        target: f64,
    },
}

/// Monitor choices
/// The sensor the user chooses for this temperature monitor.
#[derive(Clone, Debug, PartialEq, LibertasExport, LibertasAvroDecode)]
pub struct Configuration {
    /// Temperature sensor
    /// The sensor whose temperature the monitor shows and manages.
    pub sensor: LibertasDevice,

    /// Sensor connection
    /// The monitor uses this connection to read from and make changes for the selected sensor.
    #[libertas_endpoint_schema(TemperatureProtocol)]
    #[libertas_endpoint_server]
    #[libertas_endpoint_base_objects("$.sensor")]
    pub endpoint: LibertasEndpoint,
}

/// Start temperature monitor
/// The user chooses a sensor, views its current and desired temperatures, and can change the desired temperature. The monitor keeps the displayed readings current.
/// Developed by Example Developer (example_developer).
#[libertas_export]
#[libertas_singleton]
pub fn start_temperature_monitor(
    /// Monitor choices
    /// The user chooses the sensor and connection used by this monitor.
    configuration: Configuration,
) {
    let _ = configuration;
}
```

Then run:

```sh
cargo check --lib
```

Studio performs schema validation after Rust succeeds.

## TypesOnly shared package

TypeScript:

```typescript
/// Equipment availability
/// Helps compatible Apps report whether monitored equipment can be used and which observation is newest.
/// @LibertasTypesOnly

/**
 * Equipment status
 * Describes whether monitored equipment can be used and which observation is newest.
 */
export interface SharedStatus {
    /**
     * Equipment available
     * True means the monitored equipment is available for use; false means it
     * is unavailable.
     */
    online: boolean;

    /**
     * Status update number
     * Increases whenever the monitored equipment's reported status changes.
     * @LibertasNumber(type=u32)
     */
    revision: number;
}
```

Rust:

```rust
//! Equipment availability
//! Helps compatible Apps report whether monitored equipment can be used and which observation is newest.
//! #[libertas_types_only]

use libertas::*;
use libertas_macros::*;

/// Equipment status
/// Describes whether monitored equipment can be used and which observation is newest.
#[derive(Clone, Debug, PartialEq, LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub struct SharedStatus {
    /// Equipment available
    /// True means the monitored equipment is available for use; false means it
    /// is unavailable.
    pub online: bool,

    /// Status update number
    /// Increases whenever the monitored equipment's reported status changes.
    pub revision: u32,
}
```

Do not add a public non-ignored function. Every published owned type must be an
exact-name entry-module export.

## Function permission and reason

**Metadata-only recipe.** These declarations show the permission and localized
reason; the empty functions do not read location or obtain a forecast. Follow
the [permission contract](../reference/app-contract.md#functions-permissions-and-singleton-tasks)
and implement the intended SDK operations before claiming that behavior.

TypeScript:

```typescript
/// Local forecast
/// The forecast service selects weather for the Hub's location.
/// @LibertasStringResources(STRINGS)

const PERMISSIONS = [
    "libertas.permission.ACCESS_FINE_LOCATION"
] as const;

const STRINGS = {
    "libertas.permission.ACCESS_FINE_LOCATION":
        "Allow the forecast service to use the Hub location to select local weather."
} as const;

/**
 * Start local forecast
 * The forecast service uses the Hub location to select and start the local forecast. It asks for location access before reading that location.
 * Developed by Example Developer (example_developer).
 * @LibertasPermissions(PERMISSIONS)
 */
export function startLocalForecast(): void {}
```

Rust:

```rust
//! Local forecast
//! The forecast service selects weather for the Hub's location.
//! #[libertas_string_resources(STRINGS)]

const PERMISSIONS: &[&str] = &[
    "libertas.permission.ACCESS_FINE_LOCATION",
];

const STRINGS: &[(&str, &str)] = &[
    (
        "libertas.permission.ACCESS_FINE_LOCATION",
        "Allow the forecast service to use the Hub location to select local weather.",
    ),
];

/// Start local forecast
/// The forecast service uses the Hub location to select and start the local forecast. It asks for location access before reading that location.
/// Developed by Example Developer (example_developer).
#[libertas_export]
#[libertas_permissions(PERMISSIONS)]
pub fn start_local_forecast() {}
```

Every permission needs an exact same-key nonempty default-locale reason.

## Persistent data

**Schema-only recipe.** The functions below declare the persistent-data Union
but never collect, read or write a score. Implement the required Data API
operations and restart behavior using the installed SDK and the contracts below.

Ordinary Data API failures stop the task in the Hub runtime without permanently
disabling it or automatically restarting it. A function may explicitly opt into
checked status-returning calls with `DataErrorStatus`. See the
[Data API failure contract](../reference/protocols.md#persistence-failure-and-the-no-status-api)
for the flag, return values, consistency responsibility, and third-party `Enable` action.

Define structured data and associate its Union with the function. Prefer storing
domain values and generating formatted payload fields at presentation time;
see the [payload guidance](../reference/presentation.md#persisting-formatted-record-payloads).

TypeScript:

```typescript
/// Inspection score recorder
/// Records inspection scores and remembers progress when the recorder starts again.

/**
 * Inspection choices
 * Identifies the inspection whose scores the recorder saves.
 */
export interface Configuration {
    /**
     * Inspection name
     * The user enters the name used to recognize this inspection.
     */
    inspectionName: string;
}

/**
 * Recorded inspection score
 * Represents one inspection score together with the time it was recorded.
 */
export interface StoredSample {
    /**
     * Recorded inspection score
     * Represents one inspection score together with the time it was recorded.
     */
    kind: 0;

    /**
     * Score recorded at
     * Identifies when the inspection score was recorded.
     */
    at: LibertasDateTime;

    /**
     * Inspection score
     * Records the score assigned during the inspection.
     */
    value: number;
}

/**
 * Saved inspection progress
 * Marks the latest completed inspection step so recording can continue after a restart.
 */
export interface StoredCheckpoint {
    /**
     * Saved inspection progress
     * Marks the latest completed inspection step so recording can continue after a restart.
     */
    kind: 1;

    /**
     * Inspection step number
     * Identifies the latest inspection step whose score was safely saved.
     * @LibertasNumber(type=u64)
     */
    revision: number;
}

/**
 * Saved inspection record
 * Contains inspection scores and progress retained when the recorder stops and starts again.

 */
export type StoredData = StoredSample | StoredCheckpoint;

/** Persistent key kinds */
export type DataKey = { kind: 0 } | { kind: 1 };

/**
 * Record inspection scores
 * The recorder saves inspection scores with their times and remembers completed steps so the same inspection can continue after a restart.
 * Developed by Example Developer (example_developer).
 * @LibertasDataSchema(key=DataKey,value=StoredData)
 */
export function start(
    /**
     * Inspection choices
     * The user chooses the inspection whose scores the recorder saves.
     */
    configuration: Configuration,
): void {
    void configuration;
}
```

Rust:

```rust
//! Inspection score recorder
//! Records inspection scores and remembers them when the recorder starts again.

use libertas::*;
use libertas_macros::*;

/// Inspection choices
/// Identifies the inspection whose scores the recorder saves.
#[derive(Clone, Debug, PartialEq, LibertasExport, LibertasAvroDecode)]
pub struct Configuration {
    /// Inspection name
    /// The user enters the name used to recognize this inspection.
    pub inspection_name: String,
}

/// Saved inspection record
/// Contains inspection scores retained when the recorder stops and starts again.
#[derive(LibertasKeyCodec)]
pub enum DataKey { Latest, Checkpoint }

#[derive(Clone, Debug, PartialEq, LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub enum StoredData {
    /// Recorded inspection score
    /// Represents one inspection score together with the time it was recorded.
    Sample {
        /// Score recorded at
        /// Identifies when the inspection score was recorded.
        at: LibertasDateTime,

        /// Inspection score
        /// Records the score assigned during the inspection.
        value: f64,
    },
}

/// Record inspection scores
/// The recorder saves inspection scores with their times so the same inspection can continue after a restart.
/// Developed by Example Developer (example_developer).
#[libertas_export]
#[libertas_data_schema(key = DataKey, value = StoredData)]
pub fn start(
    /// Inspection choices
    /// The user chooses the inspection whose scores the recorder saves.
    configuration: Configuration,
) {
    let _ = configuration;
}
```

Keep the target an actual discriminated Union. Do not weaken validation or edit
the generated schema. Before calling a Data API, declare every single-record or
indexed-table name in the task package's `StringResources` and pass that stable
key with ordered typed arguments. The resulting canonical byte blob is both the
database identity and the localized human-name input; never substitute rendered
or Base64 text. Follow the complete
[data and database name contract](../reference/protocols.md#data-and-database-names).

## Do not copy these anti-patterns

- Rust `src/main.rs` or a binary-only Cargo target.
- Manual `schema.json`, `AppDependencies`, or Hub version numbers.
- An unqualified external type name that is not imported into native scope.
- A mutating Request without `Write` AccessPrivilege.
- `FormattedText` without [static ReadOnly](../reference/presentation.md#static-read-only-placement-and-request-values),
  or in function/configuration arguments.
- Localized text or a display symbol where a CLDR PhysicalUnit identifier is
  required.
- A hand-built Matter descriptor.
- Interpreting Endpoint peer handles or treating a session as an object ID.
- Publishing a new architecture by modifying an existing Plaza release.
