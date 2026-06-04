https://chatgpt.com/backend-api/estuary/content?id=55e97370edaca74%23file_00000000328c7208883eb8e0726a362b%23wm&ts=494599&p=fs&cid=1&sig=fcdb752f62d512fc95ffaf171f63f8ffd65dc095f54c6828fef3947cd2cd77d9&v=0
# NGINX Reverse Proxy, Load Balancing and Route 53 Integration on AWS

## Project Overview

This project demonstrates how to deploy a Flask application on multiple AWS EC2 instances and expose it through NGINX acting as a Reverse Proxy and Load Balancer. Route 53 is used to map a domain name to the NGINX server.

---

## Architecture

```text
                 Users
                    │
                    ▼
              Route 53 (DNS)
                    │
                    ▼
         NGINX Reverse Proxy / Load Balancer
               (13.232.212.96)
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
 Backend-1                Backend-2
13.203.74.236          13.233.117.184
Port 5000              Port 5000
```

---

## Scenario 1: Flask Application Deployment

### Objective
Deploy a Flask application on an EC2 instance and expose it on port 5000.

### Flask Application

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Flask Application Running"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### Run Application

```bash
python3 app.py
```

### Verify

```bash
curl http://localhost:5000
```

---

## Scenario 2: Running Flask in Background

Instead of running the application in the foreground, run it as a background process.

```bash
nohup python3 app.py > flask.log 2>&1 &
```

### Explanation

- nohup → Keeps process running after logout
- > flask.log → Redirects output to log file
- 2>&1 → Redirects error logs to the same file
- & → Runs process in background

### Verify

```bash
ps -ef | grep app.py
```

---

## Scenario 3: NGINX Reverse Proxy (Same Server)

### Architecture

```text
User
 │
 ▼
NGINX
 │
 ▼
Flask App (Port 5000)
```

### Configuration

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:5000;
    }
}
```

### Benefits

- Hide application port
- SSL termination
- Better security
- Centralized logging

---

## Scenario 4: Cross-Server Reverse Proxy

### Architecture

```text
User
 │
 ▼
NGINX Server
 │
 ▼
Backend Server
```

### Example

```text
NGINX EC2    : 13.232.212.96
Backend EC2  : 13.203.74.236
```

### Configuration

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://13.203.74.236:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Benefits

- Separate Web Tier and Application Tier
- Better scalability
- Easier maintenance
- Enhanced security

---

## Scenario 5: NGINX Load Balancing

### Architecture

```text
                 NGINX
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
 Backend-1                Backend-2
```

### Backend Servers

```text
Backend-1 : 13.203.74.236:5000
Backend-2 : 13.233.117.184:5000
```

### Configuration

```nginx
upstream backend_servers {
    server 13.203.74.236:5000;
    server 13.233.117.184:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend_servers;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Load Balancing Method

Default algorithm:

```text
Round Robin
```

### Request Distribution

```text
Request 1 → Backend-1
Request 2 → Backend-2
Request 3 → Backend-1
Request 4 → Backend-2
```

---

## Scenario 6: Failover Testing

### Objective

Validate High Availability.

### Test

Stop Backend-1.

```bash
kill <PID>
```

### Architecture

```text
                 NGINX
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
 Backend-1 ❌             Backend-2 ✅
```

### Expected Result

Traffic should automatically route to Backend-2.

### Validation

```bash
curl http://13.232.212.96
```

---

## Scenario 7: Troubleshooting

### Welcome to NGINX Page

#### Cause

Default NGINX configuration serving requests.

#### Fix

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Verify the correct configuration file is loaded.

---

### 502 Bad Gateway

#### Cause

NGINX unable to reach backend.

#### Verification

```bash
curl http://13.203.74.236:5000
curl http://13.233.117.184:5000
```

#### Common Reasons

- Application not running
- Security Group blocking port 5000
- Incorrect proxy_pass configuration

---

## Scenario 8: Route 53 DNS Integration

### Architecture

```text
User
 │
 ▼
myapp.com
 │
 ▼
Route 53
 │
 ▼
NGINX
 │
 ▼
Backend Servers
```

### Route 53 Record

```text
Record Type : A
Domain      : myapp.com
Value       : 13.232.212.96
```

### NGINX Configuration

```nginx
server {
    listen 80;
    server_name myapp.com www.myapp.com;

    location / {
        proxy_pass http://backend_servers;
    }
}
```

### Flow

```text
myapp.com
     │
     ▼
 Route 53
     │
     ▼
 NGINX
     │
     ▼
 Backend Servers
```

---

## Key Concepts Learned

### Reverse Proxy

Receives client requests and forwards them to backend servers.

```text
Client → NGINX → Application
```

### Load Balancer

Distributes traffic across multiple backend servers.

```text
Client → NGINX → Multiple Backends
```

### High Availability

Ensures application remains accessible even if one backend fails.

```text
Backend-1 Down
      │
      ▼
Traffic Redirected to Backend-2
```

### Route 53

AWS DNS service that maps domain names to IP addresses.

```text
myapp.com → 13.232.212.96
```

---

## Commands Used

### Validate NGINX Configuration

```bash
sudo nginx -t
```

### Reload NGINX

```bash
sudo systemctl reload nginx
```

### Restart NGINX

```bash
sudo systemctl restart nginx
```

### Verify Backend

```bash
curl http://13.203.74.236:5000
curl http://13.233.117.184:5000
```

### Check Listening Ports

```bash
sudo ss -tulnp
```

### View Logs

```bash
tail -f flask.log
```

---

## Conclusion

Successfully implemented:

- Flask Application Deployment on EC2
- Running Applications in Background using nohup
- NGINX Reverse Proxy
- Cross-Server Reverse Proxy
- NGINX Load Balancing
- Failover and High Availability Testing
- Route 53 DNS Integration
- Troubleshooting NGINX and Backend Connectivity

This project demonstrates practical implementation of Reverse Proxy, Load Balancing, High Availability, and DNS routing concepts commonly used in real-world DevOps, Cloud, and SRE environments.
