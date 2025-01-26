# Database Choice
PostgreSQL HA (Bitnami): Provides automatic failover with Patroni, connection pooling via Pgpool-II, and synchronous replication. Chosen for its battle-tested HA implementation and Kubernetes integration.

# Ingress Controller
NGINX Ingress: Selected for its wide adoption, flexible configuration (supports IP whitelisting via annotations), and excellent performance characteristics.

# Secrets Management
Kubernetes Secrets: Used for storing database credentials with Helm value encryption

Auto-generation: Passwords can be auto-generated during install but recommend using external secret management (Vault) in production

# High Availability
Component   HA      Strategy
Frontend	3       replicas + podAntiAffinity
Backend	    3       replicas + podAntiAffinity
PostgreSQL	3-node  cluster + synchronous replication
Pgpool	    2       replicas for connection pooling

# IP Whitelisting
- Implemented via NGINX configuration-snippet annotations
- Uses CIDR ranges for network-based access control
- Separate rules for admin vs internal API paths


# Add Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Set up passwords first
In Live Setup with sealed secrets or Vault

# Create secret for database
kubectl create secret generic postgresql-ha-passwords \
  --from-literal=postgresql-password=<supersecretpostgrespass123> \
  --from-literal=replication-password=<supersecretpostgrespass123> \
  --from-literal=repmgr-password=<supersecretpostgrespass123> \
  --from-literal=pgpool-admin-password=<supersecretpostgrespass123> \
  --from-literal=password=<supersecretpostgrespass123> \
  --from-literal=admin-password=<supersecretpostgrespass123>

# Install with auto-generated passwords
helm install assessment . 

# Notes
## Check frontend connection to backend
kubectl exec -it deployment/assessment-frontend -- curl http://assessment-backend:80


# Database
## Connect to database
kubectl run psql-client --rm -it --image=bitnami/postgresql --restart=Never -- bash
