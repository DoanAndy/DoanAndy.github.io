## Active Directory Project

 

* Status: In Progress 
* Category: Laboratory, Demonstration
* Domain: Cybersecurity, Windows, Linux, SIEM
* Tool set: Kali Linux, Windows 10 Pro, VirtualBox, Windows Server 2022, Ubuntu Server, Splunk Enterprise, Crowbar, PowerShell


# Goal

The goal of this project is to demonstrate my abilities to set up and configure a network in a virtual laboratory setting and practice using industry standard tools. I will also simulate an attack on this network using readily available tools and techniques.

#Process

Downloads
Windows 10 Installation Media tool https://www.microsoft.com/en-ca/software-download/windows10
Windows Server 2022 https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
Ubuntu Server 22.04.4 https://ubuntu.com/download/server
Splunk Enterprise for Linux
Splunk Universal Forwarder Windows 10
Sysmon
Crowbar

Setup

* Add Windows 10 Pro instance to VirtualBox
	* User: Win10_User. 
	* Security Questions: What city were you born in? st thomas, First School? elgin court, Childhood Nickname? spike
	*
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
	* Checking 192.168.10.10 can see that the Splunk service is running on the Ubuntu service.
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


Brief outline available on this youtube video https://www.youtube.com/watch?v=-iVqueVIhuQ

#Issues

Windows 10 Target instance seems to be getting hung up a lot.

