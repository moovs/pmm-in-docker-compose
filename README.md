<p align="center">
  <img width="600" height="186" src="https://github.com/moovs/pmm-in-docker-compose/blob/master/scr/percona.png">
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
# Step 1
If you just run the docker-compose file it will not work correctly due to incorrect container initialization.
<br>
- Therefore, the first step that you will need to do create the pmm-data container with default values: 
```
docker create \
   -v /opt/prometheus/data \
   -v /opt/consul-data \
   -v /var/lib/mysql \
   -v /var/lib/grafana \
   --name pmm-data \
   percona/pmm-server:latest /bin/true
```
