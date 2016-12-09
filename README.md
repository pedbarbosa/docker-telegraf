# Telegraf

Telegraf Docker image based on [`telegraf:1.1-alpine`](https://hub.docker.com/_/telegraf/) for use in pure Docker environments, but easily configurable through environment variables.

## Configuration

Environment variable are named like in the configuration file, e.g. `AGENT_INTERVAL` or `OUTPUTS_INFLUXDB_RETENTION_POLICY`, and are initialized with the usual default values. Currently only the following configuration sections are supported:
- `AGENT_*`
- `OUTPUTS_INFLUXDB_*`
- `INPUTS_DOCKER_*`
- `INPUTS_STATSD_*`
- `INPUTS_REDIS_*`

Due to a limitation in the Telegraf configuration `OUTPUTS_INFLUXDB_RETENTION_POLICY` is set to `autogen` by default and the `AGENT_HOSTNAME` and `OUTPUTS_INFLUXDB_SSL_CA_*` settings are currently ignored. As hostname is always set using `os.Hostname()` it may be changed by settings the containers hostname, e.g. `docker run --hostname myhost ...`.

InfluxDB is the only available output plugin. As opposed to the default configuration `OUTPUTS_INFLUXDB_URLS` is initially set to `["http://influxdb:8086"]` which is much more natural to use in a Docker environment and enables the use of standard linking with aliasing, e.g. `--link myinfluxdb:influxdb`.

Only StatsD, Docker and Redis input plugins are configured and enabled, but may be disabled using the `--input-filter` argument. All other input plugins are not configured and can't be enabled without mounting a custom `telegraf.conf`. *This limitation may be relaxed in the future*. The StatsD port is not configurable and exposed to its default `8125/udp`. `INPUTS_REDIS_SERVERS` is set to `["tcp://redis:6379"]` by default.

## Example

Assuming your InfluxDB is running in a container named `influxdb` and listening on the default port `8086` and you want to disabel the Redis plugin you may run the following command:

```
docker run -d --name telegraf \
    -e AGENT_OMIT_HOSTNAME=true \
    -e AGENT_METRIC_BUFFER_LIMIT=100000 \
    -e AGENT_FLUSH_JITTER=10s \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -p 8125:8125/udp \
    --link influxdb \
  macheins/telegraf:1.1 \
    --input-filter "docker:statsd"
```
