### Cluster Maintenance
- what happens if one of nodes go DOWN
	- the pod on them are not accessible
- what k8s will do:
	- if the node be UP immedately the kublet process start
- consedring th node dead is 5 min delay
- drain the node: move the pods on the node to other UP nodes `kubectl drain node-1`
	- the node are also marked as unshedulable -> no pod will be placed on
- uncodron = shecudulable
#### Lab
`kubectl drain node01 --ignore-daemonsets`
whitch pod in which node : `kubectl get pods -o wide`