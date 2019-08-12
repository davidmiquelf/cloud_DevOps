# Introduction
Deploying my trading-app project to AWS cloud, with and without Docker.
The app simulates an API for posting, browsing, buying, and selling market quotes.
It runs on EC2 micro instances that have access to a PostgreSQL RDS within the VPC.  

With Docker,  each EC2 instance pulls the docker images from Dockerhub or Github, then creates
and runs the containers. They originally ran psql locally, but in later iterations they simply connected
to the RDS.
![Docker architecture diagram](./docker_diagram.png)

There are two Docker files, one in the base folder of trading-app, and the other in the psql folder.
### trading-app
The app is dockerized by the following lines from `/dll/run_docker_app.sh`:
```
docker build -t trading-app .

docker run -d \
--restart unless-stopped \
-e "PSQL_URL=$PSQL_URL" \
-e "PSQL_USER=$PSQL_USER" \
-e "PSQL_PASSWORD=$PSQL_PASSWORD" \
-e "IEX_PUB_TOKEN=$IEX_PUB_TOKEN" \
--name trading-app \
--network trading-net \
-p 8080:8080 -t trading-app
```
This command creates a trading-app image from the Dockerfile, then runs the container with specific environment variables.
### jrvs-psql
The psql database is dockerized by similar lines in the script `/dll/run_docker_psql.sh`:  
```
cd ../psql

docker build -t jrvs-psql .

docker run --name jrvs-psql \
--restart unless-stopped \
-e "POSTGRES_PASSWORD=$PSQL_PASSWORD" \
-e POSTGRES_DB=jrvstrading \
-e "POSTGRES_USER=$PSQL_USER" \
--network trading-net \
-d -p 5432:5432 jrvs-psql
```
The Dockerfile tells the image to inject the commands in `/psql/sql_dll/schema.sql` after creating the jrvstrading database,
so the database is fully ready after being dockerized.
# Cloud Architecture Diagram
- trading app diagram
  - use draw.io and aws icons (it's in the draw.io library)
  - include ec2, alb, auto scaling, target group, rds
  - security groups
  - label all important ports(e.g. ALB HTTP, ec2 tpc:5000, RDS tcp:5432)
  
# AWS EB and Jenkins CI/CD Pipeline Diagram
- Please refer to Jenkins guide architecture diagram.
