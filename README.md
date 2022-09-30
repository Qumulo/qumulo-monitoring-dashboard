Qumulo-Openmetrics-Demo
========

A monitoring and alering solution for Qumulo Cluster based on the openmetrics API using [Prometheus](https://prometheus.io/) and [Grafana](http://grafana.org/).

This is a forked repository. So, you may want to visit the original repo at [stefanprodan/dockprom](https://github.com/stefanprodan/dockprom) to learn more about non-qumulo pieces.

## Setup

### Prerequisites:

* Docker Engine >= 1.13
* Docker Compose >= 1.11
* Qumulo Core >= 5.2.2


### Step 1 : Clone this repository on your Docker host
```
git clone git@github.com:Qumulo/openmetrics-demo.git
cd openmetrics-demo
```


### Step 2 : Point to Qumulo cluster(s)
Update the prometheus [scrape config](/prometheus/prometheus.yml#L21-L21) with the openmetrics endpoint of the Qumulo cluster(s) you want to monitor.


### Step 3 : Run the docker-compose
```
ADMIN_USER=admin ADMIN_PASSWORD=admin ADMIN_PASSWORD_HASH=JDJhJDE0JE91S1FrN0Z0VEsyWmhrQVpON1VzdHVLSDkyWHdsN0xNbEZYdnNIZm1pb2d1blg4Y09mL0ZP docker-compose up -d
```


## Containers:
* Grafana (visualize metrics) `http://<host-ip>:3000`
* Prometheus (metrics database) `http://<host-ip>:9090`
* AlertManager (alerts management) `http://<host-ip>:9093`
* NodeExporter (host metrics collector)
* cAdvisor (containers metrics collector)
* Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)


## Notes
You can modify the prometheus config(s) and reload them by making a HTTP POST call using:
```
curl -X POST http://admin:admin@<host-ip>:9090/-/reload
```