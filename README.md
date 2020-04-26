# k3s-Logging with Elasticsearch-Metribeat-Kibana
K3S logs have to be stored in centralized location

ElasticSearch was selected as such centralized location, Kibana is used for data visualisation

Metricbeat is used as a DaemonSet to ensure we get a running metricbeat daemon on each node of the cluster.

Everything is deployed under infra namespace, you can change that by updating YAML manifests under this folder.

# Environment Setup
