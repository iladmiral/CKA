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