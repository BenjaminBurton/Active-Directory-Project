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

![Resize](/images/diagram.png?width=200px)

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