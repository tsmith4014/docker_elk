# ELK Stack Setup Guide for Ubuntu using Docker

## Introduction

This guide will help you set up the ELK (Elasticsearch, Logstash, Kibana) stack on an Ubuntu system using Docker.

## Prerequisites

- A computer running Ubuntu.
- Basic command-line knowledge.

## Step 1: Install Docker Engine and Docker Compose

Before setting up the ELK stack, you need to install Docker Engine and Docker Compose.

### 1.1 Install Docker Engine

Follow the official Docker installation guide for Ubuntu: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

### 1.2 Install Docker Compose

Docker Compose is used to run multi-container Docker applications. To install Docker Compose, follow the instructions here: [Install Docker Compose](https://docs.docker.com/compose/install/).

## Step 2: Create Docker Compose File for ELK Stack

Create a `docker-compose.yml` file that defines the ELK stack services.

### 2.1 Create a Directory for Your Project

Open a terminal and run:

```bash
mkdir elk-stack
cd elk-stack
```

### 2.2 Create and Edit `docker-compose.yml`

In the `elk-stack` directory, create a file named `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Add the following content:

```yaml
version: "3.2"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
    environment:
      - discovery.type=single-node
    ports:
      - "127.0.0.1:9200:9200"
      - "127.0.0.1:9300:9300"
    volumes:
      - type: volume
        source: elasticsearch-data
        target: /usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.2
    ports:
      - "127.0.0.1:5000:5000"
    volumes:
      - type: bind
        source: ./logstash/config
        target: /usr/share/logstash/config
      - type: bind
        source: ./logstash/pipeline
        target: /usr/share/logstash/pipeline

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.2
    ports:
      - "127.0.0.1:5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200

volumes:
  elasticsearch-data:
```

Save and close the file (`CTRL+X`, then `Y` to confirm, and `Enter`).

### 2.3 Create Configuration Directories for Logstash

Run the following commands:

```bash
mkdir -p logstash/config logstash/pipeline
```

### 2.4 Create Logstash Pipeline Configuration

In the `logstash/pipeline` directory, create a file named `logstash.conf`:

```bash
nano logstash/pipeline/logstash.conf
```

Add the following content:

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

Save and close the file.

## Step 3: Start the ELK Stack

Run the ELK stack using Docker Compose.

### 3.1 Start Services

In the `elk-stack` directory, run:

```bash
docker-compose up -d
```

### 3.2 Verify the Services

Check if the services are running:

```bash
docker ps
```

## Step 4: Access Kibana

Open a web browser and go to [http://localhost:5601](http://localhost:5601) to access Kibana.

## File Structure

Your `elk-stack` directory should look like this:

```
elk-stack/
│   docker-compose.yml
│
└───logstash/
    ├───config/
    └───pipeline/
        │   logstash.conf
```

## Conclusion

You have successfully set up the ELK stack on your Ubuntu machine using Docker. This setup is ideal for development and testing purposes.

---

This README provides a complete guide for someone new to Docker and ELK, including installation and setup steps on Ubuntu. Adjustments for specific use
