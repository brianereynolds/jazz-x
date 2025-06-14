# jazz-x

## Description
A set of microservices for demonstration purposes. Using [NodePool](https://github.com/brianereynolds/nodePoolManager-helm) and [Workload](https://github.com/brianereynolds/workloadManager-helm) they can be moved between different node pools
## Pre-reqs
* A running AKS cluster
* Nodepools named "cntrlblue" and "svcsblue". See appendix for creation instructions

## Installation
```
kubectl create ns jazz-x
helm install -n jazz-x jazz-x .`
```

## Procedure
A set of Github Action pipelines are available [here](.github/workflows)

See the description in each pipeline for relevant details

0. Install [NodePool](https://github.com/brianereynolds/nodePoolManager-helm) and [Workload](https://github.com/brianereynolds/workloadManager-helm) helm charts
1. Upgrade System Pool using NodepoolManager
2. Create Surge Pools using NodepoolManager
3. Move Kanin MQ stateful set using WorkloadManager
4. Move Config deployment using WorkloadManager
5. Move multiple deployments using WorkloadManager
6. Move Central deployment using WorkloadManager
7. Delete unused node pools using NodepoolManager

Workflows available [here](https://github.com/brianereynolds/jazz-x/actions)

To run this in GitHub Actions, you will need to have the following defined in your account
* Secret -> KUBECONFIG: The contents of your $HOME/.kube/config used to connect to your AKS cluster
* Variables -> SUB_ID: Azure Subscription ID that contains your AKS cluster
* Variables -> RESOURCE_GROUP: The resource group name that contains your AKS cluster
* Variables -> CLUSTER_NAME: The name of your AKS cluster

## Appendix

### Create Nodepools

#### Install NodePool Manager
```
kubectl create ns operations
helm repo add k8smanagers https://k8smanagers.blob.core.windows.net/helm/
helm install nodepoolmanager k8smanagers/nodepoolmanager -n operations
```

#### Create nodepools 
The following GitHub Actions [pipeline can be used ](./.github/workflows/2_Create_Surge_Pools.yaml)

Alternatively, to create manually, _set or change_ the "..." values
```
SUB_ID="..."
RESOURCE_GROUP="..."

CLUSTER_NAME="..."
K8S_VERSION="1.30.7"        # Change as required
VM_SIZE="Standard_DS2_v2"   # Change as required

cat <<EOF | kubectl apply -f -
apiVersion: k8smanagers.greyridge.com/v1
kind: NodePoolManager
metadata:
  labels:
    app.kubernetes.io/name: nodepoolmanager
    app.kubernetes.io/managed-by: kustomize
  name: nodepoolmanager-sample
spec:
  subscriptionId: $SUB_ID
  resourceGroup: $RESOURCE_GROUP
  clusterName: $CLUSTER_NAME
  nodePools:
    - name: "svcsblue"
      properties: {
        orchestratorVersion: $K8S_VERSION,
        vmSize: $VM_SIZE,
        enableAutoScaling: true,
        minCount: 0,
        maxCount: 5,
        availabilityZones: [ "1" ]
      }
    - name: "cntrlblue"
      properties: {
        orchestratorVersion: $K8S_VERSION,
        vmSize: $VM_SIZE,
        enableAutoScaling: true,
        minCount: 0,
        maxCount: 1,
        availabilityZones: [ "1" ]
      }
EOF


```