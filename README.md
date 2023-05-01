<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1
- Step 2
- Step 3
- Step 4

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://imgur.com/BzAgmOc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
First step in this tutorial is to set up our recources in Azure. We need to create two Virtual Machines, the first VM is going to be named "DC-1", this VM is going to be the Domain Controller and its image is Windows Server 2022. Create a second VM called "Client-1", using Windows 10 image. Makes sure both VMs are using to same Resource Group and Vnet, you can check the topology with Network Watcher, it should look like the image shown above.
</p>
<br />

<p>
<img src="https://imgur.com/ktsgu7F.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/iubpKFJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next step is to set the Domain Controller's(DC-1) NIC Private IP adress to be static. To do this go to DC-1 VM -> Networking -> Network Interface: dcXXX -> IP Configurations -> ipconfig1 -> Static -> Save
</p>
<br />

<p>
<img src="https://imgur.com/NuDHK5o.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now lets enusre connectivity between the client and Domain Controller. First login to Client-1 with Remote Desktop and ping DC-1's private IP address. To do this, open Command Line in the VM and type "ping x.x.x.x -t".Notice that the request is timing out. We need enable ICMPv4 traffic in on the local Windows firewall on DC-1. 
</p>
<br />

<p>
<img src="https://imgur.com/JuOHjUd.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/7PG4bM1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
To enable ICMPv4, open Windows Defender FIrewall with Advanced Security -> Inbound Rules -> Enable All ICMPv4. Then check back at Client-1 to see the ping succeed.
</p>
<br />

<p>
<img src="https://imgur.com/dMWWzX7.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/U6DYlnD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/xOZ1yWp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Great! Next we are going to install Active Directory, first login to DC-1 and you will be greeted with the Windows server manager dashboard. Click on Add Roles and Features -> Server Roles -> Active Directory Domain Services -> Next then Install. When that is done installing you should recieve a notification in the top right, click on it and go to "Promot this server to a domain controller". Then you will want to add a new forest and name it "mydomain.com" and click next. Once that is done, restart and then log back into DC-1 as user: mydomain.com\labuser.

</p>
<br />

<p>
<img src="https://imgur.com/qxj3IKS.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Cool! Now we are goin going to create an admin and a normal user account in Active Directory. First, open Active Directory Users and Computers in the search bar.
</p>
<br />

<p>
<img src="https://imgur.com/M0GtGZ5.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/sL9NYcT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next, right click mydomain.com -> New -> Create Organizational Unit. Create a OU named "_EMPLOYEES" and another one named "_ADMINS". Create a new employee named “Jane Doe” (same password) with the username of “jane_admin”.
</p>
<br />

<p>
<img src="https://imgur.com/91YdwOJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Add jane_admin to the “Domain Admins” Security Group, to do this right click on Jane Doe then go to Properties -> Member Of -> Add -> Domain Admins -> Ok. Next, Log out/close the Remote Desktop connection to DC-1 and log back in as “mydomain.com\jane_admin”, User jane_admin as your admin account from now on
</p>
<br />

<p>
<img src="https://imgur.com/ILCxRWG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://imgur.com/ZSw8pqE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Awesome! Now we are going to join Client-1 to our domain(mydomain.com). From the Azure Portal navigate to Virtual Machines -> Client-1 Networking -> DNS Servers -> set Client-1’s DNS settings to the DC’s Private IP address -> Save. Next restart Client-1's VM from the Azure portal, when its done restarting, login to Client-1 (Remote Desktop) as the original local admin (labuser). We are going to join Client-1 to the domain now, to do this open Settings -> About -> Rename this PC -> Computer name -> Change -> Domain: mydomain.com -> Ok -> PC will restart.
</p>
<br />

<p>
<img src="https://imgur.com/94I2vi7.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Perfect! Now lets verify Client-1 is on the domain by logging back into DC-1, going to Active Directory Users and Computers, and checking the "Computers" folder. We should see Client-1 listed.
</p>
<br />

<p>
<img src="https://imgur.com/m0hqRJ8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next, lets setup remote desktop for non-administrative users on Client-1. First, log into Client-1 as mydomain.com\jane_admin and open system properties and click “Remote Desktop”. Then allow “domain users” access to remote desktop by clicking "add". You can now log into Client-1 as a normal, non-administrative user.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Almost done! Moving on we are now going to creat a bunch of additional user and attempt to log into Client-1 with one of the users. First login to DC-1 as jane_admin. Open Powershell_ise as an administrator.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />
