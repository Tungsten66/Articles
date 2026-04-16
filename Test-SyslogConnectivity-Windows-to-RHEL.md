# Test Syslog Connectivity from Splunk (Windows) to Microsoft Sentinel via AMA Collector

This walkthrough demonstrates how to validate syslog connectivity from a Splunk deployment running on Windows Server to a Red Hat Enterprise Linux (RHEL) server that is already configured as a syslog forwarder with the Azure Monitor Agent (AMA). The RHEL collector receives the syslog messages from Splunk and forwards them to a Log Analytics workspace, where they are ingested into Microsoft Sentinel. This is useful for confirming end-to-end network connectivity and verifying the collector is receiving data before investigating Sentinel ingestion issues.

## Cited Resources:
- [Forward Syslog data to a Log Analytics workspace with Microsoft Sentinel by using Azure Monitor Agent](https://learn.microsoft.com/en-us/azure/sentinel/forward-syslog-monitor-agent)
- [rsyslog Documentation](https://www.rsyslog.com/doc/v8-stable/)
- [Red Hat - configuring a remote logging solution](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/configuring-a-remote-logging-solution_managing-monitoring-and-updating-the-kernel)
- [.NET UdpClient Class](https://learn.microsoft.com/en-us/dotnet/api/system.net.sockets.udpclient)
- [RFC 5424 – The Syslog Protocol](https://datatracker.ietf.org/doc/html/rfc5424)
- [Splunk - Configure Syslog output](https://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Forwarddatatothird-partysystemsd)

## Assumptions:
- Splunk is installed and running on a Windows Server.
- Splunk is configured to forward syslog output to the RHEL collector's IP address on UDP port 514.
- The RHEL server (RHEL 7, 8, or 9) is already deployed and configured as a syslog forwarder using the [Azure Monitor Agent (AMA) syslog collection](https://learn.microsoft.com/en-us/azure/sentinel/forward-syslog-monitor-agent) method.
- rsyslog is installed, running, and already configured on the RHEL server to accept remote syslog over UDP 514 as part of the AMA collector setup.
- The Azure Monitor Agent is installed on the RHEL server and associated with a Data Collection Rule (DCR) that targets your Log Analytics workspace and Microsoft Sentinel.
- You have SSH access or console access to the RHEL server.
- UDP port 514 is open between the Windows Server running Splunk and the RHEL collector.
- You have sufficient privileges on both servers (local admin on Windows, sudo/root on RHEL).

---

## Part 1 — Verify the RHEL AMA Collector is Ready

Since the RHEL server is already configured as an AMA syslog collector, these steps confirm the existing configuration is healthy and listening before sending test traffic from Splunk.

### 1. Verify rsyslog is running

SSH into the RHEL server and confirm rsyslog is active:

```bash
sudo systemctl status rsyslog
```

### 2. Verify rsyslog is listening on UDP 514

```bash
sudo ss -ulnp | grep 514
```

Expected output should show rsyslog bound to port 514:

```
UNCONN 0  0  0.0.0.0:514  0.0.0.0:*  users:(("rsyslogd",pid=XXXX,fd=X))
```

### 3. Verify the firewall allows UDP 514 inbound

```bash
sudo firewall-cmd --list-ports
```

If `514/udp` is not listed, add it:

```bash
sudo firewall-cmd --permanent --add-port=514/udp
sudo firewall-cmd --reload
```

### 4. Verify the Azure Monitor Agent is running

```bash
sudo systemctl status azuremonitoragent
```

---

## Part 2 — Send a Test Syslog Message from the Splunk Windows Server

Before modifying any Splunk configuration, use the following PowerShell script to manually send a syslog message from the Windows Server to the RHEL collector. This isolates the network path from Splunk's output configuration and confirms whether connectivity is the issue.

```powershell
function Send-SyslogMessage {
    param(
        [string]$Server      = "192.168.1.100",   # RHEL AMA collector IP or hostname
        [int]$Port           = 514,
        [int]$Facility       = 1,                  # 1 = User-level messages (match your DCR facility)
        [int]$Severity       = 6,                  # 6 = Informational
        [string]$Hostname    = $env:COMPUTERNAME,
        [string]$AppName     = "Splunk",           # Mimics Splunk as the source application
        [string]$Message     = "Test syslog message from Splunk Windows Server"
    )

    # RFC 5424 priority = (Facility * 8) + Severity
    $Priority  = ($Facility * 8) + $Severity
    $Timestamp = (Get-Date).ToString("yyyy-MM-ddTHH:mm:ss.fffzzz")

    # RFC 5424 formatted syslog message
    $SyslogMessage = "<$Priority>1 $Timestamp $Hostname $AppName - - - $Message"

    $UdpClient = New-Object System.Net.Sockets.UdpClient
    try {
        $Bytes = [System.Text.Encoding]::UTF8.GetBytes($SyslogMessage)
        $UdpClient.Send($Bytes, $Bytes.Length, $Server, $Port) | Out-Null
        Write-Host "Syslog message sent to ${Server}:${Port}" -ForegroundColor Green
        Write-Host "Payload: $SyslogMessage"
    }
    catch {
        Write-Error "Failed to send syslog message: $_"
    }
    finally {
        $UdpClient.Close()
    }
}

# Send a test message — update Server to your RHEL AMA collector IP or hostname
Send-SyslogMessage -Server "192.168.1.100" -Message "Splunk connectivity test from $env:COMPUTERNAME at $(Get-Date)"
```

> Update the `-Server` parameter to the IP address or hostname of your RHEL AMA collector. Also ensure the `-Facility` matches the facility configured in your Data Collection Rule (DCR) — mismatched facilities are a common reason messages arrive at the collector but do not appear in Sentinel.

---

## Part 3 — Verify Receipt on the RHEL AMA Collector

After running the PowerShell script, switch back to the RHEL server and confirm the message was received. If the message arrives here but does not appear in Sentinel, the issue is with the AMA/DCR configuration rather than network connectivity.

### Option 1 — Watch the syslog file in real time

If you configured the `RemoteLogs` template in Part 1, look for a new directory under `/var/log/remote/`:

```bash
sudo tail -f /var/log/remote/<WINDOWS_HOSTNAME>/Splunk.log
```

Replace `<WINDOWS_HOSTNAME>` with the NetBIOS name of the Windows Server running Splunk.

### Option 2 — Check the system messages log

```bash
sudo tail -f /var/log/messages | grep -i "Splunk\|<WINDOWS_HOSTNAME>"
```

### Option 3 — Use tcpdump to capture packets at the network level

This confirms packets are arriving regardless of rsyslog processing:

```bash
sudo tcpdump -i eth0 udp port 514 -A
```

> Replace `eth0` with your actual network interface name. Use `ip link` to list available interfaces.

A successful capture will display the raw syslog payload including the RFC 5424 header and your message text.

---

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| No output in `/var/log/messages` | rsyslog not listening on UDP 514 | Uncomment `imudp` lines in `/etc/rsyslog.conf` and restart |
| tcpdump shows no packets | Firewall blocking UDP 514 | Run `firewall-cmd --add-port=514/udp` on RHEL and check Windows firewall |
| PowerShell throws a socket error | Wrong IP or port unreachable | Verify IP, run `Test-NetConnection -ComputerName <IP> -Port 514` as a TCP check |
| Messages arrive on RHEL but not in Sentinel | DCR facility/severity mismatch | Confirm the facility in the DCR matches what Splunk and the test script are sending |
| Messages arrive on RHEL but not in Sentinel | AMA not forwarding | Run `sudo systemctl status azuremonitoragent` and check AMA health |
| Splunk not sending syslog | Splunk output not configured | Verify Splunk syslog output stanza targets the RHEL collector IP and port 514 |
| SELinux blocking rsyslog | SELinux policy denial | Run `sudo ausearch -m avc -ts recent` and check for rsyslog denials |

### Check SELinux denials on RHEL

```bash
sudo ausearch -m avc -ts recent | grep rsyslog
```

If SELinux is blocking remote syslog reception, apply the appropriate policy:

```bash
sudo setsebool -P nis_enabled 1
```

Or check the audit log for the exact denial and generate a local policy module if needed.
