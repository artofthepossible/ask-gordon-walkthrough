# Container Troubleshooting with Ask Gordon

This example shows how Gordon can help diagnose and fix common container issues.

## Scenario 1: Container Keeps Crashing

### The Problem
```bash
$ docker run my-app
# Container exits immediately with code 1
```

### Ask Gordon
**Query**: "My container keeps crashing immediately after startup. Here's the error log: [paste error]. Can you help me troubleshoot?"

### Gordon's Diagnostic Process

1. **Log Analysis**: Gordon examines the error messages
2. **Common Causes**: Identifies likely root causes
3. **Step-by-step Solution**: Provides debugging steps
4. **Prevention**: Suggests ways to avoid similar issues

### Example Solution Steps

```bash
# 1. Check detailed logs
docker logs <container_id>

# 2. Run container interactively
docker run -it my-app /bin/bash

# 3. Check if the entry point exists
ls -la /app/

# 4. Verify file permissions
ls -la /app/start.sh
chmod +x /app/start.sh

# 5. Test the application manually
python /app/main.py
```

## Scenario 2: Port Binding Issues

### The Problem
```bash
$ docker run -p 8080:3000 my-web-app
# Error: port already in use
```

### Ask Gordon
**Query**: "I'm getting a 'port already in use' error when trying to run my container. How do I fix this?"

### Gordon's Solution

```bash
# 1. Check what's using the port
lsof -i :8080
# or on some systems:
netstat -tulpn | grep :8080

# 2. Stop the conflicting process
kill <process_id>

# 3. Or use a different port
docker run -p 8081:3000 my-web-app

# 4. Or stop the conflicting container
docker ps
docker stop <conflicting_container>
```

## Scenario 3: Volume Mount Problems

### The Problem
```bash
$ docker run -v /foo/dollar:/app/data my-app
# Error: permission denied
```

### Ask Gordon
**Query**: "I'm having permission issues with volume mounts. My application can't write to the mounted directory."

### Gordon's Diagnosis

```bash
# 1. Check host directory permissions
ls -la /foo/dollar

# 2. Check container user
docker run my-app id

# 3. Fix permissions on host
sudo chown -R $(id -u):$(id -g) /foo/dollar

# 4. Or run container with correct user
docker run --user $(id -u):$(id -g) -v /foo/dollar:/app/data my-app

# 5. Alternative: use named volumes
docker volume create my-data
docker run -v my-data:/app/data my-app
```

## Scenario 4: Network Connectivity Issues

### The Problem
```bash
# Container can't connect to database
$ docker run my-app
# Error: connection refused to database:5432
```

### Ask Gordon
**Query**: "My application container can't connect to my database container. They're both running but connection is refused."

### Gordon's Network Troubleshooting

```bash
# 1. Check if containers are on same network
docker network ls
docker network inspect bridge

# 2. Create custom network
docker network create my-network

# 3. Run containers on same network
docker run --network my-network --name db postgres
docker run --network my-network my-app

# 4. Test connectivity
docker exec my-app ping db

# 5. Check port exposure
docker exec db netstat -tulpn | grep 5432
```

## Scenario 5: Out of Memory Issues

### The Problem
```bash
# Container gets killed due to memory
$ docker logs my-app
# Killed
```

### Ask Gordon
**Query**: "My container is being killed, I think it's running out of memory. How can I diagnose and fix this?"

### Gordon's Memory Analysis

```bash
# 1. Check container memory usage
docker stats my-app

# 2. Check system memory
free -h
df -h

# 3. Increase memory limit
docker run --memory=2g my-app

# 4. Monitor memory usage over time
docker run --name my-app my-image &
watch docker stats my-app

# 5. Check for memory leaks in application
docker exec my-app ps aux
docker exec my-app top
```

## Interactive Troubleshooting with Gordon

### Step-by-Step Debugging Session

1. **Describe the Issue**
   ```
   "My web application container starts but returns 500 errors"
   ```

2. **Gordon Asks for Details**
   - What's the exact error message?
   - Can you share the Dockerfile?
   - What's in the application logs?

3. **Gather Information**
   ```bash
   docker logs --tail 50 my-app
   docker exec my-app env
   docker inspect my-app
   ```

4. **Apply Gordon's Suggestions**
   - Check environment variables
   - Verify file paths
   - Test dependencies

5. **Verify the Fix**
   ```bash
   curl http://localhost:8080/health
   docker logs my-app
   ```

## Common Issues Gordon Helps With

### Application Startup Issues
- Missing environment variables
- Incorrect file paths
- Permission problems
- Missing dependencies

### Network Problems
- Port conflicts
- Container isolation
- DNS resolution
- Firewall issues

### Resource Constraints
- Memory limits
- CPU throttling
- Disk space
- File descriptor limits

### Configuration Errors
- Wrong environment variables
- Incorrect volume mounts
- Missing secrets
- Bad health checks

## Prevention Tips from Gordon

### 1. Better Logging
```dockerfile
# Add proper logging
ENV PYTHONUNBUFFERED=1
COPY logging.conf /app/
```

### 2. Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/health || exit 1
```

### 3. Graceful Shutdown
```dockerfile
# Handle signals properly
STOPSIGNAL SIGTERM
```

### 4. Resource Limits
```bash
# Set appropriate limits
docker run --memory=512m --cpus=0.5 my-app
```

## Gordon's Troubleshooting Methodology

1. **Understand the Problem**: Gather symptoms and error messages
2. **Isolate the Issue**: Test components separately
3. **Check Common Causes**: Apply knowledge of frequent issues
4. **Systematic Testing**: Work through potential solutions
5. **Verify the Fix**: Ensure the problem is resolved
6. **Document Solution**: Prevent future occurrences

## Advanced Debugging Commands

```bash
# Enter a failed container
docker run --rm -it --entrypoint=/bin/bash my-app

# Check container processes
docker exec my-app ps aux

# Monitor system calls
docker exec my-app strace -p 1

# Check network connections
docker exec my-app netstat -tulpn

# Inspect container filesystem
docker exec my-app find /app -type f -name "*.log"

# Check environment
docker exec my-app printenv | sort
```

## When to Ask Gordon

- ✅ Container crashes or exits unexpectedly
- ✅ Network connectivity issues
- ✅ Permission or volume mount problems
- ✅ Performance issues
- ✅ Configuration errors
- ✅ Resource constraint problems

Remember: The more specific information you provide to Gordon, the better assistance you'll receive!
