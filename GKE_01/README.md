### 1 Deploying K8s on Cloud

1. There are 2 types of Kubernetes Cloud Services
	a. **Self Hosted/Turnkey Solutions**
	b. **Hosted Solutions/Manage Solutions**
2. **Self Hosted/Turnkey:**
	a. We can deploy the clusters and k8s ourselves
	b. AWS provides tools to make the task easier, using ***Kops*** or ***kubeOne***
	c. We can do the configuration ourselves.
	d. We are responsible for the k8s cluster and its performance ourselves.
	e. You provision and configure VMS.
3. **Hosted Solutions/Management Solutions**:
	a. The version of Kubernetes and the Master node are managed by the Cloud itself, like AWS or Google Cloud.
	b. It acts like a ***Kubernetes as a Service***
	c. Provider ***maintains, installs and manages*** the VMs and Kubernetes Management.
	d. Example: *Google Kubernetes Engine(**GKE**), Azure Kubernetes Service (**AKE**), Amazon Elastic Kubernetes Service(**KES**)*

### 1.1 Creating the GKE Cluster

1. You will have to sign up and then create a billing account, otherwise you won't be able to enable the Kubernetes Cluster.
2. You will be given 300$ for 90 days trial, and you will have to enter your credit card details.
3. You will have to select between ***static*** and ***release***. Static means you will have to update the Cluster version manually, release would do it automatically.
4. Once your cluster is ready after a few minutes, a green tick will appear. Click on connect to connect to the Google Cloud Shell.
5. Finally, to connect to the GKE cluster, you need to copy the cloud shell command to connect to it: example `gcloud container clusters get-credentials starter --region us-central1 --project feisty-wall-454-t4`
6. NOTE: You will need to **clone** yaml files to deploy them on the cloud server.

### 1.2 Problem identified and its Solution

Reference:

1. **Course:** Kubernetes for the Absolute Beginners - Hands-on]([https://www.udemy.com/course/learn-kubernetes/](https://www.udemy.com/course/learn-kubernetes/))
2. **Lecture 54**: GKE default settings requires you to set up the resources in the containers, if you don't do that declaratively, then the Autopilot injects the default resources which may restrict you from deploying additional resources.

Problem:

1. If you try to create resources out of deployment yaml files without defining resources (cpu + memory), then it will give you the following error on your GKE CLI:  
    _Warning: autopilot-default-resources-mutator:Autopilot updated Deployment default/result: defaulted unspecified 'cpu' resource for containers [result] (see [http://g.co/gke/autopilot-defaults](http://g.co/gke/autopilot-defaults))._

Solution:

1. Define resource requests + Limits in all your pods specs.
2. Then create resources, and all your pods will run.  
    Reference link of Autopilot: [https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests#defaults](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests#defaults)
3. **Add Resource Quota** resource (OPTIONAL)

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: default
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    limits.memory: "4Gi"

----
run
kubectl create -f resourcequota.yaml
```

### 1.3 Deployment + Services

1. The project consists of the following:
	a. redis-deployment.yaml
	b. worker-deployment.yaml
	c. db-deployment.yaml
	d. result-deployment.yaml
	e. vote-deployment.yaml
	f. result-service.yaml
	g. vote-service.yaml
	h. db-service.yaml
	i. redis-service.yaml

2. Add the **Resources** limits + requests inside the deployment container specs:

```
resources:
  requests:
	cpu: "100m"
	memory: "128Mi"
  limits:
    cpu: "500n"
    memory: "512Mi"

INTERPRETATION:

- 100m → 100 millicores → 0.1 CPU core (10% of 1 CPU).
- 128Mi → 128 Mebibytes (~134 MB) of memory.
```

```
redis-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - mountPath: /data
          name: redis-data
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

      volumes:
      - name: redis-data
        emptyDir: {}

-------------------------

worker-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: worker
  name: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers
      - image: dockersamples/examplevotingapp_worker
        name: worker
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "512Mi"

---------------------------------------------

db-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: db
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - image: postgres:15-alpine
        name: postgres
        env:
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          value: postgres
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-data
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
      volumes:
      - name: db-data
        emptyDir: {}

----------------------------

result-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: result
  name: result
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result
        name: result
        ports:
        - containerPort: 80
          name: result
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

--------------------------

vote-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vote
  name: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - image: dockersamples/examplevotingapp_vote
        name: vote
        ports:
        - containerPort: 80
          name: vote
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

--------------------------
result-service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: result
  name: result
spec:
  type: NodePort
  ports:
  - name: "result-service"
    port: 8081
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
---------------------------------------
vote-service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: vote
  name: vote
spec:
  type: NodePort
  ports:
  - name: "vote-service"
    port: 8080
    targetPort: 80
    nodePort: 31000
  selector:
    app: vote

------------------------

db-service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: db
  name: db
spec:
  type: ClusterIP
  ports:
  - name: "db-service"
    port: 5432
    targetPort: 5432
  selector:
    app: db
------------------------------------

redis-service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
spec:
  type: ClusterIP
  ports:
  - name: "redis-service"
    port: 6379
    targetPort: 6379
  selector:
    app: redis

```

### 1.4 Output

1. kubectl get pods,svc,deployments,nodes

```
sohaib_sharih@cloudshell:~ (feisty-wall-450814-t4)$ kubectl get pods,nodes,deployments,svc

NAME                          READY   STATUS    RESTARTS   AGE
pod/db-6bdd5f5bb4-shj24       1/1     Running   0          13h
pod/redis-59784f5478-v2dmb    1/1     Running   0          11h
pod/result-6699dc8fc6-plw9p   1/1     Running   0          11h
pod/vote-789b9976dd-wt47h     1/1     Running   0          11h
pod/worker-6845d8c8c-294v2    1/1     Running   0          11h

NAME                                    STATUS   ROLES    AGE   VERSION
node/gk3-starter-pool-2-7a505b96-zdbt   Ready    <none>   13h   v1.33.2-gke.1240000
node/gk3-starter-pool-2-ecb9e8ce-nrbw   Ready    <none>   13h   v1.33.2-gke.1240000

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db       1/1     1            1           13h
deployment.apps/redis    1/1     1            1           13h
deployment.apps/result   1/1     1            1           12h
deployment.apps/vote     1/1     1            1           12h
deployment.apps/worker   1/1     1            1           11h

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/db           ClusterIP      34.118.230.238   <none>          5432/TCP         13h
service/kubernetes   ClusterIP      34.118.224.1     <none>          443/TCP          23h
service/redis        ClusterIP      34.118.238.233   <none>          6379/TCP         13h
service/result       NodePort       34.118.234.77    <none>          8081:31001/TCP   12h
service/result-lb    LoadBalancer   34.118.230.15    34.66.196.31    80:31522/TCP     11h
service/vote         NodePort       34.118.239.230   <none>          8080:31005/TCP   12h
service/vote-lb      LoadBalancer   34.118.238.27    104.198.214.9   80:31877/TCP     11h
```

2. **Run the app on the browser** (EXPOSE + RUN ON BROWSER)
	a. Expose the services as ***loadbalancer*** service types
	b. Use `kubectl get svc` to get the ***external IP + Port***
	c. Paste it on the chrome browser and wait until the page loads.

```
EXPOSING THE SERVICES (VOTE + RESULT)

kubectl expose deployment vote --type=LoadBalancer --name=vote-lb --port=80 --target-port=80

kubectl expose deployment result --type=LoadBalancer --name=result-lb --port=80 --target-port=80


---------------------------
kubectl get svc

OUTPUT:

NAME         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
db           ClusterIP      34.118.230.238   <none>          5432/TCP         13h
kubernetes   ClusterIP      34.118.224.1     <none>          443/TCP          23h
redis        ClusterIP      34.118.238.233   <none>          6379/TCP         13h
result       NodePort       34.118.234.77    <none>          8081:31001/TCP   12h
result-lb    LoadBalancer   34.118.230.15    34.66.196.31    80:31522/TCP     11h
vote         NodePort       34.118.239.230   <none>          8080:31005/TCP   12h
vote-lb      LoadBalancer   34.118.238.27    104.198.214.9   80:31877/TCP     11h


NOTE:
1. Use the external IP + Port on the web browser (http://34.66.196.31:80)
2. Use the loadbalancer External IPs
```