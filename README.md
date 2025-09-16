# payload Installation Guide

payload is a free and open-source code-first CMS. Payload provides code-first headless CMS for developers

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
  - RAM: 1GB minimum
  - Storage: 1GB for data
  - Network: HTTP/API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default payload port)
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

# Install payload
sudo dnf install -y payload

# Enable and start service
sudo systemctl enable --now payload

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
payload --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install payload
sudo apt install -y payload

# Enable and start service
sudo systemctl enable --now payload

# Configure firewall
sudo ufw allow 3000

# Verify installation
payload --version
```

### Arch Linux

```bash
# Install payload
sudo pacman -S payload

# Enable and start service
sudo systemctl enable --now payload

# Verify installation
payload --version
```

### Alpine Linux

```bash
# Install payload
apk add --no-cache payload

# Enable and start service
rc-update add payload default
rc-service payload start

# Verify installation
payload --version
```

### openSUSE/SLES

```bash
# Install payload
sudo zypper install -y payload

# Enable and start service
sudo systemctl enable --now payload

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
payload --version
```

### macOS

```bash
# Using Homebrew
brew install payload

# Start service
brew services start payload

# Verify installation
payload --version
```

### FreeBSD

```bash
# Using pkg
pkg install payload

# Enable in rc.conf
echo 'payload_enable="YES"' >> /etc/rc.conf

# Start service
service payload start

# Verify installation
payload --version
```

### Windows

```bash
# Using Chocolatey
choco install payload

# Or using Scoop
scoop install payload

# Verify installation
payload --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/payload

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
payload --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable payload

# Start service
sudo systemctl start payload

# Stop service
sudo systemctl stop payload

# Restart service
sudo systemctl restart payload

# Check status
sudo systemctl status payload

# View logs
sudo journalctl -u payload -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add payload default

# Start service
rc-service payload start

# Stop service
rc-service payload stop

# Restart service
rc-service payload restart

# Check status
rc-service payload status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'payload_enable="YES"' >> /etc/rc.conf

# Start service
service payload start

# Stop service
service payload stop

# Restart service
service payload restart

# Check status
service payload status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start payload
brew services stop payload
brew services restart payload

# Check status
brew services list | grep payload
```

### Windows Service Manager

```powershell
# Start service
net start payload

# Stop service
net stop payload

# Using PowerShell
Start-Service payload
Stop-Service payload
Restart-Service payload

# Check status
Get-Service payload
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream payload_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name payload.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name payload.example.com;

    ssl_certificate /etc/ssl/certs/payload.example.com.crt;
    ssl_certificate_key /etc/ssl/private/payload.example.com.key;

    location / {
        proxy_pass http://payload_backend;
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
    ServerName payload.example.com
    Redirect permanent / https://payload.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName payload.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/payload.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/payload.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend payload_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/payload.pem
    redirect scheme https if !{ ssl_fc }
    default_backend payload_backend

backend payload_backend
    balance roundrobin
    server payload1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R payload:payload /etc/payload
sudo chmod 750 /etc/payload

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status payload

# View logs
sudo journalctl -u payload -f

# Monitor resource usage
top -p $(pgrep payload)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/payload"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/payload-backup-$DATE.tar.gz" /etc/payload /var/lib/payload

echo "Backup completed: $BACKUP_DIR/payload-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop payload

# Restore from backup
tar -xzf /backup/payload/payload-backup-*.tar.gz -C /

# Start service
sudo systemctl start payload
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u payload -n 100
sudo tail -f /var/log/payload/payload.log

# Check configuration
payload --version

# Check permissions
ls -la /etc/payload
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep payload)

# Check disk I/O
iotop -p $(pgrep payload)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  payload:
    image: payload:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/payload
      - ./data:/var/lib/payload
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update payload

# Debian/Ubuntu
sudo apt update && sudo apt upgrade payload

# Arch Linux
sudo pacman -Syu payload

# Alpine Linux
apk update && apk upgrade payload

# openSUSE
sudo zypper update payload

# FreeBSD
pkg update && pkg upgrade payload

# Always backup before updates
tar -czf /backup/payload-pre-update-$(date +%Y%m%d).tar.gz /etc/payload

# Restart after updates
sudo systemctl restart payload
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/payload

# Clean old logs
find /var/log/payload -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/payload
```

## Additional Resources

- Official Documentation: https://docs.payload.org/
- GitHub Repository: https://github.com/payload/payload
- Community Forum: https://forum.payload.org/
- Best Practices Guide: https://docs.payload.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
