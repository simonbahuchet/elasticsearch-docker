## ElasticSearch Dockerfile


This repository contains **Dockerfile** of [ElasticSearch](http://www.elasticsearch.org/) for [Docker](https://www.docker.com/)'s [automated build](https://registry.hub.docker.com/u/sisays/elasticsearch/) published to the public [Docker Hub Registry](https://registry.hub.docker.com/).


### Base Docker Image

* [dockerfile/java:oracle-java8](http://dockerfile.github.io/#/java)


### Installation

1. Install [Docker](https://www.docker.com/).

2. Download [automated build](https://registry.hub.docker.com/u/sisays/elasticsearch-docker/) from public [Docker Hub Registry](https://registry.hub.docker.com/): `docker pull sisays/elasticsearch-docker`

   (alternatively, you can build an image from Dockerfile: `docker build -t="sisays/elasticsearch-docker" github.com/simonbahuchet/elasticsearch-docker`)


### Usage: Run one node

```sh
docker run -d -p 9200:9200 -p 9300:9300 sisays/elasticsearch-docker
```

#### Get the corresponding IP
	
```sh
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CONTAINER_ID>
```
	
#### After a few seconds, use the REST API to retrieve the cluster state

```sh
curl -XGET http://<IP>:9200/_cluster/state?pretty=true
```	
	
### Run multiple nodes
	
Configure <data-dir>/data/config/elasticsearch.yml

```yml
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["es_node1", "es_node2", "es_node3"]
path:
   logs: /data/log
   data: /data/data
```

and then start up 3 nodes:
	
```sh
sudo docker run -d \
	-h es_node1 \
	--name=es_node1 \
	-p 9200:9200 -p 9300:9300 \
	-v <data-dir>:/data \
	sisays/elasticsearch \
	/elasticsearch/bin/elasticsearch -Des.config=/data/config/elasticsearch.yml
```

```sh
sudo docker run -d \
	-h es_node2 \
	--name=es_node2 \
	--link es_node1:es_node1 \
	-p 9201:9200 -p 9301:9300 \
	-v <data-dir>:/data \
	sisays/elasticsearch \
	/elasticsearch/bin/elasticsearch -Des.config=/data/config/elasticsearch.yml
```

```sh
sudo docker run -d \
	-h es_node3 \
	--name=es_node3 \
	--link es_node1:es_node1 --link es_node2:es_node2 \
	-p 9202:9200 -p 9302:9300 \
	-v <data-dir>:/data \
	sisays/elasticsearch \
	/elasticsearch/bin/elasticsearch -Des.config=/data/config/elasticsearch.yml
```

#### Get the IP of one the nodes
	
```sh
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CONTAINER1_ID>
```
	
#### After a few seconds, use the REST API to retrieve the cluster state
	
```sh
curl -XGET http://<IP1>:9200/_cluster/state?pretty=true
```