# cluster-config

## Quickstart

For repeatibility and reusage purposes of sealed secrets object, before starting the cluster provisioning assure that you have the keys and certificates necessary to decrypt the sealed secrets object stored in this repo in folder `./manifests/sealed-secrets/02-secrets/overlays/`default. Once you get the original keys and certificates in a secret object, you can proceed to provision them in the same namespace where you plan to deploy the sealed secret operator. A secret object embedding original keys looks like this:

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

Now you are good to go and can start provisioning the cluster:

```console
until oc apply -k bootstrap/overlays/default; do sleep 3; done
```

This will first configure your cluster with the GitOps controller. The operator will drive the deployment of the other operators, jobs, and cluster customizations:

	* OpenShift GitOps operator

ArgoCD Applications in a determinate  order which is supported by SyncWave annotations:

	* Patcher (sync-wave -3)
	* GitOps (sync-wave -2)
	* Local Storage Operator (sync-wave -1)
	* Sealed Secrets (sync-wave -1)
	* Volume discovery (sync-wave 0 IMPORTANT TO RUN AFTER Local Storage Operator)
	* ODF operator (sync-wave 3 TO RUN After Volume discovery)
	* Advanced Cluster Management operator  (sync-wave 4)
	* Advanced Cluster Management instance (sync-wave 5)
	* RH OpenShift pipelines (sync-wave 5)


## Dir Structure

* bootstrap: Here we put the items. The base directory contains GitOps controller configuration, whereas overlays contain GitOps controller configurations. There is only one default overlay. The default directory contains kustomization.yaml files with argoCD applications.

* argo-config: Here we include all the argocd applications. 

* manifests: This directory embeds all the YAML files to deploy via the respective ArgoCD application all the specifics customizations necessary to achieve the aimed functionality in the cluster. The core subdir contains specific cusotmizations for the GitOps controller that will be also managed by ArgoCD. Yes, ArgoCD lifecycle is managed by ArgoCD.  

## Object management in the management cluster

OpenShift objects can be divided into two categories:

* Objects owned by Red Hat Openshift GitOps operator that  
	* creates objects in the management cluster
	* modifies objects in the workload clusters.

* Objects owned by Red Hat Advanved Cluster Management that
	* creates workload OpenShift clusters


## Object dependencies

* sync-waves, phases, objects

* SkypDryRun: Important when you have a CR that is created in a different sync stage 

## Patterns

### DRY
Dont Repeat Yourself. We use tools like kustomize and the patch operator to avoid repeating yaml files.

### app-of-apps

To implement the app-of-apps pattern in ArgoCD, few solutions exist:

* ApplicationSets however placing the sync-wave annoation does not work between individual applications. There seems to be a way to achieve app-of-apps using appsets in [here](https://kubito.dev/posts/enable-argocd-sync-wave-between-apps/)

* An Application for each app/overlay. OPTION IMPLEMENTED NOW

* using a Helm Chart with Application defined as a template. (This could save a lot of yaml redundancies.)

