# Network-Services
This room is the first of two parts exploring a variety of network services. Part one includes SMB, Telnet, and FTP. The TryHackMe room can be found here: https://tryhackme.com/r/room/networkservices

<h2>Description</h2>
Insert a short description of the project

<h2>Utilities Used</h2>

- <b>Terminal</b> 
- <b>nmap</b>
- <b>Enum4Linux</b>
- <b>SMBClient</b>

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
- <b>Exploiting SMB</b>
<p>Now that I've gathered some information about SMB on my target, I'm going to use the anonymous SMB share access misconfiguration to gain information that will lead to shell access.</p>
<br>
<p align="center">In order to access the SMB share we need to use a client and in this room I'm using the Samba default SMBClient. To answer the first question I used the given syntax smbclient //[IP]/[SHARE] to and the tags -U [name] and -p [port] to specify the username and the port to practice writing the command. <br/>
  <img src="https://github.com/user-attachments/assets/21348724-8827-4d45-af76-e373f6c497a0" height="80%" width="80%" alt="question and answer from the TryHackMe side with a white background and black text. The answer portion is inside a rectangular box with rounded corners and a light gray background. The question reads What would be the correct syntax to access an SMB share called secret as user suit on a machine with the IP 10.10.10.2 on the default port? and inside the box is the answer smbclient //10.10.10.2/secret -U suit -p 445 to the right is a bright green rectangle with a black checkmark to the left of the words correct answer"/>
  <br />
  <br />
  So now that I know the syntax I'm going to try it on my target using the "Anonymous" user and try not providing a password and see what happens. It worked! This SMB share allows anonymous access.<br />
  <img src="https://github.com/user-attachments/assets/f6e7b867-3f5b-42bb-a6c7-25e10ddf3f02" height="80%" width="80%" alt="a linux terminal with the command smbclient //10.10.85.118/profiles -U anonymous -p 445 on the top line. The second line prompts for the user's password which I left blank and then the next line reads try help to get a list of possible commands followed by a new smb command line which starts with smb: "/>
  <br />
  <br />
  I started with an ls command to list the contents of the share, then tried to concatenate the contents of the text file "Working From Home Information.txt", but that command was not found, so I used the help command to see a list of commands.<br />
  <img src="https://github.com/user-attachments/assets/9e4dae75-c28e-4718-ba25-fab9e61caa5a" height="80%" width="80%" alt="output from the ls command showing a few directories and a text file following by my failure to use cat and then a list of commands returned from the help command."/>
  <br />
  <br />
 I tried a few commands to see if I could read the text file directly from the SMB share, but I couldn't quite figure out, but in the process I downloaded the file to the attackbox machine, so I logged out and quit so I could concatenate the file from the terminal. Reading this letter, I see that it is addressed to John Cactus, so I can assume that this profile folder belongs to John Cactus. (The answer to the 4th question) and I also see here that James, the department manager, has configured ssh to allow him to work from home (answer to the 5th question).<br />
  <img src="https://github.com/user-attachments/assets/76379a01-f7fb-42c5-a13b-b83325b42fb8" height="80%" width="80%" alt="in the first part I tried the get, mget, and open commands in smb, then logged off and in my terminal on attack box the ls command shows the text file which I printed to the terminal with cat and it reads John Cactus
As you're well aware, due to the current pandemic most of POLO inc. has insisted that, wherever possible, employees should work from home. As such- your account has now been enabled with ssh access to the main server. If there are any problems, please contact the IT department at it@polointernalcoms.uk Regards, James Department Manager"/>
  <br />
  <br />
  So I logged back into SMB to check out the ssh directory. Is used the change directory command (cd) to move to that directory and list it's contents, where I can see there are authentication keys for ssh. The id_rsa.pub is likely the public key to an asymmetrical key pair, so I think the most useful key is going to be id_rsa, which is likely the private key for that key pair. So I am going to "get" that file and then in my terminal I used chmod to change the file permissions to 600 so that I can both read and write to the file. I then used "cat" to check that I could read the file and I can.<br />
  <img src="https://github.com/user-attachments/assets/6919230a-1c69-4e3e-ac46-0d22ceb971d9" height="80%" width="80%" alt="smb output in the terminal showing an ls listing the directories and file in the smb share, then a cd command to the .ssh directory, followed by an ls of that directory which shows several keys including id_rsa. Then the get command for id_rsa file"/>
  <img src="https://github.com/user-attachments/assets/fb36d9d2-97ee-44ec-ae05-6f33990ccee5" height="80%" width="80%" alt="back in the attack box terminal an ls showing the id_rsa key has been succesfully downloaded, then the chmod command changing the file permissions and lastly the cat of the key bellow which the full rsa key can be seen."/>
  <br />
  <br />
I needed to try a couple of different combinations of John Cactus's name to find the correct one to successfully connect to ssh with the private key, which I added to the ssh command with the -i flag followed by the file name where the key is located.<br />
  <img src="https://github.com/user-attachments/assets/a22bbfda-8a02-4d9c-86b8-6e725662ff72" height="80%" width="80%" alt="multiple attempts to log into ssh with the usernames JohnCactus, John, and Cactus (all with capital letters) and finally a successful attempt with cactus all lowercase."/>
  <br />
  <br />
  Finally, now that I've logged into secure shell (ssh) I listed the content and found the flag in the smb.txt file!<br />
  <img src="https://github.com/user-attachments/assets/5f56a230-f738-42cd-9e47-c80c2daac9a9" height="80%" width="80%" alt="the ls in the ssh only shows one file titled smb.txt and when you cat that file you see the flag which reads THM{smb_is_fun_eh?}"/>
</p>

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
<img width="300" alt="Screenshot 2024-07-16 at 3 30 55 PM" src="">
