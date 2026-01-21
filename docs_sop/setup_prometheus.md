## How to install Prometheus Server on Ubuntu 24

Add a system user
```
useradd --no-create-home --shell /bin/false prometheus
```

Create a system directories on /etc and /var/lib 
```
mkdir /etc/prometheus/
mkdir /var/lib/prometheus/
```

Modify the directories ownership
```
chown prometheus:prometheus /etc/prometheus/
chown prometheus:prometheus /var/lib/prometheus/
```

Download the binary file
```
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```

Extract the archive file
```
tar xvf prometheus-3.5.0.linux-amd64.tar.gz 
```

Place the binary files on the right path
```
cd prometheus-3.5.0.linux-amd64/
cp prometheus /usr/local/bin/
cp promtool /usr/local/bin/
cp prometheus.yml /etc/prometheus/
```

Modify the the files ownership
```
chown prometheus:prometheus /usr/local/bin/prometheus 
chown prometheus:prometheus /usr/local/bin/promtool 
chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Create the daemon file for prometheus service
```
vim /etc/systemd/system/prometheus.service
```

Paste the following lines of code:

```
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
    --web.console.libraries=/etc/prometheus/console_libraries \
    --storage.tsdb.retention.time=30d\
    --web.enable-admin-api 

    [Install]
    WantedBy=multi-user.target
```

Enable and restart the Prometheus service
```
systemctl daemon-reload 
systemctl enable prometheus.service  --now
```

To the check the syntax of prometheus.yml file
```
promtool check config /etc/prometheus/prometheus.yml
```

To allow the Prometheus port on Firewalld
```
firewall-cmd --per --add-port=9090/tcp
firewall-cmd --reload
```