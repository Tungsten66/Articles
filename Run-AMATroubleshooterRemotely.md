# Run AMA troubleshooter without admin rights

This walkthrough demonstrates how to utilitize the action Run Commands to run the AMA troubleshooter without having local admin rights to the Windows server.

## Cited Resources:
- [How to use the Windows operating system (OS) Azure Monitor Agent Troubleshooter](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/run-command)
- [Run scripts in your Windows VM by using action Run Commands](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/run-command)
- [Create an Azure storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal)
- [Create SAS tokens for your storage containers](https://learn.microsoft.com/en-us/azure/ai-services/translator/document-translation/how-to-guides/create-sas-tokens?tabs=Containers)

## Assumptions:
- You already have an existing Azure Storage Account or created one to upload the troubleshooting files to during this walkthrough
- Created a container and generated a SAS token with write permissions and set token to expire in a short amount of time so it can not be malicious reused
- Your account has Virtual Machine Contributor role or higher.  Running a command requires the Microsoft.Compute/virtualMachines/runCommands/action permission. 

## Actions:

1. From the Azure Portal select the virtual machine you want to run the troubleshooter on
2. Select the _Run Command_ blade
3. Click on _RunPowerShellScript_
4. Paste in the below PowerShell code snippet
5. Click run

```powershell

# Variables
## Define the Azure Storage account details
$storageAccountName = "<storageAccountName>"
$containerName = "<containerName>"
$sasToken = "<sasToken>"

# Troubleshooter existence check
if (Test-Path -Path "C:/Packages/Plugins/Microsoft.Azure.Monitor.AzureMonitorWindowsAgent") {
    Write-Host "AMA Agent Troubleshooter is present"
    
    $currentVersion = ((Get-ChildItem -Path "Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure\HandlerState\" | Where-Object Name -like "*AzureMonitorWindowsAgent*" | ForEach-Object { $_ | Get-ItemProperty } | where InstallState -eq "Enabled").PSChildName -split ('_'))[1]

    $troubleshooterPath = "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\$currentVersion\Troubleshooter"
    Set-Location -Path $troubleshooterPath

    # Start the troubleshooter process and wait for it to complete
    $process = Start-Process -FilePath $troubleshooterPath\AgentTroubleshooter.exe -ArgumentList "--ama" -PassThru
    $process.WaitForExit()

    # Get all zip files matching the pattern
    $zipFiles = Get-ChildItem -Path $troubleshooterPath -Filter "AgentTroubleshooterOutput-*.zip"

    foreach ($zipFile in $zipFiles) {

        # Define the storage URI for each zip file
        $storageUri = "https://$storageAccountName.blob.core.usgovcloudapi.net/$containerName/$($zipFile.Name)?$sasToken"

        # Define the headers
        $headers = @{
            "x-ms-blob-type" = "BlockBlob"
            "x-ms-date"      = (Get-Date).ToString("R")
            "x-ms-version"   = "2020-10-02"
        }

        # Upload the zip file to Azure Storage
        Invoke-RestMethod -Uri $storageUri -Method Put -InFile $zipFile.FullName -ContentType "application/zip" -Headers $headers
        Write-Host "Troubleshooting file successfully uploaded" -ForegroundColor Green
    }
}
else {
    Write-Host "AMA Agent Troubleshooter is not present"
    exit
}
```

> [!NOTE]
> The storage URI used is for Azure Governement - If you want to run in Azure Cloud change https://$storageAccountName.blob.core.usgovcloudapi.net to https://$storageAccountName.blob.core.windows.net </br>
>
> For more security you can opt for using Managed identities to grant access to your storage data without the need to include SAS tokens with your HTTP requests.

## Post Condition:
- Connect to the storage account and download and review troubleshooting logs
- Additionally you could do a cleanup on the Windows server by using the Remove-Item command to delete the zip file created after it is uploaded.
