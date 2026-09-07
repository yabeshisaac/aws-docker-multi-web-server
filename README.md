# AWS Docker Multi-Web Server Deployment

Hands-on containerization project deployed on Amazon EC2 using Docker, Apache HTTPD, Nginx, Docker Hub, and Amazon ECR.

The project demonstrates how multiple isolated web-server containers can run on a single EC2 instance, how custom container images can be created and versioned, and how those images can be published to both public and private registries for reuse.

## Architecture

```text
Users
  │
  ▼
Amazon EC2
Amazon Linux + Docker Engine
  │
  ├── httpd1 : 8081 → 80
  ├── httpd2 : 8082 → 80
  ├── httpd3 : 8083 → 80
  ├── nginx1 : 8084 → 80
  ├── nginx2 : 8085 → 80
  └── nginx3 : 8086 → 80
          │
          ▼
Configured Containers
          │
          │ docker commit
          ▼
Custom Docker Images
  ├── villa-apache:v1
  └── lugx-nginx:v1
          │
          ├──────────────► Docker Hub
          │
          └──────────────► Amazon ECR
                              │
                              ▼
                     Redeployed Container
                     ecr-villa-test :8091
```

## Project Overview

- Provisioned an Amazon Linux EC2 instance as the Docker host.
- Installed and configured Docker Engine.
- Deployed six independent web-server containers.
- Ran three Apache HTTPD containers and three Nginx containers.
- Used host-to-container port mapping so each server could be accessed independently.
- Deployed a Villa Agency website inside Apache.
- Deployed a LUGX Gaming website inside Nginx.
- Created reusable custom images from the configured containers.
- Tagged and pushed images to Docker Hub.
- Tagged and pushed images to Amazon ECR.
- Used an EC2 IAM role for ECR access instead of storing long-term AWS access keys.
- Redeployed the Villa image from Amazon ECR and verified it successfully.

## Tech Stack

| Component | Technology |
|---|---|
| Cloud Platform | AWS |
| Compute | Amazon EC2 |
| Operating System | Amazon Linux |
| Container Runtime | Docker |
| Web Servers | Apache HTTPD, Nginx |
| Public Registry | Docker Hub |
| Private Registry | Amazon ECR |
| Access Control | AWS IAM |
| Networking | EC2 Security Groups, Port Mapping |
| Region | ap-south-1 (Mumbai) |

## Container Deployment

Three Apache containers were deployed:

```bash
docker run -d --name httpd1 -p 8081:80 httpd
docker run -d --name httpd2 -p 8082:80 httpd
docker run -d --name httpd3 -p 8083:80 httpd
```

Three Nginx containers were deployed:

```bash
docker run -d --name nginx1 -p 8084:80 nginx
docker run -d --name nginx2 -p 8085:80 nginx
docker run -d --name nginx3 -p 8086:80 nginx
```

This allowed all six containers to run simultaneously on one EC2 instance using different host ports.

## Website Deployment

### Apache — Villa Agency

The Villa Agency website was deployed into the Apache document root:

```bash
docker exec httpd1 sh -c 'rm -rf /usr/local/apache2/htdocs/*'
docker cp villa/templatemo_591_villa_agency/. httpd1:/usr/local/apache2/htdocs/
```

Access:

```text
http://<EC2-PUBLIC-IP>:8081
```

### Nginx — LUGX Gaming

The LUGX Gaming website was deployed into the Nginx document root:

```bash
docker exec nginx1 sh -c 'rm -rf /usr/share/nginx/html/*'
docker cp gaming/templatemo_589_lugx_gaming/. nginx1:/usr/share/nginx/html/
```

Access:

```text
http://<EC2-PUBLIC-IP>:8084
```

## Custom Image Creation

For this learning lab, the configured containers were captured as reusable images using `docker commit`.

```bash
docker commit httpd1 villa-apache:v1
docker commit nginx1 lugx-nginx:v1
```

Created images:

```text
villa-apache:v1
lugx-nginx:v1
```

> Note: `docker commit` was used here as part of the learning workflow. In a production-style CI/CD process, building reproducible images from Dockerfiles is preferred.

## Docker Hub

The custom images were tagged and pushed to Docker Hub.

```bash
docker tag villa-apache:v1 yabesh007/villa-apache:v1
docker tag lugx-nginx:v1 yabesh007/lugx-nginx:v1

docker push yabesh007/villa-apache:v1
docker push yabesh007/lugx-nginx:v1
```

## Amazon ECR

Amazon ECR was used as the AWS-managed private container registry.

The EC2 instance used an IAM role for AWS access, avoiding long-term access keys on the server.

Authentication:

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
<ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com
```

Images were tagged and pushed to ECR:

```bash
docker tag villa-apache:v1 <ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/villa-apache:v1

docker tag lugx-nginx:v1 <ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/lugx-nginx:v1
```

## ECR Redeployment Test

To verify that the ECR image was reusable, a new container was launched from the registry image:

```bash
docker run -d \
  --name ecr-villa-test \
  -p 8091:80 \
  <ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/villa-apache:v1
```

The Villa Agency website loaded successfully on port `8091`, confirming the complete image lifecycle from container configuration to registry storage and redeployment.

## Request Flow

```text
Browser
   │
   ▼
EC2 Public IP : Host Port
   │
   ▼
EC2 Security Group
   │
   ▼
Docker Port Mapping
   │
   ▼
Container Port 80
   │
   ▼
Apache / Nginx
   │
   ▼
Website
```

## Project Structure

```text
aws-docker-multi-web-server/
│
├── README.md
├── commands.md
│
├── screenshots/
│   ├── architecture-diagram.png
│   ├── ec2-instance.png
│   ├── docker-containers.png
│   ├── villa-apache.png
│   ├── lugx-nginx.png
│   ├── docker-hub.png
│   ├── ecr-repositories.png
│   └── ecr-redeployment.png
│
└── docs/
    └── Docker-Multi-Web-Server-Deployment-AWS.pdf
```

## What I Learned

Through this project, I gained hands-on experience with:

- Running multiple containers on a single Linux host.
- Understanding Docker images and containers.
- Port mapping and container networking.
- Deploying applications inside Apache and Nginx containers.
- Creating and tagging reusable Docker images.
- Publishing container images to Docker Hub.
- Using Amazon ECR as a private container registry.
- Authenticating to ECR using AWS IAM and the AWS CLI.
- Redeploying a container from an ECR-hosted image.
- Managing Docker containers from the Linux command line.

## Production Improvements

This project was intentionally built as a hands-on learning lab.

A production-style deployment could be improved with:

- Dockerfiles instead of `docker commit`
- Automated CI/CD pipelines
- HTTPS
- Application Load Balancer or reverse proxy
- Container health checks
- Centralized logging
- Vulnerability scanning
- Amazon ECS or EKS for orchestration
- Infrastructure as Code using Terraform

## Key Concepts Demonstrated

`Amazon EC2` • `Docker` • `Apache HTTPD` • `Nginx` • `Amazon ECR` • `Docker Hub` • `AWS IAM` • `Linux` • `Containerization` • `Port Mapping` • `Image Tagging` • `Registry Authentication`

---

This project was built as a hands-on implementation to strengthen my practical understanding of Docker containerization, Linux administration, AWS EC2, Amazon ECR, and container image lifecycle management.
