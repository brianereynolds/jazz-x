# jazz-x

## Description
A set of microservices for demonstration purposes that uses nodeSelector to schedule workloads

## Pre-reqs
* A running AKS cluster
* Nodepools named "cntrlblue" and "svcsblue". See appendix for creation

## Installation
```
kubectl create ns jazz-x
helm install -n jazz-x jazz-x .`
```

## Appendix

### Create Nodepools

#### Install NodePool Manager
```
kubectl create ns operations
helm repo add k8smanagers https://k8smanagers.blob.core.windows.net/helm/
helm install nodepoolmanager k8smanagers/nodepoolmanager -n operations
```

#### Create nodepools 
_Set or change the values indicated_
```
SUB_ID="..."
RESOURCE_GROUP="..."

CLUSTER_NAME="lm-cluster"
K8S_VERSION="1.30.5"
VM_SIZE="Standard_DS2_v2"

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