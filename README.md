# 🚀 Managing Kong API Gateway Clusters and Instances

This section will:
- Start the stack and verify Kong is reachable
- Create a **Service + Route** on `kong1` (and confirm it appears on `kong2` because they share the same DB)
- Send traffic through **NGINX** (port `9000`)
- Stop one Kong node and confirm traffic still continues

## 📁 Prerequisites

Make sure you have installed:

- Docker
- Docker Compose
- Git

---

You can verify your Docker installation with:
```bash
docker --version

docker compose version
```

## ✅ Getting Started
### 1.Create a Docker Network

Create a dedicated network for Kong:
```bash
docker network create kong-net
```

To check existing Docker networks, run:
```bash
docker network ls
```

You should see kong-net listed.

---

### 2.Start Kong Gateway (DB Mode) + HA Validation (2 Kong Nodes + NGINX)

Start all services using Docker Compose:
```bash
docker compose up -d
```

Verify Kong / NGINX are up

```bash
# verify
curl -i http://localhost:9000/status
curl -s http://localhost:8001 | head
```

**Create a Service + Route on kong1**

Create service httpbin:
```bash
# create service -> httpbin
curl -s -X POST http://localhost:8001/services \
  --data name=httpbin \
  --data url=http://httpbin:80
```
Expose a route /demo:
```bash
# expose route
curl -s -X POST http://localhost:8001/services/httpbin/routes \
  --data 'paths[]=/demo'
```

**Validate on BOTH Kong nodes**
```bash
curl -s http://localhost:8001/services/httpbin | jq .name
curl -s http://localhost:8011/services/httpbin | jq .name
```
Expected: both commands should return "httpbin"

**Send traffic through NGINX (port 9000)**
```bash
curl -i http://localhost:9000/demo/get
```
Run a quick loop:
```bash
for i in {1..10}; do curl -s http://localhost:9000/demo/uuid; done
```
**Kill one node and confirm traffic continues**
```bash
docker compose stop kong1

# traffic should still work
curl -i http://localhost:9000/demo/get

# bring it back
docker compose start kong1
```
If NGINX is load balancing across kong1 and kong2, traffic should continue working even when one node is down.

---
## 🧹 3.Cleanup (Stop and Remove Containers)
When you are done, you can stop and remove all running containers:
```bash
docker compose down
```

If you also want to remove the custom network:
```bash
docker network rm kong-net
```