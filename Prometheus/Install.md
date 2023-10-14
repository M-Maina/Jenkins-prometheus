## Install Prometheus on Ubuntu 22.04


## Installation

To create a system user or system account

```bash
  sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus

```

You can use the curl or wget command to download Prometheus.
Check the version

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz


```


Then, we need to extract all Prometheus files from the archive.

```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz


```

Usually, you would have a disk mounted to the data directory. For this tutorial, I will simply create a /data directory. Also, you need a folder for Prometheus configuration files.

```bash
sudo mkdir -p /data /etc/prometheus


cd prometheus-2.47.1.linux-amd64/

```


let's move the Prometheus binary and a promtool to the /usr/local/bin/. promtool is used to check configuration files and Prometheus rules.

```bash
sudo mv prometheus promtool /usr/local/bin/

```

Optionally, we can move console libraries to the Prometheus configuration directory. Console templates allow for the creation of arbitrary consoles using the Go templating language. You don't need to worry about it if you're just getting started.

```bash
sudo mv consoles/ console_libraries/ /etc/prometheus/


```

Finally, let's move the example of the main Prometheus configuration file.

```bash
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

```

To avoid permission issues, you need to set the correct ownership for the /etc/prometheus/ and data directory.

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

```
You can delete the archive and a Prometheus folder when you are done.

```bash
cd ..
rm -rf prometheus-2.47.1.linux-amd64.tar.gz

prometheus --version

prometheus --help

```

We're going to use Systemd, which is a system and service manager for Linux operating systems. For that, we need to create a Systemd unit configuration file.
```bash
sudo vim /etc/systemd/system/prometheus.service

```

Prometheus.service
```bash
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


To automatically start the Prometheus after reboot, run enable.

```bash
sudo systemctl enable prometheus

sudo systemctl start prometheus

sudo systemctl status Prometheus

<public-ip:9090>


```
Suppose you encounter any issues with Prometheus or are unable to start it. 

```bash
journalctl -u prometheus -f --no-pager

```

## Install Node Exporter on Ubuntu 22.04

let's create a system user for Node Exporter  

```bash
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter

```
Use the wget command to download the binary.
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

```

Extract the node exporter from the archive.
```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz

```
Move binary to the /usr/local/bin.
```bash
sudo mv \
  node_exporter-1.6.0.linux-amd64/node_exporter \
  /usr/local/bin/

```

Clean up, and delete node_exporter archive and a folder.

```bash
rm -rf node_exporter*

node_exporter --version

node_exporter --help


```
create a similar systemd unit file.

```bash
sudo vim /etc/systemd/system/node_exporter.service

```
node_exporter.service

```bash
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
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind

[Install]
WantedBy=multi-user.target
```


```bash
sudo systemctl enable node_exporter

sudo systemctl start node_exporter

sudo systemctl status node_exporter


```
If you have any issues, check logs with journalctl

```bash
journalctl -u node_exporter -f --no-pager

```

To create a static target, you need to add job_name with static_configs.

```
sudo vim /etc/prometheus/prometheus.yml


```
prometheus.yml

```
- job_name: node_export
    static_configs:
      - targets: ["localhost:9100"]

```

Before, restarting check if the config is valid.

```
promtool check config /etc/prometheus/prometheus.yml

```

Then, you can use a POST request to reload the config.

```
curl -X POST http://localhost:9090/-/reload

http://<ip>:9090/targets


```

## Install Grafana on Ubuntu 22.04

Add all the dependencies
```
sudo apt-get install -y apt-transport-https software-properties-common


```

add the GPG key

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

```

Add this repository for stable releases.

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

sudo apt-get update

sudo apt-get -y install grafana

sudo systemctl enable grafana-server

sudo systemctl start grafana-server

sudo systemctl status grafana-server

http://<ip>:3000

```

## Let's Monitor JENKINS SYSTEM
Goto Manage Jenkins --> Plugins --> Available Plugins
Search for Prometheus and install it

To create a static target, you need to add job_name with static_configs.

```
sudo vim /etc/prometheus/prometheus.yml

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:8080']


```
Before, restarting check if the config is valid


```
promtool check config /etc/prometheus/prometheus.yml

curl -X POST http://localhost:9090/-/reload

http://<ip>:9090/targets

Use Id 9964 and click on load
```
Click On Dashboard --> + symbol --> Import Dashboard

Use Id 9964 and click on load




