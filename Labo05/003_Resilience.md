# Task 3 - Add and exercise resilience

By now you should have understood the general principle of configuring, running and accessing applications in Kubernetes. However, the above application has no support for resilience. If a container (resp. Pod) dies, it stops working. Next, we add some resilience to the application.

## Subtask 3.1 - Add Deployments

In this task you will create Deployments that will spawn Replica Sets as health-management components.

Converting a Pod to be managed by a Deployment is quite simple.

  * Have a look at an example of a Deployment described here: <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>

  * Create Deployment versions of your application configurations (e.g. `redis-deploy.yaml` instead of `redis-pod.yaml`) and modify/extend them to contain the required Deployment parameters.

  * Again, be careful with the YAML indentation!

  * Make sure to have always 2 instances of the API and Frontend running. 

  * Use only 1 instance for the Redis-Server. Why?

    > Using only one Redis server instance ensures simplicity, consistency, and easier management. It avoids the complexity of synchronization.

  * Delete all application Pods (using `kubectl delete pod ...`) and replace them with deployment versions.

  * Verify that the application is still working and the Replica Sets are in place. (`kubectl get all`, `kubectl get pods`, `kubectl describe ...`)

## Subtask 3.2 - Verify the functionality of the Replica Sets

In this subtask you will intentionally kill (delete) Pods and verify that the application keeps working and the Replica Set is doing its task.

Hint: You can monitor the status of a resource by adding the `--watch` option to the `get` command. To watch a single resource:

```sh
$ kubectl get <resource-name> --watch
```

To watch all resources of a certain type, for example all Pods:

```sh
$ kubectl get pods --watch
```

You may also use `kubectl get all` repeatedly to see a list of all resources.  You should also verify if the application stays available by continuously reloading your browser window.

  * What happens if you delete a Frontend or API Pod? How long does it take for the system to react?
    > The Pod will be terminated, then shortly after, another one will be created.
    
  * What happens when you delete the Redis Pod?

    > The Pod will be terminated, and shortly after, another one will be created. However, all the previous data we wrote in the todo app will be lost.
    
  * How can you change the number of instances temporarily to 3? Hint: look for scaling in the deployment documentation

    > We can simply change the value in the .yaml file or use this commande    
    > `kubectl scale deployment/frontend-deploy --replicas=3`.
    
  * What autoscaling features are available? Which metrics are used?
    > Features:    
    >    `HPA`: Adjusts the number of pod replicas based on metrics like CPU or memory usage.    
    >    `VPA`: Adjusts CPU and memory requests of pods based on historical usage data.    
    >    `Cluster Autoscaler`: Adjusts the cluster size by adding or removing nodes based on resource demands.    
    >    `Custom Metrics Autoscaling`: Uses user-defined metrics (e.g., queue length, request latency) for scaling decisions.
    >          
    > Metrics:    
    >    `CPU Utilization`: Triggers scaling based on CPU usage thresholds.    
    >    `Memory Utilization`: Triggers scaling based on memory usage thresholds.    
    >    `Custom Metrics`: User-defined metrics relevant to application performance.    
    >    `Object Metrics`: Specific to Kubernetes-managed resources, like database items or queue messages.    
    
  * How can you update a component? (see "Updating a Deployment" in the deployment documentation)

    > We have to udate the deloyement .yaml file and then run the appy commande     
    > `kubectl apply -f file.yaml`

## Subtask 3.3 - Put autoscaling in place and load-test it

On the GKE cluster deploy autoscaling on the Frontend with a target CPU utilization of 30% and number of replicas between 1 and 4.     
> Using this commande :    
> `kubectl autoscale deployment frontend-deployment --cpu-percent=30 --min=1 --max=4`
Load-test using Vegeta (500 requests should be enough).

> Using this commande :    
> `echo "GET http://34.118.45.122" | vegeta attack -duration=1m -rate=500 | vegeta report --type=text`
```shell
Requests      [total, rate, throughput]         30000, 500.02, 315.66
Duration      [total, attack, wait]             1m15s, 59.998s, 14.747s
Latencies     [min, mean, 50, 90, 95, 99, max]  11.058µs, 732.647ms, 149.654ms, 973.625ms, 2.935s, 11.987s, 30s
Bytes In      [total, mean]                     14887814, 496.26
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           78.65%
Status Codes  [code:count]                      0:6406  200:23594  
```

> [!NOTE]
>
> - The autoscale may take a while to trigger.
>
> - If your autoscaling fails to get the cpu utilization metrics, run the following command
>
>   - ```sh
>     $ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
>     ```
>
>   - Then add the *resources* part in the *container part* in your `frontend-deploy` :
>
>   - ```yaml
>     spec:
>       containers:
>         - ...:
>           env:
>             - ...:
>           resources:
>             requests:
>               cpu: 10m
>     ```
>

## Deliverables

Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

> // TODO

```````sh
// TODO object descriptions
```````

```yaml
# redis-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    component: redis
    app: todo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo
      component: redis
  template:
    metadata:
      labels:
        component: redis
        app: todo
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
        args:
        - redis-server 
        - --requirepass ccp2 
        - --appendonly yes
```

```yaml
# api-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    component: api
    app: todo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: api
  template:
    metadata:
      labels:
        component: api
        app: todo
    spec:
      containers:
      - name: api
        image: icclabcna/ccp2-k8s-todo-api
        ports:
        - containerPort: 8081
        env:
        - name: REDIS_ENDPOINT
          value: redis-svc
        - name: REDIS_PWD
          value: ccp2
```

```yaml
# frontend-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    component: frontend
    app: todo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: frontend
  template:
    metadata:
      labels:
        component: frontend
        app: todo
    spec:
      containers:
      - name: frontend
        image: icclabcna/ccp2-k8s-todo-frontend
        ports:
        - containerPort: 8080
        env:
        - name: API_ENDPOINT_URL
          value: http://api-svc:8081
        resources:
          requests:
            cpu: 10m
```
