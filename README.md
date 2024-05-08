# Brute Force Attack SSH

## Introduction
In this project, I explored vulnerabilities within SSH security, utilising Microsoft Azure infrastructure for practical demonstration. SSH does not typically have a password limit. By setting up an SSH server instance and employing brute force techniques with various password combinations, I showcase how easily servers lacking adequate security measures can be compromised. 

Requirements: 
1. Create a Tenant (Organisation) aka an Azure account 
2. Create a Subscription
3. Create a Resource Group
4. Create a Virtual Network
5. Create a Windows & Linux virtual machine 
6. Install SQL Server 

## Create Windows 10 Pro Virtual Machine (Name it windows-vm)
- https://portal.azure.com/
- Subscription: Azure subscription 1
- Resource group: RG-Cyber-Lab
- Virtual machine name: windows-vm
- Region: EAST US 2
- Image: Windows 10 Pro
- Size: E2dbs_v5 (for VM size it must have a minimum 2 VCPUs)
- Username: labuser
- Password: Cyberlab123!
- Click box “I can confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights”
- Networking  → Virtual network Create new
- Name: Lab-VNet
- Review + Create  
## Create Linux Virtual Machine running Ubuntu 
- Virtual machine name: linux-vm
- Same Region, Resource Group, and VNet as windows-vm
- Image: Ubuntu Server 20.04
- Authentication type: Password
- Same Size, Username, Password windows-vm
- Same Virtual Network as Windows 

!VM CREATE

## Configure Network Security Group (Layer 4 Firewall) for both VMs to allow all traffic inbound
- NOTE: This will allow random people online to send traffic to the VMs
- Search Network Security Group → windows-vm-nsg
- Delete RDP Inbound Security Rule 
- Add Inbound Security Rule
- Destination port ranges: * (FYI this means any)
- Name: DANGER_AllowAnyCustomAnyInbound
- Add
- Search Network Security Group → linux-vm-nsg
- Delete SSH Inbound Security Rule
- Same Inbound Security Rule, Destination port ranges, Name as windows-vm-nsg
- Add

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53707483902/in/datetaken/" title="NSG"><img src="https://live.staticflickr.com/65535/53707483902_c100daca44_c.jpg" width="800" height="261" alt="NSG"/></a>

## Disable Windows Firewall and Install SQL Server and Create Vulnerabilities
- <ins>Turn off Windows Firewall</ins>
- Open Remote Desktop Connection app
- Remote Into the VM (windows-vm) by using the Public IP address
- Login with same Username & Password used to create VM
- In Startup menu search bar type: wf.msc (Microsoft Common Console Document)
- Turn Domain Profile, Private Profile, Public Profile firewall state OFF
- Open PowerShell app 
- Ping the windows-vm IP address to verify there is a connection

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708828185/in/datetaken/" title="Firewall Off"><img src="https://live.staticflickr.com/65535/53708828185_5cba475c36_c.jpg" width="800" height="598" alt="Firewall Off"/></a>

!PING VM

## Install SQL Server 2019 Evaluation exe 64-bit
- https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019
- Installation type: Download Media → ISO → Download → Open folder
- Right click Disc Image File → Mount
- Click setup → Installation → New SQL Server stand-alone installation 
- Click the box “Database Engine Services” → Next
- Database Engine Configuration → Click Mixed Mode
- Enter password: Cyberlab123!
- FYI: your username will be sa (by default)
- Click Add Current User (this allows the Windows vm account to authenticate and login to SQL).
- Install

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708725694/in/datetaken/" title="SQL Server 2019"><img src="https://live.staticflickr.com/65535/53708725694_2f012be0c1_c.jpg" width="800" height="637" alt="SQL Server 2019"/></a>

## Install SSMS (SQL Server Management Studio) 
- https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms 
- NOTE: This is an app that lets you connect to the SQL DB (You can also connect with PowerShell or any programming language but SSMS interface is more user friendly).
- Or open your VM browser and Google “SSMS” then click on Free download for SQL Server Management Studio.
- Open app and install.
- Click restart 
- Log back into windows-vm

!SSMS

## Enable logging for SQL Server to be ported into Windows Event Viewer
- Google “Write SQL Server Audit events to the Security log”
- Open Event Viewer app → Windows Logs → Security (this is where you can view all the logs for the VM). 
- NOTE: The aim is to be able to see in the Event Viewer logs when someone fails or has a successful login in SQL Server.
- Go back to webpage and copy & paste the following to provide full permission for the SQL Server service account to the registry hive: 
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Security
- NOTE: Windows Registry Editor is where you can make granular configurations to effect the OS.
- Open Registry Editor app
- Paste the HKEY registry path in the Registry Editor search box so it should read: Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Security
- Right click Security → Permissions → Add → Type “NETWORK SERVICE” in “Enter object name to select”→ OK 
- Click box to Allow Full Control → OK.
- Go back to webpage under subsection “Configure the audit object access setting in Windows using auditpol”
- Copy the following Windows Command Prompt: auditpol /set /subcategory:"application generated" /success:enable /failure:enable
- Open Command Prompt app → right click & Run as administrator
- Paste the auditpol command prompt and hit enter.

## Test SSMS login to ensure it is working properly
- Open SQL Server Management Studio App
- Authentication: SQL Server Authentication 
- Login: sa
- Password: Cyberlab123!
- Open SSMS app
- Click box “Trust server certificate” → Connect
- In the Object Explorer, right click windows-vm → Properties →  Security → Both failed and successful logins →  OK
- FYI this allows both log types to appear in Event Viewer
- Right click windows-vm and click restart.
- Click disconnect (the plug with the red x)
- Reconnect (click the plug)
- Login with a wrong password on purpose. 
- Open Event Viewer app → Application 
- The last log should appear as Event ID: 18456 Password did not match

!TEST SSMS

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708828190/in/datetaken/" title="18456 SQL Server logon fail"><img src="https://live.staticflickr.com/65535/53708828190_c0d6b8f76f_c.jpg" width="800" height="496" alt="18456 SQL Server logon fail"/></a>

## Test ping and logging into linux-vm via SSH
- <ins>Ping linux-vm</ins>
- Close your VM and go back to your main PC. 
- In Azure, copy the linux-vm Public IP address.
- In startup menu open Windows PowerShell. 
- <ins>Login to linux-vm</ins>
- To connect with SSH:
- Type: ssh (username@Public IP address)
- Type: yes → hit the enter button
- Password: Cyberlab123! (Don’t worry the words will not appear on screen but they are there. Just hit the enter button).
- You are successfully logged in when you see labuser@linux-vm
- You can verify you are logged into the linux-vm by using this linux command to check your operating system: uname -a
- To exit PowerShell type “exit”.

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708828130/in/datetaken/" title="SSH into Linux"><img src="https://live.staticflickr.com/65535/53708828130_6973b7fd2d_c.jpg" width="800" height="765" alt="SSH into Linux"/></a>

## <ins>Create Attacker VM</ins>
- Note: This purpose of this VM is to impersonate an attacker using brute force in an attempt to gain access to the Windows and Linux VMs. 
- Azure Portal → Virtual Machine → Create 
- Create new Resource group called “RG-Cyber-Lab-Attacker”
- Name: attack-vm
- Region: Australia Central (it can be any location except where the other VMs are located)
- Image: Windows 10
- Size: Standard_E2bs_v5 -(2 VCPUs minimum)
- Username: labuser
- Password: Cyberlab123!
- Click box “I can confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights”  
- Networking  → Virtual network Create new
- Name: Lab-VNet-Attacker
- Review + Create 

<ins>Remote login to VM from RDC</ins>
- Open Remote Desktop Connection app
- Copy attack-vm Public IP address and paste into RDC
- Username: labuser
- Password: Cyberlab123!
- Copy the Public IP address of windows-vm and paste into Notepad (you will use this later)

<ins>Attacker Mode (pretend you are an attacker)</ins>
- From within the “attack-vm” attempt to RDC into “windows-vm” with the wrong username and password
- This will generate some failed RDP logs against “windows-vm”.
- Repeat this step 2 more times with the wrong username and password

## <ins>Generate some failed MS SQL Auth logs against “windows-vm”</ins>
- Install SSMS (SQL Server Management Studio) 
- https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms 
- Open SSMS app
- Attempt login with windows-vm IP address, wrong username and wrong password
- Server name: windows-vm IP address 
- Authentication: SQL Server Authentication
- Login: (the correct username is “sa”) but use a wrong username
- Password: use a wrong password
- Repeat this step 2 more times with the wrong username and password.
- Then login with the correct credentials below
- Username: sa
- Password: Cyberlab123!
- Click disconnect (the plug socket with the red x)

<ins>Generated some failed SSH logs against “linux-vm”</ins>
- Still within “attack-vm”, attempt to SSH into “linux-vm” with the wrong credentials
- Go to main PC and retrieve the linux-vm Public IP address (copy & paste this into Notepad on attack-vm).
- Open PowerShell within attack-vm
- Type: ssh (fake username@linux-vm Public IP address) e.g ssh junior@20.246.100.70
- Are you sure you want to continue connecting? Type: yes
- Attempt to login with a bad password three times.
- Shut down “attack-vm”, and go back to main PC

<ins>Admin Mode (pretend you are normal admin)</ins>
- From your own computer, RDC back into “windows-vm”.
- Copy & paste windows-vm Public IP address into RDC.
- Open Event Viewer → Windows Logs → Security
- Inspect the failures and successes by clicking under the Actions menu “Filter current Log” 
- Type into search bar “4625” (this filters the results to show all RDP failed logins).
- REMEMBER: RDP = Security Log 
- SQL Server = Application Log
- Click Windows Logs > Application (to see SQL logs)
- Type into search bar EventID: 18456 (this displays SQL Server failed logins)

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708607688/in/datetaken/" title="4625 Event Viewer"><img src="https://live.staticflickr.com/65535/53708607688_06dc5401b3_c.jpg" width="800" height="460" alt="4625 Event Viewer"/></a>

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708725729/in/datetaken/" title="Event Viewer"><img src="https://live.staticflickr.com/65535/53708725729_df24baab81_c.jpg" width="800" height="459" alt="Event Viewer"/></a>

## SSH into the Linux VM to observe the logs
- From main PC open up PowerShell
- Copy linux-vm Public IP address from Azure Portal
- In PowerShell type: ssh labuser@(Public IP address)
- password: Cyberlab123!
- To see your logs first type in this linux command: cd[spacebar]/var/log
- Then type “ls” to view the logs OR
- “ls -lasht” is a formatted view of logs 
- “cat auth.log” is for the full log information
- “cat auth.log | grep password” - This command is used to filter the logs for the word “password”.
- “cat auth.log | grep Accepted” - This command is used to filter for Accepted passwords. (Capital ‘A’ must be used in Accepted).

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53708725699/in/datetaken/" title="Password Invalid Linux logs"><img src="https://live.staticflickr.com/65535/53708725699_5e4a386e56_c.jpg" width="800" height="519" alt="Password Invalid Linux logs"/></a>

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53707483897/in/datetaken/" title="Password Accepted Linux logs"><img src="https://live.staticflickr.com/65535/53707483897_cfa5d7c3b6_c.jpg" width="800" height="315" alt="Password Accepted Linux logs"/></a>

## Conclusion 
I demonstrated how to brute force SSH and log cybersecurity incidents in Event Viewer. My objective was to highlight the importance of additional measures to strengthen the security posture of an organisation such as multifactor authentication and the necessity for strong passwords.
