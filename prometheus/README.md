# Prometheus

[Github](https://github.com/prometheus/prometheus)

### Keywords

Pushgateway, Service Discovery (dns,k8s,consul), Exporters, Alert Manager, Grafana Integration


```bash
## Install
docker run --name prometheus --rm -p 127.0.0.1:9090:9090 prom/prometheus

docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                      NAMES
b8328de95d2d   prom/prometheus   "/bin/prometheus --c…"   8 seconds ago   Up 7 seconds   127.0.0.1:9090->9090/tcp   prometheus


## Access the Dashboard
http://localhost:9090/


https://hub.docker.com/r/prom/prometheus-linux-arm64

### Node Exporter
https://github.com/prometheus/node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.darwin-arm64.tar.gz
tar xvfz node_exporter-1.9.1.darwin-arm64.tar.gz
cd node_exporter-1.9.1.darwin-arm64.tar.gz
./node_exporter



### 
./node_exporter
## 

curl http://localhost:9100/metrics
curl http://localhost:9100/metrics | grep "node_"
##

docker login
docker pull prom/prometheus-linux-arm64:main
docker scout quickview prom/prometheus-linux-arm64:main

# What's next:
#     View a summary of image vulnerabilities and recommendations → docker scout quickview prom/prometheus-linux-arm64:main
# docker scout quickview prom/prometheus-linux-arm64:main
#     ✓ Image stored for indexing
#     ✓ Indexed 218 packages

#     i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
#       Review docs.docker.com ↗ for more information.

#   Target               │  prom/prometheus-linux-arm64:main  │    0C     0H     2M     0L
#     digest             │  fd6ee87adbc7                      │
#   Base image           │  busybox:1-uclibc                  │    0C     0H     0M     0L
#   Refreshed base image │  busybox:1-uclibc                  │    0C     0H     0M     0L
#                        │                                    │
#   Updated base image   │  busybox:1.36-uclibc               │    0C     0H     0M     0L
#                        │                                    │

# What's next:
#     View vulnerabilities → docker scout cves prom/prometheus-linux-arm64:main
#     View base image update recommendations → docker scout recommendations prom/prometheus-linux-arm64:main
#     Include policy results in your quickview by supplying an organization → docker scout quickview prom/prometheus-linux-arm64:main --org <organization>

### blackbox exporter
http://github.com/prometheus/blackbox_exporter
The blackbox exporter allows blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP, ICMP and gRPC.

docker run --rm \
  -p 9115/tcp \
  --name blackbox_exporter \
  -v $(pwd):/config \
  quay.io/prometheus/blackbox-exporter:latest --config.file=/config/blackbox.yml

```