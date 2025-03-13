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

---

## More explanation

### **Full JSON Example**
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

---

### **Breaking It Down Line by Line**

#### **1. The JSON Array**
```json
[
```
- The entire configuration is an **array** because `webhook` allows multiple hooks.
- Each webhook entry is an object inside this array.

#### **2. The Hook Object**
```json
{
```
- This marks the start of a **webhook definition**.

#### **3. The `id` Field**
```json
"id": "run-script",
```
- `"run-script"` is the **unique identifier** for this webhook.
- When you send a request to `http://localhost:9000/hooks/run-script`, it triggers this hook.

#### **4. The `execute-command` Field**
```json
"execute-command": "/home/deuts/apps/webhook/scripts/myscript.sh",
```
- This is the **script or command** that will run when the webhook is triggered.
- Here, it executes `/home/deuts/apps/webhook/scripts/myscript.sh`.

#### **5. The `command-working-directory` Field**
```json
"command-working-directory": "/home/deuts/apps/webhook/scripts",
```
- This sets the **working directory** before running the script.
- Useful if the script relies on relative paths.

#### **6. The `response-message` Field**
```json
"response-message": "Script executed successfully!",
```
- This is the message that gets sent back in the HTTP response after execution.
- If the webhook is triggered successfully, you’ll see this message in `curl` or any HTTP client.

#### **7. The `trigger-rule` Object**
```json
"trigger-rule": {
```
- This section **controls when the webhook executes**.
- If the request doesn’t meet these conditions, it won’t trigger.

#### **8. The `match` Object**
```json
"match": {
```
- This defines the **condition** for triggering the webhook.

#### **9. The `type` Field**
```json
"type": "value",
```
- `"value"` means we’re checking if a specific **parameter** matches a given **value**.

#### **10. The `value` Field**
```json
"value": "secret123",
```
- The expected **value** for the parameter.
- If the parameter **doesn’t match** `"secret123"`, the webhook **won’t execute**.

#### **11. The `parameter` Object**
```json
"parameter": {
```
- Specifies where the webhook should **look for the value**.

#### **12. The `source` Field**
```json
"source": "url",
```
- The value comes from the **URL query string**.
- Example request:  
  ```
  http://localhost:9000/hooks/run-script?token=secret123
  ```

#### **13. The `name` Field**
```json
"name": "token"
```
- `"token"` is the **name** of the URL parameter.
- In this case, we’re checking if `token=secret123`.

#### **14. Closing Brackets**
```json
}
}
}
]
```
- These **close** the JSON structure properly.

---

### **How It Works in Action**
1. A request is made to:
   ```sh
   curl "http://localhost:9000/hooks/run-script?token=secret123"
   ```
2. `webhook`:
   - Extracts the **token** from the URL.
   - Checks if it **matches** `"secret123"`.
   - If it does, it **executes**:
     ```sh
     /home/deuts/apps/webhook/scripts/myscript.sh
     ```
   - Responds with:
     ```
     Script executed successfully!
     ```
3. If the `token` is **incorrect or missing**, `webhook` **does nothing**.

---

### **Want to Add Another Hook?**
Just add another object inside the array:
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
  },
  {
    "id": "deploy-app",
    "execute-command": "/home/deuts/apps/webhook/scripts/deploy.sh",
    "command-working-directory": "/home/deuts/apps/webhook/scripts",
    "response-message": "Deployment started!",
    "trigger-rule": {
      "match": {
        "type": "value",
        "value": "deployme",
        "parameter": {
          "source": "url",
          "name": "token"
        }
      }
    }
  }
]
```
Now:
```sh
curl "http://localhost:9000/hooks/deploy-app?token=deployme"
```
Will trigger `deploy.sh`.

---

### **Final Notes**
- Every webhook **must have a unique `id`**.
- Always **restart** `webhook` after editing `hooks.json`:
  ```sh
  sudo systemctl restart webhook
  ```
- If something isn’t working, check logs:
  ```sh
  journalctl -u webhook -n 50 --no-pager
  ```
