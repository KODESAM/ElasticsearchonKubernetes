##### Deploying ELK stack on Docker #####

Step 1: Create a Directory /Folder with the name of ELK.
Step 2: Open the ELK Directory
Step 3: Create a File named docker-compose.yml in this directory.
Step 4: Copy-Paste the following commands : (it mentions some stuff about the dependencies we want to install, including Elastic Search and Kibana)

```
version: "3.7"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.0
    container_name: elasticsearch
    restart: always
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
    ulimits: 
      memlock:
        soft: -1 
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add: 
      - IPC_LOCK
    volumes:
      - elasticsearch-data-volume:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.12.0
    restart: always
    environment:
      SERVER_NAME: kibana
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
volumes: 
  elasticsearch-data-volume:
    driver: local
```

Step 5: Save the File and run the following command under the same directory terminal.
```
docker-compose -f docker-compose.yml up -d
```
You will see the system downloading your ELK Stack.
Step 6: Once the download is completed, you can go either to the Docker Desktop and look up 2 new Docker containers created there(as seen in image) or you can type on your terminal :
```
docker container ls
```
