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
    