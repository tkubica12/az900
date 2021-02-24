# User guide
Please follow instructor and refer to steps and scripts here.

## IaaS

1. Use portal at https://portal.azure.com to create VM with VNET, public IP and no monitoring. In wizard enable backup and use custom storage account for boot diagnostics (to allow for serial console access).
2. Access VM via SSH/RDP
3. Together with instructor investigate objects - VNET, NSG, disk, discuss snapshots and images etc.
4. Configure NSG to allow SSH/RDP access only from your source IP (to get it you can use [http://api.ipify.org/](http://api.ipify.org/))
5. Instructor will check from different source IP you have correctly blocked this traffic

## Cloud Shell

1. Click on >_ to open cloud shell and select Bash or PowerShell
2. Use Bash or PowerShell to create 5 disks in Standard SSD tier
3. Upgrade disks to Premium SSD tier
4. Create additional 5 disks in Standard SSD tier
5. Resize disks that are Standard SSD (so only 5 of 10)
6. Destroy everything

**Azure CLI with Bash option**

```bash
# Create resource group
export resourceGroup=tomaskubica-disks-rg
az group create -n $resourceGroup -l westeurope

# Create 5 disks
for i in {1..5}
do
   az disk create -n disk$i -g $resourceGroup --sku StandardSSD_LRS --size-gb 32
done

# Get all disks in resource group and upgrade to Premium SSD
for disk in $(az disk list -g $resourceGroup --query [].id -o tsv)
do
    az disk update --sku Premium_LRS --ids $disk
done

# Create another 5 disks in Standard SSD tier
for i in {1..5}
do
   az disk create -n anotherdisk$i -g $resourceGroup --sku StandardSSD_LRS --size-gb 32
done

# Resize all disk in resource group that are of Premium SSD tier
for disk in $(az disk list -g $resourceGroup --query "[?sku.name=='Premium_LRS'].id" -o tsv)
do
    az disk update --size-gb 64 --ids $disk
done

# Destroy resource group
az group delete -n $resourceGroup -y --no-wait
```

**PowerShell Az module option**

```powershell
# Create resource group
$resourceGroup = "tomaskubica-disks-rg"
New-AzResourceGroup -Name $resourceGroup -Location westeurope

# Create 5 disks
For ($i=1; $i -le 5; $i++) {
    New-AzDiskConfig -Location westeurope -DiskSizeGB 32 -SkuName StandardSSD_LRS -CreateOption Empty `
        | New-AzDisk -ResourceGroupName $resourceGroup -DiskName disk$i
}

# Get all disks in resource group and upgrade to Premium SSD
Get-AzDisk -ResourceGroupName $resourceGroup | Update-AzDisk -ResourceGroupName $resourceGroup -DiskUpdate $(New-AzDiskUpdateConfig -SkuName Premium_LRS)

# Create another 5 disks in Standard SSD tier
For ($i=1; $i -le 5; $i++) {
    New-AzDiskConfig -Location westeurope -DiskSizeGB 32 -SkuName StandardSSD_LRS -CreateOption Empty `
        | New-AzDisk -ResourceGroupName $resourceGroup -DiskName anotherdisk$i
}

# Resize all disk in resource group that are of Premium SSD tier
foreach ($disk in Get-AzDisk -ResourceGroupName $resourceGroup) {
    if ($disk.Sku.Name -eq "Premium_LRS") {
        Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $disk.Name -DiskUpdate $(New-AzDiskUpdateConfig -DiskSizeGB 64)
    }
}

# Destroy resource group
Remove-AzResourceGroup -Name $resourceGroup -Force -AsJob
```

## Monitoring and security

1. Your subscription has been configured with Policy to autoenroll to Azure Monitor and Security Center
2. Go to Inventory and Change tracking and enable
3. Access machine via Serial console
4. Create Backup
5. Configure replication to different region
6. Go to Insights and see Health, Performance and Map
7. Go to Logs and search Syslog (Linux) or Event (Windows) table
8. Go to Security Center and explore recommendations, vulnerabilities, missing updates etc.

## Azure SQL and other databases

1. Create Azure SQL, use serverless SKU for testing to save costs
2. Click Configure to see how you can scale or up down
3. Click Geo-replication and see how you can create async-replicas in other regions
4. See security features such as TDE, data masking or Always Encrypted
5. Restore button allows for point-in-time data restoration
6. Automatic tuning can be used for DB to optimize itself

## Kubernetes cluster

1. Use GUI to create simple Kubernetes cluster with monitoring enabled (use kubenet networking for simplicity)
2. Observe cluster GUI in AKS, policies, upgrades, logging and telemetry capabilities
3. As part of fundamentals training we will stop here - if you are fluent in Kubernetes you can explore more later on your own

## Application Services

1. Create WebApp using .NET Core 3.1 runtime and Windows host on new plan with P1v2
2. Click on listed URL to see your web running
3. Go to App Service Editor and create new file (using right click) named default.aspx with following code:

```
I am alive! <%=System.Environment.MachineName %>
```

4. Refresh web page to see new version of your app
5. Go to Scale out, increase number of nodes and see how traffic is balanced (using curl command as browser might be caching). Also check option around autoscaling.
6. Go to Deployment slots and create new slot called staging
7. Click on staging slot, go to App Service Editor and create new file (using right click) named default.aspx with following code:

```
VERSION 2 <%=System.Environment.MachineName %>
```

8. Go to this slot app URL and see new version is there (you have just performed A/B testing as you can see old and new version side by side)
9. Go back to main app and change percentage of traffic between slots from 100/0 to 80/20. Using curl you will see that about every fifth user (based on cookie affinity) will get new version (you have just performed canary release)
10. Click on swap slots to release version 2 to production for everyone (you have just performed green/blue deployment)

## Serverless with Azure Function App

1. Create Azure Function App with Node.js and Windows host and Application Insights enabled.
2. Create Storage Account and container "outcontainer" and queue "myqueue"
3. Create Function with storage queue trigger reacting on myqueue in storage account you create previously
4. Open Monitor page, go to storage account and create message - see logs
5. Modify Integrations to include output binding to write files to blob storage
6. Modify code and test

```javascript
module.exports = async function (context, myQueueItem) {
    context.log('JavaScript queue trigger function processed work item', myQueueItem);
    context.bindings.outputBlob = myQueueItem;
    context.done();
};
```

7. Create couple of messages and see blobs being created
8. Go to Application Insights and see application monitoring -> Application map, Transaction search, 

## Cognitive services

1. Open cognitive vision https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/#features
2. Try built in or your own pictures
3. Open text to speech: https://azure.microsoft.com/en-us/services/cognitive-services/text-to-speech/
4. Select Czech and neural model (Vlasta), put in some text and hit play