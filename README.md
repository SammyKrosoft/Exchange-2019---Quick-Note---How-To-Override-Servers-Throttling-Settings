# Exchange-2019 Quick-Note: How To Override Servers Throttling Settings

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

- Then, we tried these two settings as well in addition to the above (each time we modified the .exe.config file, a restart of MS Exchange Transport services was necessary):
```xml
add key="RecipientThreadLimit" value="2"
add key="MaxMailboxDeliveryPerMdbConnections" value="3"
```

## Resolution (more a workaround)

A couple of PowerShell commands to help removing Database or mail flow throttling on OnPrem servers (Exchange 2019, applicable to Exchange 2016)

> NOTE: We can remove temporarily the database throttling because we had 20% CPU utilization max, and more than 1GB memory free. Do NOT do this if your CPU is already pinned at 100% ! Continue the investigation, or offload the server from mailboxes (failover some databases to remove some load), or add CPU/RAM resources if you have virtual servers.

```powershell

New-SettingOverride –Name "DisableMSIPC" -Component Encryption –Section UseMSIPC –Parameters @("Enabled=false") -Reason "Disabling MSIPC stack" # on 1 SERVER
Get-ExchangeDiagnosticInfo -Process Microsoft.Exchange.Directory.TopologyService -Component VariantConfiguration -Argument Refresh # on ALL SERVERS
Restart-Service MSExchangeTransport # on ALL SERVERS

```
