<!-- #ZEROPS_REMOVE_START# -->
# Zerops x Elastic Stack

Elastic is a distributed search and analytics engine at the core of the Elastic Stack, designed for storing, searching, and analyzing large volumes of structured and unstructured data in near real-time.

![elastic](https://github.com/zeropsio/recipe-shared-assets/blob/main/covers/svg/cover-elastic.svg)

<br/>

## Elastic ELK

[ELK (Elastic Stack)](https://www.elastic.co/elastic-stack/) is a powerful open-source suite for centralized logging, monitoring, and analytics, consisting of Elasticsearch for indexing and searching, Logstash for data processing, and Kibana for visualizing the data.

Paste the following yml to Zerops GUI:
```yaml
project:
  name: recipe-elk
  tags:
    - zerops-recipe
services:
  - hostname: elkstorage
    type: elasticsearch@8.16
    mode: NON_HA
    priority: 10

  - hostname: kibana
    type: ubuntu@24.04
    buildFromGit: https://github.com/zeropsio/recipe-elk
    enableSubdomainAccess: true
    verticalAutoscaling:
      minRam: 1
    maxContainers: 1

  - hostname: logstash
    type: ubuntu@24.04
    buildFromGit: https://github.com/zeropsio/recipe-elk
    verticalAutoscaling:
      minRam: 1
    maxContainers: 1
```

To collect all Zerops logs with Logstash, set the following custom log forwarding (through GUI):
```
destination d_logstash {
  udp("logstash" port(1514));
};

log {
  source(s_src); destination(d_logstash);
};
```

Login to Kibana:
- user: `elastic`
- password: `password` environment variable of service `elkstorage` (can be found in GUI)

<br/>

### Forward Logs from External Sources
In order to send logs from other projects too, one must enable the "Public Access through IP Addresses".
1. Navigate to `recipe-elk` project > `logstash` service > "Direct access through IP address" menu > "Open port on IPv6"

![setup port routing](public-port-routing-setup.png)

2. Paste this to the logs source project "Log Forwarding", replace `<elk-project-public-ipv6>` (without `[]` brackets)
```
destination d_logstash {
  tcp6("<elk-project-public-ipv6>" port(1514));
};

log {
  source(s_src); destination(d_logstash);
};
```

## Elastic APM

[Elastic APM](https://www.elastic.co/what-is/application-performance-monitoring) is a performance monitoring tool that hooks into your app to track latency, errors, and transactions across your stack, giving you real-time insights for debugging and optimizing code.

Paste the following yml to Zerops GUI:
```yaml
project:
  name: recipe-elk
  tags:
    - zerops-recipe
services:
  - hostname: elkstorage
    type: elasticsearch@8.16
    mode: NON_HA
    priority: 10

  - hostname: kibana
    type: ubuntu@24.04
    buildFromGit: https://github.com/zeropsio/recipe-elk
    enableSubdomainAccess: true
    verticalAutoscaling:
      minRam: 1
    maxContainers: 1

  - hostname: apmserver
    type: ubuntu@24.04
    buildFromGit: https://github.com/zeropsio/recipe-elk
    maxContainers: 1
    envSecrets:
      SECRET_TOKEN: <@generateRandomString(<32>)>
```
<!-- #ZEROPS_REMOVE_END# -->

## Increasing Memory

To save resources for low-resource development environments, default heap memory size configured for Logstash is 256MB.
Feel free to increase this value by setting `LOGSTASH_HEAP_MEMORY_OVERRIDE` environment variable (e.g. `LOGSTASH_HEAP_MEMORY_OVERRIDE="2g"`) and [trigger re-deploy](https://docs.zerops.io/features/pipeline#using-zerops-gui).
Or fork this repository, change the `LOGSTASH_HEAP_MEMORY` environment variable value in `zerops.yaml` and deploy a new version through `zcli push` or Git integration. [^1].

After that, remember to increase the service scaling settings accordingly.

## After Re-deploy

If you have enabled log forwarding to a Logstash service, and you've deployed new app version to that Logstash service,
it may take some time for `syslog-ng` (running in the project `core` service) to invalidate the connection to the old container(s) and establish a new one(s).

Consider reloading the logs forwarding settings (re-save the same configuration) to receive the fresh logs immediately.

[^1]: https://www.elastic.co/docs/reference/logstash/jvm-settings#heap-size