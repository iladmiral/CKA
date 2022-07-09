## Manual Scheduling
### How schedling works
every pod have a ```nodeName:``` in his definition, this part is not showed by k8s

assign a pod nginx to the node node01
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: node01
```

commands:
 ```kubectl get cs``` : get status of scheduler & controller-manager
 
 ### Labels & Selectors
 `label` : help to give some caracters to objet
 `selector` : help to filter objets
 
 ```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: App1
    function: Front-end
spec:
  containers:
  -  image: nginx
     name: nginx
```

- select a Pod 
	- `kubectl get pods --selector app=App1`
- in replecaset, we have labels defined in 2 ears:
	- `template` : labels defined under Pods
	- at the `top` : are the labels of the replicaset
	- in creation of replicaset if area `matchLabels` match with `labels` the `Pod` will be created
	- the `annotations` part wil coantain the information about the `deployment`
	- combain selecting:
		- `kubectl get pods --selector app=App1,funtion=Frontend`
	
	### Taints and Tolerations
- `taints` : a specific role given to the node, if the pod is not tolerent for this node the schedulers will not placed in the specific node. it set in `node`
-  `tolerations` : set on `pod`
-  `kubectl taint nodes [node-name] key-value:taint-affect`
-  taint-affect:
	-  `NoSchedule` : pod will not be scheduled to the node
	- `PreferNoSchedule` : try to avoid placinf the pod on the node but no garanties
	-  `NoExecute`: new pods will not be scheduled on the node, and the exesting pods will be evicted 'expulsé' if they are not tolerent to the `taint`
		-  ex:  `kubectl taint nodes node1 app=blue:NoSchedule`
	- `tolerations`
		```yaml
			apiVersion: v1
			kind: Pod
			metadata:
			  name: nginx
			spec:
			  containers:
			  -  image: nginx
				 name: nginx
			  tolerations:
			  - key: "app"
			    operator: "Equal"
				value: "blue"
				affect: "NoSchedule"
		```
		- value need to be encoded in  `double quotes`
		- taints & tolerations does not tell the pods to go the a specific node, intead it tell nodes to only accept pods with the tolerations
- a `master` node is a node like other nodes with a tolerations 
	- `kubectl describe node kubemaster | grep Taint`

### Node selectors, Node affinity
to force a Pod to be in specific node, we have 2 methods:
1.   node selectors:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
	 name: nginx
  nodeSelector:
	size: Large  # --> will put the pod in the largest Node
```
	
- node that the node have to be label with the same caractrestic.
	- `kubectl label nodes [node-name] [label-key]=[label-value]`
2. node affinity:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
	 name: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
	    nodeSelectorTerms:
		  - matchExpressions:
		      - key : size
			    operator: In # NotIn, Exists
				values:
				  - Large
```
	
- types of node affinity:
	- Available
		- `requiredDuringSchedulingIgnoredDuringExecution`
		- `preferredDuringSchedulingIgnoredDuringExecution:`
	- Planned
		-  `requiredDuringSchedulingRequiredDuringExecution`
-  `DuringScheduling` pod not exists and is created for te first time
-  `DuringExecution`  pod exists

	## lab
	```yaml
	## blue
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: blue
	  labels:
		app: frontend
	spec:
	  template:
		metadata:
		  name: nginx
		  labels:
			app: frontend
		spec:
		  containers:
			- name: nginx
			  image: nginx
		  affinity:
			nodeAffinity:
			  requiredDuringSchedulingIgnoredDuringExecution:
				nodeSelectorTerms:
				  - matchExpressions:
					  - key: color
						operator: In
						values:
						  -  blue
	  replicas: 3
	  selector:
		matchLabels:
		  app: frontend
	```
	
	```yaml
	## red
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: red 
	  labels:
		app: frontend
	spec:
	  template:
		metadata:
		  name: nginx
		  labels:
			app: frontend
		spec:
		  containers:
			- name: nginx
			  image: nginx
		  affinity:
			nodeAffinity:
			  requiredDuringSchedulingIgnoredDuringExecution:
				nodeSelectorTerms:
				  - matchExpressions:
					  - key: node-role.kubernetes.io/master
						operator: Exists
	  replicas: 2
	  selector:
		matchLabels:
		  app: frontend
	```

### Taints and toleration Vs node affinity
- Taints and telerations garentie that node could accept just a tainted pods 
	- but pods could be placed on node with no taints
- Node affinity garenty the the pod be placed in spicific node
	- but node could accept other pods
- the solution to place a pods in specifics nodes is:
	- combain taints & tolerations and node affinity

### Resource requirements and limits
- The scheduler will place pods in node that satisfy (cpu, memory, disk) requirments
	- If there is no space (cpu, memory, disk) the pods will be in pending state.
	- in events it will show : `Insufficient cpu`
- by `default` k8s assume the the pods requirs: 0,5 cpu, 256mi
	- minimum memory 0.1 mi -> 100 m
- Docker container has no limits of resources, if it need more resources il will get it until it take all of resource of pod
	- by `default` container is limited of 1vCPU, 512Mi
		- if want to change it add `limits` to `resources` section of pod
		- container can't take more than the limit of cpu
		- container can use more memory resources than it's limits
			- if the pod try the use more memoy than it's limits contenly -> it will be terminated
		- to note: `required` is initialy used by the pods
```yaml
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: mem-limit-range
  spec:
    limits:
    - default:
        memory: 512Mi
      defaultRequest:
        memory: 256Mi
      type: Container
 ```
 
 ### Daemon Sets
daemon Sets run a copy of the pod in each node of the cluser
	- used for monitoring solution, logs viewer
		- a `kube-proxy` agent is a must for every pod, a good use case of daemonset
		- networking agent as `weave-net`
	- `kubectl get daemonsets` get daemon sets
- DaemonSet definition
						
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
	name: monitoring-agent
	namespace: kube-system
spec:
	selector:
		matchLabels:
			app: monitoring-agent
	template:
		metadata:
			labels:
				app: monitoring-agent
		spec:
			containers:
			- name: monitoring-agent
				image: monitoring-agent
```

### static pods
- if there is no api kube-apiserver, no ETCD, no controller-Manager, no kube-scheduler
- kublet will create pods from  `/etc/kubernetes/manifests` automaticly
- it can ensure that the pod is still a live
- it the files in this directory is modified the kubelet will create a new pod and delete the old
- if the file is deleted the pod will be deleted too
- it could only create `pods`
- the directory is passed as a config when creating a service 
	- kubelet.service : `--pod-manifest-path=/etc/kubernetes/manofestes`
	- OR : add to kubelet.service `--config=kubeconfig.yaml`
	- kubeconfig.yaml : `staticPodPath: /etc/kubernetes/manifestes`
- use case: create a pods for master node
	- out the yamls files in manifestes directories than 
		- etcd.yaml, apiserver.yaml, controller-Manager.yaml 
		- the pods will be created aumomaticly
	- #### Lab
		- `kubectl gets pods --all-namespaces`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-busybox
spec:
  containers:
  - command:
    - sleep 1000
    image: busybox
    name: static-busybox
```