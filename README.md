# Assessment Helm Chart

This Helm chart deploys the assessment frontend, backend, and PostgreSQL HA database.

---

## **Architectural Choices**

### **Database Choice**
- **PostgreSQL HA (Bitnami)**:
  - Provides automatic failover with Patroni, connection pooling via Pgpool-II, and synchronous replication.
  - Chosen for its battle-tested HA implementation and Kubernetes integration.

### **Ingress Controller**
- **NGINX Ingress**:
  - Selected for its wide adoption, flexible configuration (supports IP whitelisting via annotations), and excellent performance characteristics.

### **Secrets Management**
- **Kubernetes Secrets**:
  - Used for storing database credentials with Helm value encryption.
  - Passwords can be auto-generated during installation, but external secret management (e.g., HashiCorp Vault) is recommended for production.

### **IP Whitelisting**
- Implemented via NGINX `configuration-snippet` annotations.
- Uses CIDR ranges for network-based access control.
- Separate rules for `/admin` and `/api/internal` paths.

---

## **Prerequisites**

### **Create Secret for Database**
```bash
kubectl create secret generic postgresql-ha-passwords \
  --from-literal=postgresql-password=<supersecretpostgrespass123> \
  --from-literal=replication-password=<supersecretpostgrespass123> \
  --from-literal=repmgr-password=<supersecretpostgrespass123> \
  --from-literal=pgpool-admin-password=<supersecretpostgrespass123> \
  --from-literal=password=<supersecretpostgrespass123> \
  --from-literal=admin-password=<supersecretpostgrespass123>
```
### **Add Bitnami Repo**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

## **Installation**
### Install Helm Chart
```bash
helm install assessment ./assessment --namespace assessment --create-namespace
```

## **Test**
### Check frontend to backend connection
```bash
kubectl exec -it deployment/assessment-frontend -- curl http://assessment-backend:80
```

### Check ingress functionality
```bash
# (only miniube)
minikube tunnel
curl http://frontend.local
```
## **High Availabilty**
To ensure high availability for the application, the following measures were implemented:

1. ReplicaCount:
- The frontend and backend deployments are configured with multiple replicas (replicaCount) to ensure redundancy and fault tolerance.

2. PostgreSQL HA:
- The bitnami/postgresql-ha Helm chart is used to deploy a highly available PostgreSQL cluster with:
    - Replication: Multiple PostgreSQL replicas for data redundancy.
    - Pgpool: A connection pooler for load balancing and failover.

3. Pod Disruption Budget (PDB):
- PDBs are configured to ensure a minimum number of pods are always available during updates or disruptions.

4. Horizontal Pod Autoscaler (HPA):
- HPA is enabled for the frontend and backend to automatically scale the number of pods based on CPU or memory usage.

## **Ingress**
### IP Whitelisting
To restrict access to the ```/admin``` and ```/api/internal``` paths, the Helm chart uses separate Ingress resources with the ```nginx.ingress.kubernetes.io/whitelist-source-range``` annotation. This annotation allows only specified IP ranges to access these paths.

### Why
1. NGINX Ingress Limitations:
The NGINX Ingress controller does not support applying different annotations (e.g., IP whitelisting) to different paths within the same Ingress resource.

2. Modularity and Flexibility:
Separate Ingress resources allow for path-specific configurations, such as IP whitelisting for ```/admin``` and ```/api/internal```.

3. Simplified Maintenance:
Each Ingress resource can be updated or modified independently, making it easier to manage and debug.

## **Additional Thoughts**
### Namespaces
In production or staging environments, namespaces can be used to logically isolate resources, improve security, and simplify management. For example:

#### Separate Namespaces for Environments
- assessment-dev for development.
- assessment-staging for staging.
- assessment-prod for production.

#### Separate Namespaces for Components
- frontend for the frontend application.
- backend for the backend application.
- database for PostgreSQL and Pgpool.

## **Additional Architectural Choices**
1. Helm Helpers (_helpers.tpl)
- The _helpers.tpl file is used to define reusable templates and functions, reducing duplication and improving maintainability.
- Example: Standardized naming conventions for resources.

2. Horizontal Pod Autoscaler (hpa.yaml)
- HPA is configured for the frontend and backend to automatically scale pods based on CPU or memory usage, ensuring optimal performance under varying loads.

