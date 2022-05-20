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