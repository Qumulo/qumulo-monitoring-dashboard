Qumulo Monitoring Dashboard
===========================

A monitoring and alerting solution for Qumulo clusters. This uses the Qumulo OpenMetrics API with a
[Prometheus](https://prometheus.io/) time series database and [Grafana](http://grafana.org/)
monitoring software. It comes with a prebuilt set of dashboards and alerts, with the ability to
further customize by modifying them or creating new ones.

For information on the complete list of metrics available, see the
[Qumulo OpenMetrics API documentation](https://docs.qumulo.com/administrator-guide/qumulo-core/openmetrics-api-specification.html).

## Initial Configuration

### Prerequisites:

* Docker Engine >= 1.13
* Docker Compose >= 1.11
* Qumulo Core >= 5.3.0


### Step 1 : Clone this repository on your Docker host
```
git clone git@github.com:Qumulo/openmetrics-demo.git
cd openmetrics-demo
```

### Step 2 : Create a service account and access token on your Qumulo cluster(s).
Following the [Qumulo access tokens documentation](https://docs.qumulo.com/administrator-guide/qumulo-core/access-tokens.html),
create a service account and assign it a role with only PRIVILEGE_METRICS_READ. Then create an
access token for that user and temporarily save the bearer token.

### Step 3 : Configure Prometheus
Update the Prometheus [configuration](/prometheus/prometheus.yml#L21-L21) to allow Prometheus to
read metrics from the cluster(s).

Under `scrape_configs`, for each cluster copy the 'qumulo-cluster' job and fill in:

* `job_name`: A unique name that the cluster's metrics will be labeled with.
* `static_configs` -> `targets`: A list containing the DNS name or IP address of the cluster.
    * Port 8000 must be specified after the DNS name or IP address with `:8000`.
    * Prefer using a DNS name over IP address as the cluster's metrics will be labeled with the
    target.
    * Prefer using floating IP addresses over DHCP/static IP addresses as the monitoring will
    continue to work if a node goes offline.
* `authorization` -> `credentials`: The bearer token for the service account.
* `tls_config` -> `insecure_skip_verify`: If the Qumulo cluster is using the default self signed SSL
    certificate set this to `true`.

### Step 4 : Run the docker-compose
```
docker-compose up -d
```

### Step 5 : Verify Prometheus configuration
To verify that Prometheus is able to gather metrics from your Qumulo cluster(s):
1. Connect to the Prometheus server at `http://<docker-host-ip>:9090`.
2. In the menu bar at the top of the page, select `Status` -> `Targets`.
3. Find your `job_name` in the list and verify that the `State` is `Up`.
4. If the `State` is not `Up`, check the `Error` to determine why. Common problems include:
    * Typo in the `static_configs` -> `targets` DNS name or IP address.
    * Unable to connect to the cluster from the machine running the containers. Test the connection
    by using the `qq` command line tool from that machine.
    * Missing the port specification with `:8000` in `static_configs` -> `targets`.
    * Not setting `tls_config` -> `insecure_skip_verify` to `true` when using a self signed SSL
    certificate on the Qumulo cluster.

### Step 6 : Verify Grafana configuration
To verify that Grafana is able to query Prometheus and display metrics:
1. Connect to the Grafana server at `http://<docker-host-ip>:3000`.
2. Login with `username`: `admin`, `password`: `admin`.
3. Change the password to a secure password.
4. In the menu bar on the left, select `Dashboards`.
5. Under `Qumulo`, select `Cluster Overview`.
6. In the `cluster` dropdown in the top left, select your cluster.
7. Metrics for the cluster should now populate the graphs.

### Step 7 : Configure Grafana alert notifications
If you would like to be notified of Grafana alerts through email, slack, or an alerting tool:
1. Connect to the Grafana server at `http://<docker-host-ip>:3000` and login.
2. In the menu bar on the left, select `Alerting` -> `Contact Points`.
3. Under `Contact Points` select `New contact point`.
4. Enter a `Name` for the contact point.
5. Select a `Contact point type` from the dropdown.
6. Complete the form depending on the `Contact point type` selected.
7. Test the contact point using the `Test` button. It may take a few moments for the test message to
  arrive.


## Updating Configuration
### Prometheus
While Prometheus is running it will not pick up configuration changes automatically. You can stop
and restart it's container to reload the configuration or you can request that it reload it's
configuration by making a HTTP POST call using:
```
curl -X POST http://admin:admin@<docker-host-ip>:9090/-/reload
```
