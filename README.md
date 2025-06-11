# Application Event Management SaaS - README

## 1. Introduction

Welcome to the Application Event Management SaaS platform! This application is designed to operate in a Software as a Service (SaaS) model with multi-tenant capabilities, allowing different organizers to manage their events in isolated environments.

This document provides comprehensive instructions on how to set up and run this application, from local development to production deployment.

### Core Technologies

The platform is built upon a modern stack of technologies to ensure scalability, reliability, and maintainability. Here's a brief overview of the core components:

*   **[Laravel](https://laravel.com/docs):** A PHP web application framework used for the backend logic of the application.
*   **[Docker](https://docs.docker.com/get-started/):** A platform for developing, shipping, and running applications in containers. It's used to package the application and its dependencies.
*   **[Docker Compose](https://docs.docker.com/compose/):** A tool for defining and running multi-container Docker applications. It's primarily used for setting up the development environment.
*   **[Docker Swarm](https://docs.docker.com/engine/swarm/):** An orchestration tool for managing a cluster of Docker engines. It's used for deploying and scaling the application in a production environment.
*   **[PostgreSQL](https://www.postgresql.org/docs/):** A powerful, open-source object-relational database system used for both the central application data and the per-tenant databases.
*   **[Redis](https://redis.io/documentation):** An in-memory data structure store, used for caching and as a message broker for background queues.
*   **[Nginx](https://nginx.org/en/docs/):** A high-performance web server, reverse proxy, and load balancer. It serves static content and proxies requests to the PHP-FPM service.
*   **PHP-FPM (FastCGI Process Manager):** An alternative PHP FastCGI implementation with some additional features useful for heavy-loaded sites.
*   **[Traefik](https://doc.traefik.io/traefik/) / Nginx (as Reverse Proxy):** Used in production to manage incoming traffic, handle SSL/TLS termination, and route requests to the appropriate services based on subdomains for multi-tenancy. (Note: The documentation mentions Traefik or Nginx for this role; specific setup might depend on the chosen implementation).

Understanding the basics of these technologies will be helpful, though this guide aims to be as beginner-friendly as possible.

## 2. Prerequisites

Before you begin, ensure you have the following software installed on your system. These are essential for running the application both in development and for understanding the production deployment.

*   **Docker Engine:** Docker is used to containerize the application, making it easy to manage dependencies and ensure consistent environments.
    *   **Windows:** Install [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/).
    *   **macOS:** Install [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/).
    *   **Linux:** Install Docker Engine by following the instructions for your specific distribution:
        *   [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
        *   [Debian](https://docs.docker.com/engine/install/debian/)
        *   [CentOS](https://docs.docker.com/engine/install/centos/)
        *   [Fedora](https://docs.docker.com/engine/install/fedora/)
        *   For other distributions, please refer to the [official Docker documentation](https://docs.docker.com/engine/install/).

*   **Docker Compose:** Docker Compose is used to define and run multi-container Docker applications, which is essential for the development setup.
    *   Docker Desktop for Windows and Mac typically includes Docker Compose.
    *   For Linux systems, if it's not included with your Docker installation, follow the [Docker Compose installation guide](https://docs.docker.com/compose/install/). You can check if it's installed by running `docker-compose --version`.

*   **Git (Optional, but Recommended):** If you plan to clone the repository from a Git source control system (like GitHub, GitLab), you'll need Git.
    *   Download and install Git from [git-scm.com](https://git-scm.com/downloads).

*   **Web Browser:** A modern web browser (like Chrome, Firefox, Safari, or Edge) is needed to access the web application.

*   **Terminal/Command Line:** You will need to use a terminal or command-line interface to run Docker commands and interact with the setup process.
    *   Windows: PowerShell or Command Prompt (or WSL for a Linux-like environment).
    *   macOS: Terminal.app.
    *   Linux: Your distribution's default terminal (e.g., GNOME Terminal, Konsole).

Please ensure these prerequisites are correctly installed and configured before proceeding to the next steps.

## 3. Development Environment Setup

This section will guide you through setting up the application for local development using Docker Compose. This setup mirrors the production services but is simplified for ease of use on a local machine.

### 3.1. Clone the Repository

If you haven't already, clone the application repository to your local machine. If the project is hosted on a platform like GitHub, you would typically use:

```bash
git clone <repository_url>
cd <repository_directory>
```
Replace `<repository_url>` with the actual URL of the repository and `<repository_directory>` with the name of the folder created by Git.

### 3.2. Configure Environment Variables

The application uses environment variables for configuration. These are typically managed through an `.env` file.

1.  **Copy the example environment file:**
    Most Laravel projects provide an `.env.example` file. Copy this to `.env`:
    ```bash
    cp .env.example .env
    ```
    If an `.env.example` file is not present, you may need to create the `.env` file manually based on the required configurations for database connections, application keys, Redis, etc. Refer to the `docker-compose.yml` file and Laravel documentation for common variables.

2.  **Generate Application Key:**
    Laravel applications require an application key, which is used for encryption. Generate it using Artisan (Laravel's command-line tool) via Docker Compose:
    ```bash
    docker-compose run --rm php-fpm php artisan key:generate
    ```
    *(Note: The service name `php-fpm` in the command above should match the name of your PHP service in the `docker-compose.yml` file. It might be `app`, `php`, or similar.)*

3.  **Review and Customize `.env`:**
    Open the `.env` file and customize variables as needed. Key variables to check include:
    *   `DB_HOST`: Should usually be the name of your database service in `docker-compose.yml` (e.g., `pgsql_central`, `db`).
    *   `DB_PORT`: The port your database service is listening on (e.g., `5432` for PostgreSQL).
    *   `DB_DATABASE`: The name of the central database.
    *   `DB_USERNAME` and `DB_PASSWORD`: Credentials for the database.
    *   `REDIS_HOST`: Should be the name of your Redis service in `docker-compose.yml` (e.g., `redis`).
    *   `APP_URL`: Set this to `http://localhost:PORT` or `http://your-local-domain.test` if you configure a custom domain. `PORT` would be the port your web server (Nginx) is exposed on.

### 3.3. Build and Run Docker Containers

The `docker-compose.yml` file in the root of the project defines all the services needed for the application (web server, PHP, database, Redis, etc.).

1.  **Build the Docker images:**
    This command builds the images defined in your `docker-compose.yml` file, such as your custom PHP-FPM and Nginx images.
    ```bash
    docker-compose build
    ```

2.  **Start the services:**
    This command starts all the services in detached mode (running in the background).
    ```bash
    docker-compose up -d
    ```

    You can view the logs of the running containers using:
    ```bash
    docker-compose logs -f
    ```
    Or for a specific service:
    ```bash
    docker-compose logs -f <service_name>
    ```
    (e.g., `docker-compose logs -f php-fpm`)

### 3.4. Run Database Migrations and Seeders

Once the containers are running, especially the database container, you need to set up the database schema and potentially populate it with initial data.

1.  **Run Central Database Migrations:**
    This command executes Laravel's database migrations for the central tenant database.
    ```bash
    docker-compose run --rm php-fpm php artisan migrate --path=/var/www/html/database/migrations/central
    ```
    *(Adjust the service name `php-fpm` and the migration path if necessary. The path should point to where your central migrations are stored.)*

2.  **Run Central Database Seeders (Optional):**
    If there are seeders to populate the central database with initial data (e.g., a super admin user):
    ```bash
    docker-compose run --rm php-fpm php artisan db:seed --class=CentralDatabaseSeeder
    ```
    *(Adjust the seeder class name as needed.)*

    *Note on Multi-tenancy Migrations:* For a multi-tenant application using Stancl Tenancy or a similar package, tenant migrations are often run automatically when a new tenant is created or via a specific command. The initial setup usually focuses on the central database. Tenant-specific migrations will be handled later, often after creating a tenant.

### 3.5. Accessing the Application

Once all services are running and the database is set up:

*   **Web Application:** Open your web browser and navigate to `http://localhost:PORT` or the `APP_URL` you configured in your `.env` file. The `PORT` is the one exposed by your Nginx service in `docker-compose.yml` (commonly 80, 8000, or 8080).
*   **Application Logs:** Check `storage/logs/laravel.log` inside the PHP container or use `docker-compose logs -f php-fpm`.

### 3.6. Stopping the Environment

To stop all running services:
```bash
docker-compose down
```
To stop and remove volumes (useful for a clean restart, **be careful as this deletes database data**):
```bash
docker-compose down -v
```

This setup should provide you with a fully functional local development environment for the application.

## 4. Production Environment Setup (Docker Swarm)

This section outlines the steps and considerations for deploying the application to a production environment using Docker Swarm. Docker Swarm provides orchestration, scalability, and high availability for containerized applications.

**Disclaimer:** Deploying to production is a complex task that requires careful planning and understanding of your infrastructure. This guide provides a general overview based on the described architecture. Always adapt these instructions to your specific hosting environment and security requirements.

### 4.1. Docker Swarm Overview

*   **Docker Swarm:** The application is designed to be deployed on a Docker Swarm cluster. Swarm mode is built into Docker and allows you to manage a cluster of Docker nodes (managers and workers).
*   **Managers:** Responsible for cluster management, task scheduling, and maintaining the desired state. Aim for an odd number of managers (e.g., 3 or 5) for high availability.
*   **Workers:** Execute the application containers (services).

Refer to the [official Docker Swarm documentation](https://docs.docker.com/engine/swarm/) for detailed information on Swarm mode.

### 4.2. Initial Swarm Setup

1.  **Initialize the Swarm (on the first manager node):**
    ```bash
    docker swarm init --advertise-addr <MANAGER_IP>
    ```
    Replace `<MANAGER_IP>` with the IP address of the manager node. This command will output a token.

2.  **Join Manager Nodes (optional, for HA):**
    On other nodes that you want to act as managers, run the join command provided after `docker swarm init` (it will include a manager token).
    ```bash
    docker swarm join --token <MANAGER_TOKEN> <MANAGER_IP>:<PORT>
    ```

3.  **Join Worker Nodes:**
    On nodes that will act as workers, run the join command (it will include a worker token, also provided by `docker swarm init` or by running `docker swarm join-token worker` on a manager).
    ```bash
    docker swarm join --token <WORKER_TOKEN> <MANAGER_IP>:<PORT>
    ```

### 4.3. Docker Stack File

In a Swarm environment, applications are typically deployed as a "stack" defined in a YAML file (e.g., `docker-stack.yml` or `docker-compose.prod.yml`). This file is similar to a `docker-compose.yml` but includes Swarm-specific configurations like deployment strategies, replicas, resource limits, and placement constraints.

The stack file would define services like:
*   `nginx` (web server)
*   `php-fpm` (PHP application)
*   `pgsql_central` (central PostgreSQL database)
*   `redis` (cache and queue)
*   `queue-worker` (Laravel queue workers)
*   A reverse proxy/load balancer (e.g., Traefik or another Nginx instance)

**Important Note:** You will need to create a production-ready `docker-stack.yml` (or a similarly named file like `docker-compose.prod.yml`). This file defines how your services run in Docker Swarm. You can adapt your development `docker-compose.yml` as a starting point, but it must be modified for Swarm's declarative service model, incorporating settings for replicas, deployment strategies, secrets, configs, and volumes suitable for production, as outlined in the `deployment_documentation.md`.

### 4.4. Deploying the Stack

Once you have your `docker-stack.yml` file ready and your Swarm is initialized:

1.  **Copy the stack file to a manager node.**
2.  **Deploy the stack:**
    ```bash
    docker stack deploy -c docker-stack.yml <your_stack_name>
    ```
    Replace `<your_stack_name>` with a name for your application stack (e.g., `eventapp_prod`).

    This command will create/update the services defined in the stack file on the Swarm.

### 4.5. Key Production Considerations (from `deployment_documentation.md`)

These aspects are crucial for a robust production deployment:

*   **Reverse Proxy / Load Balancer (Traefik/Nginx):**
    *   **Role:** Manages incoming traffic, handles SSL/TLS termination (HTTPS), and routes requests to the correct services, especially important for multi-tenancy with subdomains.
    *   **Setup:** This component typically runs as a service in the Swarm, often on edge nodes. Its configuration will involve defining entrypoints for HTTP/HTTPS and rules for routing based on hostnames (subdomains) to the appropriate `nginx` service of the application.
    *   Refer to [Traefik Docker Swarm documentation](https://doc.traefik.io/traefik/providers/docker/) or Nginx deployment guides.

*   **Persistent Volumes:**
    *   **Purpose:** To ensure data persistence for databases (central and tenant PostgreSQL instances) and any uploaded media files, even if containers are restarted or rescheduled.
    *   **Implementation:** Use Docker named volumes with appropriate drivers for your storage backend (e.g., local storage on specific nodes, NFS, cloud storage plugins). Define these volumes in your `docker-stack.yml`.
    *   Example snippet for a service in `docker-stack.yml`:
        ```yaml
        services:
          pgsql_central:
            image: postgres:latest
            volumes:
              - pg_central_data:/var/lib/postgresql/data
        volumes:
          pg_central_data: # Define the named volume
            # driver: specific_driver_if_needed
        ```

*   **Docker Secrets:**
    *   **Purpose:** Securely manage sensitive data like database passwords, API keys, and other credentials. Secrets are encrypted and only accessible to services granted permission.
    *   **Usage:**
        1.  Create secrets: `echo "mysecretpassword" | docker secret create db_password -`
        2.  Assign secrets to services in `docker-stack.yml`:
            ```yaml
            services:
              php-fpm:
                image: myapp/php-fpm:latest
                secrets:
                  - source: db_password # Name of the secret created
                    target: db_password # Filename inside the container /run/secrets/
            secrets:
              db_password:
                external: true # Indicates the secret is pre-existing
            ```
        *   Your application code (e.g., Laravel's `config/database.php`) would then read these secrets from the specified file path (e.g., `/run/secrets/db_password`).
    *   For non-sensitive configuration that might still vary by environment (and isn't suitable for baking into the image), Docker Configs can be used in a similar way to secrets, or environment variables can be set directly in the service definition within the `docker-stack.yml` file for Swarm services.

*   **Overlay Networks:**
    *   **Purpose:** Enable secure communication between services distributed across different nodes in the Swarm.
    *   **Usage:** Define overlay networks in your `docker-stack.yml` and attach services to them. Docker Swarm handles the routing.
        ```yaml
        services:
          php-fpm:
            networks:
              - app-network
          nginx:
            networks:
              - app-network
        networks:
          app-network:
            driver: overlay
            # attachable: true # If you need to connect other containers manually
        ```

*   **Database Migrations and Seeding in Production:**
    *   Migrations for the central database and tenant databases need to be part of your deployment pipeline or run manually with care.
    *   For the central database, after deploying a new version of the application service (`php-fpm`), you might run migrations as a one-off task using `docker service update --force <php_service_id>` with a command override, or by exec-ing into a running `php-fpm` container if direct task execution isn't feasible. A more robust approach is to use a job/task container that runs migrations and then exits.
    *   Example (conceptual, actual command might vary):
        ```bash
        # This is a simplified example; production migration strategies can be complex.
        # Ensure only one instance of migrations runs.
        docker service scale <your_stack_name>_php-fpm=0 # Scale down to avoid conflicts
        # Run migrations using a temporary service or by updating the existing one with a command
        docker service update --command "php artisan migrate --force" <your_stack_name>_php-fpm
        # Monitor logs, then scale back up
        docker service scale <your_stack_name>_php-fpm=<desired_replicas>
        ```

*   **Scalability:**
    *   Adjust the `replicas` count for your services (e.g., `php-fpm`, `queue-worker`, `nginx`) in the `docker-stack.yml` and redeploy the stack or use `docker service scale <service_name>=<count>`.

### 4.6. Accessing the Production Application

Once deployed and the reverse proxy is configured with DNS pointing to its public IP:
*   The application should be accessible via the configured domain/subdomains.
*   The central application might be at `https://app.yourdomain.com`.
*   Tenant applications would be at `https://tenant1.yourdomain.com`, `https://tenant2.yourdomain.com`, etc.

This section provides a high-level guide. Each point (secrets, volumes, networking, reverse proxy) involves detailed configuration specific to your environment and choices (e.g., Traefik vs. Nginx). Always consult the official documentation for each tool.

## 5. Multi-tenancy Setup

This application is designed with a multi-tenant architecture, where each "Organization" (tenant) has its own isolated database. A central database stores global information like SuperAdmins, Admins, Organizers, and tenant configurations.

### 5.1. Architecture Overview

*   **Central Database (PostgreSQL):** Stores global application data and metadata about tenants.
*   **Tenant Databases (PostgreSQL):** Each tenant (Organization) has a dedicated PostgreSQL database, ensuring data isolation and security.
*   **Tenant Identification:** The application identifies tenants based on subdomains (e.g., `organizer1.yourapp.com`, `organizer2.yourapp.com`). The reverse proxy (Traefik/Nginx) routes requests to the main application, which then uses a tenancy package (like Stancl Tenancy for Laravel) to switch to the correct tenant context and database.

### 5.2. Provisioning a New Tenant

The exact process for provisioning a new tenant depends on the specific multi-tenancy package and custom logic implemented in the application. If using Stancl Tenancy for Laravel, the process generally involves these steps:

1.  **Creating an Organizer/Tenant Record in the Central Database:**
    *   This is often done through a SuperAdmin or Admin interface in the application.
    *   This step typically involves creating a new `Tenant` model entry in the central database, which includes information like the tenant's ID, desired subdomain, and other configuration details.

2.  **Automated Tenant Database Creation:**
    *   Upon creation of the tenant record, the tenancy package (or custom scripts) usually triggers the creation of a new PostgreSQL database for this tenant.
    *   Database user and permissions for this new database are also typically configured automatically.

3.  **Running Migrations for the New Tenant Database:**
    *   Once the tenant database is created, its schema needs to be set up. The tenancy package often handles running specific "tenant" migrations against this new database.
    *   If using Stancl Tenancy, this might be done automatically after tenant creation or via an Artisan command like:
        ```bash
        # Example for Stancl Tenancy - run from within the php-fpm container or via docker-compose exec
        php artisan tenants:migrate <tenant_id>
        ```
        Replace `<tenant_id>` with the actual ID of the tenant. Some setups might run migrations for all tenants with a single command.

4.  **Seeding Data for the New Tenant Database (Optional):**
    *   If new tenants require initial data (e.g., default settings, user roles specific to tenants), tenant-specific seeders can be run.
    *   Example for Stancl Tenancy:
        ```bash
        php artisan tenants:seed --class=TenantDatabaseSeeder <tenant_id>
        ```

5.  **DNS Configuration:**
    *   A DNS record (usually a CNAME or A record) needs to be created for the new tenant's subdomain (e.g., `neworganizer.yourapp.com`) pointing to your load balancer/reverse proxy's public IP address.
    *   The reverse proxy (Traefik/Nginx) should be configured to automatically pick up new subdomains or have wildcard SSL certificates to handle them. Traefik, for instance, can often detect new services/routes via Docker labels if configured correctly.

**Key Commands (Examples for Stancl Tenancy):**

These commands are typically run via `docker-compose run --rm php-fpm php artisan <command>` in development or by exec-ing into the production `php-fpm` container.

*   `php artisan tenants:create`: Often an interactive command to create a new tenant.
*   `php artisan tenants:list`: Lists existing tenants.
*   `php artisan tenants:migrate [--tenants=<tenant_id>]`: Runs migrations for specific or all tenants.
*   `php artisan tenants:run <command> [--tenants=<tenant_id>]`: Runs an arbitrary Artisan command for tenants.

**Important Considerations:**

*   **Automation:** The provisioning process should be as automated as possible to ensure consistency and reduce manual effort.
*   **Database User Permissions:** Ensure that the database user for each tenant database has only the necessary permissions for that specific database.
*   **Backups:** Tenant databases must be included in the overall backup strategy.
*   **Resource Allocation:** Monitor resource usage (CPU, memory, disk space) as the number of tenants grows.

Refer to the documentation of the specific multi-tenancy package used in the project (e.g., [Stancl Tenancy Documentation](https://tenancyforlaravel.com/docs/)) for precise commands and best practices.

## 6. DevOps Practices

The `deployment_documentation.md` emphasizes several DevOps practices crucial for maintaining a robust, scalable, and reliable SaaS application. While detailed implementation of these is beyond the scope of a basic setup guide, this section provides a brief overview and pointers.

### 6.1. CI/CD (Continuous Integration/Continuous Deployment)

*   **Concept:** Automating the process of building, testing, and deploying application updates. This ensures faster, more reliable releases.
*   **Typical Pipeline Stages:**
    1.  **Code Commit:** Developer pushes code to a repository (e.g., Git).
    2.  **Build:** The CI server pulls the code and builds Docker images.
    3.  **Test:** Automated tests (unit, integration, functional) are run against the build.
    4.  **Deploy to Staging (Optional):** Deploy to a staging environment for further testing.
    5.  **Deploy to Production:** Deploy to the Docker Swarm cluster (e.g., using `docker stack deploy`). This step should also handle database migrations carefully.
*   **Tools:** Jenkins, GitLab CI/CD, GitHub Actions, CircleCI, etc.
*   **Further Learning:** Search for "CI/CD with Docker Swarm" or "CI/CD for Laravel applications."

### 6.2. Monitoring

*   **Concept:** Continuously observing the health and performance of the application and infrastructure.
*   **Key Areas to Monitor:**
    *   Server/Node metrics (CPU, memory, disk, network).
    *   Container health and resource usage.
    *   Application performance (response times, error rates).
    *   Database performance (query times, connections).
    *   Queue lengths and worker activity.
*   **Tools:**
    *   **Prometheus:** A popular open-source monitoring and alerting toolkit.
    *   **Grafana:** For visualizing metrics collected by Prometheus or other sources.
    *   Application Performance Monitoring (APM) tools (e.g., New Relic, Datadog, or open-source alternatives).
*   **Further Learning:** Explore "Docker Swarm monitoring with Prometheus and Grafana."

### 6.3. Centralized Logging

*   **Concept:** Aggregating logs from all services and infrastructure components into a central location for easier searching, analysis, and troubleshooting.
*   **Benefits:** Simplifies debugging, allows for trend analysis, and helps in identifying security incidents.
*   **Tools (ELK/EFK Stack):**
    *   **Elasticsearch:** A distributed search and analytics engine for storing logs.
    *   **Logstash (or Fluentd/Filebeat):** For collecting, processing, and forwarding logs to Elasticsearch.
    *   **Kibana:** A web interface for searching and visualizing logs in Elasticsearch.
*   **Docker Logging Drivers:** Docker supports various logging drivers that can forward container logs to external systems.
*   **Further Learning:** Research "Centralized logging for Docker Swarm" or "ELK stack Docker."

### 6.4. Backup and Restoration

*   **Concept:** Regularly backing up critical data (especially databases) and having well-tested procedures for restoring data in case of failure or disaster.
*   **Strategy:**
    *   **Databases:** Implement regular automated backups for the central PostgreSQL database and all tenant PostgreSQL databases. Consider point-in-time recovery (PITR) if necessary.
    *   **Persistent Volumes:** Back up data stored in persistent volumes (e.g., user-uploaded files).
    *   **Configuration:** Version control your `docker-stack.yml`, `.env` files (with secrets managed separately and securely), and other critical configuration files.
*   **Tools & Techniques:** `pg_dump` for PostgreSQL, filesystem snapshots, cloud provider backup services, custom backup scripts.
*   **Further Learning:** Look into "PostgreSQL backup strategies" and "Docker volume backup solutions."

### 6.5. Security (General Reminder)

While not a separate DevOps category here, remember that security is an ongoing process:
*   Keep all software (OS, Docker, Laravel, databases, etc.) up to date with security patches.
*   Follow the principle of least privilege for all user accounts and service permissions.
*   Regularly review security configurations and audit logs.
*   Securely manage all secrets and credentials (as discussed in the Docker Swarm section).

Implementing these DevOps practices significantly contributes to the long-term success and stability of the application. Each of these topics is extensive, so continuous learning and adaptation are key.

## 7. Troubleshooting

This section lists common issues you might encounter and suggestions for resolving them.

### 7.1. Docker and Docker Compose Issues

*   **Issue: Docker daemon not running.**
    *   **Symptom:** Commands like `docker ps` or `docker-compose up` fail with messages like "Cannot connect to the Docker daemon."
    *   **Solution:** Ensure the Docker service/daemon is started on your machine. On Windows/macOS, check that Docker Desktop is running. On Linux, check the service status (e.g., `sudo systemctl status docker`).

*   **Issue: Port conflicts.**
    *   **Symptom:** `docker-compose up` fails with an error message indicating a port is already allocated or in use (e.g., "Bind for 0.0.0.0:80 failed: port is already allocated").
    *   **Solution:**
        *   Stop any other services running on the conflicting port. You can use tools like `netstat` or `ss` to find which process is using the port.
        *   Alternatively, change the port mapping in your `docker-compose.yml` file. For example, change `ports: - "80:80"` to `ports: - "8080:80"` to map port 80 inside the container to port 8080 on your host.

*   **Issue: Permission denied for Docker socket.**
    *   **Symptom (Linux):** Docker commands fail with permission errors.
    *   **Solution (Linux):**
        *   Add your user to the `docker` group: `sudo usermod -aG docker ${USER}`. You'll need to log out and log back in for this to take effect.
        *   Or, run Docker commands with `sudo` (not generally recommended for day-to-day use).

*   **Issue: `docker-compose build` fails.**
    *   **Symptom:** Errors during the image building process.
    *   **Solution:**
        *   Check the output carefully for specific error messages (e.g., missing packages, incorrect commands in `Dockerfile`).
        *   Ensure you have a stable internet connection, as the build process often downloads dependencies.
        *   Check for typos or syntax errors in your `Dockerfile`.

*   **Issue: Containers exit unexpectedly.**
    *   **Symptom:** `docker-compose up` starts containers, but some exit immediately or after a short time.
    *   **Solution:**
        *   Check the logs for the exiting container: `docker-compose logs <service_name>` or `docker logs <container_id>`. The logs usually provide clues about the cause (e.g., application errors, misconfiguration).
        *   Ensure all required environment variables are set correctly in the `.env` file.
        *   Verify that dependent services (like databases or Redis) are running and accessible.

### 7.2. Laravel Application Issues

*   **Issue: "No application encryption key has been specified."**
    *   **Symptom:** Laravel error page indicating the app key is missing.
    *   **Solution:** Run `docker-compose run --rm php-fpm php artisan key:generate` (adjust `php-fpm` if your service name is different).

*   **Issue: Database connection refused or other database errors.**
    *   **Symptom:** Errors related to database connectivity (e.g., "SQLSTATE[HY000] [2002] Connection refused").
    *   **Solution:**
        *   Ensure your database container (e.g., `pgsql_central`) is running: `docker-compose ps`.
        *   Verify `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD` in your `.env` file match the database service configuration in `docker-compose.yml` and the actual credentials used by the database. Remember `DB_HOST` should be the service name (e.g., `pgsql_central`), not `localhost`, when connecting from another container.
        *   Check the logs of the database container for any errors.

*   **Issue: Migrations fail.**
    *   **Symptom:** Errors when running `php artisan migrate`.
    *   **Solution:**
        *   Check the error message for details (e.g., syntax errors in migration files, issues with database schema).
        *   Ensure the database user has the correct permissions to create tables and modify the schema.
        *   If a migration was partially applied, you might need to manually fix the database state or use `php artisan migrate:rollback` (with caution).

*   **Issue: Subdomain routing not working for tenants.**
    *   **Symptom:** Accessing a tenant subdomain (e.g., `tenant1.yourapp.com`) leads to an error or the main application page without tenant context.
    *   **Solution:**
        *   **Development:** Ensure you have entries in your `/etc/hosts` file (or equivalent for Windows/macOS) mapping the tenant subdomains to `127.0.0.1` (e.g., `127.0.0.1 tenant1.localhost`).
        *   **Production:** Verify DNS records for the subdomains are correctly pointing to your reverse proxy/load balancer. Check the reverse proxy configuration (Traefik/Nginx) to ensure it's correctly routing based on hostnames and forwarding to the application service.
        *   Check the tenancy package configuration within Laravel.

### 7.3. Docker Swarm (Production) Issues

*   **Issue: Service fails to deploy or replicas don't start.**
    *   **Symptom:** `docker stack deploy` completes, but `docker service ls` or `docker service ps <service_name>` shows issues.
    *   **Solution:**
        *   Inspect service logs: `docker service logs <service_name>`.
        *   Check node health and resource availability on the Swarm nodes.
        *   Verify image names, secret names, config names, and volume mounts in your `docker-stack.yml`.
        *   Use `docker service inspect <service_name>` for detailed information.

*   **Issue: Secrets not available to a service.**
    *   **Symptom:** Application fails to read a secret (e.g., database password).
    *   **Solution:**
        *   Ensure the secret was created in the Swarm (`docker secret ls`).
        *   Verify the secret is correctly assigned to the service in `docker-stack.yml` and that the `target` path is where the application expects to find it.

### 7.4. General Tips

*   **Check Logs:** Logs are your best friend. Always check container logs (`docker-compose logs` or `docker service logs`) and application logs (e.g., `storage/logs/laravel.log`) for error messages.
*   **Read Documentation:** Refer to the official documentation for Docker, Docker Compose, Docker Swarm, Laravel, PostgreSQL, Redis, and any other specific tools or packages used.
*   **Isolate the Problem:** Try to determine which component is failing (e.g., a specific container, the network, the application code).
*   **Search Online:** Use error messages to search for solutions on platforms like Stack Overflow, GitHub issues, or relevant forums.

This list is not exhaustive but covers some of the more frequent hurdles.

## 8. Conclusion

This guide provides a comprehensive overview of setting up and deploying the Application Event Management SaaS platform. By following the steps for development and understanding the principles for production deployment, you should be able to manage and scale this application effectively.

Remember that each production environment can have unique requirements, so always adapt these guidelines to your specific needs and consult the official documentation for the technologies involved. Good luck!
