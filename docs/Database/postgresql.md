**PostgreSQL Learning Guide for Beginners**

Welcome to the world of PostgreSQL! Let's start by setting up a local Docker container running and pgAdmin installation.

**Setup:**

*   PostgreSQL database running in a Docker container (e.g., using the official `postgres` image).
*   pgAdmin installed and connected to your Docker PostgreSQL instance.
*   postgresql-client installation to use psql.

**Goal of this Document:**

To provide you with the fundamental SQL commands and concepts to:

*  Start PostgreSQL in docker container.
*  Understand basic database structure (databases, schemas, tables).
*  Create, modify, and delete tables.
*  Perform basic data manipulation (Insert, Select, Update, Delete).
*  Introduce functions, views, and triggers.

### 1. PostgreSQL Database Setup

We will use Terraform to spin up the database instance create the Terraform configuration file  main.tf and add the below content. The will be used to setup lastest version of PostgreSQL with a default user 'myuser' and creds.
 
```json
# Configure the Docker provider
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.4.0" # Use a recent version
    }
  }
}

provider "docker" {}

# --- Variables ---
# Define variables for configuration to make it reusable

variable "db_name" {
  description = "The name of the PostgreSQL database."
  type        = string
  default     = "MyPostgresDB"
}

variable "db_user" {
  description = "The PostgreSQL user."
  type        = string
  default     = "myuser"
}

variable "db_password" {
  description = "The PostgreSQL password."
  type        = string
  default     = "mypassword"
  sensitive   = true
}

variable "db_port" {
  description = "The host port to expose PostgreSQL on."
  type        = number
  default     = 5432
}

variable "pg_version" {
  description = "The version of the PostgreSQL Docker image to use."
  type        = string
  default     = "latest" # Or specify a version like "16"
}

# --- Docker Resources ---

# 1. Pull the PostgreSQL Docker image
resource "docker_image" "postgres" {
  name         = "postgres:${var.pg_version}"
  keep_locally = true # Keep the image after apply
}

# 2. Create a persistent volume for database data
resource "docker_volume" "pg_data" {
  name = "pg_data_volume" # You can name this whatever you like
}

# 3. Create a custom bridge network for the container (optional but good practice)
resource "docker_network" "pg_network" {
  name   = "pg_app_network" # Name your network
  driver = "bridge"
}

# 4. Create and run the PostgreSQL container
resource "docker_container" "postgres_server" {
  name = "postgres_container" # Give your container a meaningful name

  # Use the image resource name; Terraform will pull the image first due to implicit dependency
  image = docker_image.postgres.name

  # Map the container's internal port (5432) to a host port
  ports {
    internal = 5432
    external = var.db_port
  }

  # Set necessary environment variables for the PostgreSQL image
  # See the official PostgreSQL Docker image documentation for more options
  env = [
    "POSTGRES_DB=${var.db_name}",
    "POSTGRES_USER=${var.db_user}",
    "POSTGRES_PASSWORD=${var.db_password}"
    # "PGDATA=/var/lib/postgresql/data/pgdata" # Optional: Change data directory path if needed
  ]

  # Mount the persistent volume to the data directory inside the container
  volumes {
    volume_name    = docker_volume.pg_data.name # Use the volume resource name
    container_path = "/var/lib/postgresql/data" # This is the default data path for the official image
  }

  # Attach the container to the custom network
  networks_advanced {
    name = docker_network.pg_network.name # Use the network resource name
  }

  # Ensure the container restarts automatically if it stops
  restart = "always"
}

# --- Outputs ---
# Provide connection details after apply

output "db_host" {
  description = "The hostname/IP to connect to the database."
  # If running on the same machine as Docker, 'localhost' or '127.0.0.1' is usually correct.
  # If running remotely, this would be the IP of the machine running Docker.
  value = "localhost"
}

output "db_port" {
  description = "The port to connect to the database on the host."
  value       = docker_container.postgres_server.ports[0].external
}

output "db_name" {
  description = "The name of the database."
  value       = var.db_name
}

output "db_user" {
  description = "The database user."
  value       = var.db_user
}

output "connection_string" {
  description = "The connection string to connect to the PostgreSQL database."
  value       = "postgresql://${var.db_user}:${var.db_password}@${output.db_host}:${output.db_port}/${var.db_name}"
}
```

* Terraform initialize Initializing a configuration directory downloads and installs the providers defined in the configuration

```
terraform init
```

* Format your configuration for readability , consistency and configuration is syntactically valid and internally consistent

```
terraform validate
```

* Create the infra 

```
terraform apply
```

* validate the docker container is up as running 
```
docker ps
```

#### Connecting to Your PostgreSQL Database

You have two primary ways to connect and interact:

*   **pgAdmin (GUI - Graphical User Interface):** Great for visual browsing, data viewing, and executing queries easily.
*   **psql (Command Line Interface):** A powerful and versatile tool for executing commands, scripting, and administration. Essential for learning and understanding the raw interaction.

###### Connecting via pgAdmin

pgAdmin installation 


***Install the public key for the repository (if not done previously)***
```bash
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg
```

***Create the repository configuration file***
```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
```

***Install for both desktop and web modes:***
```bash
sudo apt install pgadmin4
```

***Once installation is completed follow below steps to connect to the database***

1.  Open pgAdmin.
2.  Expand "Servers" in the browser tree on the left.
3.  Expand your connection (the one you configured to connect to your Docker container). You might be prompted for the password.
4.  Expand "Databases". We will see `postgres` (the default database) and .`MyPostgresDB`
5.  Click on the database you want to work with (start with `postgres`).
6.  Go to `Tools -> Query Tool`. This opens a new window or tab where you can type and execute SQL commands.

###### Connecting via psql (Command Line)

This is super useful! we can execute `psql` from the terminal.We can run the `psql` prompt by connecting to the running PostgreSQL *Docker* container or install `psql` in the host machine.

***Install psql in hostmachine***

***adding the official PostgreSQL GPG signing key to your system's apt keyring***
```bash
sudo sh -c 'apt update && apt install wget ca-certificates'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
***Add entry in APT package manager configuration***
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
***Install lastest version of postgresql-client***

```bash
sudo apt update
sudo apt install postgresql-client-17
```
***Connect to the Postgresql DB via psql***

```bash
psql -h localhost -p 5432 -U myuser -d postgres
```

**Find your running container ID or name:**
```bash
docker ps
```
Look for the container running the `postgres` image. Note its ID or name (e.g., `my-postgres-container`).

**Execute psql inside the container:**
```bash
docker exec -it <container_id_or_name> psql -U <your_postgres_user> <your_database_name>
```
*   `<container_id_or_name>`: Replace with the actual ID or name.
*   `<your_postgres_user>`: The user you configured (often `postgres` by default).
*   `<your_database_name>`: The database you want to connect to (often `postgres` by default).

Example:
```bash
docker exec -it my-postgres-container psql -U postgres postgres
```

**Useful psql Commands (Meta-Commands - start with `\`):**

*   `\l`: List all databases.
*   `\c <database_name>`: Connect to a different database.
*   `\dt`: List tables in the current schema (usually `public`).
*   `\d <table_name>`: Describe a table (shows columns, types, constraints). Very useful!
*   `\dn`: List schemas.
*   `\df`: List functions.
*   `\dv`: List views.
*   `\dx`: List extensions like plpgsql.
*   `\q`: Quit psql.


You can execute standard SQL commands directly at the `postgres=#` prompt.


**Execute a .sql file from command prompt:**
```bash
createdb -h localhost -U myuser -p 5432 newdbname

# run a file from psql 
psql  -h localhost -U myuser -p 5432 newdbname < verify.sql
```
---

### 2. Basic Concepts: Databases, Schemas, and Tables

*   **Database:** A container for one or more schemas. Think of it as a top-level container for a specific application or set of related data.
*   **Schema:** A namespace within a database. It holds tables, functions, operators, etc. By default, you have a `public` schema. Schemas help organize objects and prevent naming conflicts (e.g., `users` table in `app_schema` and `users` table in `audit_schema`).
*   **Table:** A collection of related data organized in rows and columns. This is where your actual information is stored.

**Working with Databases:**

*   **List Databases (psql):** `\l`
*   **Create a New Database:**
```sql
CREATE DATABASE my_app_db;
```
    (You often need to be connected to the default `postgres` database to create others).
*   **Connect to a Database (psql):** `\c my_app_db`
*   **Connect to a Database (pgAdmin):** Just click on it in the browser tree.
*   **Drop a Database:** (Be careful! This deletes everything inside!)
    ```sql
    DROP DATABASE my_app_db;
    ```
    (You cannot be connected to `my_app_db` to drop it).

**Working with Schemas:**

*   **List Schemas (psql):** `\dn`
*   **Create a New Schema:**
```sql
CREATE SCHEMA app_data;
```
*   **Set Search Path:** PostgreSQL looks for tables in schemas listed in the `search_path`. By default, it's often `$user, public`. You can change it:
```sql
SET search_path TO app_data, public; -- Now it looks in app_data first, then public
```
*   **Drop a Schema:**
```sql
DROP SCHEMA app_data; -- Fails if schema is not empty
DROP SCHEMA app_data CASCADE; -- Drops schema and all contained objects (tables, etc.)
```

**Common Data Types:**

*   `SERIAL`: Auto-incrementing integer (often used for primary keys).
*   `INT`, `BIGINT`: Integer numbers.
*   `VARCHAR(n)`: Variable-length string, up to `n` characters. Use `TEXT` for unlimited length.
*   `BOOLEAN`: True or False.
*   `DATE`: Date (year, month, day).
*   `TIME`: Time of day.
*   `TIMESTAMP`: Date and time. Use `TIMESTAMP WITH TIME ZONE` if you need timezone awareness.
*   `NUMERIC(p, s)`, `DECIMAL(p, s)`: Exact numeric with `p` total digits and `s` digits after the decimal point.
*   `REAL`, `DOUBLE PRECISION`: Floating-point numbers.
*   `UUID`: Universally Unique Identifier.
*   `JSON`, `JSONB`: For storing JSON data (`JSONB` is better for querying).

**Common Constraints:**

*   `PRIMARY KEY`: Uniquely identifies each row. Implicitly creates a unique index and is `NOT NULL`. A table can only have one primary key.
*   `FOREIGN KEY`: Ensures referential integrity between tables. A column(s) in one table that references the `PRIMARY KEY` or `UNIQUE` key of another table.
*   `NOT NULL`: Ensures a column cannot contain `NULL` values.
*   `UNIQUE`: Ensures all values in a column (or group of columns) are distinct.
*   `DEFAULT value`: Provides a default value for a column if none is specified during insertion.
*   `CHECK (condition)`: Ensures values in a column satisfy a specific condition.

**Example: Creating a `products` table**

Creating a table in the `public` schema. You can run this in the pgAdmin Query Tool or psql.

```sql
-- Connect to your database first, e.g., \c postgres or \c my_app_db

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY, -- Auto-incrementing primary key
    name VARCHAR(255) NOT NULL UNIQUE, -- Product name, must be unique and not null
    description TEXT, -- Optional longer description
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0), -- Price, numeric with 2 decimal places, must be non-negative
    stock INT NOT NULL DEFAULT 0, -- Stock count, defaults to 0 if not provided
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() -- Timestamp when the product was added, defaults to current time
);

-- After creating, you can describe it in psql: \d products
-- Or view it in pgAdmin under Databases -> your_db -> Schemas -> public -> Tables -> products -> Columns/Constraints tab
```

**Example: Creating an `orders` table with a Foreign Key**

Now let's create another table that links back to the `products` table.

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL REFERENCES products(product_id), -- Foreign Key referencing products table
    quantity INT NOT NULL CHECK (quantity > 0),
    order_date TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- In psql: \d orders
-- In pgAdmin: Check the Constraints tab for the foreign key definition.
```
The `REFERENCES products(product_id)` clause means that any `product_id` value inserted into the `orders` table *must* already exist in the `product_id` column of the `products` table.

#### Modifying Tables (`ALTER TABLE`)

You can change table structure after creation.

*   **Add a Column:**
    ```sql
    ALTER TABLE products
    ADD COLUMN category VARCHAR(100);
    ```
*   **Drop a Column:**
    ```sql
    ALTER TABLE products
    DROP COLUMN description;
    ```
*   **Add a Constraint:** (e.g., add a CHECK constraint)
    ```sql
    ALTER TABLE products
    ADD CONSTRAINT check_stock_positive CHECK (stock >= 0); -- We already added this during creation, but this is how you'd add later
    ```
*   **Drop a Constraint:**
    ```sql
    ALTER TABLE products
    DROP CONSTRAINT check_stock_positive;
    ```
*   **Rename a Column:**
    ```sql
    ALTER TABLE products
    RENAME COLUMN name TO product_name;
    ```
*   **Rename a Table:**
    ```sql
    ALTER TABLE products
    RENAME TO inventory;
    ```

#### Dropping Tables (`DROP TABLE`)

Removes a table and all its data. Be careful!

```sql
DROP TABLE orders; -- Fails if other objects (like foreign keys) depend on it
DROP TABLE orders CASCADE; -- Drops the table AND any objects that depend on it (like the foreign key in products, though in this case the foreign key is *in* orders, not *on* orders)

DROP TABLE products; -- Fails because orders table has a foreign key referencing it
```
To drop both `orders` and `products` tables:
```sql
DROP TABLE orders;
DROP TABLE products; -- This will now work if orders is dropped first, as the foreign key dependency is gone.
```

---

### 4. Basic Data Manipulation (CRUD)

Now that you have tables, let's add, retrieve, update, and delete data. We'll use the `products` table again.

#### 4.1 Inserting Data (`INSERT INTO`)

```sql
-- Inserting a single row
INSERT INTO products (name, price, stock)
VALUES ('Laptop', 1200.00, 50); -- product_id, created_at use defaults

-- Inserting another row, specifying all columns
INSERT INTO products (product_id, name, description, price, stock, created_at)
VALUES (DEFAULT, 'Keyboard', 'Mechanical keyboard', 75.50, 150, NOW()); -- Use DEFAULT for serial

-- Inserting multiple rows
INSERT INTO products (name, price, stock)
VALUES
    ('Mouse', 25.00, 200),
    ('Monitor', 300.00, 30);

-- Check the inserted data (see SELECT below)
-- SELECT * FROM products;
```

#### Selecting Data (`SELECT`)

This is how you query and retrieve data.

```sql
-- Select all columns and all rows from the products table
SELECT * FROM products;

-- Select specific columns
SELECT name, price FROM products;

-- Select with conditions (filtering rows) using WHERE
SELECT * FROM products WHERE stock < 100;
SELECT name, price FROM products WHERE price > 100 AND stock > 10;
SELECT * FROM products WHERE name LIKE 'M%'; -- Finds names starting with 'M'
SELECT * FROM products WHERE product_id IN (1, 3);
SELECT * FROM products WHERE description IS NULL;

-- Order the results
SELECT name, price FROM products ORDER BY price DESC; -- Descending price
SELECT name, stock FROM products ORDER BY stock ASC, name ASC; -- Ascending stock, then ascending name

-- Limit the number of results
SELECT * FROM products LIMIT 2;

-- Combine ordering and limiting
SELECT name, price FROM products ORDER BY price DESC LIMIT 1; -- The most expensive product

-- Count rows
SELECT COUNT(*) FROM products;
SELECT COUNT(*) FROM products WHERE stock = 0;
```

#### Updating Data (`UPDATE`)

Modify existing data in rows.

```sql
-- Update stock for a specific product
UPDATE products
SET stock = 75
WHERE name = 'Laptop'; -- IMPORTANT: Always use WHERE or you'll update *all* rows!

-- Increase price for products with low stock
UPDATE products
SET price = price * 1.10 -- Increase price by 10%
WHERE stock < 50;

-- Update multiple columns
UPDATE products
SET stock = stock - 5, description = 'Updated description'
WHERE name = 'Keyboard';

-- Update all rows (use with caution!)
-- UPDATE products SET price = price * 0.9; -- 10% discount on all products
```

#### Deleting Data (`DELETE FROM`)

Remove rows from a table.

```sql
-- Delete products with zero stock
DELETE FROM products WHERE stock = 0; -- IMPORTANT: Always use WHERE or you'll delete *all* rows!

-- Delete a specific product by ID
DELETE FROM products WHERE product_id = 2;

-- Delete all rows (truncates the table effectively, but can be rolled back)
-- DELETE FROM products;
```

---

### 5. Programmable Objects: Functions, Views, and Triggers

These allow you to encapsulate logic and simplify operations.

#### Functions (often what users mean by "Procedures" in PostgreSQL)

Functions execute a series of SQL commands and return a value. PostgreSQL's procedural language is called PL/pgSQL.

```sql
-- Example: A function to get the total number of products in stock
CREATE OR REPLACE FUNCTION get_total_stock()
RETURNS INTEGER AS $$ -- $$ is a dollar-quoted string literal, useful for multi-line code
DECLARE
    total_stock_count INTEGER; -- Declare a variable
BEGIN
    -- Select the sum of the stock column into our variable
    SELECT SUM(stock) INTO total_stock_count FROM products;

    -- Return the variable
    RETURN total_stock_count;
END;
$$ LANGUAGE plpgsql; -- Specify the language used

-- How to call the function
SELECT get_total_stock();

-- Example: A function that takes an argument
CREATE OR REPLACE FUNCTION get_product_price(product_name VARCHAR(255))
RETURNS NUMERIC(10, 2) AS $$
DECLARE
    price_value NUMERIC(10, 2);
BEGIN
    SELECT price INTO price_value FROM products WHERE name = product_name;

    IF price_value IS NULL THEN
        -- You could raise an error or return a default value if product not found
        RETURN NULL; -- Or handle error: RAISE EXCEPTION 'Product % not found', product_name;
    END IF;

    RETURN price_value;
END;
$$ LANGUAGE plpgsql;

-- How to call the function
SELECT get_product_price('Laptop');
SELECT get_product_price('NonExistentProduct'); -- Returns NULL
```

*   `CREATE OR REPLACE FUNCTION`: Creates a new function or replaces an existing one.
*   `RETURNS ...`: Defines the data type the function returns.
*   `$$ ... $$`: Delimits the function body (PL/pgSQL code).
*   `DECLARE`: Section for variable declarations.
*   `BEGIN ... END`: The main body of the function code.
*   `LANGUAGE plpgsql`: Specifies the procedural language.

#### Stored Procedures (`CREATE PROCEDURE` / `CALL`)


```sql
-- Example: A simple procedure that doesn't return a value
CREATE OR REPLACE PROCEDURE log_message(msg TEXT)
AS $$
BEGIN
    RAISE NOTICE 'Logging message: %', msg; -- Simple output for demonstration
END;
$$ LANGUAGE plpgsql;

-- How to call the procedure
CALL log_message('Application started');
```

#### Views (`CREATE VIEW`)

Views are virtual tables based on the result set of a `SELECT` query. They don't store data themselves but provide a simplified way to access data from one or more tables.

```sql
-- Example: A view showing only products with low stock (under 100)
CREATE OR REPLACE VIEW low_stock_products AS
SELECT product_id, name, stock
FROM products
WHERE stock < 100;

-- How to query the view (looks just like querying a table)
SELECT * FROM low_stock_products;

-- Example: A view joining products and orders (assuming you re-created orders)
CREATE OR REPLACE VIEW order_details AS
SELECT
    o.order_id,
    p.name AS product_name, -- Rename columns for clarity
    o.quantity,
    o.order_date,
    (p.price * o.quantity) AS total_item_price -- Calculate a derived column
FROM orders o -- Alias orders as 'o'
JOIN products p ON o.product_id = p.product_id; -- Join the tables

-- Query the joined view
SELECT * FROM order_details WHERE order_id = 1;

-- Drop a view
DROP VIEW low_stock_products;
```

#### Triggers (`CREATE TRIGGER`)

Triggers are functions that are automatically executed *before* or *after* an `INSERT`, `UPDATE`, or `DELETE` operation on a specific table. They are often used for auditing, enforcing complex business rules, or data synchronization.

A trigger involves two parts:
1.  A **trigger function:** A PL/pgSQL function (or other language) that contains the logic to be executed. This function must return type `TRIGGER`.
2.  The **trigger definition:** Links the trigger function to a specific table and specifies *when* (BEFORE/AFTER), *what* (INSERT/UPDATE/DELETE), and *how* often (FOR EACH ROW/FOR EACH STATEMENT) it fires.

```sql
-- Example: Create an audit log when product prices are updated

-- 1. Create the audit table (assuming you have a separate schema/database for audit logs)
-- CREATE TABLE product_price_audit (
--     audit_id SERIAL PRIMARY KEY,
--     product_id INT,
--     old_price NUMERIC(10, 2),
--     new_price NUMERIC(10, 2),
--     changed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
--     changed_by VARCHAR(255) DEFAULT SESSION_USER -- Get the current user
-- );

-- Let's just use RAISE NOTICE for demonstration to keep it simple

-- 2. Create the trigger function
CREATE OR REPLACE FUNCTION log_price_change()
RETURNS TRIGGER AS $$ -- Trigger functions must return TRIGGER
BEGIN
    -- Check if the price was actually changed
    IF NEW.price <> OLD.price THEN -- NEW and OLD are special records representing the row data
        -- In a real scenario, you'd INSERT into product_price_audit table
        -- Example INSERT (commented out as we don't have the audit table here):
        -- INSERT INTO product_price_audit (product_id, old_price, new_price)
        -- VALUES (OLD.product_id, OLD.price, NEW.price);

        RAISE NOTICE 'Price changed for product ID %: from % to %', OLD.product_id, OLD.price, NEW.price;
    END IF;

    RETURN NEW; -- For BEFORE triggers, returning NEW is important to continue the operation
                -- For AFTER triggers, the return value is ignored, but RETURN NEW/OLD is convention
END;
$$ LANGUAGE plpgsql;

-- 3. Create the trigger
CREATE TRIGGER product_price_update_trigger
AFTER UPDATE ON products -- Fire this trigger AFTER an UPDATE operation on the products table
FOR EACH ROW -- Execute the trigger function for *each* row that is affected by the UPDATE
WHEN (OLD.price IS DISTINCT FROM NEW.price) -- Optional: Only fire if the price column specifically changed
EXECUTE FUNCTION log_price_change(); -- Call the trigger function

-- Test the trigger (run this in pgAdmin/psql)
-- Make sure you have some products inserted first
-- SELECT product_id, price FROM products; -- Check current prices

UPDATE products
SET price = price * 1.05 -- Increase price by 5%
WHERE name = 'Keyboard'; -- Assuming Keyboard exists

-- Check the output/messages tab in pgAdmin, or the psql output, you should see the NOTICE message.

-- Update a column *other* than price - trigger should NOT fire (due to WHEN clause)
UPDATE products
SET stock = stock + 10
WHERE name = 'Keyboard';

-- Drop the trigger
DROP TRIGGER product_price_update_trigger ON products;

-- Drop the trigger function (you must drop the trigger first)
-- DROP FUNCTION log_price_change();
```

*   `BEFORE`/`AFTER`: Specifies when the trigger fires relative to the data modification event.
*   `INSERT`/`UPDATE`/`DELETE`/`TRUNCATE`: Specifies which event(s) trigger the function.
*   `FOR EACH ROW`/`FOR EACH STATEMENT`:
    *   `FOR EACH ROW`: Executes the trigger function once for *each row* affected by the command. Access `OLD` and `NEW` row data.
    *   `FOR EACH STATEMENT`: Executes the trigger function once per SQL statement, regardless of how many rows are affected. Cannot access `OLD` or `NEW` row data directly.
*   `WHEN (condition)`: An optional condition checked *before* executing the `FOR EACH ROW` trigger function. Access `OLD` and `NEW` data.
*   `EXECUTE FUNCTION function_name()`: Specifies the trigger function to call.

### 6. Resources

*   **Sample Repository:** . [DBRepo](https://github.com/rainu1729/pgdbartifact)
