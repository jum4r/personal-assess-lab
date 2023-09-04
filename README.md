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
- Powershell & Custom [Powershell Script](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1)
- [IP Geo Location](https://ipgeolocation.io/)

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)

<h2>Setup</h2>

### Create a Free Azure Account
<img width="700" alt="image" src="https://i.imgur.com/h1oEndK.png">

1. Sign up: [Azure Free Account](https://azure.microsoft.com/en-us/free/)
2. Login: [Azure Portal](https://portal.azure.com)

### Create Vulnerable Virtual Machine (VM)
<img width="700" alt="image" src="https://i.imgur.com/9DOlkgo.png">

1. Go to [Azure Portal](https://portal.azure.com).
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
<img width="700" alt="image" src="https://i.imgur.com/I7wUEho.png">

1. Search for <b>Log Analytics Workspaces</b> and create new.
2. Configure the LAW:
   - Resource Group: RG-Honeypot
   - Name: law-honeypot
   - Region: West US 3
3. Create the LAW.

### Configure Microsoft Defender for Cloud and Connect LAW to the VM
<img width="700" alt="image" src="https://i.imgur.com/rIhqyGG.png">

1. Search for <b>Microsoft Defender for Cloud</b>.
2. Navigate to <b>Environment Settings</b> > <b>Azure Subscriptions</b> > <b>law-honeypot</b>.
3. Under Azure Defender Plans, <b>Turn Azure Defender On</b>:
   - Servers: On
   - SQL Servers: Off
   - Save
   - Data Collection: All Events
   - Save
4. Head back to <b>Log Analytics Workspaces</b> > <b>Virtual Machines</b>.
6. Click <b>vm-honeypot</b> then <b>Connect</b>.

### Create Microsoft Sentinel (SIEM)
<img width="500" height="500" alt="image" src="https://i.imgur.com/WIsYbkw.png">

1. Search for <b>Microsoft Sentinel</b> and create new.
2. Select <b>law-honeypot</b> then <b>Add</b>.

<h2>Configuring the Virtual Machine as a Honeypot</h2>

### Log into the VM & Observe Event Viewer Logs
<img width="700" alt="image" src="https://i.imgur.com/LN4a8qE.png">

1. Navigate to <b>Virtual Machines</b> > <b>vm-honeypot</b>.
2. Copy the Public IP address.
3. Remote Desktop (RDP) into the VM.
4. Intentionally fail to log in once or twice by providing the wrong username.
5. Now login with the correct credentials.
6. Inside the VM search <b>Event Viewer</b> and open.
7. Navigate to  <b>Windows Logs</b> > <b>Security</b>.
8. Observe the security event logs.
9. Particularly <b>Event IDs 4625</b>, as they represent failed logon attempts.

### Turning the VM Firewall Off
<img width="700" alt="image" src="https://i.imgur.com/jAqxpjz.png">

1. Back on your PC, search and open <b>cmd</b>.
2. Type <b>ping -t (your vm public ip)</b>, to iniate non-stop ping.
3. Return to the VM and search <b>wf.msc</b> to open the Firewall Settings.
4. Click <b>Windows Defender Firewall Properties</b>
5. Domain Profile tab
   - Firewall State: Off
6. Private Profile tab
   - Firewall State: Off
7. Public Profile tab
   - Firewall State: Off
8. Apply then OK.
9. Back on your PC, observe cmd.
10. Ensure that ping is succesful then press <b>ctrl + c</b> to stop ping.

<h2>Configuring Logging Data for Visualization</h2>

### Using Powershell with IP Geolocation to Create Custom Logs
<img width="700" alt="image" src="https://i.imgur.com/AMmdgy5.png">

1. Back in the VM, open the browser.
2. Head to [Github](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) and copy the script.
3. Windows search for <b>Powershell ISE</b> and open it.
4. Click <b>New Script</b> and then paste the script.
5. Click <b>Save Script</b> and save it to the Desktop as <b>Log_Exporter</b>.
6. Go to [IP Geo Location](https://ipgeolocation.io/) and create an account.
7. Copy your unique API key.
8. Back in Powershell, look for <b>$API_KEY</b>.
9. Paste your unique API Key over the existing one.
10. Run the script.
11. Press the Windows key + R, then type <b>C:\ProgramData</b>.
12. Open the <b>failed_rdp</b> text file and copy everything.
13. Minimize the VM.
14. Back on your PC, open Notepad, and paste the logs.
15. Save it to the desktop and name it <b>failed_rdp.log</b>

### Synchronizing Logs Between LAW and the VM
<img width="500" height="500" alt="image" src="https://i.imgur.com/MuVqzP4.png">

1. Head back to [Azure Portal](https://portal.azure.com)
2. Go to <b>Log Analytics Workspaces</b> > <b>law-honeypot</b>
3. Navigate to <b>Tables</b> > <b>Create custom log</b>
   - Sample Log: Point it to the <b>failed_rdp.log</b>
   - Record Delimiter: New line
   - Collection Path: Windows; <b>C:\ProgramData\failed_rdp.log</b>
   - Custom Log Name: FAILED_RDP_WITH_GEO
4. Create Custom Log
5. It will take a while for Log Analytics Workspace and the VM to synchronize; please allow 10-15 minutes.

### Configuring Sentinel for Attack Visualization
<img width="700" alt="image" src="https://i.imgur.com/BEo3DuW.png">

1. Head to the [Azure Portal](https://portal.azure.com).
2. Then <b>Microsoft Sentinel</b> > <b>Workbooks</b> > <b>Create new Workbook</b>
3. Click <b>Edit</b> then remove both of the existing widgets.
4. Click <b>Add</b> > <b>Add Query</b>
5. In a new tab, open [Github](https://github.com/jum4r/siem_workbook_script) and copy the entire script.
6. Paste the script into the Query box, then click <b>Run Query</b>.
7. In Query, Configure:
   - Visualization: Map
   - Size: Full
8. Configure Map Settings:
   - Location Info: Latitude/Longitude
   - Size by: event_count
   - Metric Label: label
   - Metric Value: event_count
9. Click <b>Apply</b>, then <b>Save and close</b> Map Settings.
0. On top click <b>Save</b>:
   - Title: Failed RDP World Map
   - Location: West US 3
11. Click <b>Save</b> then <b>Done Editing</b>.
12. Turn on <b>Auto Refresh</b> for 5 minutes. <b>Save</b>.
13. Leave the VM open with PowerShell still running its script.
14. It might take a while before some login attempts occur, so you can leave and return in a few hours.

<h2>Monitoring Real-Time Global Cyberattacks</h2>
<img width="700" alt="image" src="https://i.imgur.com/6bmtAJ8.png">

1. After a few hours, analyze the newly acquired data.
2. Take a look at where the attacks are happening on the map. It's fascinating.
4. You can change Map Settings to view data differently.
5. You may leave it open for a few more hours, but please keep in mind that the IP Geolocation free account has a limit of 1000 requests. If you would like more requests there are payment options on their website.
6. After you've finished, close the VM.

<h2>Cleanup</h2>

### Deleting Resource Groups
<img width="700" alt="image" src="https://i.imgur.com/W6MxJTI.png">

1. Go back to [Azure Portal](https://portal.azure.com) and navigate to <b>Resource Groups</b>
2. Select <b>RG-Honeypot</b> > <b>Delete resource group</b>
3. Once it's finished, head back to <b>Resource Groups</b> > <b>NetworkWatcherRG</b> > <b>Delete resource group</b>.
4. Refresh/Revisit [Azure Portal](https://portal.azure.com) to ensure no resource groups are left.
5. Congratulations, we've finished!

 


