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
    
     * Because the cluster's metrics are labeled with the target, use a DNS name rather than an IP address.
     
     * To allow monitoring to continue to work if a node goes offline, using floating IP addresses rather than DHCP or static IP addresses.

    * `authorization` -> `credentials`: The bearer token for the service account.

    * `tls_config` -> `insecure_skip_verify`: If the Qumulo cluster uses the default, self-signed SSL certificate, set this value to `true`.

### Step 4: Run the Docker Compose Tool

```bash
docker-compose up -d
```

### Step 5: Verify Your Prometheus configuration
This section explains how to verify that Prometheus can gather metrics from your Qumulo clusters.

1. Connect to the Prometheus server at `http://<docker-host-ip>:9090`.

1. At the top of the page, from the menu bar, select **Status > Targets**.

1. In the list, find your **job_name** and confirm that the **State** is **Up**.

   If **State** isn't **Up**, check the **Error** to determine why. Common problems include:

   * A mistake in the `static_configs` -> `targets` DNS name or IP address.

   * Inability to connect to the cluster from the machine that runs the containers.
   
     Test the connection by using the `qq` CLI from that machine.
    
   * Missing `:8000` port specification in `static_configs` -> `targets`.
    
   * `tls_config` -> `insecure_skip_verify` not set to `true` when using a self-signed SSL certificate on a Qumulo cluster.

### Step 6: Verify Your Grafana Configuration
This section explains how to verify that Grafana can query Prometheus and display metrics:

1. Connect to the Grafana server at `http://<docker-host-ip>:3000`.

1. Log in with the `admin` username and `admin` password.

1. Configure a secure password.

1. On the left menu, click **Dashboards**.

1. Under **Qumulo**, click **Cluster Overview**.

1. In the upper-left, in the **cluster** list, click your cluster.

   Metrics for your cluster begin to populate graphs.

### Step 7: Configure Grafana Alert Notifications
This section explains how to configure Grafana alerts to notify you through email, Slack, or an alerting tool.

1. Log in to the Grafana server at `http://<docker-host-ip>:3000`.

1. On the left menu, click **Alertin > Contact Points**.

1. Under **Contact Points**, click **New contact point**.

1. Enter a **Name** for the contact point.

1. From the list, click a **Contact point type**.

1. Complete the form that appears depending on the contact point type that you select.

1. To test the contact point, click **Test** button.

   **Note:** The test message might take a few minutes to arrive.


## Updating Your Configuration
This section explains updating the configuration of the Qumulo Monitoring Dashboard.

### Updating Prometheus Configuration
While Prometheus runs, it doesn't apply configuration changes automatically. To reload the configuration, you must do one of the following:

* Stop and restart the container that Prometheus runs in.

* Make an HTTP `POST` call by using the following command.

  ```bash
  curl -X POST http://admin:admin@<docker-host-ip>:9090/-/reload
  ```
