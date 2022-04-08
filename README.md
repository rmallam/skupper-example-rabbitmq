# skupper-rabbitmq

## Pre Requisistes

- Two openshift clusters 
- Skupper installed in namespace and linked with the other cluster.

## How skupper can be used to form a rabbitmq cluster across multiple kubernetes clusters.

Rabbit MQ classic_config has been used to get this working(https://www.rabbitmq.com/cluster-formation.html#peer-discovery-configuring-mechanism). The node details are pre configured in the rabbitmq config file which will be passed as configmap to the rabbitmq nodes.


### On Cluster 1

  We chose to deploy 3 nodes of rabbitmq on cluster 1. Each node will be an individual deployment, This has been chosen over statefulsets because of the skupper limitations with statefulsets and the way it handles them.
  Each node deployment will be exposed as a skupper service to make the node visible across the other openshift cluster.
```
oc apply -f skupper-rabbit-configmap.yaml
oc apply -f rabbitdeploy-0.yaml
skupper expose deployment skupper-rabbit-0
oc apply -f rabbitdeploy-1.yaml
skupper expose deployment skupper-rabbit-1
oc apply -f rabbitdeploy-2.yaml
skupper expose deployment skupper-rabbit-2
```

if you run "skupper service status", you will notice the services skupper-rabbit-0, skupper-rabbit-1, skupper-rabbit-2 created and binded to their respective deployments.

```
$skupper service status
Services exposed through Skupper:
├─ skupper-rabbit-1 (tcp ports 5672 25672 15672 4369)
│  ╰─ Targets:
│     ╰─ app=skupper-rabbit-1 name=skupper-rabbit-1
├─ skupper-rabbit-0 (tcp ports 5672 25672 15672 4369)
│  ╰─ Targets:
│     ╰─ app=skupper-rabbit-0 name=skupper-rabbit-0
├─ skupper-rabbit-2 (tcp ports 25672 15672 4369 5672)
   ╰─ Targets:
      ╰─ app=skupper-rabbit-2 name=skupper-rabbit-2

```

skupper service create skupperrabbitmq 5672 25672 15672 4369 --mapping tcp

skupper service bind skupperrabbitmq deployment skupper-rabbit-0 
skupper service bind skupperrabbitmq deployment skupper-rabbit-1
skupper service bind skupperrabbitmq deployment skupper-rabbit-2
oc expose svc skupperrabbitmq
```

### On Cluster 2
```
oc apply -f skupper-rabbit-configmap.yaml

oc apply -f rabbitdeploy-3.yaml
skupper expose deployment skupper-rabbit-3
oc apply -f rabbitdeploy-4.yaml
skupper expose deployment skupper-rabbit-4

skupper service bind skupperrabbitmq deployment skupper-rabbit-3
skupper service bind skupperrabbitmq deployment skupper-rabbit-4
oc expose svc skupperrabbitmq
```


Login to the stats page using the URL from Cluster1 or 2, to view the state of the cluser with 
username:  user
password: testing


