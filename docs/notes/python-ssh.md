---
title: Python, SSH and MkDocs
layout: default
nav_order: 100
parent: Notes
---

# Setting Up a Docker Image Based on python:3.13.2-slim-bookworm
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>


## Overview
This documentation provides a step-by-step guide on setting up a Docker image based on `python:3.13.2-slim-bookworm`. The setup includes:
- Enabling SSH
- Installing essential packages
- Creating a container user (`cont_user`)
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

# Create the user 'cont_user' with UID 1000 and GID 1000
RUN groupadd -g 1000 cont_user && \
    useradd -m -u 1000 -g 1000 -s /bin/bash cont_user

# Ensure SSH folder exists
RUN mkdir -p /run/sshd /etc/ssh

# Create a system-wide profile script to add /home/cont_user/.local/bin to the PATH
# This ensures that any executables installed in this directory are accessible globally
RUN echo 'export PATH="/home/cont_user/.local/bin:$PATH"' > /etc/profile.d/cont_user_path.sh

# Enable passwordless sudo by ensuring correct file permissions
RUN chmod 0440 /etc/sudoers.d/cont_user

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
cd /home/cont_user/mkdocs/cashier-app || exit 1
/home/cont_user/.local/bin/mkdocs serve -a 0.0.0.0:8000 &

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

### Create `sshd_config` file
Under `config/ssh_host_keys` directory, create `sshd_config` file with the following content:
```sh
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
```

### Configuring Passwordless SSH
To enable passwordless SSH for `cont_user`, place the public key in `authorized_keys`.

```bash
mkdir -p data/.ssh
chmod 700 data/.ssh
touch data/.ssh/authorized_keys
chmod 600 data/.ssh/authorized_keys
```

## Enabling Passwordless Sudo
To allow the `cont_user` user to run commands as `root` without a password, create the following file on the host system:

```bash
echo "cont_user ALL=(ALL) NOPASSWD:ALL" | sudo tee config/sudoers.d/cont_user
```

Ensure it has the correct permissions:

```bash
chmod 0440 config/sudoers.d/cont_user
```

This file is mounted into the container at `/etc/sudoers.d/cont_user`, ensuring `cont_user` has passwordless sudo access.

Important: `/etc/sudoers.d` should be owned by root!

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
      - ./data:/home/cont_user
      - ./config/ssh_host_keys:/etc/ssh
      - ./config/sudoers.d:/etc/sudoers.d
      - /home/cont_user/apps/php-server/app:/php-server
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
  ssh -p 2222 cont_user@localhost
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

## Adding a webhook server
The webhook server proves valuable if you want to execute python scripts based on command.

### Add webhook in Dockerfile
```Dockerfile
# Install OpenSSH and other necessary dependencies
RUN apt-get update && \
    apt-get install -y openssh-server sudo nano htop curl wget zip unzip tree git vim webhook && \
    rm -rf /var/lib/apt/lists/*
```
### Add port 9000 in the EXPOSE command in Dockerfile
```Dockerfile
# Expose SSH port
EXPOSE 22 8000 9000
```

### Start the service in `entrypoint.sh`
Add this line before the `exec "$@"` line:
```sh
# Start the webhook service
echo "Starting Webhook server..."
/usr/bin/webhook -hooks /home/cont_user/webhook/hooks.json -verbose &
```

### Prepare the webhook files

From the host: 

```sh
mkdir -p data/webhook
touch data/webhook/webhook.log
```

And make sure to maintain the following files and directories structure under `data/webhook` directory:
```sh
.
├── hooks.json
├── scripts
│   └── myscript.sh
└── webhook.log
```

The `hooks.json` can look something like this:
```json
[
  {
    "id": "run-script",
    "execute-command": "/home/cont_user/webhook/scripts/myscript.sh",
    "command-working-directory": "/home/cont_user/webhook/scripts",
    "include-command-output-in-response": true,
    "pass-environment-to-command": [],
    "response-headers": [
      {
        "name": "Content-Type",
        "value": "application/json"
      }
    ],
    "status-success": 200,
    "status-failure": 500,
    "trigger-rule": {
      "match": {
        "type": "value",
        "value": "secret123",
        "parameter": {
          "source": "url",
          "name": "token"
        }
      }
    }
  }
]
```

The `myscript.sh` is the script you want to run when this webhook is pinged, which could be something like this:
```bash
#!/bin/bash

# Log the webhook trigger
echo "Webhook triggered at $(date)" >> /home/deuts/webhook/webhook.log

# Capture the output and exit status properly - redirect stderr to a variable
HUGO_OUTPUT=$(hugo version 2>&1)
STATUS=$?

# Log the command output for debugging
echo "Hugo output: $HUGO_OUTPUT" >> /home/deuts/webhook/webhook.log
echo "Exit status: $STATUS" >> /home/deuts/webhook/webhook.log

# Return the appropriate response based on the exit status
if [ $STATUS -eq 0 ]; then
  echo '{"status": "success", "message": "Script executed successfully!"}'
  exit 0
else
  # This will run when hugo command fails (like when it's not found)
  echo '{"status": "error", "message": "Script failed to execute!"}'
  exit 1
fi
```

Now when you start your container, your webhook server will be listening to port 9000 of the container, which you can reverse proxy using your favorite application.

### Editing `hooks.json`
When you edit the hooks.json and/or adding more hooks, you need to restart the webhook server. You can restart this Docker container, or alternatively, run the following command while inside the container:

```sh
pkill webhook && nohup /usr/bin/webhook -hooks /home/deuts/webhook/hooks.json -verbose > /dev/null 2>&1 &
```

It is unfortunate that this can't be put into a bash script, it always throw a `Terminated` error.