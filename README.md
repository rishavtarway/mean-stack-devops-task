# Dev Ops Engineer Intern Assignment: MEAN Stack CRUD Application

<img width="1680" height="470" alt="Screenshot 2025-11-29 at 8 11 06 AM" src="https://github.com/user-attachments/assets/c3cb968b-c67d-4772-9f81-627888d55663" />

## Step-by-Step Setup and Deployment Instructions

### I. Local Code and Container Preparation

#### 1\. Code Adjustments (Initial Setup)

Before building, modified the local references in the source code:

1.  **Backend Config:** In `backend/app/config/db.config.js`, changed the MongoDB URL:
      * Changed `localhost` to `mongodb`: `url: "mongodb://mongodb:27017/dd_db"`
2.  **Frontend Service:** In `frontend/src/app/services/tutorial.service.ts`, changed the `baseUrl` to a relative path:
      * Changed `http://localhost:8080/api/tutorials` to `/api/tutorials`.

#### 2\. Build and Push Images (Multi-Architecture)

Due to the difference between my Mac (ARM64) and GCP (AMD64) architectures, images must be built explicitly for Linux/AMD64 same as my GCP VM:

1.  **Build Backend:** (Ran inside the `backend/` folder)
    ```bash
    docker build --platform linux/amd64 -t rishavtarway/mean-backend:latest .
    docker push rishavtarway/mean-backend:latest
    ```
2.  **Build Frontend:** (Ran inside the `frontend/` folder)
    ```bash
    docker build --platform linux/amd64 -t rishavtarway/mean-frontend:latest .
    docker push rishavtarway/mean-frontend:latest
    ```

    <img width="1680" height="800" alt="Screenshot 2025-11-29 at 8 08 24 AM" src="https://github.com/user-attachments/assets/4407f68a-c9e7-443f-89db-d5529b04da88" />


-----

### II. Manual Deployment (GCP VM)

This process sets up the initial environment and reverse proxy.

1.  **SSH into VM:** Connected to my GCP VM using the browser SSH tool or  generated SSH key.
2.  **Install Tools:** Installed Docker, Docker Compose, and Nginx:
    ```bash
    sudo apt update
    sudo apt install docker.io docker-compose nginx -y
    sudo usermod -aG docker $USER # Then reconnect SSH
    ```
3.  **Placed Files:** Ensure the `docker-compose.yml` file is placed in the home directory of the VM (`/home/rishavtarway/`).
4.  **Initial Deployment (Run Containers):**
    ```bash
    docker-compose up -d
    ```
5.  **Configure Nginx Reverse Proxy:** Edited the default configuration to route traffic on Port 80 to the containers:
    ```bash
    sudo nano /etc/nginx/sites-available/default
    # Configure /api to http://localhost:8080 and / to http://localhost:81
    sudo systemctl restart nginx
    ```

-----

## Nginx Setup and Infrastructure Details

The application is hosted on a **Google Cloud Platform (GCP) Compute Engine VM** running **Ubuntu 22.04 LTS**. **Nginx** is configured as a reverse proxy on the host machine to manage public traffic on Port 80.

### 1\. Infrastructure Summary

| Component | Detail | Purpose |
| :--- | :--- | :--- |
| **Instance Name** | `mean-app-vm` | Hosting the entire Docker Compose stack. |
| **External IP** | `35.200.189.253` (Example) | The public endpoint for the application. |
| **Firewall Rule** | `default-allow-http` (TCP:80) | Allows all inbound HTTP traffic from the internet (`0.0.0.0/0`). |

### 2\. Nginx Reverse Proxy Configuration

Nginx listens on host Port 80 and routes traffic to the containers running on the Docker network:

| Inbound Path | Action | Destination Container Port |
| :--- | :--- | :--- |
| `/api` | Proxy Pass | `http://localhost:8080` (Backend API) |
| `/` (Root) | Proxy Pass | `http://localhost:81` (Frontend Nginx) |

 
 ## A. Verify Nginx Service Status 

  ```bash
 sudo systemctl status nginx
 ```
 
<img width="1680" height="453" alt="Screenshot 2025-11-29 at 8 16 14 AM" src="https://github.com/user-attachments/assets/ff28c2b4-741e-4760-8645-8d5074165e2d" />

## B. Verify Nginx to Frontend Connection 


```bash
curl http://localhost:81
```

<img width="1673" height="325" alt="Screenshot 2025-11-29 at 8 20 16 AM" src="https://github.com/user-attachments/assets/40ab9809-9897-4d91-b178-c4dbddf29262" />




### III. CI/CD Pipeline (GitHub Actions)

The final step was setting up the automation pipeline.

1.  **Configure Secrets:** Set the following secrets in my GitHub repository (**Settings** \> **Secrets** \> **Actions**):

      * `DOCKER_USERNAME` (e.g., `rishavtarway`)
      * `DOCKER_PASSWORD` (Use a Docker Hub Access Token)
      * `HOST_DNS` (GCP VM External IP)
      * `USERNAME` (VM SSH Username: `rishavtarway`)
      * `EC2_SSH_KEY` (The private key content  generated on my VM).

2.  **Push Workflow:** Commit and pushed the `deploy.yml` file to my repository's `main` branch.

    *The pipeline automatically runs, builds the new images, and restarts the containers on my GCP VM, completing the full CI/CD cycle.*

**Proof:** The final pipeline run completed successfully, confirming the automation loop is functional and secure.

---
<img width="1671" height="598" alt="Screenshot 2025-11-29 at 8 06 18 AM" src="https://github.com/user-attachments/assets/cf1aec89-d5ba-4a4b-84fb-a3c87161ac7b" />


