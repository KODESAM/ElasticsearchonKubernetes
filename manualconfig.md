Step 1:

Minikube requires a values.yaml file to run Elasticsearch. Download the file with:

```
curl -O https://raw.githubusercontent.com/elastic/helm-charts/master/elasticsearch/examples/minikube/values.yaml
```

Step 2: Set Up the Values by Pod Role
1. Copy the contents of the values.yaml file using the cp command into three different pod configuration files:
```
cp values.yaml master.yaml
cp values.yaml data.yaml
cp values.yaml client.yaml
```
2. Check for the four YAML files using the ls command:
```
ls -l *.yaml
```
3. Open the master.yaml file with a text editor and add the following configuration at the beginning:
```
# master.yaml
---
clusterName: "elasticsearch"
nodeGroup: "master"
roles:
  master: "true"
  ingest: "false"
  data: "false"
replicas: 3
```
The configuration sets the node group to master in the elasticsearch cluster and sets the master role to "true". Additionally, the master.yaml creates three master node replicas.

The full master.yaml looks like the following:
```
# master.yaml
---
clusterName: "elasticsearch"
nodeGroup: "master"
roles:
  master: "true"
  ingest: "false"
  data: "false"
replicas: 3
# Permit co-located instances for solitary minikube virtual machines.
antiAffinity: "soft
# Shrink default JVM heap.
esJavaOpts: "-Xmx128m -Xms128m"
# Allocate smaller chunks of memory per pod.
resources:
  requests:
    cpu: "100m"
    memory: "512M"
  limits:
    cpu: "1000m"
    memory: "512M"
# Request smaller persistent volumes.
volumeClaimTemplate:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: "standard"
  resources:
    requests:
      storage: 100M
```
4. Save the file and close.

5. Open the data.yaml file and add the following information at the top to configure the data pods:
```
# data.yaml
---
clusterName: "elasticsearch"
nodeGroup: "data"
roles:
  master: "false"
  ingest: "true"
  data: "true"
replicas: 2
```
The setup creates two data pod replicas. Set both the data and ingest roles to "true". Save the file and close.

6. Open the client.yaml file and add the following configuration information at the top:
```
# client.yaml
---
clusterName: "elasticsearch"
nodeGroup: "client"
roles:
  master: "false"
  ingest: "false"
  data: "false"
replicas: 2
service:
  type: "LoadBalancer"
```  
  7. Save the file and close.

The client has all roles set to "false" since the client handles service requests. The service type is designated as "LoadBalancer" to balance service requests evenly across all nodes.

Step 3: Deploy Elasticsearch Pods by Role
1. Add the Helm repository:
```
helm repo add elastic https://helm.elastic.co
```
2. Use the helm install command three times, once for each custom YAML file created in the previous step:
```
helm install elasticsearch-multi-master elastic/elasticsearch -f ./master.yaml
helm install elasticsearch-multi-data elastic/elasticsearch -f ./data.yaml
helm install elasticsearch-multi-client elastic/elasticsearch -f ./client.yaml
```
3. Wait for cluster members to deploy. Use the following command to inspect the progress and confirm completion:
```
kubectl get pods
```
Step 4: Test Connection
1. To access Elasticsearch locally, forward port 9200 using the kubectl port-forward command:
```
kubectl port-forward service/elasticsearch-master
```
2. In another terminal tab, test the connection with:
```
curl localhost:9200
```
