# Install Prometheus
### Create user
`sudo useradd --no-create-home --shell /bin/false prometheus`

`sudo mkdir /etc/prometheus`
`sudo mkdir /var/lib/prometheus`

### Get Binary
Download latest binary for your device (OS, Arch)
https://prometheus.io/download/
Extract with tar: tar -xvzf prometheus-3.4.2.linux-arm64.tar.gz
### Copy binary 
`sudo cp prometheus-3.4.2.linux-arm64/prometheus /usr/local/bin/`

`sudo chown prometheus:prometheus /usr/local/bin/prometheus`

`sudo chown prometheus:prometheus /var/lib/prometheus`

### Config file
`sudo nano /etc/prometheus/prometheus.yml`
```
# Location: /opt/prometheus/prometheus.yml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

remote_write:
- url: <your_cloud_api_receiver>
  basic_auth:
    username: <your_username>
    password: <your_password>
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    scrape_interval: 30s
    scrape_timeout: 10s
    static_configs:
      - targets: ['localhost:9100']
```
# Create systemd service
`sudo nano /etc/systemd/system/prometheus.service`
```
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prome
Group=prome
Restart=on-failure

ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.agent.path=/var/lib/prometheus/ \
  --storage.agent.no-lockfile \
  --agent

[Install]
WantedBy=multi-user.target
```

`sudo systemctl daemon-reload`
`sudo systemctl restart prometheus`
`sudo systemctl status prometheus`


# Node 
## Create user
`sudo useradd --no-create-home --shell /bin/false node_exporter`
### Get Binary
https://github.com/prometheus/node_exporter/releases
### Move 
Move the node_exporter binary to /usr/local/bin/
### Create systemd service
`sudo nano /etc/systemd/system/node_exporter.service`

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

`sudo systemctl daemon-reload`
`sudo systemctl restart node_exporter`
`sudo systemctl status node_exporter`

### Test
`curl localhost:9100/metrics`
