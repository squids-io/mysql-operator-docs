---
title: Install the MySQL operator
shorttitle: Deployment
weight: 200
---

## Prerequisites

- MySQL operator requires Kubernetes v1.14.x or later or k3s.
- For the [Helm-based installation](#deploy-mysql-operator-with-helm) you need Helm v3.21.0 or later

## Deploy the MySQL operator from Kubernetes Manifests

Complete the following steps to deploy the MySQL operator using Kubernetes manifests. Alternatively, you can also [install the operator using Helm](#deploy-mysql-operator-with-helm).

1. Create a controlNamespace called "grds".

    ```bash
    kubectl create ns grds
    ```

2. Create a ServiceAccount and install cluster roles.

    ```bash
    kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/mysql-operator-docs/master/manifests/rbac.yaml
    ```

3. Apply the ClusterResources.

    ```bash
    kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/mysql-operator-docs/master/manifests/mysql.grds.cloud_mysqlclusters.yaml
    ```

4. Deploy the MySQL operator.

    ```bash
   kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/mysql-operator-docs/master/manifests/config.yaml
   kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/mysql-operator-docs/master/manifests/deployment.yaml
    ```

## Deploy MySQL operator with Helm

<p align="center"><img src="./images/helm2.svg" width="150"></p>
<p align="center">

Complete the following steps to deploy the MySQL operator using Helm. 
Alternatively, you can also [install the operator using Kubernetes manifests.](#deploy-the-mysql-operator-from-kubernetes-manifests)

> Note: For the Helm-based installation you need [Helm](https://helm.sh/docs/intro/install/#helm) v3.2.4 or later.

1. Add operator chart repository.
    - Helm v3
    ```bash
    helm repo add grdscloud-stable https://grdscloud.github.io/charts/
    helm repo update
    ```

2. Install the MySQL Operator

    ```bash
    helm upgrade --install --wait --create-namespace --namespace grds mysql-operator grdscloud-stable/mysql-operator
    ```

### Check the MySQL operator deployment

To verify that the installation was successful, complete the following steps.

1. Check the status of the pods. You should see a new mysql-operator pod

    ```bash
    $ kubectl get pods -n grds
    NAME                                        READY   STATUS    RESTARTS   AGE
    mysql-operator-76c44cdc5c-lw4z6             1/1     Running   0          53s
    ```

2. Check the CRDs. You should see the following MySQL cluster CRDs.
mysql-cluster-crd.png

    ```bash
    $ kubectl get crd | grep grds
    NAME                                    CREATED AT
    mysqlclusters.mysql.grds.cloud          2020-10-28T09:53:01Z
    ```

