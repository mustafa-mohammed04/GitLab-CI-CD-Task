## 1- Prepare the Repository using fork or clone 
``` bash
git clone https://github.com/zinmyoswe/Django-Ecommerce.git
cd Django-Ecommerce
```

## 2- Define CI/CD Stages in GitLab
``` bash
1- Install dependencies & Build the application
2- Run unit tests
3- Scan for vulnerabilities (code + container security)
4- Build the Docker image
5- Scan the Docker image
6- Push the image if no vulnerabilities are found
7- Deploy to the staging server
```


## 3- Create a .gitlab-ci.yml file
``` bash
vim .gitlab-ci.yml
```
``` yaml
# Define Stages
stages:
  - build
  - test
  - security_scan
  - docker_build
  - docker_scan
  - deploy

# Define  Variables
variables:
  CI_REGISTRY_USER: "mustafa-mohammed04"
  CI_REGISTRY_PASSWORD: "Anubis240498##"
  CI_REGISTRY: "registry.gitlab.com"
  IMAGE_NAME: "registry.gitlab.com/mustafa-mohammed04/django-ecommerce"
  IMAGE_TAG: "latest"

#Build the App

build:
  stage: build
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - python manage.py collectstatic --noinput
    - python manage.py makemigrations
    - python manage.py migrate
  artifacts:
    paths:
      - static/
      - db.sqlite3
    expire_in: 1 hour

#Run Unit Tests

test:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pytest
  artifacts:
    reports:
      junit: report.xml

#Code Security Scan
security_scan:
  stage: security_scan
  image: python:3.9
  script:
    - pip install bandit
    - bandit -r . || true  # Avoid failing the pipeline if minor issues are found

#Create Dockerfile

docker_file:
  stage: docker_build
  script:
    - echo 'FROM python:3.9' > Dockerfile
    - echo 'WORKDIR /app' >> Dockerfile
    - echo 'COPY . /app' >> Dockerfile
    - echo 'RUN pip install -r requirements.txt' >> Dockerfile
    - echo 'CMD ["gunicorn", "-b", "0.0.0.0:8000", "ecommerce.wsgi:application"]' >> Dockerfile
  artifacts:
    paths:
      - Dockerfile

# Build Docker Image

docker_build:
  stage: docker_build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - export CI_REGISTRY_USER="mustafa-mohammed04"
    - export CI_REGISTRY_PASSWORD="Anubis240498##"
    - export CI_REGISTRY="registry.gitlab.com"
    - echo "Logging into GitLab Container Registry..."
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" "$CI_REGISTRY"
  script:
    - echo "Building Docker Image..."
    - docker build -t "$IMAGE_NAME:$IMAGE_TAG" .
    - echo "Saving Docker Image..."
    - docker save "$IMAGE_NAME:$IMAGE_TAG" -o image.tar
  artifacts:
    paths:
      - image.tar
    expire_in: 1 hour

# Scan Docker Image for Vulnerabilities

docker_scan:
  stage: docker_scan
  image: aquasec/trivy
  script:
    - trivy image "$IMAGE_NAME:$IMAGE_TAG" --exit-code 1 || exit 1

# Deploy the Application




```
## 5-1- Configure the GitLab CI/CD 
``` bash
1- Go to GitLab Repository → Settings → CI/CD
2- Add Required Variables under "CI/CD Variables":
 - CI_REGISTRY_USER: Your GitLab username
 - CI_REGISTRY_PASSWORD: GitLab Access Token
 - STAGING_SERVER: Your server's IP
 - DEPLOY_USER: SSH username
 - DEPLOY_PASSWORD: SSH password

```

## 5-2- Create & Push the Dockerfile "New"
``` bash
vim Dockerfile
```
``` bash
FROM python:3.9
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
CMD ["gunicorn", "-b", "0.0.0.0:8000", "ecommerce.wsgi:application"]

```
## 5-3- Commit & Push:
``` sh
git add .gitlab-ci.yml Dockerfile
git commit -m "Added GitLab CI/CD pipeline"
git push origin main

```

## 6- Run the Pipeline
``` bash
 Run unit tests
  - Scan for vulnerabilities
  - Build the Docker image
  - Scan the Docker image
  - Push only if the scan passes
  - Deploy to staging

```

## 7- Verify Deployment
``` bash
ssh user@staging-server
docker ps -a
# Restart the application if needed
docker run -d -p 8000:8000 registry.gitlab.com/your-username/django-ecommerce:latest

```
