# Project 2: Brute-Force Detection with Windows Event Viewer and Automation

**Date:** August 11, 2026
**Environment:** Windows 11, Event Viewer, PowerShell
**Objective:** Analyze failed logon events (Event ID 4625), simulate a realistic brute-force scenario, and build an automated detection script.

---

## 1. Executive Summary

In this project, failed logon events (Event ID 4625) in the Windows Security log were manually reviewed. An automated, rapid sequence of failed logon attempts (a brute-force simulation) was then generated using PowerShell. The distinguishing characteristics of this pattern versus human behavior (short time intervals, constant speed) were analyzed. Finally, a PowerShell script was developed to automatically detect when the number of failed logons within a given time window exceeds a threshold, producing a red/green alert output; findings were exported to CSV.

## 2. Methodology and Findings

### 2.1 Manual Log Analysis
Failed logon auditing was enabled via Local Security Policy, and the Security log was filtered by `Event ID 4625` in Event Viewer.

**Fields reviewed:** Account Name, Logon Type, Failure Reason, Source Network Address, Time Created.

**Finding:** Manual/individual failed logon attempts showed timestamps spread across minutes — consistent with human behavior.

### 2.2 Brute-Force Simulation
```powershell
for ($i=1; $i -le 10; $i++) {
    $wrongPass = ConvertTo-SecureString "yanlisSifre123" -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential("$env:USERNAME", $wrongPass)
    Start-Process cmd.exe -Credential $cred -ErrorAction SilentlyContinue
    Start-Sleep -Milliseconds 300
}
```
This script generated 10 consecutive failed logon attempts against the same account at 300ms intervals.

**Finding:** These attempts appeared clustered within a narrow window of a few seconds in Event Viewer — a constant, very short interval is a typical signature of automated (script/bot) behavior.

### 2.3 Correlation Analysis — Manual vs. Automated Behavior

| Attribute | Manual Attempt | Scripted (Automated) Attempt |
|---|---|---|
| Number of attempts | 3–4 | 10 |
| Time interval | Minutes | Seconds (~300ms constant) |
| Behavior pattern | Human | Automated/bot |

**Conclusion:** The combination of same account + short time interval + constant speed is a classic brute-force attack signature.

### 2.4 Automated Detection Script
```powershell
$timeWindow = (Get-Date).AddMinutes(-5)
$threshold = 5

$failedLogons = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
    StartTime = $timeWindow
}

if ($failedLogons.Count -ge $threshold) {
    Write-Host "ALERT: Possible brute-force detected!" -ForegroundColor Red
    Write-Host "$($failedLogons.Count) failed logon attempts found in the last 5 minutes." -ForegroundColor Yellow
    $failedLogons | Select-Object TimeCreated, Id | Format-Table -AutoSize
} else {
    Write-Host "Normal level: $($failedLogons.Count) failed logons in the last 5 minutes." -ForegroundColor Green
}
```

This script counts `4625` events within a defined time window (5 minutes) and raises a red alert if the count exceeds the threshold (5) — a simplified simulation of basic SIEM alert logic.

**Issue encountered and resolution:** The script initially failed silently in a standard (non-administrator) PowerShell session (`-ErrorAction SilentlyContinue` suppressed the error, producing a misleading "0 events" result). Root cause: reading the Security log requires administrator privileges. Re-running the script in an elevated PowerShell session successfully produced the expected ALERT output.

### 2.5 Exporting Findings
```powershell
$failedLogons | Select-Object TimeCreated, Id, @{Name="Message";Expression={$_.Message}} | Export-Csv -Path "$env:USERPROFILE\brute_force_log.csv" -NoTypeInformation
```
Detected events were exported to CSV for further review in Excel.

## 3. Key Concepts Learned

- Windows Security log analysis and the Event ID concept (4625 = failed logon)
- Distinguishing manual vs. automated (bot) behavior patterns through time analysis
- Querying logs with PowerShell (`Get-WinEvent`, `-FilterHashtable`)
- Writing basic threshold-based detection logic — a simulation of core SIEM alerting concepts
- The risk of "silent failure": a tool reporting "no results" does not always mean "no events occurred" — permission issues can produce false negatives
- Exporting findings to CSV for reporting

## 4. Notes

This project originally planned to ingest findings into Splunk and configure a real SIEM alert rule; however, a technical obstacle was encountered during installation. This step is planned to be completed as a separate follow-up project.
