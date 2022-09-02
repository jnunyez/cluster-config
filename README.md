# cluster-config

## Requirements

### Management Cluster

Assumption here is that we start from a fresh cluster deployed on VMs composed by three control plane nodes.


### Workload Cluster

Assumption here is that the workload cluster is deployed on VMs for the control plane nodes and baremetal for the workload nodes.


## Quickstart

### Provision Sealed Secret on Management Cluster  
For repeatibility and reusage purposes of sealed secrets object, before starting the cluster provisioning assure that you have the keys and certificates necessary to decrypt the sealed secrets object stored in this repo in folder `./manifests/hub-cluster/sealed-secrets/02-secrets/overlays/default`. Once you get the original keys and certificates in a secret object, you can proceed to provision them in the same namespace where you plan to deploy the sealed secret operator. A secret object embedding original keys looks like this:

```apiVersion: v1
data:
  tls.crt: "YOUR CERT GOES HERE"
  tls.key: "YOUR KEY GOES HERE"
kind: Secret
metadata:
  name: sealed-secrets-secret
  namespace: sealed-secrets
type: kubernetes.io/tls
```

Now provision the secret in advance in the namespace argoCD will deploy the operator:

```console
oc create -f 01-sealed-secrets-secret.yaml
```

### Provision Management Cluster services

Now you are good to go and can start provisioning the cluster config of the hub cluster:

```console
until oc apply -k bootstrap/overlays/default; do sleep 3; done
```

The command above  will first install the Openshift GitOps operator. This operator will drive the deployment of the other operators, jobs, and cluster customizations my management cluster needs. We use ArgoCD ApplicationSet to install the following set of sercvices the management cluster needs:

	* Patcher operator
	* GitOps operator
	* Local Storage Operator 
	* Sealed Secrets operator
	* ODF operator 
	* Advanced Cluster Management operator  
	* Advanced Cluster Management instance
	* Assisted Service 
	* Red Hat OpenShift pipelines 

## Provision Workload Cluster from Management Cluster

Provision workload cluster using ArgoCD ApplicationSet:

```console
oc create -f argo-appset/workload-cluster.yaml
```

## Dir Structure

* bootstrap: Here we put the items. The base directory contains GitOps controller configuration, whereas overlays contain GitOps controller configurations. There is only one default overlay. The default directory contains kustomization.yaml files with argoCD applications.

* argo-appset: Here we include all the ArgoCD ApplicationSets. 

* manifests: This directory embeds all the YAML files to deploy via the respective ArgoCD application all the specifics customizations necessary to achieve the aimed functionality in the cluster. The core subdir contains specific customizations for the GitOps controller that will be also managed by ArgoCD. Yes, ArgoCD lifecycle is managed by ArgoCD.  

## Object management in the management cluster

OpenShift objects can be divided into two categories:

* Objects owned by Red Hat Openshift GitOps operator that  
	* creates objects in the management cluster
	* modifies objects in the workload clusters.

* Objects owned by Red Hat Advanved Cluster Management that
	* creates workload OpenShift clusters


## Patterns

### DRY
Dont Repeat Yourself. We use tools like ArgoCD ApplicationSets, kustomize and the patch operator to avoid repeating redundant yaml files.

### app-of-apps

To implement the app-of-apps pattern in ArgoCD, few solutions exist:

* ApplicationSets however placing the sync-wave annoation does not work between individual applications. There seems to be a way to achieve app-of-apps using appsets in [here](https://kubito.dev/posts/enable-argocd-sync-wave-between-apps/). *Solution Under Test*

* An Application for each app/overlay. OPTION IMPLEMENTED NOW

* using a Helm Chart with Application defined as a template. Discarded so far for the sake of ease the usage of the code.

