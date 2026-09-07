# Command Reference

This file contains the main commands used in the AWS Docker Multi-Web Server project.

> AWS account-specific values are represented using placeholders such as `<AWS_ACCOUNT_ID>` and `<EC2-PUBLIC-IP>`.

## 1. Install Docker on Amazon Linux

```bash
sudo yum update -y
sudo yum install -y docker

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker ec2-user
newgrp docker

docker --version
```

## 2. Pull Apache and Nginx Images

```bash
docker pull httpd
docker pull nginx

docker images
```

## 3. Launch Apache Containers

```bash
docker run -d --name httpd1 -p 8081:80 httpd
docker run -d --name httpd2 -p 8082:80 httpd
docker run -d --name httpd3 -p 8083:80 httpd
```

## 4. Launch Nginx Containers

```bash
docker run -d --name nginx1 -p 8084:80 nginx
docker run -d --name nginx2 -p 8085:80 nginx
docker run -d --name nginx3 -p 8086:80 nginx
```

Verify:

```bash
docker ps
docker ps -a
```

## 5. Download Website Templates

```bash
sudo yum install -y wget unzip

cd ~

wget -O villa.zip "https://templatemo.com/download/templatemo_591_villa_agency"
wget -O gaming.zip "https://templatemo.com/download/templatemo_589_lugx_gaming"

mkdir villa gaming

unzip villa.zip -d villa
unzip gaming.zip -d gaming

find villa -name index.html
find gaming -name index.html
```

## 6. Deploy Villa Website to Apache

```bash
docker exec httpd1 sh -c 'rm -rf /usr/local/apache2/htdocs/*'

docker cp villa/templatemo_591_villa_agency/. \
httpd1:/usr/local/apache2/htdocs/
```

Access:

```text
http://<EC2-PUBLIC-IP>:8081
```

## 7. Deploy LUGX Gaming Website to Nginx

```bash
docker exec nginx1 sh -c 'rm -rf /usr/share/nginx/html/*'

docker cp gaming/templatemo_589_lugx_gaming/. \
nginx1:/usr/share/nginx/html/
```

Access:

```text
http://<EC2-PUBLIC-IP>:8084
```

## 8. Create Custom Docker Images

For this learning project, the configured containers were captured using `docker commit`.

```bash
docker commit httpd1 villa-apache:v1
docker commit nginx1 lugx-nginx:v1

docker images
```

## 9. Tag Images for Docker Hub

```bash
docker tag villa-apache:v1 yabesh007/villa-apache:v1

docker tag lugx-nginx:v1 yabesh007/lugx-nginx:v1
```

## 10. Push Images to Docker Hub

```bash
docker login

docker push yabesh007/villa-apache:v1
docker push yabesh007/lugx-nginx:v1
```

## 11. Verify AWS IAM Identity

```bash
aws sts get-caller-identity
```

## 12. Authenticate Docker with Amazon ECR

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login \
--username AWS \
--password-stdin \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

The EC2 instance used an IAM role for AWS access rather than storing long-term AWS access keys.

## 13. Tag Images for Amazon ECR

```bash
docker tag villa-apache:v1 \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/villa-apache:v1
```

```bash
docker tag lugx-nginx:v1 \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/lugx-nginx:v1
```

## 14. Push Images to Amazon ECR

```bash
docker push \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/villa-apache:v1
```

```bash
docker push \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/lugx-nginx:v1
```

## 15. Redeploy Villa Image from ECR

```bash
docker run -d \
--name ecr-villa-test \
-p 8091:80 \
<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/villa-apache:v1
```

Verify:

```bash
docker ps
```

Access:

```text
http://<EC2-PUBLIC-IP>:8091
```

## 16. Configure Container Restart Policies

```bash
docker update --restart unless-stopped httpd1
docker update --restart unless-stopped httpd2
docker update --restart unless-stopped httpd3

docker update --restart unless-stopped nginx1
docker update --restart unless-stopped nginx2
docker update --restart unless-stopped nginx3
```

## Useful Docker Commands

```bash
# Running containers
docker ps

# All containers
docker ps -a

# Local images
docker images

# Container logs
docker logs <CONTAINER_NAME>

# Stop container
docker stop <CONTAINER_NAME>

# Start container
docker start <CONTAINER_NAME>

# Remove container
docker rm <CONTAINER_NAME>

# Remove image
docker rmi <IMAGE_NAME>
```

---

## Production Note

This project used `docker cp` and `docker commit` as part of the hands-on learning workflow.

For a production environment, a more reproducible workflow would use:

```text
Dockerfile
   ↓
docker build
   ↓
Versioned Image
   ↓
Amazon ECR
   ↓
Automated CI/CD Deployment
```

Dockerfile-based image builds and CI/CD automation are planned as separate improvements/projects.
