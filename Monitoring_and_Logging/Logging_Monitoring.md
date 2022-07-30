### Monitor the Cluster Components
what I would like to get as metrics in k8s cluster.
- number of nodes, the health of nodes
- CPU, Memory, Network ..
- Pos metrics

There is varouis application that provide a metrics like: netdog, dynatrace, promothus, metrics server

The metcris are provided by k8s node agent kubelet, exactly Cadvisor a component of kubelet.

To install Metrics server :
1. minikube : `minikube addons enable metrics-server`
2. or download git repo and create pods

To get the metrics:
- `kubectl top node`
- `kubectl top pod`


### Managing Application Logs
- if pod have one container:
	- `kubectl logs pod_name`
- if pod has two or more containers:
	- `kubectl logs pod_name container_name`