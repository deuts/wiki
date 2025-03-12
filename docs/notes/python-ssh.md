---
title: Python Docker Image with SSH and MkDocs

parent: Notes
---

# Setting Up a Docker Image Based on python:3.13.2-slim-bookworm

## Overview
This documentation provides a step-by-step guide on setting up a Docker image based on `python:3.13.2-slim-bookworm`. The setup includes:
- Enabling SSH
- Installing essential packages
- Creating a user (`deuts`)
- Configuring SSH host keys and passwordless SSH
- Setting up passwordless sudo
- Implementing an entrypoint script to start services
- Configuring GitHub SSH access

## Dockerfile
Below is the `Dockerfile` for setting up the base image:

```dockerfile
FROM python:3.13.2-slim-bookworm

# Install OpenSSH and other necessary dependencies
RUN apt-get update && \
    apt-get install -y openssh-server sudo nano htop curl wget zip unzip tree git vim && \
    rm -rf /var/lib/apt/lists/*

# Create the user 'deuts' with UID 1000 and GID 1000
RUN groupadd -g 1000 deuts && \
    useradd -m -u 1000 -g 1000 -s /bin/bash deuts

# Ensure SSH folder exists
RUN mkdir -p /run/sshd /etc/ssh

# Create a system-wide profile script to add /home/deuts/.local/bin to the PATH
# This ensures that any executables installed in this directory are accessible globally
RUN echo 'export PATH="/home/deuts/.local/bin:$PATH"' > /etc/profile.d/deuts_path.sh

# Enable passwordless sudo by ensuring correct file permissions
RUN chmod 0440 /etc/sudoers.d/deuts

# Copy entrypoint script
COPY config/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]

# Expose SSH and MkDocs ports
EXPOSE 22 8000

# Start SSH daemon
CMD ["/usr/sbin/sshd", "-D"]
```

## Entrypoint Script
The `entrypoint.sh` script ensures SSH is running, adds SSH keys if available, and starts MkDocs.

```bash
#!/bin/bash

# Start SSH agent
eval "$(ssh-agent -s)"

# Add the SSH key (if it exists)
if [ -f ~/.ssh/id_ed25519 ]; then
  ssh-add ~/.ssh/id_ed25519
elif [ -f ~/.ssh/id_rsa ]; then
  ssh-add ~/.ssh/id_rsa
else
  echo "No SSH key found in ~/.ssh"
fi

# Start MkDocs in the background
echo "Starting MkDocs..."
cd /home/deuts/mkdocs/cashier-app || exit 1
/home/deuts/.local/bin/mkdocs serve -a 0.0.0.0:8000 &

# Start SSH daemon
/usr/sbin/sshd -D
```

## SSH Configuration
### Generating SSH Host Keys
To ensure SSH functions correctly, generate the host keys before starting the container. The `-A` option ensures all default host key types are created, and the `-f` option allows specifying the output directory.

```bash
mkdir -p config/ssh_host_keys
ssh-keygen -A -f config/ssh_host_keys
```

By using `ssh-keygen -A -f config/ssh_host_keys`, all required SSH host keys are generated inside the `config/ssh_host_keys` directory before building the Docker image. This ensures that when the container starts, it already has the necessary keys to enable SSH access.

### Configuring Passwordless SSH
To enable passwordless SSH for `deuts`, place the public key in `authorized_keys`.

```bash
mkdir -p data/.ssh
chmod 700 data/.ssh
touch data/.ssh/authorized_keys
chmod 600 data/.ssh/authorized_keys
```

## Enabling Passwordless Sudo
To allow the `deuts` user to run commands as `root` without a password, create the following file on the host system:

```bash
echo "deuts ALL=(ALL) NOPASSWD:ALL" | sudo tee config/sudoers.d/deuts
```

Ensure it has the correct permissions:

```bash
chmod 0440 config/sudoers.d/deuts
```

This file is mounted into the container at `/etc/sudoers.d/deuts`, ensuring `deuts` has passwordless sudo access.

## Configuring GitHub SSH Access
To allow the container to interact with private GitHub repositories via SSH, follow these steps:

1. **Generate an SSH Key (if not already created)**:
   ```bash
   ssh-keygen -t ed25519 -C "your-email@example.com"
   ```
   Store the key inside `~/.ssh/`.

2. **Copy the Public Key to GitHub**:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Add the output to your GitHub account under **Settings > SSH and GPG keys**.

3. **Test the Connection**:
   ```bash
   ssh -T git@github.com
   ```
   If successful, you should see a message like:
   `Hi username! You've successfully authenticated, but GitHub does not provide shell access.`

## Compose File
Use the following `compose.yml` to build and run the container:

```yaml
services:
  app:
    build: .
    container_name: pythrax
    hostname: pythrax
    restart: unless-stopped
    environment:
      - TZ=Asia/Manila
    volumes:
      - ./data:/home/deuts
      - ./config/ssh_host_keys:/etc/ssh
      - ./config/sudoers.d:/etc/sudoers.d
      - /home/deuts/apps/php-server/app:/php-server
    user: "1000:1000"
    networks:
      - caddynetwork

networks:
  caddynetwork:
    external: true
```

## Running the Container
1. **Build the Image**:
   ```bash
   docker compose build
   ```
2. **Start the Container**:
   ```bash
   docker compose up -d
   ```

## Verifying Setup
- Check if the container is running:
  ```bash
  docker ps
  ```
- Connect via SSH:
  ```bash
  ssh -p 2222 deuts@localhost
  ```
- Ensure MkDocs is running:
  ```bash
  curl http://localhost:8000
  ```
- Verify GitHub SSH authentication:
  ```bash
  ssh -T git@github.com
  ```

This setup provides a robust development environment based on Python with SSH access, necessary utilities, GitHub SSH authentication, and MkDocs serving in the background.

