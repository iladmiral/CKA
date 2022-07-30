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