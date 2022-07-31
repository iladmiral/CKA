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