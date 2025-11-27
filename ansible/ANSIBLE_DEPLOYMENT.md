# Ansible Deployment Guide for Election Voting System

## Prerequisites

### On Your PC (Control Machine):
- ✅ Ansible installed (You have this!)
- ✅ Python 3.12.3
- ✅ SSH client

### On Target Server:
- Ubuntu 20.04+ or Debian-based Linux
- SSH access (username + password or SSH key)
- Sudo privileges
- At least 2GB RAM
- 20GB disk space

## Quick Start Guide

### Step 1: Configure Your Server Details

Edit `ansible/inventory.ini` and replace:

```ini
[webservers]
production ansible_host=YOUR_SERVER_IP ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Examples:**

For SSH key authentication (recommended):
```ini
production ansible_host=192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/my-key.pem
```

For password authentication:
```ini
production ansible_host=192.168.1.100 ansible_user=ubuntu ansible_ssh_pass=MyPassword123
```

For AWS EC2:
```ini
production ansible_host=ec2-54-123-45-67.compute-1.amazonaws.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/my-aws-key.pem
```

### Step 2: Set Environment Variables

**On Linux/WSL:**
```bash
export VITE_SUPABASE_URL=your_supabase_url
export VITE_SUPABASE_PUBLISHABLE_KEY=your_supabase_key
export VITE_SUPABASE_PROJECT_ID=your_project_id
```

**On Windows PowerShell:**
```powershell
$env:VITE_SUPABASE_URL="your_supabase_url"
$env:VITE_SUPABASE_PUBLISHABLE_KEY="your_supabase_key"
$env:VITE_SUPABASE_PROJECT_ID="your_project_id"
```

### Step 3: Test Connection

```bash
cd ansible
ansible webservers -m ping
```

Expected output:
```
production | SUCCESS => {
    "ping": "pong"
}
```

### Step 4: Run Deployment

```bash
ansible-playbook playbook.yml
```

With verbose output:
```bash
ansible-playbook playbook.yml -v
```

With password prompt (if not using SSH key):
```bash
ansible-playbook playbook.yml --ask-pass --ask-become-pass
```

## Deployment Process

The playbook will automatically:

1. ✅ Update system packages
2. ✅ Install Docker and Docker Compose
3. ✅ Clone your GitHub repository
4. ✅ Create environment configuration
5. ✅ Build Docker image
6. ✅ Start the container
7. ✅ Configure firewall
8. ✅ Verify deployment

**Total time: ~5-10 minutes**

## Access Your Application

After deployment completes:

```
http://YOUR_SERVER_IP:3000
```

## Common Commands

### Check deployment status:
```bash
ansible-playbook playbook.yml --check
```

### Deploy to specific host:
```bash
ansible-playbook playbook.yml --limit production
```

### Update application (redeploy):
```bash
ansible-playbook playbook.yml --tags update
```

### View container logs on server:
```bash
ssh user@server "docker logs election-voting-system"
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to server
```bash
# Test SSH manually
ssh -i ~/.ssh/your-key.pem user@server-ip

# Check inventory file syntax
ansible-inventory --list -i inventory.ini
```

**Problem:** Permission denied
```bash
# Ensure SSH key has correct permissions
chmod 600 ~/.ssh/your-key.pem

# Test with verbose output
ansible webservers -m ping -vvv
```

### Deployment Failures

**Problem:** Docker installation fails
- Ensure server has internet connection
- Check if Docker is already installed: `docker --version`

**Problem:** Port 3000 already in use
- Change port in `docker-compose.yml`
- Or stop existing service: `sudo lsof -ti:3000 | xargs kill -9`

**Problem:** Environment variables not set
```bash
# Verify they're set
echo $VITE_SUPABASE_URL

# Pass them explicitly
ansible-playbook playbook.yml -e "supabase_url=your_url"
```

## Server Setup (First Time)

### For AWS EC2:

1. Launch Ubuntu instance
2. Download .pem key file
3. Set key permissions: `chmod 600 my-key.pem`
4. Update inventory.ini with public IP
5. Run playbook

### For DigitalOcean Droplet:

1. Create Ubuntu droplet
2. Add your SSH key during creation
3. Note the IP address
4. Update inventory.ini
5. Run playbook

### For Local VM:

1. Install Ubuntu in VirtualBox/VMware
2. Enable SSH: `sudo apt install openssh-server`
3. Get IP: `ip addr show`
4. Update inventory.ini
5. Run playbook

## File Structure

```
ansible/
├── inventory.ini       # Server details
├── playbook.yml        # Deployment tasks
├── ansible.cfg         # Ansible configuration
└── ANSIBLE_DEPLOYMENT.md  # This guide
```

## Security Best Practices

1. **Use SSH keys instead of passwords**
2. **Keep .env.production secure** - never commit to Git
3. **Use firewall rules** - only open necessary ports
4. **Regular updates** - keep server packages updated
5. **SSL/HTTPS** - use reverse proxy (Nginx) with SSL

## Advanced Configuration

### Add SSL with Nginx:

Create `nginx-playbook.yml`:
```yaml
---
- name: Configure Nginx reverse proxy
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Configure proxy
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/voting-system
```

### Multiple Environments:

Create separate inventory files:
- `inventory-dev.ini`
- `inventory-staging.ini`
- `inventory-prod.ini`

Deploy to specific environment:
```bash
ansible-playbook -i inventory-prod.ini playbook.yml
```

## Monitoring

### Check application status:
```bash
ansible webservers -m shell -a "docker ps | grep election"
```

### Get container logs:
```bash
ansible webservers -m shell -a "docker logs election-voting-system --tail 50"
```

### Check resource usage:
```bash
ansible webservers -m shell -a "docker stats --no-stream"
```

## Rollback

If deployment fails, rollback to previous version:

```bash
ssh user@server
cd /opt/election-voting-system
git log --oneline -10  # Find previous commit
git checkout COMMIT_HASH
docker compose down
docker compose up -d --build
```

## Support

For issues:
1. Check logs: `docker logs election-voting-system`
2. Verify environment variables are set
3. Test server connectivity: `ansible webservers -m ping`
4. Run with verbose: `ansible-playbook playbook.yml -vvv`

## Next Steps

After successful deployment:
1. Set up domain name and DNS
2. Configure SSL certificate (Let's Encrypt)
3. Set up automated backups
4. Configure monitoring (Prometheus/Grafana)
5. Set up CI/CD pipeline

## Useful Links

- Ansible Documentation: https://docs.ansible.com
- Docker Documentation: https://docs.docker.com
- Ubuntu Server Guide: https://ubuntu.com/server/docs
