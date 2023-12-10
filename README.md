# Reschedule dynamically pods or Openshift Virtualization vms on a new node

Read
  * https://github.com/kubernetes-sigs/descheduler#readme

Scheduling in Kubernetes is the process of binding pending pods to nodes, and is performed by a component of Kubernetes called kube-scheduler. 

The scheduler's decisions, whether or where a pod can or can not be scheduled, are guided by its configurable policy which comprises of set of rules, called predicates and priorities. 

The scheduler's decisions are influenced by its view of a Kubernetes cluster at that point of time when a new pod appears for scheduling. 

As Kubernetes clusters are very dynamic and their state changes over time, there may be desire to move already running pods to some other nodes for various reasons:

  * Some nodes are under or over utilized
    
  * The original scheduling decision does not hold true any more, as taints or labels are added to or removed from nodes, pod/node affinity requirements are not satisfied any more.
    
  * Some nodes failed and their pods moved to other nodes.
    
  * New nodes are added to clusters.

Consequently, there might be several pods scheduled on less desired nodes in a cluster. 

Descheduler, based on its policy, finds pods that can be moved and evicts them. 

Please note, in current implementation, descheduler does not schedule replacement of evicted pods but relies on the default scheduler for that.

The Descheduler Operator supported by Redhat has to be installed (in current version in a pre-created namespace : ).

The Operator will not permit all options directly available from the upstream config.

## Redistribute load on nodes

Following strategy, you can choose a profile that defines which nodes are over utilized and which nodes are under utilized

By example 

  * a threshold of 40% of ressources is the low threshold (by example cpu requests is 30% of cpu available o the node)

  * a threshold of 70% of ressources is the high threshold (by example memory requests is 72% of memory available o the node)

When there are nodes over utilized and some under utilized, the Descheduler will find pods to be removed in the over utilized nodes and the scheduler will reschedule them on the under utilized nodes.

The pods can be targeted by namespace or by priority class, if the pods are not evicted (ex: live migration for ocp-v vms), they should not have a poddisruptionbudget to protect their deletion (ex ocp-v vms with no live migration).

## Re-apply affinity/taint/topology contraints

If context has changed (node labels, taints, etc), a constraint that was ok at scheduling time for a pod, could not be valid later.

Kubernetes scheduler will never re-schedule a pod after it is already scheduled and is running.

This is what the Descheduler will do, see constraints are not valid any more and re-apply them by removing the pod not compliant with the constraint (affinity, ...)

## Pod Life Time

## Pod high restart failure

