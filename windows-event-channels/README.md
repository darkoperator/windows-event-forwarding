# Windows Event Channels
As described in the blog post [Creating Custom Windows Event Forwarding Logs](https://blogs.technet.microsoft.com/russellt/2016/05/18/creating-custom-windows-event-forwarding-logs/), WEF can be extended with additional custom event channels. Extending the number of event channels available provides a few primary benefits:

* Each event channel can have an independent maximum size and rotation strategy.
* Each event channel can be used as a unique identifier for tagging data for ingestion into a SIEM. 
* Event channels may be placed on different disks or storage devices for improving disk I/O. 

The Event Channel manifest provided in this project consists of 16 individual providers, each with 7 channels. Channels follow a standard naming scheme of WEC[#], where the number is related to the provider. 

Once the Event Channel manifest has been compiled into a DLL, it is loaded onto the WEC server, where it will register and create the appropriate channels and log files.

If you're like us and don't trust random DLLs, feel free to use our manifest file and build your own.

## List of WEF channels
* **WEC-Powershell**: Event channel for collecting PowerShell events.
* **WEC-WMI:** Event channel for collecting WMI events.
* **WEC-EMET**: Event channel for collecting EMET events.
* **WEC-Authentication**: Event channel for collecting authentication events.
* **WEC-Services**: Event channel for collecting services events.
* **WEC-Process-Execution**: Event channel for collecting process creation/termination events.
* **WEC-Code-Integrity**: Event channel for collecting device guard and code integrity events.
* **WEC2-Registry**: Event channel for collecting registry audit events.
* **WEC2-File-System**: Event channel for collecting filesystem audit events.
* **WEC2-Applocker**: Event channel for collecting applocker events.
* **WEC2-Task-Scheduler**: Event channel for collecting scheduled task and at events.
* **WEC2-Application-Crashes**: Event channel for collecting application crash events.
* **WEC2-Windows-Defender**: Event channel for collecting windows defender events.
* **WEC2-Group-Policy-Errors**: Event channel for collecting group policy error events.
* **WEC3-Drivers**: Event channel for collecting driver events.
* **WEC3-Account-Management**: Event channel for collecting account management events.
* **WEC3-Windows-Diagnostics**: Event channel for collecting diagnostic events.
* **WEC3-Smart-Card**: Event channel for collecting smart card events.
* **WEC3-USB**: Event channel for collecting USB events.
* **WEC3-Print**: Event channel for collecting printer and print job events.
* **WEC3-Firewall**: Event channel for collecting firewall events.
* **WEC4-Wireless**: Event channel for collecting 802.1 wireless events.
* **WEC4-Shares**: Event channel for collecting SMB share events.
* **WEC4-Bits-Client**: Event channel for collecting BITS Client events.
* **WEC4-Windows-Update**: Event channel for collecting windows update events.
* **WEC4-Hotpatching-Errors**: Event channel for collecting hotpatching error events.
* **WEC4-DNS**: Event channel for collecting DNS query and DLL loading events.
* **WEC4-System-Time-Change**: Event channel for collecting time change events.
* **WEC5-Operating-System**: Event channel for collecting operating system events.
* **WEC5-Certificate-Authority**: Event channel for collecting CA events.
* **WEC5-Crypto-API**: Event channel for collecting crypto API events.
* **WEC5-MSI-Packages**: Event channel for collecting package installation events.
* **WEC5-Log-Deletion-Security**: Event channel for collecting log deletion events.
* **WEC5-Log-Deletion-System**: Event channel for collecting log deletion events.
* **WEC5-Autoruns**: Event channel for collecting Autoruns-To-Wineventlog events.

## Pre-Requisites:
You will need the following software to build the DLL:
- Windows 10 SDK
- Windows Workstation

## Editing:
Launch the Manifest Generator
```
�C:\Program Files (x86)\Windows Kits\10\bin\x64\ecmangen.exe�
```
Load the CustomEventChannels.man file from this repo.

Make any changes to the file. Ensure the following settings are observed:
- All channels are marked as Operational and Enabled.
- No more than 7 channels are added to each provider.
- Channels following the naming scheme (WEC#-Name)
- Symbols use underscores and not hyphens.

## Compiling:
To compile, perform the following from a cmd.exe shell:
```
"C:\Program Files (x86)\Windows Kits\10\bin\x64\mc.exe" CustomEventChannels.man
"C:\Program Files (x86)\Windows Kits\10\bin\x64\mc.exe" -css CustomEventChannels.DummyEvent CustomEventChannels.man
"C:\Program Files (x86)\Windows Kits\10\bin\x64\rc.exe" CustomEventChannels.rc
"csc.exe" /win32res:CustomEventChannels.res /unsafe /target:library /out:CustomEventChannels.dll C:CustomEventChannels.cs
```

## Deployment:

For each WEF server you need to deploy this to, perform the following:

1) Disable the Windows Event Collector Service:

```
net stop Wecsvc
```

2) Disable all current WEF subscriptions.


3) Unload the current Event Channel file:
```
wevtutil um C:\windows\system32\CustomEventChannels.man
```

4) Copy (and replace) the following files to each WEF server under C:\Windows\system32:
```
CustomEventChannels.dll
CustomEventChannels.man
```

5) Load the new Event Channel file:
```
wevtutil im C:\windows\system32\CustomEventChannels.man
```

6) Resize the log files:
```
$xml = wevtutil el | select-string -pattern "WEC"
    foreach ($subscription in $xml) { 
      wevtutil sl $subscription /ms:4194304 
    }
```

7) Re-enable the WEF subscriptions.

8) Re-enable the Windows Event Collector service
