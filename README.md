# Telegraf

Telegraf Docker image based on [`telegraf:1.2-alpine`](https://hub.docker.com/_/telegraf/) for use in pure Docker environments, but easily configurable through environment variables.

## Configuration

Environment variable are named like in the configuration file, e.g. `AGENT_INTERVAL` or `OUTPUTS_INFLUXDB_RETENTION_POLICY`, and are initialized with the usual default values. Currently only the following configuration sections are supported:
- `AGENT_*`
- `OUTPUTS_INFLUXDB_*`
- `INPUTS_DOCKER_*`

Due to a limitation in the Telegraf configuration the `AGENT_HOSTNAME` settings is currently ignored. As the hostname is always set using `os.Hostname()` it may be changed by settings the containers hostname, e.g. `docker run --hostname myhost ...`.

### Plugins

All plugins, which can be configured using environment variables, are enabled by default, but may be disabled using the `--input-filter` argument. All other plugins are not configured and can't be enabled without mounting a custom `telegraf.conf`. *This limitation may be relaxed in the future*.

#### Outputs

InfluxDB is the only available output plugin. As opposed to the default configuration `OUTPUTS_INFLUXDB_URLS` is initially set to `["http://localhost:8086"]` which is much more natural to use in a Docker environment and enables the use of standard linking with aliasing, e.g. `--link myinfluxdb:influxdb`. `OUTPUTS_INFLUXDB_RETENTION_POLICY` is set to `autogen` by default and `OUTPUTS_INFLUXDB_SSL_CA_*` settings are currently ignored.

### Inputs

Only exceptions from the defaults are listed here.

## Example

Assuming your InfluxDB is running in a container named `influxdb`, listening on the default port `8086` and you want to only enable the Docker and StatsD plugin you may execute the following command:
```
docker run -d --name telegraf \
    -e AGENT_OMIT_HOSTNAME=true \
    -e AGENT_METRIC_BUFFER_LIMIT=100000 \
    -e AGENT_FLUSH_JITTER=10s \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -p 8125:8125/udp \
    --link influxdb \
  macheins/telegraf:1.2 \
```
