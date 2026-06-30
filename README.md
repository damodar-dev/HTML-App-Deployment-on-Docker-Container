# HTML App Deployment on Docker Container

A static HTML/CSS/JS website ("Guarder" — a security services template) containerized with **Nginx** and **Docker**, with CI/CD pipeline configs for **AWS CodeBuild** to push images to **Amazon ECR** and deploy to **Amazon ECS**.

## Project Structure

```
.
├── index.html              # Home page
├── about.html               # About page
├── service.html              # Services page
├── contact.html             # Contact page
├── guard.html               # Guard/landing page
├── css/                      # Stylesheets (Bootstrap, custom, responsive)
├── js/                       # JavaScript (jQuery, Bootstrap, custom scripts)
├── images/                   # Image assets
├── fonts/                    # Web fonts (Font Awesome, custom)
├── Dockerfile                # Nginx-based image definition
├── buildspec.yml             # CodeBuild spec for building static site artifacts
└── buildspec (1).yml         # CodeBuild spec for building & pushing Docker image to ECR
```

## Tech Stack

- HTML5, CSS3 (Bootstrap), JavaScript (jQuery)
- Nginx (web server)
- Docker
- AWS CodeBuild, Amazon ECR, Amazon ECS (CI/CD)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed and running
- (Optional, for AWS pipeline) AWS CLI configured with access to ECR/ECS

## Getting Started Locally

### 1. Clone the repository

```bash
git clone https://github.com/damodar-dev/HTML-App-Deployment-on-Docker-Container.git
cd HTML-App-Deployment-on-Docker-Container
```

### 2. Build the Docker image

```bash
docker build -t html-app .
```

### 3. Run the container

```bash
docker run -d -p 8080:80 --name html-app-container html-app
```

### 4. View the site

Open your browser at [http://localhost:8080](http://localhost:8080)

### 5. Stop and remove the container

```bash
docker stop html-app-container
docker rm html-app-container
```

## Dockerfile Overview

The image is based on `nginx:latest` and simply copies all project files into Nginx's default web root, serving the static site on port 80:

```dockerfile
FROM nginx:latest
COPY . /usr/share/nginx/html/
EXPOSE 80
```

## CI/CD Pipeline (AWS)

This repo includes two AWS CodeBuild specs:

- **`buildspec.yml`** — packages the static HTML site as a build artifact (no Docker involved).
- **`buildspec (1).yml`** — builds the Docker image, tags it, authenticates with Amazon ECR, pushes the image, and generates an `imagedefinitions.json` file for use in an ECS deployment stage.

To use the ECR/ECS pipeline, update the following environment variables in `buildspec (1).yml` to match your own AWS account:

| Variable | Description |
|---|---|
| `AWS_ACCOUNT_ID` | 040591922284 |
| `REPOSITORY_NAME` | Damodararao |
| `CONTAINER_NAME` | The container name defined in your ECS task definition |
| `CLUSTER_NAME` | Your ECS cluster name |
| `AWS_DEFAULT_REGION` | AWS region (ap-south-1) |
| `IMAGE_TAG` | Docker image tag (latest) |

A typical pipeline flow: **CodePipeline → CodeBuild (build & push image to ECR) → ECS (deploy new task using `imagedefinitions.json`)**.

> Note: it's recommended to rename `buildspec (1).yml` to something like `buildspec-ecr.yml` (spaces and parentheses in filenames can cause issues in some CI tools), and to keep AWS account IDs and other identifiers out of version control by parameterizing them via CodeBuild environment variables instead of hardcoding.
