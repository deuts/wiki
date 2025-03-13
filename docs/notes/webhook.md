---
title: Lightweight Webhook Server
parent: Notes
---
# **Installing and Configuring `webhook` on a Debian-Based System**
Note: we are installing this in the host instead of Docker because I want to execute bash scripts on the host. 

## **1. Install `webhook`**
Update your package list and install `webhook` using `apt`:  

```sh
sudo apt-get update
sudo apt-get install webhook -y
```

Verify installation:  

```sh
webhook --version
```

## Enable systemd service
```sh
sudo systemctl enable webhook
```

## **2. Create a Webhook Configuration File**
The webhook configuration (`hooks.json`) defines the endpoints and their corresponding scripts.  

Create the directory and file:  
```sh
sudo mkdir -p /etc/webhook
sudo nano /etc/webhook/hooks.json
```

Add the following example configuration:  
```json
[
  {
    "id": "run-script",
    "execute-command": "/etc/webhook/scripts/myscript.sh",
    "command-working-directory": "/etc/webhook/scripts",
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
This configuration:
- Triggers `myscript.sh` when accessing `/hooks/run-script`  
- Requires a `token=secret123` in the URL  

## **3. Create the Script to Execute**
Now, create the script that will be executed:  

```sh
sudo mkdir -p /etc/webhook/scripts
sudo nano /etc/webhook/scripts/myscript.sh
```

Add the following content:  
```bash
#!/bin/bash
echo "Webhook triggered at $(date)" >> /etc/webhook/webhook.log
```

Make it executable:  
```sh
sudo chmod +x /etc/webhook/scripts/myscript.sh
```

## **4. Create a Systemd Service**
To run `webhook` as a background service, create a systemd service file:  

```sh
sudo nano /etc/systemd/system/webhook.service
```

Paste the following configuration:  
```ini
[Unit]
Description=Lightweight Webhook Server
After=network.target

[Service]
ExecStart=/usr/bin/webhook -hooks /etc/webhook/hooks.json -verbose
Restart=unless-stopped
User=root
WorkingDirectory=/etc/webhook
Environment="PATH=/usr/local/bin:/usr/bin:/bin"

[Install]
WantedBy=multi-user.target
```

## **5. Start and Enable the Webhook Service**
Reload systemd and start the webhook service:  

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now webhook
```

Check if it’s running:  
```sh
sudo systemctl status webhook
```

## **6. Test the Webhook**
Run the following command to trigger the webhook:  
```sh
curl "http://localhost:9000/hooks/run-script?token=secret123"
```

Check the log file to confirm the script executed:  
```sh
cat /etc/webhook/webhook.log
```

---

## **Optional: Configure Webhook to Run on a Different Port**
By default, `webhook` runs on port **9000**. To change this, modify the systemd service:  

```sh
sudo nano /etc/systemd/system/webhook.service
```
Modify the `ExecStart` line to specify a different port (e.g., 8080):  
```ini
ExecStart=/usr/bin/webhook -hooks /etc/webhook/hooks.json -port 8080 -verbose
```

Reload systemd and restart `webhook`:  
```sh
sudo systemctl daemon-reload
sudo systemctl restart webhook
```

Now, it will be accessible at:  
```sh
curl "http://localhost:8080/hooks/run-script?token=secret123"
```
