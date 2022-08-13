# Install Prometheus (ubuntu) and scrap metrics for Astra Streaming

## Congfigure Prometheus
```
$ sudo apt update
```
```
Hit:1 http://repomirror.sjc.dsinternal.org/ubuntu bionic InRelease
Hit:2 
```

Download Prometheus
[Download \| Prometheus](https://prometheus.io/download/)

Downloading it in /opt/prometheus directory previously created
```
$ opt/prometheus > curl -LO url -LO https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz
```
Untar downloaded file
```
$ /opt/prometheus> tar -xvf prometheus-2.37.0.linux-amd64.tar.gz
```
Access newly directory created after untaring it:
```
$ /opt/prometheus> cd prometheus-2.37.0.linux-amd64/
```
List content of prometheus directory 
```
$ /opt/prometheus/prometheus-2.37.0.linux-amd64> ls -la
```
```
-rw-r--r-- 1 automaton automaton     11357 Jul 14 15:30 LICENSE
-rw-r--r-- 1 automaton automaton      3773 Jul 14 15:30 NOTICE
drwxr-xr-x 2 automaton automaton      4096 Jul 14 15:30 console_libraries
drwxr-xr-x 2 automaton automaton      4096 Jul 14 15:30 consoles
-rwxr-xr-x 1 automaton automaton 109655433 Jul 14 15:16 prometheus
-rw-r--r-- 1 automaton automaton       934 Jul 14 15:30 prometheus.yml
-rwxr-xr-x 1 automaton automaton 101469395 Jul 14 15:18 promtool
```


Make /data directory for prometheus
```
$ /opt/prometheus/prometheus-2.37.0.linux-amd64> sudo mkdir data
```

Change ownership to the created data directory
```
$ /opt/prometheus/prometheus-2.37.0.linux-amd64> sudo chown -R automaton:automaton data
```

Create service file for prometheus
```
$ sudo vi /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=automaton
Group=automaton
Type=simple
ExecStart=/opt/prometheus/prometheus-2.37.0.linux-amd64/prometheus \
    --config.file /opt/prometheus/prometheus-2.37.0.linux-amd64/prometheus.yml \
    --storage.tsdb.path /opt/prometheus/prometheus-2.37.0.linux-amd64/data \
    --web.console.templates=/opt/prometheus/prometheus-2.37.0.linux-amd64/consoles \
    --web.console.libraries=/opt/prometheus/prometheus-2.37.0.linux-amd64/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload the system service to register prometheus service
```
$ sudo systemctl daemon-reload
```
Start prometheus service
```
$ sudo systemctl start prometheus
```
Check prometeus service status
```
$ sudo systemctl status prometheus
```
```
● prometheus.service - Prometheus
   Loaded: loaded (/etc/systemd/system/prometheus.service; disabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-08-13 07:21:29 UTC; 11s ago
 Main PID: 25413 (prometheus)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/prometheus.service
           └─25413 /opt/prometheus/prometheus-2.37.0.linux-amd64/prometheus --config.file /opt/prometheus/prom
```

additionally, in order to turn service on next reboot
```
sudo systemctl enable prometheus
```



Go to your favourity browser to open the GUI:
```
http://<YOUR IP>:9090
````

![Screen Shot 2022-08-13 at 8.46.31 AM.png](:storage/1ec698cc-e5b8-4beb-9c95-2590270ccd6b/dda2b431.png)

# Set up Astra Streaming

Go to your Astra instance and get prometheus configuration details for Astra Streaming. Download the configuration file in order to get your secret credentials

![Screen Shot 2022-08-13 at 8.54.03 AM.png](:storage/1ec698cc-e5b8-4beb-9c95-2590270ccd6b/97d3145f.png)


This is my downloaded Astra Streaming configuration file for prometheus:

```
global:
  scrape_interval: 60s # Set the scrape interval to every 60 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

scrape_configs:
  - job_name: "astra-pulsar-metrics"

    scheme: 'https'
    metrics_path: '/pulsarmetrics/ivantenant'
    authorization:
      credentials: 'eyJhbGciOiJSU....'

    static_configs:
      - targets: ['prometheus-gcp-useast1.streaming.datastax.com']
```

Modify prometheus.yaml

prometheus.yml before changes
```
$ /opt/prometheus/prometheus-2.37.0.linux-amd64$ cat prometheus.yml
```
```
# my global config
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
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```


prometheus.yml after changes
```
$ /opt/prometheus/prometheus-2.37.0.linux-amd64$ vi prometheus.yml
```
```
# my global config
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
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  # Astra Streaming job
  - job_name: "astra-pulsar-metrics"

    scheme: 'https'
    metrics_path: '/pulsarmetrics/ivantenant'
    authorization:
      credentials: 'eyJhbGciOiJSU....'

    static_configs:
      - targets: ['prometheus-gcp-useast1.streaming.datastax.com']
```


## Testing Prometheus for Astra Streaming:

First, go to your Astra Streaming GUI and create a tenant, namespace and topic.

Second in Astra Streaming GUI go to "Try" Me tab and send a message:

![Screen Shot 2022-08-13 at 9.45.17 AM.png](:storage/1ec698cc-e5b8-4beb-9c95-2590270ccd6b/98dbbe2f.png)


Third go to prometheus GUI and in the graph tab  within the search bar start typing pulsar, all the scrapped metrics available can be seen at this stage, choose one to visualise it:

![Screen Shot 2022-08-13 at 9.47.42 AM.png](:storage/1ec698cc-e5b8-4beb-9c95-2590270ccd6b/6ced41dc.png)