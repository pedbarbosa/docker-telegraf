# Telegraf

Telegraf Docker image based on [`telegraf:1.2.1`](https://hub.docker.com/_/telegraf/) for use in pure Docker environments, but easily configurable through environment variables.

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

```
docker run -d --name telegraf \
    --hostname=$HOSTNAME
    -e OUTPUTS_INFLUXDB_URLS='["http://influxdb-server-ip:8086"]' \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    --restart=always \
    pedbarbosa/docker-telegraf:1.2.1
```

## Repository update

```
docker build .
DOCKER_IMAGE=$(docker images -q | head -1)
docker login
docker tag $DOCKER_IMAGE pedbarbosa/docker-telegraf:1.2.1
docker push pedbarbosa/docker-telegraf:1.2.1
```
