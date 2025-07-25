# Security Scanning with Ask Gordon

This example demonstrates how Gordon can help identify and fix security vulnerabilities in your Docker images and containers.

## Overview

Gordon can assist with:
- Vulnerability scanning and analysis
- Security best practices implementation

## Scenario 1: Basic Vulnerability Scanning

### Ask Gordon
**Query**: "Can you scan my Docker image for security vulnerabilities?"

### Gordon's Scanning Process

```bash
# Gordon might suggest using Docker Scout
docker scout cves my-app:latest

# Or other scanning tools
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image my-app:latest
```

### Sample Vulnerability Report Analysis

```
HIGH SEVERITY VULNERABILITIES FOUND:
- CVE-2023-1234: OpenSSL vulnerability in base image
- CVE-2023-5678: Package manager security issue
- CVE-2023-9012: Python library vulnerability

RECOMMENDATIONS:
1. Update base image to latest version
2. Remove unnecessary packages
3. Update vulnerable dependencies
```

### Gordon's Remediation Suggestions

```dockerfile
# Before: Vulnerable base image
FROM ubuntu:18.04

# After: Updated secure base image
FROM ubuntu:22.04

# Before: Installing unnecessary packages
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    vim \
    git \
    python3 \
    python3-pip

# After: Minimal package installation
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*
```

## Scenario 2: Configuration Security Review

### Ask Gordon
**Query**: "Can you review my Docker configuration for security issues?"

### Sample Docker Compose Analysis

```yaml
# BEFORE: Insecure configuration
version: '3.8'
services:
  web:
    image: my-app
    ports:
      - "80:8080"
    volumes:
      - /:/host  # ❌ Dangerous: mounting entire host filesystem
    environment:
      - DATABASE_PASSWORD=secret123  # ❌ Hardcoded password
    privileged: true  # ❌ Unnecessary privileges
    user: root  # ❌ Running as root
```

### Gordon's Secure Configuration

```yaml
# AFTER: Secure configuration
version: '3.8'
services:
  web:
    image: my-app
    ports:
      - "8080:8080"  # ✅ Non-privileged port
    volumes:
      - ./app-data:/app/data:ro  # ✅ Specific mount, read-only
    environment:
      - DATABASE_PASSWORD_FILE=/run/secrets/db_password  # ✅ Using secrets
    secrets:
      - db_password
    user: "1000:1000"  # ✅ Non-root user
    read_only: true  # ✅ Read-only filesystem
    security_opt:
      - no-new-privileges:true  # ✅ Security options
    cap_drop:
      - ALL  # ✅ Drop all capabilities
    cap_add:
      - NET_BIND_SERVICE  # ✅ Add only needed capabilities

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## Scenario 3: Dockerfile Security Hardening

### Ask Gordon
**Query**: "How can I make my Dockerfile more secure?"

### Before: Basic Dockerfile
```dockerfile
FROM python:latest
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 8000
CMD ["python", "app.py"]
```

### After: Gordon's Hardened Dockerfile
```dockerfile
# Use specific version and minimal base
FROM python:3.11-slim@sha256:specific-hash

# Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser -d /app -s /sbin/nologin appuser

# Set secure environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Install security updates
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        # Only essential packages
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Copy requirements first for better caching
COPY requirements.txt /app/
WORKDIR /app

# Install dependencies with hash verification
RUN pip install --no-cache-dir --require-hashes -r requirements.txt

# Copy application code
COPY --chown=appuser:appuser . /app

# Switch to non-root user
USER appuser

# Use non-privileged port
EXPOSE 8080

# Add health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8080/health')"

# Run with proper signal handling
CMD ["python", "app.py"]
```

## Scenario 4: Secret Management

### Ask Gordon
**Query**: "How should I handle secrets securely in Docker?"

### ❌ Insecure Practices Gordon Identifies

```dockerfile
# DON'T: Hardcode secrets
ENV API_KEY=super-secret-key

# DON'T: Copy secrets into image
COPY secrets.txt /app/

# DON'T: Pass secrets as build args
ARG DATABASE_PASSWORD=secret123
```

### ✅ Secure Alternatives Gordon Suggests

```dockerfile
# Use Docker secrets
# docker service create --secret my-secret my-app

# Use environment files (not in image)
# docker run --env-file secrets.env my-app

# Use external secret management
ENV SECRET_PROVIDER=vault
ENV VAULT_ADDR=https://vault.example.com
```
## Scenario 5: Docker Hardened Images Migration

### Ask Gordon
**Query**: "Help me migrate to Docker Hardened Images for better security"

## Security Best Practices 

### 1. Principle of Least Privilege
- Use non-root users
- Drop unnecessary capabilities
- Read-only filesystems where possible
- Minimal package installation

### 2. Image Security
- Use official base images
- Pin image versions with digests
- Regular vulnerability scanning
- Multi-stage builds for production

### 3. Runtime Security
- Consider Security controls in your in Managed Kubernetes Solution
- AppArmor/SELinux profiles
- Network policies
- Resource limits

### 4. Secret Management
- Never hardcode secrets
- Use secret management systems
- Rotate secrets regularly
- Audit secret access
