# Watchtower
Watchtower is a drift-detection app for Cloud Foundry. It can be run anywhere
that it will be able to reach the Cloud Controller API, meaning it doesn't
have to run as a Cloud Foundry App, although that is the most likely use-case.
Watchtower does what any real-life watchtower would do -- it observes an area
(in this case, a Cloud Foundry environment) and if something happens that isn't
supposed to, it will communicate that to the authorities (a Prometheus server).

## How it works
Watchtower reads in a `config.yaml` file that contains an allowed list of Cloud
Foundry resources. It will scrape the CF API and detect any resources that are
not in the allowed list. It is expected that Watchtower is being monitored by a
Prometheus instance, as "Unknown" resources are reflected in the exported `/metrics`
endpoint as a Prometheus Gauge. For example, if 2 apps were found that were not
in the provided `config.yaml` allow list, the `watchtower_unknown_apps_total`
Gauge would be set to `2`.

Resources are checked on an opt-in model, meaning if you
provide any app in the `config.yaml`, then all deployed apps must match the allow
list.

## Running Watchtower
Watchtower can be run from anywhere that is able to hit your cloud foundry api.
To run, either download a pre-compiled binary from the [releases](https://github.com/18F/watchtower/releases)
page, or compile the go source yourself using `go build`.

### Environment Variables
The following environment variables are required for watchtower to interact with
Cloud Foundry:

| Environment variable name | Description |
| --- | --- |
| `CF_API` | The full URL of the Cloud Foundry API that Watchtower should interact with. Using the [CF CLI](https://docs.cloudfoundry.org/cf-cli/instal    l-go-cli.html), this value can be found with `cf api`. |
| `CF_USER` | The username of the Cloud Foundry User account for Watchtower to authenticate with. |
| `CF_PASS` | The password of the Cloud Foundry User account for Watchtower to authenticate with. |

### Service Account and Permissions
The user/pass provided to watchtower should be a service account with access to
space auditor permissions. For environments that span multiple spaces such as a
`dev-web` and `dev-data` that are both parts of a larger "dev" environment, the
user provided to Watchtower should have auditor permissions on all spaces that
contain resources you wish to monitor. Keep in mind that once you give auditor
permissions for a space, `list` operations for a resource will now include all
resources of that type from the included space, and thus need to be reflected
in the config file provided to Watchtower to avoid false positives due to
"unknown" resources showing up.

## Watchtower Config
Generic placeholder definitions:
* `<boolean>`: a boolean that can take the values `true` or `false`
* `<string>`: a regular string
* `<secret>`: a regular string that is a secret, such as a password

### Global
```yaml
app_config:
  # Whether to enable monitoring of CF Apps
  [ enabled: <boolean> | default = false ]

  # List of CF Apps to monitor
  apps:
    [ - <app_config> ... ]

space_config:
  # Whether to enable monitoring of CF Spaces
  [ enabled: <boolean> | default = false ]

  # List of CF Apps to monitor
  spaces:
    [ - <space_config> ... ]
```

### `<app_config>`
```yaml
name: <string>
```

### `<space_config>`
```yaml
name: <string>
allow_ssh: <boolean>
```

## Endpoints

| Endpoint | Description |
| --- | --- |
| /metrics | Prometheus-style metrics endpoint containing all Watchtower metrics |
| /config | The current Watchtower config |

## Exported Application Metrics
The following table includes all application-specific prometheus metrics that are exported

| Metric | Type | Description |
| --- | --- | --- |
| `watchtower_app_updates_failed_total` | Counter | Number of times the config refresh for V3Apps has failed for any reason |
| `watchtower_app_updates_success_total` | Counter | Number of times the config refresh for V3Apps has succeeded |
| `watchtower_route_updates_failed_total` | Counter | Number of times the config refresh for Routes has failed for any reason |
| `watchtower_route_updates_success_total` | Counter | Number of times the config refresh for Routes has succeeded |
| `watchtower_shared_domain_updates_failed_total` | Counter | Number of times the config refresh for Shared Domains has failed for any reason |
| `watchtower_shared_domain_success_failed_total` | Counter | Number of times the config refresh for Shared Domains has succeeded |
| `watchtower_unknown_apps_total` | Gauge | Number of Apps deployed that are not in the allowed config file |