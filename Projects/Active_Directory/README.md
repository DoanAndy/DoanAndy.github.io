## Active Directory Project

 

* Status: In Progress 
* Category: Laboratory, Demonstration
* Domain: Cybersecurity, Windows, Linux, SIEM
* Tool set: Kali Linux, Windows 10 Pro, VirtualBox, Windows Server 2022, Ubuntu Server, Splunk Enterprise, Crowbar, PowerShell


# Goal

The goal of this project is to demonstrate my abilities to set up and configure a network in a virtual laboratory setting and practice using industry standard tools. I will also simulate an attack on this network using readily available tools and techniques.

If successful the network will be configured exactly like the below diagram and feature the following:

* Windows Server 2022 with a static IP, Active Directory enabled, Splunk Universal Forwarder and Sysmon installed
* Ubuntu Server with a static IP and Splunk Server installed
* Windows 10 Pro with an IP assigned using DHCP with Splunk Universal Forwarder and Sysmon installed
* Kali Linux with a static IP and the default configuration

#Process

Downloads
Windows 10 Installation Media tool https://www.microsoft.com/en-ca/software-download/windows10
Windows Server 2022 https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
Ubuntu Server 22.04.4 https://ubuntu.com/download/server
Splunk Enterprise for Linux
Splunk Universal Forwarder Windows 10
Sysmon https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
Crowbar

>Step One: Install and configure the virtual machines

The plan is to have all of the VMs running on a single computer. I have a Windows 10 machine with 32 gigs of ram and this will handle everything without problems. The first step was to download and install all four operation systems on the VirtualBox. The OS were downloaded from there respective sites one at a time. They were all assigned 4 gigabytes of ram and 50 gigabytes of file space except the Ubuntu machine it was given 8 gigabytes ram and 100 gigabytes disk space to give it a little more power for running queries. I already had VirtualBox and baseline Kali Linux instances running on my machine. 

I created a NAT Network on VirtualBox and connected all of the VMs to this network.I named the network AD-Project and set the IPv4 Prefix to 192.168.10.0/24. Before moving to the next step I created a snapshot of each OS instance and named it "Setup Baseline". 

Picture of Vbox NAT-network

>Step Two: Setup Splunk Server

I edited the 00-installer-config.yaml to reflect the desired state of the machine and set the static IP to 192.168.10.10. On the Ubuntu version I deployed I was not able to directly apply netplan until I installed openvswitch-switch-dpdk firsts. 
Pic of  00-installer-config.yaml

The Splunk Enterprise for Linux was downloaded on the parent machine so I needed to install virtualbox-guest-additions-iso, and virtualbox-guest-utils in order to mount a shared drive between the VM parent and the Ubuntu instance. I also needed to set up the share in VirtualBox and point it to the directory containing the download. 

Splunk Enterprise was installed from the shared directory and the default Splunk user was created. I enabled Splunk to run on startup via "sudo ./splunk enable boot-start -user splunk". Splunk was now running on the Ubuntu server and the dashboard is reachable via the browser at 192.168.10.10:8000 although no data was currently flowing to it.

>Step Three: Add Splunk Universal Forwarder to the Domain Controller and the Windows 10 machine.

The setup process for both the Window Server 2022 and the Windows 10 Pro machine were the same. Splunk Universal Forwarder was downloaded and installed from the Splunk website and installed normally and configured to send logs to 192.168.10.10:9997. Sysmon was downloaded from a Microsoft page from a google search. I also used a configuration for sysmon that was posted on a GitHub users page (this was recommended by a youtuber). Sysmon was installed using PowerShell and the configuration .XML file. 

Once both devices were set up with the forwarder and sysmon was configured I was able to see telemetry data flowing to the Splunk server. The Splunk server need to be set up to look for inputs on the "endpoint" and after that searches for "index=endpoint" showed data coming from 2 different hosts (Windows Server and Windows 10). 

>Step Four: Installing Active Directory

On the server computer I installed and configured Active Director via Server Manager -> Manage -> Add Roles and Features. I created the domain ANDY.LOCAL and added to organizational units (IT and HR). I then created two fictional users (Teri Smith to HR and Jenny Smith to IT) and assigned them to the org units. 

Once the server was set up I went to the Windows 10 instance and joined the newly created domain. I changed the network settings to reflect the desired state pointing to 192.168.10.7 as the DNS and logged in with one of the fictional user accounts. From here I was able to test that I had access to the network and that i could ping the splunk server. 

In preparation for the next step I prepared the Windows 10 PC for the attack phase by enabling a service that is disabled by default. I enabled remote desktop on the target machine and left it running and signed in. 

Step Five: Attack Target PC

From the Kali Linux VM I installed crowbar and unzipped the passwords file bundled with the package. I edited the password file to include the actual password I set on the fictional user account for purposes of this test so that I would for sure get a positive result. Crowbar attempted to sign in on port 3389 to the rdp service using the account name and IP address I provided using every password in the password file. 

pic of kali-crowbar

This demonstrates a weakness in the Remote Desktop Protocol to dictionary attacks. If EDP in enabled and the attacker is able to obtain a valid user name in Active Directory they could launch a dictionary attack they may be successful if strong passwords are not enforced. After this "attack" I was able to see in Splunk evidence of a number of almost simultaneous failed login attempts followed by a successful attempt all attributed to a IP address that would not be a part of a authorized list. This type of activity may alert a system monitor to suspicious ativity from this user and could trigger action to block the IP at the firewall.

On the target system I also installed AtomicRedTeam via Powershell. Once installed I was able to run Atomic 1136.001 "Persistence Create Account - Local Account" and successfully able to create an account. I also ran 1059.001 "Command and Scripting Interpreter - PowerShell" and  failed due to lack of access. These "attacks" are directly linked to common attack types from the MITRE ATT&CK framework and demonstrate vulnerability (in the case of 1136.001) to the default configuration. If an attacked gain access to an active directory account they would be able to move laterially into a new account to gain persistence in the network. 


* Windows 10 Installation Media tool https://www.microsoft.com/en-ca/software-download/windows10
* Windows Server 2022 https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
* Ubuntu Server 22.04.4 https://ubuntu.com/download/server

Setup

* Add Windows 10 Pro instance to VirtualBox
	* User: Win10_User. 
	
* Add Kali Linux Instance to VirtualBox

* Add Windows Server 2022 instance to VirtualBox

* Create NAT Network "AD-Project", Connect all VMs to this network (pic VirtualBox_NAT_Network_setup.png)

* on Ubuntu Server, configure /etc/netplan/00-installer-config.yaml (pic 00-installer-config.png)
	* received an error when I tried ran "sudo netplan apply" WARNING:root:Cannot call Open vSwitch: ovsdb-server.service is not running.
	* from https://stackoverflow.com/questions/77352932/ovsdb-server-service-from-no-where
		* "sudo apt-get install openvswitch-switch-dpdk" - seemed to work after this
	* on Ubuntu server installed virtualbox-guest-additions-iso via "sudo apt-get install virtualbox-guest-additions-iso"
	* on Ubuntu Server installed virtualbox-guest-utils via "sudo apt-get install virtualbox-guest-utils"
	* added a shared folder to the Ubuntu VM called "Active_Directory"
	* added user andy to vboxsf via "sudo adduser andy vboxsf"
	* created dir share
	* mounted the shared vbox directory to the share folder in the Ubuntu server (this contains a download of Splunk)
	* install Splunk from the shared folder via "sudo dpkg -i splunk-9.2.1-78803f08aabb-linux-2.6-amd64.deb"
	* changed to the splunk user via "sudo -u splunk bash"
	* started the splunk installer on Ubuntu via "cd bin | ./splunk start"
	* Set Splunk to automatically startup with the vm via "cd bin | sudo ./splunk enable boot-start -user splunk"
	
* On Windows 10 Target
	* computer name set to TARGET-PC
	* Download and install Splunk Universal Forwarder
	* Downloaded Sysmon and the Sysmon configuration file from GitHub User Olaf. 
	* installed Splunk Universal Forwarder from downloaded file.
	* used PowerShell to install Sysmon using the configuration file from GitHub
	* Manually created the inputs.conf file and installed it in the etc/system/local folder.
	* in the windows services utility set Splunk to run as a local account on the host and restarted the system.
		* this is required when ever there are manual edits done to the inputs.cong file.
	* Checking 192.168.10.10:8000 can see that the Splunk service is running on the Ubuntu service.
	* configure the Splunk server to look for the inputs from the "endpoint" (Target PC)
	* Doing a Splunk search "index=endpoint" there is actual log data being received from 3 of the hosts I configured.
	
* On Microsoft Server - Repeat the same steps for Splunk Universal Forwarder set up.	
	* computer name set to ADDC01
	* Configure Server management
	* Install Active Directory Domain Features via the Server Manager -> Manage -> Add Roles and Features
	* Configure Active directory
	* Create domain ANDY.LOCAL
	* Add organizational unit IT and HR.
	* Add Users to Active Directory and assign them to organizational groups
		* Jenny Smith assigned to IT
		* Teri Smith assigned to HR
		
* On Windows 10 Target computer	
	* join newly create domain
	* Change network settings, point the DNS at the server (192.168.10.7)
	* Log in to Windows 10 machine as one of the AD users we created. 
	* Enable remote desktop on target Windows 10 machine
	
* On Kali Linux
	* Set static IP to 192.168.10.250
	* Create a project folder AD-Project on desktop
	* install Crowbar from the command line via "sudo apt-get install -y crowbar"
	* Pro Tip: Run "sudo apt-get update && sudo apt-get upgrade" often!!!
	* unziped and copied the passwords.txt file to the project folder
	* Ran the crowbar tool using "sudo -b rdp -u tsmith _C passwords.txt -s 192.168.10.100/32"
		* this was a simulated brute force attack against the Target Windows 10 machine, I included the actual password set for un: tsmith in order to get a positive result.
	* Crowbar attack against the Remote Desktop Protocol - rdp on ip 192.168.10.100 was successful and the password was revealed.
	
* Logged in to the Splunk dashboard running on 192.168.10.10:8000
	* Searched Splunk for "index=enpoint tsmith" 
		* revealed 20 events on Event Code 4625 - An account failed to log on.
		* events taged "Account For Which Logon Failed" appear to be stamped within a few seconds on un: tsmith indicating the possibility of a brute force attack using an automated tool. 
		
* On Target Machine 
	* install AtomicRedTeam via PowerShell command: "IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);" and "Install-AtomicRedTeam -getAtomics"
		* need to create an exemption for the C:\ first so that Windows Defender isn't triggered by this application
	* I was able to run two attacks on the target PC based on vulnerabilities found on the MITRE ATT&CK website
		* MITRE ATT&CK T1059.001 - Command and Scripting Interpreter - PowerShell - Failed due to lack of access.
		* MITRE ATT&CK T1136.001 - Persistence Create Account - Local Account - Successm able to create an account


#Result
Summary: For this project I was able to successful deploy and configure four different operating systems on a simulated network. Each device was setup according to it's role in the exercise and they all functioned properly based on the plan. I was able to deploy an instance of Splunk in the simulation and pass telemetry to the service from two different network devices. Active Directory was installed on the domain controller and user accounts were created. The target endpoint was able to join the domain and act like an regular user with limited permissions. Once configured I was able to launch a couple of different attacks on the network using available tools and techniques. 

Working through this material was very rewarding for me. I was able to use a few tools that I have limited experience with (Splunk, and Active Directory) as well as increase my skills with others I am more familiar with(Linux, VirtualBox). I was also able to see how easily attacks against a network can happen when systems are not configured properly and know vulnerabilities are not accounted for. 

The most rewarding aspect of this was definitely seeing the direct relationship between the input and output with an industry standard SIEM tool. I was able to launch an attack and then immediately login to Splunk to see what the logs looked like. This was a valuable experience for me and I am sure to expand on this foundation and I continue to practice and study these common techniques. 

Conclusion: Overall a successful and entertaining experience. Lots of really cool tools and techniques demonstrated through the process. I am very grateful for the opportunity and thankful to this YouTuber for putting it together for me! 

#Resources

Ultimate IT Security - Windows Security Log Events - https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
MyDFIR GitHub Active-Directory-Project repo - https://github.com/MyDFIR/Active-Directory-Project
MITRE ATT&CK framework https://attack.mitre.org/
Network drawing http://draw.io


Active Directory Project Video 1 https://www.youtube.com/watch?v=mWqYyl89QaY&t=525s
Active Directory Project Video 2 https://www.youtube.com/watch?v=2cEj3bS5C0Q
Active Directory Project Video 3 https://www.youtube.com/watch?v=uXRxoPKX65Q&t=897s
Active Directory Project Video 4 https://www.youtube.com/watch?v=1XeDht_B-bA&t=5s
Active Directory Project Video 5 https://www.youtube.com/watch?v=orq-OPIdV9M&t=923s

#Issues

Windows 10 Target instance seems to be getting hung up a lot.

