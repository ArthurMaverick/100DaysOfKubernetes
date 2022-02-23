# Provider Infrastucture 

## Composite resource (XRs) are always cluster scoped - they exit outside of any namespace. This allow an XR to represent infrastucture that mighr to be consumed from several different namespaces. This is often true for VPC - AN infrastructure operato may wish to define a VPC network XR and a SQL instance XR, only the latter of which may be managed by application operators. The application operators are restrited to their team's namespace, but theis SQL instances shold all be attached to VPC network that the infrastucture operator manages. Crossplane enavles scenarios like the by allowing the infrastructure operator th offer their application  operators a composite resource clain (XRC). A (XRC) is a namespaced proxy for an XR; the schema of an XRC is indentical to that  of its corresponding XR. When an application operator creates an XRC, a corresponding backing is create automatically. This model has similarities to Percistent Volumes (PV) and Persistent Volume Claims (PVC) in kubernetes.

## Clain your Infrastucture
**The configuration package we installed in the last section:**

- Defines a XpostgreSQLInstance
- Offers a PostgreSQLInstance clain (XRC) for said XR.
- Create a Composition that can satisfy our XR.

This means that we can create a PostgreSQLInstance XRC in the default namespace to provision a PostgreSQL instance and all the suporting infrastucture (VPCs, firewall rules, resources groups, etc) that it may need!


_Note that this resource will create an RDS instance using your default VPC, which may or may not allow connections from the internet depending on how it is configured._
```
apiVersion: database.example.org/v1alpha
kind: PostgreSQLInstance
metadata: 
  name: my-db
  namespace: default
spec:
  parameters:
    storageGB: 20
  compositionSelector:
    matchLabels:
      provider: aws
      vpc: default
  writeConnectionSecretRef:
    name: db-conn
```
```
kubectl apply -f https://raw.githubusercontent.com/crossplane/crossplane/release-1.6/docs/snippets/compose/claim-aws.yaml
```

**After creating the PostgreSQLInstance Crossplane will bein provisining a database instance on your provide of choice. Once provisioning is complete you should see READY=true in the output when you run:**

```
kubectl get postgresqlinstance my-db

```
_Note: while waiting for the PostgreSQLInstance to become ready, you may want to look at other resources in your cluster. The following commands will allow you to view groups of Crossplane resources:_

- kubectl get claim: get all resources of all claim kinds, like PostgreSQLInstance.
- kubectl get composite: get all resources that are of composite kind, like XPostgreSQLInstance.
- kubectl get managed: get all resources that represent a unit of external infrastructure.
- kubectl get <name-of-provider>: get all resources related to <provider>.
- kubectl get crossplane: get all resources related to Crossplane.

try the following command th watch your provisioning resource become ready:
```
kubectl get crossplane -l crossplane.io/claim-name=my-db
```
Once your PostgreSQLInstance is ready, you should see a Secret in the default namespace named db-conn that contains keys that we defined in XRD. If they were filled by the composition, then they should appear:

```
kubectl describe secrets db-conn
Name:         db-conn
Namespace:    default
...

Type:  connection.crossplane.io/v1alpha1

Data
====
password:  27 bytes
port:      4 bytes
username:  25 bytes
endpoint:  45 bytes
```

## Consume Your Infrastructure
Because connection secrets are written as a Kubernetes Secret they can easily be consumed by Kubernetes primitives. The most basic building block in Kubernetes is the Pod. Let’s define a Pod that will show that we are able to connect to our newly provisioned database.

_Note that if you’re using a hosted Crossplane you’ll need to copy the db-conn connection secret over to your own Kubernetes cluster and run this pod there._

```
piVersion: v1
kind: Pod
metadata:
  name: see-db
  namespace: default
spec:
  containers:
  - name: see-db
    image: postgres:12
    command: ['psql']
    args: ['-c', 'SELECT current_database();']
    env:
    - name: PGDATABASE
      value: postgres
    - name: PGHOST
      valueFrom:
        secretKeyRef:
          name: db-conn
          key: endpoint
    - name: PGUSER
      valueFrom:
        secretKeyRef:
          name: db-conn
          key: username
    - name: PGPASSWORD
      valueFrom:
        secretKeyRef:
          name: db-conn
          key: password
    - name: PGPORT
      valueFrom:
        secretKeyRef:
          name: db-conn
          key: port
```
```
kubectl apply -f https://raw.githubusercontent.com/crossplane/crossplane/release-1.6/docs/snippets/compose/pod.yaml
```

This Pod simply connects to a PostgreSQL database and prints its name, so you should see the following output (or similar) after creating it if you run kubectl logs see-db:

```
kubectl logs see-db

current_database
------------------
 postgres
(1 row)

```