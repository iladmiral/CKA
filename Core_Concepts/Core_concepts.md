#### Pods
- Create pod 
`Kubectl run nginx --image nginx` 
- See pod 
`Kubectl get pods` 
- Create a pod based on file 
`Kubectl create -f pod-definition.yml` 
- More information about the pod 
`kubectl describe pod myapp-pod` 
- Delete pod 
`Kubectl delete pods nginx`

#### YAML in k8s
```yaml
#A must 
apiVersion: v1  #--> string 
kind: Pod  #--> string 
metadata: 
  name: myapp-pod 
  labels: 
     app: myapp 
     type: front-end 
spec: 
  containers: 
   - name: nginx-container 
     image: nginx 
```

- `kind` : 
	-  Pod 
	- Service 
	- ReplicaSet 
	- Deployemnt
	- .. 
`metadata` -> keys specfied by k8s 
`labels` -> you can specify any key you want

#### ReplicaSets
Replication controller-> could run multiple instances of pod -> Hight Availability -> create a load balancing & scaling
- Create replication controller:
	```yaml
	apiVersion: v1  
	kind: ReplicationController 
	metadata: 
	  name: myapp-rc 
	  labels: 
	    app: myapp 
		type: front-end 
    spec: 
	  template: 
	    metadata: 
		  name: myapp-pod 
          labels: 
		    app: myapp 
			type: front-end 
        spec: 
		  containers: 
		   - name: nginx-container 
			 image: nginx 
	  replicas: 3
	```
	
	- See the replication controller:
		- `Kubectl get replicationcontroller`
	- Create replicasSets:  could manage even pod that not created in this time
		- Is a process to monitor pods if one og pod get down the replicasset will created for you 
		- Labels help the replicas set to kown which pods to monitor
		```yaml
		apiVersion: apps/v1  
		kind: ReplicasSet 
        metadata: 
		  name: myapp-replicaset 
          labels: 
		    app: myapp 
		    type: front-end 
        spec: 
		  template: 
            metadata: 
			  name: myapp-pod 
              labels: 
			    app: myapp 
			    type: front-end 
            spec: 
              containers: 
			    - name: nginx-container 
				  image: nginx 
		  replicas: 3 
		  selector: 
            matchLabels: 
			  type: front-end
		```
		- See the created replicas: `kubectl get replicaset`
		- Scale the apps: Change the replicas number than run `kubectl replace -f file.yml`
			- OR: `kubectl scale --replicas=6 -f file.yml`
		- Delete replicaset: `Kubectl delete replicaset name`
		- Edit the running replicaset: `kubectl edit rs/name`

#### Deployment: 
- Rolling update : update intances one by one 
- Pods -> replicas set -> deployment
- YAML
	```yaml
	#Deployment definition: 
	apiVersion: apps/v1  
	kind: Deployment 
	metadata: 
	  name: myapp-replicaset 
      labels: 
	    app: myapp 
	    type: front-end 
    spec: 
	  template: 
        metadata: 
		  name: myapp-pod 
		  labels: 
		    app: myapp 
		    type: front-end 
        spec: 
          containers: 
		    - name: nginx-container 
			  image: nginx 
	  replicas: 3 
	  selector: 
        matchLabels: 
		  type: front-end 
	```
	- Create deployment: `kubectl create -f deployment-definition.yml`
	- Get the deployment: `kubectl get deployments`
	- See all: `kubectl get all`

#### Namespace 
 - Default: namespace used if we don't specify the namespace for created objects 
 - kube-system : namespace used for own k8s objctes 
 - Each namespace could have rules to define who can do what 
 - Resources limites: give a namespace a certain amount of resources 
- Refer service: `mysql.connect("db-service")` --> same namespace 
- Reach another namespace: 
`mysql.connect("db-service.dev.srv.cluster.local")`  --> servicename.namespace.srv.domain 
`kubectl get pods` --> list pods in the default namespace 
`kubectl get pods --namespace=kube-system` --> specify the namespace 
- Create a pod in another namespace: 
`kubectl create -f pod-definition.yml --namespace=dev` 
- Change the namespace to the pod yml definition: 
	``` yaml
	apiVersion: v1 
	kind: Pod  
	metadata: 
	  name: myapp-pod 
	  namespace: dev 
	  labels: 
		  app: myapp 
		  type: front-end 
	spec: 
	  containers: 
	   - name: nginx-container 
		  image: nginx 
	```
- Create a new namespace: 
	``` yaml
		apiVersion: v1 
        kind: Namespace 
        metadata: 
		  name:dev 
	```

Or 
`kubectl create namespace dev` 
- Switch namespaces: 
`Kubectl config set-context $(kubectl config current-context) --namespace=dev`

- View pods in all namespaces: 
Kubectl get pods --all-namespaces 

- Limite resources: 
	- Create resource quota  
	```yaml
		apiVersion: v1 
		kind: ResourceQuota 
        metadata: 
		  name: compute-quota 
		  namespace: dev 
        spec: 
          hard: 
			pods: "10" 
			requests.cpu: "4" 
			requests.memory: 5Gi 
			limits.cpu: "10" 
			limits.memory: 10Gi
	```

#### Service
- K8s service enable comminication between components --> outside in the application 

 - Help us comments applications together with other application or user 

- Services types: 
	-   NodePort   
	-   ClusterIP 
	-   LoadBalancer

#### Useful cmds
- How to generates templates: 
Pods: `kubectl run nginx --image=nginx --dry-run=client -o yaml `
Deployment: `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml `
- See the infos of one deploymnet: 
`kubectl get all -l=app=httpd-frontend` 