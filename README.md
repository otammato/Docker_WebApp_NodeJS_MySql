  
# Docker_WebApp_NodeJS_AWS_RDS_MySql (the page is under development)

For this project, we'll be deploying a web app from another project (built using Node.JS and Express) located at https://github.com/otammato/WebApp_NodeJS_AWS_RDS_MySql.git. The previous deployment was done on EC2 and RDS instances.

This time, we're taking a different approach by using containerization and deploying the app with Docker. We'll launch two containers on an EC2 instance running AWS Cloud9 IDE. One container will host the app, and the other will host the MySQL database. After deployment, we'll store the Docker images permanently in the AWS Elastic Container Registry (ECR).

<br><br>
<p align="center" >
  <img width="700" alt="Screenshot 2023-02-01 at 20 11 38" src="https://user-images.githubusercontent.com/104728608/216415601-4f8b42e4-d7f6-4e0a-9274-16a062b7591d.png">
</p>
<br><br>

Launch your IDE (I am using AWS Cloud9), download the archive with project files and untar them in the home/ec2-user/environment
```
wget https://github.com/otammato/Docker_WebApp_NodeJS_AWS_RDS_MySql/blob/main/Docker_WebApp_NodeJS_AWS_RDS_MySql.tar.gz

tar -xzvf Docker_WebApp_NodeJS_AWS_RDS_MySql.tar.gz
```
<br><br>


Initiate git locally, make the first push to GitHub
```
git -v
git init
git commit -m "first commit"
git branch -M main
git remote add origin <https://github.com/.......place here your GitHub repository...................>
git push -u origin main
```

<br><br>
Make the new directory, move the app files there and create the Dockerfile
```
mkdir containers
cd containers
mkdir node_app
cd node_app
mv ~/environment/resources/codebase_partner ~/environment/containers/node_app
cd ~/environment/containers/node_app/codebase_partner
touch Dockerfile
```
<br><br>

This is a Dockerfile for a Node.js application based on the Alpine Linux image version 11. The Dockerfile performs the following steps:

- It specifies the base image as node:11-alpine.
- It creates a directory "/usr/src/app" in the image.
- It sets the current working directory to "/usr/src/app".
- It copies the contents of the current directory to "/usr/src/app" in the image.
- It runs the command "npm install" to install the dependencies of the Node.js application.
- It exposes port 3000.
- It sets the command to run when a container is created from this image as "npm run start".

```
FROM node:11-alpine
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "start"]
```
<br><br>


Push to GitHub
```
git add .
git commit -m "Docker file created"
git push -u origin

# to unstage the changes
# git reset

# to unstage partially
# git restore --staged ../../../Terraform_template/terraform.tfstate
# git restore --staged ../../../Terraform_template/terraform.tfstate.1675123997.backup
# git restore --staged ../../../Terraform_template/terraform.tfstate.backup

# this is to resolve the issue with pull requests if arise
git config pull.ff true
git pull
```

<br><br>
This is a directory tree in my Cloud9 after all the above id done:
<br>
<p align="center" >
  <img width="846" alt="Screenshot 2023-02-02 at 19 07 53" src="https://user-images.githubusercontent.com/104728608/216427012-f5721567-4be6-4a0c-8cb7-ecf44bbfcd8f.png">
</p>
<br><br>
<br><br>

Navigate to /containers/node_app and run these commands to create and run a Docker container for the Node.js application:

- docker build --tag node_app . builds a Docker image using the Dockerfile in the current directory with the tag "node_app".
- docker images lists all the Docker images present on the system.
- docker run -d --name node_app_1 -p 3000:3000 node_app runs a Docker container in the background with the name "node_app_1", maps host port 3000 to container port 3000 and uses the image "node_app".
- docker container ls lists all running containers.
- curl http://localhost:3000 sends a HTTP request to the URL "http://localhost:3000" and returns the response. This can be used to test if the Node.js application is running correctly inside the container.


```
docker build --tag node_app .
docker images
docker run -d --name node_app_1 -p 3000:3000 node_app
docker container ls
curl http://localhost:3000
```
<br><br>

Adjust the security group of the AWS EC2 instance to allow network traffic on port 3000 from your computer:

<br><br>
<p align="center" >
  <img width="657" alt="Screenshot 2023-02-02 at 21 52 29" src="https://user-images.githubusercontent.com/104728608/216458138-8c5325cf-87a9-4625-ab49-f5c4dc324daa.png">
</p>
<br><br>

Paste the <puplic_ip>:3000 in your browser (access the web interface of the application, which is now running in a container)
<br><br>
<br><br>

That is what we have done so far:
<br><br>
<p align="center" >
  <img width="700" alt="Screenshot 2023-02-02 at 10 46 14" src="https://user-images.githubusercontent.com/104728608/216304326-f0d44a4b-37b2-4056-b22b-e2f01d749260.png">
</p>
<br><br>

- You copied the code base into a directory, which acted as your build area [a]. You also created a Dockerfile that provided instructions for how to create a Docker image. That Dockerfile specified a FROM instruction that identified a starter image to use.
- You then ran the docker build command [b]. Docker read the Dockerfile and requested the starter image from an image repository [c]. The image repository returned the starter image file [d].
- The docker build command then finished building the image according to the instructions in the Dockerfile, which resulted in the Docker image [e].
- Finally, you ran the docker run command [f] to run a Docker container [g].
<p align="center" >  
  <img width="700" alt="Screenshot 2023-02-02 at 11 07 58" src="https://user-images.githubusercontent.com/104728608/216308903-d17880bb-4714-47d9-9070-267a6f02b6b8.png">
</p>
<br><br>


These commands will get you inside the container's shell to check your current env variables. You can place variables here or pass them with the CLI commands later. They are used later by the JS script to connect to the database.
```
docker ps
docker exec -ti 4e446ba88388  sh
whoami
su node
env
```
Here are the variables:
```
APP_DB_HOST=<paste here the output endpoint of the created RDS instance> 
APP_DB_USER=admin 
APP_DB_PASSWORD="<your password>" 
APP_DB_NAME=COFFEE
```

```
docker ps
docker stop node_app_1 && docker rm node_app_1
```
<br><br>

Passing the env variable via CLI:
```
docker run -d --name node_app_1 -p 3000:3000 -e APP_DB_HOST="<ip-address>" node_ap
```
<br><br>

Download the mysql backup file using mysqldump. In my case, I dumped the database from my AWS RDS instance.
```
mysqldump -P 3306 -h  <mysql-host-ip-address> -u nodeapp -p --databases COFFEE > ../../my_sql.sql
```

```
cd /home/ec2-user/environment/containers
mkdir mysql
cd mysql
```

```
touch Dockerfile
cp /home/ec2-user/environment/resources/my_sql.sql .

```

```
FROM mysql:8.0.23
COPY ./my_sql.sql /
EXPOSE 3306
```

```
docker rmi -f $(docker image ls -a -q)
sudo docker image prune -f && sudo docker container prune -f
```

```
docker build --tag mysql_server .
docker images
docker run --name mysql_1 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=rootpw -d mysql_server
docker container ls
docker exec -i mysql_1 mysql -u root -p12345678 < my_sql.sql 

docker exec -i mysql_1 mysql -u root  -p12345678 -e "CREATE USER 'nodeapp' IDENTIFIED WITH mysql_native_password BY '12345678'; GRANT all privileges on *.* to 'nodeapp'@'%';" 
```

```
docker inspect network bridge

#pass the discovered ip address of mysql_server
docker run -d --name node_app_1 -p 3001:3000 -e APP_DB_HOST=172.17.0.2 node_app
docker ps
```

```
aws ecr get-login-password \
--region us-east-1 | docker login --username AWS \
--password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

```
aws ecr create-repository --repository-name node-app

docker tag node_app:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/node-app:latest

docker images

docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/node-app:latest
```

```
aws ecr list-images --repository-name node-app

aws ecr batch-delete-image \
     --repository-name node-app \
     --image-ids imageTag=latest
     
aws ecr delete-repository --repository-name node-app
```
