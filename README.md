# Onedrive-to-Local-Virtual-Machine-using-FreeFileSync
Guideline to  Transfering Onedrive to Backup Server

Proxmox VM Auto-Sync with FreeFileSync
VM IP: 192.168.88.52
Status: Production Ready
Last Updated: September 2026

Project Overview
This project establishes an automated file synchronization workflow where a laptop connects to a VM hosted on Proxmox (192.168.88.52), and files are automatically synced between the VM and a target destination using FreeFileSync with RealTimeSync for automation.

Key Features
Seamless laptop-to-VM connectivity via SSH/RDP

Automatic file synchronization on change detection

RealTimeSync runs on VM startup (no manual intervention)

Supports local drives, network shares, and USB destinations

Repository Structure
text
/
├── README.md                          # You are here
├── /docs/
│   ├── network-setup.md               # Proxmox bridge configuration
│   ├── freefilesync-setup.md          # FreeFileSync installation & batch config
│   ├── automation-setup.md            # RealTimeSync + scheduled tasks
│   └── troubleshooting.md             # Common issues & fixes
├── /configs/
│   ├── sync-job.ffs_batch             # FreeFileSync batch configuration
│   └── real-time-sync.ffs_real        # RealTimeSync configuration
└── /scripts/
    ├── start-sync.bat                 # Windows startup script
    └── check-status.ps1               # PowerShell script to check sync status
Quick Start Guide
Prerequisites
□ Laptop on same network as Proxmox host (192.168.88.x)
□ VM is powered on and accessible at 192.168.88.52
□ FreeFileSync installed on the VM
Step 1: Connect to the VM
From Linux/Mac:

bash
ssh username@192.168.88.52
From Windows (PowerShell/Terminal):

bash
ssh username@192.168.88.52
From Windows (Remote Desktop):

Open Remote Desktop Connection

Computer: 192.168.88.52

Enter your VM credentials

Step 2: Test Connectivity
bash
ping 192.168.88.52
Expected output: Reply from 192.168.88.52: bytes=32 time<1ms TTL=128

Step 3: Configure FreeFileSync
Launch FreeFileSync on the VM

Set your source and destination folders

Save as a batch job: File -> Save as batch job...

Name it: sync-job.ffs_batch

Save to: C:\FFS_Jobs\sync-job.ffs_batch

Step 4: Set Up RealTimeSync
Launch RealTimeSync

Add folders to monitor (your source folder)

Set command:

text
"C:\Program Files\FreeFileSync\FreeFileSync.exe" "C:\FFS_Jobs\sync-job.ffs_batch"
Set idle time: 10 seconds

Save as: C:\FFS_Jobs\real-time-sync.ffs_real

Step 5: Enable Auto-Start
Option A: Startup Folder

text
shell:startup -> Create shortcut to:
"C:\Program Files\FreeFileSync\RealTimeSync.exe" "C:\FFS_Jobs\real-time-sync.ffs_real"
Option B: Task Scheduler (Recommended)

Trigger: At startup

Action: Start RealTimeSync.exe with arguments pointing to your .ffs_real file

Run whether user is logged on or not

Architecture Diagram
text
+-------------+      SSH/RDP      +-----------------+
|   Laptop    | -----------------> |  Proxmox Host   |
|  (Local)    |                    |  (192.168.88.x) |
+-------------+                    +--------+--------+
                                            |
                                            v
                                   +-----------------+
                                   |  VM (Target)    |
                                   | 192.168.88.52   |
                                   +--------+--------+
                                            |
                              +-------------+-------------+
                              v             v             v
                    +-------------+ +-------------+ +-------------+
                    |  Local      | |  Network    | |  External   |
                    |  Drive      | |  Share      | |  USB Drive  |
                    |  (D:\)      | |  (NAS)      | |  (E:\)      |
                    +-------------+ +-------------+ +-------------+
                         ^               ^               ^
                         +---------------+---------------+ 
                                   +-------------+
                                   | RealTimeSync|
                                   |  (Monitors  |
                                   |  changes)   |
                                   +-------------+
Configuration Files
sync-job.ffs_batch (Example)
xml
<?xml version="1.0" encoding="utf-8"?>
<FreeFileSync>
  <Compare>
    <Variant>TimeAndSize</Variant>
  </Compare>
  <Synchronize>
    <Variant>Mirror</Variant>
  </Synchronize>
  <Batch>
    <ProgressDialog>Minimized</ProgressDialog>
    <ErrorDialog>Show</ErrorDialog>
    <WarningDialog>Show</WarningDialog>
  </Batch>
  <Left>
    <Folder>C:\Users\YourUser\Data</Folder>
  </Left>
  <Right>
    <Folder>D:\Backup</Folder>
  </Right>
</FreeFileSync>
real-time-sync.ffs_real (Example)
text
Folders to monitor:
  - C:\Users\YourUser\Data

Command:
  "C:\Program Files\FreeFileSync\FreeFileSync.exe" "C:\FFS_Jobs\sync-job.ffs_batch"

Idle Time: 10 seconds

Auto-start: Enabled (via Task Scheduler)
Verification Checklist
□ VM is reachable at 192.168.88.52 (ping succeeds)
□ FreeFileSync is installed on the VM
□ Batch job (.ffs_batch) created and tested manually
□ RealTimeSync monitors correct folders
□ RealTimeSync launches on VM startup
□ Sync triggers automatically when files change
□ Destination folder is accessible and writable
Troubleshooting
Issue	Cause	Solution
Can't ping 192.168.88.52	VM powered off or network issue	Start VM in Proxmox; verify network adapter
SSH connection refused	SSH service not running	On VM: sudo systemctl enable ssh && sudo systemctl start ssh
Batch job fails	Invalid source/destination paths	Open .ffs_batch in GUI and verify paths
RTS doesn't detect changes	Network drive doesn't support notifications	Monitor local folder instead; use scheduled syncs as backup
Permission denied	VM user lacks write permissions	Grant write access to destination folder
Infinite sync loop	Destination folder inside monitored folder	Separate source and destination folders
Maintenance Notes
Daily Checks
bash
# Check if RealTimeSync is running (Windows)
tasklist | findstr RealTimeSync

# Check sync logs
dir C:\FFS_Jobs\logs\
Weekly Maintenance
Review sync logs for errors

Verify destination has enough storage space

Test manual sync to ensure batch job still works

Monthly Maintenance
Update FreeFileSync to latest version

Review and archive old logs

Test disaster recovery (restore from backup)

License
MIT License

Contributors
Your Name - Intern

Supervisor Name - Project Lead

Related Links
FreeFileSync Official Documentation

Proxmox VE Network Configuration

Important Notes for the Next Intern/Team Member
VM Access: SSH into 192.168.88.52 using provided credentials

Config Location: All configs are in C:\FFS_Jobs\

To Pause Auto-Sync: Kill RealTimeSync.exe in Task Manager

To Resume: Restart the scheduled task or run the startup shortcut

Logs Location: C:\FFS_Jobs\logs\ (check here if sync fails)

Screenshots
[INSERT SCREENSHOT: Proxmox VM Network Settings]
Figure 1: VM network configuration showing IP 192.168.88.52

[INSERT SCREENSHOT: FreeFileSync Batch Configuration]
Figure 2: FreeFileSync sync-job.ffs_batch configuration

[INSERT SCREENSHOT: RealTimeSync Running]
Figure 3: RealTimeSync monitoring folder for changes

[INSERT SCREENSHOT: Task Scheduler Auto-Start Entry]
Figure 4: Windows Task Scheduler setup for automatic startup

Ready to deploy? Follow the Quick Start Guide above or check the /docs/ folder for detailed setup instructions.


