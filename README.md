# laminar Installation Guide

laminar is a free and open-source lightweight CI. Laminar provides lightweight, modular continuous integration service

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 10GB for jobs
  - Network: HTTP interface
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default laminar port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install laminar
sudo dnf install -y laminar

# Enable and start service
sudo systemctl enable --now laminar

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
laminar --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install laminar
sudo apt install -y laminar

# Enable and start service
sudo systemctl enable --now laminar

# Configure firewall
sudo ufw allow 8080

# Verify installation
laminar --version
```

### Arch Linux

```bash
# Install laminar
sudo pacman -S laminar

# Enable and start service
sudo systemctl enable --now laminar

# Verify installation
laminar --version
```

### Alpine Linux

```bash
# Install laminar
apk add --no-cache laminar

# Enable and start service
rc-update add laminar default
rc-service laminar start

# Verify installation
laminar --version
```

### openSUSE/SLES

```bash
# Install laminar
sudo zypper install -y laminar

# Enable and start service
sudo systemctl enable --now laminar

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
laminar --version
```

### macOS

```bash
# Using Homebrew
brew install laminar

# Start service
brew services start laminar

# Verify installation
laminar --version
```

### FreeBSD

```bash
# Using pkg
pkg install laminar

# Enable in rc.conf
echo 'laminar_enable="YES"' >> /etc/rc.conf

# Start service
service laminar start

# Verify installation
laminar --version
```

### Windows

```bash
# Using Chocolatey
choco install laminar

# Or using Scoop
scoop install laminar

# Verify installation
laminar --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/laminar

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
laminar --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable laminar

# Start service
sudo systemctl start laminar

# Stop service
sudo systemctl stop laminar

# Restart service
sudo systemctl restart laminar

# Check status
sudo systemctl status laminar

# View logs
sudo journalctl -u laminar -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add laminar default

# Start service
rc-service laminar start

# Stop service
rc-service laminar stop

# Restart service
rc-service laminar restart

# Check status
rc-service laminar status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'laminar_enable="YES"' >> /etc/rc.conf

# Start service
service laminar start

# Stop service
service laminar stop

# Restart service
service laminar restart

# Check status
service laminar status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start laminar
brew services stop laminar
brew services restart laminar

# Check status
brew services list | grep laminar
```

### Windows Service Manager

```powershell
# Start service
net start laminar

# Stop service
net stop laminar

# Using PowerShell
Start-Service laminar
Stop-Service laminar
Restart-Service laminar

# Check status
Get-Service laminar
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream laminar_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name laminar.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name laminar.example.com;

    ssl_certificate /etc/ssl/certs/laminar.example.com.crt;
    ssl_certificate_key /etc/ssl/private/laminar.example.com.key;

    location / {
        proxy_pass http://laminar_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName laminar.example.com
    Redirect permanent / https://laminar.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName laminar.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/laminar.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/laminar.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend laminar_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/laminar.pem
    redirect scheme https if !{ ssl_fc }
    default_backend laminar_backend

backend laminar_backend
    balance roundrobin
    server laminar1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R laminar:laminar /etc/laminar
sudo chmod 750 /etc/laminar

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status laminar

# View logs
sudo journalctl -u laminar -f

# Monitor resource usage
top -p $(pgrep laminar)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/laminar"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/laminar-backup-$DATE.tar.gz" /etc/laminar /var/lib/laminar

echo "Backup completed: $BACKUP_DIR/laminar-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop laminar

# Restore from backup
tar -xzf /backup/laminar/laminar-backup-*.tar.gz -C /

# Start service
sudo systemctl start laminar
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u laminar -n 100
sudo tail -f /var/log/laminar/laminar.log

# Check configuration
laminar --version

# Check permissions
ls -la /etc/laminar
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep laminar)

# Check disk I/O
iotop -p $(pgrep laminar)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  laminar:
    image: laminar:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/laminar
      - ./data:/var/lib/laminar
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update laminar

# Debian/Ubuntu
sudo apt update && sudo apt upgrade laminar

# Arch Linux
sudo pacman -Syu laminar

# Alpine Linux
apk update && apk upgrade laminar

# openSUSE
sudo zypper update laminar

# FreeBSD
pkg update && pkg upgrade laminar

# Always backup before updates
tar -czf /backup/laminar-pre-update-$(date +%Y%m%d).tar.gz /etc/laminar

# Restart after updates
sudo systemctl restart laminar
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/laminar

# Clean old logs
find /var/log/laminar -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/laminar
```

## Additional Resources

- Official Documentation: https://docs.laminar.org/
- GitHub Repository: https://github.com/laminar/laminar
- Community Forum: https://forum.laminar.org/
- Best Practices Guide: https://docs.laminar.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
