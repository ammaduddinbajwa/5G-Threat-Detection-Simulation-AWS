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

## Published Materials and Reproducibility

This repository currently contains this README and the thesis report, `final.pdf`. Standalone deployment scripts, Open5GS/srsRAN configurations, and sanitised finding exports are not included. The reported 350+ findings are a thesis result, not independently reproduced by this repository.

The original README mixed an Amazon Linux 2/t2.micro connectivity prototype with an Ubuntu/t2.medium 5G environment. The architecture below describes the stated 5G environment; consult the report for implementation evidence. This README is not a complete deployment guide.

## Reported System Requirements

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

## Environment Overview

- Dedicated VPC with core and RAN subnets.
- Open5GS core and srsRAN components hosted on EC2, as described in the thesis.
- GuardDuty monitoring with supporting AWS logging.
- Reported threat scenarios include reconnaissance, failed SSH authentication, and DNS-related activity.

Exact component versions, configurations, attack commands, and the mapping from each scenario to observed findings should be checked against `final.pdf` before attempting reproduction. A DNS lookup or port scan alone does not demonstrate DNS exfiltration, and the total finding count does not measure detection accuracy or unique attacks.

## Access and Lab Scope

Use an isolated environment you own or are authorised to test. The broad AWS FullAccess policies listed in the report describe the lab setup; they are not a least-privilege production recommendation. Restrict access to the required resources and remove chargeable resources after testing.

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

## Implementation Evidence

See [final.pdf](final.pdf) for the original methodology, screenshots, and reported outputs. Runnable configuration files and sanitised finding data would be needed to make independent reproduction possible.


---

**Ammad Ud Din Bajwa** — MSc Cybersecurity, National College of Ireland · SOC Analyst | Incident Response | Cybersecurity Operations
[LinkedIn](https://www.linkedin.com/in/ammaduddinbajwa) · [GitHub](https://github.com/ammaduddinbajwa) · ammaduddinbajwa702@gmail.com