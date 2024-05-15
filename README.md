# Install Prometheus
  #### install binary package for prometheus
  ```bash
  wget https://github.com/prometheus/prometheus/releases/download/v2.40.1/prometheus-2.40.1.linux-amd64.tar.gz
  ```

  #### etract tar file 
  ```bash
  tar xvf prometheus-2.40.1.linux-amd64.tar.gz
  ```

  #### go into the directory
  ```bash
  cd prometheus-2.40.1.linux-amd64
  ```
  
  #### run manually
  ```bash
  ./prometheus 
  ./prometheus > /dev/null 2>&1 &
  ```
  

# Create a service for prometheus
  #### create a user specifically for prometheus
  ```bash
  useradd --no-create-home --shell /bin/false prometheus
  ```
  
  #### create directories
  ```bash
  mkdir /etc/prometheus
  mkdir /var/lib/prometheus
  ```

  #### change permissions
  ```bash
  chown prometheus:prometheus /etc/prometheus
  chown prometheus:prometheus /var/lib/prometheus
  ```
  
  #### rename directory for convenience
  ```bash
  mv prometheus-2.40.1.linux-amd64 prometheuspackage
  ```

  #### copy executable/binary files to bin directory
  ```bash
  cp prometheuspackage/prometheus /usr/local/bin/
  cp prometheuspackage/promtool /usr/local/bin/
  ```

  #### change permissions
  ```bash
  chown prometheus:prometheus /usr/local/bin/prometheus
  chown prometheus:prometheus /usr/local/bin/promtool
  ```
  
  #### copy library and template files tpo etc 
  ```bash
  cp -r prometheuspackage/consoles /etc/prometheus
  cp -r prometheuspackage/console_libraries /etc/prometheus
  ```

  #### change ownership
  ```bash
  chown -R prometheus:prometheus /etc/prometheus/consoles
  chown -R prometheus:prometheus /etc/prometheus/console_libraries
  ```

  #### create config file for peometheus
  ```bash
  vim /etc/prometheus/prometheus.yml
  ```
  Add the following configuration to the file.

  ```yaml
  global:
    scrape_interval: 10s

  scrape_configs:
    - job_name: 'prometheus_master'
      scrape_interval: 5s
      static_configs:
        - targets: ['localhost:9090']
  ```

  #### change permissions
  ```bash
  chown prometheus:prometheus /etc/prometheus/prometheus.yml
  ```
  
  #### create service for prometheus
  ```bash
  vim /etc/systemd/system/prometheus.service
  ```
  Define the service as follows,

  ```text
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

  Reload the systemd service.
  #### reload daemon, to update changes
  ```bash
  systemctl daemon-reload
  ```

  #### Start prometheus
  ```bash
  systemctl start prometheus
  ```

  #### Check service status.
  ```bash
  systemctl status prometheus
  ```


# Install Node Exporter

  ```bash
  wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
  ```
  ```bash
  tar xvf node_exporter-1.4.0.linux-amd64.tar.gz 
  ```
  ```bash
  cd node_exporter-1.4.0.linux-amd64/
  ```
  ```bash
  ./node_exporter > /dev/null 2>&1 &
  ```


