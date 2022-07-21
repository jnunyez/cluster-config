# cluster-config

## Quickstart

To start the cluster provisioning, follow these steps:

```console
oc apply -k bootstrap/overlays/default
```
This will first configure your cluster with the RH GitOps controller. These operator will drive the deployment of the other operators, jobs, and cluster customizations:

	* OpenShift GitOps operator

ArgoCD Applications in a determinate  order which is supported by SyncWave annotations:

	* Patcher (sync-wave -3)
	* GitOps (sync-wave -2)
	* Local Storage Operator (sync-wave -1)
	* Volume discovery (sync-wave 0 IMPORTANT TO RUN AFTER Local Storage Operator)
	* ODF operator (sync-wave 3 TO RUN After Volume discovery)
	* Advanced Cluster Management operator  (sync-wave 4)
	* Advanced Cluster Management instance (sync-wave 5)
	* RH OpenShift pipelines (sync-wave 5)


## Dir Structure

## Object management in the management cluster

OpenShift objects can be divided into two categories:

* Objects owned by Red Hat Openshift GitOps operator that  
	* creates objects in the management cluster
	* modifies objects in the workload clusters.

* Objects owned by Red Hat Advanved Cluster Management that
	* creates workload OpenShift clusters


## Object dependencies

* sync-waves, phases, objects

## Patterns

### DRY
Dont Repeat Yourself. We use tools like kustomize and the patch operator to avoid repeating yaml files.

### app-of-apps

To implement the app-of-apps pattern in ArgoCD, few solutions exist:

* ApplicationSets however placing the sync-wave annoation does not work between individual applications. There seems to be a way to achieve app-of-apps using appsets in [here](https://kubito.dev/posts/enable-argocd-sync-wave-between-apps/)

* An Application for each app/overlay. OPTION IMPLEMENTED NOW

* using a Helm Chart with Application defined as a template. 


