# Installing WSL on Win10, Win11 or Windows Server 2022+
## Enable WSL

In an admin shell:

C:\Windows\system32>dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
C:\Windows\system32>dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
C:\Windows\system32>dism.exe /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V

Restart (if it wasn't already installed) 

C:\Windows\system32>Restart-Computer -Force

