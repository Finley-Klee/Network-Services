# Network-Services
This room is the first of two parts exploring a variety of network services. Part one includes SMB, Telnet, and FTP. The TryHackMe room can be found here: https://tryhackme.com/r/room/networkservices


<h2>Utilities Used</h2>

- <b>Terminal</b> 
- <b>nmap</b>
- <b>SMB</b>
- <b>Enum4Linux</b>
- <b>SMBClient</b>
- <b>SSH with a private key</b>
- <b>Telnet</b>
- <b>FTP</b>
- <b>Hydra</b>

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

- <b>Enumberating Telnet</b>
<p>Moving on to a new target machine, I'm searching for information on this machine with regard to the next networking service, Telnet. Telnet is an old, plaintext service which has largely been replaced by secure shell because Telnet lacks encryption and ssh is encrypted.</p>
<br>
<p align="center">I ran an nmap scan using the same flags as in the SMB section and discovered one open port. (If you run the scan without the -p- flag for all ports, the scan will return no open ports)<br/>
  <img src="https://github.com/user-attachments/assets/bd90cf37-127b-4c87-ba43-6bf7a27b13af" height="80%" width="80%" alt="nmap scan output with the line highlighted showing that an open port has been found: tcp port 8012"/>
  <br />
  <br />
   This open port is a nonstandard port, but because of the information gleaned from the nmap scan we can see that it has a title of "SKIDY'S BACKDOOR" so based on the title I can answer the questions for what it's used for and who it likely belongs to.<br />
  <img src="https://github.com/user-attachments/assets/19170d03-508b-4788-85e4-5dc809cdab9d" height="80%" width="80%" alt="in the info of the nmap scan the words SKIDY'S BACKDOOR are highlighted"/>
</p>

- <b>Exploiting Telnet</b>
<p>Now that I have found Skidy's backdoor on port 8012, I can use it to connect using the Telnet service.</p>
<br>
<p align="center">I connected to telnet over port 8012 and received the welcome message "SKIDY'S BACKDOOR" and tried a few commands, but my inputs didn't lead to anything being returned.<br/>
  <img src="https://github.com/user-attachments/assets/ce170602-2108-4504-b53c-e96196d81012" height="80%" width="80%" alt="a linux terminal with the telnet command to connect to the service followed by the commands ls and pwd with no return"/>
  <br />
  <br />
  So to check if my input is being executed as a system command I set up a tcpdump listener on the attack box.<br />
  <img src="https://github.com/user-attachments/assets/4ec3425c-8a46-47c6-9a83-942534c91bd8" height="80%" width="80%" alt="two terminal windows, one in front of the other, the front window is smaller than the back window and text from both can be seen though the top left of the back window is obscured by the front. The front terminal shows the tcpdum listener setup."/>
  <br />
  <br />
  I then ran the ping command from the telnet machine to check if the commands from that machine were executing, which resulted in the listener picking up the ping, proving that the commands are properly executing.<br />
  <img src="https://github.com/user-attachments/assets/5671682c-36fc-42da-8261-2255ae320db8" height="80%" width="80%" alt="those same two terminal windows, but now the back window shows the .RUN command followed by the command to ping the attackbox and the front window shows that the tcpdump listener received the ping."/>
   <br />
  <br />
 Great! So now to create a reverse shell I used the command syntax given to me in the TryHackMe room to create an msfvenom payload (which starts with mkfifo).<br />
  <img src="https://github.com/user-attachments/assets/c7a35ee2-936a-4bb5-824d-977e4864f6f2" height="80%" width="80%" alt="in a separate terminal winodw, an msfvenom command with the attackbox ip address listed for the host and the port to use designated as 4444, which then output a payload which is highlighted in white and begins with mkfifo."/>
   <br />
  <br />
  I ran the payload on the Telnet machine which allowed me to establish a connection through a reverse shell in the terminal on the attackbox. That allowed me to list the contents where I found a text file named flag.txt. I concatenated flag.txt and found the flag!<br />
  <img src="https://github.com/user-attachments/assets/65b2b14f-8880-4463-9c85-1bfd24e4addd" height="80%" width="80%" alt="once again there are two terminal windows, a front and a back. The front window is the attackbox terminal and the back window is the Telnet service. At the bottom of the text on the back window the .RUN command followed by the msfvenom payload can be seen, and on the front window this resulted in a connection received. Below the connection received line reads ls, flag.txt, cat flag.txt THM you got the telnet flag."/>
</p>

- <b>Enumerating FTP</b>
<p>Once again I'm searching for information on a new target machine.</p>
<br>
<p align="center">I ran another nmap scan and discovered ftp running on the usual port, tcp 21.<br/>
  <img src="https://github.com/user-attachments/assets/1e5bd50c-cd6a-4fc9-837c-94f10da84efd" height="80%" width="80%" alt="the nmap scan results with the port discovered highlighted as port 21 open running ftp."/>
  <br />
  <br />
 Scrolling down to the results summary, I found the version of ftp that's running is vsftpd.<br />
  <img src="https://github.com/user-attachments/assets/27f47367-30b7-41b3-800e-9f1004a45016" height="80%" width="80%" alt="another view of the nmap scan results, this time with the ftp version highlighted, reading vsftpd."/>
  <br />
  <br />
  I could also see from the nmap scan that anonymous logon is possible, so I logged on to ftp using the anonymous user name and no password. I listed the contents of the ftp server and found a public notice document. I transfered the public notice back to the attack box to read. <br />
  <img src="https://github.com/user-attachments/assets/d976d50a-23da-40b3-83cf-fe4f068a7071" height="80%" width="80%" alt="command line interface showing an anonymous logon to the ftp server, followed by listing the contents, then getting the public_notice.txt document, and lastly logging out of the ftp."/>
   <br />
  <br />
  For the last step in the enumeration phase, I concatenated the contents of the public notice, which is how I found a possible user named Mike.<br />
  <img src="https://github.com/user-attachments/assets/fee38986-a4bb-4795-8177-b4b410e49453" height="80%" width="80%" alt="linux terminal showing the listing of directories and files including PUBLIC_NOTICE.txt, then the cat command for public_notice.txt which reads message from system administrators, hello, I hope everyone is aware that the FTP server will not be available over the weekend - we will be carrying out routine system maintenance. Backups will be made to my account so I recommend encrypting any sensitive data. cheers, Mike."/>
</p>

- <b>Exploiting FTP</b>
<p>Now that I have an idea of a username, I need to figure out the password and use those credentials to log in to Mike's ftp server.</p>
<br>
<p align="center">I used Hydra to brute force Mike's password with the command hydra and the tags -t 4 to run 4 parallel connections on the target, -l Mike, to designate the username, -P /usr/share/wordlists/rockyou.txt to point to the wordlist to use as passowrds to try, -vV to increase the verbosity of the response, then the ip address, and lastly ftp to indicate the protocol.<br/>
  Then I waited a long time for Hydra to try the passwords.<br />
  Like, hours.<br />
  Only to realize later that if I had to extend the machine timers I probably did something wrong, because the room should be doable within the alloted time to run the machines, so I looked up what I might be doing wrong and discovered that the username shouldn't be capitalized and that the password is "password". When I ran hydra again with the correct username the answer popped up within seconds, so you live and you learn.<br />
  <img src="https://github.com/user-attachments/assets/3957bc25-e889-481e-935c-c7bbd9f0df06" height="80%" width="80%" alt="linux terminal showing the hydra command and the output found a successful username and password combination of mike and password."/>
  <br />
  <br />
 Then I used the username and password to log in to the ftp server and get the file ftp.txt. I concatenated on the attack box to find the flag.<br />
  <img src="https://github.com/user-attachments/assets/85e84f6a-86d3-4483-9fd8-6eace6c8e0b1" height="80%" width="80%" alt="linux terminal with commands to log in to ftp and get the ftp.txt file, which when concatenated reads THM you got the ftp flag in obfuscated text with numbers in place of some letters and underscores between the words."/>
</p>

