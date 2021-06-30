<p align="center">
  <img width="600" height="170" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/pmm.png">
</p>

<p align="center">
  <b>Percona Monitoring and Management in Docker-compose</b>
</p>

***

This a little guide how to deploy and setting [Percona Monitoring and Management (PMM)](https://www.percona.com/doc/percona-monitoring-and-management/index.html) in docker-compose.<br>
Percona Monitoring and Management (PMM) is an open-source platform for managing and monitoring MySQL and MongoDB performance.<br> 
It is developed by Percona in collaboration with experts in the field of managed database services, support and consulting.
<br>
PMM is a free and open-source solution that you can run in your own environment for maximum security and reliability. It provides thorough time-based analysis for MySQL and MongoDB servers to ensure that your data works as efficiently as possible.
***

## GETTING A PMM SERVER RUNNING ON DOCKER IS JUST MATTER OF FOLLOWING A FEW SIMPLE STEPS.
If you just run the docker-compose file it will not work correctly due to incorrect container initialization, therefore:
<br>
- the first step that you will need to do create the ```pmm-data container``` with default values: 
```c
docker create \
   -v /opt/prometheus/data \
   -v /opt/consul-data \
   -v /var/lib/mysql \
   -v /var/lib/grafana \
   --name pmm-data \
   percona/pmm-server:latest /bin/true
```
- the second step is create and start ```pmm-server container``` to initialize the data directory correctly:
```c
docker run -d \
  -p 81:80 \
  --volumes-from pmm-data \
  --name pmm-server \
  --restart always \
  percona/pmm-server:latest
```
##
- the next step you need stop and remove ```pmm-server container```:
```
root@host:~# docker stop pmm-server_container
```

```
root@host:~# docker rm pmm-server_container
```
##
- after that you should сopy volumes files from ```pmm-data container``` to your host:
<br>

> copy prometheus data:

```
root@host:~# docker cp id_pmm-data_container:/opt/prometheus/ /your/prometheus/data/on/host
```
> copy consul data:

```
root@host:~# docker cp id_pmm-data_container:/opt/consul-data/ /your/consul/data/on/host
```
> copy mysql data:

```
root@host:~# docker cp id_pmm-data_container:/var/lib/mysql /your/mysql/data/on/host
```
> copy grafana data:

```
root@host:~# docker cp id_pmm-data_container:/var/lib/grafana /your/grafana/data/on/host
```
##
- after that you need delete ```pmm-data container```:

```
root@host:~# docker rm pmm-data_container
```
##
- the last step you need to specify you hosts volumes for pmm-data in docker-compose file as show below: 
```yml
version: '2'

services:
  pmm-data:
    image: percona/pmm-server:latest
    container_name: pmm-data
    volumes:
      - /your/host/prometheus/data:/opt/prometheus/data
      - /your/host/consul-data:/opt/consul-data
      - /your/host/mysql:/var/lib/mysql
      - /your/host/grafana:/var/lib/grafana
    entrypoint: /bin/true
```

-  after that just to run next command for launch new correct initialized ```pmm-server & pmm-data containers``` in docker-compose:
```
root@host:~# docker-compose up -d
```
##
After successfully completing the above steps, open your browser and type your IP of the host where the PMM Server is running and port what you specify in docker-compose:

<img width="250" height="40" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/ip.png">

If everything went well, you will see below page:

<img width="1000" height="400" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/home.png">


## INSTALL PMM-CLIENT 
You need to install PMM-CLIENT on host with your Database that you want to monitor:
- firstly you need to fetch the repository package:
```
root@host:~# wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
```
- install the downloaded repository package with ```dpkg```:

```
root@host:~# dpkg -i percona-release_latest.generic_all.deb
```
- apt-get update
```
root@host:~# apt-get update
```
- install the PMM Client package:
```
root@host:~# apt-get install pmm-client
```
This will install on your host latest version PMM Client.

## CONNECTING PMM CLIENT TO PMM SERVER
- to connect your Pmm Client to Pmm Server you need to do next:
```
root@host:~# pmm-admin config --server 192.168.0.0:81 --server-password YoUr-PassWorD
```
>in docker-compose file you may specify -SERVER_PASSWORD for security your Pmm-Server, but in this case during registration yourr host you need specify your --SERVER-PASSWORD as in the example above.
- you 

## ADDING MYSQL METRICS SERVICE
- to add your MySQL service to monitoring you may use next command:
```
root@host:~# pmm-admin add mysql
```
- after some time in section ```Mysql > Overview``` you can see information like this:
<img width="1000" height="500" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/mysql.png">

- finally, check the all services are up and running using below command:
```
root@host:~# pmm-admin list
```

## CONFIGURING MYSQL FOR BEST RESULTS
PMM can collect query data either from the slow query log or from Performance Schema. The slow query log provides maximum details, but can impact performance on heavily loaded systems. On Percona Server the query sampling feature may reduce the performance impact.
## 
- to enable user statistics, set the ```userstat``` variable to ```1``` in your ```my.cnf``` config file. But after that you need to restart you MySQL instance. If you don't have this opportunity you may set this in mysql directly:
```
> mysql: SET GLOBAL userstat=ON;
```
##
Query response time distribution is a feature available in Percona Server. It provides information about changes in query response time for different groups of queries, often allowing to spot performance problems before they lead to serious issues.
To enable collection of query response time:
1. Install the QUERY_RESPONSE_TIME plugins:
```
mysql> INSTALL PLUGIN QUERY_RESPONSE_TIME_AUDIT SONAME 'query_response_time.so';
mysql> INSTALL PLUGIN QUERY_RESPONSE_TIME SONAME 'query_response_time.so';
mysql> INSTALL PLUGIN QUERY_RESPONSE_TIME_READ SONAME 'query_response_time.so';
mysql> INSTALL PLUGIN QUERY_RESPONSE_TIME_WRITE SONAME 'query_response_time.so';
```
2. Set the global varible query_response_time_stats to ON:
```
mysql> SET GLOBAL query_response_time_stats=ON;
```
- after that you will see information like this:
<img width="1000" height="500" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/query.png">

##

## LINKS TO MORE INFORMATION
> https://www.percona.com/doc/percona-monitoring-and-management/index.html | Percona Monitoring and Management Documentation <br>
> https://www.percona.com/doc/percona-monitoring-and-management/pmm-admin.html#pmm-admin-add | Managing PMM Client <br>
> https://www.percona.com/doc/percona-monitoring-and-management/conf-mysql.html#pmm-conf-mysql-slow-log-settings | Configuring MySQL for Best Results <br>
> https://www.percona.com/doc/percona-monitoring-and-management/index.metrics-monitor.dashboard.html | Metrics Monitor Dashboards <br>
> https://www.percona.com/doc/percona-monitoring-and-management/glossary.option.html | PMM Server Additional Options <br>
> https://www.percona.com/doc/percona-monitoring-and-management/deploy/client/apt.html | Installing PMM Client on Debian or Ubuntu<br>
