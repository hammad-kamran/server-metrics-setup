# Prometheus and Grafana Installation and Configuration Guide

## Group Members
1. Member 1
2. Member 2
3. Member 3
4. Member 4
5. Member 5

## Why Choose These Tools?

Prometheus and Grafana are popular open-source monitoring and alerting tools that are widely used for monitoring the performance of servers and applications. Prometheus is known for its robust data collection and storage capabilities, while Grafana provides a powerful visualization layer. Together, they offer a comprehensive monitoring solution with the following benefits:

- **Prometheus**: Efficient time-series data collection, flexible query language (PromQL), built-in alerting, and extensive integration capabilities.
- **Grafana**: Highly customizable dashboards, support for multiple data sources, powerful visualization options, and alerting capabilities.

## Steps to Download and Deploy

### Install Prometheus on Ubuntu 20.04

1. Create a dedicated system user for Prometheus:
    ```sh
    sudo useradd --system --no-create-home --shell /bin/false prometheus
    ```

2. Download Prometheus:
    ```sh
    wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz
    ```

3. Extract the downloaded files:
    ```sh
    tar -xvf prometheus-2.32.1.linux-amd64.tar.gz
    ```

4. Create necessary directories:
    ```sh
    sudo mkdir -p /data /etc/prometheus
    ```

5. Move Prometheus binaries:
    ```sh
    sudo mv prometheus promtool /usr/local/bin/
    ```

6. Move configuration files:
    ```sh
    sudo mv consoles/ console_libraries/ /etc/prometheus/
    sudo mv prometheus.yml /etc/prometheus/prometheus.yml
    ```

7. Set correct ownership:
    ```sh
    sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
    ```

8. Create a systemd service for Prometheus:
    ```sh
    sudo vim /etc/systemd/system/prometheus.service
    ```
    Add the following content:
    ```
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
      --config.file=/etc/prometheus/prometheus.yml \
      --storage.tsdb.path=/data \
      --web.console.templates=/etc/prometheus/consoles \
      --web.console.libraries=/etc/prometheus/console_libraries \
      --web.listen-address=0.0.0.0:9090 \
      --web.enable-lifecycle

    [Install]
    WantedBy=multi-user.target
    ```

9. Enable and start Prometheus:
    ```sh
    sudo systemctl enable prometheus
    sudo systemctl start prometheus
    ```

10. Check Prometheus status:
    ```sh
    sudo systemctl status prometheus
    ```

### Install Node Exporter on Ubuntu 20.04

1. Create a dedicated system user for Node Exporter:
    ```sh
    sudo useradd --system --no-create-home --shell /bin/false node_exporter
    ```

2. Download Node Exporter:
    ```sh
    wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
    ```

3. Extract the downloaded files:
    ```sh
    tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
    ```

4. Move Node Exporter binary:
    ```sh
    sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
    ```

5. Clean up:
    ```sh
    rm -rf node_exporter*
    ```

6. Verify Node Exporter installation:
    ```sh
    node_exporter --version
    ```

7. Create a systemd service for Node Exporter:
    ```sh
    sudo vim /etc/systemd/system/node_exporter.service
    ```
    Add the following content:
    ```
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/node_exporter --collector.logind

    [Install]
    WantedBy=multi-user.target
    ```

8. Enable and start Node Exporter:
    ```sh
    sudo systemctl enable node_exporter
    sudo systemctl start node_exporter
    ```

9. Check Node Exporter status:
    ```sh
    sudo systemctl status node_exporter
    ```

### Install Grafana on Ubuntu 20.04

1. Install dependencies:
    ```sh
    sudo apt-get install -y apt-transport-https software-properties-common
    ```

2. Add GPG key:
    ```sh
    wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    ```

3. Add Grafana repository:
    ```sh
    echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```

4. Update and install Grafana:
    ```sh
    sudo apt-get update
    sudo apt-get -y install grafana
    ```

5. Enable and start Grafana:
    ```sh
    sudo systemctl enable grafana-server
    sudo systemctl start grafana-server
    ```

6. Check Grafana status:
    ```sh
    sudo systemctl status grafana-server
    ```

7. Access Grafana at http://<ip>:3000 using default credentials (username: `admin`, password: `admin`).

8. Add Prometheus as a data source:
    - Go to Configuration > Data Sources > Add data source.
    - Select Prometheus.
    - Set URL to `http://localhost:9090`.
    - Click Save & Test.

### Install Pushgateway on Ubuntu 20.04

1. Create a dedicated system user for Pushgateway:
    ```sh
    sudo useradd --system --no-create-home --shell /bin/false pushgateway
    ```

2. Download Pushgateway:
    ```sh
    wget https://github.com/prometheus/pushgateway/releases/download/v1.4.2/pushgateway-1.4.2.linux-amd64.tar.gz
    ```

3. Extract the downloaded files:
    ```sh
    tar -xvf pushgateway-1.4.2.linux-amd64.tar.gz
    ```

4. Move Pushgateway binary:
    ```sh
    sudo mv pushgateway-1.4.2.linux-amd64/pushgateway /usr/local/bin/
    ```

5. Clean up:
    ```sh
    rm -rf pushgateway*
    ```

6. Verify Pushgateway installation:
    ```sh
    pushgateway --version
    ```

7. Create a systemd service for Pushgateway:
    ```sh
    sudo vim /etc/systemd/system/pushgateway.service
    ```
    Add the following content:
    ```
    [Unit]
    Description=Pushgateway
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=pushgateway
    Group=pushgateway
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/pushgateway

    [Install]
    WantedBy=multi-user.target
    ```

8. Enable and start Pushgateway:
    ```sh
    sudo systemctl enable pushgateway
    sudo systemctl start pushgateway
    ```

9. Check Pushgateway status:
    ```sh
    sudo systemctl status pushgateway
    ```

### Install Alertmanager on Ubuntu 20.04

1. Create a dedicated system user for Alertmanager:
    ```sh
    sudo useradd --system --no-create-home --shell /bin/false alertmanager
    ```

2. Download Alertmanager:
    ```sh
    wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
    ```

3. Extract the downloaded files:
    ```sh
    tar -xvf alertmanager-0.23.0.linux-amd64.tar.gz
    ```

4. Create necessary directories:
    ```sh
    sudo mkdir -p /alertmanager-data /etc/alertmanager
    ```

5. Move Alertmanager binary and configuration:
    ```sh
    sudo mv alertmanager-0.23.0.linux-amd64/alertmanager /usr/local/bin/
    sudo mv alertmanager-0.23.0.linux-amd64/alertmanager.yml /etc/alertmanager/
    ```

6. Set correct ownership:
    ```sh
    sudo chown -R alertmanager:alertmanager /etc/alertmanager/ /alertmanager-data/
    ```

7. Create a systemd service for Alertmanager:
    ```sh
    sudo vim /etc/systemd/system/alertmanager.service
    ```
    Add the following content:
    ```
    [Unit]
    Description=Alertmanager
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=alertmanager
    Group=alertmanager
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/alertmanager \
      --config.file=/etc/alertmanager/alertmanager.yml \
      --storage.path=/alertmanager-data

    [Install]
    WantedBy=multi-user.target
    ```

8. Enable and start Alertmanager:
    ```sh
    sudo systemctl enable alertmanager
    sudo systemctl start alertmanager
    ```

9. Check Alertmanager status:
    ```sh
    sudo systemctl status alertmanager
    ```

### Alertmanager Configuration for Email Alerts

1. Edit Alertmanager configuration file:
    ```sh
    sudo vim /etc/alertmanager/alertmanager.yml
    ```

2. Add the following configuration to send email alerts:
    ```yaml
    global:
      smtp_smarthost: 'smtp.example.com:587'
      smtp_from: 'alertmanager@example.com'
      smtp_auth_username: 'alertmanager@example.com'
      smtp_auth_password: 'yourpassword'

    route:
      group_by: ['alertname']
      receiver: 'email'

    receivers:
      - name: 'email'
        email_configs:
          - to: 'your-email@example.com'
            send_resolved: true
    ```

3. Restart Alertmanager:
    ```sh
    sudo systemctl restart alertmanager
    ```

## Summary of Questions and Answers

### 1. Why Choose These Tools?
Prometheus and Grafana are chosen for their robustness, flexibility, and wide adoption in the industry for monitoring and alerting purposes. They provide a comprehensive solution for collecting, storing, visualizing, and alerting on metrics from various sources.

### 2. How to Install Prometheus and Grafana?
Detailed step-by-step installation instructions for Prometheus, Node Exporter, Grafana, Pushgateway, and Alertmanager on Ubuntu 20.04 are provided above.

### 3. Email Alerts Configuration
The configuration for sending email alerts using Alertmanager is included, with an example SMTP configuration.

---

By following this guide, you should have a fully functional monitoring and alerting setup using Prometheus, Grafana, Node Exporter, Pushgateway, and Alertmanager. This setup will enable you to monitor system performance, visualize data, and receive alerts for critical issues.

