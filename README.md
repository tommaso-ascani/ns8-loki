# Loki

Start and configure an instance of Loki, a log aggregation system.
The module use the [Loki official docker image](https://github.com/grafana/loki/releases)

## Install

Instantiate the module, example:
```
add-module ghcr.io/nethserver/loki:latest 1
```

The output of the command will return the instance name.
Output example:
```
{"module_id": "loki1", "image_name": "Loki log aggregation system", "image_url": "ghcr.io/nethserver/loki:latest"}
```

After the installation, the Loki server will listen on the IP address of the selected node's VPN interface, using the default fixed port `3100`.

## Usage

The instance can be queried with logcli. eg:
```
root@leader:~# add-module ghcr.io/nethserver/loki:latest 1
Extracting container filesystem ui to /var/lib/nethserver/cluster/ui/apps/loki1
ui/index.html
b89469d5a964b4e97ca2d40b25758cd2e06a96ebe6c00af7d95b1a2d5cf635a5
{"module_id": "loki1", "image_name": "Loki log aggregation system", "image_url": "ghcr.io/nethserver/loki:latest"}
root@leader:~# add-module ghcr.io/nethserver/promtail:latest 1
Extracting container filesystem ui to /var/lib/nethserver/cluster/ui/apps/promtail1
ui/index.html
fe8d71c5b5f0c579ba96e4a64660b275e3a667ffddcc576b1e4cab9f3a8bf9f8
{"module_id": "promtail1", "image_name": "Promtail logs collector for Loki", "image_url": "ghcr.io/nethserver/promtail:latest"}
root@leader:~# logcli labels -q
__name__
job
nodename
root@leader:~# logcli labels nodename -q
bullseye
leader
root@leader:~# logcli  query -q --no-labels -t '{nodename="leader"} | json | line_format "{{.MESSAGE}}"'
2021-05-28T15:49:27Z Created slice cgroup user-libpod_pod_d32e9a0f2237ca996c90f6790ca90447b3e8cbb30d16cc91701ab8257bb704d6.slice.
2021-05-28T15:49:27Z 2021-05-28 15:49:27.621079036 +0000 UTC m=+0.106351212 container create 6e51520a9fa4f63ac0f3dbbf89ef2ef075041dd90c5df952a35206a14691c654 (image=k8s.gcr.io/pause:3.2, name=d32e9a0f2237-infra)
2021-05-28T15:49:27Z 2021-05-28 15:49:27.62160088 +0000 UTC m=+0.106873062 pod create d32e9a0f2237ca996c90f6790ca90447b3e8cbb30d16cc91701ab8257bb704d6 (image=, name=loki)
2021-05-28T15:49:27Z d32e9a0f2237ca996c90f6790ca90447b3e8cbb30d16cc91701ab8257bb704d6
2021-05-28T15:49:27Z loki.service: Found left-over process 19008 (podman pause) in control group while starting unit. Ignoring.
2021-05-28T15:49:27Z This usually indicates unclean termination of a previous run, or service implementation deficiencies.
```

`logcli` will use the default Loki instance of the cluster, this can be changed using the environment variable [`LOKI_ADDR`](https://grafana.com/docs/loki/latest/getting-started/logcli/#example)

## APIs

The module provides some APIs to interact with the Loki instance:

- `configure-module`
- `get-configuration`
- `start-loki2syslog`
- `start-loki2gigasys`
- `stop-loki2syslog`
- `stop-loki2gigasys`
- `get-loki2syslog`
- `get-loki2gigasys`

### `configure-module`

Configure the Loki instance.

#### Parameters

- `retention_days`: The number of days to keep the logs.

#### Example

```bash
api-cli run module/loki1/configure-module '{"retention_days": 7}'
```

### `get-configuration`

Get the Loki instance configuration.

#### Example

```bash
api-cli run module/loki1/get-configuration
```

```json
{
  "retention_days": 7,
  "active_from": "2021-05-28T15:49:27Z+00:00",
  "active_to": "2021-05-28T15:49:27Z+00:00"
}
```

Note: `active_to` field WILL miss if the instance is still active.

### `start-loki2syslog`

Configure Loki 2 Syslog, logs forwarding service from loki to syslog server.

#### Parameters

- `address`: Syslog server address.
- `port`: Syslog server port.
- `protocol`: Sending protocol, valid values are `udp` and `tcp`.
- `format`: Logs format, valid values are `rfcC3164` and `rfc5424`.
- `backlog`: Backlog period value, `hours` of old logs to be sent (default `0`).

#### Example

```bash
api-cli run module/loki1/start-loki2syslog --data '{"address": "127.0.0.1", "port": "514", "protocol": "udp", "format": "rfc3164", "backlog": "24"}'
```

### `start-loki2gigasys`

Configure Loki 2 Gigasys, logs forwarding service from loki to gigasys server.

#### Parameters

- `uuid`: Machine UUID identifier. (To generate a new one, use `uuidgen` command)
- `backlog`: Backlog period value, `hours` of old logs to be sent (default `0`).
- `address`: Gigasys server address.
- `tenant`: Gigasys server tenant.

#### Example

```bash
api-cli run module/loki1/start-loki2gigasys --data '{"uuid": "...", "backlog": "24", "address": "https://gigasys.it", "tenant": "gigasys"}'
```

### `stop-loki2syslog`

Stop and unset Loki 2 Syslog service.

#### Example

```bash
api-cli run module/loki1/stop-loki2syslog
```

### `stop-loki2gigasys`

Stop and unset Loki 2 Gigasys service.

#### Example

```bash
api-cli run module/loki1/stop-loki2gigasys
```

### `get-loki2syslog`

Get Loki 2 Syslog service configuration.

#### Example

```bash
api-cli run module/loki1/get-loki2syslog
```

```json
{
  "address": "172.18.0.2",
  "port": "514",
  "protocol": "udp",
  "format": "rfc3164"
}
```

### `get-loki2gigasys`

Get Loki 2 Gigasys service configuration.

#### Example

```bash
api-cli run module/loki1/get-loki2gigasys
```

```json
{
  "uuid": "...", 
  "address": "https://gigasys.it", 
  "tenant": "gigasys"
}
```

## Uninstall

To uninstall the instance:
```
remove-module loki1
```
