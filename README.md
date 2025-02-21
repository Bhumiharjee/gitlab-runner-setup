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

## 5. Configure GitLab Runner TOML File

Edit the GitLab Runner configuration file:
```bash
sudo nano /etc/gitlab-runner/config.toml
```

Replace the content with:
```toml
concurrent = 1
check_interval = 0
shutdown_timeout = 0
[session_server]
  session_timeout = 1800
[[runners]]
  name = "tabson-fastapi-deploy-qa-new"
  url = "https://gitlab.sauravkumar.cloud"
  id = 40
  token = "glrt-t3_3nNn2gnq4Bs1nP8mFgqr"
  token_obtained_at = 2025-02-21T11:53:12Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:20.10.16"
    privileged = true
    disable_entrypoint_overwrite = false
    user = "root"
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/etc/docker/daemon.json:/etc/docker/daemon.json"]
    allowed_images = ["docker:*"]
    allowed_services = ["docker:*-dind"]
    shm_size = 0
    network_mtu = 0
[[runners]]
  name = "build-publish-image"
  url = "https://gitlab.sauravkumar.cloud"
  id = 26
  token = "glrt-t3_jeHQbcubpmhUCwe5fzU_"
  token_obtained_at = 2025-01-04T16:53:11Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "python3.8"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    network_mtu = 0
```

After modifying the configuration:
```bash
sudo gitlab-runner restart
```

## 6. Configure Docker Daemon

1. Create/edit the Docker daemon configuration file:
```bash
sudo nano /etc/docker/daemon.json
```

2. Add the following configuration:
```json
{
  "insecure-registries": ["registory-ip:registory-port"],
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

5. Verify the TOML configuration:
```bash
sudo gitlab-runner verify
```
