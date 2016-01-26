# The numbat system

The numbat modules:

* [numbat-emitter](https://github.com/numbat-metrics/numbat-emitter): a module you're intended to require anywhere you need it. Make an emitter object, hang onto it, emit metrics with it.  
* [numbat-collector](https://github.com/numbat-metrics/numbat-collector): receiver that runs on every host  
* [numbat-analyzer](https://github.com/numbat-metrics/numbat-analyzer): a server that accepts data streams from the collector & processes them
* [numbat-influx](https://github.com/numbat-metrics/numbat-influx): an InfluxDB 0.9+ output sink for the collector
* [numbat-redis](https://github.com/npm/numbat-redis): emit some interesting redis stats
* [numbat-haproxy](https://github.com/npm/numbat-haproxy): watch what your haproxies are up to
* [numbat-process](https://github.com/npm/numbat-process): emit periodic stats about your node process

Design:

- There's a cluster of [InfluxDB](http://influxdb.com)s.
- Each host runs a [numbat-collector](https://github.com/numbat-metrics/numbat-collector).
- Each service has a client that sends a heartbeat to `numbat-collector`.
- Each service also sends interesting datapoints to `numbat-collector`.
- The per-host collectors send all data to InfluxDB.
- They also send it to the [numbat-analyzer](https://github.com/numbat-metrics/numbat-analyzer).
- `numbat-analyzer` analyzes stats & feeds data to InfluxDB.
- `numbat-analyzer` is also responsible for sending alerts & generating timeseries events for these alerts.
- A [grafana](http://grafana.org) dashboard shows the data.
- If this service does its job, you delete your nagios installation.

An example setup might look like this, with many service/collector pairs:

![](diagrams/processing_metrics.png)

NOTE: This is essentially what we run in production at npm, but the documentation for this component is lagging way behind.

### Data flow

Implications:

- everything goes into InfluxDB: metrics, operational actions, other human actions
- Dashboard needs to include both visual data (graphs) & current alert status
- data should probably get tagged with "how to display this" so a new stream of info from numbat can be displayed usefully sans config
- Dashboard should link to the matching Grafana historical data displays for each metric.

CONSIDER: dashboard data displays *are* grafana, just of a different slice of influxdb data (rotated out regularly?) Dashboard page then becomes grafana with the alert stuff in an iframe or something like that. In this approach, the dashboard service is an extra-complex configurable set of hekad-style rules in javascript instead of Lua.

### What does a metric data point look like?

It must be a valid InfluxDB data point. Inspired by Riemann's events.

```javascript
{
    host: 'hostname.example.com',
    name: 'name.of.metric',
    status: 'okay' | 'warning' | 'critical' | 'unknown',
    description: 'textual description',
    time: ts-in-ms,
    ttl: ms-to-live,
    value: 42,
    field: 'abitrary data',
    field2: 'another tag'
}
```

Use tags to carry metadata. Some possibilities:

- `annotation`: a singular event, like a deploy.
- `counter`, `gauge`, etc: hints about how to chart

## Contributing

Yes, please. See [contributing.md](contributing.md) for information about how and our code of conduct.
