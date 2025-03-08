# Step-by-Step Guide to Install Node Exporter, Configure Prometheus, Set Up Alerting, and Integrate with Grafana
## Step-by-Step Installation of Node Exporter:
#
### Copy the Node Exporter download link
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```
### Extract the downloaded file
```bash
tar xvf node_exporter-1.5.0.linux-amd64.tar.gz
```
### Move into the extracted Node Exporter directory
```bash
cd node_exporter-1.5.0.linux-amd64/
```
#### Start Node Exporter by running the executable:
```bash
./node_exporter
```
#### press ctrl+c in terminal
#
#### Setting Up Node Exporter as a Service
#
#### Copy Node Exporter binary to /usr/local/bin
```bash
sudo cp node_exporter /usr/local/bin
```
#### Create a system user for Node Exporter
```bash
sudo useradd node_exporter --no-create-home --shell /bin/false
```
#### Change ownership of the Node Exporter binary:
```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
#### Create a systemd service file for Node Exporter:
```bash
sudo nano /etc/systemd/system/node_exporter.service
```
#### Add the following content to the service file
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
#### Reload systemd
```bash
sudo systemctl daemon-reload
```
#### Start and enable Node Exporter as a service
```bash
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```
#### Check the status of the Node Exporter service
```bash
sudo systemctl status node_exporter
```
#### http://ip-address:9100/metrics
#
#### Note if you use ec2 allow the port 9100 in security groups
#### If you use linux or ubuntu --- allow 9100 (sudo ufw allow 9100)

#
## Step-by-Step Installation of Prometheus:
#
#### Create a system user for Prometheus
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```
#### Create directories for Prometheus configuration and data
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
#### Set the ownership of the data directory
```bash
sudo chown prometheus:prometheus /var/lib/prometheus
```
#### Move to the /tmp directory
```bash
cd /tmp/
```
#### Download Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.31.1/prometheus-2.31.1.linux-amd64.tar.gz
```
#### Extract the downloaded Prometheus package
```bash
tar -xvf prometheus-2.31.1.linux-amd64.tar.gz
```
#### Move to the extracted directory
```bash
cd prometheus-2.31.1.linux-amd64
```
#### Move configuration files and set ownership
```bash
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```
#### Move binaries and set ownership
```bash
sudo mv prometheus /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```
## Creating Prometheus Systemd Service
#
#### Create a systemd service file for Prometheus
```bash
sudo nano /etc/systemd/system/prometheus.service
```
#### Add the following content
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

```
#### Reload systemd to apply the new service
```bash
sudo systemctl daemon-reload
```
#### Enable and start Prometheus service
```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
#### Check if the Prometheus Service is running
```bash
sudo systemctl status prometheus
```
## Acess the application in browser
```bash
http://ip-address:9090
```
#### Note if you use ec2 allow the port 9090 in security groups
#### If you use linux or ubuntu --- allow 9090 (sudo ufw allow 9090)
#
## Adding Targets in prometheus.yml
#### Configuring Prometheus to Scrape Node Exporter Metrics
#### Edit the prometheus.yml file
```bash
sudo nano /etc/prometheus/prometheus.yml
```
#### Add the Node Exporter targets under scrape_configs
```bash
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['ipaddress:9100']
```
##### localhost:9100=nodeexportor ipaddress and port
#
#### Restart Prometheus to apply the changes
```bash
sudo systemctl restart prometheus
```
#### Verify targets in Prometheus UI:
```bash
visit http://ip-address:9090/targets from a browser. 
```
#### In promethus ui - click the status and select targets herw you see a targets
#
## Configure Alerting Rules in Prometheus
#### Create Alert Rules
```bash
sudo nano /etc/prometheus/alert.rules.yml
```
#### add( sample configuration)
```bash
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage detected on {{ $labels.instance }}"
          description: "CPU usage is above 80% for more than 5 minutes."
```
#### Edit prometheus.yml
```bash
sudo nano /etc/prometheus/prometheus.yml
```
#### The file looks like this
```bash
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alert.rules.yml"  # Add all your rule files here

# Scrape configurations
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["ip-addess:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["ip-address:9100"]
```
#### Restart Prometheus
```bash
sudo systemctl restart prometheus
```
## Install and Configure Alertmanager
#### Download and Install Alertmanager
```bash
# Download Alertmanager
curl -LO https://github.com/prometheus/alertmanager/releases/latest/download/alertmanager-0.26.0.linux-amd64.tar.gz

# Extract and move binaries
tar xvf alertmanager-*.tar.gz
sudo mv alertmanager-*/alertmanager /usr/local/bin/
sudo mv alertmanager-*/amtool /usr/local/bin/
```
#### Configure Alertmanager
```bash 
sudo nano /etc/alertmanager/alertmanager.yml
```
#### add sample email notification
```bash
global:
  resolve_timeout: 5m

route:
  receiver: 'email_alert'
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 3h

receivers:
  - name: 'email_alert'
    email_configs:
      - to: 'your_email@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'   
        auth_username: 'your_email@example.com'
        auth_password: 'your_email_password'
        require_tls: true
```
#### Create an Alertmanager Service
```bash
sudo tee /etc/systemd/system/alertmanager.service <<EOF
[Unit]
Description=Alertmanager
After=network.target

[Service]
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
EOF
```
#### Start Alertmanager
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now alertmanager
```
#### Verify Alertmanager
```bash
systemctl status alertmanager
```
##  Install and Configure Grafana
#### Install Grafana
```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y grafana
```
#### Start and Enable Grafana
```bash
sudo systemctl enable --now grafana-server
```
#### Access Grafana
```bash
http://ip-address:3000
```
#### Note if you use ec2 allow port 3000 in inbound rules 
#### if you use linux/ubuntu allow 3000 (sudo ufw allow 3000)
#### Default login:
```bash
Username: admin
Password: admin
```
#### when you login it ask enter new password and update your password
#### Add Prometheus as a Data Source
##### Go to Configuration > Data Sources
##### Click Add data source
##### Select Prometheus
##### Set the promethus URL to
```bash 
http://ip-address:9090

```
##### Click Save & Test
#### Import Dashboards
```bash
Go to the Grafana Dashboard (on the right side, you will see the "New" option).
Click New, then select Create a Folder and name it as desired.
After creating the folder, select New and then click Import.
Enter a name for your dashboard.
In the Grafana.com Dashboard ID field, enter 1860 (this is the Node Exporter dashboard ID).
Click Load.
Select the previously created folder for organizing the dashboard.
Under Data Source, choose Prometheus.
Click Import to complete the process.
Finally you see the Dashboard.
```
