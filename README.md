# **Monitoring Project :smile:**
### In this we will do monitoring using Grafana and Prometheus of our 2-Tier-Flask-App-and-MYSQL Project that we did earlier. If you're interested in exploring my 2-Tier Flask App with MySQL project, to check out my project : [Click here](https://github.com/sudhajobs0107/2-Tier-Flask-App-and-MYSQL.git).
![Architecture](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/monitoring-diagram.gif)

___
# Prerequisites
### Before starting the project you should have these things in your system :-
>+ ### Account on AWS
>+ ### The application should be running
![App running](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/app-running.PNG)

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

+ ### Now we will make a docker-container.yml to run prometheus, redis & cAdvisor for this write a docker-compose file and run command **docker-compose up -d**.
+ ### Now if we will do **docker ps** we will see our prometheus, cadvisor & redis container is running.
![prom images](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/prom-images.PNG)

+ ### Now we saw prometheus container is running. So copy and paste **Public IPv4 address:9090** in a new tab and you will see :- 
![pro-running](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/pro-running.PNG)
+ ### Now we saw cadvisor container is running. So copy and paste **Public IPv4 address:8080** in a new tab and you will see :- 
![cadvisor](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/cAdvisor.PNG)

+ ### Now we want docker's log and our app container is already added in cAdvisor. So add cAdvisor code in prometheus.yml as given below :-
```
  - job_name: "docker"
    static_configs:
      - targets: ["cAdvisor:8080"]
```
+ ### Now once we have to restart prometheus container so that prometheus.yml will also update inside our prometheous container. To restart use command docker restart container_id.

+ ### Now if we do refresh prometheus page, we will see docker up.
![docker-up](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/docker-up.PNG)
+ ### Now if we want see cpu load average in graph, so Goto Graph → Write → rate(container_cpu_load_average_10s{name="node-app"}[5m]).
![graph](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/graph.PNG)

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
![grafana-active](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/grafana-active.PNG)
+ ### Now copy and paste **Public IPv4 address:3000** in a new tab and you will see grafana interface :- 
![login-interface](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/login-interface.PNG)
+ ### Now to login grafana we don't know username & password, so initial username and password for Grafana are username=admin & password=admin.
![dashboard-interface](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/dashboard-interface.PNG)
+ ### Now to visualize Prometheus metrics in Grafana, we have to add prometheus(data source) in grafana. So Goto Connection → Search & Click on "Prometheus" → Click "Add new data source" → In Connection Paste Prometheus URL → Save & Test.

___
## **Prometheus Dashboard**
+ ### Now we will make a dashboard to make it easier to view metrics, you can follow these steps :-

+ ### Click "Dashboard" → Click "Add visualization" → Select "Prometheus" → Select Metric "container_memory_usage_bytes" → Select Name of container "ubuntu_flaskapp_1" → Click "Run queries" → We will a visualization dashboard of our container memory.
 ![dashboard](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/dashboard.PNG)

 + ### Same we can see other metrics also like errors of app container.
 ![dashboard2](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/dashboard2.PNG)
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
![node-image](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/node-image.PNG)

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
![node-up](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/node-up.PNG)
+ ### Now Prometheus is already added in our connection in Grafana. So to view node metrics, we can import a pre-configured dashboard follow these steps :-
+ ### Click on Dashboard → Click "Import" → Paste id (e.g., code 1860) → Click "Load" → Select data source "Prometheus" → Click "Import".
+ ### Now we will see a Grafana dashboard set up to visualize metrics from Prometheus.
![dashboad3-node-exporter](https://github.com/sudhajobs0107/Monitoring-2-Tier/blob/main/images/dashboard3-node-exporter.PNG)


___
### Our Monitoring Project is completed :smile:.
___
