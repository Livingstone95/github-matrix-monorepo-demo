# **GitHub Actions Pipeline for DevSecOps: Build and Deploy to QA**  

This GitHub Actions workflow automates the build, security scan, and deployment of microservices in a QA (UAT) environment. It runs when there is a push to any branch matching `releases/**` and affects files in the `ms-**/**` directories. The pipeline is structured into two main jobs:  

1. **Change Detection (`changes`)**: Identifies which services have been modified.  
2. **Build and Deploy (`services_job`)**: Builds, scans, and deploys the affected services.  

---

## **Workflow Breakdown**  

### **1. Triggering the Workflow (`on` Section)**
- The pipeline is triggered when there is a `push` to branches matching the pattern `releases/**`.  
- It only runs if changes are detected in directories matching `ms-**/**`, ensuring unnecessary builds do not occur.  

---

## **2. Environment Variables (`env` Section)**
- `PROJECT_ID`: Shared project ID stored in GitHub Secrets.  
- `GITHUB_SHA`: Commit SHA of the pushed code.  
- `GITHUB_REF`: Branch reference where the change occurred.  
- `RUN_ID`: Unique identifier for the workflow run.  
- `REGISTRY_HOSTNAME`: Container registry in GCP where Docker images will be stored.  
- `VERSION`: Fixed as `uat` for UAT deployment.  
- `GCP_PROJECT_NAME`: Name of the GCP project where artifacts will be deployed.  

---

## **3. Detecting Changes (`changes` Job)**
- This job determines which microservices (`ms-**`) have changed.  
- Uses the `dorny/paths-filter@v2` action to check for changes in specific service directories (`service-icecream`, `service-popcord`, `service-ondo`, `service-fries`, `service-yam`).  
- Outputs the list of changed services to be processed in the next job.  

---

## **4. Processing Changed Services (`services_job` Job)**
- Runs for each changed service detected in the previous job.  
- Uses a **matrix strategy** to run the job separately for each service.  
- Each service is built, scanned, and deployed independently.  

### **Steps in `services_job`**
#### **Step 1: Checkout Repository**
- Checks out the code from GitHub using `actions/checkout@v3`.  

#### **Step 2: Set Up Java Environment**
- Installs Java 17 using `actions/setup-java@v3`, required for building the services.  

#### **Step 3: Authenticate to Google Cloud**
- Uses `google-github-actions/auth@v1` to authenticate with GCP.  
- Configures Docker authentication for Google Artifact Registry.  

#### **Step 4: Print Debugging Information**
- Logs commit details, ref, and actor information for debugging.  

#### **Step 5: Build Dependencies for `ms-card` (if applicable)**
- If the changed service is `ms-card`, it builds an additional dependency (`thredd-client`).  

#### **Step 6: Build the Java Project**
- Runs `mvn clean install` for the changed service.  

#### **Step 7: SonarCloud Security Scan**
- Uses `SonarSource/sonarqube-scan-action@master` to scan the service for security vulnerabilities and code quality.  
- Uploads the scan results to SonarCloud.  

#### **Step 8: Build and Tag Docker Image**
- Creates multiple tagged versions of the Docker image:  
  - `$VERSION` (e.g., `uat`)  
  - `$VERSION-YYYY-MM-DD_HH-MM-SS` (timestamp-based)  
  - `$VERSION-${GITHUB_SHA::8}` (short SHA)  
  - `$VERSION-$RUN_ID` (GitHub run ID)  
  - `latest`  
- Pushes all tagged images to Google Artifact Registry.  

#### **Step 9: Trigger Integration Tests**
- Uses `peter-evans/repository-dispatch@v3` to trigger an external integration test process for the deployed service.  

---

## **Summary**
- This workflow is a **multi-service build and deployment pipeline** for a UAT environment.  
- It **automatically detects changes**, **builds Java microservices**, **runs security scans**, **packages the application as a Docker image**, and **deploys it to Google Artifact Registry**.  
- The **matrix strategy** ensures only modified services are built, improving efficiency.  
- Integration tests are triggered at the end to validate the deployment.  