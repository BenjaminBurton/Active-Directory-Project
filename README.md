# Active-Directory-Project


| Virtual Machine                                            | Associated Role                                    |
| ----------------------------------------------- | ----------------------------------------------------- |
| Kali Linux                          |         <a href="https://github.com/BenjaminBurton/Active-Directory-Project/blob/main/src/kali-linux/README.md">Attack Machine</a>| 
| Splunk w/ Ubuntu                      | <a href="https://github.com/BenjaminBurton/Active-Directory-Project/blob/main/src/splunk-ubuntu/README.md">Log Analysis & EDR</a>|
| Windows 10                                | <a href="https://github.com/BenjaminBurton/Active-Directory-Project/blob/main/src/windows10-Target/README.md"> Target Machine|
| Windows Server 2022                   | <a href="https://github.com/BenjaminBurton/Active-Directory-Project/blob/main/src/windows-server-2022/README.md">Active Directory|

```js
- Kali
- Ubuntu (for our Splunk Server)
- Windows 10 (Target Machine)
- Windows 2022 Server

`The Documentation for each machine will be in src folder`
```

# Logical Diagram for Active Directory

![screen shot](src/images/diagram.png?width=200px)


# AD Documentation 

	Database containing Users Computers Groups and many more
	In order to use Active Directory A server must install a 
	service called Active Directory Domain Services ( AD DS )
	An then the server must then be promoted to a Domain 
	Controller ( DC ) By promoting the server to a DC it will 
	Grant us the capability to perform Authentication Using a 
	protocol called Kerberos and Authorization For our domain.
	With AD DS we will have a database that will Contain Objects
	Such as Users, Computers, Groups and more These Objects 
	will take Attributes which is Information about the object 
	(metadata) Example : Object might be user : Bob with an 
	Attribute of First Name - Bob Last Name - Smith


# Overview of our 5 parts 
	Domain, Network, Splunk Server, Active directory, Attacker


# Domain : 
	Our Domain will have a Logical boundary for organizing and 
	managing resources within the network. Centralizing user 
	account management. Providing authentication and authorization 
	services. Facilitating resource sharing and management.
	
# Network : 
	Routing and switching directing data packets between 
	devices on the network to ensure proper communication.
	Connectivity and establishing connections between servers, 
	client machines and othe network devices.
	Security implementing firewalls, intrusion detection systems, 
	and encryption to protect against unauthorized access and data 
	breach.
	Bandwith management optimizing network performance and 
	bandwith utilization to ensure efficient data transfer.
	
# Splunk Server :
	We will be doing Log collection gathering log files and event 
	data from domain controllers, servers, and client machines.
	Indexing and  organizing log data to facilitate fast 
	and efficient searching. Search and Analysis allowing 
	adminstrators to search, correlate, and analyze log data to 
	identify security incidents, troubleshoot issues, and gain 
	insights into system performance. Generating reports 
	and visualizations to present findings and trends derived from 
	log data analysis.
	
# Active Directory :
	Our AD will provide us with user authentication and verifying 
	the identity of users and computers attempting to access 
	network resources. User authorization determinng which 
	resources users are allowed to access based on their 
	permissions and group memberships. Group policy management 
	enforcing centralized security policies, configurations, and 
	settings across the network. Directory services storing and 
	organizing information about network resources, including 
	users, computers, groups, and policies. Replication 
	synchronizing directory data between domain controllers to 
	ensure consistency and fault tolerance.
	
# Attacker :
	We will be Exploiting vulnerabilities identifying and 
	exploring security weaknesses in network services, 
	applications, or configurations Gaining unauthorized access 
	attempting to obtain unauthorized access to sensetive data, 
	user accounts, or network resources. Conducting reconnaissance 
	gathering information about the network, such as open ports, 
	services, and user accounts, to plan and execute attacks.
	Launch various types of attacks including malware infections, 
	phishing campaigns, brute force attacks, privilege escalation, 
	denial-of-service attacks. Evading detection employing 
	techniques to avoid detection by security measures such as 
	firewalls, intrusion detection systems, or antivirus software.

# Part 1 ( Diagram )

	Create a Logical Diagram ( Using Draw.io )
	Specify Hardware Requirements
		Splunk Server ( Running on Ubuntu 22.04 )
			IP : 172.16.188.0
		
		Active Directory Server ( Windows Server 2022 )
			IP : 172.16.188.10
			Splunk Universal Forwarder
			Sysmon
			
		Windows 10 ( Target Machine )
			IP : DHCP
			Splunk Universal Forwarder
			Sysmon
			Atomic Red Team
			
		Kali Linux ( Attack Machine )
			IP : 172.16.188.100

# Part 2

	Use the Hypervisor of your choice
	Virtualbox or VMware
	I'll be using VMware
	Download your ISOs
	Windows 10, Kali Linux, Windows Server 2022, Ubuntu 22.04
	
	Windows 10 Configuration Specs
		Download ISO
		Memory ( RAM ) 4GB ( 4096 )
		1 Processor ( 1 CPU )
		Virtual Hard Disk 50GB
		Select Windows 10 Pro
		Windows 10 Professional W269N-WFGWX-YVC9B-4J6C9-T83GX 
	
	Kali Linux
		Download ISO or a Pre Built kali Linux VM
		The file is a 7zip file 
		so you can get a Application called 7zip to unzip
		On a Mac you just double click to unzip
		If you have Pre Built VM just Double click the file
		It will automatically open in VMware
	
	Windows Server 2022 
		Download ISO ( 64bit edition )
		Name VM AD-DC
		Memory ( RAM ) 4GB ( 4096 )
		1 Processor ( 1 CPU )
		Virtual Hard Disk 50GB
		Select Standard Evaluation ( Desktop Experience )
		Once its loaded to unlock the screen on
		VMware go to the top of the screen and select
		Virtual Machine and select ctl + alt + delete
	
	Ubuntu Server 22.04
		Download ISO
		Name Splunk 
		Memory ( RAM ) 8GB ( 8192 )
		2 Processors ( 2 CPU )
		Virtual Hard Disk 100GB
		name : 
		servers name :
		username :
		Password :
		Install OpenSSH Server
		Once installed run the commands
		sudo apt-get update && sudo apt-get upgrade

# Diagram Layout

```js
Create a Logical Diagram ( Using Draw.io )
	Specify Hardware Requirements
		Splunk Server ( Running on Ubuntu 22.04 )
			IP : set static address

		Active Directory Server ( Windows Server 2022 )
			IP : set static address
			Splunk Universal Forwarder
			Sysmon

		Windows 10 ( Target Machine )
			IP : DHCP
			Splunk Universal Forwarder
			Sysmon
			Atomic Red Team

		Kali Linux ( Attack Machine )
			IP : set static address
```



> Credit : MyDFIR 


An extraordinary help for us beginners trying to get our foot in the door