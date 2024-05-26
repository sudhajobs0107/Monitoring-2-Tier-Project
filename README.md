# **Monitoring Project :smile:**
### In this we will do monitoring using Grafana and Prometheus of our 2-Tier-Flask-App-and-MYSQL Project that we did earlier. If you're interested in exploring my 2-Tier Flask App with MySQL project, check out my project here: [Click here](https://github.com/sudhajobs0107/2-Tier-Flask-App-and-MYSQL.git).
![Architecture](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/architecture.png)

___
# Prerequisites
### Before starting the project you should have these things in your system :-
>+ ### Account on AWS
>+ ### The application should be running
![App running](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/architecture.png)

___
# **Part 1** : **Install Prometheus**
+ ### Prometheus we use for metrics & alerting. Now make sure docker.io & docker-compose is installed. Now to configure **Prometheus & cAdvisor** use command as given below [cAdvisor (short for container Advisor) analyzes and exposes resource usage and performance data from running containers. cAdvisor exposes Prometheus metrics out of the box] :-
```
sudo apt-get update
wget https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/examples/prometheus.yml
```
+ ### Now to see prometheus.yml use command :-
```
ls
```
![prom.yml](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/zip.PNG)

+ ### Now we will make a docker-container to run prometheus, redis & cAdvisor for this write a docker-compose file and run command **docker-compose up -d**.
```
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```
+ ### Now if we will do **docker ps** we will see our prometheus, cadvisor & redis container is running.
![prom images](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/zip.PNG)

+ ### Now we saw prometheus container is running. So copy and paste **Public IPv4 address:9090** in a new tab and you will see :- 
![pro-running](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)
+ ### Now we saw cadvisor container is running. So copy and paste **Public IPv4 address:8080** in a new tab and you will see :- 
![cadvisor](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)

+ ### Now we want docker's log and our app container is already added in cAdvisor. So add cAdvisor code in prometheus.yml as given below :-
```
  - job_name: "docker"
    static_configs:
      - targets: ["cAdvisor:8080"]
```
+ ### Now once we have to restart prometheus container so that prometheus.yml will also update inside our prometheous container. To restart use command docker restart container_id.

+ ### Now if we do refresh prometheus page, we will see docker up.
![docker-up](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/metrics.PNG)
+ ### Now if we want see cpu load average in graph, so Goto Graph → Write → rate(container_cpu_load_average_10s{name="node-app"}[5m]).
![graph](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/metrics.PNG)

___
# **Part 2** : Install **Grafana** and set it up to **Work with Prometheus**
+ ### Grafana we use for visualization. Now to install grafana on Ubuntu, use command :-
```
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
+ ### Update the package list and install Grafana, use command :-
```
sudo apt-get update
sudo apt-get install grafana
```
+ ### To enable and Start Grafana Service, use command :-
```
sudo systemctl enable grafana-server
```
```
sudo systemctl start grafana-server
```
+ ### To check Grafana Status, use command :-
```
sudo systemctl status grafana-server
```
![grafana-active](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/zip.PNG)
+ ### Now copy and paste **Public IPv4 address:3000** in a new tab and you will see grafana interface :- 
![login-interface](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)
+ ### Now to login grafana we don't know username & password, so initial username and password for Grafana are username=admin & password=admin.
![dashboard-interface](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)
+ ### Now to visualize Prometheus metrics in Grafana, we have to add prometheus(data source) in grafana. So Goto Connection → Search & Click on "Prometheus" → Click "Add new data source" → In Connection Paste Prometheus URL → Save & Test.

___
## **Prometheus Dashboard**
+ ### Now we will make a dashboard to make it easier to view metrics, you can follow these steps :-

+ ### Click "Dashboard" → Click "Add visualization" → Select "Prometheus" → Select Metric "container_memory_usage_bytes" → Select Name of container "ubuntu_flaskapp_1" → Click "Run queries" → We will a visualization dashboard of our container memory.
 ![dashboard](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)

 + ### Same we can see other metrics also like errors of app container.
 ![dashboard2](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/promethus-page.PNG)
+ ### We've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.

___
## **Node Exporter**
+ ### Node Exporter we use for to scrape detailed servers/systems metrics. Now we want node exporter in the network of prometheus,so add given code in the prometheus's docker-compose file :-
```
node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - 9100
```
+ ### Now run command **docker-compose up -d** and if we will do **docker ps** we will see our node-exporter container is running.
![node-image](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/zip.PNG)

+ ### Now to configure Prometheus to scrape metrics from Node Exporter, we need to modify the prometheus.yml file so :-
```
vim prometheus.yml
```
+ ### And in the prometheus.yml file add as given below :-
```
  - job_name: "NodeExporter"
    static_configs:
      - targets: ["node-exporter:9100"]
```
+ ### Now once restart prometheus container.
+ ### Now if we do refresh prometheus page, we will see node-exporter up.
![node-up](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/metrics.PNG)
+ ### Now Prometheus is already added in our connection in Grafana. So to view node metrics, we can import a pre-configured dashboard follow these steps :-
+ ### Click on Dashboard → Click "Import" → Paste id (e.g., code 1860) → Click "Load" → Select data source "Prometheus" → Click "Import".
+ ### Now we will see a Grafana dashboard set up to visualize metrics from Prometheus.
![dashboad3-node-exporter](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/node.PNG)
+ ### We can also give stress on cpu and check the metrics. So run command :-
### 1. sudo apt install stress
### 2. stress -c 60 --timeout 10

___
## **Loki & Promtail**
+ ### Loki we use for log aggregation(the process of gathering logs from multiple sources and centralizing them in one location for easier management, analysis and storage). Promtail is a scraper, we use it for to collect logs from various sources and ship them to Loki for aggregation and storage. It acts similarly to how Logstash or Fluentd work in an ELK stack. Promtail is designed to work perfectly with Loki, making it a key component of the Loki system.
+ ### Now to configure **Loki** use command as given below :-
```
wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml
```
+ ### Now to configure **Promtail** use command as given below :-
```
wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```
+ ### Now we will run loki docker-container, for this run command:-
```
docker run -d --name loki -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:2.8.0 --config.file=/mnt/config/loki-config.yaml
```
+ ### Now we will run promtail docker-container, for this run command:-
```
docker run -d --name promtail -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:2.8.0 --config.file=/mnt/config/promtail-config.yaml
```
+ ### Now if we will do **docker ps** we will see our loki & promtail container is running.
![loki&prom-images](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/zip.PNG)
+ ### Now to visualize Loki logs in Grafana, we have to add loki(data source) in grafana. So Go To Connection → Search & Click on "Loki" → Click "Add new data source" → In Connection Paste Loki URL → Save & Test.
+ ### Suppose we will want to see nginx's logs in grafana. So install nginx :-
```
sudo apt-get install nginx
```
+ ### Open port no. 80. Copy & paste **Public IPv4 address:80** in a new tab and you will see :-
![nginx](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/node.PNG)
+ ### Now if we do Public IPv4 address:80/hello so we will get error 404 Not Found. So in system nginx's logs will store in path : /var/log/nginx. Copy this path and add a job in promtail.yml as given below :-
```
- job_name: nginx
  static_configs:
  - targets:
      - localhost
    labels:
      job: nginxlogs
      __path__: /var/log/nginx/*
```
+ ### Now once restart promtail container. So to view logs, we can import a pre-configured dashboard follow these steps :-
+ ### Click on Dashboard → Click "New" & "Import" → Paste id (e.g., code 1860) → Click "Load" → Select data source "Loki" → Click "Import".
+ ### Now we will see a Grafana dashboard set up to visualize logs from Loki.
![logs](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/public/assets/node.PNG)


___
### Our Monitoring Project is completed :smile:.
___
