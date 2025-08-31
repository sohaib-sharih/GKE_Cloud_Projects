### 9.4 Azure AKS

1. Create a free account and subscribe for 200$ free credit.
2. You need to setup the cluster from the ***Create a Cluster*** under the Kubernetes Services Tab. And leave the default settings as they are, just do the following:
	a. Name the cluster
	b. Name the resource group
	c. Change the nodes to d series, something that is compatible to your free tier account.
	d. **Node size:**  ***Standard_D2ads_v6*** -> This works, tried and tested.
3. On the left hand side panel, under Kubernetes Services, you can click on the ***run command*** to run commands on your K8s Cluster.
4. On top right corner, you will see a button with a ***Terminal icon***, click on that, it will allow you to enter the **K8s Azure Cloud Shell**
5. You can either use the ***browser console*** Azure Cloud Shell, or configure it on your personal laptop or PC to run ***kubectl*** commands from your laptop by connecting it to the kubernetes cluster on your Azure Account.

### 9.4.1 How to configure Kubectl Azure CLI and configure shell to connect to your Azure K8s Cluster?

1. Reference link: https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli
2. Use the ref link to download the CLI if the previous link gets stuck in your terminal: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?view=azure-cli-latest&pivots=apt
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
3. Check version:
```
az version
{
  "azure-cli": "2.67.0",
  "azure-cli-core": "2.67.0",
  "azure-cli-telemetry": "1.1.0",
  "extensions": {}
}
```

4. **Login from your terminal**

	a. **Tenant ID:** Search for ***Tenant properties*** from the search bar of the console, to get the ID.
	b. **Second Option:** Home -> Microsoft Entra ID -> Overview *You will get the **tenant id** under the basic information tab*

```
az login

OR

az login --tenant <ID>
Note: Copy the ID from your Cluster Dashboard from the Subscription ID section

OR
az login --use-device-code

Note:
1. Use any of the above options to login to the azure account.
2. Use your VS Code wsl or ubuntu shell.
3. If you try to use ubuntu shell separately, it may not be able to open up the default browser through your CLI, because your ubuntu doesn't have a GUI, you are using it through WSL installed inside Windows, therefore using vscode is a better option.
```

5. **Connect to the Azure Cluster**

```
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

1. Use the resource group name from your Dashboard cluster overview.
2. Cluster name to your cluster name
   
----------------------

Check
kubectl config view
```
6. **Deploy Services + Deployments**

```
kubectl create -f redis-deployment.yaml
kubectl create -f redis-service.yaml
kubectl create -f worker-deployment.yaml
kubectl create -f result-deployment.yaml
kubectl create -f result-service.yaml
kubectl create -f vote-deployment.yaml
kubectl create -f vote-service.yaml
kubectl create -f db-deployment.yaml
kubectl create -f db-service.yaml

----------------------------
check
kubectl get pods,deployments,nodes,svc

OUTPUT:

NAME                          READY   STATUS    RESTARTS   AGE
pod/db-fdcb96d6b-pr44f        1/1     Running   0          130m
pod/redis-667f998757-qhqbf    1/1     Running   0          130m
pod/result-84c95d764c-nlpxj   1/1     Running   0          130m
pod/vote-679d9c7ccf-qgthq     1/1     Running   0          75m
pod/worker-f4dfddcd6-cxsjx    1/1     Running   0          75m

NAME                                     STATUS   ROLES    AGE    VERSION
node/aks-agentpool-30975695-vmss000001   Ready    <none>   2d8h   v1.32.6
node/aks-agentpool-30975695-vmss000002   Ready    <none>   2d8h   v1.32.6

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db       1/1     1            1           130m
deployment.apps/redis    1/1     1            1           130m
deployment.apps/result   1/1     1            1           130m
deployment.apps/vote     1/1     1            1           75m
deployment.apps/worker   1/1     1            1           75m

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
service/db           ClusterIP      10.0.132.144   <none>          5432/TCP         130m
service/kubernetes   ClusterIP      10.0.0.1       <none>          443/TCP          2d9h
service/redis        ClusterIP      10.0.121.60    <none>          6379/TCP         130m
service/result       NodePort       10.0.123.196   <none>          8081:31001/TCP   129m
service/result-lb    LoadBalancer   10.0.189.228   20.242.203.69   80:32098/TCP     67m
service/vote         NodePort       10.0.107.36    <none>          8080:31000/TCP   75m
service/vote-lb      LoadBalancer   10.0.1.24      57.151.44.64    80:30610/TCP     65m
```

7. **Configure the inbound traffic:**
	a. By default, azure doesn't allow inbound traffic, you will have to manually configure it.


### 9.4.2 Inbound traffic is restricted (SETTINGS gemini)

1. If you are using a ***loadbalancer type*** of service configuration in your deployment, then kubernetes will assign a public/external ip to your services, exposing your service to the internet. Where the external ip connects to the port of the service, and the service port then uses matchLabels to identify the app pods to route traffic internally using the cluster IP.
2. Find the: **MC_voting-resource-group_example-voting-app_eastus** 
	a. Type: ***AKS Node Resource Group*** or just ***Resource Group***, click on Resource Groups
	b. You Resource Group would be available on top of the list, in most cases, it follows the convention of ***your cluster name***, following by the ***region***, and the word ***resource*** would be mentioned in the middle. Would be similar to the following: *****MC_voting-resource-group_example-voting-app_eastus*****
	c. Click on an item on the list menu that has ***NSG*** written on it. Example: *aks-agentpool-29238863-nsg*
	d. Then go to the ***Settings*** tab on the *side panel* and click on ***inbound security rules***
	e. Click on the ***Add button*** on top of the table, mostly on the left side.
3. **Inbound Security Rules Settings:**
	a. **Source:** any
	b. **Source port ranges**: Asterisk (`*`)
	c. **Destination**: any
	d. **Service**: any
	e. **Action:** allow
	f. **Priority**: 200
	g. Give it a name and description(optional) and save it.

### 9.4.3 The End-to-End Traffic Flow (NodePort + LoadBalancer Gemini)

1. **End User Access (External IP + Port):** The end user initiates a connection to the public IP address provided by the `LoadBalancer` service (e.g., `57.151.44.64`). They use the port specified in the service definition, which in your case is `80`. This is the **Service's external port**.
2. **Load Balancer Routing:** The Azure Load Balancer (or the cloud provider's equivalent) is the first point of contact for this traffic. It's configured to listen on that public IP address and port `80`. It then forwards this traffic to a node in the Kubernetes cluster on a specific port, known as the `NodePort`. This is where your previous output of `80:30610/TCP` comes into play. The load balancer knows to send traffic from its public IP on port `80` to one of the worker nodes on port `30610`.
3. **Internal Cluster Routing (kube-proxy):** This is where the magic of Kubernetes networking happens. Each node in the cluster runs a component called `kube-proxy`. This component constantly watches the Kubernetes API for service and endpoint changes. When it sees a `LoadBalancer` service, it configures the node's networking rules (specifically `iptables` or IPVS) to:
	a. Listen on the `NodePort` (`30610` in your case).
    b. Route traffic arriving on that `NodePort` to the Service's `ClusterIP` (`10.0.1.24`) and its `port` (`80`).
4. **Service-to-Pod Routing:** The `kube-proxy` rules, when applied to the `ClusterIP` and `port`, perform the final routing. They look at the `Service` definition's `selector` (e.g., `app=vote`) to identify the pods that match that label. Kubernetes then selects one of these matching pods and routes the traffic from the `Service` port (`80`) to the pod's `targetPort` (`80`), which is the port your application is actually listening on.
### Summary of the Ports and IPs in the Chain

1. **External IP:** The public-facing address used by the end user.
2. **Service Port (`.spec.ports[].port`):** The port the `Service` itself is exposed on internally, which is used by the `LoadBalancer`. This is typically the same as the external port (e.g., `80`).
3. **NodePort (`.spec.ports[].nodePort`):** A port opened on every node in the cluster (`30610` in your case) that the load balancer uses to send traffic to.
4. **Pod's `targetPort` (`.spec.ports[].targetPort`):** The port that the containerized application is actually listening on inside the pod.

**Conclusion**: the external IP and port are the entry point, which then uses the `LoadBalancer` to forward traffic to the service, and the service uses `matchLabels` to send traffic to the correct pods. You've nailed the core concept.

