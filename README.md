Trino + Hive Metastore + PostgreSQL + Apache Spark + Prometheus + Grafana on Kubernetes
This guide explains how to set up a data stack with Trino, Hive Metastore, PostgreSQL, Apache Spark, Prometheus, and Grafana inside Kubernetes. It’s designed for local clusters (Docker Desktop, Minikube, kind) or cloud environments.

📦 Components
PostgreSQL → Stores Hive Metastore metadata

Hive Metastore → Provides metadata services for Hive-compatible engines

Trino → Distributed SQL query engine that connects to Hive Metastore

Apache Spark → Distributed compute engine for batch, streaming, and machine learning workloads

Prometheus → Metrics collection and scraping for Trino, Spark, and cluster components

Grafana → Dashboard visualization for Prometheus metrics

🚀 Prerequisites
A running Kubernetes cluster (local or cloud)

kubectl configured to access the cluster

Optional: helm for package management

⚙️ Setup Workflow
1. Deploy PostgreSQL
Run PostgreSQL as a StatefulSet with persistent storage.

Configure database name, user, and password.

Expose it via a Service so Hive Metastore can connect.

2. Deploy Hive Metastore
Deploy Hive Metastore as a Deployment.

Configure it to use PostgreSQL (host, database, user, password).

Expose it via a Service on port 9083.

3. Initialize Hive Schema
Run Hive’s schematool once to initialize the schema in PostgreSQL.

After initialization, set SKIP_SCHEMA_INIT=true in Hive Metastore to avoid re-running schema setup.

4. Deploy Trino
Deploy Trino as a Deployment.

Mount configuration files via a ConfigMap.

Expose it via a Service on port 8080.

5. Configure Trino Catalog
Add a hive.properties file in a ConfigMap.

Point it to Hive Metastore using:

Code
connector.name=hive
hive.metastore.uri=thrift://hive-metastore:9083

6. Deploy Apache Spark
Run Spark in cluster mode with a master and worker pods.

Configure Spark to use Hive Metastore for metadata.

Ensure Spark pods have access to the Hive Metastore service (9083).

Optionally, integrate Spark with Trino by pointing Spark SQL queries to the same Hive catalog.

7. Deploy Prometheus
Deploy Prometheus to scrape metrics from Trino, Spark, and Kubernetes.

Use a Service to expose the Prometheus endpoint.

Enable Trino metrics collection in the Trino config if needed.

8. Deploy Grafana
Deploy Grafana to visualize Prometheus metrics.

Connect Grafana to Prometheus as a data source.

Import or create dashboards for Trino and cluster metrics.

✅ Verification
Check pods:

bash
kubectl get pods
Port-forward Trino:

bash
kubectl port-forward svc/trino 8080:8080
Access UI at: http://localhost:8080

Port-forward Prometheus:

bash
kubectl port-forward svc/prometheus 9090:9090
Access UI at: http://localhost:9090

Port-forward Grafana:

bash
kubectl port-forward svc/grafana 3000:3000
Access UI at: http://localhost:3000

Submit a Spark job:

bash
kubectl exec -it <spark-pod> -- spark-sql --master local --conf hive.metastore.uris=thrift://hive-metastore:9083

Run queries in both Trino and Spark against the same Hive tables.

📖 Notes
PostgreSQL replaces Derby for production readiness.

ConfigMaps are recommended for Trino and Spark configs.

Persistent Volumes ensure PostgreSQL data durability.

Prometheus collects metrics and Grafana visualizes them for easier monitoring.

Spark can act as an execution engine for Hive queries if configured (hive.execution.engine=spark).

🛠️ Next Steps
Add HiveServer2 if needed for legacy Hive clients.

Scale Trino workers and Spark executors for larger workloads.

Integrate with external storage (S3, HDFS, MinIO).

Add Grafana dashboards for Trino query performance and Spark job metrics.

Use Spark for ETL pipelines and Trino for interactive queries.