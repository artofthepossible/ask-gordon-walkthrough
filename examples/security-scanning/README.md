# Security Scanning with Ask Gordon

This example demonstrates how Gordon can help identify and fix security vulnerabilities in your Docker images and containers.

## Overview

Gordon can assist with:
- Vulnerability scanning and analysis
- Security best practices implementation
- Configuration security reviews
- Docker Hardened Images migration
- Compliance recommendations

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

### Example: Using Docker Secrets

```yaml
version: '3.8'
services:
  app:
    image: my-app
    secrets:
      - api_key
      - db_password
    environment:
      - API_KEY_FILE=/run/secrets/api_key
      - DB_PASSWORD_FILE=/run/secrets/db_password

secrets:
  api_key:
    external: true
  db_password:
    external: true
```

## Scenario 5: Docker Hardened Images Migration

### Ask Gordon
**Query**: "Help me migrate to Docker Hardened Images for better security"

### Gordon's Migration Plan

1. **Assessment**
   ```bash
   # Check current image
   docker scout cves my-app:current
   
   # Analyze dependencies
   docker run --rm my-app:current pip list
   ```

2. **Select Hardened Base**
   ```dockerfile
   # FROM python:3.11-slim
   FROM cgr.dev/chainguard/python:latest-dev AS builder
   
   # Multi-stage build for hardened production
   FROM cgr.dev/chainguard/python:latest AS production
   ```

3. **Test Compatibility**
   ```bash
   # Build with hardened image
   docker build -t my-app:hardened .
   
   # Test functionality
   docker run --rm my-app:hardened python -c "import sys; print(sys.version)"
   ```

4. **Performance Comparison**
   ```bash
   # Compare image sizes
   docker images | grep my-app
   
   # Compare vulnerability counts
   docker scout cves my-app:current
   docker scout cves my-app:hardened
   ```

## Security Best Practices Gordon Teaches

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
- Security contexts in Kubernetes
- AppArmor/SELinux profiles
- Network policies
- Resource limits

### 4. Secret Management
- Never hardcode secrets
- Use secret management systems
- Rotate secrets regularly
- Audit secret access

## Interactive Security Review

### Ask Gordon for a Complete Security Audit

**Query**: "Can you perform a comprehensive security review of my Docker setup?"

Gordon will examine:

1. **Dockerfile Security**
   - Base image vulnerabilities
   - Package management
   - User privileges
   - File permissions

2. **Runtime Configuration**
   - Docker Compose security
   - Environment variables
   - Volume mounts
   - Network exposure

3. **Infrastructure Security**
   - Container orchestration
   - Network policies
   - Access controls
   - Monitoring setup

4. **Compliance Requirements**
   - Industry standards (PCI, HIPAA, SOC2)
   - Regulatory compliance
   - Security benchmarks
   - Audit requirements

## Security Scanning Tools Gordon Recommends

### Built-in Docker Scout
```bash
# Enable Docker Scout
docker scout enroll

# Scan for vulnerabilities
docker scout cves my-image:tag

# View recommendations
docker scout recommendations my-image:tag
```

### Third-party Security Tools
```bash
# Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image my-image:tag

# Snyk
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  snyk/snyk:docker test my-image:tag

# Anchore
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/grype my-image:tag
```

## Monitoring and Alerting

### Ask Gordon about Security Monitoring

**Query**: "How can I monitor my containers for security issues in production?"

Gordon's recommendations:
- Runtime security monitoring
- Vulnerability scanning automation
- Security event logging
- Compliance reporting
- Incident response procedures

## Security Checklist from Gordon

### Pre-deployment
- [ ] Vulnerability scan passed
- [ ] No hardcoded secrets
- [ ] Non-root user configured
- [ ] Minimal package installation
- [ ] Security contexts defined

### Runtime
- [ ] Network policies in place
- [ ] Resource limits configured
- [ ] Logging enabled
- [ ] Monitoring active
- [ ] Access controls enforced

### Ongoing
- [ ] Regular vulnerability scans
- [ ] Security patches applied
- [ ] Secret rotation
- [ ] Audit logs reviewed
- [ ] Compliance verified

## Common Security Issues Gordon Identifies

1. **Vulnerable base images**
2. **Hardcoded secrets**
3. **Running as root**
4. **Excessive privileges**
5. **Insecure network exposure**
6. **Missing security updates**
7. **Weak access controls**
8. **Insufficient monitoring**

Remember: Security is an ongoing process, not a one-time check. Regularly ask Gordon to review your security posture!
