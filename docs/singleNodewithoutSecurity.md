# Single Node – Without Security

#### Docker Compose File
```yaml
# version: "3.8" // shows warning: obsolete, can be removed

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:9.1.5
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=false   # disable security for now
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    networks:
      - elastic
    healthcheck:
      test: ["CMD-SHELL", "curl -s http://localhost:9200 | grep 'cluster_name'"]
      interval: 10s
      timeout: 10s
      retries: 10

  kibana:
    image: docker.elastic.co/kibana/kibana:9.1.5
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://es01:9200
    ports:
      - "5601:5601"
    depends_on:
      - es01
    networks:
      - elastic

volumes:
  esdata01:
    driver: local

networks:
  elastic:
    driver: bridge
```

---

## **Overview**

This Docker Compose setup spins up a **local Elasticsearch + Kibana environment** on a single node with **security disabled**. It is intended for **development and testing**, not production.

**flow:**

1. **Elasticsearch Service (`es01`)**

    * Pulls ES version `9.1.5` from Docker Hub.
    * Runs a **single-node cluster** (`discovery.type=single-node`).
    * Security is disabled (`xpack.security.enabled=false`).
    * Allocates 1 GB JVM memory (`ES_JAVA_OPTS=-Xms1g -Xmx1g`).
    * Stores data persistently in volume `esdata01`.
    * Exposes **ports 9200 and 9300**.
    * Healthcheck ensures Elasticsearch is ready before Kibana starts.

2. **Kibana Service (`kibana`)**

    * Pulls Kibana version `9.1.5`.
    * Connects to Elasticsearch at `http://es01:9200`.
    * Exposes **port 5601** for web access.
    * Depends on Elasticsearch to start first.

3. **Volumes and Networks**

    * `esdata01` persists Elasticsearch data across container restarts.
    * `elastic` network allows ES and Kibana to communicate internally.

---

> I have listed the use of a few tags used. We can move to **Single Node with Security** next.

### **Key Concepts & Docker Compose Tags**

| **Tag / Concept**         | **Explanation**                                                                                                         |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Cluster**               | Group of Elasticsearch nodes working together to store/search data. Single-node cluster has only one node.              |
| **Service**               | Defines a container in Docker Compose, including image, config, ports, volumes, and environment variables.              |
| **Volume**                | Persistent storage for containers. Data survives container restarts. Example: `esdata01` for ES data.                   |
| **Network**               | Virtual network for container communication. Allows services to reach each other via container names.                   |
| **Default**               | If no network specified, container joins the default bridge network.                                                    |
| **xpack**                 | Elasticsearch plugin for security, monitoring, and advanced features. `xpack.security.enabled=false` disables auth/TLS. |
| **ulimits**               | Sets resource limits. `memlock` prevents JVM memory from swapping. `soft` = runtime limit, `hard` = max enforceable.    |
| **discovery.type**        | Determines node discovery. `single-node` used for development. Multi-node clusters use other mechanisms.                |
| **bootstrap.memory_lock** | Locks JVM heap to prevent swapping. Improves performance and stability.                                                 |
| **depends_on**            | Ensures container starts after dependencies. Does **not** wait for readiness; use with `healthcheck`.                   |
| **healthcheck**           | Command to check if container is ready. Ensures dependent services wait until service is responsive.                    |
| **ports**                 | Maps container ports to host. Example: `9200:9200` for ES REST API, `9300:9300` for cluster communication.              |
| **environment**           | Sets container environment variables for configuration like JVM options, node/cluster names, and security.              |
| **volumes**               | Maps host or named storage to container paths. Persists data across container restarts.                                 |
| **networks**              | Connects containers to virtual networks, allowing internal communication.                                               |


## **Single Node vs Multi-Node Elasticsearch**

* **Single Node**: One Elasticsearch node. Simple, lightweight, for **development/testing**. No redundancy; if it fails, data is unavailable.
* **Multi-Node**: Multiple nodes forming a cluster. **Production-ready**, fault-tolerant, scalable. Data is distributed across nodes.
* **Quick Guide**:

  * Development / Learning → Single Node
  * Production / High Availability → Multi-Node

> We’ll discuss the **significance and implications** of single vs multi-node setups in the next section.
