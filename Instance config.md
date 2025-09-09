# Setting up AWS EC2 Instance

This guide provides a clear, step-by-step process to set up an **AWS EC2 instance**, configure it for use as a **GitHub Actions runner**, install **Nginx**, and secure it with **SSL**.  

---

## ✅ Prerequisites
Before starting, ensure you have:

- An **AWS account** ([Sign up here](https://aws.amazon.com/))
- A **GitHub account** with repository access
- A local machine with:
  - **Windows**: PowerShell (Run as Administrator) + [PuTTY](https://www.putty.org/) (if you want to use `.ppk` instead of `.pem`)
  - **Linux/macOS**: OpenSSH client (usually pre-installed)
- Basic familiarity with terminal/command line
- (Optional) A domain name if you want SSL with your custom domain

---

## 🖥️ Step 1: Launch an AWS EC2 Instance
1. Go to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to **EC2 → Instances → Launch Instance**.
3. Choose an **Amazon Machine Image (AMI)**, e.g. **Ubuntu 22.04 LTS**.
4. Select your **instance type** (e.g., t2.micro for free tier).
5. Create a **key pair**:
   - `.pem` file for Linux/macOS users
   - `.ppk` file for Windows users (convert `.pem` using PuTTYgen if needed)
   - **Download and save it securely** — you’ll need it later.
6. Configure **security group**:
   - Add rule: **SSH** (TCP 22) → Source: Anywhere (IPv4)
   - Add rule: **HTTP** (TCP 80) → Source: Anywhere (IPv4)
   - Add rule: **HTTPS** (TCP 443) → Source: Anywhere (IPv4)
7. Launch the instance.

---

## 🔑 Step 2: Connect to the Instance
1. In the AWS **EC2 Dashboard**, select your instance.
2. Go to the **Connect** tab → choose **SSH client**.
3. Copy the example SSH command, e.g.:

   ```bash
   ssh -i "path/to/your-key.pem" ubuntu@ec2-xx-xxx-xx-xx.compute-1.amazonaws.com

## Step 3: Fix Windows PowerShell Permission Issues (if any)

1. icacls E:\aws\dev.pem /inheritance:r
2. icacls E:\aws\dev.pem /grant:r "$($env:USERNAME):(R)"
3. icacls E:\aws\dev.pem /remove "NT AUTHORITY\Authenticated Users"
4. icacls E:\aws\dev.pem /remove "BUILTIN\Users"
