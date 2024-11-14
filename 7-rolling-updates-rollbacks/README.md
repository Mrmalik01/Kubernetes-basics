## Rolling Updates and Rollbacks

Rollouts update deployments - any changes in the configuration will trigger a rollout. It could be a change in the environment variable, or the container image version etc.

There are many rollout strategies for updates
- Default rollout strategy
    - Update in groups, which means both versions of the pods could exist for some time.
- Alternative - Recreate strategy
    - All-at-once strategy - but it causes some downtime
- Also scaling is not a rollout

### Rollout Strategy In Group

In Kubernetes, maxSurge and maxUnavailable are settings used in a RollingUpdate deployment strategy to control how many Pods can be added or removed during an update.

#### maxSurge
**Definition**: maxSurge specifies the maximum number of extra Pods that can be created temporarily during the update process.  
**Example**: If maxSurge is set to 25% and you have 10 replicas, Kubernetes can add up to 2 additional Pods (25% of 10) temporarily while updating. So, during the update, you could have up to 12 Pods running at once (10 original + 2 extra).

#### maxUnavailable
**Definition**: maxUnavailable specifies the maximum number of Pods that can be unavailable (not running) during the update.  
**Example**: If maxUnavailable is set to 25% and you have 10 replicas, up to 2 Pods (25% of 10) can be temporarily unavailable during the update. So, during the update, you could have as few as 8 Pods running at a given time.

**Why Use These Settings?**  
They help balance availability and speed during updates.
With a higher maxSurge, the deployment may complete faster but temporarily use more resources.
With a higher maxUnavailable, you might finish the update faster, but you risk reduced availability during the update.

By running the edit deployment command, I updated the container name and then checked the rollout status using the following command.
```
>tmuxkubectl rollout -n dp-5 status deployment app-tier                       
Waiting for deployment "app-tier" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "app-tier" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 3 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "app-tier" rollout to finish: 2 old replicas are pending termination...
deployment "app-tier" successfully rolled out
```

### Kubernetes Commands

#### Command to pause rollout
`kubectl rollout -n <namespace> pause deployment <deployment_name>`

#### Command to resume rollout
`kubectl rollout -n <namespace> resume deployment <deployment_name>`

#### Command to check rollout status for deployment

`kubectl rollout -n deployments status deployment app-tier` 

- `-w` : for watching
- `--watch`: for watching

#### Command to undo the most recent rollout

 `kubectl rollout -n deployments undo deployment app-tier`

#### Command to get the history of the roll out deployment

`kubectl rollout history -n deployments deployment app-tier`
