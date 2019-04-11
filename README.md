<p align="center">
  <img width="600" height="170" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/src/percona.png">
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

## Getting a PMM server running on docker is just matter of following a few simple steps.
If you just run the docker-compose file it will not work correctly due to incorrect container initialization, therefore:
<br>
- the first step that you will need to do create the ```pmm-data container``` with default values: 
```js
docker create \
   -v /opt/prometheus/data \
   -v /opt/consul-data \
   -v /var/lib/mysql \
   -v /var/lib/grafana \
   --name pmm-data \
   percona/pmm-server:latest /bin/true
```
- the second step is create and start ```pmm-server container``` to initialize the data directory correctly:
```js
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
- the last step is just to run next command for launch new correct initialized ```pmm-server & pmm-data containers``` in docker-compose:
```
root@host:~# docker-compose up -d
```
##


<p align="center">
<img src="https://octodex.github.com/images/dojocat.jpg" width="200">
</p>
<p align="center">
<b>sayonara</b>
</p>
