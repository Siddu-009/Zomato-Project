Complete GitHub Actions DevSecOps CI/CD Pipeline (Instead of Jenkins)

You can do the entire workflow using GitHub Actions without installing Jenkins.

Your pipeline will perform:

Code checkout SonarQube scan npm install OWASP dependency scan Trivy scan Docker build Docker push Deploy container Architecture Flow GitHub Push → GitHub Actions Workflow → SonarQube Scan → Dependency Check → Trivy Scan → Docker Build → DockerHub Push → Deploy Container STEP-1: Launch EC2 Instance

Launch Amazon Web Services EC2:

Setting Value OS Ubuntu 24.04 Instance Type t2.large Storage 30GB Security Group 22, 80, 3000 STEP-2: Install Docker & Trivy

Connect to EC2.

Install Docker sudo apt update -y

sudo apt install docker.io -y

sudo systemctl start docker

sudo systemctl enable docker

sudo chmod 777 /var/run/docker.sock

Verify:

docker --version Install Trivy wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz

tar zxvf trivy_0.18.3_Linux-64bit.tar.gz

sudo mv trivy /usr/local/bin/

echo "export PATH=$PATH:/usr/local/bin/" >> ~/.bashrc

source ~/.bashrc

Verify:

trivy --version STEP-3: Install SonarQube Using Docker docker run -d
--name sonarqube
-p 9000:9000
sonarqube:lts-community

Access:

http://:9000

Default login:

admin admin STEP-4: Generate SonarQube Token

Go to:

My Account → Security → Generate Token

Copy token.

STEP-5: Create DockerHub Access Token

Go to Docker Hub:

Account Settings → Security → Access Tokens

Create token.

STEP-6: Add GitHub Secrets

Go to your GitHub Repository:

Settings → Secrets and Variables → Actions → New Repository Secret

Add:

Secret Name Value SONAR_TOKEN SonarQube token SONAR_HOST_URL http://:9000 DOCKER_USERNAME DockerHub username DOCKER_PASSWORD DockerHub token STEP-7: Create GitHub Actions Workflow

Inside project:

.github/workflows/main.yml Final GitHub Actions Pipeline name: DevSecOps Pipeline

on: push: branches: - master

jobs:

build:

runs-on: ubuntu-latest

steps:

- name: Checkout Code
  uses: actions/checkout@v4

- name: Setup NodeJS
  uses: actions/setup-node@v4
  with:
    node-version: 18

- name: Install Dependencies
  run: npm install

- name: Install Sonar Scanner
  run: npm install -g sonarqube-scanner

- name: SonarQube Scan
  uses: SonarSource/sonarqube-scan-action@v6
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
  with:
    args: >
      -Dsonar.projectKey=zomato
      -Dsonar.projectName=zomato

- name: Install Trivy
  run: |
    wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
    tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
    sudo mv trivy /usr/local/bin/

- name: Trivy File Scan
  run: trivy fs .

- name: Build Docker Image
  run: docker build -t image1 .

- name: Tag Docker Image
  run: docker tag image1 siddu009/mydockerprojectfor5pm:myzomatoimage

- name: Docker Login
  run: |
    echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

- name: Push Docker Image
  run: docker push siddu009/mydockerprojectfor5pm:myzomatoimage

- name: Trivy Image Scan
  run: trivy image siddu009/mydockerprojectfor5pm:myzomatoimageSTEP-8: Push Code to GitHub
STEP-9: Watch Pipeline

Go to:

GitHub Repository → Actions Tab

Pipeline starts automatically.

STEP-10: Pull & Run Docker Image on EC2

docker pull siddu009/mydockerprojectfor5pm:myzomatoimage
docker run -d -p 3000:3000 --name zomato
siddu009/mydockerprojectfor5pm:myzomatoimage
Access Application http://:3000
