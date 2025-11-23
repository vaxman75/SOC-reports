# SOC Project - Part A: Web Server Log Analysis

## Overview
This report analyzes suspicious activity found in the web server access logs. The goal is to identify malicious behavior, understand the attack flow, and recommend mitigation steps. All findings are based solely on the provided log entries.

## Attack Flow Diagram (ASCII)
```
             Attacker: 10.0.0.50
                     |
                     | 1. Reconnaissance
                     v
          +---------------------------+
          |   GET /                  | 200
          |   GET /robots.txt        | 404
          |   GET /admin/            | 403
          +---------------------------+
                     |
                     | 2. Find upload.php
                     v
          +---------------------------+
          |   GET /upload.php        | 200
          +---------------------------+
                     |
                     | 3. Upload malicious file
                     v
          +---------------------------+
          |   POST /upload.php       | 200
          |   Uploaded               |
          |   reverse_proxy.php      |
          +---------------------------+
                     |
                     | 4. Execute shell
                     v
          +---------------------------+
          | GET /uploads/reverse_proxy.php
          | GET /uploads/reverse_proxy.php?check=1
          | GET /uploads/reverse_proxy.php?status=poll
          | User-Agent: curl
          +---------------------------+
                     |
                     | 5. Remote access confirmed
                     v
                 [Shell Active]
```

## Incident Summary
On 23 November 2025 the IP address 10.0.0.50 performed multiple actions indicating a structured attack. The attacker used Nmap for directory enumeration, exploited an upload vulnerability, uploaded a malicious PHP file, and executed it using curl to gain remote code execution.

## Timeline of Attack
| Time | Action |
|------|--------|
| 14:01:09 | Initial enumeration via Nmap |
| 14:01:12 | Attempt to access admin directory |
| 14:01:35 | Access to upload.php |
| 14:02:01 | File uploaded successfully |
| 14:02:03 | Executing reverse_proxy.php |
| 14:02:05 | Remote status check |
| 14:04:12 | curl communication to web shell |

## Indicators of Compromise
### Malicious IP
10.0.0.50

### Malicious File
/uploads/reverse_proxy.php

### Suspicious User Agents
Nmap Scripting Engine  
curl 7.81.0

### Vulnerable Component
upload.php

## Impact Assessment
The attacker successfully uploaded and executed arbitrary PHP code on the server. This grants the attacker the ability to run commands remotely, access or manipulate files, elevate privileges, install backdoors, or perform lateral movement.

## Root Cause
The upload.php interface allowed unrestricted file upload without validation, sanitization, or permission separation. The uploads directory allowed execution of uploaded files.

## Recommended Mitigation Steps
1. Block IP 10.0.0.50.  
2. Remove reverse_proxy.php from uploads directory.  
3. Inspect uploads for additional malicious files.  
4. Disable uploads temporarily.  
5. Implement file type whitelisting.  
6. Prevent execution inside uploads directory.  
7. Add authentication to upload endpoint.  
8. Deploy WAF rules.  
9. Harden server configuration and monitoring.

## Conclusion
The evidence confirms a successful compromise of the server using an unrestricted file upload vulnerability. Immediate remediation is required to secure the environment.

## Files Included
- web_access.log  
- web_error.log  
- firewall_nc.log  
- terminal_monitor.log  
- README.md
