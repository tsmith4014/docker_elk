# ELK Stack Setup Guide for Mac M2 using Docker

## Introduction

This guide provides detailed instructions for setting up the ELK (Elasticsearch, Logstash, Kibana) stack on a Mac with an M2 chip using Docker. This setup is suitable for development and testing purposes.

## Step 1: Create Docker Compose File

First, we'll create a `docker-compose.yml` file for defining the ELK stack services.

### 1.1 Create a Directory for Your Project

- Open a terminal.
- Create a new directory and navigate into it:
  ```bash
  mkdir elk-stack-m2
  cd elk-stack-m2
  ```

### 1.2 Create and Edit `docker-compose.yml`

- In your new directory, create a file named `docker-compose.yml`:
  ```bash
  nano docker-compose.yml
  ```
- Copy and paste the following content into the file:

  ```yaml
  version: "3.2"

  services:
    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
      platform: linux/arm64
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
      platform: linux/arm64
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
      platform: linux/arm64
      ports:
        - "127.0.0.1:5601:5601"
      environment:
        ELASTICSEARCH_HOSTS: http://elasticsearch:9200

  volumes:
    elasticsearch-data:
  ```

- Save and exit the editor (`CTRL+X`, then `Y` to confirm, and `Enter`).

### 1.3 Create Configuration Directories for Logstash

- Create directories for Logstash configuration:
  ```bash
  mkdir -p logstash/config logstash/pipeline
  ```

### 1.4 Create Logstash Configuration File

- In the `logstash/pipeline` directory, create a file named `logstash.conf`:
  ```bash
  nano logstash/pipeline/logstash.conf
  ```
- Add the following content and save the file:
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

## Step 2: Start the ELK Stack

- Run the ELK stack using Docker Compose:
  ```bash
  docker-compose up
  ```

## Step 3: Verify the Services

- Check if the services are running:
  ```bash
  docker ps
  ```

## Step 4: Access Kibana

- Open a web browser and go to [http://localhost:5601](http://localhost:5601) to access Kibana.

## File Structure

Your `elk-stack-m2` directory should look like this:

```
elk-stack-m2/
│   docker-compose.yml
│
└───logstash/
    ├───config/
    └───pipeline/
        │   logstash.conf
```

## Conclusion

You have successfully set up the ELK stack on your Mac M2 using Docker. This setup is ideal for development, testing, and exploring the capabilities of Elasticsearch, Logstash, and Kibana.

---

This README provides a complete guide for setting up the ELK stack on a Mac M2 chip, tailored for users who have Docker already installed and are familiar with basic command-line operations.
