# Qumulo Monitoring Dashboard
This dashboard is a monitoring and alerting solution for Qumulo clusters. This solution uses the Qumulo OpenMetrics API with a [Prometheus](https://prometheus.io/) time-series database and [Grafana](http://grafana.org/) monitoring software and includes a set of dashboards and alerts that you can customize or use as templates.

For detailed information about available metrics, see [Qumulo OpenMetrics API Specification](https://docs.qumulo.com/administrator-guide/metrics/openmetrics-api-specification.html) on the Qumulo Documentation Portal.

**Table of Contents**
* [Initial Configuration](#initial-configuration)
  * [Prerequisites](#prerequisites)
  * [Step 1: Clone This Repository to Your Docker Host](#clone-repository)
  * [Step 2: Create a Service Account and Access Token on your Qumulo Clusters](#create-account)
  * [Step 3: Configure Prometheus](#configure-prometheus)
  * [Step 4: Start Prometheus and Grafana](#start-prometheus-grafana)
  * [Step 5: Verify Your Prometheus Configuration](#verify-prometheus-configuration)
  * [Step 6: Verify Your Grafana Configuration](#verify-grafana-configuration)
  * [Step 7: Configure Grafana Alert Notifications](#configure-grafana-alerts)
* [Updating Your Qumulo Monitoring Dashboard Configuration](#updating-configuration)
  * [Updating Your Prometheus Configuration](#updating-prometheus-configuration)
  * [Updating Your Grafana Configuration](#updating-grafana-configuration) 

<a id="initial-configuration"></a>
## Initial Configuration
This section explains the initial configuration of the Qumulo Monitoring Dashboard.

<a id="prerequisites"></a>
### Prerequisites
Before you begin, ensure that you have the following minimum software versions:

* [Git](https://docs.github.com/en/get-started/quickstart/set-up-git)
* Docker Engine 1.13
* Docker Compose 1.11
* Qumulo Core 5.3.0

<a id="clone-repository"></a>
### Step 1: Clone This Repository to Your Docker Host

1. Log in to your Docker host.

1. Use the `git` CLI to clone this repository.

   For more information, see [Cloning a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) in the GitHub documentation.

1. Navigate to the `qumulo-monitoring-dashboard` directory.

<a id="create-account"></a>
### Step 2: Create a Service Account and Access Token on your Qumulo Clusters
For this section, follow [Working with Qumulo Access Tokens](https://docs.qumulo.com/administrator-guide/external-services/using-access-tokens.html) on the Qumulo Documentation Portal.

1. Create a service account.

1. Assign a role with only `PRIVILEGE_METRICS_READ` to the service account.

1. Create an access token for the service account.

1. Save the bearer token temporarily.

   <a id="bearer-token"></a>A _bearer token_ is an item in the `Authorization` HTTP header which acts as the authentication mechanism for the Qumulo REST API.

<a id="configure-prometheus"></a>
### Step 3: Configure Prometheus
1. <a id="define-prometheus-yml"></a>To let Prometheus read metrics from your clusters, update the Prometheus configuration in [`prometheus.yml`](/prometheus/prometheus.yml#L22).

   **⚠️ Important:** Perform the following step for each of your clusters.

1. Into the `scrape_configs` section, copy the `qumulo-cluster` job and fill in the following:

   * `job_name`: A unique name for labeling the cluster's metrics.

   * In the `static_configs` block, for `targets`: A list that contains the cluster's DNS name or IP address.

     **ℹ️ Notes:**
     
     * To specify a port, you must append `:8000` to the DNS name or IP address.
    
     * Because the cluster's metrics are labeled with the target variable, use a DNS name rather than an IP address.
     
     * To allow monitoring to continue to work if a node goes offline, using floating IP addresses rather than DHCP or static IP addresses.

   * In the `authorization` block, for `credentials`: The [bearer token](#bearer-token) for the service account.

   * In the `tls_config` block, for `insecure_skip_verify`: If the Qumulo cluster uses the default, self-signed SSL certificate, set this value to `true`.

1. (Optional) Before you start Prometheus and Grafana, change the default administrator credentials in [`docker-compose.yml`](/docker-compose.yml#L62).

   * For `ADMIN_USER` and `ADMIN_PASSWORD`, specify credentials that conform to your security policies.
   
     **⚠️ Important:** If you don't specify your credentials, Docker uses the default values.
   
   * For `ADMIN_PASSWORD_HASH`, specify the password hash. To [generate the hash for your password](https://caddyserver.com/docs/command-line#caddy-hash-password), install and run [the version of `caddy`](https://caddyserver.com/docs/install) that matches the version of the container that the Qumulo Monitoring Dashboard uses (2.6.2).
   
   **ℹ️ Note:**  Alternatively, you can set your credentials as environment variables:

   * On Linux:
      
     a. To set the environment variables, use the `export` command.
      
        ```bash
        export ADMIN_USER='<username>'
        export ADMIN_PASSWORD='<password>'
        export ADMIN_PASSWORD_HASH='<password-hash>'
        ```
           
     b. To verify the environment variables, use the `printenv` command.
      
   * On Windows:
      
     a. To set the environment variables, use the `setx` command.
      
        ```powershell
        setx ADMIN_USER "<username>"
        setx ADMIN_PASSWORD <password>
        setx ADMIN_PASSWORD_HASH <password-hash>
        ```
           
     b. To verify the environment variables, use the `set` command.
        
<a id="start-prometheus-grafana"></a>
### Step 4: Start Prometheus and Grafana
To start Prometheus and Grafana on the Docker host, run the following command.

**ℹ️ Note:** The `-d` flag runs the container in the background.

```bash
docker-compose up -d
```

<a id="verify-prometheus-configuration"></a>
### Step 5: Verify Your Prometheus Configuration
This section explains how to verify that Prometheus can gather metrics from your Qumulo clusters.

1. Connect to the Prometheus server at `http://<docker-host-ip>:9090`.

1. Log in with the `admin` username and `admin` password.

1. On the top menu bar, select **Status > Targets**.

1. On the **Targets** page, find job name [that you defined in the `prometheus.yml` file](#define-prometheus-yml) and then confirm that that the **State** is **Up**.

   If the **State** isn't **Up**, check the **Error** column.
   
   Common problems include:

   * In the `static_configs` block, a mistake in `targets`, in a DNS name or IP address.

   * Inability to connect to the cluster from the machine that runs the containers.
   
     Test the connection by using the `qq` CLI from that machine.
    
   * In the `static_configs` block, for `targets`, a missing `:8000` port specification.
    
   * In the `tls_config` block `insecure_skip_verify` not set to `true` when using a self-signed SSL certificate on a Qumulo cluster.

<a id="verify-grafana-configuration"></a>
### Step 6: Verify Your Grafana Configuration
This section explains how to verify that Grafana can query Prometheus and display metrics:

1. Connect to the Grafana server at `http://<docker-host-ip>:3000`.

1. Log in with the `admin` username and `admin` password.

1. On the **Welcome to Grafana** page, enter a new, secure password and click **Submit**.

1. On the left menu bar, click <img alt="Dashboards" src="images/dashboards.png" width="15">.

1. On the **Dashboards** page, click the **Qumulo** folder and then click **Cluster Overview**.

1. On the **Qumulo / Cluster Overview Page**, next to the **cluster** label filter, select your cluster from the list.

   Metrics for your cluster begin to populate graphs.

<a id="configure-grafana-alerts"></a>
### Step 7: Configure Grafana Alert Notifications
This section explains how to configure Grafana alerts to notify you through email, Slack, or an alerting tool.

1. On the left menu, click <img alt="Alerting" src="images/alerting.png" width="15">.

1. On the **Alerting** page, click **Contact Points**.

1. In the **Contact Points** section, on the right,  click **+ New contact point**.

1. On the **New contact point** page, do the following.

   a. Enter a **Name** for the contact point.
   
   b. Select a **Contact point type** and fill out the fields that appear depending on the contact point.

   c. To test the contact point click **Test**.
   
      **ℹ️ Note:** The test message might take a few minutes to arrive.
   
1. Click **Save contact point**.

   Grafana begins to use the contact point to deliver alerts.

For SMTP email server configuration, you can add your email server info in the [`grafana.ini`](/grafana/grafana.ini#L663).
SMTP configuration can be referenced here [Configuring Grafana - SMTP configuration](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#smtp) in the Grafana documentation.

<a id="updating-configuration"></a>
## Updating Your Qumulo Monitoring Dashboard Configuration
This section explains how to update the configuration of the Qumulo Monitoring Dashboard for your system.

<a id="updating-prometheus-configuration"></a>
### Updating Your Prometheus Configuration
This section explains how to update the Prometheus configuration for your system.

While Prometheus runs, it doesn't apply configuration changes automatically. To reload the configuration, you must do one of the following:

* To stop and restart the container in which Prometheus runs on your Docker host, run the following commands.

  **ℹ️ Note:** The `-d` flag runs the container in the background.

  ```bash
  docker-compose down
  docker-compose up -d
  ```

* To make an HTTP `POST` call, use the `curl` command. For example:

  ```bash
  curl -X POST http://admin:admin@203.0.113.1:9090/-/reload
  ```

<a id="updating-grafana-configuration"></a>
### Updating Your Grafana Configuration
This section explains how to update the Grafana configuration for your system. To update the built-in Grafana alerts, you must modify their configuration files. To create new alerts, use the Grafana web UI.

**⚠️ Important:** Because the functionality of certain vendors' disks degrades before reaching 0% endurance, by default, the **Disk endurance low** (`low_disk_endurance`) alert notifies when 20% endurance remains. For endurance information, check your disks' vendor documentation.

**⚠️ Important:** Because of dependecies with caddy and container network components, it is not recommended to change the current port settings for accessing Grafana (port 3000). Please make backups of the [`grafana.ini`](/grafana/grafana.ini) before making any changes.

For information about working with Grafana dashboards, see [Create a dashboard](https://grafana.com/docs/grafana/latest/getting-started/build-first-dashboard/#create-a-dashboard) in the Grafana documentation.

For information about working with Grafana configuration using the [`grafana.ini`](/grafana/grafana.ini), see [Configuring Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/) in the Grafana documentation.


While Grafana runs, it doesn't apply alert or any other configuration changes automatically. To reload the configuration, you must do one of the following:

* To stop and restart the container in which Grafana runs on your Docker host, run the following commands.

  **ℹ️ Note:** The `-d` flag runs the container in the background.

  ```bash
  docker-compose down
  docker-compose up -d
  ```

* To make an HTTP `POST` call, use the `curl` command. For example:

  ```bash
  curl -X POST --user admin:admin http://203.0.113.1:3000/api/admin/provisioning/alerting/reload
  ```
