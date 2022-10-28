### How to deploy ELK stack on Kubernetes ###

Deploy Elastic Search
First we will create a values file which will expose the elastic search using ingress. Be sure to deploy the ingress controller beforehand. Create a file, values-2.yaml with the following content:
```
replicas: 1
minimumMasterNodes: 1

ingress:
  enabled: true
  hosts:
    - host: es-elk.s9.devopscloud.link #Change the hostname to the one you need
      paths:
        - path: /
  
volumeClaimTemplate:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 10Gi
```
Now execute the following commands to add the Elastic Search helm repo:
```
helm repo add elastic https://helm.elastic.co
helm repo update
```

Now to deploy the elastic search, execute the command:
```
helm install elk-elasticsearch elastic/elasticsearch -f values-2.yaml --namespace logging --create-namespace
```
Deploy Kibana

Now, we will create a custom values file for Kibana helm chart. Create a file values-2.yaml with the following content:
```
elasticsearchHosts: "http://elasticsearch-master:9200"
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: kibana-elk.s9.devopscloud.link
      paths:
        - path: /
 ```
Now, to deploy the helm chart use the command:
```
helm install elk-kibana elastic/kibana -f values-2.yamls
```
To verify the kibana is working fine, use the ingress host on browser.

Deploy the logstash

Now, we will create a custom values file for Logstash helm chart. Create a file values-2.yaml with the following content:
```
persistence:
  enabled: true

logstashConfig:
  logstash.yml: |
    http.host: 0.0.0.0
    xpack.monitoring.enabled: false

logstashPipeline: 
 logstash.conf: |
    input {
      beats {
        port => 5044
      }
    }
    output {
      elasticsearch {
        hosts => "http://elasticsearch-master.logging.svc.cluster.local:9200"
        manage_template => false
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
      }
    }

service:
  type: ClusterIP
  ports:
    - name: beats
      port: 5044
      protocol: TCP
      targetPort: 5044
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
 ```
Now to deploy the logstash, execute the following command:
```
helm install elk-logstash elastic/logstash -f values-2.yaml
```

Deploy the filebeat

Now, we will create a custom values file for Logstash helm chart. Create a file values-2.yaml with the following content:
```
daemonset:
  filebeatConfig:
    filebeat.yml: |
      filebeat.inputs:
      - type: container
        paths:
          - /var/log/containers/*.log
        processors:
        - add_kubernetes_metadata:
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"

      output.logstash:
        hosts: ["elk-logstash-logstash:5044"]
        
```
Now, to deploy the logstash use the following command:
```
helm install elk-filebeat elastic/filebeat -f values-2.yaml
```
