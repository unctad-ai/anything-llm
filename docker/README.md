# Deployment Guide

This document outlines the process for deploying the AnythingLLM application using Docker Compose and managing it with a `systemd` service for resilience.

## Docker Compose Configuration

Docker Compose uses a base file and an override file to create a flexible and environment-specific configuration.

- `docker-compose.yml`: This is the main configuration file. It defines the core services, volumes, and initial network. It is intended to be generic and should not be modified for specific environments.

- `docker-compose.override.yml`: This file is used to customize the base configuration for a specific deployment (e.g., production). Docker Compose automatically merges this file with the base file. Key overrides include:

  - **Networking**: It disconnects the service from the default `anything-llm` bridge network and connects it to a pre-existing external network (`deploy_default`). This is ideal for integrating the service into a larger infrastructure.
  - **Ports**: It maps port `3333` on the host to the container's port, which is necessary for services like the health check and external access.
  - **Labels**: It adds Traefik labels for reverse proxying, enabling features like automatic SSL and host-based routing.

- `docker-healthcheck.sh`: A simple script that checks if the application's API is responsive. It is used to monitor the health of the container.

## Systemd Service for Resilience

To ensure the application runs continuously, we use a `systemd` service file (`anything-llm.service`) to manage the Docker Compose stack.

### Prerequisites

1.  The project files have been copied to `/opt/anything-llm` on the target Ubuntu server.
2.  Docker and Docker Compose are installed on the server.

### Setup Instructions

Follow these steps on your Ubuntu server to set up the service.

#### 1. Determine the Correct User

The service must run as the user who owns the project files. To find the owner, run:

```bash
ls -ld /opt/anything-llm
```

The output will show the user and group (e.g., `ansible ansible`). The `anything-llm.service` file has been updated to use this `ansible` user.

#### 2. Add User to the `docker` Group

The user running the service must have permission to interact with the Docker daemon. Add the user to the `docker` group:

```bash
sudo usermod -aG docker ansible
```

#### 3. Install the Service File

Copy the `anything-llm.service` file to the `systemd` directory on your server:

```bash
sudo cp /opt/anything-llm/docker/anything-llm.service /etc/systemd/system/anything-llm.service
```

#### 4. Reload `systemd` and Start the Service

After copying the file, reload the `systemd` daemon to recognize the new service, then enable and start it.

```bash
# Reload the systemd configuration
sudo systemctl daemon-reload

# Enable the service to start on boot
sudo systemctl enable anything-llm.service

# Start the service immediately
sudo systemctl start anything-llm.service
```

#### 5. Check the Service Status

You can verify that the service is running correctly with the following command:

```bash
sudo systemctl status anything-llm.service
```

To view the live logs from the service, use:

```bash
journalctl -u anything-llm.service -f
```
