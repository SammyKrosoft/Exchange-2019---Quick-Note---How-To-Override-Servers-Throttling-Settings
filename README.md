# Exchange-2019 Quick-Note: How To and Why Override Servers Throttling Settings

*DISCLAIMER: Overriding server database throttling should only be considered a temporary solution for database throttling issues and is preferable for known high-load scenarios (e.g., temporary bursts of mailbox activities or mass emailing to all users). Additionally, this action should only be taken if your server has the resources to handle the additional load (e.g., CPU usage not exceeding 70% on average, at least 1GB of available RAM, and no disk latency). It is strongly recommended to work with Microsoft Support to identify the root cause of the database throttling.*

## Issue

Exchange Server 2019 was experiencing throttling issues, causing delays in email delivery to databases. The following error messages were observed:

- `432 4.3.2 STOREDRV.Deliver - dynamic mailbox database throttling limit exceeded`
- `451 4.4.397 Error communicating with target host -> 421 4.4.2 Connection dropped due to timeout`

Emails were queuing with these error messages, particularly the first one, which seemed to occur immediately after the second one in chronological order.

Modifications to disable throttling had no impact on Exchange Server 2019 when adjusting the `EdgeTransport.exe.config` configuration file (located at `%ExchangeInstallPath%Bin\EdgeTransport.exe.config`).

We tried disabling the throttling by editing the aforementioned file and adding or modifying the following lines (if already present):

- First, we tried to disable throttling (a restart of MS Exchange Transport services is necessary for the settings to be updated in memory and applied to mail flow processing):

  ```xml
  add key="MailboxDeliveryThrottlingEnabled" value="False"
  ```

> Note: You can restart the Exchange transport services using the Services window (for example, you can launch it by using Windows + R then type `services.msc`), or using the following PowerShell command:
> 
> ```powershell
> Restart-Service MSExchangeTransport # on ALL SERVERS
> ```

- Then, we tried these two settings as well in addition to the above (each time we modified the .exe.config file, a restart of MS Exchange Transport services was necessary):

  ```xml
  add key="RecipientThreadLimit" value="2"
  add key="MaxMailboxDeliveryPerMdbConnections" value="3"
  ```

See the above note for options to restart your Transport service on OnPrem Exchange server(s).


But the above did not have any effect on our Exchange 2019 servers. Microsoft Support then asked us to run the below commands (you can revert the changed when the issue is gone, or if the below pin your CPU to 100%)

## Resolution (more a workaround)

Here are a couple of PowerShell commands to help removing Database or mail flow throttling on OnPrem servers (Exchange 2019, applicable to Exchange 2016) in the following conditions:

- your CPU utilization is **not** pinned to 80%-100%
- your have sufficient free RAM
- your disk latency is below 50 milliseconds for your databases disks, and below 20 milliseconds for your log disks (check the `Logical disk \ Avg Disk sec/read` and `\Avg Disk sec/write` performance counters for your database disks as well as your log disks if these are on separate disks)

> NOTE: We can remove temporarily the database throttling because we had 20% CPU utilization max, and more than 1GB memory free. Do NOT do this if your CPU is already pinned at 100% ! Continue the investigation, or offload the server from mailboxes (failover some databases to remove some load), or add CPU/RAM resources if you have virtual servers.

```powershell

New-SettingOverride –Name "DisableMSIPC" -Component Encryption –Section UseMSIPC –Parameters @("Enabled=false") -Reason "Disabling MSIPC stack" # on 1 SERVER
Get-ExchangeDiagnosticInfo -Process Microsoft.Exchange.Directory.TopologyService -Component VariantConfiguration -Argument Refresh # on ALL SERVERS
Restart-Service MSExchangeTransport # on ALL SERVERS

```
