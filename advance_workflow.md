# DevSecOps GitHub Actions Pipeline Documentation

## Overview
This GitHub Actions workflow automates the **DevSecOps pipeline** for services in the `devsecops` branch. It integrates security and compliance checks, ensuring code changes are analyzed for vulnerabilities before deployment.

## Workflow Structure
The pipeline consists of multiple jobs, each handling a specific security or compliance check.

### 1. Change Detection (`changes`)
- Uses the `paths-filter` action to detect changes in specific service directories.
- Outputs a list of changed services to be processed by later jobs.

### 2. Compilation (`compile`)
- Runs only for detected changes.
- Uses Go to compile the service executable.
- Archives the compiled binary for further analysis.

### 3. SBOM Generation (`generate_sbom`)
- Uses **CycloneDX** to generate a **Software Bill of Materials (SBOM)**.
- Uploads the SBOM to **DefectDojo**, a security vulnerability management tool.

### 4. Unit Testing (`unit_test`)
- Placeholder job intended for running unit tests (not yet implemented).

### 5. License Compliance Check (`license_checker`)
- Placeholder job to ensure license compliance (not yet implemented).

### 6. Software Composition Analysis (`sca`)
- Runs **dependency scanning** to detect vulnerabilities in third-party libraries.
- Uploads scan results to **DefectDojo**.

### 7. Static Application Security Testing (`sast`)
- Expected to perform **SAST** (Static Code Analysis) after all security and compliance checks.
- Not yet fully implemented in the current workflow.

## Security & Compliance Integrations
### DefectDojo
- Used to manage security reports from SBOM and SCA scans.

### CycloneDX
- Generates **SBOM** to track open-source dependencies and potential vulnerabilities.

### Dependency-Check
- Scans project dependencies for known security issues.

## Pipeline Flow
1. Detect changes in the repository.
2. Compile the source code.
3. Generate an SBOM and upload it to DefectDojo.
4. Perform Software Composition Analysis (SCA).
5. Run Static Application Security Testing (SAST) (future enhancement).
6. Validate license compliance (future enhancement).
7. Upload all reports to DefectDojo for security auditing.

## Purpose
This pipeline ensures that **every code change** in the `devsecops` branch is:
- Properly built and compiled.
- Scanned for security vulnerabilities.
- Checked for dependency and license issues.
- Uploaded to security tools for further analysis and tracking.
