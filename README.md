# Docker ELK Stack - Implementation Guide

This guide will walk you through setting up and running an ELK (Elasticsearch, Logstash, Kibana) stack using Docker. The ELK stack is a collection of three open-source products — Elasticsearch, Logstash, and Kibana — that allows you to search, analyze, and visualize logs generated by your applications and services in real-time.

## File Structure

Here's the directory and file structure for your Docker ELK stack project:

```
docker_elk/
├── docker-compose.yml
└── logstash/
    ├── config/
    │   ├── logstash.yml
    │   └── pipelines.yml
    └── pipeline/
        └── logstash.conf
```

## Step-by-Step Implementation

### 1. Docker Compose File (`docker-compose.yml`)

This file defines the services needed for your ELK stack, including Elasticsearch, Logstash, and Kibana.

```yaml
version: "3.2"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
    platform: linux/arm64
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - type: volume
        source: elasticsearch-data
        target: /usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.2
    platform: linux/arm64
    ports:
      - "5044:5044"
    volumes:
      - type: bind
        source: ./logstash/config
        target: /usr/share/logstash/config
      - type: bind
        source: ./logstash/pipeline
        target: /usr/share/logstash/pipeline
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.2
    platform: linux/arm64
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200

volumes:
  elasticsearch-data:
```

### 2. Logstash Configuration (`logstash/config/logstash.yml`)

This file configures Logstash, specifying how it binds to the HTTP host and connects to Elasticsearch for monitoring.

```yaml
http.host: "127.0.0.1"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]
```

### 3. Logstash Pipelines Configuration (`logstash/config/pipelines.yml`)

This file defines the pipelines used by Logstash for processing data.

```yaml
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

### 4. Logstash Pipeline (`logstash/pipeline/logstash.conf`)

This file details the input and output configurations for Logstash.

```conf
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

### 5. Running the Stack

To get your ELK stack up and running, execute the following command:

```sh
docker-compose up -d
```

This command starts all the services defined in your `docker-compose.yml` file in detached mode.

### 6. Accessing Kibana

Once the stack is running, you can access Kibana by navigating to `http://localhost:5601` in your web browser.

## Conclusion

This guide provided a step-by-step walkthrough for setting up a basic ELK stack using Docker. You can further customize this setup by modifying the configurations based on your specific requirements.
