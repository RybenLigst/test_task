# test_task
test task "На ВМ Proxmox розвернути front+back на контейнерах docker і отримати SSL для фронтового nginx. front+back повинні бути описані сервісами і запущені в docker swarm"

# Docker Swarm: Nginx (SSL) + Backend Load Balancing

## Architecture

The project consists of two services:
1. **Frontend:** Nginx (Alpine), which accepts HTTPS requests on port 443 and proxies them to the backend.
2. **Backend:** `traefik/whoami` (run in 2 replicas) to demonstrate load balancing.

### Technical features of the solution
Due to the specifics of the deployment environment (Nested Virtualization / LXC limitations), I did this:
* **Host Mode Networking:** Frontend ports were published in `mode: host` mode to avoid problems with Ingress Routing Mesh.
* **DNS Round Robin (`dnsrr`):** Used for Service Discovery of the backend, which allows Nginx to obtain direct IP addresses of containers, bypassing virtual IPs.

## How to start

### 1. Initialize Swarm
`docker swarm init`

### 2. Generating self-signed SSL cert
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx.key -out nginx.crt \
  -subj "/C=UA/ST=Kyiv/L=Kyiv/O=DevOps/OU=IT/CN=localhost"
```

### 3. Creating secrets and config

```
docker secret create site_crt nginx.crt
docker secret create site_key nginx.key
docker config create nginx_conf nginx.conf
```
4. Starting
```
docker stack deploy -c docker-stack.yml my_app
```

The screenshot shows that the load balancer is working.


<img width="499" height="927" alt="image" src="https://github.com/user-attachments/assets/3510bb53-2b23-49d2-b894-7cc3645a8e35" />
