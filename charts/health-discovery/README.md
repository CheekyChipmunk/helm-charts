# Health Discovery Helm Chart
running
This chart deploys Health Discovery in your Kubernetes cluster.

## Prerequisites

- An Averbis account
- A Kubernetes cluster
- Persistent volume provisioner support in the underlying Kubernetes infrastructure
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)

## Creating secret to pull Images

1. Open [https://registry.averbis.com](https://registry.averbis.com) and **Login via OICD Provider** using your Averbis account credentials. 

2. Click your username at the top right corner of the screen and select **User Profile**.

3. Click the clipboard icon to copy the temporary `CLI secret` associated with your account.

4. Create a kubernetes secret named `averbis-docker-registry` with your username and `CLI secret` as password to pull images the Averbis docker registry.

```bash
kubectl create secret docker-registry averbis-docker-registry \
--docker-server=https://registry.averbis.com \
--docker-username=me@example.com \
--docker-password='My CLI secret'
```

## Adding the Helm Repository
```
helm repo add averbis https://averbis.github.io/helm-charts/
```

## Installing the Chart

The chart can be installed using the helm repository or by checking out the chart sources.

### Installing using Helm Repository
To install the chart with the release name `hd`:
```
helm install hd averbis/health-discovery
```
### Installing using Chart Sources
To install the chart with the release name `hd` using cloned chart sources:
```
helm install hd .
```

### Chart Parameters
The chart can optionally be configured using the following parameters:

| Name        | Description         | Default Value     |
| :----------:|:-------------------:| :----------------:|
| `maxMemory` | Maximum memory      | 24G               |
| `existingDbSecret`  | Use MariaDB credentials from an existing secret. The secret has to contain the keys `databaseUsername` and `databasePassword`. Please refer to the [kubernetes documentation](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/) for more information about how to create secrets | "" |
| `externalDbConnectionUrl` | JDBC connection URL of an external MariaDB 10.x database. Requires the `existingDbSecret` parameter to be set | "" |
| `registryUrl` | Container registry URL      | registry.averbis.com               |
| `database.tolerations` | Node Tolerations | [] |
| `database.affinity` | Node Affinity | {} |
| `database.nodeSelector` | Node Selector | {} |
| `healthDiscovery.tolerations` | Node Tolerations | [] |
| `healthDiscovery.affinity` | Node Affinity | {} |
| `healthDiscovery.nodeSelector` | Node Selector | {} |
| `healthDiscovery.licenseSecret` | Existing secret that contains the license in subPath `license.lic` | "" |
| `solr.tolerations` | Node Tolerations | [] |
| `solr.affinity` | Node Affinity | {} |
| `solr.nodeSelector` | Node Selector | {} |


Specify each parameter using the `--set name=value` argument to `helm install` and `helm upgrade`  to overwrite the chart default values, for example:

```
helm install hd averbis/health-discovery --set maxMemory=24G,existingDbSecret=my-secret,externalDbConnectionUrl=jdbc:mariadb://my-mariadb-host:3306/mydb?useMysqlMetadata=true,
registryUrl=my-registry-base-url
```

NOTE: Once this chart is deployed, it is not possible to change the MariaDB access credentials, such as usernames or passwords, using Helm.

NOTE: If using an alternative registry please make sure the structure of the image repositories is exactly as in the Averbis registry, i.e. currently:
/health-discovery/health-discovery
/mariadb/mariadb
/solr/solr

### Exposing the Application
Create a kubernetes `service` of type `loadBalancer` to access the application from outside the kubernetes cluster. Out of the box this only works
with cloud providers like Google GKE or AWS EKS.

```
kubectl expose deployment health-discovery --type=LoadBalancer --name=hd-load-balancer
```

Determine the load balancer URL:
```
kubectl get service

NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
database                   ClusterIP      10.3.243.52    <none>         3306/TCP         9m30s
gcm                        ClusterIP      10.3.247.190   <none>         8181/TCP         9m30s
hd-load-balancer           LoadBalancer   10.3.249.243   34.91.59.210   8080:30790/TCP   6m2s
health-discovery           ClusterIP      10.3.246.164   <none>         8080/TCP         9m30s
kubernetes                 ClusterIP      10.3.240.1     <none>         443/TCP          21m
solr                       ClusterIP      10.3.253.18    <none>         8983/TCP         9m30s
```

You can access the application using the `EXTERNAL-IP` of the `hd-load-balancer` service at http://EXTERNAL-IP:8080/health-discovery


## Upgrading the Chart
Upgrade the `hd` release to the latest chart version. Data and configuration settings will be migrated.
```
helm repo update
helm upgrade hd averbis/health-discovery
```

## Uninstalling the Chart
Uninstall the `hd` release. This will delete all Kubernetes components associated with this chart. All data and configuration settings will be deleted as well.

```
helm uninstall hd
```
