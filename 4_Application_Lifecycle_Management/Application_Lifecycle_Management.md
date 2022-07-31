### Rolling Updates and Rollbacks

#### Rollout and Versioning
Each time the application is deployed it triggers a Rollout 
- a new rollout create a new deployment revision ex: Revision 1

- see  the status of rollout
```bash
kubectl rollout status deployment/name_of_deployment
```

- see the revision and history
```bash
kubectl rollout history deployment/name_of_deployment
```

#### Deployment stategy
Recreate stategy: destroy exisant pods than -> create a new pods `application downtime` -> not a defaut deployment strategy

Rolling Update: destroy one create one .. `zero downtime` -> default strategy
Any modification of yaml defination is an update.
Update just a image of deployment:
```bash
kubectl set image deployment/name_of_deployment container_name=nginx:1.9.1
```

#### Rollback
Rolling update create a new Replicaset with a new pods, that kill one old pod and switch one new pod
```bash
kubectl rollout undo deployment/name_of_deployment
```


#### Configure Applications 

Configuring applications comprises of understanding the following concepts:
-   Configuring Command and Arguments on applications   
-   Configuring Environment Variables
-   Configuring Secrets

### Application Commands

- In docker
	- The container only lives as long as the process inside it is lives, if the web service inside the container cruches/stop -> the container exit
	- If you look at the dockerfile the  `CMD ["nginx"]` specify whitch application will run 
	- Unlike virtual machines, containers are not meant to hist operating system

#### How do you specify a different command to start the container?
- One option is to append to the docker run command and that way it overrides the default command specified whtin the image
	```bash
	$ docker run ubuntu sleep 5
	```
- make this change parmanent 
	```Dockerfile
	FROM Ubuntu
	CMD sleep 5
	```

- In k8s
	- anything is appended to docker will go into `args`  in the pod definition
	```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: ubuntu-sleeper-pod
	spec:
	 containers:
	 - name: ubuntu-sleeper
	   image: ubuntu-sleeper
	   command: ["sleep2.0"] # --> ENTRYPOINT ["sleep"]
	   args: ["10"] # --> CMD ["5"]
	```

#### Configure Environment Varaibles in Applications
- use the environment variable
	```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: ubuntu-sleeper-pod
	spec:
	 containers:
	 - name: ubuntu-sleeper
	   image: ubuntu-sleeper
	   ports:
	     - caintainerPort: 8080
	   env:
	     - name: 
	       value: 
	```

- use ConfigMap
	``` yaml
	env:
	  - name: APP_COLOR
	    valueFrom: 
	      configMapKeyRef:
	```

- use secret:
	```yaml
	env:
	  - name: APP_COLOR
	    valueFrom:
	      secretKeyRef:
	```

#### Configuring ConfigMaps in Applications
- configMap used when we have a log of data, we put together in one file
1. create the configMap
	1. imperative:
		1. `kubectl create configmap <config-name> --from-literal=<key>=<value>`
		2. `kubectl create configmap <config-name> --from-file=<path-to-file>`
	2. declarative:
		```yaml
		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: app-config
		data:
		  APP_COLOR: blue
		  APP_MODE: prod
	    ```
- view configmaps: `kubectl get configmaps`
- describe configmap: `kubectl describe configmaps`
1. inject then into the pod
```yaml
spec:
  containers:
    envFrom:
      - configMapRef:
          name: app-config
```

- we can inject data as : single ENV, Volume
- lab:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-07-31T18:51:58Z"
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
  resourceVersion: "640"
  uid: 2ca41485-ca71-44fb-8d12-75e1f307a9c8
spec:
  containers:
    - name: webapp-color
      image: kodekloud/webapp-color
      imagePullPolicy: Always      
      envFrom:
        - configMapRef:
            name: webapp-config-map
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File      
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-wt2xp
          readOnly: true
```