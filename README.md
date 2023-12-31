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

---

## Docker ELK Stack - Comprehensive Implementation and Testing Guide part2/draft2 this is work in progress. Below is basicly same as above with some minor changes

This guide covers the setup of an ELK (Elasticsearch, Logstash, Kibana) stack using Docker and how to test it with JSON data over TCP. The ELK stack is a powerful suite of tools for searching, analyzing, and visualizing logs from applications and services in real-time.

### File Structure

Your Docker ELK stack project should have the following structure:

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

Defines the services for the ELK stack, including Elasticsearch, Logstash, and Kibana.

```yaml
version: "3.2"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
    platform: linux/arm64 # Uncomment for Mac M2, comment out for non-Mac
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
    platform: linux/arm64 # Uncomment for Mac M2, comment out for non-Mac
    ports:
      - "5044:5044" # For Beats input
      - "5000:5000" # For TCP input (JSON data)
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
    platform: linux/arm64 # Uncomment for Mac M2, comment out for non-Mac
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200

volumes:
  elasticsearch-data:
```

### 2. Logstash Configuration

#### `logstash/config/logstash.yml`

Configures Logstash settings, including HTTP host binding and Elasticsearch monitoring.

#### `logstash/config/pipelines.yml`

Defines the main pipeline for Logstash.

#### `logstash/pipeline/logstash.conf`

Specifies the input and output configurations for Logstash.

```conf
input {
  beats {
    port => 5044
  }
  tcp {
    port => 5000
    codec => json
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "test-logstash-index"
  }
  stdout { codec => rubydebug }
}
```

### 3. Running the ELK Stack

Execute the following command to start the services:

```sh
docker-compose up -d
```

### 4. Accessing Kibana

Kibana is accessible at `http://localhost:5601`.

### 5. Testing the Setup

#### Sending JSON Data to Logstash

Send a JSON message to Logstash's TCP input:

```bash
echo '{"status": "success", "message": "merryxmas"}' | nc 127.0.0.1 5000
```

#### Verifying Data in Elasticsearch

Check if the message is indexed in Elasticsearch:

```bash
curl -X GET "http://127.0.0.1:9200/test-logstash-index/_search?q=message:merryxmas&pretty"
```

## Conclusion

This comprehensive guide provides detailed instructions for setting up and testing a Docker-based ELK stack, suitable for both Mac M2 and non-Mac systems. You can customize this setup further based on your specific requirements, and the testing steps ensure that your configuration is functioning correctly.

---

---

# Docker ELK Stack - Complete Setup and Configuration Guide

This comprehensive guide provides step-by-step instructions for setting up an ELK (Elasticsearch, Logstash, Kibana) stack using Docker, including advanced configuration and testing procedures.

## File Structure

Your project directory should look like this:

```plaintext
docker_elk/
├── docker-compose.yml
└── logstash/
    ├── config/
    │   ├── logstash.yml
    │   └── pipelines.yml
    └── pipeline/
        └── logstash.conf
```

## Docker Compose Setup (`docker-compose.yml`)

Create a `docker-compose.yml` file with the following content:

```yaml
version: "3.2"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
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
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200

volumes:
  elasticsearch-data:
```

Run the Docker Compose:

```bash
docker-compose up -d
```

## Logstash Configuration

### `logstash/config/logstash.yml`

```yaml
http.host: "127.0.0.1"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]
```

### `logstash/config/pipelines.yml`

```yaml
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

### `logstash/pipeline/logstash.conf`

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

## Accessing Kibana

Open a web browser and navigate to `http://localhost:5601`.

## Additional Configuration Steps

### Configuring Elasticsearch Index Format

1. **Shell Script (`setup_elasticsearch.sh`)**

   Create a shell script to manage the Elasticsearch index.

   ```bash
   #!/bin/bash
   # Delete the existing index
   curl -XDELETE "http://localhost:9200/test-logstash-index"
   # Create a new index with custom mappings
   curl -XPUT "http://localhost:9200/test-logstash-index" -H 'Content-Type: application/json' -d'
   {
     "mappings": {
       "properties": {
         "host": {
           "type": "object"
         }
       }
     }
   }'
   ```

   Make the script executable:

   ```bash
   chmod +x setup_elasticsearch.sh
   ```

   Run the script:

   ```bash
   ./setup_elasticsearch.sh
   ```

### Monitoring and Debugging

- **Elasticsearch Indices**

  ```bash
  curl -X GET "localhost:9200/_cat/indices?v"
  ```

- **Logstash Logs**

  ```bash
  docker logs [logstash_container_id]
  ```

### Ensuring Successful Logging to Kibana

- **Check Logstash Configuration**: Ensure that your Logstash configuration is correctly set up to send data to Elasticsearch.
- **Restart Services**: If logging appears intermittent, try restarting the Logstash service.

## Conclusion

This guide provides a thorough walkthrough for setting up a Docker-based ELK stack. Customize this setup as needed to suit your specific requirements and ensure efficient logging and monitoring of your applications and services.

---

---

# Docker ELK Stack - Complete Setup and Configuration Guide With Filebeat

This guide offers a comprehensive walkthrough for setting up an ELK (Elasticsearch, Logstash, Kibana) stack using Docker, specifically tailored for Mac M1/M2 users, along with configuring Filebeat for log forwarding.

## File Structure

Your project should follow this directory structure:

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

## Docker Compose Setup (`docker-compose.yml`)

Create a `docker-compose.yml` file with the following content:

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

Run the stack:

```bash
docker-compose up -d
```

## Logstash Configuration

### `logstash/config/logstash.yml`

```yaml
http.host: "127.0.0.1"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]
```

### `logstash/config/pipelines.yml`

```yaml
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

### `logstash/pipeline/logstash.conf`

```conf
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "test-logstash-index"
  }
  stdout { codec => rubydebug }
}
```

## Accessing Kibana

Navigate to `http://localhost:5601` in your web browser.

## Configuring Filebeat on Mac

After installing Filebeat, edit its configuration:

```bash
nano /opt/homebrew/etc/filebeat/filebeat.yml
```

### Simplified Filebeat Configuration

```yaml
filebeat.inputs:
  - type: filestream
    id: my-filestream-id
    enabled: true
    paths:
      - /var/log/*.log

setup.kibana:

output.logstash:
  hosts: ["localhost:5044"]

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

This configuration sets up Filebeat to forward logs to Logstash. Save the file and start Filebeat:

```bash
filebeat -e
```

## Additional Configuration Steps

### Elasticsearch Index Format Configuration

Create a shell script (`setup_elasticsearch.sh`) for configuring the index:

```bash
#!/bin/bash
curl -XDELETE "http://localhost:9200/test-logstash-index"
curl -XPUT "http://localhost:9200/test-logstash-index" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "host": {
        "type": "object"
      }
    }
  }
}'
```

Make the script executable and run it:

```bash
chmod +x setup_elasticsearch.sh
./setup_elasticsearch.sh
```

### Monitoring and Debugging

- **Check Elasticsearch Indices**: `curl -X GET "localhost:9200/_cat/indices?v"`
- **View Logstash Logs**: `docker logs [logstash_container_id]`

### Ensuring Successful Logging to Kibana
