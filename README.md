# Introduction

The purpose of this document is to identify the steps required to deploy a cluster using cluster API in OCI.

# Prerequisites

1. VM running OL8.

2. Policy required.

    ```
    allow any-user to manage instance-family in compartment <comp-name> 
    allow any-user to manage virtual-network-family in compartment <comp-name>
    allow any-user to manage volume-family in compartment <comp-name>
    ```

3. Install [kind](https://kind.sigs.k8s.io/)

4. Install `kubectl`, `helm`, `docker`.

5. Install [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start#install-clusterctl).

6. Install and setup oci-cli

7. Install and setup [packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli)

8. Install [image-builder](https://image-builder.sigs.k8s.io/capi/quickstart.html)



# Build custom image

Follow the steps from [here](https://github.com/robo-cap/image-builder/blob/main/images/capi/README.md).

Update the image OCID in the `cluster-export.sh` file.

# Installation

1. Create a new K8s cluster using kind.

    ```
    kind create cluster
    ```

2. Create the cluster:

    The official documentation [page](https://oracle.github.io/cluster-api-provider-oci/introduction.html).

    Update the OCIDs in the file `cluster-export.sh`.
    
    ```  
    source cluster-export.sh
    clusterctl init --infrastructure oci
    
    clusterctl generate cluster poc --from cluster-template.yaml | kubectl apply -f -
    
    clusterctl get kubeconfig poc -n default > poc.kubeconfig
    ```

3. Install Oracle CCM

    Update the OCIDs in the file `cloud-provider-example.yaml`.

    ```
    kubectl  --kubeconfig poc.kubeconfig create secret generic oci-cloud-controller-manager \
     -n kube-system \
     --from-file=cloud-provider.yaml=cloud-provider-example.yaml
  
    kubectl apply -f oci-cloud-controller-manager-rbac.yaml --kubeconfig poc.kubeconfig 
    kubectl apply -f oci-cloud-controller-manager.yaml --kubeconfig poc.kubeconfig 
    ```

    Install CNI

    ```
    helm repo add projectcalico https://docs.tigera.io/calico/charts
    helm install calico projectcalico/tigera-operator --kubeconfig poc.kubeconfig --version v3.28.2 --create-namespace --namespace tigera-operator -f values-override.yaml
    ```

If some calico pods are failing, fix the NSG (allow traffic between control_plane nsg and workers nsg).

