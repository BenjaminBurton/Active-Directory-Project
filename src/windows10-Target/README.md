# Windows 10 Target Machine Documentation


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
	
	in the web address bar type in the ip of the splunk server
	with a colon and port number 8000 so
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
	And make sure An on-premisis Splunk Enterprise Instance is 
	selected
	
	Hit next 
	
	for username type admin and keep Generate random password 
	checked
	
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
	
	the Sysmon configuration we will use is by olaf
	
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

	## Splunk Inputs.conf

	[WinEventLog://Application]

	index = endpoint

	disabled = false

	[WinEventLog://Security]

	index = endpoint

	disabled = false

	[WinEventLog://System]

	index = endpoint

	disabled = false

	[WinEventLog://Microsoft-Windows-Sysmon/Operational]

	index = endpoint

	disabled = false

	renderXml = true

	source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
	
	this is instructing our splunk forwarder to push events 
	related to Application, Security, System as well as sysmon
	over to our splunk server
	
	take note of the index we are using
	
	the index is pointing to an endpoint
	
	this is IMPORTANT to know because whatever events fall under 
	these categories will be sent over to splunk and placed under
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
		 under the column "log On As" you see NT Service so double 
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
		 Forwarding and receiving. 
		 
		 Under the receive data click configure receiving + New 
		 receving port and type in 9997
		 for Listening on this port + save
		 
		 If everything is setup correctly we should see data 
		 coming in from our target machine.
		 
		 In the top of the screen left side select 
		 apps + Search & Reporting and skip the tour
		 Now in the search bar type in 
			 index=endpoint
			and search last 24 hours
			
			Now you have successfully installed and configured 
			Splunk.
		