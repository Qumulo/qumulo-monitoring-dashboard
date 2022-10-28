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

A _bearer token_ is an item in the `Authorization` HTTP header which acts as the authentication mechanism for the Qumulo REST API.

1. Create a service account.

1. Assign a role with only `PRIVILEGE_METRICS_READ` to the service account.

1. Create an access token for the service account.

1. Save the bearer token temporarily.

### Step 3: Configure Prometheus
1. To let Prometheus read metrics from your clusters, update the Prometheus configuration in [`prometheus.yml`](/prometheus/prometheus.yml#L22).

   **Note:** Perform the following step for each of your clusters.

1. Into the `scrape_configs` section, copy the `qumulo-cluster` job and fill in the following:

   * `job_name`: A unique name for labeling the cluster's metrics.

   * In the `static_configs` block, for `targets`: A list that contains the cluster's DNS name or IP address.

     **Notes:**
     
     * To specify the port, append `:8000` to the DNS name or IP address.
    
     * Because the cluster's metrics are labeled with the target variable, use a DNS name rather than an IP address.
     
     * To allow monitoring to continue to work if a node goes offline, using floating IP addresses rather than DHCP or static IP addresses.

   * In the `authorization` block, for `credentials`: The bearer token for the service account.

   * In the `tls_config` block, for `insecure_skip_verify`: If the Qumulo cluster uses the default, self-signed SSL certificate, set this value to `true`.

### Step 4: Start Prometheus and Grafana
To start Prometheus and Grafana on the Docker host, run the following command. The `-d` flag runs the container in the background.

```bash
docker-compose up -d
```

### Step 5: Verify Your Prometheus configuration
This section explains how to verify that Prometheus can gather metrics from your Qumulo clusters.

1. Connect to the Prometheus server at `http://<docker-host-ip>:9090`.

1. Log in with the `admin` username and `admin` password.

1. On the top menu bar, select **Status > Targets**.

1. On the **Targets** page, find job name that you previously define in the `prometheus.yml` file and then confirm that that the **State** is **Up**.

   If the **State** isn't **Up**, check the **Error** column.
   
   Common problems include:

   * In the `static_configs` block, a mistake in `targets`, in a DNS name or IP address.

   * Inability to connect to the cluster from the machine that runs the containers.
   
     Test the connection by using the `qq` CLI from that machine.
    
   * In the `static_configs` block, for `targets`, a missing `:8000` port specification.
    
   * In the `tls_config` block `insecure_skip_verify` not set to `true` when using a self-signed SSL certificate on a Qumulo cluster.

### Step 6: Verify Your Grafana Configuration
This section explains how to verify that Grafana can query Prometheus and display metrics:

1. Connect to the Grafana server at `http://<docker-host-ip>:3000`.

1. Log in with the `admin` username and `admin` password.

1. On the **Welcome to Grafana** page, enter a new, secure password and click **Submit**.

1. On the left menu bar, click [screenshot] **Dashboards**.

1. On the **Dashboards** page, click the **Qumulo** folder and then click **Cluster Overview**.

1. On the **Qumulo / Cluster Overview Page**, next to the **cluster** label filter, select your cluster from the lisut.

   Metrics for your cluster begin to populate graphs.

### Step 7: Configure Grafana Alert Notifications
This section explains how to configure Grafana alerts to notify you through email, Slack, or an alerting tool.

1. On the left menu, click [screenshot] **Alerting**.

1. On the **Alerting** page, click **Contact Points**.

1. In the **Contact Points** section, on the right,  click **+ New contact point**.

1. On the **New contact point** page, do the following.

   a. Enter a **Name** for the contact point.
   
   b. Select a **Contact point type** and fill out the fields that appear depending on the contact point.

   c. To test the contact point click **Test**.
   
      **Note:** The test message might take a few minutes to arrive.
   
1. Click **Save contact point**.

   Grafana begins to use the contact point to deliver alerts.


## Updating Your Configuration
This section explains updating the configuration of the Qumulo Monitoring Dashboard.

### Updating Prometheus Configuration
While Prometheus runs, it doesn't apply configuration changes automatically. To reload the configuration, you must do one of the following:

* To stop and restart the container that Prometheus runs in, on your Docker host, run the following commands. The `-d` flag runs the container in the background.

  ```bash
  docker-compose down
  docker-compose up -d
  ```

* Make an HTTP `POST` call by using the following command.

  ```bash
  curl -X POST http://admin:admin@<docker-host-ip>:9090/-/reload
  ```
