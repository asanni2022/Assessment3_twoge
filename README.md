## Deploy Python Twoge Web App to Elastic Beanstalk using GitHub Actions


# Deploy Twoge Achitectural Diagram
![image](https://github.com/asanni2022/Assessment3_twoge/assets/104282577/dfc98273-0550-4852-ae5a-2ee051eeae04)

```
https://excalidraw.com/#json=aqTRWEXdCivW1McpdN_g3,mfMd66Eml9RZ_pD0XNfiig

```
## Step 1. Create Directory and Clone Repo
```
  mkdir eb-assgmnt-twoge && cd eb-assgmnt-twoge/
  git clone https://github.com/asanni2022/twoge
  cd twoge/
```
### Set GitHub Token Access
```
  DOCKER_USERNAME
  DOCKER_PASSWORD
  AWS_ACCESS_KEY_ID
  AWS_SECRET_ACCESS_KEY
```

## Step 2. Dockerize project root Directory
```
  From alpine:latest
  workdir /app
  COPY . /app

  RUN apk update && apk upgrade
  RUN apk add python3
  RUN apk add py3-pip
  RUN python -m venv .venv
  RUN source .venv/bin/activate
  RUN pip install -r requirements.txt

  EXPOSE 80

  CMD ["python", "app.py"]
```
### Build Docker Image locally
```
 docker build -t twogeassmt3 .
 docker images
 ```
### Create Database Container
```
  docker run -itd -p 5432:5432 --name postgres-twoge-cont -e POSTGRES_USER=twoge -e POSTGRES_DB=twoge_db -e POSTGRES_PASSWORD=postgres123 postgres
  export SQLALCHEMY_DATABASE_URI=postgresql://twoge:postgres123@localhost:5432/twoge_db
  python3 -m venv venv
  source venv/bin/activate
  pip install -r requirements.txt
  gunicorn app:app -c gunicorn_config.py
```

## Step 3: Implementing CI/CD with GitHub Actions
```
mkdir -p .github/workflows/ && cd workflows
touch docker-push-cicd.yml
```

### yml
```
name: Deploy Static game to EB using twoge Dockerhub

on: push

env:
#   DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
#   DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  AWS_REGION: 'us-east-2'

jobs:
  build_and_push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          ref: master

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/cpassgmnt-twoge-sda-hackathon:latest

  eb_deploy:
    needs: build_and_push
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        # with:
        #   ref: master

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.x

      - name: Install EB CLI
        run: |
          pip install awsebcli --upgrade

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Init Elastic Beanstalk
        run: |
          echo n | eb init 

      - name: Deploy to Elastic Beanstalk
        run: |
          eb deploy
```
## Commit Code
```
  git add .
  git commit -m "code updated"
  git push origin master
```

### Step 4. Setting up elastic beanstalk
```
  eb init
```
### .elasticbeanstalk/ config.yml
```

branch-defaults:
  master:
    environment: twogeassmnt111
global:
  application_name: twoge
  branch: null
  default_ec2_keyname: null
  default_platform: Docker running on 64bit Amazon Linux 2
  default_region: us-east-2
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: null
  repository: null
  sc: git
  workspace_type: Application

```

### Create Environment
```
  eb create twogeassmnt111 --single
```
### Set Database Environment Variables
```
 eb setenv DATABASE_URL=postgresql://postgres:passwords@twogedb.cg8xzkizfi6t.us-east-2.rds.amazonaws.com:5432/twoge_db
```

### Build and Push
```
  git add .
  git commit -m "code updated"
  git push origin master
```

## Step 5.  Configuring Environment Variables and AWS RDS
 ### Set Up RDS Postgres DB
```
POSTGRES_USER=
POSTGRES_DB=
POSTGRES_PORT=
POSTGRES_PASSWORD=
SQLALCHEMY_DATABASE_URI=SQLALCHEMY_DATABASE_URI=postgresql://postgres:passwords@postgresdb.chzveui56egk.us-east-1.rds.amazonaws.com:5432/twoge_db
```

## Step 6: Running project using docker compose.
### Remove all prebuild Images
```
  docker rmi $(docker images)
```
### Stop all prebuild Containers
```
  docker stop $(docker ps -aq)
```
### Remove all prebuild Containers
```
  docker rm $(docker ps -aq)
```
### Create a docker-compose yml file
```
version: '3.8'

services:
  web:
    build: .
    command: gunicorn --bind 0.0.0.0:9876 app:app
    ports:
      - 9876:8080
    env_file:
      - ./.env
    depends_on:
      - database
    volumes:
      - .:/usr/share/nginx/html/

  database:
    image: postgres:13
    container_name: postgres_twoge_c
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.db
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "${POSTGRES_DB}", "-U", "${POSTGRES_USER}"]
      interval: 30s
      timeout: 45s
      retries: 5
      start_period: 30s

volumes:
  postgres_data:
```
### Build Image and Create Containers
```
docker compose up --build
```
### Verify
```
  docker ps
  http://localhost/
```




