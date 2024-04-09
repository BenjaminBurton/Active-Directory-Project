## Documentation Installing Splunk on Ubuntu 22.04 on a MacOS

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

At the end of this documentation I will be sharing
how to set a static address from the command line
instead of at the initial setup of ubuntu, but for now during setup we will
manually set a static ip address

Go to splunk.com on your Computer NOT the VMs
Make an account and once you get access go to products
in the upper left corner and in the dropdown go to free trials & Downloads
Scroll down to splunk enterprise and
Click on Get My Free Trial and select
Linux as your operating system.
The file you will download is on top
the .deb file and select download now.
Save it in a directory of your choice.
its probably a good idea to save it in its own folder
like Active-directory-project or something

Now to get Ubuntu to access the shared folders on VMware Fusion
Follow these steps :

- First we want to install open-vm-tools && open-vm-tools-desktop by running these commands


```bash
sudo apt-get install open-vm-tools &&
sudo apt-get install open-vm-tools-desktop
```

- Now lets create a shared folders directory for later


```bash
mkdir shared_folders
```

- Then Create the hgfs folder


```bash
sudo mkdir /mnt/hgfs
```

- Create the file /etc/systemd/system/mnt-hgfs.mount


```bash
sudo nano /etc/systemd/system/mnt-hgfs.mount
```

- Then add this to the configuration per VMware article KB74650 systemd mount


```bash
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
```

- Now we need to create or modify the file /etc/modules-load.d/open-vm-tools.conf
    
- So run this command


```bash
sudo nano or vi /etc/modules-load.d/open-vm-tools.conf
```

- Then add the word fuse to that file

- then we enable and activate the mount service


```bash
sudo systemctl enable --now mnt-hgfs.mount
```

- now lets run this command to make sure its loaded


```bash
sudo modprobe -v fuse
```

- now check that its being shared by running this command


```bash
ls -ahl /mnt/hgfs
```

- then reboot to make sure it still auto mounts

- Should be successful BUT . . . . .
- We dont have write permission so lets get that fixed

- First off we will rename or move the /mnt/hgfs.mount file
- so that it will mount to home-username-shared_folders


```bash
sudo mv /etc/systemd/system/mnt-hgfs.mount /etc/systemd/system/home-username-shared_folders.mount
```

- Then create the folder


```bash
mkdir shared_folders
```

Now we need to edit the contents of the mount file
so it matches our new desired shared_folders location
and our uid and gid so we have permissions to the files and folders

```bash
sudo nano /etc/systemd/system/home-username-shared_folders.mount
```

```bash
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
```


- Now lets do a Daemon reload so the updated config is read


```bash
sudo systemctl daemon-reload
```

- Then enable it


```bash
sudo systemctl enable --now home-username-shared_folders.mount
```

- now your username and id should be the owner


```bash
ls -ahl to check
```

- Now try and create and remove a folder so we have
write access and permissions are set


```bash  
mkdir test-copy.txt
rm test-copy.txt
```

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


(SKIP THIS IF YOU DONT MIND THE UNDERSCORE)

CONTINUE AT sudo dpkg splunk if so


OTHERWISE FOLLOW BELOW

you will have to type the follow these commands



```bash
touch $(systemd-escape -p --suffix=mount "/home/username/shared-folders")
```

- that will close the file in single quotes and changes the dash to a backslash with x2dfolders.mount

- so what you want to do is cat the contents from our current config to the new file


```bash
cat /etc/systemd/home-username-shared_folders.mount > 'home-username-shared\x2dfolders.mount'
```


then


```bash
cat 'home-username-shared\x2dfolders.mount'
```

- then update the file so the line points to the shared dash folder instead of shared_folders in the home directory

- now make the folder for the mount


```bash
mkdir shared-folders
```

- then unmount the shared_folders



```bash
sudo umount shared_folders
```

- then disable the previous config



```bash
sudo systemctl disable --now home-username-shared_folders.mount
```

- and remove it



```bash
sudo rm /etc/systemd/system/home-username-shared_folders.mount
```

- now we need to move the new config file to the forward /etc


```bash
sudo mv 'home-username-shared\x2dfolders.mount' /etc/systemd/system
```

- now we can enable the service


```bash
sudo systemctl enable --now home-username-shared\\x2dfolders.mount
```

- and confirm the mount took place


```bash
ls -ahl shared-folders
```

- Now that you have the shared folder working and accessible
- go into the splunk folder where the installer is and

- run the command

CONTINUED HERE 
```bash
sudo dpkg -i splunk-9.2.0.1xxxxxxxxx.deb
and hit enter and let it install
```

- then go to the /opt/splunk folder


```bash
cd /opt/splunk
```

- then type


```bash
ls -la
```

- notice all of the users and groups belong to splunk
- which is good because it limits the permissions to that user.

- now lets change into the user splunk by typing the command


```bash
sudo -u splunk bash
```

- now we are acting as the user splunk

- now change into the directory bin



```bash
cd bin (/opt/splunk/bin)
```

- all the files listed in here are all the binaries that splunk can use.

- The one we are interested in is the ./splunk 

- so type the command


```bash
./splunk start
```

- this will run the installer. 
- We'll get a license and terms agreement 
- just hit q then type in y to accept + enter

- then enter an administrator username + a password
- now the installation is completed and we will
- run one more command to make sure splunk starts up everytime
- the Virtual machine reboots.

- So first type exit to exit out of the user splunk
- exit + enter

- then change into the directory of bin


```bash
cd bin + hit enter
Make sure your in the /opt/splunk/bin
```

- then type this command


```bash
sudo ./splunk enable boot-start -user splunk
```

- this will make it so anytime the virtual machine reboots
- splunk will run with the user splunk

- Mission complete Splunk is all setup and ready for the next phase
- More to come on this later
