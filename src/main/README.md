

#### AD Documentation 

	Database containing Users Computers Groups and many more
	In order to use Active Directory A server must install a 
	service called Active Directory Domain Services ( AD DS )
	An then the server must then be promoted to a Domain 
	Controller ( DC ) By promoting our server to a DC it will 
	Grant us the capability to perform Authentication Using a 
	protocol called Kerberos and Authorization For our domain
	With AD DS we will have a database that will Contain Objects
	Such as Users, Computers, Groups and Many more These Objects 
	will take Attributes which is Information about the object 
	(metadata) Example : Object might be user : Bob with an 
	Attribute of First Name - Bob Last Name - Smith


#### Steps to the 5 parts 


	Domain :
	Network : 
	Splunk Server :
	Active Directory :
	Attacker :

##### Part 1 ( Diagram )

	Create a Logical Diagram ( Using Draw.io )
	Specify Hardware Requirements
		Splunk Server ( Running on Ubuntu 22.04 )
			IP :
		
		Active Directory Server ( Windows Server 2022 )
			IP :
			Splunk Universal Forwarder
			Sysmon
			
		Windows 10 ( Target Machine )
			IP : DHCP
			Splunk Universal Forwarder
			Sysmon
			Atomic Red Team
			
		Kali Linux ( Attack Machine )
			IP :

##### Part 2

	Use your Hypervisor of choice
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

##### Part 3

	Install && Configure Sysmon && Splunk on
	Windows Target Machine && Windows Server Machine
	so they can start collecting telemetry and send logs 
	over to our splunk server
	
	Splunk Server
		Set to NAT Network
		Check IP address by running the command 
			ip a
		At the end of this documentation I will be sharing
		how to set a static address from the command line 
		instead of at the setup, but for now during setup we will
		manually set a static ip address
		
		Go to splunk.com on your Computer NOT the VMs
		Make an account and once you get access go to products
		and in the dropdown go to free trials & Downloads
		 Scroll down to splunk enterprise and
		 Click on Get My Free Trial and select
		 Linux as your operating system
		 the file you will down load is on top
		 the .deb file and select download now
		 save it in a directory of your choice 
		 its probably a good idea to save it in its own folder
		 like Active-directory-project or something
	
	Now to get Ubuntu to access the shared folders on VMware Fusion
	Follow these steps : 

	First we want to install open-vm-tools && open-vm-tools-desktop
		sudo apt-get install open-vm-tools &&
		sudo apt-get install open-vm-tools-desktop
	
	Then Create the hgfs folder
		sudo mkdir /mnt/hgfs

	
	Create the file /etc/systemd/system/mnt-hgfs.mount
		sudo nano /etc/systemd/system/mnt-hgfs.mount
		
	Then add 
	
	[Unit]
	Description=VMware mount for hgfs
	DefaultDependencies=no
	Before=umount.target
	ConditionVirtualization=vmware
	After=sys-fs-fuse-connections.mount
	
	[Mount]
	What=vmhgfs-fuse
	Where=/mnt/hgfs
	Type=fuse
	Options=default_permissions,allow_other
	
	[Install]
	WantedBy=multi-user.target
	
	Now we need to create or modify the file
	/etc/modules-load.d/open-vm-tools.conf 
	
	sudo nano or vi /etc/modules-load.d/open-vm-tools.conf 
	
	Then add the word fuse to the file
	
	then we enable and activate the mount service
	
	sudo systemctl enable --now mnt-hgfs.mount
	
	now lets run this command to make sure its loaded
	
	sudo modprobe -v fuse 
	
	now check that its being shared by running this command
	
	ls -ahl /mnt/hgfs
	
	then reboot to make sure it still auto mounts
	
	Should be successful BUT . . . . . 
	We dont have write permission so lets get that fixed
	
	First off we will rename or move the /mnt/hgfs.mount file
	so that it will mount to home-username-shared_folders
	
	sudo mv /etc/systemd/system/mnt-hgfs.mount /etc/systemd/system/home-username-shared_folders.mount
	
	Then create the folder
	mkdir shared_folders
	
	Now we need to edit the contents of the mount file
	so it matches our new desired shared_folders location
	and our uid and gid so we have permissions to the folders
	and files 
	
	
	sudo nano /etc/systemd/system/home-username-
	shared_folders.mount
	
	[Unit]
	Description=VMware mount for hgfs
	DefaultDependencies=no
	Before=umount.target
	ConditionVirtualization=vmware
	After=sys-fs-fuse-connections.mount
	
	[Mount]
	What=vmhgfs-fuse
	Where=/home/username/shared_folders
	Type=fuse
	Options=default_permissions,allow_other,uid=1000,gid=1000
	
	[Install]
	WantedBy=multi-user.target
	
	Now lets do a Daemon reload so the updated config is read
		sudo systemctl daemon-reload
	
	Then enable it 
	sudo systemctl enable --now home-username-shared_folders.mount
	
	now your username and id should be the owner
		ls -ahl to check
	
	Now try and create and remove a folder so we have
	write access and permissions are set
		mkdir test-copy.txt
		rm test-copy.txt
	
	now reboot to make sure everything has taken effect
	SUCCESS
	
	now the reason we used the underscore on shared_folders
	instead of the dash (shared-folders) is beacause the dash 
	is sort of a delimiter in the unit file when your doing
	your file name that your going to be mapping things to
	so if you want to have a dash in the actual folder name
	then your going to have to delimit it. Now in order
	to do that you have to use a special command
	
	so to use shared-folders instead
	
	you will have to type the following command
	
	touch $(systemd-escape -p --suffix=mount 
	"/home/username/shared-folders")
	
	that will close the file in single quotes and changes the dash
	to a backslash with x2dfolders.mount
	
	so what you want to do is cat the contents from our current
	config to the new file 
	
	cat /etc/systemd/home-username-shared_folders.mount 
	> 'home-username-shared\x2dfolders.mount' 
	
	then enter
	
	then
		cat 'home-username-shared\x2dfolders.mount'
	
	then update the file so the line points to the shared
	dash folder instead of shared_folders in the home directory
	
	now make the folder for the mount
		mkdir shared-folders
	
	then unmount the shared_folders
		sudo umount shared_folders
	
	then disable the previous config
	sudo systemctl disable --now home-username-shared_folders.mount
	
	and remove it
	sudo rm /etc/systemd/system/home-username-shared_folders.mount
	
	now we need to move the new config file to the forward /etc
	
	sudo mv 'home-username-shared\x2dfolders.mount' 
	/etc/systemd/system
	
	now we can enable the service 
		sudo systemctl enable --now home-username-
		shared\\x2dfolders.mount
	
	and confirm the mount took place
		ls -ahl shared-folders

	Now that you have the shared folder working and accessible
	go into the splunk folder where the installer is and 
	run the command 
		sudo dpkg -i splunk-9.2.0.1xxxxxxxxx.deb
	and hit enter and let it install
	then go to the /opt/splunk folder
		cd /opt/splunk
	then type 
		ls -la
	notice all of the users and groups belong to splunk
	which is good because it limits the permissions to that user.
	now lets change into the user splunk by typing the command
		sudo -u splunk bash
	now we are acting as the user splunk
	now change into the directory bin
		cd bin
	all the files listed in here are all the binaries 
	that splunk can use. The one we are interested in is 
	the ./splunk so type the command 
		./splunk start 
	to run the installer. We'll get a 
	license and terms agreement just hit q then 
	type in y to accept + enter 
	then enter your administrator username + a password
	now the installation is completed and we will
	run one more command to make sure splunk starts up everytime
	the Virtual machine reboots.
	So first type exit to exit out of the user splunk
		exit + enter
	then change into the directory of bin
		cd bin + hit enter
	then type this command 
		sudo ./splunk enable boot-start -user splunk
	this will make it so anytime the virtual machine reboots
	splunk will run with the user splunk

	Windows 10 ( Target Machine )
		Lets name our machine
		go to the search bar and type pc and go to properties
		then go to Rename this PC
		type in Target-PC + next and restart now
		once restarted check the rename took place
		next go to seach bar and type cmd for command prompt
		check th ip address + we want to change the ip so
		at the bottom of the screen go to the network icon
		right click and click Open Network & internet settings
		click on change adapter options
		right click on the adapter ( Ethernet )
		click on properties and
		double click Internet Protocol Version ( TCP/IPv4 )
		in the box select Use the following IP address
		and set a static IP + default gateway + DNS server
		Back in Command prompt type 
			ipconfig
		and the IP address + gateway should be changed so
		there wont be any conflicts happening 
	
	Now lets open up a web browser (in Windows 10) 
	and access our splunk server
	in the web address bar type in the ip 
	with a colon and port 8000 so
		ipaddress:8000
	FYI splunk listens on port 8000 so specify the port otherwise
	you wont connect to splunk.
	Now as you can see you can access its login page
	now lets install splunk universal forwarder by heading over
	to splunk.com then login and go to products top left of page
	select `Free Trials and Downloads` from the dropdown
	scroll down to universal forwarder and 
	click `Get My Free Download`
	Now select Windows 64-bit + click Download now
	once its downloaded go to your downloads folder
	and double click the msi file
	click on check this box to accept the license Agreement
	And make sure 
	An on-premisis Splunk Enterprise Instance is selected
	Hit next 
	for username type admin
	and keep Generate random password checked
	then next
	the deployment server we dont have so click next
	For the Receiving Indexer this is going to be 
	our splunk server so the IP of the splunk server
	and the default port for splunk is 9997 
	when its receiving events so leave that as 9997
	and select next + install
	Now start downloading Sysmon by searching for 
	Sysmon by sysinternal scroll down and click
	download Sysmon
	and the Sysmon configuration we will use is by olaf
	so search olaf sysmon config (its on github)
	in the repository scroll down and select
		sysmonconfig.xml 
	click on that and click raw on right hand side
	then right click and save as and save under downloads directory
	now go to downloads directory
	go to the Sysmon zip file and right click + extract all
	in that directory click on the file explorer bar 
	and copy the path
	then open up PowerSehll wih admin privileges (run as admin) 
	now just type cd + paste the path + enter 
	so were in that directory
	now type 
	.\Sysmon64.exe -i ..\sysmonconfig.xml + enter
	The -i specifies we want a config file which is one 
	directory back so thats the reason for the ..\
	this will install sysmon for you so select agree
	once installed you should see the splunk pop up windows
	saying Universal forwarder was successfully installed
	close it out by clicking on finish
	close out PowerShell
	Now the MOST IMPORTANT PART
	we need to instruct our splunk forwarder on
	what we want to send over to our splunk server
	to do this we must configure a file called inputs.conf
	this file is located under the C: Drive + Program Files
	SplunkUniversalForwarder + etc + system + default 
	and the files is there
	now we can copy that file and create a new file under
	local because you do not want to edit the default 
	inputs.conf.
	DO NOT EDIT THE inputs.conf in the defaults directory
	we will create the inputs.conf in the Local directory
	this folder requires adminstrator priviliges so
	serach up notepad and run it as admin
	the contents that goes in this notepad are in the notes 
	down below for you to copy and paste in 
	this is instructing our splunk forwarder to push events 
	related to Application, Security, System as well as sysmon
	over to our splunk server
	take note of the index we are using
	the index is pointing to an endpoint
	this is IMPORTANT to know because whatever events fall under 
	these categories will be sent over to splunkand placed under
	the index endpoint if our splunk server does not have an index 
	named endpoint it will not receive any of these events.
	
	Now we will save this notepad under 
		 program files + SplunkUniversalForwarder + etc + system + 
		 local. and will be saved in the local directory
		 name it inputs and change Save as type to all files
		 and put an extension as .conf click on save and exit out
		 Now we should see inputs.conf in our local directory
		 ANY time you edit inputs.conf file you MUST restart 
		 SplunkUniversalForwarder service in order to do that 
		 search up "services" run as admin and look for Splunk 
		 Forwarder service. If you scroll over to the right and 
		 under the column log on as you see NT Service so double 
		 click it and click on the log on tab and having this  
		 account checked (NT Service) it might not be able to 
		 collect the events due to some of the permissions so 
		 select Local System Account and click apply. It will
		 say the new logon wont take affect until you restart 
		 the service which is what we did so click ok.
		 Now the logon as column shows local system, and scroll 
		 down a little bit and you should see Sysmon64 running
		 Now right click Splunk Forwarder  and select restart.
		 Most likely you will get hit with a warning saying
		 windows could not stop the Splunk Forwarder so click ok
		 and just start the service.
		 
		 Now we have our Sysmon installed & Splunk Universal 
		 Forwarder installed along with our updated inputs.conf
		 file.
		 
		 Now lets finalize our Splunk configuration. Head over to 
		 our Splunk web Portal & log in using the credentials we 
		 created during the splunk install on our server.
		 
		 When we are presented with the screen we want to select 
		 settings at the top and then head over to indexes.
		 Remember the inputs.conf file all our events are being
		 sent over to an index called endpoint. this is where we 
		 create that index. If you scroll down you wont see a 
		 index on the left named endpoint so we will create one.
		 Scroll all the way up and click on new index.
		 For the name type endpoint and click save and now a new 
		 endpoint index should be created.
		 Now go back up top to settings and select 
		 Forwarding and receiving. Under the receive data click 
		 configure receiving + New receving port and type in 9997
		 for Listen on this port + save
		 
		 If everything is setup correctly we should see data 
		 coming in from our target machine.
		 
		 In the top of the screen left side select 
		 apps + Search & Reporting and skip the tour
		 Now in the search bar type in 
			 index=endpoint
			and search last 24 hours
			
			Now you have successfully installed and configured 
			Splunk.
		
		Windows Server 2022
			Search for PC go to properties and Rename this PC
			change the name to AD-DC-001 click next + restart

##### Part 4


	UNDERCONSTRUCTION


##### Part 5

	UNDERCONSTRUCTION



#### Set Static ip address for Ubuntu ( BONUS )

	Login to your Ubuntu and run the command 
		sudo nano /etc/netplan/00-installer-config.yaml
	Under the dhcp setting change it to no for NO dhcp
	press enter for a new line then hit tab 3 times and type
		addresses: [ip address goes here with a /24 at the end]
	and yes put the ip inside the brackets
	press enter again for new line again and hit tab 3 times again
	and type
		nameservers: 
	press enter again and this time hit tab 5 times and type
		addresses: [dns ip w/ brackets goes here ex. 8.8.8.8]
	press enter again hit tab 3 times and type 
		routes: 
	because we want to add a default route. 
	Then hit enter for a new line and hit tab 5 times and type
		- to: default
	press enter for new line and hit tab 6 times and type
		via: ip address without brackets goes here 
	This is for our gateway 
	to save this configuration 
	hold control + hit x 
	to save modified buffer
	then hit y
	then hit enter
	now apply the changes by typing the command
		sudo netplan apply + enter
	Ignore the warnings about permission 
	since we are in a lab environment
	to test connectivity ping google
		ping -c 3 google.com
	if you dont any connectivity check your formatting