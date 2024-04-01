# Basic Prometheus Usage 


```
promtool check config /etc/prometheus/prometheus.yml
```

#### /etc/prometheus/prometheus.yml >> File Path

```yaml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9091"]

  - job_name: "UAT_SERVERS"
    static_configs:
      - targets:
        - "189.65.65.48:9090"
        - "189.65.65.49:9090"
        - "189.65.65.42:9090"

  - job_name: "QA_SERVERS"
    static_configs:
      - targets:
        - "189.65.65.41:9090"

# You get the point to use job_name by now (Hopefully)
```
