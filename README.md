# Qumulo Monitoring Dashboard
This dashboard is a monitoring and alerting solution for Qumulo clusters. This solution uses the Qumulo OpenMetrics API with a [Prometheus](https://prometheus.io/) time-series database and [Grafana](http://grafana.org/) monitoring software and includes a set of dashboards and alerts that you can customize or use as templates.

For detailed information about available metrics, see
[Qumulo OpenMetrics API Specification](https://docs.qumulo.com/administrator-guide/qumulo-core/openmetrics-api-specification.html) on the Qumulo Documentation Portal.

## Initial Configuration
This section explains the initial configuration of the Qumulo Monitoring Dashboard.

### Prerequisites
Before you begin, ensure that you have the following (or higher) software versions:

* Docker Engine 1.13
* Docker Compose 1.11
* Qumulo Core 5.3.0


### Step 1: Clone This Repository to Your Docker Host

1. Log in to your Docker host.

1. Use the `git` CLI to clone this repository.

   ```bash
   git clone git@github.com:Qumulo/qumulo-monitoring-dashboard.git
   ```

1. Navigate to the `qumulo-monitoring-dashboard` directory.


### Step 2: Create a Service Account and Access Token on your Qumulo Clusters
For this section, follow the guidance in [Working with Qumulo Access Tokens](https://docs.qumulo.com/administrator-guide/qumulo-core/access-tokens.html) on the Qumulo Documentation Portal.

1. Create a service account.

1. Assign a role with only `PRIVILEGE_METRICS_READ` to the service account.

1. Create an access token for the role.

1. Save the bearer token temporarily.


### Step 3: Configure Prometheus
1. To let Prometheus read metrics from your clusters, update the Prometheus configuration in [`prometheus.yml`](/prometheus/prometheus.yml#L21-L21).

   **Note:** Perform the following step for each of your clusters.

1. Under `scrape_configs`, copy the `qumulo-cluster` job and fill in the following:

   * `job_name`: A unique name for labeling the cluster's metrics.

   * `static_configs` -> `targets`: A list that contains the cluster's DNS name or IP address.

     **Notes:**
     
       * To specify the port, append `:8000` to the DNS name or IP address.
    
       * Because the cluster's metrics are labeled with the target, use a DNS name rather than IP address.
     
       * To allow monitoring to continue to work if a node goes offline, using floating IP addresses rather than DHCP or static IP addresses.

    * `authorization` -> `credentials`: The bearer token for the service account.

    * `tls_config` -> `insecure_skip_verify`: If the Qumulo cluster uses the default, self-signed SSL certificate, set this value to `true`.


### Step 4: Run the Docker Compose Tool

```bash
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
and restart its container to reload the configuration or you can request that it reload its
configuration by making an HTTP POST call using:
```
curl -X POST http://admin:admin@<docker-host-ip>:9090/-/reload
```
