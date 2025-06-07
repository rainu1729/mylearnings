Okay, let's set up a 3-node CockroachDB cluster on your local machine using Docker and Terraform.

This setup will be **insecure** (no SSL certificates) for simplicity on a local machine. For production, you'd need to configure security.

**Prerequisites:**

1.  **Docker Desktop (or Docker Engine):** Ensure Docker is installed and running.
    *   Install: [Docker Installation](/Container/docker)
2.  **Terraform:** Install the Terraform CLI.
    *   Install: [Terraform Installation](/Container/terraform)
3.  **Install client tools like DBeaver:**
    *   Install: [DBeaver](https://dbeaver.io/)

**Step 1: Project Setup**

1.  Create a new directory for your project, e.g., `cockroach-tf-docker`.
2.  Navigate into this directory: `cd cockroach-tf-docker`

**Step 2: Create Terraform Configuration File (`main.tf`)**

Create a file named `main.tf` in your `cockroach-tf-docker` directory and paste the following content:

```terraform
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0" # Or latest compatible version
    }
  }
}

provider "docker" {}

# Define a Docker network for the cluster nodes to communicate
resource "docker_network" "roachnet" {
  name = "roachnet"
}

# --- Node 1 ---
resource "docker_volume" "roach1_data" {
  name = "roach1_data"
}

resource "docker_container" "roach1" {
  name     = "roach1"
  hostname = "roach1"
  image    = "cockroachdb/cockroach:v23.2.1" # Use a specific version

  networks_advanced {
    name    = docker_network.roachnet.name
    aliases = ["roach1"]
  }

  ports {
    internal = 26257 # SQL port
    external = 26257
  }
  ports {
    internal = 8080  # DB Console HTTP port
    external = 8080
  }

  volumes {
    volume_name    = docker_volume.roach1_data.name
    container_path = "/cockroach/cockroach-data"
  }

  # Command to start the first node
  # --insecure: Use insecure mode (no TLS)
  # --join: List of nodes to join (itself and others)
  # --advertise-addr: Address other nodes use to reach this one (must be resolvable within the docker network)
  # --listen-addr: Address this node listens on within the container
  # --http-addr: Address for the DB Console within the container
  command = [
    "start",
    "--insecure",
    "--join=roach1:26257,roach2:26257,roach3:26257",
    "--advertise-addr=roach1:26257",
    "--listen-addr=0.0.0.0:26257",
    "--http-addr=0.0.0.0:8080",
    "--cache=.25",      # Limit cache size for local dev
    "--max-sql-memory=.25" # Limit SQL memory for local dev
  ]

  restart = "unless-stopped"
}

# --- Node 2 ---
resource "docker_volume" "roach2_data" {
  name = "roach2_data"
}

resource "docker_container" "roach2" {
  name     = "roach2"
  hostname = "roach2"
  image    = docker_container.roach1.image # Use the same image as node 1

  networks_advanced {
    name    = docker_network.roachnet.name
    aliases = ["roach2"]
  }

  ports {
    internal = 26257
    external = 26258 # Expose on a different host port
  }
  ports {
    internal = 8080
    external = 8081  # Expose on a different host port
  }

  volumes {
    volume_name    = docker_volume.roach2_data.name
    container_path = "/cockroach/cockroach-data"
  }

  command = [
    "start",
    "--insecure",
    "--join=roach1:26257,roach2:26257,roach3:26257", # Join the first node
    "--advertise-addr=roach2:26257",
    "--listen-addr=0.0.0.0:26257",
    "--http-addr=0.0.0.0:8080",
    "--cache=.25",
    "--max-sql-memory=.25"
  ]

  # Ensure roach1 is started before roach2 attempts to join
  depends_on = [docker_container.roach1]
  restart    = "unless-stopped"
}

# --- Node 3 ---
resource "docker_volume" "roach3_data" {
  name = "roach3_data"
}

resource "docker_container" "roach3" {
  name     = "roach3"
  hostname = "roach3"
  image    = docker_container.roach1.image # Use the same image as node 1

  networks_advanced {
    name    = docker_network.roachnet.name
    aliases = ["roach3"]
  }

  ports {
    internal = 26257
    external = 26259 # Expose on a different host port
  }
  ports {
    internal = 8080
    external = 8082  # Expose on a different host port
  }

  volumes {
    volume_name    = docker_volume.roach3_data.name
    container_path = "/cockroach/cockroach-data"
  }

  command = [
    "start",
    "--insecure",
    "--join=roach1:26257,roach2:26257,roach3:26257", # Join the first node
    "--advertise-addr=roach3:26257",
    "--listen-addr=0.0.0.0:26257",
    "--http-addr=0.0.0.0:8080",
    "--cache=.25",
    "--max-sql-memory=.25"
  ]

  depends_on = [docker_container.roach1] # Also depends on roach1
  restart    = "unless-stopped"
}

# --- Outputs ---
output "db_console_node1" {
  description = "DB Console URL for Node 1"
  value       = "http://localhost:8080"
}

output "db_console_node2" {
  description = "DB Console URL for Node 2"
  value       = "http://localhost:8081"
}

output "db_console_node3" {
  description = "DB Console URL for Node 3"
  value       = "http://localhost:8082"
}

output "sql_connection_node1" {
  description = "SQL connection string for Node 1 (using CockroachDB client)"
  value       = "cockroach sql --insecure --host=localhost --port=26257"
}

output "sql_connection_node2" {
  description = "SQL connection string for Node 2 (using CockroachDB client)"
  value       = "cockroach sql --insecure --host=localhost --port=26258"
}

output "sql_connection_node3" {
  description = "SQL connection string for Node 3 (using CockroachDB client)"
  value       = "cockroach sql --insecure --host=localhost --port=26259"
}

output "psql_connection_node1" {
  description = "PostgreSQL compatible connection string for Node 1"
  value       = "postgresql://root@localhost:26257/defaultdb?sslmode=disable"
}
```

**Step 3: Initialize Terraform**

Open your terminal in the `cockroach-tf-docker` directory and run:

```bash
terraform init
```

This command downloads the necessary Docker provider plugin.

**Step 4: Plan the Deployment**

Run the following command to see what Terraform will create:

```bash
terraform plan
```

Review the plan to ensure it's going to create three Docker containers, one network, and three volumes.

**Step 5: Apply the Configuration (Spin up the cluster)**

Execute the command to create the resources:

```bash
terraform apply
```

Terraform will ask for confirmation. Type `yes` and press Enter.
This will:
1.  Create the `roachnet` Docker network.
2.  Create the `roachX_data` Docker volumes.
3.  Pull the `cockroachdb/cockroach` image if you don't have it.
4.  Start the three `roachX` containers.

Wait for the command to complete. It might take a minute or two for the nodes to start and find each other.

**Step 6: Initialize the CockroachDB Cluster**

Once the containers are running, you need to perform a one-time initialization of the cluster. This tells one of the nodes to bootstrap the cluster.

You can do this by executing the `cockroach init` command inside one of the containers. We'll use `roach1`:

```bash
docker exec -it roach1 ./cockroach init --insecure
```

You should see a success message like:
`Cluster successfully initialized`

**Step 7: Verify the Cluster**

1.  **DB Console:**
    *   Open your web browser and go to `http://localhost:8080` (for roach1).
    *   You should see the CockroachDB Console.
    *   Click on "View nodes list" on the metrics overview page or navigate to "Cluster Overview" -> "Nodes". You should see all 3 nodes listed as healthy.
    *   You can also access the console for other nodes: `http://localhost:8081` (roach2) and `http://localhost:8082` (roach3).

2.  **SQL Client (using the CockroachDB binary):**
    If you have the `cockroach` binary installed locally:
    ```bash
    cockroach sql --insecure --host=localhost --port=26257
    ```
    Or connect to another node:
    ```bash
    cockroach sql --insecure --host=localhost --port=26258
    ```
    Once connected, try some commands:
    ```sql
    SHOW DATABASES;
    CREATE DATABASE myapp;
    USE myapp;
    CREATE TABLE users (id INT PRIMARY KEY, name STRING);
    INSERT INTO users VALUES (1, 'Alice'), (2, 'Bob');
    SELECT * FROM users;
    SHOW RANGES FROM TABLE users; -- See data distribution
    \q
    ```
**Step 8: Manage and Clean Up**

*   **View running containers:** `docker ps`
*   **View container logs:** `docker logs roach1` (or `roach2`, `roach3`)
*   **Stop and remove the cluster (Terraform):**
    When you're done, you can destroy all resources managed by Terraform:
    ```bash
    terraform destroy
    ```
    Type `yes` to confirm. This will stop and remove the containers, network, and volumes.

**Important Considerations:**

*   **Insecure Mode:** This setup uses `--insecure`. **Never use this for production.** Production deployments require proper TLS certificates for secure communication between nodes and clients.
*   **Data Persistence:** The `docker_volume` resources ensure that your data persists even if the containers are stopped and restarted (as long as the volumes are not deleted). If you run `terraform destroy`, the volumes will also be removed by default.
*   **Resource Limits:** The `--cache` and `--max-sql-memory` flags are set low for local development. For more serious use, you'd adjust these or remove them to let CockroachDB use more resources.
*   **CockroachDB Version:** Pinning the `cockroachdb/cockroach` image version (e.g., `:v23.2.1`) is good practice for reproducibility.
*   **`depends_on`:** While `depends_on` helps guide Terraform's creation order, CockroachDB nodes are designed to find each other and form a cluster even if they start in a slightly different order, as long as the `--join` flags are correct and network connectivity exists.

You now have a 3-node CockroachDB cluster running locally, managed by Terraform!