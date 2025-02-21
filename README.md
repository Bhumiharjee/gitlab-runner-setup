# GitLab Runner Setup Guide

This guide walks through the complete process of setting up GitLab Runner with Docker on Ubuntu.

## 1. Install Docker

Follow the official Docker installation guide for Ubuntu: https://docs.docker.com/engine/install/ubuntu/

## 2. Install GitLab Runner

Follow the official GitLab Runner installation guide: https://docs.gitlab.com/runner/install/linux-repository/

## 3. Configure GitLab Runner

Register your runner with GitLab:
```bash
sudo gitlab-runner register
```

You will need:
- GitLab instance URL
- Registration token
- Runner description
- Tags (optional)
- Executor type (select 'docker')

## 4. Set Required Permissions

Run these commands to set up proper permissions:
```bash
# Add gitlab-runner user to docker group
sudo usermod -aG docker gitlab-runner

# Change ownership of the Docker socket
sudo chown root:docker /var/run/docker.sock

# Set proper permissions on Docker socket
sudo chmod 666 /var/run/docker.sock

# Restart Docker service
sudo systemctl restart docker

# Restart GitLab Runner service
sudo systemctl restart gitlab-runner
```

## 5. Configure Docker Daemon

1. Create/edit the Docker daemon configuration file:
```bash
sudo nano /etc/docker/daemon.json
```

2. Add the following configuration:
```json
{
  "insecure-registries": ["34.93.164.72:8082"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

3. Verify the JSON syntax:
```bash
sudo cat /etc/docker/daemon.json | jq '.'
```

4. Apply the changes:
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Troubleshooting

If you encounter issues:

1. Check Docker service status:
```bash
sudo systemctl status docker
```

2. Check GitLab Runner status:
```bash
sudo systemctl status gitlab-runner
```

3. Verify permissions:
```bash
ls -l /var/run/docker.sock
```

4. Check GitLab Runner logs:
```bash
sudo gitlab-runner debug
```
