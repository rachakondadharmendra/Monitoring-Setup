# Install via bash script with some initial manual setup

```
mkdir grafana_monitor_setup_scripts ; cd grafana_monitor_setup_scripts;
touch prometheus_setup.sh && chmod 755 prometheus_setup.sh; vi prometheus_setup.sh
```


Prometheus shell script && usage

```
# Check if the grafana_monitor_setup directory exists
if [[ ! -d grafana_monitor_setup ]]; then
  # Create the directory
  mkdir ~/grafana_monitor_setup
  echo -e "Step $step: Created the grafana_monitor_setup directory.\n"
fi
# Change directory to the grafana_monitor_setup directory
mkdir ~/grafana_monitor_setup/logs/; touch ~/grafana_monitor_setup/logs/prometheus_logs
./prometheus_setup.sh > grafana_monitor_setup/logs/prometheus_logs 2>&1

```

```bash

#!/bin/bash
set -e
# Set a variable to store the step that we are currently on
step=1

# Start the script
sudo su
cd ~/grafana_monitor_setup

# Create the /etc/prometheus and /var/lib/prometheus directories
mkdir /etc/prometheus /var/lib/prometheus
echo -e "Step $step: Created the /etc/prometheus and /var/lib/prometheus directories.\n"

# Add the prometheus user
useradd --no-create-home --home-dir / --shell /bin/false prometheus
echo -e "Step $step: Added the prometheus user.\n"

# Download the Prometheus binary
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
echo -e "Step $step: Downloaded the Prometheus binary.\n"

# Extract the Prometheus binary
tar -zxvf prometheus-*.tar.gz; rm -rf prometheus-*.tar.gz
echo -e "Step $step: Extracted the Prometheus binary.\n"

# Move the Prometheus binary and configuration files to their respective locations
cd prometheus-*
cp prometheus promtool /usr/local/bin
cp prometheus.yml /etc/prometheus
cp -r consoles console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus/prometheus.yml
echo -e "Step $step: Moved the Prometheus binary and configuration files to their respective locations.\n"

# Create the Prometheus service file
bash -c 'cat << EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
ExecStart=/usr/l
ocal/bin/prometheus \\
  --config.file /etc/prometheus/prometheus.yml \\
  --storage.tsdb.path /var/lib/prometheus/ \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries
  --web.listen-address=0.0.0.0:9090
ExecReload=/bin/kill -HUP $MAINPID
ProtectHome=true
ProtectSystem=full

[Install]
WantedBy=multi-user.target
EOF'
echo -e "Step $step: Created the Prometheus service file.\n"

# Start the Prometheus service
sudo systemctl start prometheus
echo -e "Step $step: Started the Prometheus service.\n"

# Enable the Prometheus service to start automatically at boot time
sudo systemctl enable prometheus
echo -e "Step $step: Enabled the Prometheus service to start automatically at boot time.\n"

# Exit the script
exit 0

```
