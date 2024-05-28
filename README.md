# My Secure Dynamic Resume

This project showcases my skills in web development, cloud computing, DevOps, containerization, orchestration, and cybersecurity through a dynamic, secure resume hosted on AWS and Kubernetes.

## Features

- **Responsive Design**: Built with HTML, CSS, and JavaScript.
- **Containerization**: Application containerized using Docker with security best practices.
- **Kubernetes Orchestration**: Deployed on a secure Kubernetes cluster.
- **Infrastructure as Code**: Managed using Terraform with secure configurations.
- **Continuous Deployment**: Automated with GitHub Actions, including security checks.
- **Dynamic Content**: Powered by Flask and Firebase Firestore.
- **Cybersecurity Best Practices**: Implemented secure authentication, vulnerability scanning, logging, and monitoring.

## Cybersecurity Highlights

### Secure Coding Practices

- **Input Validation**: All user inputs are validated to prevent injection attacks.
- **Security Headers**: Implemented headers such as Content-Security-Policy (CSP), X-Content-Type-Options, and X-Frame-Options.
- **Authentication**: Secure authentication using JWT and Firebase Authentication.

### Container Security

- **Docker Bench for Security**: Used to ensure Docker containers follow security best practices.
- **Clair**: Integrated for scanning Docker images for vulnerabilities.

### Kubernetes Security

- **RBAC**: Role-Based Access Control to limit permissions within the cluster.
- **Network Policies**: Used to control traffic between pods, enhancing security.
- **Pod Security Policies**: Enforced security standards on Kubernetes pods.

### Infrastructure as Code (IaC) Security

- **Terraform**: Managed infrastructure as code, ensuring consistent and secure deployments.
- **Security Configurations**: Applied best practices for AWS S3, CloudFront, and other services.

### CI/CD Security

- **OWASP ZAP**: Integrated into the CI/CD pipeline for automated security testing.
- **Snyk**: Used for dependency scanning to identify and fix vulnerabilities.

### Monitoring and Incident Response

- **Prometheus and Grafana**: Set up for monitoring the health and performance of the application and infrastructure.
- **ELK Stack**: Used for centralized logging and alerting on suspicious activities.

## Getting Started

### Prerequisites

- Docker installed
- Kubernetes cluster (Minikube for local development or EKS/GKE/AKS for cloud)
- Terraform installed
- AWS account with appropriate permissions
- Firebase account

### Deployment

1. **Clone the repository**:
    ```bash
    git clone https://github.com/yourusername/my-secure-dynamic-resume.git
    cd my-secure-dynamic-resume
    ```

2. **Build Docker image**:
    ```bash
    docker build -t my-flask-app .
    ```

3. **Run Docker container**:
    ```bash
    docker run -p 4000:80 my-flask-app
    ```

4. **Deploy to Kubernetes**:
    - Apply Kubernetes manifests:
        ```bash
        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml
        ```

    - Or use Helm:
        ```bash
        helm install flask-app ./flask-app
        ```

5. **Set up Firebase**:
    Configure Firebase Firestore for dynamic content.

6. **Access the Application**:
    Access your resume at the provided external IP after deployment.

## Usage

- Edit `index.html`, `styles.css`, and `scripts.js` to update content and styles.
- Modify `app.py` for backend logic and API endpoints.

## Contributing

Feel free to fork this repository and submit pull requests to improve the project.

## License

This project is licensed under the MIT License.
