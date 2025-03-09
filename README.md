<p align="center">
    <img src="/images/banner.png" width="1152" alt="Banner">
</p>

## Overview  
A secure, self-hosted file server using Tailscale (zero-trust VPN) and FileBrowser, featuring:  
- 🔒 Traffic restricted to Tailscale devices via firewall rules  
- ⚡ Automated service startup with Task Scheduler  
- 👩💻 Granular user permissions (admin vs. restricted roles)  

## Step 1: Tailscale Setup (Zero-trust VPN) on Host PC

1. Download Tailscale for Windows and install

2. Sign up for a free account (personal use accounts allow 20 devices)

3. Right-click the Tailscale app in the system tray, hover over the account name, and then click `Add Panel…` and verify that the current system has been added to the device list

4. Run `tailscale status` in Command Prompt to confirm that the device is Active

## Step 2: FileBrowser Setup (Lightweight Web File Manager)

1. Install FileBrowser for Windows by running the command:

    ```powershell
    iwr -useb https://raw.githubusercontent.com/filebrowser/get/master/get.ps1 | iex
    filebrowser -r /path/to/your/files
    ```

2. Create the `filebrowser.json` configuration file in the `C:\Program Files\filebrowser` folder with the following contents:
    
        
    ```json
    {
    "root": "D:\\Put-Media-Drive-Here",
    "address": "Put-Tail-Scale-IP-Address-Here",
    "port": 8080
    }
    ```
    In the code snippet above `Put-Media-Drive-Here` should be replaced for the drive to be used by the file server and `Put-Tail-Scale-IP-Address-Here` should be replaced with the Tailscale IP address. To find the Tailscale IP Address, run `tailscale ip` in Command Prompt

## Step 3: Schedule Windows Startup Task for FileBrowser with Custom Config

1. Run `Task Scheduler` as administrator and click `Create Task…`

2. Fill out the following settings and then click `OK`:

    - **General Tab:**

        - Name: `FileBrowser Auto-Start`

        - Tick: ✅`Run whether user is logged on or not`

        - Tick: `Run with highest privileges`

    - **Triggers Tab:**

        - Click `New…`, select `At startup` (delayed by 30 seconds if needed), and click `OK`

    - **Actions Tab:**

        - Action: "Start a program"

        - Program/script: cmd.exe

        - Arguments: 
            ```cmd
            /c "cd /d C:\Program Files\filebrowser && filebrowser --config filebrowser.json"
            ```

<img src="/images/scheduled-task.png" alt="Scheduled Task">

3. Input the admin credentials to approve the creation of this new task

4. Restart the host PC and run the following command in Command Prompt to verify the established connected:
    ```cmd
    netstat -ano | findstr :8080
    ```

## Step 4: Firewall/Router Rules

1. Allow inbound TCP port 8080 (FileBrowser’s port) only through Tailscale by running the following command in PowerShell:

```powershell
# Allow FileBrowser access ONLY via Tailscale
New-NetFirewallRule -DisplayName "FileBrowser (Tailscale Only)" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080 -InterfaceAlias "Tailscale" -RemoteAddress 100.64.0.0/10
```

<img src="/images/firewall-rule.png" alt="Firewall Rule">

## Step 5: Configuring Secure FileBrowser Credentials

1. Open a browser and navigate to `http://Put-Tail-Scale-IP-Address-Here:8080` (e.g. `http://100.101.102.103:8080`)

2. Enter the default credentials for FileBrowser
    - Username: admin
    - Password: admin

3. Navigate to `Settings` > `User Management` and edit the default admin credentials to be more secure

<img src="/images/example-admin.png" alt="Example Admin">

4. Add a second user:
    - Username: jane.doe
    - Password: Brevvt15!
    - Untick Administrator
    - Untick Delete files and directories

<img src="/images/jane-doe.png" alt="Jane Doe">

5. Drag and example document into the `My Files` tab

<img src="/images/example-document.png" alt="Example Document">

## Step 6: Remote Access Testing
- From the host PC:
    - Right-click the Tailscale app in the system tray, hover over the account name, and then click `Add Panel…`
    - Go to the `Users` tab and click `Invite external users`
    - Invite an example employee (Jane Doe) to the file server for testing purposes

- From a remote device (outside the network):

    - From the email of the example employee, accept the Tailscale invitation and download and log-in to the Tailscale client

    - Connect to the host PC's network

    - Open a browser and navigate to `http://Put-Tail-Scale-IP-Address-Here:8080` (e.g. `http://100.101.102.103:8080`)

    - Log-in with their FileBrowser credentials
        - Username: jane.doe
        - Password: Brevvt15!

    - Test Jane Doe's privileges:
        - View: Open the example document that was previously added on the host PC
        - Upload: Drag-and-drop a new file from the remote device into the file server
        - Download: Click a file to download it to the remote device.
    
    <img src="/images/access-server-remotely.png" alt="Access Server Remotely">
