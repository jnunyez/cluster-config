# cluster-config

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


