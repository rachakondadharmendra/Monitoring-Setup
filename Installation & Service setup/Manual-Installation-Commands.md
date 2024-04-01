# Manual Installation :

```
#Prometheus

mkdir /etc/prometheus /var/lib/prometheus
useradd --no-create-home --home-dir / --shell /bin/false prometheus
chown -R prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar -zxvf prometheus-*.tar.gz; rm -rf prometheus-*.tar.gz
cd prometheus-*
cp prometheus promtool /usr/local/bin
cp prometheus.yml /etc/prometheus
cp -r consoles console_libraries /etc/prometheus


#Alert Manager
wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
tar -zxvf alertmanager-*.tar.gz; rm -rf alertmanager-*.tar.gz
cd alertmanager-*
cp alertmanager /usr/local/bin
mkdir /etc/alertmanager /var/lib/alertmanager
cp alertmanager.yml /etc/alertmanager
useradd --no-create-home --home-dir / --shell /bin/false alertmanager
chown -R alertmanager:alertmanager /var/lib/alertmanager

#Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xf node_exporter-*.tar.gz; rm -rf node_exporter-*.tar.gz
cd node_exporter-*
cp node_exporter /usr/local/bin
useradd --no-create-home --home-dir / --shell /bin/false node_exporter

#Grafana [Refere Official page for installation https://grafana.com/grafana/download ]
sudo apt-get install -y adduser libfontconfig1 musl  
wget [https://dl.grafana.com/enterprise/release/grafana-enterprise_10.1.1_amd64.deb](https://dl.grafana.com/enterprise/release/grafana-enterprise_10.1.1_amd64.deb)  
sudo dpkg -i grafana-enterprise_10.1.1_amd64.deb

```

```
systemctl start grafana; systemctl status grafana

#Grafana Template code : 15172

```
#### Creation of Service:

```
#Creating a new service on linux.

- Navigate to systemd/system folder
cd /etc/systemd/system
vi <service-name>.service
or 
vi /etc/systemd/system/<service-name.service

```

- Prometheus service file
##### /etc/systemd/system/prometheus.service 
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.listen-address=0.0.0.0:9090
ExecReload=/bin/kill -HUP $MAINPID
ProtectHome=true
ProtectSystem=full

[Install]
WantedBy=default.target

```

```
systemctl daemon-reload; systemctl start prometheus; systemctl status prometheus;
```
- Node Exporter
#### /etc/systemd/system/nodeexporter.service 
```bash
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
```

```
systemctl daemon-reload; systemctl start nodeexporter; systemctl status nodeexporter;
```

- Alert Manager 
##### /etc/systemd/system/alertmanager.service
```bash
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
```

```
systemctl daemon-reload; systemctl start alertmanager; systemctl status alertmanager;
```

- Commands to check the working of the **service files**

```
systemctl daemon-reload; systemctl start alertmanager; systemctl status alertmanager;
systemctl daemon-reload; systemctl start nodeexporter; systemctl status nodeexporter;
systemctl daemon-reload; systemctl start prometheus; systemctl status prometheus;
systemctl start grafana; systemctl status grafana
```
```
sudo systemctl enable alertmanager
sudo systemctl enable nodeexporter 
sudo systemctl enable prometheus
sudo systemctl enable grafana
sudo systemctl list-unit-files --type=service --state=enabled
```


