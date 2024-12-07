## Purple Team Cloud Lab Overview
## Overview
The Purple Team Cloud Lab is a comprehensive, cloud-based Active Directory (AD) lab environment designed to simulate real-world cybersecurity scenarios. This lab facilitates attack simulations and the development of detection rules, enabling a controlled environment for threat hunting and incident response practices. 
This lab is hosted on AWS Cloud using Terraform for automated provisioning and includes:
1.	AD Domain: Two machines (Domain Controller and Workstation) running Windows Server 2019 with Sysmon and Winlogbeat installed for log collection.
   
3.	Blue Team Environment: Features Threat Hunting ELK (HELK) for monitoring, analysis, and rule creation.
   
5.	Red Team Network: Equipped with Caldera for simulating MITRE ATT&CK techniques and includes Atomic Red Team for running pre-configured atomic tests.

## Lab Components
1.	Active Directory Domain:
o	Built using PowerShell scripts, the domain includes:
	Domain Controller (DC): Acts as the primary AD controller and DNS server.
	Workstation: Simulates a standard endpoint joined to the domain.
o	Tools installed:
	Sysmon & Winlogbeat for detailed event logging.
	Atomic Red Team for red team simulations.
	Other utilities like Wireshark, Chrome, and Powershell Remoting.
2.	Blue Team Machine (HELK):
o	Configured with:
	HELK (Hunting ELK): ElasticSearch, Kibana, Logstash, and ElastAlert running in Docker containers for log aggregation and analysis.
	Detection Tools: Sigma rules for creating new detections and alerts.
	Forensic Tools: Includes Kansa, Kape, and Volatility for digital forensics and incident response (DFIR).
3.	Red Team Machine (Caldera):
o	Includes:
	MITRE’s Caldera for simulating advanced attack techniques.
	Options for backdoor deployment, remote command execution, and over 800 MITRE ATT&CK simulations.
o	Alternate option to simulate techniques using Atomic Red Team.

## Lab Capabilities
The lab provides a platform for:

•	Simulating attacks to observe their impact on system logs and defenses.

•	Developing and testing detection rules.

•	Testing endpoint detection and response (EDR) solutions.

•	Purple Team exercises: Integrating red and blue team activities to enhance organizational defenses.

•	Active Directory lab scenarios for malware analysis, reverse engineering, or SIEM use case testing.

## Deployment Instructions
Prerequisites
To set up the lab, ensure you have:

•	An AWS Account

•	A Windows host machine (or a Windows VM) to run PowerShell scripts.

•	AWS CLI installed.

•	Terraform added to any folder in your system’s %PATH% environment.

## Installation Steps
1.	Prepare AWS:
   
o	Create an AWS user with administrator permissions and obtain the ACCESS KEY and SECRET KEY.

o	Run aws configure in the command prompt and provide the keys. Set the region to eu-west-1.

o	In the AWS console, switch to the eu-west-1 region.

o	Navigate to EC2 → Key Pair, create a key named ec2_key_pair (lowercase), and save the .pem file.

2.	Configure SSH Key:
   
o	Copy the .pem file to C:\Users\<your-username>\.ssh and rename it to id_rsa.

3.	Deploy the Lab:

Open the command prompt, navigate to the project folder, and run:

            terraform init  

            terraform apply

•  Confirm with "yes" when prompted. The setup will take approximately 40 minutes.

4.	  Access Your Lab:
   
•	Use the provided credentials (see below) to access different machines.

•	Remember, keeping the lab running will incur AWS costs.

5.	Cleanup:
   
•	To destroy the lab, run:

            terraform destroy --auto-approve  

## Credentials:
A. To access the domain, you can use these credentials:

•	Administrator, Pass:LabPass1: this is the local admin account, you can use it remotely with RDP or Powershell remoting

•	adlab\ddean, Pass:LabPass1: This is the domain admin account, you can use it to access both machines as a domain admin

•	adlab\kbaehr, Pass:LabPass1: This is the workstation user, it doesn't have admin privileges (Possible privilege escalation?). Feel free to add him to the local admin group if you want to test UAC bypasses

B. To access the blue team machine HELK:

•	SSH: you can connect using ssh ec2-user@<blueteam public ip> and it will use the key pair you moved to .ssh folder and renamed to id_rsa

•	From the browser: use the machine public IP and connect to it using https from your browser with credentials: helk/LabPass1

C. To access the red team caldera machine:

•	SSH: you can connect using ssh ec2-user@<redteam public IP>

•	From the browser: connect using HTTP to port 8888. The credentials are red/LabPass1 or blue/LabPass1.

## Capabilities:
## ADLAB Domain

The ADLAB domain is a very simple domains. It includes the DC which works as a domain controller and a DNS server. The domain has 3 users. And it has these tools installed:

•	Sysmon and winlogbeat for Log Analysis

•	Powershell remoting installed and configured to use (with self-signed certificates)

•	Atomic Red Team installed (for red team simulations)

•	Wireshark installed using chocolatey

•	Chrome installed as well.

The whole domain is created using Powershell scripts which makes the creation highly customizable and a good learning material. Feel free to read, modify, and update Setup-AD.ps1 and Setup-Workstation.ps1 scripts.
Also, feel free to add additional machines and users and execute Setup-Workstaion.ps1 or a modified version of it for each additional machine you add to the AD
I decided to not use Ansible in this project to make it a good learning material for you. But, with Ansible, it will be much easier to develop.

## Blue Team HELK Machine
This machine is created not just for log analysis but also for more in-depth investigation. It includes:

•	Threat Hunting ELK: This has elasticsearch, kibana, logstash and elastialert. All running docker images receive all the sysmon and the other logs using winlogbeat.

•	Sigma Rules: Sigma is useful in creating new alerts or new detections, you can compile them for elastialert. Please follow HELK documentation for how to do that.

•	Powershell Remoting: Powershell remoting installation is quite tricky to work on Linux but it's here and working like a charm. It's very useful to connect directly to ADLAB domain and perform some actions as a blue teamer to investigate attacks or transfer files to the ADLAB domain.

•	Kansa & Kape: Both these tools are known for DFIR and remote triage. Kansa.ps1 won't work properly on Linux but you can execute its modules directly on the AD machines using PowerShell remoting

•	Volatility: volatility is a known tool for memory forensics and here we have volatility and volatility3 (with its symbols) installed by default with all their additional modules

•	Dumpit: Dumpit is a known tool to create a memory dump on Windows machine, it's included inside the tools folder, transfer it to the AD machines using Powershell remoting, and collects the memory dump back

•	Python2 and Python3: We have both python2 & 3 installed, just be careful when you are using Python or pip

To use PowerShell remoting:
This script will connect to ADLAB Domain controller, feel free to update it to connect to the workstation or to use another user.

         $DefaultPassword = "LabPass1"
         $IPAddr = "192.168.10.100"
         $securePassword = ConvertTo-SecureString -AsPlainText -Force $DefaultPassword
         $ddean = New-Object System.Management.Automation.PSCredential "adlab\ddean", $securePassword
         Enter-PSSession -ComputerName $IPAddr -Authentication Negotiate -Credential $ddean

## Red Team Caldera
This machine has Caldera installed. Using Caldera, you can drop a backdoor into your ADLAB machines and use it to control the domain, execute PowerShell commands remotely, and simulate over 800 MITRE ATT&CK techniques. Caldera has up to 4 different types of backdoors and it's built by MITRE organization itself.

## Use Cases
•	Simulating different attacks and see what type of footprint they leave on the logs or on the system

•	Test different attack techniques and develop detections for them

•	Customize the ADLAB environment to simulate your organization environment, test different attack techniques and look for gaps in your organization logs and your detections (Purple teaming)

•	EDR Testing lab

•	Product Security Lab

•	Enterprise Active Directory lab with domain-joined devices

•	Malware/ reverse engineering to study artifacts against domain-joined devices

•	SIEM / Threat Hunting / DFIR / Live Response lab with HELK



