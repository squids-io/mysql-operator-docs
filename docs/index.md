<h1>Concepts</h1>

The MySQL [operator](https://coreos.com/blog/introducing-operators.html)
manages MySQL clusters on Kubernetes (K8s):

1. The operator watches additions, updates, and deletions of MySQL cluster
   manifests and changes the running clusters accordingly.  For example, when a
   user submits a new manifest, the operator fetches that manifest and spawns a
   new MySQL cluster along with all necessary entities such as K8s
   StatefulSets and MySQL roles.  See this
   [MySQL cluster manifest](../manifests/minimal-mysql-manifest.yaml)
   for settings that a manifest may contain.

2. The operator also watches updates to [its own configuration](../manifests/configmap.yaml)
   and alters running MySQL clusters if necessary.  For instance, if the
   Docker image in a pod is changed, the operator carries out the rolling
   update, which means it re-spawns pods of each managed StatefulSet one-by-one
   with the new Docker image.

3. Finally, the operator periodically synchronizes the actual state of each
   MySQL cluster with the desired state defined in the cluster's manifest.

4. The operator aims to be hands free as configuration works only via manifests.
   This enables easy integration in automated deploy pipelines with no access to
   K8s directly.

## Scope


## Overview of involved entities



## Status



## Talks



## Posts


