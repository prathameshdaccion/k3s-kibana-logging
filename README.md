# k3s-Logging with Elasticsearch-Metribeat-Kibana
K3S logs have to be stored in centralized location

ElasticSearch was selected as such centralized location, Kibana is used for data visualisation

Metricbeat is used as a DaemonSet to ensure we get a running metricbeat daemon on each node of the cluster.

Everything is deployed under infra namespace, you can change that by updating YAML manifests under this folder.

# Assumptions

You have a K3S environment

You have namespace created with name infra or you can make the changes in yaml files as per your namespace requirement

# Environment Setup

#### 1. Setup Elasticsearch on your k3s setup

The first node of the cluster we’re going to set up is the master which is responsible for controlling the cluster.

###### Command

$ kubectl apply -f es-master-cm.yaml -f es-master-svc.yaml -f es-master-dep.yaml

The second node of the cluster we’re going to set up is the data node that is responsible for hosting the data and executing the queries (CRUD, search, aggregation).

###### Command

$ kubectl apply -f es-data-cm.yaml -f es-data-svc.yaml -f es-data-dep.yaml

The third node client is responsible for exposing an HTTP interface and pass queries to the data node.

###### Command

$ kubectl apply -f es-client-cm.yaml -f es-client-svc.yaml -f es-client-dep.yaml

After a couple of minutes, each node of the cluster should reconcile and the master node should log the following sentence:

###### "Cluster health status changed from [YELLOW] to [GREEN]"

###### Command

$ kubectl logs -f -n infra $(kubectl get pods -n infra | grep elasticsearch-master | sed -n 1p | awk '{print $1}') \
| grep "Cluster health status changed from \[YELLOW\] to \[GREEN\]"

Now,Generate a password and store in a k3s secret.

###### Command

kubectl exec -it $(kubectl get pods -n infra | grep elasticsearch-client | sed -n 1p | awk '{print $1}') -n infra -- bin/elasticsearch-setup-passwords auto -b

$ kubectl create secret generic elasticsearch-pw-elastic -n infra --from-literal password={elasticsearch-password}

#### 2. Setup Kibana on your k3s setup

Deploy Kibana and after a couple of minutes, check the logs for 

###### Status changed from yellow to green

###### Command

$ kubectl apply -f kibana-cm.yaml -f kibana-svc.yaml -f kibana-dep.yaml -f kibana-ingress.yaml

Once, the logs say “green”, you can access Kibana from your browser by ingress url. ( Update ingress host url as per your requirement )

Login with username elastic and the password (previously generated and stored in a secret).

#### 2. Setup Metricbeat on your k3s setup

Deploy metribeat using below command and check list of indices in elasticsearch.
Note: You need to update elastic password in metribeat.yaml against below configuration. Replace value of 43HzGXJmyMCqxX0uu8Rk with your password.

    output.elasticsearch:
      hosts: '${ELASTICSEARCH_HOSTS:http://elasticsearch-client.infra.svc.cluster.local:9200}'
      username: '${ELASTICSEARCH_USERNAME:elastic}'
      password: '${ELASTICSEARCH_PASSWORD:43HzGXJmyMCqxX0uu8Rk}'

###### Command

$ kubectl apply -f metricbeat.yaml

$ curl http://10.43.165.131:9200/_cat/indices -u elastic

#### 4. Create index pattern in kibana for logging

Now our setup is up and running, create an index pattern in kibana. 

###### Go to Management > Kibana > Index Patterns and click on Create index pattern for your index of metricbeat. 

You can see an aggregated view of all the logs printed from every pod in metribeat-* index. You can filter the logs by any 

attributes attached to the log (for example a Kubernetes label) and navigate over the time.
