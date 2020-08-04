---
title: "How to: Enable Snapshot Policies for Azure NetApp Files"
date: 2020-08-04T10:37:05-04:00
categories:
- azure netapp files
- data protection
tags:
- azure
- snapshots
- netapp
- cloud
- ransomware
- howto
keywords:
- ransomware
- azure
- snapshots
thumbnailImage: /img/snapshotthumb.png
---
<img src="/img/anf.svg" height="80" align="left" style="margin: 0px 20px 0px 0px;" />
<img src="/img/snapshotthumb.png" height="80" align="left" style="margin: 0px 20px 0px 0px;" />
Snapshot policies are here! Until today, if you wanted to schedule or automate the creation (and retention) of snapshots in the Azure NetApp Files service, you needed to BYOA (bring your own automation). Like any first-party Azure service, ANF supports all of the standard Azure APIs and automation tools, so this wasn't terribly difficult and there was even a [Logic App](https://github.com/ANFTechTeam/anfScheduler) that was simple to deploy and took care of the heavy lifting.

But of course, Microsoft and NetApp continue to bring new features and more value to this great service. There is one tiny caveat... at this time, the snapshot policy feature is currently in preview. But don't worry, the registration process is painless. We'll have you creating snapshot polcies in just a few minutes.

As a prerequisite, you'll need Azure PowerShell installed and connected to your Azure account:
{{< highlight powershell >}}
Install-Module -Name Az -AllowClobber -Scope CurrentUser
{{< / highlight >}}
{{< highlight powershell >}}
Connect-AzAccount
{{< / highlight >}}

Once you have successfully connected to your Azure account, continue with registering the 'ANFSnapshotPolicy' feature:

{{< highlight powershell >}}
Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSnapshotPolicy
{{< / highlight >}}

Lastly, verify the feature is 'Registered':

{{< highlight powershell >}}
Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSnapshotPolicy
{{< / highlight >}}

<img src="/img/snapshotverifyfeature.png" />
<br>
The registration process should take about ten minutes, but your experience may vary slightly. At this point, you should see the 'Data protection' sub-heading and 'Snapshot policy' menu item within the Azure portal (take a look at the screen shot below). You are now ready to create your first snapshot policy. Head on over to the official [Azure NetApp Files documentation](https://docs.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-manage-snapshots) to get started.   
<br>
<img src="/img/snapshotpolicy.png" />
<!--more-->