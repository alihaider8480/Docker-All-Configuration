# Docker-All-Configuration
-------------------------------------------------------------------------------------------------------------
                                          Ali Ahmed Wali
version: '2.1'

services:


  
  # Kakfa Web-UI
  kafdrop:
    image: obsidiandynamics/kafdrop
    restart: "no"
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: "kafka1:19092"
      JVM_OPTS: "-Xms16M -Xmx48M -Xss180K -XX:-TieredCompilation -XX:+UseStringDeduplication -noverify"
    depends_on:
      - "kafka1"

  # Zookeeper instance
  zoo1:
    image: zookeeper:3.4.9
    hostname: zoo1
    ports:
      - "2181:2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_PORT: 2181
      ZOO_SERVERS: server.1=zoo1:2888:3888
    volumes:
      - ./zk-single-kafka-single/zoo1/data:/data
      - ./zk-single-kafka-single/zoo1/datalog:/datalog

  # Kafka
  kafka1:
    image: confluentinc/cp-kafka:5.5.1
    hostname: kafka1
    ports:
      - "9092:9092"
      - "9999:9999"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka1:19092,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: ${DOCKER_HOST_IP:-127.0.0.1}
    volumes:
      - ./zk-single-kafka-single/kafka1/data:/var/lib/kafka/data
    depends_on:
      - zoo1
----------------------------------------------------------------------------------------------------------------------------------

                                      metrics.yml
version: '3.5'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - 9090:9090
    volumes:
      - $HOME/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
    networks:
      - pgnw
      
  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - 3000:3000
    networks:
      - pgnw
      
volumes:
  grafana-data:
    driver: local
    
networks:
  pgnw:


------------------------------------------------------------------------------------------------------------------------------------



                                                            prometheus.yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
  
  
rule_files:


scrape_configs:
  - job_name: 'AHDPAPI'
    metrics_path: '/q/metrics'
    scrape_interval: 5s
    static_configs:
    - targets: ['192.168.20.10:8080']


-----------------------------------------------------------------------------------------------------------------------------------------

                                                                gitlab ki image ka lia
services:

  ide:
    image: codercom/code-server
    container_name: code-serverr
    ports:
      - 9000:8080
    environment:
      - "DOCKER_USER=$USER"
    volumes:
      - "$HOME/.config:/home/code/.config"
      - "$PWD/workspace:/home/code/projects"
      - "/var/run/docker.sock:/var/run/docker.sock"
  
  gitlab:
     image: "gitlab/gitlab-ce:latest"
     restart: always
     hostname: "www.gitlab.com"
     ports:
       - 8080:8080
     volumes:
       - "./gitlab/config:/etc/gitlab"
       - "./gitlab/logs:/var/log/gitlab"
       - "./gitlab/data:/var/opt/gitlab"
      

  registry:
     image: registry
     container_name: registryy 
     ports: 
       - 5003:5003 

----------------------------------------------------------------------------------------------------------------------------

                                                  elk.yml
version: '3.5'

services:
  elasticsearch:
    image: elasticsearch:7.16.2
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xms512m -Xmx512m"
      discovery.type: single-node
    networks:
      - elk
      
  logstash:
    image: logstash:7.16.2
    volumes:
      - source: $HOME/pipelines
        target: /usr/share/logstash/pipeline
        type: bind
    ports:
      - "12201:12201/udp"
      - "5000:5000"
      - "9600:9600"
    networks:
      - elk
    depends_on:
      - elasticsearch
      
  kibana:
    image: kibana:7.16.2
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch
    
networks:
  elk:
-------------------------------------------------------------------------------------------------------

        
                                            Db2 Database image run
volumes:
  db2data:

services:
  db:
    image: ibmcom/db2
    container_name: db2
    restart: always
    environment:
      LICENSE: accept
      DB2INST1_PASSWORD: db2admin
      DBNAME: testdb
    volumes:
      - "db2data:/database"
    ports:
      - "50000:50000"
    privileged: true
