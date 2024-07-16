# Network-Services
This room is the first of two parts exploring a variety of network services. Part one includes SMB, Telnet, and FTP. The TryHackMe room can be found here: https://tryhackme.com/r/room/networkservices

<h2>Description</h2>
Insert a short description of the project

<h2>Utilities Used</h2>

- <b>Terminal</b> 
- <b>nmap</b>
- <b>Enum4Linux</b>

<h2>Environments Used </h2>

- <b>Linux Ubuntu</b>

<h2>Project walk-through:</h2>

- <b>Enumerating SMB</b>
<p>To gather information about our target machine I'm using the Enum4Linux tool, which is described in this room as a wrapper around the tools in the Samba package to extract information about a target pertaining to SMB.</p>
<br>
<p align="center">First, however, we need to run a quick nmap port scan to discover which ports are open and on which port or ports SMB is running. I used the -vv tag to increase the verbosity of the scan output, the -T5 flag to use the fastest timing template because in a TryHackMe lab I'm not concerned about being stealthy and didn't want to wait a long time for the scan, the -A tag to gather service and OS information with the scan, and the -p- tag to scan all ports. According to our port scan there are 3 open ports. SSH running on tcp port 22, and SMB (Server Message Block) running on tcp ports 139 and 445. (These are the answers to the first two questions for this section on TryHackMe)<br/>
  <img src="https://github.com/user-attachments/assets/8bcbacce-9e64-459c-ad64-37ebd0f309f3" height="80%" width="80%" alt="A Linux termal with the nmap command scanning the target machine and lots of output"/>
  <img src="https://github.com/user-attachments/assets/edb57853-6fde-4d7b-a0dd-a8eaec8f095a" height="80%" width="80%" alt="In this image of the output of the nmap scan we get details of the open ports which are 22, 139, and 445 running ssh, and smb."/>
  <br />
  <br />
  Next, I ran a full basic scan with enum4linux (the -a tag) and discovered that the workgroup name is simply "WORKGROUP" and the name of the machine is "POLOSMB".<br />
  <img src="https://github.com/user-attachments/assets/7baeb49d-1100-49cb-8915-7b7304ef6472" height="80%" width="80%" alt="the start of the enum4linux output which shows information such as known usernames, the working group name, etc."/>
  <br />
  <br />
  Scrolling down in the output, we find the OS information section. This is where I found the answer to the next question, the operating system version running is 6.1 <br />
  <img src="https://github.com/user-attachments/assets/7515a1cb-e320-4098-9f08-fb64a04fd774" height="80%" width="80%" alt="in the OS information section of the output, the OS version line is highlighted in white shoing the OS version number is 6.1"/>
   <br />
  <br />
  Lastly for this enumeration section, I scrolled down to the share enumeration section, where we see that there is a share called profiles. This is definitely of interest as we move into exploitation in the next section. <br />
  <img src="https://github.com/user-attachments/assets/be544bdf-029a-4c7e-92a3-ece11cbbcb37" height="80%" width="80%" alt="in the share enumeration section there are 4 entries: netlogon, profiles, print$ and IPC$. Profiles is highlighted as this is the one asked for in the question."/>
</p>
<br />
<br />
- <b>Section Name</b>
<p>Description</p>
<br>
<p align="center">Step One: <br/>
  <img src="" height="80%" width="80%" alt="image one"/>
  <br />
  <br />
  Step Two: <br />
  <img src="" height="80%" width="80%" alt="image two"/>
  <br />
  <br />
  Step Three: <br />
  <img src="" height="80%" width="80%" alt="image three"/>
   <br />
  <br />
  Step Four: <br />
  <img src="" height="80%" width="80%" alt="image four"/>
   <br />
  <br />
  Step Five: <br />
  <img src="" height="80%" width="80%" alt="image five"/>
</p>

<img width="1117" alt="Screenshot 2024-07-16 at 2 08 41 PM" src="">
