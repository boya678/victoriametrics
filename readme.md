Victoria Metrics Installation

What is?
Victoria metrics is a proposal different to Prometheus and Mimir timescale databases for monitoring and logging, is around or 4x more efficient in resources using and in response time.
Repo:
ri-dsb_devops-monitoring/victoriametrics at master · AppGate-TFP/ri-dsb_devops-monitoring (github.com)
Focus:
Devops, Sysops and monitoring teams
Objective:
Install a complete replacement for prometheus stack
Helm repos:
firts you need to add dependencies charts if you want to install grafana, but the prometheus chart is required to install the node exporter.
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add vm https://victoriametrics.github.io/helm-charts/
helm repo update
Installation:
for the installation we are going to use this values:
 
What are you installing?
this values are going to install the next components:
•	Node exporter: is taken from prometheus community charts and export machine metrics from nodes.
•	VM operator: is the victoria metrics operator with de CRD´s for vmservicescrape, vmpodscrape, vmnodescrpae, vmsingle, vmagent, etc. Custom resources (victoriametrics.com)
•	VM agent: is the component that make the scraping and receive the metrics in remotewrite mode and sends to victoriametrics with remotewriteto.
•	VM single: is the database and receive the metrics in remotewrite incoming.
•	VM alert: is the alert manager embebed.
•	VM rules: are the rulefiles CRD (vmrule) for scraping custom metrics or alerts.
Steps:
•	Modify the alert manager config for your sns or another component like teams webhook or something in values yaml route: alertmanager.config
•	Install the vm stack with the previous values.
helm upgrade --install --namespace vm --create-namespace vm vm/victoria-metrics-k8s-stack -f values.yaml
•	Apply this service that connects to cluster for standar kubelet scraping:
 
kubectl apply -f kubelet.yaml
•	Apply this vmservicescrape to make the scraping for the previous service
 
kubectl apply -f vmservicescrape-kubelet.yaml
•	import the dashboards about vm stack monitoring:      
OTLP configuration:
for this one is to easy, we only need to configure a remotewrite exporter in the OTLP metrics side like this:
exporters:
  prometheusremotewrite:
    endpoint: http://vmagent-vm.vm:8429/prometheus/api/v1/write
and add the prometheusremotewrite field in the array of metrics:
metrics:
  exporters:
  - prometheusremotewrite
Enjoy:
now you must delete all prometheus stack, go to bar and drink some cups!
Backup
For backup VM has a component compatible with S3, BlobStorages, GCP Storage.
VictoriaMetrics best practices
vmbackup (victoriametrics.com)
vmrestore (victoriametrics.com)
