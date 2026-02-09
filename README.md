# Lab M2.02 - SSH Connection and Security Best Practices

**Repository:** [https://github.com/cloud-engineering-bootcamp/ce-lab-ssh-connection-security](https://github.com/cloud-engineering-bootcamp/ce-lab-ssh-connection-security)

**Activity Type:** Individual  
**Estimated Time:** 30-45 minutes

## Learning Objectives

- [ ] Practice secure SSH connection techniques
- [ ] Configure SSH config file for easier connections
- [ ] Test and troubleshoot security group rules
- [ ] Implement SSH security best practices
- [ ] Verify instance accessibility from different contexts

## Prerequisites

- [ ] Completed Lab M2.01 (EC2 instance running)
- [ ] SSH key pair from previous lab
- [ ] Basic understanding of security groups

---

## Introduction

In this lab, you'll level up your SSH skills and learn professional practices for managing remote server access. You'll configure your SSH client for convenience while maintaining security, and test various connection scenarios.

## Scenario

Your manager wants you to demonstrate that you can:
- Connect to servers quickly without remembering IPs
- Ensure only authorized IPs can access servers
- Troubleshoot connectivity issues
- Document your security configurations

---

## Your Task

**What you'll accomplish:**
- Configure SSH config file for easy connections
- Test security group rules by modifying them
- Practice troubleshooting connection issues
- Create comprehensive security documentation

**Success criteria:**
- [ ] SSH config file working correctly
- [ ] Can connect using alias instead of full command
- [ ] Tested security group restrictions
- [ ] Documented all security measures
- [ ] Created troubleshooting guide

**Time limit:** 30-45 minutes

---

## Step-by-Step Instructions

### Step 1: Create SSH Config File

Make SSH connections easier with a config file.

1. **Create/edit SSH config:**
   ```bash
   # Create .ssh directory if it doesn't exist
   mkdir -p ~/.ssh
   
   # Edit config file
   nano ~/.ssh/config
   ```

2. **Add this configuration:**
   ```
   # Week 2 EC2 Instance
   Host bootcamp-web
       HostName YOUR_PUBLIC_IP
       User ec2-user
       IdentityFile ~/.ssh/bootcamp-week2-key.pem
       ServerAliveInterval 60
       ServerAliveCountMax 3
   ```
   
   Replace `YOUR_PUBLIC_IP` with your actual EC2 instance public IP.

3. **Save and exit:**
   - Press `Ctrl + O` to save
   - Press `Enter` to confirm
   - Press `Ctrl + X` to exit

4. **Set proper permissions:**
   ```bash
   chmod 600 ~/.ssh/config
   ```

5. **Test the new alias:**
   ```bash
   # Now you can connect with just:
   ssh bootcamp-web
   
   # Instead of:
   ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@YOUR_PUBLIC_IP
   ```

**Expected outcome:** You can connect using the simple alias `ssh bootcamp-web`.

---

### Step 2: Test Security Group Restrictions

Now let's verify your security group is actually securing your instance.

#### Test 1: SSH from Allowed IP

```bash
# This should work (your IP is allowed)
ssh bootcamp-web
```

**Expected:** ✅ Connection successful

#### Test 2: HTTP from Anywhere

```bash
# Test HTTP access
curl -I http://YOUR_PUBLIC_IP

# Or visit in browser
open http://YOUR_PUBLIC_IP
```

**Expected:** ✅ Returns HTTP 200 OK

#### Test 3: Try Unsupported Protocol

```bash
# Try to ping (ICMP protocol not allowed in security group)
ping YOUR_PUBLIC_IP
```

**Expected:** ❌ Request timeout (this is good - ICMP is blocked!)

#### Test 4: Try Blocked Port

```bash
# Try to connect to MySQL port (not in security group)
telnet YOUR_PUBLIC_IP 3306
# or
nc -zv YOUR_PUBLIC_IP 3306
```

**Expected:** ❌ Connection refused or timeout (port 3306 not allowed)

**Document your test results in a file called `security-test-results.txt`**

---

### Step 3: Modify Security Group Rules

Practice adding and removing rules.

#### Add HTTPS Support

1. **Go to EC2 Console** → Security Groups → Select `week2-web-server-sg`

2. **Inbound rules** → **Edit inbound rules** → **Add rule**
   - **Type:** HTTPS
   - **Port:** 443
   - **Source:** 0.0.0.0/0
   - **Description:** `HTTPS access for testing`

3. **Save rules**

4. **Test (will fail because no HTTPS service running):**
   ```bash
   curl -I https://YOUR_PUBLIC_IP
   # Will timeout - no service on 443
   ```

#### Remove the HTTPS Rule

1. **Edit inbound rules** → Find HTTPS rule → **Delete** → **Save**

2. **Verify rule is gone** - Take screenshot

---

### Step 4: Configure SSH Connection Timeouts

Keep your SSH connection alive even during idle periods.

1. **On your local machine**, edit SSH config:
   ```bash
   nano ~/.ssh/config
   ```

2. **Add keep-alive settings** to your host entry:
   ```
   Host bootcamp-web
       HostName 3.236.59.249
       User ec2-user
       IdentityFile ~/.ssh/bootcamp-week2-key.pem
       ServerAliveInterval 60
       ServerAliveCountMax 3
       TCPKeepAlive yes
       Compression yes
   ```

3. **What these settings mean:**
   - `ServerAliveInterval 60`: Send keep-alive every 60 seconds
   - `ServerAliveCountMax 3`: Disconnect after 3 failed keep-alives
   - `TCPKeepAlive yes`: Enable TCP keep-alive
   - `Compression yes`: Compress data (faster on slow connections)

4. **Save and test:**
   ```bash
   ssh bootcamp-web
   # Leave connection idle for 2 minutes
   # Should stay connected!
   ```

---

### Step 5: Practice SCP (Secure Copy)

Transfer files between your computer and EC2 instance.

#### Copy TO Instance

```bash
# Create a test file locally
echo "Hello from my computer!" > test-file.txt

# Copy to instance
scp test-file.txt bootcamp-web:~/

# Verify on instance
ssh bootcamp-web "cat ~/test-file.txt"
```

#### Copy FROM Instance

```bash
# Copy instance-info from instance to local
scp bootcamp-web:~/instance-info.txt ./downloaded-info.txt

# Verify locally
cat downloaded-info.txt
```

#### Copy Directory

```bash
# Create directory with files locally
mkdir my-website
echo "<h1>My Site</h1>" > my-website/index.html
echo "body { background: lightblue; }" > my-website/style.css

# Copy entire directory
scp -r my-website bootcamp-web:~/

# Verify on instance
ssh bootcamp-web "ls -la ~/my-website/"
```

**Document your SCP commands and results in `file-transfer-log.txt`**

---

### Step 6: Harden SSH Configuration (On Instance)

Make the SSH server itself more secure.

1. **Connect to your instance:**
   ```bash
   ssh bootcamp-web
   ```

2. **Backup SSH config:**
   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
   ```

3. **View current SSH configuration:**
   ```bash
   sudo cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"
   ```

4. **Check important security settings:**
   ```bash
   # Should be 'no' (password login disabled)
   grep "^PasswordAuthentication" /etc/ssh/sshd_config
   
   # Should be 'no' (root can't SSH directly)
   grep "^PermitRootLogin" /etc/ssh/sshd_config
   
   # Should be 'yes' (pubkey auth enabled)
   grep "^PubkeyAuthentication" /etc/ssh/sshd_config
   ```

5. **Verify these security settings are in place:**
   - ✅ `PasswordAuthentication no` - Only key-based auth
   - ✅ `PermitRootLogin no` - Root can't login directly
   - ✅ `PubkeyAuthentication yes` - SSH keys enabled

**DO NOT modify the sshd_config in this lab** - just verify settings.

**Save the output to `ssh-config-audit.txt`**

---

### Step 7: Create Connection Aliases with Scripts

Create a simple script for common operations.

1. **On your local machine**, create a script:
   ```bash
   nano ~/connect-bootcamp.sh
   ```

2. **Add this content:**
   ```bash
   #!/bin/bash
   # Quick connection script for bootcamp instance
   
   # Colors for output
   GREEN='\033[0;32m'
   BLUE='\033[0;34m'
   NC='\033[0m' # No Color
   
   echo -e "${BLUE}Connecting to bootcamp instance...${NC}"
   ssh bootcamp-web
   ```

3. **Make it executable:**
   ```bash
   chmod +x ~/connect-bootcamp.sh
   ```

4. **Test it:**
   ```bash
   ~/connect-bootcamp.sh
   ```

5. **(Optional) Add to PATH:**
   ```bash
   # Add to ~/.zshrc or ~/.bashrc
   echo 'export PATH="$HOME:$PATH"' >> ~/.zshrc
   
   # Then you can run from anywhere:
   connect-bootcamp.sh
   ```

---

### Step 8: Security Group Audit

Document your security group configuration comprehensively.

1. **Via AWS CLI** (if installed):
   ```bash
   # Get detailed security group info
   aws ec2 describe-security-groups \
       --group-names week2-web-server-sg \
       --output json > security-group-audit.json
   
   # Get formatted table view
   aws ec2 describe-security-groups \
       --group-names week2-web-server-sg \
       --output table > security-group-audit.txt
   ```

2. **Create a security matrix** in `security-matrix.md`:

   ```markdown
   # Security Configuration Matrix
   
   ## Instance Details
   - Instance ID: i-xxxxxxxxxxxxx
   - Instance Type: t3.micro
   - Public IP: x.x.x.x
   - Private IP: x.x.x.x
   
   ## Security Group: week2-web-server-sg
   
   ### Inbound Rules
   | Protocol | Port | Source | Purpose | Risk Level |
   |----------|------|--------|---------|------------|
   | TCP | 22 | YOUR_IP/32 | SSH admin access | Low (restricted IP) |
   | TCP | 80 | 0.0.0.0/0 | Public web traffic | Low (expected) |
   
   ### Outbound Rules
   | Protocol | Port | Destination | Purpose |
   |----------|------|-------------|---------|
   | All | All | 0.0.0.0/0 | Unrestricted egress |
   
   ## Security Assessment
   - ✅ SSH restricted to specific IP
   - ✅ No unnecessary ports open
   - ✅ HTTP enabled for web server purpose
   - ⚠️ Consider: Restrict outbound traffic
   
   ## Recommendations
   1. Add monitoring alerts for SSH attempts
   2. Consider using Session Manager instead of SSH
   3. Enable VPC Flow Logs
   ```

---

## What to Submit

Update your GitHub repository `ce-lab-launch-ec2-instance` or create `ce-lab-ssh-security`:

### Required Files

**1. README.md** - Update with:
- SSH configuration process
- Security group testing results
- SCP examples and screenshots
- Challenges and solutions

**2. Configuration files:**
- `ssh-config.txt` - Your SSH config file (redact sensitive IPs)
- `security-test-results.txt` - Results of your security tests
- `file-transfer-log.txt` - SCP commands and results
- `ssh-config-audit.txt` - SSH daemon configuration check
- `security-matrix.md` - Complete security documentation

**3. Scripts:**
- `connect-bootcamp.sh` - Your connection script

**4. Additional screenshots:**
- `06-ssh-config-working.png` - Connecting via alias
- `07-security-group-edit.png` - Adding/removing HTTPS rule
- `08-scp-file-transfer.png` - Successful file transfer

---

## Bonus Challenges

### Challenge 1: Restrict Outbound Traffic

Modify security group to only allow necessary outbound:
- HTTPS (443) for updates
- HTTP (80) for package downloads
- Document what breaks and why

### Challenge 2: Multiple Security Groups

- Create a second security group for application-specific rules
- Attach both to your instance
- Document how multiple SGs work together

### Challenge 3: Connection Multiplexing

Speed up subsequent SSH connections:

```bash
# Add to ~/.ssh/config
Host bootcamp-web
    HostName YOUR_PUBLIC_IP
    User ec2-user
    IdentityFile ~/.ssh/bootcamp-week2-key.pem
    ControlMaster auto
    ControlPath ~/.ssh/control-%r@%h:%p
    ControlPersist 10m
```

Test connection speed improvement!

### Challenge 4: SSH Port Forwarding

Forward a port from instance to your local machine:

```bash
# Forward remote port 8080 to local port 8080
ssh -L 8080:localhost:8080 bootcamp-web

# Now access http://localhost:8080 in your browser
# It will reach the instance's port 8080
```

Use case: Access services not exposed to internet

---

## Common SSH Errors Decoded

### 1. Permission Denied (Publickey)

```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

**Causes:**
- Wrong key file
- Wrong username
- Key permissions too open
- Public key not on instance

**Debug:**
```bash
# Verbose SSH output
ssh -vvv -i ~/.ssh/bootcamp-week2-key.pem ec2-user@IP

# Look for lines like:
# "Offering public key: /path/to/key"
# "Server accepts key: pkalg ssh-rsa"
```

### 2. Connection Timed Out

```
ssh: connect to host 54.123.45.67 port 22: Connection timed out
```

**Causes:**
- Security group doesn't allow port 22
- Network ACL blocking
- Instance stopped
- Wrong IP address

**Debug:**
```bash
# Check instance status
aws ec2 describe-instances --instance-ids i-xxxxx

# Test if port is reachable
nc -zv 54.123.45.67 22
```

### 3. Host Key Verification Failed

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

**Causes:**
- Instance was terminated and new one has same IP
- Instance was restored from snapshot
- Man-in-the-middle attack (rare)

**Fix (if you launched a new instance):**
```bash
# Remove old host key
ssh-keygen -R YOUR_PUBLIC_IP

# Connect again (will prompt to verify new key)
ssh bootcamp-web
```

---

## SSH Security Checklist

Use this checklist to audit your SSH security:

### On Your Local Machine

- [ ] Private key permissions set to 400
- [ ] Private keys NOT in cloud-synced folders
- [ ] Private keys NOT committed to Git
- [ ] Different keys for different environments
- [ ] SSH config file has proper permissions (600)
- [ ] Known_hosts file being maintained

**Verify:**
```bash
# Check key permissions
ls -l ~/.ssh/*.pem

# Check config permissions
ls -l ~/.ssh/config

# Check for keys in Git
git status  # Should not show .pem files
```

### On EC2 Instance

- [ ] PasswordAuthentication disabled
- [ ] PermitRootLogin disabled
- [ ] PubkeyAuthentication enabled
- [ ] Security group limits SSH to specific IPs
- [ ] fail2ban installed (optional but recommended)

**Verify on instance:**
```bash
# Check SSH daemon config
sudo sshd -T | grep -i "passwordauthentication\|permitrootlogin\|pubkeyauthentication"

# Check security group
aws ec2 describe-security-groups \
    --group-ids $(ec2-metadata --security-groups | cut -d ' ' -f 2)
```

---

## Step 9: Create Troubleshooting Flowchart

Document common issues and solutions.

Create `ssh-troubleshooting-guide.md`:

```markdown
# SSH Connection Troubleshooting Guide

## Decision Tree

1. **Can you ping the instance?**
   - ❌ No → Check if instance is running, verify IP address
   - ✅ Yes → Continue

2. **Does `nc -zv IP 22` connect?**
   - ❌ No → Check security group allows port 22 from your IP
   - ✅ Yes → Continue

3. **What error do you get when SSH'ing?**
   - "Permission denied" → Check key file and username
   - "Connection refused" → Check SSH service is running on instance
   - "Connection timed out" → Check security group and network ACL
   - "Host key verification failed" → Remove old key from known_hosts

## Quick Checks

```bash
# 1. Verify key permissions
ls -l ~/.ssh/bootcamp-week2-key.pem
# Should show: -r--------

# 2. Verify correct key and user
ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@IP

# 3. Check security group
aws ec2 describe-security-groups --group-ids sg-xxxxx

# 4. Test port connectivity
nc -zv YOUR_PUBLIC_IP 22

# 5. SSH with verbose output
ssh -vvv bootcamp-web 2>&1 | tee ssh-debug.log
```

## Common Fixes

### Fix 1: Key Permissions
\```bash
chmod 400 ~/.ssh/bootcamp-week2-key.pem
\```

### Fix 2: Update Security Group
- Add rule: SSH (22) from YOUR_IP/32

### Fix 3: Verify Instance Running
- Check EC2 console - status should be "Running"
- Check Public IPv4 address hasn't changed

### Fix 4: Remove Old Host Key
\```bash
ssh-keygen -R YOUR_PUBLIC_IP
\```
```

---

## Step 10: Security Best Practices Implementation

Implement and document advanced security measures.

### 1. Change SSH Port (Advanced)

**Why:** Reduces automated attacks on default port 22

**How:**
```bash
# On instance
sudo nano /etc/ssh/sshd_config

# Change line:
# Port 22
to
Port 2222

# Restart SSH
sudo systemctl restart sshd

# Update security group to allow port 2222
# Connect with:
ssh -p 2222 bootcamp-web
```

**⚠️ Warning:** Don't do this in lab unless you're confident! You could lock yourself out.

### 2. Enable MFA for SSH (Advanced)

**Why:** Two-factor authentication adds extra security

**How:** Use google-authenticator package (requires setup)

### 3. Set Up Session Logging

```bash
# On instance - log all commands
sudo nano /etc/profile.d/session-logging.sh
```

Add:
```bash
# Log all user commands
export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" )"'
```

---

## What to Submit

Add to your GitHub repository:

### Required Files

**1. README.md** - Add section for Lab M2.02:
- SSH config configuration
- Security testing results
- SCP examples
- Troubleshooting you encountered

**2. Configuration and documentation:**
- `ssh-config.txt` - Your SSH config (redact sensitive IPs if public)
- `security-test-results.txt` - Results of all security tests
- `file-transfer-log.txt` - SCP commands used
- `ssh-troubleshooting-guide.md` - Your troubleshooting guide
- `security-matrix.md` - Security configuration matrix
- `connect-bootcamp.sh` - Your connection script

**3. Additional screenshots:**
- `09-ssh-config-alias.png` - Connecting via alias
- `10-security-tests.png` - Terminal showing security tests
- `11-scp-file-transfer.png` - File transfer with SCP

---

## Validation Checklist

Before submitting:

- [ ] Can connect using SSH config alias
- [ ] All security tests performed and documented
- [ ] File transfers tested (both directions)
- [ ] SSH security settings verified
- [ ] Troubleshooting guide created
- [ ] All screenshots clear and labeled
- [ ] Repository well-organized
- [ ] README explains your process clearly

---

## Grading Rubric

| Criteria | Points | Requirements |
|----------|--------|--------------|
| **SSH Configuration** | 20 | Config file working, aliases functional |
| **Security Testing** | 25 | All tests performed, results documented |
| **File Transfers** | 15 | SCP working, examples provided |
| **Troubleshooting Guide** | 20 | Comprehensive, clear, actionable |
| **Documentation** | 20 | Complete, well-organized, professional |
| **Total** | **100** | |

---

## Real-World Applications

**These SSH skills are used daily by cloud engineers for:**
- Deploying code to servers
- Troubleshooting production issues
- Running maintenance tasks
- Configuring servers
- Transferring logs and data
- Managing multiple cloud environments

**Pro tip:** Build a personal SSH config with all your servers for instant access to any environment!

---

## Next Steps

In tomorrow's lab, you'll:
- Create more complex security group configurations
- Set up a multi-tier application architecture
- Configure security groups to reference each other
- Test traffic flow between tiers

**Keep your instance running (or stopped) - you'll need it tomorrow!**

---

## Time Tracking

- [ ] SSH config setup: _____ minutes
- [ ] Security testing: _____ minutes
- [ ] SCP practice: _____ minutes
- [ ] Documentation: _____ minutes

**Total time:** _____ minutes

---

## 📤 Submission

**Submission Type:** File Upload (ZIP)

Create a ZIP file containing:
1. **SSH Config File** (`~/.ssh/config`) - Your configured SSH settings
2. **Screenshots** (in `screenshots/` folder):
   - Security group rules showing port 22 configuration
   - Successful SSH connection
   - SCP file transfer completion
   - SSH hardening settings (if completed)
3. **Lab Documentation** (`README.md`):
   - Your EC2 instance details (public IP, security group ID)
   - SSH connection command you used
   - SCP commands used for file transfer
   - Troubleshooting steps you encountered (if any)
   - Answers to reflection questions

**File Structure:**
```
lab-m2-ssh-security.zip
├── README.md
├── ssh-config.txt (copy of your ~/.ssh/config)
└── screenshots/
    ├── 01-security-group-ssh-rule.png
    ├── 02-ssh-connection-success.png
    ├── 03-scp-file-transfer.png
    └── 04-ssh-hardening.png (optional)
```

Upload the ZIP file to the course platform.

---

## Additional Resources

**SSH Documentation:**
- [OpenSSH Config File](https://www.ssh.com/academy/ssh/config)
- [SSH Best Practices](https://www.ssh.com/academy/ssh/best-practices)

**AWS Documentation:**
- [Connect to Linux Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
- [Security Group Rules](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules.html)

**Tools:**
- [SSH Config Generator](https://www.ssh.com/academy/ssh/config)
- [Security Group Analyzer](https://console.aws.amazon.com/vpc/home#SecurityGroups:)

Great work on mastering SSH and security groups! 🔐
