# AWS EC2 Monitoring and Visualization

![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900?logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/Compute-Amazon%20EC2-orange)
![Prometheus](https://img.shields.io/badge/Metrics-Prometheus-E6522C?logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Visualization-Grafana-F46800?logo=grafana&logoColor=white)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-E95420?logo=ubuntu&logoColor=white)

A hands-on monitoring project using **Amazon CloudWatch** to view
standard EC2 metrics and **Node Exporter, Prometheus, and Grafana** to
collect, query, and visualize Linux system metrics on an Ubuntu-based
Amazon EC2 instance.

------------------------------------------------------------------------

## 📌 Table of Contents

-   [Project Overview](#-project-overview)
-   [Objectives](#-objectives)
-   [Architecture](#-architecture)
-   [Tools and Technologies](#-tools-and-technologies)
-   [Metrics Monitored](#-metrics-monitored)
-   [Implementation Steps](#-implementation-steps)
    -   [1. Prepare EC2](#1-prepare-ec2)
    -   [2. Install Prometheus and Node
        Exporter](#2-install-prometheus-and-node-exporter)
    -   [3. Install Grafana](#3-install-grafana)
    -   [4. Configure Grafana Data
        Source](#4-configure-grafana-data-source)
    -   [5. Run PromQL Queries](#5-run-promql-queries)
    -   [6. Verify Prometheus Targets](#6-verify-prometheus-targets)
    -   [7. View CloudWatch Metrics](#7-view-cloudwatch-metrics)
-   [PromQL Query Reference](#-promql-query-reference)
-   [Security Considerations](#-security-considerations)
-   [Validation Checklist](#-validation-checklist)
-   [Current Scope and Limitations](#-current-scope-and-limitations)
-   [Future Improvements](#-future-improvements)
-   [Cleanup and Cost Awareness](#-cleanup-and-cost-awareness)
-   [References](#-references)

------------------------------------------------------------------------

## 📖 Project Overview

Cloud servers need to be monitored so their performance and health can
be checked over time. This project demonstrates a basic monitoring setup
on an **Ubuntu EC2 instance**.

The implementation has two separate monitoring paths:

-   **Amazon CloudWatch:** Used in the AWS Console to view standard EC2
    metrics such as CPU utilization and network traffic.
-   **Node Exporter + Prometheus + Grafana:** Node Exporter exposes
    Linux system metrics, Prometheus collects and stores metrics from
    configured targets, and Grafana queries Prometheus to display
    selected metrics in dashboard panels.

> **Scope note:** CloudWatch metrics are viewed separately in the AWS
> Console. CloudWatch is not configured as a Grafana data source in this
> implementation. A CloudWatch Agent and alerting rules are not part of
> the current setup.

## 🎯 Objectives

-   Use an Ubuntu EC2 instance as the monitoring host.
-   View standard EC2 monitoring metrics in CloudWatch.
-   Expose Linux system metrics using Node Exporter.
-   Collect and query metrics using Prometheus and PromQL.
-   Configure Grafana to use Prometheus as a data source.
-   Visualize CPU and memory utilization in Grafana panels.
-   Verify service health and Prometheus target availability.

------------------------------------------------------------------------

## 🏗 Architecture

### 1. CloudWatch monitoring path

``` text
Amazon EC2
    |
    v
Amazon CloudWatch
    |
    v
AWS Console - EC2 Metrics
```

CloudWatch provides standard EC2 monitoring metrics that can be
inspected from the AWS Console.

### 2. Linux metrics and visualization path

``` text
Ubuntu EC2 Instance
        |
        v
   Node Exporter
        |
        | Exposes Linux system metrics
        v
    Prometheus
        |
        | PromQL queries
        v
      Grafana
        |
        v
CPU and Memory Dashboard Panels
```

**Data flow:**

1.  Node Exporter exposes supported Linux system metrics.
2.  Prometheus scrapes metrics from its configured target and stores
    them as time-series data.
3.  PromQL queries calculate values such as CPU and memory utilization.
4.  Grafana queries Prometheus and visualizes the results in dashboard
    panels.

------------------------------------------------------------------------

## 🛠 Tools and Technologies

  -----------------------------------------------------------------------
  Technology                          Role
  ----------------------------------- -----------------------------------
  **Amazon EC2**                      Provides the Ubuntu virtual server
                                      for monitoring.

  **Amazon CloudWatch**               Displays standard EC2 monitoring
                                      metrics in the AWS Console.

  **Ubuntu Linux**                    Operating system hosting the
                                      monitoring services.

  **Node Exporter**                   Exposes Linux system metrics for
                                      Prometheus to scrape.

  **Prometheus**                      Collects and stores time-series
                                      metrics from configured targets.

  **PromQL**                          Queries and calculates monitoring
                                      metrics.

  **Grafana**                         Visualizes Prometheus metrics in
                                      dashboards.

  **AWS Security Groups**             Control network access to the EC2
                                      instance.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 📊 Metrics Monitored

### Amazon CloudWatch

These standard EC2 metrics can be viewed in CloudWatch, subject to
availability for the selected instance and monitoring configuration.

  -----------------------------------------------------------------------
  Metric                              Description
  ----------------------------------- -----------------------------------
  `CPUUtilization`                    Percentage of allocated EC2 CPU
                                      capacity in use.

  `NetworkIn`                         Bytes received by the instance over
                                      the measurement period.

  `NetworkOut`                        Bytes sent by the instance over the
                                      measurement period.

  `StatusCheckFailed`                 Indicates whether an EC2 status
                                      check has failed.
  -----------------------------------------------------------------------

### Prometheus and Node Exporter

  -----------------------------------------------------------------------
  Metric                              Purpose
  ----------------------------------- -----------------------------------
  `node_cpu_seconds_total`            Cumulative CPU time, separated by
                                      CPU and mode.

  `node_memory_MemAvailable_bytes`    Estimate of memory available for
                                      starting new applications, in
                                      bytes.

  `node_memory_MemTotal_bytes`        Total physical memory, in bytes.
  -----------------------------------------------------------------------

The PromQL queries in this project use these metrics to calculate CPU
and memory utilization percentages.

------------------------------------------------------------------------

## 🚀 Implementation Steps

### 1. Prepare EC2

1.  Launch an Ubuntu EC2 instance in the required AWS Region.
2.  Configure a security group to allow access only from trusted source
    IP addresses.
3.  Connect to the instance using SSH.
4.  Update package lists:

``` bash
sudo apt update
sudo apt upgrade -y
```

**Commonly used ports**

  -----------------------------------------------------------------------
  Port                    Service                 Recommended access
  ----------------------- ----------------------- -----------------------
  `22`                    SSH                     Your trusted IP address

  `3000`                  Grafana                 Your trusted IP address

  `9090`                  Prometheus web          Trusted IP only, if
                          interface               external access is
                                                  needed

  `9100`                  Node Exporter           Keep private; do not
                                                  expose publicly
  -----------------------------------------------------------------------

Restrict inbound rules to the minimum access needed.

### 2. Install Prometheus and Node Exporter

Install both packages on Ubuntu:

``` bash
sudo apt install prometheus prometheus-node-exporter -y
```

Enable and start both services:

``` bash
sudo systemctl enable --now prometheus prometheus-node-exporter
```

Verify the services:

``` bash
sudo systemctl status prometheus
sudo systemctl status prometheus-node-exporter
```

A healthy running service normally reports `active (running)`. Press `q`
to exit the status screen.

### 3. Install Grafana

Grafana was installed using its official APT repository instructions for
Debian/Ubuntu.

Install prerequisite packages:

``` bash
sudo apt-get install -y apt-transport-https wget gnupg
```

Create the APT keyrings directory:

``` bash
sudo mkdir -p /etc/apt/keyrings/
```

Download and store the Grafana signing key:

``` bash
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

Add the stable Grafana repository:

``` bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

Update package lists and install Grafana:

``` bash
sudo apt-get update
sudo apt-get install -y grafana
```

Reload systemd, start Grafana, and enable it at boot:

``` bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Verify the service:

``` bash
sudo systemctl status grafana-server
```

**Expected status:** `active (running)`

Open Grafana in a browser:

``` text
http://YOUR-EC2-PUBLIC-IP:3000
```

Replace `YOUR-EC2-PUBLIC-IP` with the instance's current public IP
address. Access requires a reachable instance and an appropriate
security group rule.

Official installation guide: [Grafana for
Debian/Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)

> If the services are already installed, do not rerun the installation
> steps just for a demonstration. Check their status instead.

### 4. Configure Grafana Data Source

1.  Sign in to Grafana.
2.  Open **Connections → Data sources** (the menu can vary by Grafana
    version).
3.  Add or select **Prometheus**.
4.  Set the Prometheus server URL:

``` text
http://localhost:9090
```

5.  Save the configuration and use the connection test if available.

The `localhost` URL is appropriate when Grafana and Prometheus are
running on the same EC2 host and Grafana can reach Prometheus through
localhost.

### 5. Run PromQL Queries

These queries can be run in Prometheus or used in Grafana panels with
Prometheus configured as the data source.

**CPU utilization**

``` promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Memory utilization**

``` promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

See the [PromQL Query Reference](#-promql-query-reference) for
explanations.

### 6. Verify Prometheus Targets

1.  Open the Prometheus web interface:

``` text
http://YOUR-EC2-PUBLIC-IP:9090
```

2.  Navigate to **Status → Targets**.
3.  Check the status of the configured target.

-   **UP:** Prometheus can scrape the target successfully.
-   **DOWN:** Prometheus cannot currently scrape the target; check
    service status, target configuration, connectivity, and port access.

### 7. View CloudWatch Metrics

1.  Open the AWS Management Console.
2.  Navigate to **CloudWatch → Metrics → EC2 metrics**.
3.  Select the relevant EC2 instance or its available metrics.
4.  Review metrics such as `CPUUtilization`, `NetworkIn`, `NetworkOut`,
    and `StatusCheckFailed`.

CloudWatch is viewed separately from the Prometheus/Grafana workflow in
this project.

------------------------------------------------------------------------

## 🧮 PromQL Query Reference

### 1. CPU Utilization

``` promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Purpose:** Estimates CPU utilization as a percentage by calculating
the average idle CPU rate and subtracting it from 100%.

**How it works:**

-   `node_cpu_seconds_total` is a cumulative CPU-time metric exposed by
    Node Exporter.
-   `{mode="idle"}` selects CPU time spent idle.
-   `[5m]` provides a five-minute range of samples for the rate
    calculation.
-   `irate()` estimates the recent per-second rate of change using the
    latest two samples in the range.
-   `avg by (instance)` averages the idle rate across CPU series for
    each instance.
-   Multiplication by `100` expresses the idle rate as a percentage;
    subtracting it from `100` estimates CPU utilization.

**Illustrative example:** If average CPU idle time is approximately 70%,
estimated CPU utilization is approximately 30%.

### 2. Memory Utilization

``` promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**Purpose:** Calculates memory utilization as a percentage using total
and available system memory.

**How it works:**

-   `node_memory_MemAvailable_bytes` reports an estimate of memory
    available for starting new applications.
-   `node_memory_MemTotal_bytes` reports total physical memory.
-   Available memory divided by total memory gives the available
    proportion.
-   Subtracting that proportion from `1` gives the proportion in use.
-   Multiplication by `100` converts the result to a percentage.

**Illustrative example:** If 25% of memory is available, the query
calculates approximately 75% memory utilization.

> Actual values depend on the live state of the EC2 instance. The
> examples above are illustrative, not measured project results.

------------------------------------------------------------------------

## 🧪 Validation Checklist

-   [ ] Ubuntu EC2 instance is running and reachable.
-   [ ] Prometheus service reports `active (running)`.
-   [ ] Node Exporter service reports `active (running)`.
-   [ ] Grafana service reports `active (running)`.
-   [ ] Prometheus shows the expected configured target as `UP`.
-   [ ] CPU PromQL query returns data.
-   [ ] Memory PromQL query returns data.
-   [ ] Grafana is configured to use Prometheus as its data source.
-   [ ] Grafana dashboard displays CPU and memory utilization.
-   [ ] Standard EC2 metrics are visible in CloudWatch.

------------------------------------------------------------------------

## 🔐 Security Considerations

-   Restrict SSH access to trusted IP addresses.
-   Restrict Grafana and Prometheus web access to the source IPs that
    require it.
-   Do not expose Node Exporter's port `9100` publicly.
-   Use strong credentials for Grafana.
-   Do not commit passwords, private keys, AWS access keys, or other
    secrets to the repository.
-   Review AWS resource usage and remove resources that are no longer
    required.

------------------------------------------------------------------------

## ⚠️ Current Scope and Limitations

-   CloudWatch metrics are viewed separately in the AWS Console;
    CloudWatch is not integrated into the Grafana dashboard.
-   A CloudWatch Agent was not installed as part of this implementation.
-   Alerting rules and notification channels were not configured.
-   The Grafana dashboard focuses on selected CPU and memory utilization
    metrics.

These limitations reflect the current implementation and help define
possible next steps.

## 🔮 Future Improvements

-   [ ] Add CloudWatch as a Grafana data source to display AWS and Linux
    metrics in one dashboard.
-   [ ] Configure alert rules and notification channels for selected
    thresholds.
-   [ ] Expand the dashboard with additional system metrics.
-   [ ] Automate installation and configuration with scripts or a
    configuration management tool.
-   [ ] Review authentication, network restrictions, and HTTPS access
    for a more production-oriented deployment.

------------------------------------------------------------------------

## 🧹 Cleanup and Cost Awareness

AWS resources can incur charges while running. When the project is no
longer needed:

1.  Stop or terminate the EC2 instance if it is no longer required.
2.  Review attached EBS volumes and snapshots.
3.  Release allocated Elastic IP addresses that are no longer needed.
4.  Review CloudWatch usage and other billable resources.

Check the AWS billing console before and after cleanup to confirm that
unused resources have been removed. Exact cleanup steps depend on how
the EC2 instance and storage were created.

------------------------------------------------------------------------

## 📚 References

-   [Amazon EC2
    Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
-   [Amazon CloudWatch
    Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
-   [Prometheus
    Overview](https://prometheus.io/docs/introduction/overview/)
-   [PromQL
    Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
-   [Node Exporter](https://github.com/prometheus/node_exporter)
-   [Grafana Documentation](https://grafana.com/docs/)
-   [Grafana Installation for
    Debian/Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)

------------------------------------------------------------------------

## 👨‍💻 Author

**Niketan Sarode**
*Cloud & DevOps *

------------------------------------------------------------------------

*Built as a hands-on project to practice AWS EC2 monitoring, Linux
metrics collection, Prometheus queries, and Grafana visualization.*
