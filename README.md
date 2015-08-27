# Docker Image for Titan Graph Database

Titan is a free, open source database that is capable of processing
extremely large graphs and it supports a variety of indexing and storage backends,
which makes it easier to extend than some popular NoSQL Graph databases.

This docker image instantiaties a Titan graph database that is capable of
integrating with an ElasticSearch container (Indexing) and a Cassandra container (Storage).

The default distribution of Titan runs on a single node, so I thought it would be helpful
if there was a modular way at runtime to hook up Titan to its dependencies.

Enter Docker. Now it is possible to run Titan and it's dependencies in separate Docker containers.

## Titan

This container is using Titan 0.9.0. Please refer to
its [page](https://github.com/thinkaurelius/titan/wiki/Downloads) for more information.

## Tinkerpop and Gremlin

[Tinkerpop](http://www.tinkerpop.com/) is a vendor-independent API specification for
manipulating and access Graph databases.

## Running

The minimum system requirements for this stack is 1 GB with 2 cores.

```
docker run -d --name es elasticsearch
docker run -d -P --name cas1 spotify/cassandra
//docker run -d -P --name titan --link es:elasticsearch --link cas1:cassandra yubozhao/titan-gremlin
docker run -d --link es:elasticsearch --link cas1:cassandra -p 8182:8182 -p 8183:8183 -p 8184:8184 --name titan yubozhao/titan-gremlin
```

I run with a 3 node Cassandra cluster and some local ports exported, like so:

```
docker run -d --name es elasticsearch

docker run -d --name cas1 -p 7000:7000 -p 7001:7001 -p 7199:7199 -p 9160:9160 -p 9042:9042 poklet/cassandra
docker run -d --name cas2 poklet/cassandra start `docker inspect --format '{{ .NetworkSettings.IPAddress }}' cas1`
docker run -d --name cas3 poklet/cassandra start `docker inspect --format '{{ .NetworkSettings.IPAddress }}' cas1`

docker run -d --link es:elasticsearch --link cas1:cassandra -p 8182:8182 -p 8183:8183 -p 8184:8184 --name titan yubozhao/titan-gremlin
```

### Ports

8182: HTTP port for REST API

8184: JMX Port (You won't need to use this, probably)

To test out the REST API (over Boot2docker):

```
curl "http://docker.local:8182?gremlin=100-1"
curl "http://docker.local:8182?gremlin=g.addV('Name','Eric')"
curl "http://docker.local:8182?gremlin=g.V()"
```

## Dependencies

I've tested this container with the following containers:

	- poklet/cassandra: This is the Cassandra Storage backend for Titan. It scales well for large datasets.
	- dockerfile/elasticsearch: This is the ElasticSearch Indexing backend for Titan. It provides search capabilities for Titan graph datasets.

## Roadmap

In the near future, I'd like to add support for:

	- Scaling/Clustering Cassandra and ElasticSearch backends.
	- External volumes for persistent data.
	- Security between Titan and its backends.
	- Example application stack integrating with Titan.

