## VMware KB 74650 enabling shared folders Fix

VMware's article KB 74650 enabling hgfs shared folders on fusion or workstation
hosted linux VMs for open-vm-tools. This is to get your shared folders auto
mounting in your Linux guest OS using the systemd mount unit.

Problem is the /mnt/hgfs is owned by root. 
The shared folders under /mnt/hgfs are chmod 0700 and owned by user id 502 and group id dialout 
so you wont be able to access without sudo or changing into root.

So we will be mounting shared folders under our home directory for easy access 
from the terminal setting the user:group to our uid:gid so we can actually access 
the shared folders.

First we want to install open-vm-tools && open-vm-tools-desktop

```bash
		sudo apt-get install open-vm-tools &&
		sudo apt-get install open-vm-tools-desktop
```


Then Create the hgfs folder

```bash
		sudo mkdir /mnt/hgfs
```

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
	
	
	sudo nano /etc/systemd/system/home-username-shared_folders.mount
	
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
	
	touch $(systemd-escape -p --suffix=mount "/home/username/shared-folders")
	
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
	
	