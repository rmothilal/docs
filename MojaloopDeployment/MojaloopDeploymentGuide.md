#moving to documentation#

# Deployment and Setup Introduction
This document will provide guidelines to deploy and configure the Mojaloop applications on a local environment, utilizing Kubernetes within Docker.

* [Software List](#1-software-list)
  * [Deployment Recommendations](#11-deployment-recommendations)
  * [Local Deployment and Testing Tools](#12-local-deployment-and-testing-tools)
* [Deployment](#2-deployment)
  * [Kubernetes](#21-kubernetes)
    * [Kubernetes Installation with Docker](#211-kubernetes-installation-with-docker)
    * [Kubernetes environment setup](#212-kubernetes-environment-setup)
  * [Helm](#22-helm)
    * [Helm Chart Installation](#221-helm-chart-installation)
  * [Postman](#23-postman)
    * [Installing Postman](#231-installing-postman)
    * [Setup Postman](#231-setup-postman)
* [Errors During Setup](#24-errors-on-setup)

## 1 Software List 
Before proceeding, please have a look at [Deployment Recommedations](#11-deployment-recommendations) to insure the minimum resource requirements are availlable.

### 1.1 Deployment Recommendations
This provides environment resource recommendations with a view of the infrastructure architecture.

##### Resources Requirements:
- Control Plane (i.e. Master Node)
   ```http request
   https://kubernetes.io/docs/setup/cluster-large/#size-of-master-and-master-components
   ```
   - 3x Master Nodes for future node scaling and HA (High Availability)
- ETCd Plane:
   ```http request
   https://coreos.com/etcd/docs/latest/op-guide/hardware.html
   ```
    - 3x ETCd nodes for HA (High Availability)
- Compute Plane (i.e. Worker Node):

  TBC once load testing has been concluded. However the current general *recommended size:
    - 3x Worker nodes, each being:
      - 4x vCPUs, 16GB of RAM, and 40gb storage
      
  *Note that this would also depend on your underlaying infrastructure, and it does NOT include requirements for persistent volumes/storage.
  
  ![Mojaloop Deployment Recommendations - Infrastructure Architecture](../Wiki/KubeInfrastructureArch.svg)
  
  [Mojaloop Deployment Recommendations - Infrastructure Architecture](../Wiki/KubeInfrastructureArch.svg)


### 1.2 Local Deployment and Testing Tools
The tool set to be deployed as part of the deployment process.

|Tool|Required/Optional|Description|Install Info|
|---|---|---|---|
|Docker|Required|<ul><li>Docker Engine and CLI Client</li><li>Local Kubernetes single node cluster</li></ul>|<ul><li>[https://docs.docker.com/install](https://docs.docker.com/install)</li></ul>|
|Kubectl|Required|<ul><li>Kubernetes CLI for Kubernetes Management</li><li>Note Docker installs this part of Kubernetes install</li></ul>|<ul><li>[https://kubernetes.io/doc/tasks/tools/install-kuberctl](https://kubernetes.io/doc/tasks/tools/install-kuberctl)</li><li>Docker Kubernetes Install (as per this guide)</li><li>Mac: brew install kubernetes-cli</li></ul>|
|Kubectx|Optional(useful tool)|<ul><li>Kubernetes CLI for Kubernetes Context Management Helper</li><li>Note Docker insgtalls this as part of Kubernetes install</li></ul>|<ul><li>[https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx)</li><li>Mac: brew install kubectx</li></ul>|
|Kubetail|Optional(useful tool)|<ul><li>Bash script that enables you to aggregate (tail/follow) logs from multiple pods into one stream. This is the same as running `kubectl logs -f` but for multiple pods.</li><li>Example usage `kubetail moja.* -n demo`</li></ul>|<ul><li>https://github.com/johanhaleby/kubetail</li></ul>|
|Helm|Required|<ul><li>Helm helps you manage Kubernetes applications</li><li>Helm charts help you define, install and upgrade even the most complex Kubernetes application</li></ul>|<ul><li>[https://docs.helm.sh/using_helm/#installing-helm](https://docs.helm.sh/using_helm/#installing-helm)</li><li>Mac: brew install kubernetes-helm</li></ul>|
|Postman|Required|<ul><li>Postman is a Google Chrome application for the interacting with HTTP API's. It presents you with a friendly GUI for the construction requests and reading responces.</li></ul>|<ul><li>[https://www.getpostman.com/apps](https://www.getpostman.com/apps)</li></ul>|

## 2 Deployment
This section will guide the reader through the deployment process to setup Kubernetes within Docker.

### 2.1 Kubernetes
If you are new to Kubernetes it is strongly recommended to familiarize yourself with Kubernetes. [Kubernetes Concepts](https://kubernetes.io/docs/concepts/overview/) is a good place to start and will provide an overview.

The following are Kubernetes concepts used within the project. An understanding of these concepts is imperative before attempting the deployment;
- Deployment
- Pod
- ReplicaSets
- Service
- Ingress
- StatefulSet
- DaemonSet
- Ingress Controller
- ConfigMap
- Secret

#### 2.1.1 Kubernetes Installation with Docker
To install Kubernetes with Docker, follow the steps below;
 - Click on the Docker icon on the status barr
   - Select **Preferences**
   - Go to **Advanced**
     - Increase the CPU allocation to at least 4
     - Increase the Memory allocation to at least 8.0 GiB

![Kubernetes Install with Docker 1](../Wiki/KubernetesInstallWithDocker-1.png)

[Kubernetes Install with Docker 1](../Wiki/KubernetesInstallWithDocker-1.png)

   - Go to **Kubernetes**
     - Select **Enable Kubernetes** tick box
     - Make sure **Kubernetes** is selected
     - Click **Apply**
     - Click **Install** on the confirmation tab. 
 - The option is available to wait for completion or run as a background task.

![Kubernetes Install with Docker 2](../Wiki/KubernetesInstallWithDocker-2.png)

[Kubernetes Install with Docker 2](../Wiki/KubernetesInstallWithDocker-2.png)

#### 2.1.2 Kubernetes environment setup:
The following are all command line executables specifically for Mac. 
1. List the current Kubernetes context;
   ```bash
   kubectl config get-contexts
   ```
   or
   ```bash
   kubectx
   ```
1. Change your Contexts; 
   ```bash
   kubectl config use-contexts
   ```
   or
   ```bash
   kubectx docker-for-desktop
   ```
1. Install Kubernetes Dashboard roles, services & deployment. (Alternative install for Dashboard using Helm: [kubernetes-dashboard](https://github.com/helm/charts/tree/master/stable/kubernetes-dashboard))

   **IMPORTANT:**  Always verify current [kubernetes-dashboard](https://github.com/kubernetes/dashboard) yaml file for the below create command.
   ```bash
   kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
   ```
1. Verify Kubernetes Dashboard;
   ```bash
   kubectl get pod --namespace=kube-system |grep dashboard
   ```
1. Start proxy for local UI in new terminal;
   ```bash
   kubectl proxy ui
   ```
1. Open URI in default browser
   ```bash
   http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/
   ```
   Select __Token__. Generate a token to use there by:
   ```bash
   kubectl -n kube-system get secrets | grep dashboard-token
   ```
   ___
   The token to use is shown on the last line of the output of that command.
   ___
   ```bash
   kubectl -n kube-system describe secrets/kubernetes-dashboard-token-btbwf
   ```
   The __{kubernetes-dashboard-token-btbwf}__ is retrieved from the output in the previous step.
   For more information on generating the token, follow the __Authentication__ link in the window.
   
   ![kubernetes-dashboard](../Wiki/kubernetesDashboard.png)

   [kubernetes-dashboard](../Wiki/kubernetesDashboard.png)

1. Config Helm CLI and install Helm Tiller on K8s cluster
   ```bash
   helm init
   ```
1. Validate Helm Tiller is up and running
   ```bash
   kubectl -n kube-system get po | grep tiller
   ```
1. Add mojaloop repo to your Helm config (optional)
   ```bash
   helm repo add mojaloop http://mojaloop.io/helm/repo/
   ```
1. Add the incubator. This is needed to resolve Helm Chart dependencies required by Mojaloop charts
   ```bash
   helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
   ```

1. Update helm repositories
   ```bash
   helm repro update
   ```
1. Install nginx-ingress for load balancing & external access
   ```bash
   helm --namespace kube-public install stable/nginx-ingress
   ```
1. Add the following to your /ect/hosts
   ```text
   127.0.0.1       interop-switch.local central-kms.local forensic-logging-sidecar.local central-ledger.local central-end-user-registry.local central-directory.local central-hub.local central-settlements.local ml-api-adapter.local
   ```  
1. Test system health in your browser after installation
   ___
   ml-api-adapter health test
   ```http request
   http://ml-api-adapter/health
   ```
   ___
   central-ledger health test
   ```http request
   http://central-ledger/health
   ```

### 2.2 Helm
Please review [Mojaloop Helm Chart](../Diagrams/MojaloopHelmChart.md) to understand the relationships between the deployed Mojaloop helm charts.

##### 2.2.1 Helm Chart Installation
This section will provide guidelines to delete, list, install and upgrade of the helm charts. For a comprehensive deployment documentation, please see [Helm Chart Deployment](https://github.com/mojaloop/helm/blob/master/README.md)

1. Lets start by **listing** the current helm charts deployed
   ```bash
   helm list
   ```

1. If you would like to **delete** a deployed helm chart
   ```bash
   helm del --purge moja
   ```
   __Note:__ for demo purposes we are using __moja__ as the chart __name__. Please verify and use the correct chart name from the listing above.
		
1. To **install** Mojaloop chart(s)
   
   To install the full mojaloop project
   ```bash
   helm install --namespace=demo --name=moja mojaloop/mojaloop
   ```
   Alternative directly from the repository: 
   ```bash
   helm install --namespace=demo --name=moja --repo=http://mojaloop.io/helm/repo mojaloop
   ```

   __or__ install a specific mojaloop chart eg. Central-ledger
   ```bash
   helm install --namespace=demo --name=moja mojaloop/centralledger
   ```
   Alternative directly from the repository: 
   ```bash
   helm install --namespace=demo --name=moja --repo=http://mojaloop.io/helm/repo centralledger
   ```
1. To upgrade Mojaloop chart(s)
   Note: 'v5.1.1' is an example value. 
   ```bash
   helm upgrade moja --set central.centralledger.centralledger-services.containers.api.image.tag=v5.1.1-snapshot mojaloop
   ```
1. To upgrade a specific chart eg. Central-ledger	     
   ```bash
   helm upgrade moja --set centralledger-services.containers.api.image.tag=v5.1.1-snapshot mojaloop/centralledger
   ```

### 2.3 Postman
Postman is used to send requests and receive responses.

##### 2.3.1 Installing Postman
Please, follow these instructions: [Get Postman](https://www.getpostman.com/postman)

Alternatively on **Ubuntu** you may run:
```bash
wget https://dl.pstmn.io/download/latest/linux64 -O postman.tar.gz
sudo tar -xzf postman.tar.gz -C /opt
rm postman.tar.gz
sudo ln -s /opt/Postman/Postman /usr/bin/postman
```

#### 2.3.1 Setup Postman
* Download this file https://raw.githubusercontent.com/mojaloop/postman/master/Mojaloop.postman_collection.json
* Open **Postman**
* Click **Import** and then **Import File**
* Select the *Mojaloop.postman_collection.json* file you downloaded
* You'll now need to import environment variables. For local testing, download this file https://raw.githubusercontent.com/mojaloop/postman/master/environments/MojaloopLocal.postman_environment.json
* Click **Import** and then **Import File**
* Select the *MojaloopLocal.postman_environment.json* file you downloaded
* In the imported collection, navigate to the *central_ledger* directory  

### 2.4 Errors On Setup
* `central-ledger’s server IP address could not be found.
   ERR_NAME_NOT_RESOLVED`
   Resolved by:
   * Verify that a helm chart(s) was installed by executing
     ```bash
     helm list
     ```
     If the helm charts are not listed, see the [Helm Chart Installation](#221-helm-chart-installation) section to install a chart.
___
