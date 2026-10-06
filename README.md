# Flask Docker Jenkins CI/CD Pipeline

A containerized Flask web application with an automated CI/CD pipeline built using **Jenkins** and **Docker**. The app is a responsive landing page for a fictional off-road bike event ("Off-Road Riders"). Each Jenkins build pulls the latest code from GitHub, builds a fresh Docker image, and replaces the running container.

## Architecture

```text
Developer → GitHub → Jenkins → Docker Build → Docker Run → Flask App (port 5000)
```

1. Code is pushed to GitHub.
2. Jenkins pulls the latest source code.
3. **Build stage:** the old container and image are removed, then a new image `flask-app` is built.
4. **Run stage:** a new container `flask-container` is started and exposed on port 5000.

## Tech Stack

| Tool            | Purpose                                |
| --------------- | -------------------------------------- |
| Python / Flask  | Web application (HTML and CSS served from `app.py`) |
| Docker          | Containerization                       |
| Jenkins         | CI/CD automation                       |
| Git / GitHub    | Version control and source management  |
| AWS EC2 (Linux) | Hosting environment                    |

## Project Structure

```text
flask-docker-jenkins-cicd/
├── app.py          # Flask application (serves the event landing page)
├── Dockerfile      # Docker image definition
├── Jenkinsfile     # Jenkins pipeline definition
└── README.md
```

## Dockerfile Overview

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py /app
RUN pip install flask
EXPOSE 5000
CMD ["python3", "app.py"]
```

The image uses a lightweight Python 3.11 base, installs Flask, and runs the app on port 5000.

## Run Locally

**Prerequisite:** Docker installed.

```bash
git clone https://github.com/<your-username>/flask-docker-jenkins-cicd.git
cd flask-docker-jenkins-cicd

docker build -t flask-app .
docker run -d -p 5000:5000 --name flask-container flask-app
```

Open **http://localhost:5000** in your browser to view the page.

To stop and remove the container:

```bash
docker stop flask-container && docker rm flask-container
```

> The page loads its images from Unsplash, so an internet connection is required to display them.

## Jenkins Pipeline

The pipeline is defined in the `Jenkinsfile` and has two stages:

| Stage     | Description |
| --------- | ----------- |
| **Build** | Stops and removes any existing `flask-container`, removes the old `flask-app:latest` image, then builds a new image from the Dockerfile |
| **Run**   | Starts a new `flask-container` from `flask-app:latest`, mapping port 5000 |

### Jenkins Setup

1. Install Docker and Jenkins on a Linux server (for example, AWS EC2).
2. The pipeline runs `sudo docker ...`, so allow the `jenkins` user to run Docker without a password prompt:
```bash
   echo "jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker" | sudo tee /etc/sudoers.d/jenkins
```
3. Create a new **Pipeline** job in Jenkins.
4. Under **Pipeline**, select **Pipeline script from SCM**, choose **Git**, and enter this repository's URL.
5. Set the script path to `Jenkinsfile` and click **Build Now**.
6. Open port **5000** in the EC2 security group.

## Deployment

The pipeline was tested on an AWS EC2 instance running Jenkins and Docker. The instance has since been terminated to avoid costs,
so there is no live URL. When deployed, the app is served at `http://<EC2-PUBLIC-IP>:5000`.

## Future Improvements

- GitHub webhooks to trigger builds automatically on push
- Automated tests in the pipeline
- Push images to Docker Hub or another container registry
- Move the HTML into Flask templates and static files
- Nginx reverse proxy with HTTPS
- Monitoring and centralized logging

## Author

**Angeline Samritha** – AWS & DevOps Learner
[GitHub](https://github.com/<Angelinesamritha1>) 
