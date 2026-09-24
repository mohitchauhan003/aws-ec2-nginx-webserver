#AWS EC2 Nginx Web Server

Hosted a custom web page on an Nginx server running on an AWS EC2 (Ubuntu) instance. I secured it with a security group, a UFW firewall and key-based SSH, and verified it by reading the server logs.

Built by: Mohit Chauhan

Architecture
User (Browser) → Internet → Internet Gateway → Route Table → Public Subnet
→ Security Group (ports 22, 80) → EC2 Ubuntu (UFW + Nginx)
Environment
Item	Value
Cloud	AWS
Region	Europe (Stockholm) eu-north-1
Availability Zone	eu-north-1a
Instance type	t3.micro
OS	Ubuntu Server (LTS)
Web server	Nginx
Network	Default VPC + manually created public subnet public-subnet-1a
What I did
Networking: the default VPC had no subnets in the region, so I created public-subnet-1a (172.31.0.0/20) and enabled auto-assign public IPv4. I verified the route table sends 0.0.0.0/0 to the Internet Gateway.
Launched EC2: Ubuntu on a t3.micro, with a key pair (.pem) for SSH login.
Security group: SSH (port 22) allowed only from my own IP, HTTP (port 80) allowed from anywhere.
Connected over SSH using the key file.
Installed Nginx with apt, checked it with systemctl status nginx, and enabled it to start on boot.
Replaced the default page with a custom HTML page.
Enabled UFW firewall, allowing ports 22 and 80 (SSH allowed first to avoid a lockout).
Verified SSH password login is disabled (passwordauthentication no), so access is key-based only.
Read Nginx access logs with tail -f and watched live requests from my browser.
Key commands
bash
# Connect
ssh -i my-ec2-key.pem ubuntu@<public-ip>

# Install and manage Nginx
sudo apt update
sudo apt install nginx -y
sudo systemctl status nginx
sudo systemctl enable nginx

# Firewall (allow SSH before enabling)
sudo ufw allow 22
sudo ufw allow 80
sudo ufw enable
sudo ufw status

# Verify SSH password login is disabled
sudo sshd -T | grep -i passwordauthentication

# Read live access logs
sudo tail -f /var/log/nginx/access.log
Problems I faced and how I fixed them

Problem: "No subnets found" when launching the instance

The default VPC existed but had no subnets in the region, so the Subnet dropdown was empty and I couldn't launch.

Fix: I created a subnet manually in the VPC console, enabled auto-assign public IPv4, and checked that the route table had 0.0.0.0/0 pointing to the Internet Gateway. After refreshing the launch page, the subnet appeared and the instance launched.

What I learned: a server needs a subnet, a route to an Internet Gateway, a public IP and an open security group port to be reachable. If any of these is missing, it can't be reached from the internet.

Screenshots

1. Problem: no subnets found Show Image

2. Fix: subnet with route to the Internet Gateway Show Image

3. EC2 instance running Show Image

4. Security group rules (SSH from my IP, HTTP open) Show Image

5. Connected to the server over SSH Show Image

6. Nginx service active (running) Show Image

7. Default Nginx welcome page Show Image

8. My custom page Show Image

9. UFW firewall active Show Image

10. SSH password authentication disabled Show Image

11. Nginx access logs Show Image

What I learned
Launching and connecting to a Linux server on AWS with key-based SSH
How a VPC, subnet, route table and Internet Gateway work together
The difference between AWS security groups (outside the server) and UFW (inside the server)
Installing, running and managing services with systemctl
Reading web server logs and understanding HTTP status codes (200, 304, 404)
Troubleshooting a real networking problem step by step
Possible improvements
Restrict SSH in UFW to my IP as well
Add HTTPS with Let's Encrypt
Provision the whole setup with Terraform
Add a CI/CD pipeline to deploy page updates automatically

The instance was terminated after the project to avoid charges.
