# Bash scripting using function call

```

install_prometheus() { 

PROMETHEUS_VERSION=$(curl -s https://raw.githubusercontent.com/prometheus/prometheus/master/VERSION)

wget -q https://github.com/prometheus/prometheus/releases/download/v${PROMETHEUS_VERSION}/prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz -P $HOME/monitoring_setup/prometheus_setup

cd $HOME/monitoring_setup/prometheus_setup
tar -zxf prometheus-*.tar.gz; rm -rf prometheus-*.tar.gz
cd prometheus-*


mkdir /etc/prometheus
mkdir /var/lib/prometheus
touch /etc/prometheus/prometheus.yml  


useradd --no-create-home --home-dir / --shell /bin/false prometheus

cp prometheus /usr/local/bin/
cp promtool /usr/local/bin/
cp -r consoles /etc/prometheus
cp -r console_libraries /etc/prometheus
cp -r prometheus.yml /etc/prometheus

# Change ownership of Prometheus directories and files
chown -R prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
chown prometheus:prometheus /etc/prometheus/prometheus.yml


bash -c 'cat << EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=on-failure
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

systemctl daemon-reload; systemctl start prometheus; systemctl status prometheus; sudo systemctl enable prometheus

}


```


```

install_node_exporter() { 

NODE_VERSION=$(curl -s https://raw.githubusercontent.com/prometheus/node_exporter/master/VERSION)

wget -q https://github.com/prometheus/node_exporter/releases/download/v${NODE_VERSION}/node_exporter-${NODE_VERSION}.linux-amd64.tar.gz -P $HOME/monitoring_setup/nodeexporter_setup

cd $HOME/monitoring_setup/nodeexporter_setup
tar -zxf node_exporter-*.tar.gz; rm -rf node_exporter-*.tar.gz
cd node_exporter-*
useradd --no-create-home --home-dir / --shell /bin/false node_exporter
cp node_exporter /usr/local/bin
chown -R node_exporter:node_exporter /usr/local/bin/node_exporter

bash -c 'cat << EOF > /etc/systemd/system/nodeexporter.service 
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter \
	   --web.listen-address=0.0.0.0:9100

SyslogIdentifier=node_exporter
Restart=always

PrivateTmp=yes
ProtectHome=yes
NoNewPrivileges=yes

ProtectSystem=strict
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=yes

[Install]
WantedBy=multi-user.target
EOF'


systemctl daemon-reload; systemctl start nodeexporter; systemctl status nodeexporter; systemctl enable nodeexporter 

}
```

```
install_alertmanager() { 
ALERTMANAGER_VERSION=$(curl -s https://raw.githubusercontent.com/prometheus/alertmanager/master/VERSION)

wget -q https://github.com/prometheus/alertmanager/releases/download/v${ALERTMANAGER_VERSION}/alertmanager-${ALERTMANAGER_VERSION}.linux-amd64.tar.gz -P $HOME/monitoring_setup/alertmanager_setup


# Alert Manager
cd $HOME/monitoring_setup/alertmanager_setup
tar -zxf alertmanager-*.tar.gz; rm -rf alertmanager-*.tar.gz
cd alertmanager-*
mkdir  /etc/alertmanager /var/lib/alertmanager
cp alertmanager /usr/local/bin
cp alertmanager.yml /etc/alertmanager
useradd --no-create-home --home-dir / --shell /bin/false alertmanager
chown -R alertmanager:alertmanager /var/lib/alertmanager

bash -c 'cat << EOF > /etc/systemd/system/alertmanager.service 
[Unit]
Description=Alertmanager for prometheus
After=network.target

[Service]
User=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager/ \
  --web.listen-address=0.0.0.0:9093
ExecReload=/bin/kill -HUP $MAINPID

NoNewPrivileges=true
ProtectHome=true
ProtectSystem=full

[Install]
WantedBy=multi-user.target
EOF'

systemctl daemon-reload; systemctl start alertmanager; systemctl status alertmanager; systemctl enable alertmanager



}
```

```
install_grafana() {
  if [[ -f /etc/os-release ]]; then
    . /etc/os-release
    OS=$ID
  elif [[ -f /etc/redhat-release ]]; then
    OS="rhel"
  else
    echo "Unsupported OS"
    return 1
  fi

  if [[ $OS == "ubuntu" ]]; then
    apt-get update
    apt-get install -y apt-transport-https
    apt-get install -y software-properties-common wget
    wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
    echo "deb https://packages.grafana.com/oss/deb stable main" | tee -a /etc/apt/sources.list.d/grafana.list
    apt-get update
    apt-get install -y grafana

  elif [[ $OS == "amzn" ]]; then
    yum install -y https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.3.4-1.x86_64.rpm

  elif [[ $OS == "rhel" ]]; then
    yum install -y https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.3.4-1.x86_64.rpm

  else
    echo "Unsupported OS $OS"
    return 1
  fi
}
```
