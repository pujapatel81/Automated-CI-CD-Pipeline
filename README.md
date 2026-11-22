# Automated CI/CD Pipeline using Jenkins, Docker & AWS EC2

## Project Overview
This project demonstrates an automated CI/CD pipeline using Jenkins, Docker, GitHub, Maven and AWS EC2 to build, test, containerize and deploy a sample Spring Boot application.

## project structure 
Automated-CI-CD-Pipeline/
│
├── src/
│   └── main/
│       ├── java/
│       │   └── com/devops/cicd/
│       │       └── CidcDemoApplication.java
│       │
│       └── resources/
│           ├── application.properties
│           └── static/
│               └── index.html
│
├── .gitignore
├── Dockerfile
├── Jenkinsfile
├── mvnw
├── mvnw.cmd
└── pom.xml

## 1. Launch AWS EC2 Instance

- AMI: Ubuntu Server 24.04 LTS
- Instance Type: Suitable instance with sufficient resources
- Create or select an SSH Key Pair.

### Security Group Inbound Rules

| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| Custom TCP | 8080 | My IP |
| HTTP | 80 | 0.0.0.0/0 |

Connect to the instance using SSH.

## 2. Install Java 21

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jdk -y
java -version
```

## 3. Install Git

```bash
sudo apt install git -y
git --version
```

## 4. Install Maven

```bash
sudo apt install maven -y
mvn -version
```

## 5. Install Docker

```bash
sudo apt update
sudo apt install ca-certificates curl -y

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo systemctl enable --now docker

sudo docker run hello-world
```

## 6. Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

Get the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open Jenkins:

```text
http://YOUR_EC2_PUBLIC_IP:8080
```

Complete Jenkins initial setup.

## 7. Configure Docker Permissions for Jenkins

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo -u jenkins docker version
```

## 8. Configure Docker Hub Credentials

1. Log in to Docker Hub.
2. Create a repository named `cicd-demo`.
3. Generate an Access Token with Read & Write permissions.
4. Open Jenkins → Manage Jenkins → Credentials.
5. Add the following credentials:

- Kind: Username with password
- Username: Docker Hub username
- Password: Docker Hub Access Token
- ID: `dockerhub-creds`

Never share or upload access tokens or passwords.

## 9. Configure Jenkins Pipeline

1. Open Jenkins Dashboard.
2. Click New Item.
3. Enter job name: `Automated-CI-CD-Pipeline`.
4. Select Pipeline.
5. Choose Pipeline script from SCM.
6. SCM: Git.
7. Enter the GitHub repository URL.
8. Branch: `*/main`.
9. Script Path: `Jenkinsfile`.
10. Click Save.

Click Build Now to execute the pipeline.

## 10. Access the Application

After successful deployment, open:

```text
http://YOUR_EC2_PUBLIC_IP
```

## CI/CD Pipeline Stages

1. Checkout – Fetch source code from GitHub.
2. Build – Build application using Maven.
3. Test – Run application tests.
4. Docker Build – Create Docker image.
5. Docker Push – Upload Docker image to Docker Hub.
6. Deploy – Deploy Docker container on EC2.

## Important Notes

- Replace `YOUR_EC2_PUBLIC_IP` with the new instance's public IP.
- Configure Docker Hub credentials again on a new Jenkins installation.
- Jenkins jobs, credentials and build history are not automatically restored after EC2 termination.
- GitHub source code, Dockerfile and Jenkinsfile remain available in the repository.
- Keep private keys, passwords and access tokens secure.
- Terminate unused EC2 instances to avoid unnecessary compute charges. Check separately for billable EBS volumes and other resources.
