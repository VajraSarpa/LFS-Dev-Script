# LFS-Dev-Script
This installs all packages needed for development of my projects.

# EC2 LFS Development Environment Setup Script

A comprehensive Python script that automatically configures your AWS EC2 instance with all necessary tools for building Linux From Scratch (LFS).

## What This Script Does

This script automates the complete setup of your EC2 instance, installing and configuring:

- ✅ **System Updates** - Latest security patches and system packages
- ✅ **LFS Build Tools** - GCC, Make, Binutils, and 20+ essential development packages
- ✅ **AWS CLI v2** - Latest AWS command-line interface
- ✅ **AWS SDK (boto3)** - Python AWS library for automation
- ✅ **VS Code Server** - Browser-based VS Code IDE
- ✅ **GitHub CLI** - Modern GitHub integration (no SSH keys needed)
- ✅ **LFS Environment** - Directory structure and environment variables
- ✅ **Git Configuration** - Username, email, and authentication
- ✅ **Your GitHub Repository** - Automatically clones your project
- ✅ **Helpful Aliases** - Shortcuts for common commands

## Prerequisites

- AWS EC2 instance running **Ubuntu 22.04 LTS** (or similar Debian-based system)
- Access to EC2 via **EC2 Instance Connect** (browser terminal)
- A **GitHub account** and repository created
- Internet connection

## Installation

### Step 1: Copy Script to EC2 Instance

**In your EC2 browser terminal**, create the script file:

```bash
nano setup-lfs-dev.py
```

**Paste the entire Python script**, then save and exit:
- Press `Ctrl+O` to save
- Press `Enter` to confirm
- Press `Ctrl+X` to exit

### Step 2: Make Script Executable

```bash
chmod +x setup-lfs-dev.py
```

### Step 3: Run the Script

```bash
python3 setup-lfs-dev.py
```

## What to Expect

### Interactive Prompts

The script will prompt you for:

1. **Confirmation to proceed** - Type `y` and press Enter
2. **GitHub Authentication** - Follow the on-screen instructions:
   - Choose **GitHub.com**
   - Choose **HTTPS** (not SSH)
   - Choose **Yes** to authenticate Git
   - Choose **Login with a web browser**
   - Copy the one-time code (format: `XXXX-XXXX`)
   - On your desktop browser, go to https://github.com/login/device
   - Paste the code and authorize
3. **Git Configuration** - Enter your name and email for commits
4. **Repository Details** - Enter your GitHub repository in format: `username/repo-name`
   - Example: `johndoe/lfs-build-project`

### Installation Time

- **Total Duration**: 10-15 minutes
- **Most time-consuming steps**:
  - System update (2-3 minutes)
  - Installing build tools (3-4 minutes)
  - AWS CLI installation (2-3 minutes)

### Script Output

The script provides color-coded feedback:
- 🟢 **Green** - Success messages
- 🔵 **Cyan** - Informational messages
- 🟡 **Yellow** - Warnings (non-critical issues)
- 🔴 **Red** - Errors (may continue anyway)

## Post-Installation

### Verify Installation

After the script completes, verify everything is installed:

```bash
# Reload bash configuration
source ~/.bashrc

# Check installations
aws --version                    # Should show AWS CLI v2.x.x
gh --version                     # Should show GitHub CLI version
python3 -c 'import boto3; print(boto3.__version__)'  # Should show boto3 version
echo $LFS                        # Should show: /mnt/lfs
git --version                    # Should show Git version
```

### Navigate to Your Project

```bash
# Go to your cloned repository
cd ~/lfs-build-project  # Or use alias: goproject

# Verify files are there
ls -la

# Check git status
git status
```

### Available Aliases

The script creates helpful aliases automatically:

| Alias | Command | Description |
|-------|---------|-------------|
| `ll` | `ls -lah` | Detailed directory listing |
| `lfs` | `sudo su - lfs` | Switch to lfs user |
| `gosrc` | `cd /mnt/lfs/sources` | Go to LFS sources directory |
| `goproject` | `cd ~/lfs-build-project` | Go to your GitHub repo |
| `status` | `df -h && free -h && uptime` | Show system status |
| `gst` | `git status` | Quick git status |
| `gpull` | `git pull` | Quick git pull |
| `gpush` | `git add . && git commit -m "Update" && git push` | Quick save to GitHub |
| `tmuxlfs` | `tmux new -s lfs-build` | Start tmux session |
| `tmuxattach` | `tmux attach -t lfs-build` | Reattach to tmux session |

## Next Steps

### 1. Download LFS Packages

```bash
# Go to sources directory
cd /mnt/lfs/sources

# Download package list
sudo wget https://www.linuxfromscratch.org/lfs/view/stable/wget-list

# Download all packages (~500MB, takes 5-10 minutes)
sudo wget --input-file=wget-list --continue --directory-prefix=/mnt/lfs/sources

# Download checksums
sudo wget https://www.linuxfromscratch.org/lfs/view/stable/md5sums

# Verify packages
md5sum -c md5sums
```

All packages should show `OK`.

### 2. Create LFS User

```bash
# Create lfs group and user
sudo groupadd lfs
sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs
sudo passwd lfs
# Enter password (e.g., lfs2024)

# Grant ownership
sudo chown -v lfs /mnt/lfs/{usr,lib,var,etc,bin,sbin,tools,sources}
sudo chown -v lfs /mnt/lfs

# Set up lfs user environment
sudo -u lfs bash << "EOF"
cat > ~/.bash_profile << "PROFILE_EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
PROFILE_EOF

cat > ~/.bashrc << "BASHRC_EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
MAKEFLAGS=-j$(nproc)
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE MAKEFLAGS
BASHRC_EOF
EOF
```

### 3. Start Building

```bash
# Start persistent tmux session
tmux new -s lfs-build

# Switch to lfs user
sudo su - lfs

# Navigate to sources
cd /mnt/lfs/sources

# Begin Chapter 5 of LFS book!
# Follow: https://www.linuxfromscratch.org/lfs/view/stable/
```

## Troubleshooting

### Script Fails to Start

**Error: `python3: command not found`**
```bash
sudo apt update
sudo apt install -y python3
```

**Error: `Permission denied`**
```bash
chmod +x setup-lfs-dev.py
```

### GitHub Authentication Fails

**If authentication doesn't complete:**
```bash
# Re-authenticate manually
gh auth login

# Follow the prompts again
```

### Repository Clone Fails

**If repository doesn't clone:**
```bash
# Check authentication
gh auth status

# If not authenticated
gh auth login

# Clone manually
gh repo clone username/repo-name
# Or with HTTPS
git clone https://github.com/username/repo-name.git
```

### AWS CLI Installation Fails

**If AWS CLI doesn't install:**
```bash
# Install manually
cd /tmp
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Package Installation Errors

**If apt packages fail to install:**
```bash
# Update package lists
sudo apt update

# Try installing problem packages individually
sudo apt install -y build-essential
sudo apt install -y python3-pip
# etc.
```

## What Gets Installed

### LFS Build Tools (20+ packages)
- `build-essential` - GCC, G++, Make
- `bison` - Parser generator
- `gawk` - GNU awk
- `texinfo` - Documentation system
- `wget`, `curl` - File downloaders
- `libssl-dev` - OpenSSL development files
- `bc` - Basic calculator
- `libelf-dev` - ELF library
- `flex` - Lexical analyzer
- `libncurses5-dev` - Terminal handling
- `gperf` - Perfect hash function generator
- `autoconf`, `automake`, `libtool` - Build system tools
- `pkg-config` - Library management
- `python3`, `python3-pip` - Python interpreter and package manager
- `git` - Version control
- `vim` - Text editor
- `tmux` - Terminal multiplexer
- `htop` - Process viewer
- `unzip` - Archive extractor
- `jq` - JSON processor

### AWS Tools
- **AWS CLI v2** - Full AWS command-line interface
- **boto3** - AWS SDK for Python
- **botocore** - Low-level AWS SDK

### Development Tools
- **GitHub CLI (gh)** - GitHub integration
- **Git** - Version control
- **VS Code Server (code-server)** - Browser-based IDE

### System Configuration
- **LFS Environment Variables** - `$LFS` set to `/mnt/lfs`
- **Directory Structure** - `/mnt/lfs/{sources,tools,boot}` created
- **Bash Aliases** - Helpful shortcuts added to `~/.bashrc`

## File Locations

| Item | Location |
|------|----------|
| Script | `~/setup-lfs-dev.py` |
| LFS directory | `/mnt/lfs` |
| LFS sources | `/mnt/lfs/sources` |
| Your GitHub repo | `~/lfs-build-project` (or your repo name) |
| Bash configuration | `~/.bashrc` |
| Setup completion marker | `~/.lfs-setup-complete` |
| System marker | `/var/log/lfs-setup-complete.log` |

## Security Considerations

### What This Script Does NOT Install
- ❌ No SSH keys generated
- ❌ No passwords stored
- ❌ No AWS credentials saved
- ❌ No sensitive data in git config

### Safe Practices
- ✅ GitHub authentication via browser (secure OAuth)
- ✅ AWS CLI installed but not configured with credentials
- ✅ Git credentials stored securely by GitHub CLI
- ✅ All installations from official sources

### AWS CLI Configuration

The script installs AWS CLI but does NOT configure credentials. To configure:

```bash
aws configure
# Enter when prompted:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region (e.g., us-east-1)
# - Default output format (json)
```

## Updating the Script

To update or modify the script:

```bash
# Edit the script
nano setup-lfs-dev.py

# Make changes, then save

# Run updated version
python3 setup-lfs-dev.py
```

The script is idempotent - it checks if components are already installed and skips them.

## Uninstalling

To remove installed components:

```bash
# Remove AWS CLI
sudo rm /usr/local/bin/aws
sudo rm /usr/local/bin/aws_completer
sudo rm -rf /usr/local/aws-cli

# Remove GitHub CLI
sudo apt remove gh -y

# Remove VS Code Server
sudo systemctl stop code-server@$USER
sudo systemctl disable code-server@$USER
sudo apt remove code-server -y

# Remove LFS directory (WARNING: deletes all LFS work!)
sudo rm -rf /mnt/lfs

# Remove aliases from bashrc
nano ~/.bashrc
# Delete the "LFS Build Aliases" section
```

## Support & Resources

### Official Documentation
- [LFS Book](https://www.linuxfromscratch.org/lfs/view/stable/)
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [VS Code Server](https://github.com/coder/code-server)

### Community Support
- [LFS Mailing Lists](https://www.linuxfromscratch.org/mail.html)
- [LFS FAQ](https://www.linuxfromscratch.org/faq/)
- [GitHub CLI Discussions](https://github.com/cli/cli/discussions)

## Contributing

If you find issues with this script or have improvements:

1. Document the issue
2. Test your fix
3. Update this README if needed
4. Share your improvements!

## License

This script is provided as-is for educational purposes. Feel free to modify and distribute.

## Author

Created for the LFS Build Project  
Part of the Linux From Scratch journey

---

## Quick Reference

### Run Script
```bash
python3 setup-lfs-dev.py
```

### Verify Installation
```bash
source ~/.bashrc
aws --version
gh --version
echo $LFS
```

### Start Building
```bash
tmux new -s lfs-build
sudo su - lfs
cd /mnt/lfs/sources
```

### Git Workflow
```bash
git status
git add .
git commit -m "Your message"
git push
```

---

**Setup time: ~15 minutes | One-time setup | Fully automated**

Ready to build Linux From Scratch! 🚀
