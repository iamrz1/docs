## KubeDB

1. Install Helm:
```bash
$ sudo snap install helm --classic
$ helm init
```
2. Add kubeDB repo:
```bash
$ helm repo add appscode https://charts.appscode.com/stable/
$ helm repo update
```
3. Install kubedb operator chart
```bash
$ helm install appscode/kubedb --name kubedb-operator --version 0.9.0 \
  --namespace kube-system
```
4. Install KubeDB catalog of database versions
```bash
$ helm install appscode/kubedb-catalog --name kubedb-catalog --version 0.9.0 \
  --namespace kube-system
```
(3 & 4 => Every time minikube starts)
##
#### Create a PG Admin (deployment+service):
```bash
$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/postgres-admin.yaml
```
Watch your pods in demo namespace:
```bash
$ kubectl get pods -n demo --watch
```
Get the url <minikube ip>:<nodeport> to your PGAdmin dashboard (U/P:admin).
```bash
$ minikube service pgadmin -n demo --url
```
## Create a PostgreSQL database Using CRD
#### Create a PG server (custom resource) for storage using Postgres CRD

````bash
$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/postgres-db.yaml
````

DB Server IP is ``$ kubectl get pods quick-postgres-0 -n demo -o yaml | grep podIP``

Deocde secret username(postgres) and password from secrets. Add a server using these credentials.

## Snapshot

A Snapshot is a Kubernetes Custom Resource Definitions (CRD).
It provides declarative configuration for database snapshots in a Kubernetes native way.
In Snapshot valid values for .metadata.labels are Postgres or Elasticsearch.

### Instant backups

Snapshots can be directly used to take instant backups.

To create a local or remote instant backups, use local (filepath or pvc) or remote (gcs or aws) credentials. 

For local instant backups, use one of hostpath or pvc:

````bash
$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/local_snapshot_hostpath.yaml
````

or  

````bash
$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/PVC_for_local_snapshot.yaml

$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/local_snapshot_PVC.yaml
````



For GCS instant backups:

````bash
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv <downloaded-json-key.json> GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret -n demo generic gcs-secret \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/remote_snapshot_GCS.yaml
````

In GCS, bucket is the name of storage space (kubedb-dev).

### Scheduled Backup

Scheduled Backups can be added to the Postgres Database server (object of postgrace custom resource)  by providing credentials and backup interval period.

````bash
$ kubectl apply -f https://raw.githubusercontent.com/iamrz1/docs/master/files/kubedb_yamls/pg_with_scheduled_GCS_backup.yaml
````

### Wall-G (Continuous archiving)




