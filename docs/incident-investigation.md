# SOC Investigation Case Study: Encoded PowerShell Execution

## Incident Overview

This case study documents the investigation of an encoded PowerShell execution detected within the SOC Detection Engineering & Threat Hunting Home Lab.

The activity was intentionally generated on the Windows 11 endpoint to validate whether endpoint telemetry, Wazuh log collection, and a custom detection rule could identify potentially obfuscated PowerShell execution.

| Field | Value |
|---|---|
| Alert | Encoded PowerShell Command Detected |
| Wazuh Rule | 100109 |
| Severity | Level 14 |
| Endpoint | Windows11-Endpoint |
| Platform | Windows 11 |
| Detection Source | Sysmon / process command-line telemetry |
| SIEM | Wazuh |
| ATT&CK Mapping | T1059.001 — PowerShell |
| Final Verdict | Authorized Lab Simulation / True Positive |

---

## 1. Alert Summary

A custom Wazuh detection generated a Level 14 alert after PowerShell was executed with the `-EncodedCommand` parameter on the monitored Windows 11 endpoint.

Encoded PowerShell can be used legitimately, but it can also obscure the contents of commands and is therefore useful telemetry during security monitoring.

The custom detection rule was designed to identify the presence of `-EncodedCommand` in PowerShell command-line telemetry.

---

## 2. Detection Logic

The following custom Wazuh rule generated the alert:

```xml
<group name="homelab_powershell_encoded,">
  <rule id="100109" level="14">
    <if_sid>92057</if_sid>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)-EncodedCommand</field>
    <description>HomeLab: Encoded PowerShell command detected - possible obfuscated execution.</description>
  </rule>
</group>
```

The rule examines Windows process telemetry and searches the command-line field for the case-insensitive `-EncodedCommand` parameter.

Because encoded PowerShell may indicate an attempt to obscure command contents, the rule was assigned a high alert level for analyst review.

---

## 3. Initial Triage

After the alert appeared in Wazuh, the investigation focused on answering several questions:

1. Which endpoint generated the alert?
2. Which process triggered the detection?
3. What command-line arguments were used?
4. Did the event actually contain encoded PowerShell execution?
5. Was the activity expected or unauthorized?

The alert was associated with the monitored Windows 11 endpoint.

Rather than relying only on the Wazuh rule description, the underlying event was opened in the Threat Hunting interface to inspect the original telemetry.

---

## 4. Evidence Review

The event details exposed the process command line responsible for the detection.

![Encoded PowerShell Investigation](../screenshots/encoded-powershell-investigation.png)

The investigation confirmed that:

- `powershell.exe` was executed.
- The process command line contained `-EncodedCommand`.
- The activity originated from the Windows 11 endpoint.
- The command-line behavior matched the condition defined in Rule 100109.
- Wazuh successfully generated the expected Level 14 alert.

This established a direct relationship between the endpoint activity, collected telemetry, custom detection logic, and resulting SIEM alert.

---

## 5. Detection Pipeline

The event followed the lab's endpoint detection pipeline:

`PowerShell Execution`

↓

`Sysmon Process / Command-Line Telemetry`

↓

`Wazuh Agent`

↓

`Wazuh Manager`

↓

`Custom Rule 100109`

↓

`Level 14 Alert`

↓

`Threat Hunting Investigation`

This demonstrated that the lab could capture endpoint activity and transform raw process telemetry into an actionable custom detection.

---

## 6. Analysis

The presence of `-EncodedCommand` increased the event's investigative value because encoding can make command contents less immediately readable to an analyst.

However, the parameter alone does not prove malicious activity.

Additional context is required before classifying the event as malicious, including the originating user, process context, surrounding activity, and whether the execution was expected.

In this case, the activity was intentionally generated as part of the controlled home-lab validation process.

Therefore, the detection itself was a **true positive** because the rule correctly identified the behavior it was designed to detect, while the security disposition was **authorized lab activity rather than a real compromise**.

---

## 7. MITRE ATT&CK Mapping

### T1059.001 — Command and Scripting Interpreter: PowerShell

The observed behavior involved execution through Windows PowerShell and is mapped to:

**MITRE ATT&CK T1059.001 — PowerShell**

The detection focuses specifically on encoded PowerShell command-line activity.

The ATT&CK mapping describes the observed execution technique; it does not by itself establish malicious intent.

---

## 8. Findings

The investigation produced the following findings:

- Endpoint process telemetry was successfully collected.
- PowerShell command-line arguments were visible to the monitoring pipeline.
- Custom Rule 100109 correctly identified the encoded-command parameter.
- Wazuh elevated the event to a Level 14 alert.
- The underlying telemetry could be examined during investigation.
- The alert was traceable back to the activity that caused it.
- The activity was authorized and intentionally generated for detection validation.

---

## 9. Final Verdict

**True Positive — Authorized Security Test**

Rule 100109 operated as designed and correctly detected encoded PowerShell execution.

No actual compromise occurred because the command was intentionally executed as part of the controlled SOC home-lab exercise.

The test validated the complete detection lifecycle:

**Activity → Telemetry → Detection → Alert → Investigation → Classification**

---

## 10. Potential Analyst Response in a Production Environment

If similar activity occurred unexpectedly in a production environment, additional investigation would be required before determining whether the system was compromised.

An analyst could:

- Identify the user responsible for the PowerShell process.
- Review the complete process command line.
- Decode and inspect the encoded command safely.
- Examine parent and child processes.
- Review surrounding Sysmon and Windows Security events.
- Look for related network connections or file activity.
- Determine whether the execution was associated with approved administrative activity.
- Search other endpoints for similar PowerShell behavior.
- Escalate or contain the endpoint if additional evidence indicated malicious activity.

This highlights an important distinction in security monitoring: a detection identifies behavior worth investigating, while analyst context determines the final security disposition.
