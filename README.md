Qumulo-Openmetrics-Demo
========

A monitoring and alering solution for Qumulo Cluster based on the openmetrics API using [Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [cAdvisor](https://github.com/google/cadvisor),
[NodeExporter](https://github.com/prometheus/node_exporter) and alerting with [AlertManager](https://github.com/prometheus/alertmanager).  

This is a forked repository. So, you may want to visit the original repo at [Einsteinish/Docker-Compose-Prometheus-and-Grafana](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana)

## Install


### Create .env:
```
ADMIN_USER=admin  
ADMIN_PASSWORD=admin
```

### Clone this repository on your Docker host, cd into ravi-test directory and run compose up:

```
git clone https://github.com/Qumulo/ravi-test.git

# update the cluster dns and port in https://github.com/Qumulo/ravi-test/blob/master/prometheus/prometheus.yml#L21-L21

docker-compose up -d
```

## Prerequisites:

* Docker Engine >= 1.13
* Docker Compose >= 1.11

## Containers:

* Prometheus (metrics database) `http://<host-ip>:9090`
* AlertManager (alerts management) `http://<host-ip>:9093`
* Grafana (visualize metrics) `http://<host-ip>:3000`
* NodeExporter (host metrics collector)
* cAdvisor (containers metrics collector)
* Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)


## Define alerts

Three alert groups have been setup within the [alert.rules.yml](https://github.com/Qumulo/ravi-test/blob/master/prometheus/alert.rules.yml) configuration file:

* Qumulo cluster alerts [targets](https://github.com/Qumulo/ravi-test/blob/master/prometheus/alert.rules#L4-L29)
* Monitoring services alerts [targets](https://github.com/Qumulo/ravi-test/blob/master/prometheus/alert.rules#L31-40)
* Docker Host alerts [host](https://github.com/Qumulo/ravi-test/blob/master/prometheus/alert.rules#L44-L69)
* Docker Containers alerts [containers](https://github.com/Qumulo/ravi-test/blob/master/prometheus/alert.rules#L73-L98)

You can modify the alert rules and reload them by making a HTTP POST call to Prometheus:

```
curl -X POST http://admin:admin@<host-ip>:9090/-/reload
```

## Setup alerting

The AlertManager service is responsible for handling alerts sent by Prometheus server.
AlertManager can send notifications via email, Pushover, Slack, HipChat or any other system that exposes a webhook interface.
A complete list of integrations can be found [here](https://prometheus.io/docs/alerting/configuration).

You can view and silence notifications by accessing `http://<host-ip>:9093`.

The notification receivers can be configured in [alertmanager/config.yml](https://github.com/Qumulo/ravi-test/blob/master/alertmanager/config.yml) file.

You can find more details on setting up Slack integration [here](http://www.robustperception.io/using-slack-with-the-alertmanager/).

![Slack Notifications](https://raw.githubusercontent.com/Qumulo/ravi-test/master/screens/Slack_Notifications.png)