# Setting Up Apache Airflow Locally with Docker Compose (Ubuntu)

This guide explains how to quickly spin up Apache Airflow on your local Ubuntu machine using Docker Compose. 
Ensure Docker and Docker Compose are already installed.

## Steps

1. **In a directory of your choice clone the docker compose file (Airflow 3.0.6) and create 3 directories**
```bash
curl -LfO 'curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.0.6/docker-compose.yaml''
mkdir -p ./dags ./logs ./plugins ./config
```

3. **Initialize Airflow set the AIRFLOW_UID to current user's id and save in a .env file**
```bash
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

4. **Start Airflow Services**
```bash
docker compose up
```

5. **Access the Airflow Web UI**

    Open your browser and go to: [http://localhost:8080](http://localhost:8080)

    Default credentials:
    - Username: `airflow`
    - Password: `airflow`

## References

- [Official Apache Airflow Docker Setup Guide](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
