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