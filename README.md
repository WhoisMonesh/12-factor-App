# 12 Factor App: Detailed Guide with GitHub Implementation

The **12 Factor App** methodology defines best practices for building modern, scalable, and maintainable software-as-a-service (SaaS) applications. This guide explores each factor in detail, with practical examples using **GitHub** tools.

---

## 1. **Codebase**  
**One codebase tracked in revision control, many deploys**  
- **Description**: A single repository for all environments (production, staging, etc.).  
- **Implementation**:  
  - Use Git for version control.  
  - Avoid multiple repos for the same app (e.g., split via microservices).  
- **GitHub Example**:  
  - One repository per application.  
  - Use branches (e.g., `main`, `develop`) and environments (GitHub Environments) for deployments.  
- **Reference**:  
  - [GitHub Repositories](https://docs.github.com/en/repositories)  
  - [12factor.net Codebase](https://12factor.net/codebase)

---

## 2. **Dependencies**  
**Explicitly declare and isolate dependencies**  
- **Description**: Never rely on implicit system-wide packages.  
- **Implementation**:  
  - Use dependency manifests (e.g., `package.json`, `requirements.txt`).  
  - Isolate environments with tools like `venv` (Python) or `bundler` (Ruby).  
- **GitHub Example**:  
  - Use GitHub Actions to install dependencies in CI/CD:  
    ```yaml
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: 20.x
      - run: npm ci
    ```  
  - Scan for vulnerabilities with [Dependabot](https://docs.github.com/en/code-security/dependabot).  
- **Reference**:  
  - [12factor.net Dependencies](https://12factor.net/dependencies)

---

## 3. **Config**  
**Store config in the environment**  
- **Description**: Keep configuration (e.g., API keys, database URLs) separate from code.  
- **Implementation**:  
  - Use environment variables, **not** hardcoded values.  
  - Avoid config files (e.g., `config.yml`) in the repo.  
- **GitHub Example**:  
  - Store secrets in **GitHub Environment Secrets** or **Repository Secrets**.  
  - Access secrets in GitHub Actions:  
    ```yaml
    env:
      DB_URL: ${{ secrets.DATABASE_URL }}
    ```  
- **Reference**:  
  - [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)  
  - [12factor.net Config](https://12factor.net/config)

---

## 4. **Backing Services**  
**Treat backing services as attached resources**  
- **Description**: Databases, queues, etc., are interchangeable via config.  
- **Implementation**:  
  - Use environment variables to define service connections (e.g., `DATABASE_URL`).  
- **GitHub Example**:  
  - Connect to GitHub Packages (container registry, npm registry) via tokens.  
  - Use GitHub Actions to deploy to AWS/GCP with environment-specific credentials.  
- **Reference**:  
  - [GitHub Packages](https://docs.github.com/en/packages)  
  - [12factor.net Backing Services](https://12factor.net/backing-services)

---

## 5. **Build, Release, Run**  
**Strictly separate build and run stages**  
- **Description**:  
  - **Build**: Convert code to an executable bundle.  
  - **Release**: Combine build with config.  
  - **Run**: Execute the release in the environment.  
- **GitHub Example**:  
  - Use GitHub Actions workflows for each stage:  
    ```yaml
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Build Docker Image
            run: docker build -t myapp:$GITHUB_SHA .
      release:
        needs: build
        runs-on: ubuntu-latest
        steps:
          - name: Deploy to Staging
            uses: mycompany/deploy-action@v1
            env:
              ENVIRONMENT: staging
    ```  
- **Reference**:  
  - [12factor.net Build Release Run](https://12factor.net/build-release-run)

---

## 6. **Processes**  
**Execute the app as one or more stateless processes**  
- **Description**: No in-memory state or sticky sessions.  
- **Implementation**:  
  - Store state in databases or caches (e.g., Redis).  
- **GitHub Example**:  
  - Use GitHub Actions to test statelessness by scaling containers in CI.  
- **Reference**:  
  - [12factor.net Processes](https://12factor.net/processes)

---

## 7. **Port Binding**  
**Export services via port binding**  
- **Description**: The app is self-contained and binds to a port.  
- **Implementation**:  
  - Avoid relying on webservers like Apache for execution.  
  - Example: A Node.js app binding to `process.env.PORT`.  
- **GitHub Example**:  
  - Use Docker `EXPOSE` and GitHub Actions to test port binding.  
    ```Dockerfile
    FROM node:20
    EXPOSE 3000
    CMD ["node", "app.js"]
    ```  
- **Reference**:  
  - [12factor.net Port Binding](https://12factor.net/port-binding)

---

## 8. **Concurrency**  
**Scale out via the process model**  
- **Description**: Scale horizontally via processes (e.g., Kubernetes pods).  
- **GitHub Example**:  
  - Use Kubernetes deployments in GitHub Actions:  
    ```yaml
    - name: Deploy to Kubernetes
      uses: azure/k8s-deploy@v1
      with:
        namespace: production
        replicas: 3
    ```  
- **Reference**:  
  - [12factor.net Concurrency](https://12factor.net/concurrency)

---

## 9. **Disposability**  
**Maximize robustness with fast startup/shutdown**  
- **Description**: Processes should handle graceful shutdowns (SIGTERM).  
- **GitHub Example**:  
  - Test shutdowns in CI using GitHub Actions:  
    ```yaml
    - name: Test Graceful Shutdown
      run: |
        docker run -d myapp
        docker stop myapp-container
    ```  
- **Reference**:  
  - [12factor.net Disposability](https://12factor.net/disposability)

---

## 10. **Dev/Prod Parity**  
**Keep development, staging, and production as similar as possible**  
- **Description**: Use the same databases and OS in all environments.  
- **GitHub Example**:  
  - Mirror production data to staging using GitHub Environments.  
  - Use GitHub Codespaces for identical dev environments.  
- **Reference**:  
  - [GitHub Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)  
  - [12factor.net Dev Prod Parity](https://12factor.net/dev-prod-parity)

---

## 11. **Logs**  
**Treat logs as event streams**  
- **Description**: Write logs to stdout/stderr, not files.  
- **Implementation**:  
  - Aggregate logs with tools like ELK or Datadog.  
- **GitHub Example**:  
  - Capture Action logs via `run:` steps.  
  - Integrate with [Sematext](https://sematext.com/docs/logs/github-actions/) for analysis.  
- **Reference**:  
  - [12factor.net Logs](https://12factor.net/logs)

---

## 12. **Admin Processes**  
**Run admin/management tasks as one-off processes**  
- **Description**: Use the same environment for admin scripts (e.g., DB migrations).  
- **GitHub Example**:  
  - Run migrations via GitHub Actions:  
    ```yaml
    - name: Run Database Migrations
      run: python manage.py migrate
      env:
        DATABASE_URL: ${{ secrets.PRODUCTION_DB_URL }}
    ```  
- **Reference**:  
  - [12factor.net Admin Processes](https://12factor.net/admin-processes)

---

## Conclusion  
Adopting the 12 Factor App principles with GitHub ensures your app is **scalable**, **maintainable**, and **cloud-ready**. Leverage GitHub Actions, Environments, and Packages to automate compliance with these best practices.

## References  
- [12factor.net](https://12factor.net)  
- [GitHub Actions Documentation](https://docs.github.com/en/actions)  
- [GitHub Environment Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)  
- [GitHub Packages](https://docs.github.com/en/packages)
