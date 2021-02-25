# Admin guide - what to prepare for hands-on labs

## Steps for admin
1. Create new subscription in your tenant preferably using EA -> delete after training
2. Make sure sufficient quota on cores is assigned (will be OK with EA, but check with other types - each participant will need up to 8 cores of B/series or D/series) -> trial subscriptions are too limited to be used as shared resource for whole group
3. Invite me as Owner on subscription scope
4. Invite attendees as Owner on subscription scope

## Subscription preparation (I can do that after gaining access)
1. Create resource group for monitoring
2. Create Log Analytics workspace
3. Configure Security Center to Standard tier, autoenroll to created workspace, configure to collect logs
4. Go to Azure Monitor -> Virtual Machines -> Other onboarding options -> Enable using Policy to autoenroll VMs to VM Insights
5. Create Automation Account
6. In Automation Account configure Inventory, Change Tracking and Update management and then click on Manage machines and select Enable on all available and future machines

## Check participants finished blocking of SSH/RDP traffic
```powershell
$WarningPreference = 'SilentlyContinue'
Get-AzPublicIpAddress | ForEach-Object -Parallel  {
    $ssh = Test-NetConnection -ComputerName $_.IpAddress -Port 22 -InformationLevel Quiet
    $rdp = Test-NetConnection -ComputerName $_.IpAddress -Port 3389 -InformationLevel Quiet
    $output = "Resource Group:{0,-20} IP:{1,-16} SSH:{2,-6} RDP:{3,-6}" -f $_.ResourceGroupName, $_.IpAddress, $ssh.ToString(), $rdp.ToString()
    Write-Host $output
} -ThrottleLimit 100
```

## Delete all Backup Vaults
```powershell
foreach ($vault in Get-AzRecoveryServicesVault) {
    # Recover any soft-deleted backups
    $Containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -VaultId $vault.Id
    foreach ($Container in $Containers) {
        $BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureVM -VaultId $vault.Id -DeleteState ToBeDeleted
        if ($BackupItem) {
            Undo-AzRecoveryServicesBackupItemDeletion -Item $BackupItem -VaultId $vault.Id
        } 
    }

    # Disable soft-delete
    Set-AzRecoveryServicesVaultProperty -VaultId $vault.Id -SoftDeleteFeatureState Disable

    # Delete all VMs and backups
    $Containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -VaultId $vault.Id
    foreach ($Container in $Containers) {
        $BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureVM -VaultId $vault.Id
        if ($BackupItem) {
            Disable-AzRecoveryServicesBackupProtection -Item $BackupItem -VaultId $vault.Id -RemoveRecoveryPoints -Force
        }
    }

    # Delete vault
    Remove-AzRecoveryServicesVault -Vault $vault
}

```