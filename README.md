# AppDynamics Cluster Agent Lab for MicroK8s

The goal of this lab is to deploy the [AppDynamics Cluster Agent](https://docs.appdynamics.com/display/PRO45/Monitoring+Kubernetes+with+the+Cluster+Agent) into a [MicroK8s](https://microk8s.io/) Kubernetes cluster in under five minutes. 

Access to an [AppDynamics SaaS Controller](https://www.appdynamics.com/) is required.

# MicroK8s

MicroK8s provides a simple and fast installation of a fully-conformant Kubernetes cluster. Read more about MicroK8s and Kubernetes at: [MicroK8s](https://microk8s.io/) and [Kubernetes](https://kubernetes.io/)

# Getting started

This lab is designed to run on Ubuntu, using MicroK8s and Docker CE. Clone this repository using:

  `git clone https://github.com/APPDRYDER/AppD-Cluster-Agent-MicroK8s.git`

  `cd AppD-Cluster-Agent-Microk8s`

# AppDynamics Cluster agent

Download the AppDynamics Ubuntu cluster agent from: [AppDynamics Downloads](https://download.appdynamics.com/download/#version=&apm=cluster-agent&os=&platform_admin_os=&appdynamics_cluster_os=&events=&eum=&page=1
) into the directory `AppD-Cluster-Agent-Microk8s`

This will download the zip archive `appdynamics-cluster-agent-ubuntu-4.5.16.780.zip` or a higher version.

# Install the AppDynamics Cluster Agent

From the directory `AppD-Cluster-Agent-Microk8s` unzip the Cluster Agent into the directory `cluster-agent` using:

  `unzip appdynamics-cluster-agent-ubuntu-4.5.16.780.zip -d cluster-agent`

# Configure the Cluster Agent

From the AppDynamics SaaS controller obtain the following configuration parameters: `controllerUrl`, `Account Name` and `Access Key`

In the cluster agent directory `cluster-agent` edit the resource definition `cluster-agent.yaml`:

  Modify the fields:
  ````
  appName: "<app-name>"
  controllerUrl: "http://<appdynamics-controller-host>:8080"
  account: "<account-name>"
  image: "docker.io/appdynamics/cluster-agent:4.5.16"
  ````

  Choose a unique `<app-name>` to represent this cluster. The above configuration uses the Cluster Agent `image` provided by AppDynamics for Ubuntu. This ok for this lab, however you can [Build the Cluster Agent Container Image](https://docs.appdynamics.com/display/PRO45/Build+the+Cluster+Agent+Container+Image) and source from a private respository.

  This lab deploys pods into the namespace `test`. Include additional `namespaces` to monitor by adding the field `nsToMonitor`, add the following namespaces:
  ````
    nsToMonitor:
      - test
      - default
      - appdynamics
      - kube-system
  ````
For additional configuration options review [Configure The Cluster Agent](https://docs.appdynamics.com/display/PRO45/Configure+the+Cluster+Agent)

# Update Ubuntu, install and configure MicroK8s Kubernetes Cluster:

In the directory `AppD-Cluster-Agent-Microk8s` run the following commands using the script `ctl.sh`

  ### Update Ubuntu
  ````./ctl.sh ubuntu-update````

  ### Install Docker Community Edition
  ````./ctl.sh docker-install````

  ### Install MicroK8s
  ````./ctl.sh k8s-install````

  The `docker-install` and `k8s-install` commands require the current ssh/shell session to be restarted, for the usermod command to work successfully. Exit the current ssh/shell session and re-enter before continuing.

  ### Start the MicroK8s Kubernetes Cluster
  ````./ctl.sh k8s-start````

Review the pods and services running in the cluster using:

```microk8s.kubectl get pods,services --all-namespaces```

```
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-7b67f9f8c-8s6zq                 1/1     Running   1          113s
kube-system   pod/metrics-server-v0.2.1-598c8978c-c2vjf   2/2     Running   0          107s

NAMESPACE     NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   10.152.183.1     <none>        443/TCP                  2m52s
kube-system   service/kube-dns         ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   113s
kube-system   service/metrics-server   ClusterIP   10.152.183.207   <none>        443/TCP                  107s
```
The `kube-dbs` and `metrics-server` should be running in the cluster.

# Deploy Pods to the MicroK8s Kubernetes Cluster

In the directory `AppD-Cluster-Agent-Microk8s` run the following commands using the script `ctl.sh`

  ````./ctl.sh pods-create````

The above command will create a namespace called "test" and deploy two pods (alpine1, alpine2) with single containers, and two services (busyboxes1, busyboxes2) each with two containers.

Review the K8s resource definitions in the directory [pods](https://github.com/APPDRYDER/AppD-Cluster-Agent-MicroK8s/tree/master/pods) for details of these resources.

Review what pods and namespaces are running in the cluster using the command:

  ````microk8s.kubectl get pods --all-namespaces````

The output should show two alpine pods and four busybox pods running:

```
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-7b67f9f8c-8s6zq                 1/1     Running   1          5m30s
kube-system   metrics-server-v0.2.1-598c8978c-c2vjf   2/2     Running   0          5m24s
test          alpine1                                 1/1     Running   0          20s
test          alpine2                                 1/1     Running   0          20s
test          busybox1a                               1/1     Running   0          20s
test          busybox1b                               1/1     Running   0          20s
test          busybox2a                               1/1     Running   0          19s
test          busybox2b                               1/1     Running   0          19s
```

# Deploy the AppDynamics Cluster Agent

Obtain the Account Access Key from the AppDynamics SaaS controller and configure the enviroment variable:

  `export APPDYNAMICS_AGENT_ACCOUNT_ACCESS_KEY=<access-key>`

In the directory `AppD-Cluster-Agent-Microk8s`

Deploy and start the AppDynamics Cluster Agent using the command:

  `./ctl.sh appd-create-cluster-agent`

Please note the above command will look for the Cluster Agent resources in the sub-directory `cluster-agent`

Check that the AppDynamics Cluster Agent and Operator are in the `Running` state, using the command:

  ````microk8s.kubectl get pods,services --all-namespaces````

If errors are reported, check the resource defintion file `cluster-agent.yaml`. Additonal steps are required for SSL and proxy services. See [Proxy and SSL Configuration](https://docs.appdynamics.com/display/PRO45/Configure+the+Cluster+Agent)

The Cluster Agent resources: agent, operator, secret and namespace, can be deleted using:

  `./ctl.sh appd-delete-cluster-agent`
  
Then restarted using:

  `./ctl.sh appd-create-cluster-agent`

# AppDynamics Cluster Agent Visibilty

Login into the AppDynamics Conroller and click through the `Servers` tab to the `Cluster` view. The cluster named by the field `appName: "<app-name>"` should be visible.

Click into this cluster to see the visibility AppDynamics provides into Kubernetes.

Details of how to use the AppDynamics Cluster Agent Visibility are provided here: [Use the Cluster Agent](https://docs.appdynamics.com/display/PRO45/Use+The+Cluster+Agent)

# Next Steps

Review the automation script `ctl.sh` for exact details of how this deployment was performed and configured.

Thank you for using this lab please provide feedback. Please fork, improve and submit pull requests.
