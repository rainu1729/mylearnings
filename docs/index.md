# Welcome to My Learning Journey

Welcome to my personal documentation hub where I compile notes, guides, and tutorials on software installations, architecture concepts, scripting languages, and utility tools.

---

## 📂 Navigation Structure

This documentation site is organized to separate **Software Installation & Setup** from **Use Cases & References** within each technology category. This makes it easy to either spin up a new service or learn how to run queries, commands, and tasks:

*   🗄️ **Databases**
    *   *Setup*: Oracle Free, CockroachDB (Terraform+Docker), PostgreSQL (Docker), IBM DB2 (Docker).
    *   *Operations*: Google BigQuery basics, Data Lake architecture.
*   ☁️ **Cloud**
    *   *Setup*: Google Cloud SDK (gcloud CLI).
*   🐳 **Containers & IAC**
    *   *Setup*: Docker engine, Terraform CLI, Kubernetes local clusters (Minikube / Kind).
*   🐚 **Scripting**
    *   *Operations*: Unix Shell syntax, scripting essentials, file and system commands.
*   ⏱️ **Schedulers**
    *   *Setup*: Apache Airflow local setup via Docker Compose.

---

## 🆕 Recently Added Pages

Here are the most recently added or restructured guides:

1.  **[Kubernetes Local Cluster Setup](Containerization/kubernetes.md)** - Learn how to set up `kubectl`, `minikube`, and `kind` locally.
2.  **[GCP Cloud SDK CLI Setup](Cloud/gcp.md)** - Guide to install, authenticate, and configure the Google Cloud SDK.
3.  **[IBM DB2 Local Docker Setup](Database/ibmdb2.md)** - Spin up an IBM DB2 database container in under two minutes.
4.  **[Data Lake Concepts & Design](Database/datalake.md)** - Core architectural patterns, layers, and comparisons to Data Warehouses.
5.  **[Oracle Database Free local Setup](Database/oracle.md)** - Run Oracle Database Free container using Podman.

---

## 🚀 Spin Up Locally

To serve these docs locally on your machine, run:

```bash
# Using uv (Recommended)
uv run mkdocs serve

# Or using activated virtual environment
mkdocs serve
```
Then visit [http://127.0.0.1:8000](http://127.0.0.1:8000).
