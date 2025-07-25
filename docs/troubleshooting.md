# Troubleshooting Ask Gordon

This guide helps you resolve common issues when using Gordon, Docker's AI assistant.

## Common Issues

### 1. Gordon Not Available

**Problem**: Can't find or access Gordon in Docker Desktop

**Solutions**:
- ✅ **Check Docker Desktop Version**: Ensure you have version 4.38.0 or later
  ```bash
  docker --version
  ```
- ✅ **Enable Gordon**: Go to Docker Desktop Settings and enable AI features
- ✅ **Restart Docker Desktop**: Close and reopen the application
- ✅ **Update Docker Desktop**: Download the latest version

### 2. CLI Command Not Working

**Problem**: `docker ai` command not recognized

**Solutions**:
- ✅ **Verify Installation**: Ensure Docker Desktop is properly installed
- ✅ **Check PATH**: Make sure Docker CLI is in your system PATH
- ✅ **Restart Terminal**: Open a new terminal session
- ✅ **Update CLI**: Ensure you have the latest Docker CLI version

### 3. Gordon Not Responding

**Problem**: Gordon doesn't respond to queries or seems unresponsive

**Solutions**:
- ✅ **Check Internet Connection**: Gordon may require internet access
- ✅ **Wait and Retry**: Sometimes there may be temporary delays
- ✅ **Restart Docker Desktop**: Close and reopen the application
- ✅ **Clear Cache**: Try clearing Docker Desktop cache

### 4. Limited or Poor Responses

**Problem**: Gordon gives generic or unhelpful responses

**Solutions**:
- ✅ **Be More Specific**: Provide detailed context and specific questions
- ✅ **Include Error Messages**: Share exact error outputs
- ✅ **Provide Context**: Include relevant configuration details
- ✅ **Ask Follow-up Questions**: Iterate to get better responses

### 5. Feature Not Available

**Problem**: Specific Gordon features mentioned in documentation aren't working

**Solutions**:
- ✅ **Check Version**: Ensure you have the required Docker Desktop version
- ✅ **Feature Rollout**: Some features may be gradually rolled out
- ✅ **Regional Availability**: Check if the feature is available in your region
- ✅ **Account Status**: Verify your Docker account status

## Platform-Specific Issues

### Windows

**Common Issues**:
- WSL2 compatibility problems
- Windows Defender interference
- Hyper-V conflicts

**Solutions**:
- Update WSL2 to latest version
- Add Docker to Windows Defender exclusions
- Ensure Hyper-V is properly configured

### macOS

**Common Issues**:
- Rosetta 2 compatibility (Apple Silicon)
- Permission issues
- VirtioFS problems

**Solutions**:
- Install Rosetta 2 if on Apple Silicon
- Check Docker Desktop permissions in System Preferences
- Try different file sharing implementations

### Linux

**Common Issues**:
- Docker daemon not running
- Permission problems
- Systemd integration issues

**Solutions**:
- Start Docker daemon: `sudo systemctl start docker`
- Add user to docker group: `sudo usermod -aG docker $USER`
- Enable Docker service: `sudo systemctl enable docker`

## Diagnostic Steps

### 1. Basic Health Check

```bash
# Check Docker version
docker --version

# Check Docker Desktop status
docker system info

# Test basic Docker functionality
docker run hello-world
```

### 2. Gordon-Specific Checks

```bash
# Try the CLI command
docker ai --help

# Check for available AI features
docker ai "Hello, are you working?"
```

### 3. Log Analysis

Check Docker Desktop logs:
- **Windows**: `%APPDATA%\Docker\log.txt`
- **macOS**: `~/Library/Containers/com.docker.docker/Data/log/`
- **Linux**: `~/.docker/desktop/log/`

## Getting Help

### Self-Service Resources

1. **Docker Documentation**: [docs.docker.com](https://docs.docker.com/)
2. **Community Forums**: [Docker Community Forum](https://forums.docker.com/)
3. **Stack Overflow**: Search for Docker-related questions
4. **GitHub Issues**: Check Docker Desktop repository

### Reporting Issues

When reporting issues, include:
- Docker Desktop version
- Operating system details
- Exact error messages
- Steps to reproduce
- Expected vs actual behavior

### Contact Information

- **Docker Support**: For enterprise customers
- **Community Support**: Docker forums and Stack Overflow
- **Bug Reports**: GitHub repository issues

## Prevention Tips

### Regular Maintenance

1. **Keep Updated**: Regularly update Docker Desktop
2. **Clean Up**: Remove unused images and containers
3. **Monitor Resources**: Check system resource usage
4. **Backup**: Save important Docker configurations

### Best Practices

1. **System Requirements**: Ensure your system meets minimum requirements
2. **Resource Allocation**: Allocate sufficient memory and CPU to Docker
3. **Network Configuration**: Ensure proper network settings
4. **Security Settings**: Keep security features enabled

## Advanced Troubleshooting

### Reset Docker Desktop

If all else fails, try resetting Docker Desktop:
1. Go to Docker Desktop Settings
2. Navigate to "Troubleshoot"
3. Click "Reset to factory defaults"
4. Reconfigure your settings

⚠️ **Warning**: This will remove all containers, images, and volumes

### Reinstallation

As a last resort:
1. Completely uninstall Docker Desktop
2. Remove all Docker-related files and folders
3. Restart your system
4. Install the latest version of Docker Desktop
5. Reconfigure Gordon and other settings

## Still Need Help?

If you're still experiencing issues:
1. Watch the [walkthrough video](https://www.loom.com/share/3425574a21fa4a328037a5d7c9a6429f?sid=6bf1f7f3-4bff-4421-8b8d-c5ac72508115) for additional guidance
2. Check the [examples](../examples/) for working configurations
3. Visit the Docker community forums for peer support
