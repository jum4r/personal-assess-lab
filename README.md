<p align="center">
<img width="450" height="250" src="https://i.imgur.com/njQs1EI.png" alt="SIEM Traffic"/>
</p>

<h1>Utilizing Security Information and Event Management (SIEM) for Geospatial Attack Data Visualization</h1>
In this lab, we will set up a Virtual Machine (VM) and expose it to the internet. We will then set up the Log Analytics Workspace combined with Sentinel (SIEM) to map out attacks from different parts of the world.

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Computer)
- Microsoft Azure (Log Analytics Workspaces)
- Microsoft Azure (Microsoft Sentinel [SIEM])
- Microsoft Azure (Microsoft Defender for Cloud)
- Powershell Script
- [IP Geo Location](https://ipgeolocation.io/)

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)

<h2>High-Level Steps</h2>

- Note
- Note
- Note
- Note
- Note

<h2>Setup</h2>

### Create a Free Azure Account
<img width="786" alt="image" src="https://i.imgur.com/h1oEndK.png">

1. Sign up: [Azure Free Account](https://azure.microsoft.com/en-us/free/)
2. Login: [Azure Portal](https://portal.azure.com)

### Create Vulnerable Virtual Machine (VM)
<img width="786" alt="image" src="https://i.imgur.com/9DOlkgo.png">

1. Go to [Azure Portal](https://portal.azure.com)
2. Search for Virtual Machines and create a new Virtual Machine.
3. Configure the VM:
   - Resource Group: RG-Honeypot
   - VM Name: vm-honeypot
   - Region: West US 3 (or any region you prefer)
   - Image: Windows 10 Pro
   - Size: 2 vCPUs, 8 GiB memory
   - Username: siemadmin / Password123!
   - <b>Networking</b>
   - NIC Network Security Group: Advanced
   - Configure Network Security Group: Create New
   - Delete existing Inbound Rules
   - Add an Inbound Rule
   - Destination Port Ranges: *
   - Protocol: Any
   - Action: Allow
   - Priority: 100
   - Name: DANGER_ALL_TRAFFIC
4. Create the VM.

### Create Log Analytics Workspaces (LAW)
<img width="786" alt="image" src="https://i.imgur.com/I7wUEho.png">

1. Search for Log Analytics Workspaces and create new.
2. Configure the LAW
   - Resource Group: RG-Honeypot
   - Name: law-honeypot
   - Region: West US 3
3. Create the LAW

### Configure Microsoft Defender for Cloud and Connect LAW to the VM
<img width="786" alt="image" src="https://i.imgur.com/rIhqyGG.png">

1. Search for Microsoft Defender for Cloud
2. Navigate to <b>Environment Settings</b> > <b>Azure Subscriptions</b> > <b>law-honeypot</b>
3. Under Azure Defender Plans Turn Azure Defender On:
   - Servers: On
   - SQL Servers: Off
   - Save
   - Data Collection: All Events
   - Save
4. Head back to Log Analytics Workspaces page
5. On the left pane look for Virtual Machines
6. Click vm-honeypot then Connect

### Create Microsoft Sentinel (SIEM)
<img width="786" alt="image" src="https://i.imgur.com/WIsYbkw.png">

1. Search for Microsoft Sentinel and create new.
2. Select law-honeypot then Add

### Log into VM & Observe Event Viewer Logs
<img width="786" alt="image" src="https://i.imgur.com/LN4a8qE.png">

1. Search for Virtual Machines
2. Select vm-honeypot and copy Public IP address
3. Remote Desktop Connect (RDP) into the VM
4. Intentionally fail to log in once or twice by providing the wrong username.
5. Now login with the correct credentials.
6. Inside the VM search Event Viewer and open
7. Navigate to  <b>Windows Logs</b> > <b>Security</b>
8. Observe the security event logs
9. Particularly Event IDs 4625, as they represent failed logon attempts.

### Turn VM Firewall Off
<img width="786" alt="image" src="https://i.imgur.com/jAqxpjz.png">

1. Back on your PC, search and open cmd
2. Type ping -t (your vm public ip), to iniate non-stop ping
3. Return to the VM and search wf.msc to open the Firewall Settings.
4. Click Windows Defender Firewall Properties
5. Domain Profile tab
   - Firewall State: Off
6. Private Profile tab
   - Firewall State: Off
7. Public Profile tab
   - Firewall State: Off
8. Apply then OK
9. Back on your PC, observe cmd
10. Ensure that ping is succesful then ctrl + c to stop ping

### Using Powershell to Create Custon Logs
<img width="786" alt="image" src="https://i.imgur.com/AMmdgy5.png">

1. Back in the VM, open the browser
2. Head to [Powershell Script](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) and copy the script
3. Windows search Powershell ISE and open
4. Click New Script then paste the script
5. Click Save Script; save to Desktop as Log_Exporter
6. Go to [IP Geo Location](https://ipgeolocation.io/) and create a free account
7. Copy your unique API key
8. Back in Powershell, look for $API_KEY
9. Paste your unique API Key over the existing one
10. Run the script
11. Press Windows key + r then type C:\ProgramData\
12. Open failed_rdp text file and copy everything
13. Minimize the VM
14. Back on your PC open Notepad and paste the logs
15. Save to desktop and name it failed_rdp.log

### Using Powershell to Create Custon Logs
<img width="786" alt="image" src="https://i.imgur.com/MuVqzP4.png">

1. Head back to [Azure Portal](https://portal.azure.com)
2. Search Log Analytics Workspaces > law-honeypot
3. Navigate to Tables > Create custom log
   - Sample Log: Point it to the failed_rdp.log
   - Record Delimiter: New line
   - Collection Path: Windows; C:\ProgramData\failed_rdp.log
   - Custom Log Name: FAILED_RDP_WITH_GEO
4. Create Custom Log
5. It will take a while for Log Analytics Workspace and the VM to synchronize; please allow 10-15 minutes

### Using Powershell to Create Custon Logs
<img width="786" alt="image" src="https://i.imgur.com/BEo3DuW.png">

1. [Azure Portal](https://portal.azure.com)
 


