<h1>User Guide</h1>
Learn how to work with the MySQL Operator in a Kubernetes (K8s) environment.

## Create ServiceAccount for MySQLCluster

Specify the namespace of the deploy MySQL cluster

```shell
export NAMESPACE=<your-namespace>
```

Apply ServiceAccount and ClusterRoleBinding

```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysql-operator
  namespace: $NAMESPACE
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: mysql-operator-rbac-$NAMESPACE
  labels:
    app.kubernetes.io/name: mysql-operator
subjects:
  - kind: ServiceAccount
    name: mysql-operator
    namespace: $NAMESPACE
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
EOF
```


## Create a manifest for a new MySQL cluster

Make sure you have [set up](quickstart.md) the operator. Then you can create a
new MySQL cluster by applying manifest like this [minimal example](../manifests/minimal-mysql-manifest.yaml):

```yaml
kubectl apply -f - <<EOF
apiVersion: mysql.grds.cloud/v1
kind: MysqlCluster
metadata:
  name: mysqlcluster-sample
  namespace: $NAMESPACE
spec:
  clusterSpec:
    version: "5.7"
    mode: HACluster
  databaseResource:
    resources:
      limits:
        cpu: "1"
        memory: 1Gi
      requests:
        cpu: 500m
        memory: 1Gi
    storage:
      size: 21474836480
  replicas: 2
  slaveReplicas: 1
  configTemplate: "mysql-5.7.0-audit"
EOF
```

Once you cloned the MySQL Operator [repository](https://github.com/GrdsCloud/mysql-operator-docs.git)
you can find this example also in the manifests folder:

```bash
kubectl create -f manifests/minimal-mysql-manifest.yaml
```

Make sure, the `spec` section of the manifest contains at least a `replicas`, the
`slaveReplicas` and the `MySQL` object with the `version` specified.
The minimum volume size to run the `MySQL` resource is `1Gi`.

## Watch pods being created

Check if the database pods are coming up. Use the label `created-by=grds.cloud` to filter

```bash
kubectl get pods -l created-by=grds.cloud -w
```

The operator also emits K8s events to the MySQL CRD which can be inspected
in the operator logs or with:

```bash
kubectl describe Mysqlcluster mysqlcluster-sample
```

## Connect to MySQL master database

With a `port-forward` on one of the database pods (e.g. the master) you can
connect to the MySQL database. Use labels to filter for the master pod of
our test cluster.

```bash
# set up port forward to master service
kubectl port-forward  svc/mysqlcluster-sample-primary 30306:3306
```

Open another CLI and connect to the database. Use the generated secret of the
`mysql` root user to connect to our `mysqlcluster-sample` master running in k8s

```bash
export PASSWORD=$(kubectl get secret mysqlcluster-sample-root-suffix -o 'jsonpath={.data.password}' | base64 -d)
mysql -hlocalhost -uroot -P30306 -p${PASSWORD}
```

## Modify instance specifications (cpu, memory)

Modify database instance specifications globally (cpu, memory)

```yaml
spec: 
  databaseResource:
    resources:
      limits:
        cpu: "2"
        memory: 1Gi
      requests:
        cpu: 500m
        memory: 1Gi
```

Once you modified the MySQL cluster manifest, you can apply this modify to the apiserver

```bash
kubectl apply -f manifests/minimal-mysql-manifest.yaml
```

## Modify instance storage

Modify database instance storage globally

```yaml
spec: 
  databaseResource:
    storage:
      size: 31474836480
```

Once you modified the MySQL cluster manifest, you can apply this modify to the apiserver

```bash
kubectl apply -f manifests/minimal-mysql-manifest.yaml
```

## Custom instance specifications (cpu, memory)

If you want to specify the specification of a database instance in the cluster
you can use the `customResources` field to achieve.

Execute the following execution to view the database instance list of the `mysqlcluster-sample` cluster

```bash
kubectl get statefulset -l created-by=grds.cloud,app-name=mysqlcluster-sample
```

For Example: Mofify database instance `mysqlcluster-sample-slave0`'s `cpu=3` and `memory=2Gi`

```yaml
spec: 
  customResources:
    mysqlcluster-sample-slave0:
      resources:
        limits:
          cpu: 3
          memory: 2Gi
```

Once you modified the MySQL cluster manifest, you can apply this modify to the apiserver

```bash
kubectl apply -f manifests/minimal-mysql-manifest.yaml
```

## Custom instance storage volume capacity

For Example: Mofify database instance `mysqlcluster-sample-slave0`'s `storage.size=31474836480`(byte)

```yaml
spec: 
  customResources:
   mysqlcluster-sample-slave0:
     storage:
       size: 31474836480
```

Once you modified the MySQL cluster manifest, you can apply this modify to the apiserver

```bash
kubectl apply -f manifests/minimal-mysql-manifest.yaml
```

## Recreate the database instance

Recreate will recreate the databse instance and the volume used by the databse instance

Execute the following execution to view the database instance list of the `mysqlcluster-sample` cluster

```bash
kubectl get pods -l created-by=grds.cloud,app-name=mysqlcluster-sample
```

For Example: Recreate mysqlcluster-sample-slave0-0

```yaml
metadata:
  annotations:
    grds.cloud/recreate-pod: mysqlcluster-sample-slave0-0
```

Once you modified the MySQL cluster manifest, you can apply this modify to the apiserver

```bash
kubectl apply -f manifests/minimal-mysql-manifest.yaml
```

## Restart the database instance

