
---

# 🧠 What You'll Achieve:

You will deploy a production-ready **MySQL replication setup** with:

* One **primary (master)** node — for reads and writes
* Multiple **replica (slave)** nodes — for read-only access
* **Persistent volumes** to store data
* **Replication enabled** by default

---

# ✅ Prerequisites

Make sure you have:

* Access to an OpenShift or Kubernetes cluster
* `kubectl` or `oc` installed and configured
* [Helm](https://helm.sh/docs/intro/install/) installed (`helm version` should work)

---

# 🚀 Step-by-Step Instructions

---

## 🔹 Step 1: Add the Bitnami Helm chart repository

Bitnami maintains well-tested Helm charts for many apps.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

This adds Bitnami’s chart repo to your local Helm client.

---

## 🔹 Step 2: Create a new namespace for MySQL

This helps isolate your MySQL setup.

```bash
kubectl create namespace mysql-replication
```

Or if you're using `oc`:

```bash
oc new-project mysql-replication
```

---
## 🔹 Create a Kubernetes Secret in your cluster:

```bash
kubectl create secret generic mysql-secret \
  --namespace mysql-replication \
  --from-literal=mysql-root-password=MyRootPassword123 \
  --from-literal=mysql-password=MyAppUserPass123 \
  --from-literal=mysql-replication-password=MyReplPassword123

```


## 🔹 Step 3: Create a custom configuration file

Configure your values-mysql.yaml to reference this existing secret:
This file enables replication and sets user credentials.
Run this command to create the file:

```bash
nano values-mysql.yaml
```

Paste the following content:

```yaml
architecture: replication

# you can optionally set a custom DB user:
auth:
  existingSecret: "mysql-secret"
  replicationUser: "repl"
  replicationPassword: ""     # ignored as well
  rootPassword: ""            # ignored when `existingSecret` is set
  username: "appuser"
  password: ""                # will be read from secret


primary:
  persistence:
    enabled: true
    size: 5Gi

secondary:
  replicaCount: 2
  persistence:
    enabled: true
    size: 5Gi
```

> 📌 This means:
>
> * Root user password: `MyRootPassword123`
> * Replication user: `repl` / `MyReplPassword123`
> * One master and two replicas
> * Each with 5Gi of persistent storage

Press `Ctrl + O`, then `Enter` to save. Press `Ctrl + X` to exit.

---

## 🔹 Step 4: Install the MySQL replication chart

Now deploy MySQL with Helm:

```bash
helm install mysql-cluster bitnami/mysql \
  -n mysql-replication \
  -f values-mysql.yaml
```
```bash
#################################OUTPUT:
cloudtrain@openshift:~$ helm install mysql-cluster bitnami/mysql \
  -n mysql-replication \
  -f sts/values-mysql.yaml
NAME: mysql-cluster
LAST DEPLOYED: Mon Jul 28 20:10:01 2025
NAMESPACE: mysql-replication
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 14.0.0
APP VERSION: 9.4.0

NOTICE: Starting August 28th, 2025, only a limited subset of images/charts will remain available for free. Backup will be available for some time at the 'Bitnami Legacy' repository. More info at https://github.com/bitnami/containers/issues/83267

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace mysql-replication

Services:

  echo Primary: mysql-cluster-primary.mysql-replication.svc.cluster.local:3306
  echo Secondary: mysql-cluster-secondary.mysql-replication.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql-replication mysql-cluster -o jsonpath="{.data.mysql-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mysql-cluster-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:9.4.0-debian-12-r0 --namespace mysql-replication --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mysql-cluster-primary.mysql-replication.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

  3. To connect to secondary service (read-only):

      mysql -h mysql-cluster-secondary.mysql-replication.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"






WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - primary.resources
  - secondary.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
#################################################
```



---

## 🔹 Step 5: Check that it's running

Run this to see your pods and services:

```bash
kubectl get pods -n mysql-replication
kubectl get svc -n mysql-replication
```

You should see pods like:

```
mysql-cluster-primary-0
mysql-cluster-secondary-0
mysql-cluster-secondary-1
```

And services like:

```
mysql-cluster-primary            # connect to master
mysql-cluster-secondary          # connect to replicas
```

---

## 🔹 Step 6: Access MySQL and test replication

### Start a temporary MySQL client pod:

```bash
kubectl run mysql-client -it --rm --image=bitnami/mysql:5.7 \
  --namespace mysql-replication --env MYSQL_ROOT_PASSWORD=MyRootPassword123 \
  -- bash
```

Inside the pod, connect to **master**:

```bash
mysql -h mysql-cluster-primary.mysql-replication.svc.cluster.local -uroot -p
```

Enter the password: `MyRootPassword123`

Create a test database and table:

```sql
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Alice');
```

Now connect to **a replica**:

```bash
mysql -h mysql-cluster-secondary.mysql-replication.svc.cluster.local -uroot -p
```
Then enter the root password:
```bash
MyRootPassword123
```
Check the data:

```sql
USE testdb;
SELECT * FROM users;
```

You should see the row `1, Alice`, confirming replication is working! 🎉

---

# 📦 Summary

| Step                | Description                        |
| ------------------- | ---------------------------------- |
| Helm repo added     | Get the Bitnami MySQL chart        |
| Namespace created   | Isolate the setup                  |
| values.yaml used    | Configured replication and storage |
| Helm install run    | Deployed MySQL StatefulSet         |
| Accessed MySQL pods | Validated master-slave replication |

---

Cleaning up your MySQL replication setup deployed via the Bitnami Helm chart involves removing all the Kubernetes resources and persistent storage created during the installation.

Here’s how to do a **safe and complete cleanup** step-by-step:

---

## 1. Delete the Helm release

Run this command to uninstall the release and delete all Kubernetes resources (Deployments, StatefulSets, Services, ConfigMaps, Secrets, etc.) created by Helm:

```bash
helm uninstall mysql-cluster -n mysql-replication
```

---

## 2. Delete PersistentVolumeClaims (PVCs)

Helm usually does **not** delete PVCs automatically to prevent data loss. You should delete them manually if you want to fully clean up:

List PVCs in your namespace:

```bash
kubectl get pvc -n mysql-replication
```

Delete all PVCs in that namespace:

```bash
kubectl delete pvc --all -n mysql-replication
```

---

## 3. (Optional) Delete PersistentVolumes (PVs)

If your storage class dynamically provisions volumes, PVC deletion should clean up PVs automatically. But to check and delete any leftover PVs:

List PVs and find ones bound to your MySQL PVCs:

```bash
kubectl get pv
```

Delete any PVs related to your MySQL cluster (careful!):

```bash
kubectl delete pv <pv-name>
```

---

## 4. Delete the namespace (optional)

If you want to clean the entire namespace:

```bash
oc delete project mysql-replication
```

---

## Summary of cleanup commands

```bash
helm uninstall mysql-cluster -n mysql-replication
kubectl delete pvc --all -n mysql-replication
kubectl delete namespace mysql-replication  # optional
```

---

**⚠️ Warning:**
Deleting PVCs and namespaces **permanently deletes all stored data**. Make sure you have backups if needed.

---

🔹 Explore more about chart:  Download the MySQL chart (without installing)
```bash
helm pull bitnami/mysql --untar
cd mysql/
ls
```
