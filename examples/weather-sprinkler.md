<h1 style="font-size:2.6rem;line-height:1.1;">Weather-based sprinkler App</h1>

**Water according to plant needs and the weather.**

The `libertas-sprinkler_agent` App calculates and executes irrigation schedules
using weather, estimated root-zone water deficit, and observed valve open time.
Plant and sprinkler-head profiles shape the amount and timing.

<h2 style="font-size:2rem;line-height:1.2;">From forecast to action</h2>

- **Plan a suitable window.** Forecast rain, temperature, wind, and humidity
  influence scheduling. With location available, the App also considers sunrise.
- **Respect direct controls.** Users adjust watering amounts, set no-watering
  periods, and select Active or Winterization mode.
- **Operate and explain.** The App coordinates Matter irrigation valves and
  publishes live state, upcoming watering, and report charts.
- **Continue through missing forecasts.** A deficit-based offline schedule
  remains available; configured hold-offs remain no-watering intervals.

A configured task connects a weather-service endpoint and a Matter irrigation
valve for each zone. Users choose plant and sprinkler-head types and notification
recipients. Libertas provides the configuration forms, request editors, live
views, and chart presentation from the App's declarations.

This is the irrigation application, distinct from the synthetic garden showcase
used in the [development tutorial](../guides/developing-apps.md).

### For App authors

The [sample source](https://github.com/LibertasIoT/libertas-samples) separates
reusable weather declarations, weather acquisition, and sprinkler control.
Declare both storage unions under the
[storage schema contract](../reference/protocols.md#storage-schema-declarations).
Include the endpoint or valve identity before the ordered archive position in
keys, so a prefix selects one owner's records. Commit related weather metadata
and periods together; commit related watering state and accounting together.
Use transaction-scoped pagination for a stable archive view and bounded
transactions for retention cleanup. The
[KV specification](../reference/protocols.md#transactional-key-value-api) owns
targets, ordering, pagination, failure behavior and limits.

[Explore chart presentation](../guides/chart-gallery.md) · [Back to Libertas](../index.md)
