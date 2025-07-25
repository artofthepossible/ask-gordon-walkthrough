# Dockerfile Improvements with Ask Gordon

This example demonstrates how Gordon can help improve your Dockerfiles.

## Before: Unoptimized Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y curl
RUN apt-get install -y git

COPY . /app
WORKDIR /app

RUN pip3 install -r requirements.txt

EXPOSE 8000

CMD ["python3", "app.py"]
```

## Ask Gordon for Help

**Query**: "Can you help me optimize this Dockerfile for a Python web application?"

## After: Gordon's Optimized Version

```dockerfile
# Use specific version and smaller base image
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Install system dependencies in single layer
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements first (better caching)
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Change ownership to non-root user
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["python", "app.py"]
```

## Key Improvements Gordon Identified

### 1. **Base Image Optimization**
- ❌ `ubuntu:latest` → ✅ `python:3.11-slim`
- Smaller image size and includes Python by default

### 2. **Layer Optimization**
- ❌ Multiple RUN commands → ✅ Combined into single layers
- Better caching and smaller image

### 3. **Security Enhancements**
- ✅ Non-root user creation
- ✅ Proper file ownership
- ✅ Environment variables for Python optimization

### 4. **Better Caching Strategy**
- ✅ Copy requirements.txt first
- ✅ Install dependencies before copying code
- ✅ Leverages Docker layer caching

### 5. **Production Readiness**
- ✅ Health check endpoint
- ✅ Proper environment variables
- ✅ Cleanup of package manager cache

## Asking Gordon for Specific Improvements

### Security Focus
**Query**: "How can I make this Dockerfile more secure?"

Gordon might suggest:
- Using distroless images
- Scanning for vulnerabilities
- Implementing least privilege principles
- Adding security scanning in CI/CD

### Performance Focus
**Query**: "How can I make this Docker image build faster and run more efficiently?"

Gordon might suggest:
- Multi-stage builds
- Better .dockerignore file
- Parallel dependency installation
- Resource limits and requests

### Size Optimization
**Query**: "How can I make this Docker image smaller?"

Gordon might suggest:
- Alpine-based images
- Removing development dependencies
- Using multi-stage builds
- Compressing assets

## Additional Examples

### Multi-stage Build Suggestion

```dockerfile
# Build stage
FROM python:3.11-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-slim

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app
COPY . .
RUN chown -R appuser:appuser /app
USER appuser

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

EXPOSE 8000
CMD ["python", "app.py"]
```

## Best Practices Gordon Teaches

1. **Use specific base image versions**
2. **Minimize layers and image size**
3. **Follow security best practices**
4. **Leverage build cache effectively**
5. **Add health checks for production**
6. **Use non-root users**
7. **Clean up package manager caches**
8. **Set appropriate environment variables**

## Try It Yourself

1. Create a Dockerfile for your application
2. Ask Gordon: "Can you review and improve this Dockerfile?"
3. Implement the suggested improvements
4. Test the optimized version
5. Compare build time and image size

## Common Gordon Suggestions

- **Replace latest tags** with specific versions
- **Combine RUN commands** to reduce layers
- **Use .dockerignore** to exclude unnecessary files
- **Add health checks** for container monitoring
- **Implement proper user management** for security
- **Use multi-stage builds** for complex applications
