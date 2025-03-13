---
title: Lightweight Webhook Server
parent: Notes
---
# **Installing and Configuring `webhook` on a Debian-Based System**
Note: we are installing this in the host instead of Docker because I want to execute bash scripts on the host. 

## **1. Install `webhook`**
```sh
sudo apt-get update
sudo apt-get install webhook -y
```

## **2. Create Webhook Configuration in `/home/deuts/apps`**
```sh
mkdir -p /home/deuts/apps/webhook
nano /home/deuts/apps/webhook/hooks.json
```

Paste this configuration:  
```json
[
  {
    "id": "run-script",
    "execute-command": "/home/deuts/apps/webhook/scripts/myscript.sh",
    "command-working-directory": "/home/deuts/apps/webhook/scripts",
    "response-message": "Script executed successfully!",
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

## **3. Create the Script**
```sh
mkdir -p /home/deuts/apps/webhook/scripts
nano /home/deuts/apps/webhook/scripts/myscript.sh
```

Add:  
```bash
#!/bin/bash
echo "Webhook triggered at $(date)" >> /home/deuts/apps/webhook/webhook.log
```

Make it executable:  
```sh
chmod +x /home/deuts/apps/webhook/scripts/myscript.sh
```

## **4. Create a Systemd Service**
```sh
sudo nano /etc/systemd/system/webhook.service
```

Paste:  
```ini
[Unit]
Description=Lightweight Webhook Server
After=network.target

[Service]
ExecStart=/usr/bin/webhook -hooks /home/deuts/apps/webhook/hooks.json -verbose
Restart=unless-stopped
User=deuts
WorkingDirectory=/home/deuts/apps/webhook
Environment="PATH=/usr/local/bin:/usr/bin:/bin"

[Install]
WantedBy=multi-user.target
```

## **5. Start and Enable Webhook**
```sh
sudo systemctl daemon-reload
sudo systemctl enable --now webhook
```

## **6. Test the Webhook**
```sh
curl "http://localhost:9000/hooks/run-script?token=secret123"
cat /home/deuts/apps/webhook/webhook.log
```

