# 5G Threat Detection Simulation on AWS

![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20GuardDuty%20%7C%20CloudTrail-FF9900?logo=amazonaws&logoColor=white)
![Open5GS](https://img.shields.io/badge/5G%20Core-Open5GS-1B2A4A)
![srsRAN](https://img.shields.io/badge/RAN-srsRAN-1B2A4A)
![Linux](https://img.shields.io/badge/Linux-Ubuntu%20%7C%20Amazon%20Linux-FCC624?logo=linux&logoColor=black)
![Status](https://img.shields.io/badge/MSc%20Thesis-National%20College%20of%20Ireland-2EA44F)

MSc Cybersecurity thesis project (National College of Ireland): a simulated private 5G network built on AWS, used to evaluate how well Amazon GuardDuty detects real attacks against telecom infrastructure.

**At a glance**
- Simulated a full private 5G stack on AWS — Open5GS core network + srsRAN radio access network — across a dedicated multi-subnet VPC.
- Launched real attack scenarios (port/network scanning, SSH brute-force, DNS exfiltration) against the environment.
- Amazon GuardDuty logged **350+ findings**, including high-severity **TorRelay** and **DNS exfiltration** activity, across UnauthorizedAccess, Recon and PrivilegeEscalation categories.
- Full write-up, methodology and screenshots in [final.pdf](final.pdf).

## Overview
This project demonstrates the setup of a simulated private 5G network on Amazon Web Services (AWS) to evaluate the real-time threat detection capabilities of Amazon GuardDuty. The environment uses Open5GS for core network functions and srsRAN for radio access network (RAN) simulation. It includes configurations for threat simulation scenarios, such as network scans, brute-force attacks, and DNS exfiltration, to test GuardDuty's detection accuracy, response time, and performance overhead.

This setup provides a secure, cloud-native testbed for integrating 5G networks with AWS security services, making it ideal for exploring cybersecurity in modern telecom infrastructures.

**Key Skills Demonstrated:**
- AWS Services (EC2, VPC, GuardDuty, CloudTrail, VPC Flow Logs)
- 5G Network Simulation (Open5GS, srsRAN)
- Network Security and Threat Simulation (nmap, SSH brute-force, DNS exfiltration)
- Linux System Administration (Ubuntu, Amazon Linux)
- DevOps Tools (Git, Meson, Ninja, CMake)

## System Requirements

### Hardware Requirements
| Component     | Instance Type | Storage      | Operating System       | Network CIDR | Subnets          |
|---------------|---------------|--------------|------------------------|--------------|------------------|
| Core Instance | t2.medium     | 20 GiB (gp2) | Ubuntu Server 20.04 LTS| 10.0.0.0/16  | Core: 10.0.1.0/24|
| RAN Instance  | t2.medium     | 20 GiB (gp2) | Ubuntu Server 20.04 LTS| 10.0.0.0/16  | RAN: 10.0.2.0/24 |

### Software Requirements
| Component     | Required Software                     |
|---------------|---------------------------------------|
| Core Instance | Open5GS, MongoDB, build-essential, meson, ninja-build |
| RAN Instance  | srsRAN, libuhd-dev, cmake, gcc        |

### Dependencies
| Component     | Required Tools   |
|---------------|------------------|
| Core Instance | net-tools, curl  |
| RAN Instance  | nmap, iperf3     |

### AWS Services and Policies
- **IAM Policies:** AmazonEC2FullAccess, AmazonVPCFullAccess, AmazonGuardDutyFullAccess, CloudWatchLogsFullAccess
- **Networking:** VPC with Internet Gateway, Security Groups for Core and RAN Subnets

## Setup Instructions
Follow these steps to reproduce the environment. For detailed reproduction, refer to the [Appendix](#appendix-detail-instructions-for-result-reproduction) or the attached [final.pdf](final.pdf).

### 1. Create VPC and Subnets
- VPC: MVP-5G-VPC (CIDR: 10.0.0.0/16)
- Core Subnet: MVP-5G-Core-Subnet (CIDR: 10.0.1.0/24)
- RAN Subnet: MVP-5G-RAN-Subnet (CIDR: 10.0.2.0/24)

### 2. Create and Attach Internet Gateway
- Internet Gateway: MVP-5G-IGW attached to MVP-5G-VPC

### 3. Configure Route Tables
- Core Route Table: MVP-5G-Core-RT with route 0.0.0.0/0 to MVP-5G-IGW, associated with MVP-5G-Core-Subnet

### 4. Launch EC2 Instances
- **Core Instance:** MVP-5G-Core-Instance (Amazon Linux 2, t2.micro, MVP-5G-Core-Subnet, Security: SSH 22, HTTP 80)
- **RAN Instance:** MVP-5G-RAN-Instance (Amazon Linux 2, t2.micro, MVP-5G-RAN-Subnet, Security: SSH 22, Custom TCP 8080)

### 5. Install and Configure Software on Core Instance
- SSH into Core: `ssh -i AmmadMVPKeyPair.pem ec2-user@<core-ip>`
- Update: `sudo yum update -y`
- Install Apache: `sudo yum install httpd -y`
- Start HTTP: `sudo systemctl start httpd && sudo systemctl enable httpd`
- Create Web Page: `echo "<h1>MVP 5G Core Instance</h1>" > /var/www/html/index.html`

For Open5GS setup (on Ubuntu instance):
- Install Dependencies: `sudo apt install -y build-essential meson ninja-build ...` (full list in PDF)
- Clone and Build Open5GS: `git clone https://github.com/open5gs/open5gs.git && cd open5gs && meson build --prefix=`pwd`/install && ninja -C build install`
- Start MongoDB: `sudo apt install -y mongodb && sudo systemctl start mongodb`
- Add Subscriber: `./open5gs-dbctl add 001010123456789 123456789012345`
- Start Services: `./open5gs-mmed & ./open5gs-smfd & ...`

### 6. Install and Configure Software on RAN Instance
- SSH into RAN: `ssh -i AmmadMVPKeyPair.pem ec2-user@<ran-ip>`
- Update: `sudo yum update -y`
- Install curl: `sudo yum install curl -y`

For srsRAN setup:
- Install Dependencies: `sudo apt install -y git build-essential cmake libconfig++-dev`
- Clone and Build: `git clone https://github.com/srsran/srsRAN.git && cd srsRAN && mkdir build && cd build && cmake ../ && make && sudo make install`
- Configure gNB: Edit `/usr/local/etc/srsran/gnb.conf` to set core IP
- Start UE Simulator: `sudo srsue`

### 7. Simulate Network Traffic and Threats
- From RAN to Core: `curl http://<core-ip>` or loop: `while true; do curl http://<core-ip>; sleep 1; done`
- Install nmap: `sudo yum install nmap -y`
- Port Scan: `nmap -sS -Pn <core-ip>`
- SSH Brute Force: `ssh ec2-user@<core-ip>` (simulate failed attempts)
- DNS Exfiltration: `sudo yum install bind-utils -y && dig baddomain.example.com && nc -zv <core-ip> 23-25`

### 8. Enable and Monitor GuardDuty
- Enable GuardDuty in AWS Console
- Monitor findings for detected threats (e.g., TorRelay, Recon, UnauthorizedAccess)

## Results
- GuardDuty detected over 350 findings, including high-severity threats like TorRelay and DNS exfiltration.
- Common finding types: UnauthorizedAccess, Recon, PrivilegeEscalation.
- Graphs and summaries available in [final.pdf](final.pdf), showing severity distribution and attack details.

## References
1. Open5GS Repository: https://github.com/open5gs/open5gs
2. MongoDB Community Edition: https://www.mongodb.com/docs/manual/installation/
3. srsRAN Repository: https://github.com/srsran/srsRAN
4. Ubuntu Server 20.04 LTS: https://ubuntu.com/download/server
5. Build Tools: https://packages.ubuntu.com/
6. Meson Build System: https://mesonbuild.com/
7. Ninja Build System: https://ninja-build.org/
8. BIND Utilities (dig): https://linux.die.net/man/1/dig
9. Netcat (nc): https://netcat.sourceforge.net/

## Appendix: Detail Instructions for Result Reproduction
| Instructions | Code/Output |
|--------------|-------------|
| Using Root Privileges | - Access type: AWS Management Console access<br>- Policies: AmazonEC2FullAccess, AmazonVPCFullAccess, AmazonGuardDutyFullAccess, CloudWatchLogsFullAccess |
| Create VPC to Simulate 5G Network | MVP-5G-VPC<br>10.0.0.0/16 |
| Core Subnet | MVP-5G-Core-Subnet<br>10.0.1.0/24 |
| RAN Subnet | MVP-5G-RAN-Subnet<br>10.0.2.0/24 |
| Create and Attach Internet Gateway | MVP-5G-IGW<br>MVP-5G-VPC |
| Configure Route Tables | MVP-5G-Core-RT<br>Destination 0.0.0.0/0 → Target MVP-5G-IGW<br>MVP-5G-Core-Subnet |
| Launch EC2 Instances | Core: MVP-5G-Core-Instance, Amazon Linux 2, t2.micro, MVP-5G-Core-Subnet, SSH(22), HTTP(80)<br>RAN: MVP-5G-RAN-Instance, Amazon Linux 2, t2.micro, MVP-5G-RAN-Subnet, SSH(22), Custom TCP(8080) |
| ... (Full appendix in PDF) | ... |

For the complete guide with screenshots and detailed outputs, download [final.pdf](final.pdf).

This setup showcases hands-on experience with cloud security, 5G tech, and threat modeling—perfect for roles in cybersecurity, cloud engineering, or telecom. Feel free to reach out for collaborations!

---

**Ammad Ud Din Bajwa** — MSc Cybersecurity, National College of Ireland · SOC Analyst | Incident Response | Cybersecurity Operations
[LinkedIn](https://www.linkedin.com/in/ammaduddinbajwa) · [GitHub](https://github.com/ammaduddinbajwa) · ammaduddinbajwa702@gmail.com