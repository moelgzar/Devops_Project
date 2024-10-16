# Promeithes monitoring

### We will create vm for prometheus and grafana
- after create vm we will install prometheus and grafana

## **Install prometheus** 

```bash
sudo apt update
```

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux
amd64.tar.gz
```
```bash
tar -xvf prometheus-2.37.0.linux-amd64.tar.gz
```

```bash
 cd prometheus-2.37.0.linux-amd64/
 ```
```bash
./prometheus
```
- now prometheus is running on port 9090 ( `http://localhost:9090` )

## Prometheus Installation SystemD 
1. **Create user prometheus**

```bash
 sudo useradd--no-create-home --shell /bin/false prometheus
 ```
2. **Create directory for prometheus**
```bash
 sudo mkdir /etc/prometheus
  sudo mkdir /var/lib/prometheus
```
3. **cChange permissions**
```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```  
4. **Copy prometheus binaries**
```bash
 sudo cp prometheus /usr/local/bin/
 sudo cp promtool /usr/local/bin/
```
5. **Change permissions for prometheus user**
```bash
 sudo chown prometheus:prometheus /usr/local/bin/prometheus
 sudo chown prometheus:prometheus /usr/local/bin/promtool
```
6. **Copy consoles and console_libraries**
```bash
 sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
``` 
7. **Change permissions for  directory andall of the files within it**
```bash
 sudo chown -R prometheus:prometheus /etc/prometheus/consoles
```
```bash
sudo chown-R prometheus:prometheus /etc/prometheus/console_libraries
``` 
8. **Copy prometheus.yaml**
```bash
 sudo cp prometheus.yaml /etc/prometheus/prometheus.yml
 ```
9. **Change permissions**
```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml 
```
10. **Start prometheus**

```bash
 sudo -u prometheus /usr/local/bin/prometheus \--config.file /etc/prometheus/prometheus.yml \--storage.tsdb.path /var/lib/prometheus/ \--web.console.templates=/etc/prometheus/consoles \--web.console.libraries=/etc/prometheus/console_libraries 
 ```
11. **Open prometheus service to check** 
```bash
sudo vi /etc/systemd/system/prometheus.service
```
12. **Reload service**
```bash
sudo systemctl daemon-reload
```
12. **Start service**
```bash
sudo systemctl start prometheus
```
13. **Enable service**
```bash
sudo systemctl enable prometheus
```
14. **Check status**
```bash
sudo systemctl status prometheus
```
- now prometheus is running on port 9090 ( `http://localhost:9090` )

##  Blackbox exporter

1. **Download blackbox exporter**
```bash
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
```
2. **Extract blackbox exporter**
```bash
tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
```

```bash 
cd blackbox_exporter-0.25.0.linux-amd64/
```
3. **Start blackbox exporter in background**
```bash
./blackbox_exporter &
```
- now blackbox exporter is running on port 9115 ( `http://localhost:9115` )

## **Now we will edit in yaml file for prometheus**
1. **open prometheus.yaml file**
```bash
sudo vi /etc/prometheus/prometheus.yml
```
2. **Add blackbox exporter in `prometheus.yaml` file**
```bash
global:
  scrape_interval: 15s  # Default scrape interval
  evaluation_interval: 15s  # Default evaluation interval

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - <my_application_ip:port>
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <black_box_exporter_machine_ip:9115>

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<node_exporter_ip>:9100'] #these exporter for jenkins

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:  
      - targets: ['<jenkins_server_ip>:8080']
```
- In yaml file i added
 1. - blackbox exporter
 2. - jenkins jobs and 
 3. - node exporter 

3. **Kill prometheus service**
```bash
 ps aux | grep prometheus
```

```bash
sudo kill -9 PID
```
4. **Restart prometheus service**
```bash
sudo systemctl restart prometheus
```
- Now go to `http://localhost:9090` and check the blackbox exporter and node exporter and jenkins server 



# Install grafana

### **Add repo for grafana**
```bash
sudo apt update
sudo apt install -y software-properties-common wget
sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```
```bash
sudo apt update
sudo apt install -y grafana
```
### **Start grafana service**    

```bash
sudo systemctl start grafana-server
```
```bash
sudo systemctl status grafana-server
```
- now grafana is running on port 3000 ( `http://localhost:3000` )
- the default username  is `admin` and passsword is  `admin`
### **import grafana dashboard**
1. **go to `http://localhost:3000` and login with default username and password**
2. **click on `+` icon and then click on `Import`**
3. **paste id of dashboard `9614` and click on `Load`**
4. **select prometheus as data source and click on `Import`**
- now blackbox dashboard is imported and you can see cpu and ram usage of your application

5. **import node exporter dashboard for jenkins server**
- go to `http://localhost:3000` and login with default username and password
- click on `+` icon and then click on `Import`
- paste id of dashboard `11074` and click on `Load`
- select prometheus as data source and click on `Import`
- now node exporter dashboard is imported and you can see cpu and ram usage of your jenkins server