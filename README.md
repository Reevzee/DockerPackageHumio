# Docker

The Humio Package for Docker provides operational insight into your Docker containers. 
The Package includes Dashboards that allow you to view your container performance statistics for 
CPU, memory, and the network. It also provides visibility into container events such as `start`, `stop`, and 
other important commands.

## Package Contents
_Dashboards_

Metrics - This module fetches metrics from Docker containers. The default metric sets are: `container`, `cpu`, `diskio`, `healthcheck`, `info`, `events`, `memory` and `network`.

## Use Case
- DevOps
- SecOps
- ITOps
- IoT/OT

## Technology Vendor
Docker

## Support
Beta support from Humio (no SLA). Contact support@humio.com, or go to our Slack channel for more.

Tested with metricbeat-oss:7.11.1

## Installation
This setup demonstrates how to set up Elastic Metricbeat to report docker metricsets to Humio.
The metrics are in JSON format, and there is no need for a parser since all workable fields are available automatically in Humio.

1.) Download the example docker config using the curl command supplied or copy and past the example config from below in to desired directory. 

```bash
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.11/deploy/docker/metricbeat.docker.yml 
```

## Humio Configured Config ##

```yaml
metricbeat.config:
  modules:
    # path: ${path.config}/modules.d/*.yml
    # Reload module configs as they change:
    # reload.enabled: false

metricbeat.autodiscover:
  providers:
    - type: docker
      hints.enabled: true

metricbeat.modules:
- module: docker
  metricsets:
    - "container"
    - "cpu"
    - "diskio"
    - "event"
    - "healthcheck"
    - "info"
    - "image"
    - "memory"
    - "network"
  hosts: ["unix:///var/run/docker.sock"]
  period: 10s
  enabled: true

processors:
  - add_cloud_metadata: ~

output.elasticsearch:
  hosts: '${HUMIO_SERVER}:9200'
  password: '${INGEST_TOKEN}'
  # protocol is either `http` (default) or `https`. Remove # on protocol otherwise it defaults to http.
  #protocol: "https"
```
2.) Edit the config example and fill in `${HUMIO_SERVER}` with the URL of your Humio installation, and `${INGEST_TOKEN}` with the ingest token from the repository you want the data in. 

## Deploy MetricBeat in Docker ##

The below command is the quickest and easiest way to get metric beat running with the config supplied: metricbeat.docker.yml via a volume mount. 
With docker run, the volume mount can be specified like this assuming ${metricbeat_config_directory} with the full path to the config locally on your machine.

```bash
docker run -d \
  --name=metricbeat \
  --user=root \
  --volume="/${metricbeat_config_directory}/metricbeat.docker.yml:/usr/share/metricbeat/metricbeat.yml:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  docker.elastic.co/beats/metricbeat-oss:7.11.1 metricbeat -e
```
  
## Dependencies
Important Note: You must download and install the open source version of Elastic Metricbeat.
The standard version of Metricbeat is designed to only work with licensed Elasticsearch and will not connect to Humio successfully. 
Please make sure that the file name of the file that you download looks like metricbeat-oss:7.11.1.zip - https://www.docker.elastic.co/r/beats/metricbeat-oss:7.11.1
